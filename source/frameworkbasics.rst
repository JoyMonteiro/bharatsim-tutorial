Framework Basics
================

Inputs and Outputs in BharatSim
-------------------------------

Inputs
~~~~~~

BharatSim uses a `CSV <https://en.wikipedia.org/wiki/Comma-separated_values>`_ file as an input. It is equipped to **ingest** data from a file, by reading it and converting the data to the network.

In order to ingest data, we need to make use of three functions:

* ``ingestData``, which is a method of the ``simulation`` class
* ``ingestCSVData``, a method of the ``ContextBuilder`` object
* A user-defined function which tells the program what data to extract and what to do with it

Using the framework-defined functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

First, we need to import the necessary packages:

.. code-block:: scala

  import com.bharatsim.engine.ContextBuilder._
  import com.bharatsim.engine.execution.Simulation
  import com.bharatsim.engine.graph.ingestion.{GraphData, Relation}

.. hint:: You'll see why we import ``GraphData`` and ``Relation`` in the next section!

The next step is to create an instance of the simulation class in the main function,

.. code-block:: scala

  val simulation = Simulation()

We then ingest the data in the following way:

.. code-block:: scala

  simulation.ingestData(implicit context => {
    ingestCSVData("input.csv", myCsvDataExtractor)
    logger.debug("Ingestion done")
  })

where ``myCsvDataExtractor`` is the user-defined function.

.. note:: The above block of code essentially causes the data from the CSV file to be read one line at a time

Using the User-Defined function
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The user-defined function (``myCsvDataExtractor``, in our case) will depend on the data we want to extract. As an example, let's consider that we have data on a number of cats, each with their own ID, name, city of residence, an integer ID for the city, and a particular colour. Our CSV file would look like

.. csv-table:: input.csv
   :file: _static/csvs/example_input.csv
   :widths: 5, 10, 10, 10, 10
   :header-rows: 1

Let's assume we've already defined the following:

* a case class ``Cat`` with three attributes, ``id``, ``name`` and ``colour``
* another case class ``City`` with two attributes, ``id`` and ``cityname``

Our function ``myCsvDataExtractor`` should do the following

* accept a map of keys (the CSV headers) and values (the CSV element in the row corresponding to the header)
* accept the context as an implicit parameter
* return a ``GraphData`` object (which you can read about `here <#>`_)

.. note:: The map is already provided by the ``ingestCsvData`` function

.. code-block:: scala

  private def myCsvDataExtractor(map: Map[String, String](implicit context: Context): GraphData = {}

The first thing we need to do in the function is store the CSV data to appropriate variables.

.. code-block:: scala

    val catName = map("Name").toString
    val catID = map("ID").toLong
    val catCity = map("City").toString
    val catCityID = map("CityID").toLong
    val catColour = map("Colour").toString

.. note:: The key of the ``map`` is the header from the CSV file.

We then use a `Constructor <https://alvinalexander.com/scala/scala-class-examples-constructors-case-classes-parameters/>`_ to create an instance of the ``Cat`` class, for the cat pertaining to a particular row in the CSV. We then do the same for the ``City`` class.

.. code-block:: scala

    val singleCat: Cat = Cat(
      catID,
      catName,
      catColour
    )

    val singleCity: City = City(
      catCityId,
      catCity
    )

Next, we establish *relations* that will link nodes on the graph. We make a ``livesIn`` relation between the cat and the city, and a ``contains`` relation between the city and the cat. To do this, we specify the classes the relation is formed between, and then the unique IDs of the nodes with the relation in between them.

.. code-block:: scala

    val livesIn = Relation[Cat, City](catID, "LIVES_IN", catCityID)
    val contains = Relation[City, Cat](catCityID, "CONTAINS", catID)

We then create an instance of the ``GraphData`` class, and add the nodes and relations to it

.. code-block:: scala

    val graphData = GraphData()
    graphData.addNode(catID, singleCat)
    graphData.addNode(catCityID, singleCity)
    graphData.addRelations(staysAt, contains)

.. note:: The first parameter of ``graphData.addNode`` is the unique key of the node.

Finally, we need our function to return the ``graphData`` object we've made:

.. code-block:: scala

    graphData

.. hint:: In scala, the last line of a function is treated as a return, and so this is valid syntax.

Putting it all together, our user-defined ``myCsvDataExtractor`` function is

.. code-block:: scala

  private def myCsvDataExtractor(map: Map[String, String])(implicit context: Context): GraphData = {

    val catName = map("Name").toString
    val catID = map("ID").toLong
    val catCity = map("City").toString
    val catCityID = map("CityID").toLong
    val catColour = map("Colour").toString

    val singleCat: Cat = Cat(
      catID,
      catName,
      catColour
    )

    val singleCity: City = City(
      catCityId,
      catCity
    )

    val livesIn = Relation[Cat, City](catID, "LIVES_IN", catCityID)
    val contains = Relation[City, Cat](catCityID, "CONTAINS", catID)

    val graphData = GraphData()
    graphData.addNode(catID, singleCat)
    graphData.addNode(catCityID, singleCity)
    graphData.addRelations(staysAt, contains)

    graphData
  }

.. note:: You may have noticed that in the CSV file, two cats (namely, Coppe and Marie) both live in the same city (Crossbell). That does not, however, lead to two nodes being created for the same city. A node is defined by it's unique key and it's instance. In this example, the unique key is the city ID (which is the same for both cats - ``100``) and the instance is the corresponding object ``singleCity``, which is again identical for both the cats (the attributes are ``100`` and ``"Crossbell"``, respectively). As such, the same node is used, and the city doesn't duplicate in the graph.

Outputs
~~~~~~~

A convenient way to store the output is by using a CSV file. Scala is `capable of writing to files <https://alvinalexander.com/scala/how-to-write-text-files-in-scala-printwriter-filewriter/>`_, but BharatSim simplifies the process when it comes to CSV outputs.

.. note:: In case the quantities you'd like to output are fairly simple, you could use Scala's ``println`` function to directly output what you need.

Saving your output to a CSV file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

BharatSim relies on a trait called ``SimulationListener`` to help output data .

``SimulationListener`` contains 4 methods, each of which allow us to perform a task in one of the following situations:

* At the start of the simulation
* At the start of every time step
* At the end of every time step
* At the end of the simulation

The BharatSim engine also contains a class called ``CsvOutputGenerator``, an extension of ``SimulationListener`` which has two attributes:

* ``path``, the desired path for the output file to be stored
* ``csvSpecs``, a user-defined class that outputs the headers and the rows required. Note that this user-defined class should extend the ``CSVSpecs`` trait and override the ``getHeaders`` and ``getRows`` methods.

This class writes the headers at the start of the simulation, writes the rows at the start of every time step, and closes the writer at the end of the simulation.

Output at a single instant of time
__________________________________

We can define a class as follows:

.. code-block:: scala

  import com.bharatsim.engine.Context
  import com.bharatsim.engine.listeners.CSVSpecs

  class MyOutputSpec(context: Context) extends CSVSpecs {
    override def getHeaders: List[String] =
      List(
        "Header1",
        "Header2",
        "Header3"
      )
    override def getRows(): List[List[Any]] = {
      val elementInRow: String = "row" + context.getCurrentStep.toString
      val row = List(
        elementInRow,
        elementInRow,
        elementInRow
      )
      List(row)
    }
  }

Now, we need to create an instance of the ``CsvOutputGenerator`` class that uses ``MyOutputSpec``, and call the required methods. First, we need to import ``CsvOutputGenerator`` into our main class:

.. code-block:: scala

  import com.bharatsim.engine.listeners.CsvOutputGenerator

Next, we add the following code snippet inside ``simulation.defineSimulation`` in the main function:

.. code-block:: scala

  var outputGenerator = new CsvOutputGenerator("src/main/resources/output.csv", new MyOutputSpec(context))
  outputGenerator.onSimulationStart(context)
  outputGenerator.onStepStart(context)
  outputGenerator.onSimulationEnd(context)

.. note:: Calling the ``onStepEnd`` method of the class isn't necessary, as the ``CsvOutputGenerator`` class currently does nothing when it's called.

The output is

.. csv-table:: output.csv
   :file: _static/csvs/single_output.csv
   :widths: 20, 20, 20
   :header-rows: 1


.. hint:: In case you want your outputs generated *after* the simulation is completed, you can place the above 4 lines of code inside ``simulation.onCompleteSimulation``.

You can see a more in-depth example of this in :ref:`Saving location-level information from the simulation`.

Output at every time step
_________________________

If we'd like to investigate the dynamics of the simulation as it evolves with time, we essentially need to call the three methods described above every time step. BharatSim simplifies things with ``SimulationListenerRegistry``, which allows us to **register** the output generator in the simulation (similar to how we registered `agents <#>`_), so that it writes data to the CSV file at every time step.

First, we must import ``CsvOutputGenerator`` and ``SimulationListenerRegistry``

.. code-block:: scala
   
  import com.bharatsim.engine.listeners.{CsvOutputGenerator, SimulationListenerRegistry}


Next, we register it using the ``register`` method of ``SimulationListenerRegistry``. Note that the following code snippet must go inside ``simulation.defineSimulation`` in the main function.

.. code-block:: scala

  SimulationListenerRegistry.register(
    new CsvOutputGenerator("src/main/resources/output.csv", new myOutputSpec(context))
    )

where ``myCsvSpecs`` is the user-defined class which requires the context as an attribute.

Now, the output is

.. csv-table:: output.csv
   :file: _static/csvs/multiple_output_truncated.csv
   :widths: 20, 20, 20
   :header-rows: 1

and so on, until the tick at which the simulation ends.

.. hint:: Running the above block of code once will cause a file called ``output`` to be created at ``src/main/resources/``. However, running it again will rewrite the contents of the file with the new output. You can get around this by adding the current time to the output as a string. For example,

  .. code-block:: scala

    val currentTime = new Date().getTime

    SimulationListenerRegistry.register(
        new CsvOutputGenerator("src/main/resources/output_" + currentTime + ".csv", new SIROutputSpec(context))
      )
      
  Note that ``Date().getTime`` returns the time as a `UNIX timestamp <https://en.wikipedia.org/wiki/Unix_time>`_, and so your output will contain a long integer after the underscore.

For a more detailed example of how to output data to a CSV file, please refer to the `Writing your first program <#>`_ section.

Interventions in BharatSim
---------------------------------------

Interventions are events that get activated when the provided condition is satisfied. 
Each intervention is identified uniquely by the **name** of an intervention.
Each intervention needs to have an **activation condition** and a **condition to deactivate**.
The activation condition and deactivation condition are ``boolean`` decisions.

Additionally, the user can define a **activation action** and **per tick action**, which are optional.
 **activation action**: This action is invoked only once, i.e., at the activation of the intervention.

 **per tick action**: This action is invoked per tick for which an intervention is active.

There are four different intervention classes available in BharatSim


1. Intervention
~~~~~~~~~~~~~~~~~~~~~
This is a Generic intervention that can be invoked several times or once when the activation condition is met based on 
the parameters passed. This can be used to implement lockdowns during an epidemic every time the infected fraction of 
agent reaches a certain threshold.

Five parameters can be passed ``Intervention`` object:

    * ``interventionName``: This is the unique name of the intervention.
    * ``activationCondition``: This function tells whether this intervention should be activated or not. The function should return a Boolean value.
    * ``deActivationCondition``: This function tells whether this intervention should be deactivated. The function should return a Boolean value.
    * ``firstTimeExecution``: This is an optional function that will be executed at the start of the intervention
    * ``whenActiveActionFunc``: This is an optional function  executed per tick when intervention is active.

Example using the ``Intervention`` object:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Here we will use the ``Intervention`` class to implement a lockdown every time the number of infected
agents are greater than or equal to 2000.

First, we will define the intervention named addLockdown and initialise the variables.

  ``interventionActivatedAt`` stores the information about when the intervention was activated. Here it is initialised to be zero and 
  will be updated once the intervention is activated.

  ``activationCondition`` has a boolean value that defines when the intervention has to be activated. 
  Here the intervention gets activated once the number of infected agents is greater than or equal to 2000.

  ``firstTimeExecution`` specifies what should be done once the intervention is activated. This is only executed once when the
  intervention is activated

  ``deActivationCondition`` specifies the condition when the intervention should be stoped. Here it is stopped after 14 days, i.e. ``2*14``
  ticks after the intervention activation.

.. code-block:: scala

    private def addLockdown(implicit context: Context): Unit = {

    var interventionActivatedAt = 0
    val interventionName = "lockdown"
    val activationCondition = (context: Context) => getInfectedCount(context) >= 2000
    val firstTimeExecution = (context: Context) => interventionActivatedAt = context.getCurrentStep
    val deActivationCondition = (context: Context) => {
    context.getCurrentStep >= interventionActivatedAt + 2*14
    }
  }

Now we have to create an instance of the ``Intervention`` object. Here we define ``intervention`` and pass the values defined 
in earlier` to the ``Intervention`` object.

.. code-block:: scala

    val intervention =
    Intervention(interventionName, activationCondition, deActivationCondition, firstTimeExecution)

Now we define a new schedule that has to come into effect once the intervention is activated.
Here we define the ``lockdownSchedule`` such that all agent stays in the house throughout the day.
Here the 0,1 passed to the ``add[House]`` makes the agent stay in its house from tick 0 to the end of tick 1 in a day.
Here one day is defined as having two ticks, i.e. 0 and 1. So this makes the agent stay home for the entire day as long as 
the intervention remains activated.

.. code-block:: scala

   val lockdownSchedule = (myDay, myTick).add[House](0, 1)

Now we have to register the register the intervention and the schedules using ``registerIntervention`` and ``registerSchedules``
respectively. We also have to pass the ``Agent`` and ``Context`` to ``registerSchedules``.

.. code-block:: scala
  
    registerIntervention(intervention)
    registerSchedules(
      (
        lockdownSchedule,
        (agent: Agent, context: Context)
      )
    )

The complete definition of ``addLockdown`` is given below.

.. code-block:: scala

    private def addLockdown(implicit context: Context): Unit = {

    var interventionActivatedAt = 0
    val interventionName = "lockdown"
    val activationCondition = (context: Context) => getInfectedCount(context) >= 2000
    val firstTimeExecution = (context: Context) => interventionActivatedAt = context.getCurrentStep
    val deActivationCondition = (context: Context) => {
    context.getCurrentStep >= interventionActivatedAt + 2*14
    }
    val intervention =
    Intervention(interventionName, activationCondition, deActivationCondition, firstTimeExecution)

    val lockdownSchedule = (myDay, myTick).add[House](0, 1)

    registerIntervention(intervention)
    registerSchedules(
    (
        lockdownSchedule,
        (agent: Agent, context: Context) 
    )
    )
    }

.. hint:: ``addLockdown`` should be included in the definition of the simulation.

  .. code-block:: scala

    simulation.defineSimulation(implicit context => {
    addLockdown
    }

2.IntervalBasedIntervention
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This can be used to invoke an intervention that starts and end at specific ticks. This can be used for 
giving relaxations in the epidemic regulations for a specified period, for example, during a festival.

Five paramertes can be passed to the ``IntervalBasedIntervention`` object:

    * ``interventionName``: This is the unique intervention name.
    * ``startTick``: This integer specifies the start tick for intervention (inclusive); it should not be greater than endTick.
    * ``endTick``: This is an integer that specifies the end tick for the intervention (It is exclusive, and intervention will not be active at "endTick".)
    * ``firstTimeActionFunc``: This is an optional function which gets executed when simulation starts.
    * ``whenActiveActionFunc``: This is an optional function executed per tick when the simulation is active.

Example using the ``IntervalBasedIntervention`` object:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We will try to implement during the 50th to the 100th tick, i.e. for 25 days. (Since in 1 day is defined as two ticks by default)

First we will define the variables ``interventionName``. In the function call of IntervalBasedIntervention(), we will pass the 
``interventionName``, ``startTick`` and ``endTick``.

.. code-block:: scala

    private def addLockdown(implicit context: Context): Unit = {

    val interventionName = "lockdown"
    val intervention = IntervalBasedIntervention(interventionName, 50, 100)
    }

Now we define the ``lockdownSchedule`` to force all agents to stay home for the entire day throughout all ticks when the 
intervention is active.

.. code-block:: scala

  val lockdownSchedule = (Day, Hour).add[House](0, 1)

Now we will register both the intervention as well as the schedule.

.. code-block:: scala

  registerIntervention(intervention)
    registerSchedules(
    (

        lockdownSchedule,
        (agent: Agent, context: Context) 

    )
    )

The entire definition of ``addLockdown`` intervention using the IntervalBasedIntervention is given below:

.. code-block:: scala

    private def addLockdown(implicit context: Context): Unit = {

    val interventionName = "lockdown"
    val intervention = IntervalBasedIntervention(interventionName, 50, 100)

    val lockdownSchedule = (Day, Hour).add[House](0, 1)

    registerIntervention(intervention)
    registerSchedules(
    (

        lockdownSchedule,
        (agent: Agent, context: Context) 

    )
    )
    }

.. hint:: ``addLockdown`` should be included in the definition of the simulation.

  .. code-block:: scala

    simulation.defineSimulation(implicit context => {
    addLockdown
    }

3.OffsetBasedIntervention
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This can be used to invoke interventions that end after 'n' ticks. It gets invoked when the ``shouldActivateWhen`` function is true.
This can be used to implement a lockdown when the number of infected agents reaches a particular threshold and stays active till n ticks.

Five parameters can be passed to the ``OffsetBasedIntervention`` object:

    * ``interventionName``: This is the unique name of the intervention.
    * ``shouldActivateWhen``: This function decides when should the intervention be activated.
    * ``endAfterNTicks``: This is the offset 'n'; simulation will end after n ticks from the star tick.
    * ``firstTimeActionFunc``:This is an optional function which gets executed when simulation starts.
    * ``whenActiveActionFunc``: This is an optional function executed per tick when the simulation is active.

Example suing the ``OffsetBasedIntervention`` object:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
We will implement a lockdown when the number of infected individuals is greater than or equal to 2000 and stays active for 28 ticks
(14 days) from the start of the lockdown.
First, we will define the intervention named addLockdown and initialise the variables.

  ``activationCondition`` has a boolean value that defines when the intervention has to be activated. 
  Here the intervention gets activated once the number of infected agents
  ``interventionName`` contains the name of the intervention

.. code-block:: scala

    private def addLockdown(implicit context: Context): Unit = {

    val interventionName = "lockdown"
    val activationCondition = (context: Context) => getInfectedCount(context) >= 2000
    }
  }

Now we have to create an instance of the ``Intervention`` object. Here we define ``intervention`` and pass 
the ``interventionName``, ``activationCondition`` and the number of ticks after which
the intervention has to stop to the ``Intervention`` object.

.. code-block:: scala

    val intervention =
    Intervention(interventionName, activationCondition,28)

Now we define a new schedule that has to come into effect once the intervention is activated.
Here we define the ``lockdownSchedule`` such that all agent stays in the house throughout the day.
Here the 0,1 passed to the ``add[House]`` makes the agent stay in its house from tick 0 to the end of tick 1 in a day.
Here one day is defined as having two ticks, i.e. 0 and 1. So this makes the agent stay home for the entire day as long as 
the intervention remains activated.

.. code-block:: scala

   val lockdownSchedule = (myDay, myTick).add[House](0, 1)

Now we have to register the register the intervention and the schedules using ``registerIntervention`` and ``registerSchedules``
respectively. We also have to pass the ``Agent`` and ``Context`` to ``registerSchedules``.

.. code-block:: scala
  
    registerIntervention(intervention)
    registerSchedules(
      (
        lockdownSchedule,
        (agent: Agent, context: Context)
      )
    )

The complete definition of ``addLockdown`` using OffsetBasedIntervention is given below.

.. code-block:: scala

    private def addLockdown(implicit context: Context): Unit = {

    var interventionActivatedAt = 0
    val interventionName = "lockdown"
    val activationCondition = (context: Context) => getInfectedCount(context) >= 2000
    val firstTimeExecution = (context: Context) => interventionActivatedAt = context.getCurrentStep
    val deActivationCondition = (context: Context) => {
    context.getCurrentStep >= interventionActivatedAt + 2*14
    }
    val intervention =
    Intervention(interventionName, activationCondition, deActivationCondition, firstTimeExecution)

    val lockdownSchedule = (myDay, myTick).add[House](0, 1)

    registerIntervention(intervention)
    registerSchedules(
    (
        lockdownSchedule,
        (agent: Agent, context: Context) 
    )
    )
    }

.. hint:: ``addLockdown`` should be included in the definition of the simulation.

  .. code-block:: scala

    simulation.defineSimulation(implicit context => {
    addLockdown
    }

4.SingleInvocationIntervention
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This can be used to create an intervention that will be invoked only once in the simulation.

Five parameters can be passed to the ``SingleInvocationIntervention`` object:

    * ``interventionName``: This is the unique name of the intervention.
    * ``shouldActivateFunc``: This function tells whether this intervention should be activated.
    * ``shouldDeactivateFunc``: This function tells whether this intervention should be deactivated.
    * ``firstTimeActionFunc``: This is an optional function  executed at the start of the intervention.
    * ``whenActiveActionFunc``: This is an optional function  executed per tick when intervention is active.

Example using the ``SingleInvocationIntervention`` object:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Here we will use the ``Intervention`` class to implement a lockdown just once when the number of infected
agent crosses 2000.

First, we will define the intervention named addLockdown and initialise the variables.

  ``interventionActivatedAt`` stores the information about when the intervention was activated. Here it is initialised to be zero and 
  will be updated once the intervention is activated.

  ``activationCondition`` has a boolean value that defines when the intervention has to be activated. 
  Here the intervention gets activated once the number of infected agents is greater than or equal to 2000.

  ``firstTimeExecution`` specifies what should be done once the intervention is activated. This is only executed once when the
  intervention is activated

  ``deActivationCondition`` specifies the condition when the intervention should be stoped. Here it is stopped after 14 days, i.e. ``2*14``
  ticks after the intervention activation.

.. code-block:: scala

    private def addLockdown(implicit context: Context): Unit = {

    var interventionActivatedAt = 0
    val interventionName = "lockdown"
    val activationCondition = (context: Context) => getInfectedCount(context) >= 2000
    val firstTimeExecution = (context: Context) => interventionActivatedAt = context.getCurrentStep
    val deActivationCondition = (context: Context) => {
    context.getCurrentStep >= interventionActivatedAt + 2*14
    }
  }

Now we have to create an instance of the ``Intervention`` object. Here we define ``intervention`` and pass the values defined 
in earlier` to the ``Intervention`` object.

.. code-block:: scala

    val intervention =
    SingleInvocationIntervention(interventionName, activationCondition, deActivationCondition, firstTimeExecution)

Now we define a new schedule that has to come into effect once the intervention is activated.
Here we define the ``lockdownSchedule`` such that all agent stays in the house throughout the day.
Here the 0,1 passed to the ``add[House]`` makes the agent stay in its house from tick 0 to the end of tick 1 in a day.
Here one day is defined as having two ticks, i.e. 0 and 1. So this makes the agent stay home for the entire day as long as 
the intervention remains activated.

.. code-block:: scala

   val lockdownSchedule = (myDay, myTick).add[House](0, 1)

Now we have to register the register the intervention and the schedules using ``registerIntervention`` and ``registerSchedules``
respectively. We also have to pass the ``Agent`` and ``Context`` to ``registerSchedules``.

.. code-block:: scala
  
    registerIntervention(intervention)
    registerSchedules(
      (
        lockdownSchedule,
        (agent: Agent, context: Context)
      )
    )

The complete definition of ``addLockdown`` using SingleInvocationIntervention is given below.

.. code-block:: scala

    private def addLockdown(implicit context: Context): Unit = {

    var interventionActivatedAt = 0
    val interventionName = "lockdown"
    val activationCondition = (context: Context) => getInfectedCount(context) >= 2000
    val firstTimeExecution = (context: Context) => interventionActivatedAt = context.getCurrentStep
    val deActivationCondition = (context: Context) => {
    context.getCurrentStep >= interventionActivatedAt + 2*14
    }
    val intervention =
    SingleInvocationIntervention(interventionName, activationCondition, deActivationCondition, firstTimeExecution)

    val lockdownSchedule = (myDay, myTick).add[House](0, 1)

    registerIntervention(intervention)
    registerSchedules(
    (
        lockdownSchedule,
        (agent: Agent, context: Context) 
    )
    )
    }

.. hint:: ``addLockdown`` should be included in the definition of the simulation.

  .. code-block:: scala

    simulation.defineSimulation(implicit context => {
    addLockdown
    }

