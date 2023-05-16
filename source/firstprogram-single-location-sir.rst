
Single Location SIR
===================

This section will look at the disease progression in a single location and observe its dynamics. 


Creating an Empty Class
^^^^^^^^^^^^^^^^^^^^^^^
Create a new project in the required directory. There is no need to create a new folder, as creating a new project automatically creates a new folder. Name this folder ``sir`` and change the language to scala. The build system should be chosen to be sbt. 

Navigate to ``src\main\scala`` and right click on the folder in IntelliJ. Select a new package and rename it sir. Again right click on sir package, select a Scala class followed by object. Call this object ``Main``. 

The empty class should look like this. 

.. image:: _static/images/New_class.png

Now we can define a main function that has no input and has no output. The syntax and indentation of defining a function is as follows

.. code-block:: scala

    def main(args: Array[String]): Unit = {
    }

The ``args`` means that the argument or the input is an array of Strings and the output is of type ``Unit``, which corresponds to void means that there is no output. The code should look like this, 

.. note::  void return means that the function returns nothing at all. Remember nothing is different from 0, or empty list. 

.. image:: _static/images/EmptyClass.png

.. note:: Notice how the object ``Main`` has changed color from grey to white. This is an IntelliJ feature which lets the user know if the object/class/variable is being used

On running this, the output message should read ``Process finished with exit code 0``


Implementing a single-location SIR
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Before we can input a file or simulate a disease, we need to make a few classes which are essential to the workings of the framework. These classes need to be imported to the main class to make the code easier to understand and clutter-free. The framework is extremely inter-connected and defining the same functions over and over again is tedious and computationally heavy. 

.. tabs::

  .. group-tab:: InfectionStatus.scala

    InfectionStatus class is a scala object class that stores the compartments of the disease, and in our case ``Susceptible``, ``Infected``, and ``Recovered``. This class connects the instance of the compartments to the their string counterparts. 

    * BasicDecoder: This functions takes a string value and converts it to either a node or throws an exception. The latter is only the case when the input type is not in form of a string. 
    * BasicEncoder: This takes the instance and converts it to a string. In the simple case, there are three possibilities which are ``Susceptible``, ``Infected`` and ``Recovered``
    * Extends: This allows the functions of one class to be used in another. In this case, the functions of ``Enumeration`` are made available in the class ``InfectedStatus`` because of ``extends``


  .. group-tab:: Disease.scala

    Much like ``InfectionStatus``, this is also a scala object class and this stores the characteristics of the disease; the beta value and the when the infection will end. 

    * Final val: this value can not be over-written in any other class or function. 

    A disease is defined by beta and how long it lasts (for further information, refer to `Epidemiology  <https://bharatsim.readthedocs.io/en/latest/epidemiology.html>`_), and final val makes sure that the defining characteristics of the disease does not change during the course of the simulation. 

  .. group-tab:: House.scala

    This is scala case class that is a stores the locations of the individuals which are the part of the network. Since there is only one location, then only one class is required to define the location.

    * addRelation: connects the individuals to the location or in this case the ``House``. On further expanding the locations, we will keep addings relationships in different classes. 
    * Person: It is a class defining the agent (or individual) in this simulation. 

  .. group-tab:: Person.scala

    This is also a scala case class. This class decribes the behaviours of the individuals in the ``Network``, how their schedule looks like, the manner in which they can get infected and recovered. Since this is a simple case, only the relationship should be taken care of.

    * ``Age`` and ``Infection`` day are of type Int. There are only a limited number of values these variables can take and hence datatype Int will be suffice.
    * The data type of ``ID`` is long since there are many citizens and larger data space is required than Int and hence long is used. 

The code for each of the above class is provided below. 

.. tabs::

  .. code-tab:: scala InfectionStatus.scala
    

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

  .. code-tab:: scala Disease.scala 

    package sir

    object Disease {
      final val beta: Double = 0.3
      final val lastDay: Int = 12
    }

  .. code-tab:: scala House.scala

    package sir
    import com.bharatsim.engine.models.Network

    case class House(id: Long) extends Network {
      addRelation[Person]("HOUSES")

      override def getContactProbability(): Double = 1
    }

  .. code-tab:: scala Person.scala

    package sir

    import com.bharatsim.engine.models.{Agent, Node}
    import sir.InfectionStatus._

    case class Person(id: Long, age: Int, infectionState: InfectionStatus, infectionDay: Int) extends Agent {

      addRelation[House]("STAYS_AT")
    }

Inputting a File
^^^^^^^^^^^^^^^^

To begin we must import a series of libraries and the function of each libraries will be explained as and when they are required. 

.. code-block:: scala

  import com.bharatsim.engine.Context
  import com.bharatsim.engine.ContextBuilder._
  import com.bharatsim.engine.execution.Simulation
  import com.bharatsim.engine.graph.ingestion.{GraphData, Relation}
  import com.typesafe.scalalogging.LazyLogging
  import com.bharatsim.engine.utils.Probability.biasedCoinToss
  import com.bharatsim.engine.basicConversions.encoders.DefaultEncoders._

There needs to be a modification in the line where we have defined the object. We need to make use of a keywork called ``extends`` which allows one class to inherit the properties of another class. 

.. code-block:: scala

  object Main extends LazyLogging

By extending ``LazyLogging``, all the properties of this class are made available in ``Main``. The ``LazyLogging`` class allows the user to display or output information. It can be thought of as better version of ``SystemOut``.

.. note:: When libraries or variables are not being used they appear grey in color, and as soon as they are called, they become colored again

Since ``LazyLogging`` is being used, it changes color from grey. 

The next step is to define a private value called ``initialInfectedFraction`` and set it to 0.01. Private value means that this will only be available in the defining class and not outside. This will be made accessible to the function we are about to define. 

In the main function we had earlier defined, we can create an instance of the simulation class. 

.. code-block:: scala 

  val simulation = Simulation()

.. note:: val is an immutable variable and this implies that the value of this can not change. 

Then we ingest the csv file in the following manner 

.. code-block:: scala

  simulation.ingestData(implicit context => {
  ingestCSVData("input.csv", csvDataExtractor) 
  logger.debug("Ingestion done")
  })

Here ``csvDataExtractor`` is a user defined function which we will get to later. 

On running the code, an error pops up displaying that ``csvDataExtractor`` is not defined. 

The ``csvDataExtractor`` function is defined in the following manner

.. code-block:: scala
  
  private def csvDataExtractor(map: Map[String, String])(implicit context: Context): GraphData = {
  }

Once the function is defined and we need it to the following things, 

1. Accept the Context as an input parameter
2. CSV header and corresponding values
3. Return the data in the form of GraphData

The first step depends on the CSV file that is being imported since it depends on the headers of the data. In BharatSim, the CSV files usually have the following columns, 

.. code-block:: scala

    val citizenId = map("Agent_ID").toLong
    val age = map("Age").toInt
    val homeId = map("HHID").toLong

.. note:: The csvDataExtractor reads the csv file line by line and defines each citizen line by line. 

The next step is to determine if the citizen imported is infected or not. 

.. code-block:: scala

  val initialInfectionState = if (biasedCoinToss(initialInfectedFraction)) "Infected" else "Susceptible"
  
If the ``biasedCoinToss`` returns ``True``, then the citizen analyzed is infected from the disease. Using the data obtained from the CSV file and the infection state, we can create an instance of the citizen.

.. code-block:: scala

    val citizen: Person = Person(
    citizenId,
    age,
    InfectionStatus.withName(initialInfectionState),
    0
    )

Once this is done, ``relationships`` need to be established that will connect the nodes on the graph. The citizen will ``Stay At`` the house, and the house will ``House`` the citizen. The ``relationship`` needs to be established both the ways, as the first relationship links the citizen node to the house node and the second one links the house node to the citizen one. 

.. code-block:: scala
  
  val home = House(homeId)
  val staysAt = Relation[Person, House](citizenId, "STAYS_AT", homeId)
  val memberOf = Relation[House, Person](homeId, "HOUSES", citizenId)

.. note:: A House ``HOUSES`` an Agent and an Agent ``STAYS_AT`` a House so these two relations are inherently reflections of each other. The first relation is specified in the House class, while the second one is specified in the ``Person`` class (Refer to the classes above). The same defination of relationships can be extended to any pair of Agents (``Student``, ``Employer``) and corresponding locations (``School``, ``Office``). 


Then we create an instance of the ``GraphData`` and add the aforementioned nodes and relationships

.. code-block:: scala

  val graphData = GraphData()
  graphData.addNode(citizenId, citizen)
  graphData.addNode(homeId, home)
  graphData.addRelations(staysAt, memberOf)

Once the nodes and relationships have been established, we can then return the ``GraphData``. Unlike python, no return keywork is actually required. In scala, the last line has to be just value that has to be returned. 

.. code-block:: scala

  graphData

Compiling all the lines together, the ``csvDataExtractor`` function and the main function looks like 

.. code-block:: scala

  def main(args: Array[String]): Unit = {

    var beforeCount = 0
    val simulation = Simulation()

    simulation.ingestData(implicit context => {
      ingestCSVData("citizen10k.csv", csvDataExtractor)
      logger.debug("Ingestion done")
    })

  private def csvDataExtractor(map: Map[String, String])(implicit context: Context): GraphData = {

    val citizenId = map("Agent_ID").toLong
    val age = map("Age").toInt
    val homeId = map("HHID").toLong

    val initialInfectionState = if (biasedCoinToss(initialInfectedFraction)) "Infected" else "Susceptible"

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

    graphData
  }
