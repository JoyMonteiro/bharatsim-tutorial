Interventions in BharatSim
---------------------------------------

Interventions are punctual events that occur when some external condition is satisfied. These could, for example, be policy-level decisions made by governments, or environmental factors like the introduction of new viral strains, for example. Each intervention is identified uniquely by the the ``Name`` of the intervention. Additionally, each intervention requies an `activation condition`, and a `deactivation condition`. These conditions are ``boolean`` decisions.

Additionally, the user can further define an optional ``activationAction`` and a ``perTickAction``.

 * ``activationAction``: an action that is invoked whenever the intervention is activated.

 * ``perTickAction``: an action that is invoked for every tick for which the intervention is active.

There are four different intervention classes available in BharatSim


``Intervention``
~~~~~~~~~~~~~~~~

This is a generic intervention that can be invoked whenever the activation condition is met based on the parameters passed. This can be used to implement lockdowns during an epidemic every time the infected fraction of agent reaches a certain threshold, for example.

Five parameters can be passed ``Intervention`` object:

    * ``interventionName``: This is the unique name of the intervention.
    * ``activationCondition``: This function tells whether this intervention should be activated or not. The function should return a Boolean value.
    * ``deActivationCondition``: This function tells whether this intervention should be deactivated. The function should return a Boolean value.
    * ``firstTimeExecution``: This is an optional function that will be executed at the start of the intervention
    * ``whenActiveActionFunc``: This is an optional function  executed per tick when intervention is active.

Example using the ``Intervention`` object
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Here we will use the ``Intervention`` class to implement a two-week lockdown every time the number of infected agents are greater than or equal to 2000. First, we will define the intervention named addLockdown and initialise the variables.

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

Now we have to create an instance of the ``Intervention`` object and pass the values defined earlier it:

.. code-block:: scala

    val intervention =
    Intervention(interventionName, activationCondition, deActivationCondition, firstTimeExecution)

Next, we define a new schedule that has to come into effect once the intervention is activated. Here we define the ``lockdownSchedule`` such that all agent stays in the house throughout the day. The code below makes the agent stay home for the entire day as long as the intervention remains activated.

.. code-block:: scala

   val lockdownSchedule = (myDay, myTick).add[House](0, 1)

Now we have to register the register the intervention and the schedules using ``registerIntervention`` and ``registerSchedules`` respectively. We also have to pass the ``Agent`` and ``Context`` to ``registerSchedules``.

.. code-block:: scala
  
    registerIntervention(intervention)
    registerSchedules(
      (
        lockdownSchedule,
        (agent: Agent, context: Context)
      )
    )

The complete definition of our user-defined ``addLockdown`` is given below.

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

.. hint:: ``addLockdown`` should be called in the definition of the simulation.

  .. code-block:: scala

    simulation.defineSimulation(implicit context => {
    addLockdown
    }

``IntervalBasedIntervention``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This can be used to invoke an intervention that starts and end at specific ticks. This can be used for giving relaxations in the epidemic regulations for a specified period, for example, during a festival.

Five paramertes can be passed to the ``IntervalBasedIntervention`` object:

    * ``interventionName``: This is the unique intervention name.
    * ``startTick``: This integer specifies the start tick for intervention (inclusive); it should not be greater than endTick.
    * ``endTick``: This is an integer that specifies the end tick for the intervention (It is exclusive, and intervention will not be active at "endTick".)
    * ``firstTimeActionFunc``: This is an optional function which gets executed when simulation starts.
    * ``whenActiveActionFunc``: This is an optional function executed per tick when the simulation is active.

Example using the ``IntervalBasedIntervention`` object
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We will try to implement during the 50th to the 100th tick, i.e. for 25 days.

First we will define the variables ``interventionName``. In the function call of ``IntervalBasedIntervention()``, we will pass the ``interventionName``, ``startTick`` and ``endTick``.

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

The entire definition of ``addLockdown`` intervention using the ``IntervalBasedIntervention`` is given below:

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

``OffsetBasedIntervention``
~~~~~~~~~~~~~~~~~~~~~~~~~~~

This can be used to invoke interventions that end after 'n' ticks. It gets invoked when the ``shouldActivateWhen`` function is true.
This can be used to implement a lockdown when the number of infected agents reaches a particular threshold and stays active till n ticks.

Five parameters can be passed to the ``OffsetBasedIntervention`` object:

    * ``interventionName``: This is the unique name of the intervention.
    * ``shouldActivateWhen``: This function decides when should the intervention be activated.
    * ``endAfterNTicks``: This is the offset 'n'; simulation will end after n ticks from the star tick.
    * ``firstTimeActionFunc``:This is an optional function which gets executed when simulation starts.
    * ``whenActiveActionFunc``: This is an optional function executed per tick when the simulation is active.

Example using the ``OffsetBasedIntervention`` object
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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

.. hint:: As before, ``addLockdown`` should be called in the definition of the simulation.

  .. code-block:: scala

    simulation.defineSimulation(implicit context => {
    addLockdown
    }

``SingleInvocationIntervention``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This can be used to create an intervention that will be invoked only once in the simulation.

Five parameters can be passed to the ``SingleInvocationIntervention`` object:

    * ``interventionName``: This is the unique name of the intervention.
    * ``shouldActivateFunc``: This function tells whether this intervention should be activated.
    * ``shouldDeactivateFunc``: This function tells whether this intervention should be deactivated.
    * ``firstTimeActionFunc``: This is an optional function  executed at the start of the intervention.
    * ``whenActiveActionFunc``: This is an optional function  executed per tick when intervention is active.

Example using the ``SingleInvocationIntervention`` object
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
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