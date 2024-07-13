Optimisation
===============


Per-tick cache
--------------

BharatSim includes a framework-defined map-based temporary data store, which is used as a `cache <https://en.wikipedia.org/wiki/Cache_(computing)>`_. Accessing the data store for each individual agent is a time- and resource-intensive process. This is undesirable, especially if the data that is being accessed from the store has already been accessed before. In such cases, accessing the store more than once is redundant. The per-tick cache is designed to speed up this process. If the value is already computed, it can be stored in the per-tick cache, from where it can be accessed by future calls. This storage is *temporary*, since the per-tick cache is automatically cleaned at the beginning of each tick. This allows the cache to remain manageably small, and quick to access.

Example: storing location-level information
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In general, the per-tick cache is most useful when the same data needs to be retrieved for multiple agents. Here, we describe one particular implementation in the context of a standard epidemiological simulation.

In an agent-based model of disease transmission, we need to compute the number of susceptible individuals at every time-step that get infected. This number, as we have described in detail in :ref:`A Basic Introduction to Epidemic Modelling`, is decided by the number of infected individuals they are in contact with. This number is related to the *probability* that each susceptible agent has of getting infected. As a result, one might naively expect that this quantity has to be computed for every susceptible individual. This can be a costly process, since it requires accessing the underlying *Graph* database.

However, at every time-step, the number of infected individuals in a location is assumed to not change. As a result, for a given location, once the number of infected in that location is computed, it does not need to be computed again. This can be implemented using the per-tick cache. A modeller can store the number of infected computed in each location in the cache, so that all future calls will simply access the cache, and not search through the entire population.

In order to implement this sucessfully, the modeller will need to specify a unique key under which to store the data in a `HashMap <https://en.wikipedia.org/wiki/Hash_table>`_. The data will be computed if and only if that key is not present. In the example below, we will define the unique key to be the type of location ("Home", "Work", or "School"), followed by its id.

To make this more concrete, let us take a specific example: consider a case where a specific ``Office`` (with ``id = 12``) has 1000 agents, of which 10 are infected and the remaining susceptible. If we needed to compute the number of infected separately for each agent, we would have to query the graph for all 990 susceptible agents. However, using the per-tick cache, the value of "10" could be stored under the key ``Office1000``, which would only have to be calculated once, and would then simply be accessed 989 times from the per-tick cache. As a result, 989 fewer calls will be made to the graph database, which could significantly speed up the simulation.

Below, we describe how to implement this in the context of the SIR model we've described so far. Note that in our example, it is not the *number* of infected but the *fraction* of infected that is important. We will thus be storing this fraction to the cache. However, the idea remains the same.

.. note::
  BharatSim's per-tick cache implementation is present in the ``perTickCache`` function of the framework's ``Context``.
..
  TODO: Describe the Context explicitly in Framework Basics.


Our implementation requires us to define two functions:

* ``computeTheValue`` which explicitly goes through the graph and computes the fraction of infected in a specific location ``Node`` and returns this number.
* ``fetchInfectedFraction`` which accepts the location ``Node`` and the ``placeType`` and creates the unique key and checks the framework-defined ``perTickCache`` to see if the value has already been computed. If it has, it returns the value from the cache. If it hasn't, it calls ``computeTheValue`` and stores the value in the cache.

.. code-block:: Scala

    private def computeTheValue(node: Node): Double = {
    val total = node.getConnectionCount(node.getRelation[Person]().get)
    if (total == 0)
      return 0.0
    val infected = node.getConnectionCount(node.getRelation[Person]().get, "infectionState" equ Infected)

    infected.toDouble/total.toDouble
    }

    private  def fetchInfectedFraction(node: Node, placeType: String, context: Context): Double = {
    val cache = context.perTickCache
    val uniqueKey = (placeType, node.internalId)
    cache.getOrUpdate(uniqueKey, () => computeTheValue(node)).asInstanceOf[Double]
    }


Outputting data to a CSV
------------------------

The target to be optimized
~~~~~~~~~~~~~~~~~~~~~~~~~~

In the :ref:`section on writing output to a file <Writing output to file>` (in :ref:`Writing your First Program`), we use a user-defined class called ``SIROutputSpec``. In the ``getRows`` method of the class, we obtain the number of people in each compartment with the following lines of code:

.. code-block:: scala

      graphProvider.fetchCount(label, "infectionState" equ Susceptible),
      graphProvider.fetchCount(label, "infectionState" equ Infected),
      graphProvider.fetchCount(label, "infectionState" equ Removed)

.. note:: 

   These lines of code are being run at every time step.

``fetchCount`` iterates over every node in the graph, and checks if the node matches a particular pattern (in this case, nodes with a label of ``Person`` who fit the desired ``infectionState``). Looping over the graph is fairly expensive computationally, and can use up large amounts of time for a large population size.

A (naive) approach to help
~~~~~~~~~~~~~~~~~~~~~~~~~~

We can fix this problem in a fairly simple manner: We only need to iterate over the graph once, and check *all the patterns* during the iteration. First, we initialize a hash map to store the results: The keys will be strings (one for each infection state) and the values the number of people in the state.

.. code-block:: scala

    val countMap = mutable.HashMap.empty[String, Int]


We then get all of the ``Person`` nodes on the graph

.. code-block:: scala

    val nodes = graphProvider.fetchNodes(label)

We now iterate over the nodes, typecaste them to the ``Person`` class, check the appropriate ``infectionState`` attribute, and increment ``countMap`` accordingly.

.. code-block:: scala

    nodes.foreach(node => {
      val infectedState = node.as[Person].infectionState
      val existingCount = countMap.getOrElse(infectedState, 0)
      countMap.put(infectedState, existingCount + 1)
    })

.. hint::

  The typecasting is done because ``fetchNodes`` returns an iterator of ``GraphNode`` objects, which lack the ``infectionState`` attribute that we need.

We can then fairly easily obtain the counts from the hashmap, using the ``getOrElse`` attribute (which returns the value present in the map, and if it doesn't exist, returns a set default)

.. code-block:: scala

    countMap.getOrElse(Susceptible.toString, 0)
    countMap.getOrElse(Infected.toString, 0)
    countMap.getOrElse(Removed.toString, 0)

However, this approach can take even longer than the original one, despite only looping over the graph once instead of thrice. What went wrong?

The answer is in the **typecasting**: using ``as`` creates a new instance of the ``Person`` class. As this is being done for every node on the graph, it takes a fair amount of time, and ends up slowing down the code to the point that it's slower than the original.


The fastest solution
~~~~~~~~~~~~~~~~~~~~

We only want a single parameter of the ``Person`` class (namely, the ``InfectionState``). As such, there's another method that we can use to do the same thing as typecasting, but faster.

.. caution::

    The method described below works in this specific use case, but may not in others. Furthermore, it's implementation is not particularly readable: consider the tradeoff between readability and performance, and what's right for you.

Using the ``getParams`` and ``apply`` methods of the ``GraphNode`` class together, we can obtain a parameter of the node:

.. code-block:: scala

    val infectedState = node.getParams.apply("infectionState").toString

.. caution::

  This will only work if you know for a fact that your node is a ``Person``: Houses, workplaces, etc are also stored as nodes on the graph, and so you have to be certain that you're only running the above line on the appropriate nodes.

  In this case, we've already filtered the nodes, by fetching only the ones with the ``Person`` label.

Putting it all together, the class ``SIROutputSpec`` is as follows:


.. code-block:: scala

    package com.bharatsim.examples.epidemiology.sir

    import com.bharatsim.engine.Context
    import com.bharatsim.engine.listeners.CSVSpecs
    import com.bharatsim.examples.epidemiology.sir.InfectionStatus.{Infected, Removed, Susceptible}

    import scala.collection.mutable
    
    class SIROutputSpec(context: Context) extends CSVSpecs {
      override def getHeaders: List[String] =
        List(
          "Step",
          "Susceptible",
          "Infected",
          "Removed"
        )
      
      override def getRows(): List[List[Any]] = {

        val graphProvider = context.graphProvider
        val label = "Person"
        val countMap = mutable.HashMap.empty[String, Int]
        val nodes = graphProvider.fetchNodes(label)
        nodes.foreach(node => {
          val infectedState = node.getParams.apply("infectionState").toString
          val existingCount = countMap.getOrElse(infectedState, 0)
          countMap.put(infectedState, existingCount + 1)
        })

        val row = List(
          context.getCurrentStep,
          countMap.getOrElse(Susceptible.toString, 0),
          countMap.getOrElse(Infected.toString, 0),
          countMap.getOrElse(Removed.toString, 0)
        )
        return List(row)
        
      }
    }

.. hint::
  We import ``scala.collection.mutable`` in order to use the ``HashMap`` datatype.

This implementation can lead to considerable performance improvements, especially if you were originally looping over the graph a large number of times.
