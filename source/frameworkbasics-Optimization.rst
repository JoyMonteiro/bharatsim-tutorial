Optimisation
===============


PerTickCache
----------------
PerTickCache is a Map-based temporary data store, which is used as a cache. It is a time and resource consuming process to access 
the data store for each agent, most of which may even be redundant. We can use the PerTickCache for accessing such data. 
If the value is already computed, we can acces it from there or compute the result and then store it as cache. 
The PerTickCache is auto cleaned at the beginning of the tick. Example: In epidemiology, if we are trying to compute whom all 
might get infected based on their area(Home, office, etc.), the total number of infected in the area remains constant for a 
particular tick, and the same can be used for all susceptible agents.

The user has to determine a unique key such as placeType to avoid overriding the results. A new value will be computed only
if the value of that key is not present.
Consider a case where a particular placeType, say office_1, has 1000 agents in a particular tick. So if we have to find the infected
fraction for each agent, we have to query the data store for all the 1000 agents. But if we are using PerTickCache, we need to 
access the data store only once. This significantly speeds up our simulation.

Example for PerTickCache implementation:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Here we define the function which computes the value to be stored as cache.

.. code-block:: Scala

    private def computeTheValue(node: Node): Double = {
    val total = node.getConnectionCount(node.getRelation[Person]().get)
    if (total == 0)
      return 0.0
    val infected = node.getConnectionCount(node.getRelation[Person]().get, "infectionState" equ Infected)

    infected.toDouble/total.toDouble
    }

The below function computes the infected fraction at a place, i.e. the number of infected people divided by the total number of people
in that particular place. 

.. code-block:: Scala

    private  def fetchInfectedFraction(node: Node, placeType: String, context: Context): Double = {
    val cache = context.perTickCache
    val uniqueKey = (placeType, node.internalId)
    cache.getOrUpdate(uniqueKey, () => computeTheValue(node)).asInstanceOf[Double]
    }

The unique key used here is place type. Since in a particular tick, all agents in the same place will
have the same infected fraction, we do not have to compute this for every agent; instead, we use the value from the cache. This improves the
computation speed and minimises resource usage.


Outputting data to a csv
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

Using the ``getParams``` and ``apply`` methods of the ``GraphNode`` class together, we can obtain a parameter of the node:

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
