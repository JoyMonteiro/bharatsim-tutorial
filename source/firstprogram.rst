Writing your First Program
==========================

The SIR Model in BharatSim
--------------------------

BharatSim is an open-source large-scale agent-based simulation framework that allows you to build and simulate sophisticated agent-based models. Agents form the main component of such a simulation. They are entities that are defined with a set of parameters and obey certain specified conditions as the simulation runs.

In the purview of the SIR model, the agents are people, which go about their daily lives and follow regular schedules. Here, the parameters defining an agent are a unique agent id, their age, their infection status, the time for which they remain infected and so on. The framework allows you to introduce new parameters as and when required.

The rules governing the behaviour of all agents include the particular schedules which they follow and the functions which allow the agents to transit from one compartment to another as described before.

Model Components
~~~~~~~~~~~~~~~~

The model consists of different objects and classes and can be separated into individual ``.scala`` files.

#. **Nodes of the Network:** These include ``House``, ``School``, ``Office``, etc, which essentially correspond to the different geographical locations in which the agents spend their time according to some schedule. These classes have some attributes associated with them. For eg. ``getContactProbability`` is one such function that gives the transmission probability of the disease in that particular location. In order to add new Nodes to the Network (for eg. Hospital), one has to create a new scala class inside the same package corresponding to the new Node.

    .. image:: _static/images/House.png

#. **Disease:** ``Disease`` is a class where all epidemiological parameters are conveniently defined. For example, the transmission rate :math:`\beta` is defined in this class. This class allows for a convenient way to store and modify variables as required.

#. **Infection Status:** The ``InfectionStatus`` is the class where all the compartments of the model are defined. The ``InfectionStatus`` object in the class, is a serialisable object which includes all the compartments. Inside this class, there is an encoder function which converts the compartment into a corresponding string and a decoder function, which decodes a string into a particular compartment. To add a new compartment like ‘Exposed’ refer to the figure below:-

    .. code-block:: scala

      package com.bharatsim.examples.epidemiology.sir

      import com.bharatsim.engine.basicConversions.StringValue
      import com.bharatsim.engine.basicConversions.decoders.BasicDecoder
      import com.bharatsim.engine.basicConversions.encoders.BasicEncoder

      object InfectionStatus extends Enumeration {
        type InfectionStatus = Value
        val Susceptible,Exposed, Infected, Removed = Value

        implicit val infectionStatusDecoder: BasicDecoder[InfectionStatus] = {
            case StringValue(v) => withName(v)
            case _ => throw new RuntimeException("Infection status was not stored as a string")
        }

        implicit val infectionStatusEncoder: BasicEncoder[InfectionStatus] = {
            case Susceptible => StringValue("Susceptible")
            case Exposed => StringValue("Exposed")
            case Infected => StringValue("Infected")
            case Removed => StringValue("Removed")
        }
      }

#. **Output Specification:** ``CSVSpecs`` is a framework defined trait which helps in generating a .csv file for the simulation output. The ``SEIROutputSpec`` is a class that extends this trait and defines how the output ``.csv`` file should look. Inside this class, the ``getHeaders`` basically specifies the list of column headers in the ``.csv`` file, while the ``getRows`` fetches the values of these quantities from context. For eg. one can use the ``fetchcount`` function in the framework defined trait ``graphProvider`` to display the number of people in each compartment at every tick.

Person class
~~~~~~~~~~~~

The ``Person`` class is an extension of the framework defined ``Agent`` class. It defines all the attributes of an agent, and also the behaviours which the agent follows throughout the simulation.

An attribute is essentially a characteristic of an agent like age, agent id and so on. It can also include certain quantities associated with an agent like its current epidemiological state, or the time for which it is infected. The attributes are defined in the Person class as follows:-

.. code-block:: scala

   case class Person(id: Long, age: Int, infectionState: InfectionStatus, infectionDur: Int, betaMultiplier:Double) extends Agent

Some attributes have constant values throughout the simulation like age, agent id, while there are some attributes which are ‘dynamic’ and their values change as the simulation runs.

In this framework, A **Behaviour** is an user-defined function whose arguments are the dynamic attributes. It acts on every agent at each tick.  An example for a behaviour which counts the number of days for which an agent is infected, is as follows:

.. code-block:: scala

   private val incrementInfectionDay: Context => Unit = (context: Context) => {
      if (isInfected && context.getCurrentStep % Disease.numberOfTicksInADay == 0) {
         updateParam("infectionDur", infectionDur + 1)
      }
   }

Moreover, a Behaviour could be a rule which governs the transfer of agents from one compartment to another. For example, if person X is in the same location as an infected person Y, person X will also get infected, with some probability.
Similarly, if a person is infected, they can move to the ``Removed`` compartment, with a certain rate, :math:`\lambda_I`. The code snippet below is a Behaviour ``checkForRecovery`` that performs this transition.

.. code-block:: scala

   private val checkForRecovery: Context => Unit = (context: Context) => {
       if (isInfected) {
           val RecoveryProb = Disease.lambdaI*Disease.dt
           val InfectionState = if (biasedCoinToss(RecoveryProb)) "Removed" else "Infected"
           if (InfectionState == "Removed") {
                updateParam("infectionState", Removed)
           }
       }
   }


It is important that one defines the Behaviours in the order that they are to be compiled. This is based on the epidemiological SIR model, where an agent transitions from one compartment to another in a chronological manner, from S to I to R, as shown below.

.. image:: _static/images/SIR-compartment.png

After the behaviours are defined, they need to be added to the simulation in the order in which they are executed. This is done using the framework defined ``addBehaviour`` function.

.. code-block:: scala

   addBehaviour(incrementInfectionDay)
   addBehaviour(checkForInfection)
   addBehaviour(checkForRecovery)

Each agent goes through the above-mentioned behaviours chronologically during each tick. For example, if there are 100 agents in the simulation, all 100 of them go through the behaviours as listed chronologically above, and this process repeats at each tick.


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
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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

    peopleWithRelation.foreach (relatedPerson => {}

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
