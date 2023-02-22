Getting Started
===============

The BharatSim framework is written in Scala 2, so once the source code is obtained, a development environment needs to be set up. The following set of instructions need to be followed to obtain the said development environment. 


Setup Requirements
------------------

This section will deal with the preliminary requirements to run BharatSim once you have the source code. BharatSim requires a JDK (Java Development Kit)

Setting up Scala
----------------

Scala requires setting up

* JDK (Java Development Kit)
* Scala Compiler and SBT (Scala Build Tool) and a few tools

Installing JDK
~~~~~~~~~~~~~~

Java 8 or 11 can be installed from Oracle or through OpenJDK.

`Download OpenJDK <https://adoptium.net/temurin/releases/?version=11>`_. Choose the appropriate operating system with x64 JDK with versions 8 or 11. 

Installing Scala with Coursier
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Coursier** can be used to setup Scala in an easy way.


Follow the installation instructions at the `Coursier Site <https://get-coursier.io/docs/cli-installation>`_.


Once ``cs`` is downloaded, run:

.. code-block:: console

    $ ./cs setup

To check whether the set up was successful, run:

.. code-block:: console

    $ scala -version  

If the version along with a copyright notice is not returned, then reboot the system and try running the above line of code again. 

Setting up an IDE and First Run
-------------------------------

The recommended IDE is **IntelliJ Idea**. It provides a lot of features helpful for newcomers to a language. It also handles large projects well.

Running IntelliJ Idea is a bit resource intensive, so if you really have no choice [due to hardware limitations, old systems, etc.], then any text editor works with command line. We talk about IntelliJ Idea and VSCode here.

IntelliJ Idea
~~~~~~~~~~~~~

* Download the `IntelliJ Idea Community Edition <https://www.jetbrains.com/idea/download/>`_.


.. note:: On Linux, a ``.tar.gz`` file is downloaded.

  * Unzip the archive: ``tar -xvf archivename.tar.gz``.
  * Go the unarchived folder. Run the ``bin/idea.sh`` file to open IntelliJ Idea.

* Go to File --> Settings [Or press ``CTRL+ALT+S``] to open the Settings Menu. Go to **Plugins** and search and install the **Scala** Plugin.

  .. image:: _static/images/idea-scala-plugin.png

  The IDE is now setup.

  Let's open the Project. The following steps are assuming the source code directory is named ``BharatSim``.

* Go to File --> Open, and select the ``build.sbt`` file in the BharatSim source code directory. Now, select **Open as Project**. IntelliJ Idea will load the project.

* Now, we **run the SIR Model**. SIR is a simple compartmental model to analyze epidemics, where a person can be either Susceptible (S), Infected (I) or Recovered (R). We will see SIR Model in detail in the Epidemiology section.

  * In IntelliJ Idea, on the left is the project directory structure and tree. Navigate to ``BharatSim/src/main/scala/com/bharatsim/examples/epidemiology/sir`` and open the ``Main.scala`` file.

  * There will again be a Green arrow/triangle besides the line containing ``object Main``. Click on it, and ``Run 'Main'``.

    .. error:: If it gives an error like ``Ingestion Failed : java.nio.file.NoSuchFileException: citizen10k.csv``, make sure there is a file named ``citizen10k.csv`` inside the BharatSim folder. If it is not present, it might have been mistakenly deleted or misplaced. Get the source code again in that case.

  * Wait till the program finishes running. At the end, it should look like this:

    .. image:: _static/images/sir-run.png

  * The output CSV file is present at ``BharatSim/src/main/resources/output_unixtimestamp.csv``. This contains the output as per the specification in the program. This can be used to further analyze the results of the SIR Model run.


If you reached till here, Congratulations! IntelliJ Idea is setup and working correctly.

.. tip:: If the green "Run" arrows do not appear, or some other issues occur and the program does not start to run due to failed dependencies or Scala versions, then there is a simple trick to try.

  IntelliJ Idea stores its own configuration for the project inside a ``.idea`` directory in the project folder. Delete this folder, and start from scratch, by opening the ``build.sbt`` file again and then importing the project.


Visual Studio Code
~~~~~~~~~~~~~~~~~~

* Download the `Visual Studio Code <https://code.visualstudio.com/download>`_. Open VSCode.

* Go to View --> Extensions and search "Scala". Install the **Scala Syntax (official)** and **Scala (Metals)** extensions.

  .. image:: _static/images/vscode-extensions.png

  Let's open the Project. The following steps are assuming the source code directory is named ``BharatSim``.

* Go to File --> Open Folder, and select the ``BharatSim`` folder. When prompted by VSCode, click on **Import Build**. This uses an open source tool called sbt to compile and test Scala projects.

  * If you miss it somehow, go to View --> Command Palette [or press ``CTRL+SHIFT+P``] and search for "Import build". Click on "Metals: Import build" and sit back for a while as VSCode goes through the project structure and builds the project. If you are unable to find such an option, make sure you installed the Metals extension. Restart VSCode if needed.

    .. error:: If there is an error notification during the import build process, click on the "more information" option. A new tab will open called Metal Doctor and it will display the source of the error. If the error is in Debugging, then the warning can be ignored and set up process can be carried on. 

* Now, we **run the SIR Model**. SIR is a simple compartmental model to analyze epidemics, where a person can be either Susceptible (S), Infected (I) or Recovered (R). We will see SIR Model in detail in the Epidemiology section.

  * In VSCode, on the left is the project directory structure and tree. Navigate to ``BharatSim/src/main/scala/com/bharatsim/examples/epidemiology/sir`` and open the ``Main.scala`` file.
  * There will again be a ``run | debug`` above the line containing ``object Main``. Click on ``run``.

    .. error:: If it gives an error like ``Ingestion Failed : java.nio.file.NoSuchFileException: citizen10k.csv``, make sure there is a file named ``citizen10k.csv`` inside the BharatSim folder. If it is not present, it might have been mistakenly deleted or misplaced. Get the source code again in that case. If the problem persists, then copy the ``citizen.csv`` and place it in the main folder ``BharatSim``.

  * Wait till the program finishes running. At the end, it should look like this:

    .. image:: _static/images/vscode-sir-run.png

  * The output CSV file is present at ``BharatSim/src/main/resources/output_unixtimestamp.csv``. This contains the output as per the specification in the program. This can be used to further analyze the results of the SIR Model run.


If you reached till here, Congratulations! VSCode is setup and working correctly.


Running Scala on Command Line
-----------------------------

Let's assume the source code directory is named ``BharatSim``. Navigate to the directory in terminal. The sbt tool is often utilized to build a project, which is nothing but compiling,  running and testing the project. It also offers the capability of executing each of these processes individually.

* Compile the project:

  .. code-block:: console

    $ sbt compile

* Now, we **run the SIR Model**. SIR is a simple compartmental model to analyze epidemics, where a person can be either Susceptible (S), Infected (I) or Recovered (R). We will see SIR Model in detail in the Epidemiology section.

  * Do ``sbt run`` and wait for a still of main classes to appear on the screen. Select the class number associated to ``com.bharatsim.examples.epidemiology.sir.Main``. It may appear as if the class number is not being typed, but it is! Just input the number and press ENTER . It should start running the simulation.

    .. error:: If it gives an error like ``Ingestion Failed : java.nio.file.NoSuchFileException: citizen10k.csv``, make sure there is a file named ``citizen10k.csv`` inside the BharatSim folder. If it is not present, it might have been mistakenly deleted or misplaced. Get the source code again in that case. If the problem persists, then copy the ``citizen.csv`` and place it in the main folder ``BharatSim``.

    It should look like this:

    .. image:: _static/images/cli-sir-run.png

* The output CSV file is present at ``BharatSim/src/main/resources/output_unixtimestamp.csv``. This contains the output as per the specification in the program. This can be used to further analyze the results of the SIR Model run.

This is how Scala programs can be run through the command line.

.. tip:: Another way to operate Scala through the command line is to simply type ``sbt`` and run the sbt console. The other commands can now be run in succession simply as ``compile``, ``run`` and more.

  .. image:: _static/images/sbt-console.png
