FSM in SIR
========================

Futher information can be found :ref:`Finite State Machine`


In this case of Finite State Machine, the abstract machine will be the agents in the population. Each of these agents can be either Susceptible, Infected or Recovered, and there is a well defined procedure on moving from one state to another. When the FSM is implemented, the ``Transition`` condition will be dictated by the probability of contracting the infection or recovering from the infection. In more complicated systems, there can be 2 or more transitions are possible, for example from Exposed to Asymptomatic or Symptomatic states. These transitions will be dictated by there respective probabilities and again only one of these transitions can take place. 

Earlier we had introduced Disease Dynamics in the form of ``behaviours``, and these dictated whether an agent would be ``isInfected`` or ``isRecovered``. As discussed earlier, in the FSM ``Transition`` will dictate whether an agent is in ``InfectedState`` or ``RecoveredState``.  

To introduce a Finite State Machine, we need to make the following changes:

1. Define disease states, and define transitions between them
2. Modify our agents to now be extensions of the ``StatefulAgent`` class, instead of the ``Agent`` class like before.

.. tabs::

  .. code-tab:: scala Disease.scala

    package sir

    object Disease {
      final val beta: Double = 0.3 
      final val lastDay: Int = 12
      final val lamda: Double = 0.14
      final val dt: Double = 0.5
    }
  
  .. code-tab:: scala House.scala

    package sir
    import com.bharatsim.engine.models.Network

    case class House(id: Long) extends Network {
      addRelation[Person]("HOUSES")

      override def getContactProbability(): Double = 1
    }

  .. code-tab:: scala InfectedState.scala 

    package sir
    import com.bharatsim.engine.basicConversions.StringValue
    import com.bharatsim.engine.basicConversions.decoders.BasicDecoder
    import com.bharatsim.engine.basicConversions.encoders.BasicEncoder

    object InfectionStatus extends Enumeration {
      type InfectionStatus = Value
      val Susceptible, Infected, Removed = Value

      implicit val infectionStatusDecoder: BasicDecoder[InfectionStatus] = {
        case StringValue(v) => withName(v)
        case _ => throw new RuntimeException("Infection status was not stored as a string")
      }

      implicit val infectionStatusEncoder: BasicEncoder[InfectionStatus] = {
        case Susceptible => StringValue("Susceptible")
        case Infected => StringValue("Infected")
        case Removed => StringValue("Removed")
      }
    }

    .. code-tab:: scala Main.scala 

      package sir
      import com.bharatsim.engine.Context
      import com.bharatsim.engine.ContextBuilder._
      import com.bharatsim.engine.execution.Simulation
      import com.bharatsim.engine.graph.ingestion.{GraphData, Relation}
      import com.typesafe.scalalogging.LazyLogging
      import com.bharatsim.engine.utils.Probability.biasedCoinToss
      import com.bharatsim.engine.basicConversions.encoders.DefaultEncoders._
      import sir.InfectionStatus._
      import com.bharatsim.engine.{Context, Day, Hour, ScheduleUnit}
      import com.bharatsim.engine.actions.StopSimulation
      import com.bharatsim.engine.listeners.{CsvOutputGenerator, SimulationListenerRegistry}
      import com.bharatsim.engine.models.Agent
      import java.util.Date
      import com.bharatsim.engine.basicConversions.decoders.DefaultDecoders._
      import com.bharatsim.engine.graph.patternMatcher.MatchCondition._
      import com.bharatsim.engine.dsl.SyntaxHelpers._

      object Main extends LazyLogging {
        private val initialInfectedFraction = 0.01

        final val inverse_dt = 2
        final val dt: Double = 1f / inverse_dt 

        var myTick: ScheduleUnit = new ScheduleUnit(1)
        var myDay: ScheduleUnit = new ScheduleUnit(myTick * inverse_dt)

        def main(args: Array[String]): Unit = {

          var beforeCount = 0
          val simulation = Simulation()

          simulation.ingestData(implicit context => {
            ingestCSVData("citizen10k.csv", csvDataExtractor)
            logger.debug("Ingestion done")
          })

          simulation.defineSimulation(implicit context => {
            create12HourSchedules()

            registerAction(
              StopSimulation,
              (c: Context) => {
                getInfectedCount(c) == 0
              }
            )

            beforeCount = getInfectedCount(context)

            registerAgent[Person]

            val currentTime = new Date().getTime

            SimulationListenerRegistry.register(
              new CsvOutputGenerator("src/main/" + currentTime + ".csv", new SIROutputSpec(context))
            )
          })

          simulation.onCompleteSimulation { implicit context =>
            printStats(beforeCount)
            teardown()
          }

          val startTime = System.currentTimeMillis()
          simulation.run()
          val endTime = System.currentTimeMillis()
          logger.info("Total time: {} s", (endTime - startTime) / 1000)
        }

        private def create12HourSchedules()(implicit context: Context): Unit = {
          val EmployeeSchedule = (myDay, myTick)
            .add[House](0, 0)
            .add[Office](1, 1)

          val StudentSchedule = (myDay, myTick)
            .add[House](0, 0)
            .add[School](1, 1)

          val quarantinedSchedule = (Day, Hour)
            .add[House](0, 23)

          registerSchedules(
            (quarantinedSchedule, (agent: Agent, _: Context) => agent.asInstanceOf[Person].isInfected, 1),
            (EmployeeSchedule, (agent: Agent, _: Context) => agent.asInstanceOf[Person].age >= 18, 2),
            (StudentSchedule, (agent: Agent, _: Context) => agent.asInstanceOf[Person].age < 18, 3)
          )
        }

        private def csvDataExtractor(map: Map[String, String])(implicit context: Context): GraphData = {

          val citizenId = map("Agent_ID").toLong
          val age = map("Age").toInt
          val initialInfectionState = if (biasedCoinToss(initialInfectedFraction)) "Infected" else "Susceptible"

          val homeId = map("HHID").toLong
          val schoolId = map("school_id").toLong
          val officeId = map("WorkPlaceID").toLong

          val citizen: Person = Person(
            citizenId,
            age,
            InfectionStatus.withName(initialInfectionState),
            0
          )

          val home = House(homeId)
          val staysAt = Relation[Person, House](citizenId, "STAYS_AT", homeId)
          val memberOf = Relation[House, Person](homeId, "HOUSES", citizenId)

          val graphData = GraphData()
          graphData.addNode(citizenId, citizen)
          graphData.addNode(homeId, home)
          graphData.addRelations(staysAt, memberOf)

          if (age >= 18) {
            val office = Office(officeId)
            val worksAt = Relation[Person, Office](citizenId, "WORKS_AT", officeId)
            val employerOf = Relation[Office, Person](officeId, "EMPLOYER_OF", citizenId)

            graphData.addNode(officeId, office)
            graphData.addRelations(worksAt, employerOf)
          } else {
            val school = School(schoolId)
            val studiesAt = Relation[Person, School](citizenId, "STUDIES_AT", schoolId)
            val studentOf = Relation[School, Person](schoolId, "STUDENT_OF", citizenId)

            graphData.addNode(schoolId, school)
            graphData.addRelations(studiesAt, studentOf)
          }

          graphData
        }

        private def printStats(beforeCount: Int)(implicit context: Context): Unit = {
          val afterCountSusceptible = getSusceptibleCount(context)
          val afterCountInfected = getInfectedCount(context)
          val afterCountRecovered = getRemovedCount(context)

          logger.info("Infected before: {}", beforeCount)
          logger.info("Infected after: {}", afterCountInfected)
          logger.info("Recovered: {}", afterCountRecovered)
          logger.info("Susceptible: {}", afterCountSusceptible)
        }

        private def getSusceptibleCount(context: Context) = {
          context.graphProvider.fetchCount("Person", "infectionState" equ Susceptible)
        }

        private def getInfectedCount(context: Context): Int = {
          context.graphProvider.fetchCount("Person", ("infectionState" equ Infected))
        }

        private def getRemovedCount(context: Context) = {
          context.graphProvider.fetchCount("Person", "infectionState" equ Removed)
        }
      }

  .. code-tab:: scala Office.scala 

    package sir

    import com.bharatsim.engine.models.Network

    case class Office(id: Long) extends Network {
      addRelation[Person]("EMPLOYER_OF")

      override def getContactProbability(): Double = 1
    }

  .. code-tab:: scala Person.scala 

    package sir

    import com.bharatsim.engine.Context
    import com.bharatsim.engine.basicConversions.decoders.DefaultDecoders._
    import com.bharatsim.engine.basicConversions.encoders.DefaultEncoders._
    import com.bharatsim.engine.graph.GraphNode
    import com.bharatsim.engine.models.{Agent, Node}
    import com.bharatsim.engine.utils.Probability.toss
    import com.bharatsim.engine.utils.Probability.{biasedCoinToss, toss}
    import sir.InfectionStatus._

    case class Person(id: Long, age: Int, infectionState: InfectionStatus, infectionDay: Int) extends Agent {
      final val numberOfTicksInADay: Int = 2
      private val incrementInfectionDuration: Context => Unit = (context: Context) => {
        if (isInfected && context.getCurrentStep % numberOfTicksInADay == 0) { 
          updateParam("infectionDay", infectionDay + 1)
        }
      }
      private val checkForInfection: Context => Unit = (context: Context) => {
        if (isSusceptible) {
          val infectionProb = Disease.beta*Disease.dt

          val schedule = context.fetchScheduleFor(this).get

          val currentStep = context.getCurrentStep
          val placeType: String = schedule.getForStep(currentStep)

          val places = getConnections(getRelation(placeType).get).toList
          if (places.nonEmpty) {
            val place = places.head
            val decodedPlace = decodeNode(placeType, place) 
            val infectedNeighbourCount = decodedPlace
              .getConnections(decodedPlace.getRelation[Person]().get)
              .count(x => x.as[Person].isInfected)

            val N = decodedPlace
              .getConnections(decodedPlace.getRelation[Person]().get)
              .count(x => x.as[Person].age > 0)


            val shouldInfect = biasedCoinToss(infectionProb*infectedNeighbourCount/N) 
            if (shouldInfect) {
              updateParam("infectionState", Infected) 
            }
          }
        }
      }

      private val checkForRecovery: Context => Unit = (context: Context) => {
        if (isInfected &&  biasedCoinToss(Disease.lamda * Disease.dt) 
        ) 
          updateParam("infectionState", Removed)
      }

      def isSusceptible: Boolean = infectionState == Susceptible

      def isInfected: Boolean = infectionState == Infected

      def isRecovered: Boolean = infectionState == Removed

      private def decodeNode(classType: String, node: GraphNode): Node = {
        classType match {
          case "House" => node.as[House]
          case "Office" => node.as[Office]
          case "School" => node.as[School]
        }
      }
      addBehaviour(incrementInfectionDuration)
      addBehaviour(checkForInfection)
      addBehaviour(checkForRecovery)

      addRelation[House]("STAYS_AT")
      addRelation[Office]("WORKS_AT")
      addRelation[School]("STUDIES_AT")
    }

  .. code-tab:: scala School.scala 

    package sir

    import com.bharatsim.engine.models.Network

    case class School(id: Long) extends Network {
      addRelation[Person]("TEACHES")

      override def getContactProbability(): Double = 1
    }

  .. code-tab:: scala SIROutputSpec.scala 

    package sir

    import com.bharatsim.engine.Context
    import com.bharatsim.engine.graph.patternMatcher.MatchCondition._
    import com.bharatsim.engine.listeners.CSVSpecs
    import sir.InfectionStatus.{Susceptible, Infected, Removed}

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

Right click on the folder, and hover over the Refactor option, and then click on copy classes. Rename these sets of classes as FSMsir. 

.. error:: If the name appears as FSMsir.sir then simply rename the file through refactor as FSMsir

The ``Person`` extends the ``Agent`` class but now that we are re-defining how a person is thought off, we need to extend a pre-defined class called ``StatefulAgent``. There is no need to import another package, it was added in the code above. Create a new package in the current and name it ``DiseaseStates``, and create case classes called ``SusceptibleState``, ``InfectedState``, and ``RecoveredState``.

In the DiseaseStates classes, import the following packages,

.. code-block:: scala 

  import com.bharatsim.engine.Context
  import com.bharatsim.engine.basicConversions.decoders.DefaultDecoders._
  import com.bharatsim.engine.basicConversions.encoders.DefaultEncoders._
  import com.bharatsim.engine.fsm.State
  import com.bharatsim.engine.graph.patternMatcher.MatchCondition._
  import com.bharatsim.engine.models.{Network, StatefulAgent}
  import com.bharatsim.engine.utils.Probability.biasedCoinToss
  import FSMsir.InfectionStatus._
  import FSMsir.{Disease, Person}

For each of the classes also extend the ``State`` Class. What we aim to achieve in these classes is have a defination of what it means to be of that ``State`` and a ``Transition`` out of that ``State``. 


.. note::  It is important to note that we define the probability to leave that ``State`` and not enter the ``State``. That will be defined in the previous ``State`` to the current. 

By doing so, we can remove a major portion of the code written in the ``Person`` class, since that was the governing the disease dynamics. It is more convenient to start by defining the ``Transition``. The syntax is as follows, 

.. code-block:: scala 

  addTransition(
  when = ,
  to = context =>
  )

``addTransition`` requires two parameters, when to execute the ``Transition`` and where does the agent go. The former is ``Boolean`` while the latter is a ``State``. To tackle the ``when`` parameter we can define a function called ``shouldBeInfected``, which does the same thing as ``checkforInfection`` in the ``Person`` class. As to where the agent will go after the ``Transition``, that is the ``InfectedState`` we have just written. The ``Transition`` will be the following, 

.. code-block:: scala

  addTransition(
  when = shouldBeInfected,
  to = context => InfectedState()
  )

Now it comes to defining the ``shouldbeInfected`` function, and this can be done by updating the ``checkforInfection`` function, however I use a different approach. This has incorporated `PerTickCache <https://bharatsim.readthedocs.io/en/latest/Optimization.html>`_  method to reduce computational time. I will briefly explain the advantages of this method of computation. More often that not, there are multiple agents present at one location at any given tick and the current simulation calculates quantities like ``infectedCount``, ``infectedNeighbourCount`` for each and every of these agents. At every Tick, the system has become static and the information of the location does not change, and it is becomes tedious to calculate all these quantities over and over again. ``PerTickCache`` calculates the information about the location once, and stores the information. If another agent belongs to the locations whose information was previously computed, then the stored information is utilized and if there is no information present, then it calculates and stores it for any other agent who might be present here. After the Tick has been completed, then it deletes the information. If there are N locations, then there will be a maximum of N times these quantities will be calculated. 

.. code-block:: scala 

  def shouldBeInfected(context: Context, agent: StatefulAgent): Boolean = {
    if (agent.activeState == SusceptibleState()) {
      val infectionRate = Disease.beta
      val dt = Disease.dt

      val schedule = context.fetchScheduleFor(agent).get

      val currentStep = context.getCurrentStep
      val placeType: String = schedule.getForStep(currentStep)

      val places = agent.getConnections(agent.getRelation(placeType).get).toList
      if (places.nonEmpty) {
        val place = places.head
        val decodedPlace = agent.asInstanceOf[Person].decodeNode(placeType, place)

        val infectedFraction = fetchInfectedFraction(decodedPlace, placeType, context)
        return biasedCoinToss(infectionRate * infectedFraction * dt)
      }
    }
    false
  }

This function is every similar to ``checkforInfection`` except for the conversion from ``Agent`` to ``StatefulAgent``. Here the ``infectedFraction`` is not calculated, instead a value from another function is obtained. This is where ``PerTickCache`` is implemented. 

.. code-block:: scala 

  private def fetchInfectedFraction(decodedPlace: Network, place: String, context: Context): Double = {
    val cache = context.perTickCache

    val tuple = (place, decodedPlace.internalId)
    cache.getOrUpdate(tuple, () => fetchFromStore(decodedPlace)).asInstanceOf[Double]
  }

The above is a hashmap, which requires a ``tuple`` as a key which is unique for every location. The key which is a  ``tuple`` that stores the place and the internalId of the place. This is fed in ``getOrUpdate``, which looks into the stored memory to see if any information about the place can be found. If there exist some prior information, then it gets the information. If there is no prior information, then it calculates the values and updates it so the next time it will not have to calculate. The symbol ``()`` means that there is no information is present, and the computer is asked to use the function ``fetchFromStore`` to find the infected number. This is the same code as the one in ``Person`` class.

.. code-block:: scala 

  private def fetchFromStore(decodedPlace: Network): Double = {
    val infectedPattern =
      ("infectionState" equ Infected)
    val total = decodedPlace.getConnectionCount(decodedPlace.getRelation[Person]().get)

    total
  }

These are the things that need to added to ``SusceptibleState`` class. From the agent ``Transitions`` to ``InfectedState``. Again it is easier to add the ``Transition`` first. 

.. code-block:: scala 

  addTransition(
    when = checkForRecovered,
    to = context => RecoveredState()
  )

The function ``checkForRecovered`` is just a ``biasedCoinToss`` with the appropriate probabilities. 

.. code-block:: scala

  def checkForRecovered(context: Context, agent: StatefulAgent): Boolean = {
    return biasedCoinToss(Disease.lamda * Disease.dt)
  }


This is all for ``InfectedState``. Nothing needs to be added for ``RecoveredState`` since they cant participate in the dynamics or ``Transition`` out of the ``State``. However, if we were to model a system with waning immunity to a disease - for example, an "SIRS" model where recovered individuals transition back to the Susceptible state -- we will need to include the dynamics of this in the ``RecoveredState``. 









    


