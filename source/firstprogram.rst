Writing your First Program
==========================

This section is a detailed guide on how to build a `SIR Model <link for SIR URL>`_ in BharatSim from scratch. By the end of this section, you should be able to write and execute a SIR model by yourself.

Any model built using this framework contains different classes which are essentially extensions of different `Nodes <#>`_. To properly build a SIR model from scratch, you will first need to define these classes and the properties associated with them. These classes are grouped under different components of the model. Here, the SIR model has two different components:-
1. `Agent <#>`_
2. `Network <#>`_

These components are related to each other via simple relations which are described in detail later.

Setting up the components of the Network
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Network class gives a sense of geography to the model. The different components of the Network are simple locations like houses, offices, schools etc. The agents move about between these locations as shown in the figure below.

The first step to building your model is to create locations which are part of the Network, by creating a new scala `class <https://docs.scala-lang.org/tour/classes.htm>`_. You can create a new class for adding houses to the network, in the following way:-

1. Since ``House`` is a component of the Network, you have to import the Network class.

.. code-block:: scala

  import com.bharatsim.engine.models.Network

2. The ``House`` class is a `case class <https://docs.scala-lang.org/tour/case-classes.html>`_ and it extends the framework defined Network class.

.. code-block:: scala

  case class House extends Network

3. However, each house in the Network requires a unique house id which is given as an argument to the ``House`` class. This house id is an ‘attribute’ of the corresponding house.

.. code-block:: scala

  case class House(id: Long) extends Network

.. note:: Long is just a datatype of Scala.

4. You can define the relationship of the case-class with the agent by using the framework defined ``addRelation`` function within the class definition. A house houses an agent, so the relation is simply given by the string, “HOUSES”. These relations are defined in the Main class and are user-definable.

.. code-block:: scala

  addRelation[Person]("HOUSES")
.. note:: A House “HOUSES” an Agent and an Agent “STAYS_AT” a House so these two relations are inherently reflections of each other. The first relation is specified in the House class, while the second one is specified in the Person class(put link). The same logic can be extended to any pair of Agents and corresponding Network case classes. These relations are defined in the ``Main`` class which is explained later.


5. Similarly, an Office is also a possible component of the Network which has a different relation with the agent. Just like the ``House`` class, an ``Office`` class is defined by a unique office id. Since an office employs an agent, the relation here is simply given by “EMPLOYER_OF”.

.. code-block:: scala

  addRelation[Person]("EMPLOYER_OF")


6. Another characteristic of the case classes extended from the network is the ``getContactProbability``. This value is defined in the Network class, and hence is overridden to define the value one needs, as shown below, within the case-class definition.

.. code-block:: scala

  override def getContactProbability(): Double = 1.0

The importance of this function will become evident after the Disease Dynamics section.

7. The entire case class should look like this :-

.. code-block:: scala

  package com.bharatsim.examples.epidemiology.sir


  case class House(id: Long) extends Network {

   addRelation[Person]("HOUSES")



   override def getContactProbability(): Double = 1.0

 }


Setting Up the Agents
^^^^^^^^^^^^^^^^^^^^^^^^^^^
Both Agents and the Components of the Network are extensions of the `Node<node link>`    class. However, agents differ from the components of the Network in the logical sense that the Network components are static geographical locations like houses, offices etc. between which the agents move about. So, the agents are in a sense ‘dynamic’.


In the context of the framework, agents are the extension of the ``Agent`` class which in turn, is an extension of the ``Node`` class. The agents follow certain user-defined conditions called ‘Behaviours’. These behaviours are functions that can be defined in the Agent class. These behaviours are especially important when modelling disease dynamics which is described below:


1. Create a case class by the name “Person”. Since it is an extension of the Agent class which is an extension of the Node class, it is important to import these as shown below.

.. code-block:: scala

  import com.bharatsim.engine.models.{Agent, Node}

.. note:: It can be named as you please. For the sake of clarity, it has been named as **Person** here

2. Similar to the ``House`` case class described above, the ``Person`` case class is defined by a set of attributes. These attributes are generally the characteristics of a generic person like a person id, age etc. To define the Person case class, one must also call its attributes, which in this case are the id and age.

.. code-block:: scala

  case class Person(id: Long, age: Int) extends Agent {}

3. In order to add the relationship between the Person and the components of the Network, write the following code within the case class Person.

.. code-block:: scala

  addRelation[House]("STAYS_AT")
  addRelation[Office]("WORKS_AT")
  addRelation[School]("STUDIES_AT")

4. Given below is an example which will help you to understand the importance of attributes as well as behaviours. Consider the year ‘1984’. During this time, Big Brother doesn’t allow people below the age of 25 to watch ‘Harry Potter’ movies. To model this scenario, you can add a parameter ‘canIWatchHarryPotter’ when defining the ``Person`` case class and let it’s default value be “No”.

.. code-block:: scala

  import com.bharatsim.engine.Context

  case class Person(id:Long, age:Int, canIWatchHarryPotter = "No": String) extends Agent{}

.. note:: String is a data-type which takes strings as the arguments.


Assume that the name of this behaviour is ``watchMovie``. So, the task of the behaviour is to change the value of the parameter ``canIWatchHarryPotter`` from ‘No’ to ‘Yes’ for people above the age of 25.

.. note:: The behaviour takes ``Context`` as an argument so it has to be imported.


This can be done using the framework defined ``updateParam`` function which updates the specified parameters. The function takes two arguments, the parameter which is to be updated and the updated value.

.. code-block:: scala

  val watchMovie : Context => Unit = (context:Context) => {
      if (age >= 25) {
          updateParam("canIWatchHarryPotter", "Yes")
        }


It is important to use ``addBehaviour`` within the same case class.

.. code-block:: scala

  addBehaviour(watchMovie)

Saving your output
^^^^^^^^^^^^^^^^^^

Suppose you wanted your output to give you the numbers of susceptible, infected and recovered people at every time step. You can then write the following:

.. code-block:: scala

  import com.bharatsim.engine.Context
  import com.bharatsim.engine.graph.patternMatcher.MatchCondition._
  import com.bharatsim.engine.listeners.CSVSpecs
  import com.bharatsim.examples.epidemiology.SIR.InfectionStatus.{Infected, Removed, Susceptible}

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
      val row = List(
        context.getCurrentStep,
        graphProvider.fetchCount(label, "infectionState" equ Susceptible),
        graphProvider.fetchCount(label, "infectionState" equ Infected),
        graphProvider.fetchCount(label, "infectionState" equ Removed)
      )
      List(row)
    }
  }

* The first column (Step) stores the current time step, obtained using the ``context.getCurrentStep`` function
* The next 3 columns store the number of Susceptible, Infected and Removed people respectively, by fetching the total number of ``Person`` nodes on the graph with the appropriate appropriate `infection status <#>`_.

Now we simply have to register it in the simulation. Note that the following code snippet should be located inside ``simulation.defineSimulation`` in the main function:

.. code-block:: scala

  SimulationListenerRegistry.register(
    new CsvOutputGenerator("src/main/resources/output.csv", new SIROutputSpec(context))
      )


Computing the number of people in a location
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In our example of the SIR model, we decided if a person would be infected or not by:

* Retrieving the type of location that the person in question was supposed to be in from their schedule
* Computing the number of people who could potentially infect them

We use the following function to accomplish the second part of the algorithm:

.. code-block:: scala

    def computeInfectedFraction(node: Node, placeType: String, context: Context): Double = {

    val totalNeighbourCount = node.getConnectionCount(node.getRelation[Person]().get)
    if (totalNeighbourCount == 0)
      return 0d

    val infectedNeighbourCount = node.getConnectionCount(node.getRelation[Person]().get,
      "infectionState" equ Infected).toDouble

    infectedNeighbourCount / totalNeighbourCount.toDouble
  }


Let's take a closer look at the first line, how we calculate ``totalNeighbourCount``.

.. code-block:: scala

  val totalNeighbourCount = node.getConnectionCount(node.getRelation[Person]().get)

Assume that the node was an ``Office``. In that case,

.. code-block:: scala

  node.getRelation[Person]().get

returns the ``EMPLOYER_OF`` string. Therefore, ``totalNeighbourCount`` counts the total number of nodes liked to this particular ``Office`` node with the ``EMPLOYER_OF`` relation.

The problem arises with different methods of scheduling. Someone who's infected may follow a different schedule, and stay at home. However, ``totalNeighbourCount`` *doesn't care* about the location a person has at a particular tick: all it does is count the number of people with the appropriate relations. As a consequence of this, the results would not be what the modeller intended.

.. note:: The same thing happens while calculating ``infectedNeighbourCount``. This has effects not just on the workplace, but on homes, hospitals, and other locations in your model too.


There are two currently proposed methods to deal with the problem:

Using an attribute of the ``Person`` class
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We can solve the problem by adding an attribute called ``currentLocation`` to the ``Person`` class.

.. code-block:: scala

  case class Person(id: Long, age: Int, infectionState: InfectionStatus, infectionTick: Int,
                  RecoveryTick: Double, currentLocation: String = "HOUSE") extends StatefulAgent {}


.. tip:: We've set ``"House"`` as the `default value <https://docs.scala-lang.org/tour/default-parameter-values.html>`_ of the attribute, and so it's no longer necessary to initialize it when creating an instance of the ``Person`` class in the user-defined ``csvDataExtractor`` function.

After doing so, we need to add a behaviour which changes the ``currentLocation`` at every tick. First, we define what we want the behaviour to do with the following block of code in the ``Person`` class:

.. code-block:: scala

  private val checkCurrentLocation: Context => Unit = (context: Context) => {
    val schedule = context.fetchScheduleFor(this).get
    val locationNextTick: String = schedule.getForStep(context.getCurrentStep + 1)
    if (currentLocation != locationNextTick) {
      this.updateParam("currentLocation", locationNextTick)
    }
  }

Next, we need to register the behaviour so that it's executed every tick:

.. code-block:: scala

  addBehaviour(checkCurrentLocation)

.. hint:: ``updateParam`` only updates the value of the attribute at the **end** of the tick. Thus, for all practical purposes, it's useful to view the function as one that changes the value of the attribute on the *subsequent tick*. As such, we store the place the person is expected to be on the next tick, and hence use ``context.getCurrentStep+1`` as an argument to ``schedule.getForStep``.

Now, we need use this attribute when we compute ``totalNeighbourCount`` and ``infectedNeighbourCount``. The basic structure of the function remains the same:

.. code-block:: scala

  def computeInfectedFraction(node: Node, placeType: String, context: Context): Double = {}

``node.getConnectionCount`` has another (optional) argument besides the relation, which is ``matchPattern``. Using it, we can get counts of the people with a specific relation who also satisy some other condition based on their attributes: in this case, we'll look for the people who have the ``currentLocation`` attribute equal to the ``placeType`` of the node.

.. code-block:: scala

    val totalNeighbourCount = node.getConnectionCount(node.getRelation[Person]().get,
      "currentLocation" equ placeType)

As we did before, we return ``0`` if there are no neighbours (as otherwise we'd be dividing by 0):

.. code-block:: scala

    if (totalNeighbourCount == 0) return 0d

Next, we need the total count of infected people. We can do that by checking that the person's ``infectionState`` is ``Infected``, in addition to what we did before:

.. code-block:: scala

    val infectedNeighbourCount = node.getConnectionCount(node.getRelation[Person]().get,
      ("infectionState" equ Infected) and ("currentLocation" equ placeType))

.. note:: You need to use ``equ``, ``and`` and other pattern-matching relations instead of the scala versions ``==``, ``&&``, etc. They're defined in ``com.bharatsim.engine.graph.patternMatcher.MatchCondition``. Remember to import them!

Finally, we return the infected fraction,

.. code-block:: scala

    infectedNeighbourCount.toDouble / totalNeighbourCount.toDouble

Putting it all together, our function is

.. code-block:: scala

  def computeInfectedFraction(node: Node, placeType: String, context: Context): Double = {
    val totalNeighbourCount = node.getConnectionCount(node.getRelation[Person]().get,
      "currentLocation" equ placeType)

    if (totalNeighbourCount == 0) return 0d

    val infectedNeighbourCount = node.getConnectionCount(node.getRelation[Person]().get,
      ("infectionState" equ Infected) and ("currentLocation" equ placeType))

    infectedNeighbourCount.toDouble / totalNeighbourCount.toDouble
  }

Checking the locations without a ``currentLocation`` attribute
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``updateParam`` updates a node on the graph, and is called once per person per tick. That can potentially slow the program down, and another possibility is to avoid using it entirely. We'll still do the same thing - get the schedule for the agent, check if they're actually at the place you're looking at, and then get the total and infected counts.

.. note:: We can't use ``getConnectionCount`` anymore, cause there's no attribute to match to. As such, the calculation of the total and infecteded neighbour counts is done by iterating over every person with the relation, and adding them in.

Let's break it up: the structure of the function remains identical

.. code-block:: scala

  def computeInfectedFraction(node: Node, placeType: String, context: Context): Double = {}

First, we assign two variables to count the number of total and infected neighbors. These will be incremented later.

.. code-block:: scala

    var totalNeighbourCount: Int = 0
    var infectedNeighbourCount: Int = 0

We now find everyone with the appropriate relation:

.. code-block:: scala

    val peopleWithRelation: Iterator[GraphNode] = node.getConnections(node.getRelation[Person]().get)

.. note:: ``peopleWithRelation`` is a convenient data structure called an `iterator <https://docs.scala-lang.org/overviews/collections/iterators.html>`_. It's very useful if you want to loop through a container, as we do here.

Now, we want to check the ``currentLocation`` and ``infectionState`` for every one of these people. We iterate over the iterator using the ``foreach`` method:

.. code-block:: scala

    peopleWithRelation.foreach (relatedPerson => {})

.. hint:: The function inside the curly brackets is executed for every ``GraphNode`` in the iterator. We can easily reference that particular node with ``relatedPerson``.

The first thing we want to do for each ``relatedPerson`` is to get the location they're expected to be at this tick

.. code-block:: scala

      val schedule = context.fetchScheduleFor(relatedPerson.as[Person]).get
      val locationThisTick: String = schedule.getForStep(context.getCurrentStep)

First we check if the ``relatedPerson`` is actually in the place we're looking at, and if so we increment ``totalNeighbourCount``. If they're also infected, we increment ``infectedNeighbourCount``.

.. code-block:: scala

      if (locationThisTick == placeType) {
        totalNeighbourCount += 1
        if (relatedPerson.as[Person].isInfected) {
          infectedNeighbourCount += 1
        }
      }

That's all we need to do for each ``relatedPerson``: outside the loop, we now have to check for the edge case where ``totalNeighbourCount = 0``, and return the infected fraction

.. code-block:: scala

    if (totalNeighbourCount == 0) return 0d

    infectedNeighbourCount.toDouble / totalNeighbourCount.toDouble

All in all, the function we use is

.. code-block:: scala

  def computeInfectedFraction(node: Node, placeType: String, context: Context): Double = {
    var totalNeighbourCount: Int = 0
    var infectedNeighbourCount: Int = 0
    val peopleWithRelation: Iterator[GraphNode] = node.getConnections(node.getRelation[Person]().get)
    peopleWithRelation.foreach (relatedPerson => {
      val schedule = context.fetchScheduleFor(relatedPerson.as[Person]).get
      val locationThisTick: String = schedule.getForStep(context.getCurrentStep)
      if (locationThisTick == placeType) {
        totalNeighbourCount += 1
        if (relatedPerson.as[Person].isInfected) {
          infectedNeighbourCount += 1
        }
      }
    })
    if (totalNeighbourCount == 0) return 0d

    infectedNeighbourCount.toDouble / totalNeighbourCount.toDouble
  }

At the moment, we cannot say which method is preferable as there hasn't been much testing to see how they scale up with the size of the population.
