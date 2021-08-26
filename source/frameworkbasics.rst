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

  private def myCsvDataExtractor(map: Map[String, String](implicit context: Context): GraphData = {

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