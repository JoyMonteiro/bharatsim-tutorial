
Expanding the Network
=====================

Ealier we had one location class which was the ``House``. In this section we increase the location classes to ``House``,  ``Office``, and ``School``. Every person has a unique house and either an office or a school and this categorized on the basis of age. 

Implementing multiple houses, offices, and schools
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As mention while creating the ``House.scala`` class, we mentioned that each of the locations will require a separate class. In addition to the new location classes, the person class needs to updated to establish the relationships. 

.. tabs::

  .. group-tab:: Office.scala 

    This scala class defines the relationship betweeen the agent of type ``Person`` and ``Office``.  Again since there are numerous offices, the datatype required is Long. 

  .. group-tab:: School.scala

    This scala class defines the relationship betweeen the agent of type ``Person`` and ``School``.  Again since there are numerous schools, the datatype required is Long. 

  .. group-tab:: Person.scala 

    This is the same as last class we defined but now we have to add relationships that corresponds to the relationships define in the Network classes earlier. 

.. tabs::
  
  .. code-tab:: scala Office.scala

    package sir

    import com.bharatsim.engine.models.Network

    case class Office(id: Long) extends Network {
      addRelation[Person]("EMPLOYER_OF")

      override def getContactProbability(): Double = 1
    }

  .. code-tab:: scala School.scala 

    package sir

    import com.bharatsim.engine.models.Network

    case class School(id: Long) extends Network {
      addRelation[Person]("TEACHES")

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
    import com.bharatsim.examples.epidemiology.sir.InfectionStatus._

    case class Person(id: Long, age: Int, infectionState: InfectionStatus, infectionDay: Int) extends Agent {
      final val numberOfTicksInADay: Int = 24
      private val incrementInfectionDuration: Context => Unit = (context: Context) => {
        if (isInfected && context.getCurrentStep % numberOfTicksInADay == 0) { 
          updateParam("infectionDay", infectionDay + 1)
        }
      }
      private val checkForInfection: Context => Unit = (context: Context) => {
        if (isSusceptible) { 
          val infectionRate = Disease.beta 

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

            val shouldInfect = toss(infectionRate, infectedNeighbourCount) 
            if (shouldInfect) {
              updateParam("infectionState", Infected) 
            }
          }
        }
      }

      private val checkForRecovery: Context => Unit = (context: Context) => {
        if (isInfected && infectionDay == Disease.lastDay 
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


The main file doesnt need major alterations, but the changes that have to be implemented are crucial conceptually and for the program to give the correct output. The majority of the changes are in two areas which are

* Categorization of people: We have different locations in the network but only one type of Person. We need to make a distinction and categorize the individuals to send them to different locations. In this section, the categorization is done on the basis of age; any over the age of 18 works in an office and anyone under the age of 18 goes to a school. After creating these different people, we need to define the relationship between the people and their respective nodes. All these changes are made in the csvDataExtractor. 

.. note:: The age of the citizens are provided in the input csv file. 

* createSchedules: Now that we have defined office-goers and school-goers, we need to decide their schedules and timings. 

The csvDataExtractor function is the same and changes are made after the nodes (house, citizen) and relationship (house and person) is defined. Regardless of the age of the individual, they still have a house that they are associated to and therefore no changes are required when defining the aforementioned nodes and relationships. The next part is adding new nodes and relationships for individuals and their additional network and this is rather straightforward. An if condition is used to categorize on the basis of age and in the conditional block the relationships and nodes are added, similar to the house and citizen case. 

.. code-block:: scala 

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


Implementing Schedules
^^^^^^^^^^^^^^^^^^^^^^

After this distinction has been made, the changes in schedules have to be made. Employee and student schedule are just when they leave for their the house and when they return. First we need to define an hour to be ``myTick`` and there are 24 hours in ``myDay``. Before ``create24HourSchedules`` can be made, ``myTick`` and ``myDay`` needs to be defined outside the main function. 

.. code-block:: scala 

    private val myTick: ScheduleUnit = new ScheduleUnit(1)
    private val myDay: ScheduleUnit = new ScheduleUnit(myTick * 24)

With these values defined, ``create24HourSchedules`` can be made. However, when there are more than one schedules running, there needs to be a priority list that needs to be made. In this case, Student and Employee schedules are independent of each other so a either schedules can be prioritized over the other. In later cases, quarantine will be introduced where individuals will stay at their house the whole time and this gets priority over office and school schedules. 

.. code-block:: scala 

    private def create24HourSchedules()(implicit context: Context): Unit = {
      val employeeSchedule = (myDay, myTick)
        .add[House](0, 8)
        .add[Office](9, 17)
        .add[House](18,23)

      val studentSchedule = (myDay, myTick)
        .add[House](0, 8)
        .add[Office](9, 16)
        .add[House](17, 23)

      registerSchedules(
        (employeeSchedule, (agent: Agent, _: Context) => agent.asInstanceOf[Person].age >= 18, 1),
        (studentSchedule, (agent: Agent, _: Context) => agent.asInstanceOf[Person].age < 18, 2)
      )
    }

.. note:: The timings of departure and return are to be made in the 24 hour format.  


Handling Current Locations
^^^^^^^^^^^^^^^^^^^^^^^^^^
Now, we also need to handle the location of the individual at every step to ensure that the individual is only in contact with the people in the same location. This is done by adding a new ``currentLocation`` parameter in the person class. We modify the constructor to include this parameter.

.. code-block:: scala 

    case class Person(id: Long, age: Int, infectionState: InfectionStatus, infectionDay: Int, currentLocation: String = "") extends Agent {

Then, we add a behaviour to update the current location of the person at every step.

.. code-block:: scala 

    private def getCurrentLocation(context: Context): Option[String] = {
      val schedule = context.fetchScheduleFor(this).get
      val currentStep = context.getCurrentStep
      val placeType: String = schedule.getForStep(currentStep)

      Some(placeType)
    }

    private def updateCurrentLocation(context: Context): Unit = {
      val currentLocationOption = getCurrentLocation(context)
      currentLocationOption match {
        case Some(x) => {
          if (this.currentLocation != x) {
            updateParam("currentLocation", x)
          }
        }

        case _ => 
      }
    }

    addBehaviour(updateCurrentLocation)

We then also need to update the ``checkForInfection`` function to only check for infections in the same location. We start by counting the total number of people in the Person's current location, and then count the number of infected people in the same location. Then, we modify our infection rate calculation to be based on the number of infected people in the same location.

.. code-block:: scala

    import com.bharatsim.engine.graph.patternMatcher.MatchCondition._

    val neighbours = decodedPlace.getConnections(decodedPlace.getRelation[Person]().get)
    val totalNeighbourCount = decodedPlace.getConnectionCount(decodedPlace.getRelation[Person]().get, ("currentLocation" equ currentLocation))

    if (totalNeighbourCount > 0) {
      val infectedNeighbourCount = decodedPlace
        .getConnectionCount(decodedPlace.getRelation[Person]().get, ("currentLocation" equ currentLocation) and ("infectionState" equ Infected))

      val finalInfectionRate = infectionRate * infectedNeighbourCount / totalNeighbourCount
      val toBeInfected = biasedCoinToss(finalInfectionRate)

      if (toBeInfected) {
        updateParam("infectionState", Infected)
      }
    }


The final code for the ``Main`` and ``Person`` files are as follows:


.. tabs::

  .. code-tab:: scala Main.scala 

    package sir

    import java.util.Date
    import com.bharatsim.engine.ContextBuilder._
    import com.bharatsim.engine._
    import com.bharatsim.engine.actions.StopSimulation
    import com.bharatsim.engine.basicConversions.decoders.DefaultDecoders._
    import com.bharatsim.engine.basicConversions.encoders.DefaultEncoders._
    import com.bharatsim.engine.dsl.SyntaxHelpers._
    import com.bharatsim.engine.execution.Simulation
    import com.bharatsim.engine.graph.ingestion.{GraphData, Relation}
    import com.bharatsim.engine.graph.patternMatcher.MatchCondition._
    import com.bharatsim.engine.listeners.{CsvOutputGenerator, SimulationListenerRegistry}
    import com.bharatsim.engine.models.Agent
    import com.bharatsim.engine.utils.Probability.biasedCoinToss
    import com.bharatsim.examples.epidemiology.sir.InfectionStatus._
    import com.typesafe.scalalogging.LazyLogging

    object Main extends LazyLogging {
      private val initialInfectedFraction = 0.01

      private val myTick: ScheduleUnit = new ScheduleUnit(1)
      private val myDay: ScheduleUnit = new ScheduleUnit(myTick * 24)

      def main(args: Array[String]): Unit = {

        var beforeCount = 0
        val simulation = Simulation()

        simulation.ingestData(implicit context => {
          ingestCSVData("citizen10k.csv", csvDataExtractor)
          logger.debug("Ingestion done")
        })

        simulation.defineSimulation(implicit context => {
          create24HourSchedules()

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
            new CsvOutputGenerator("src/main" + currentTime + ".csv", new SIROutputSpec(context))
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

      private def create24HourSchedules()(implicit context: Context): Unit = {
        val employeeSchedule = (myDay, myTick)
          .add[House](0, 8)
          .add[Office](9, 17)
          .add[House](18,23)

        val studentSchedule = (myDay, myTick)
          .add[House](0, 8)
          .add[Office](9, 16)
          .add[House](17, 23)

        registerSchedules(
          (employeeSchedule, (agent: Agent, _: Context) => agent.asInstanceOf[Person].age >= 18, 1),
          (studentSchedule, (agent: Agent, _: Context) => agent.asInstanceOf[Person].age < 18, 2)
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

  .. code-tab:: scala Person.scala 

    package sir

    import com.bharatsim.engine.Context
    import com.bharatsim.engine.basicConversions.decoders.DefaultDecoders._
    import com.bharatsim.engine.basicConversions.encoders.DefaultEncoders._
    import com.bharatsim.engine.graph.GraphNode
    import com.bharatsim.engine.models.{Agent, Node}
    import com.bharatsim.engine.graph.patternMatcher.MatchCondition._
    import com.bharatsim.engine.utils.Probability.{toss, biasedCoinToss}
    import sir.InfectionStatus._

    case class Person(id: Long, age: Int, infectionState: InfectionStatus, infectionDay: Int, currentLocation: String = "") extends Agent {
      final val numberOfTicksInADay: Int = 24
      private val incrementInfectionDuration: Context => Unit = (context: Context) => {
        if (isInfected && context.getCurrentStep % numberOfTicksInADay == 0) {
          updateParam("infectionDay", infectionDay + 1)
        }
      }

      private val checkForInfection: Context => Unit = (context: Context) => {
        if (isSusceptible) {
          val infectionRate = Disease.beta

          val schedule = context.fetchScheduleFor(this).get

          val currentStep = context.getCurrentStep
          val placeType: String = schedule.getForStep(currentStep)

          val places = getConnections(getRelation(placeType).get).toList

          if (places.nonEmpty) {
            val place = places.head
            val decodedPlace = decodeNode(placeType, place)

            val neighbours = decodedPlace.getConnections(decodedPlace.getRelation[Person]().get)
            val totalNeighbourCount = decodedPlace.getConnectionCount(decodedPlace.getRelation[Person]().get, ("currentLocation" equ currentLocation))

            if (totalNeighbourCount > 0) {
              val infectedNeighbourCount = decodedPlace
                .getConnectionCount(decodedPlace.getRelation[Person]().get, ("currentLocation" equ currentLocation) and ("infectionState" equ Infected))

              val finalInfectionRate = infectionRate * infectedNeighbourCount / totalNeighbourCount
              val toBeInfected = biasedCoinToss(finalInfectionRate)

              if (toBeInfected) {
                updateParam("infectionState", Infected)
              }
            }
          }
        }
      }

      private val checkForRecovery: Context => Unit = (context: Context) => {
        if (isInfected && infectionDay == Disease.lastDay)
          updateParam("infectionState", Removed)
      }

      def isSusceptible: Boolean = infectionState == Susceptible

      def isInfected: Boolean = infectionState == Infected

      def isRecovered: Boolean = infectionState == Removed

      private def getCurrentLocation(context: Context): Option[String] = {
        val schedule = context.fetchScheduleFor(this).get
        val currentStep = context.getCurrentStep
        val placeType: String = schedule.getForStep(currentStep)

        Some(placeType)
      }

      private def updateCurrentLocation(context: Context): Unit = {
        val currentLocationOption = getCurrentLocation(context)
        currentLocationOption match {
          case Some(x) => {
            if (this.currentLocation != x) {
              updateParam("currentLocation", x)
            }
          }

          case _ => 
        }
      }

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
      addBehaviour(updateCurrentLocation)

      addRelation[House]("STAYS_AT")
      addRelation[Office]("WORKS_AT")
      addRelation[School]("STUDIES_AT")
    }

