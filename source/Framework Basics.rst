Framework Basics
=================

Interventions
-------------
Interventions can help you execute specific actions based on the current state of the simulation. The simulation engine of Bharat-Sim checks if an inactive intervention can be activated based on the given conditions during every tick. All the active interventions will be recorded in the context and the user can query the context using ``context.activeInterventionNames.contains(interventionName)``.

There are 4 different kinds of Interventions available in Bharat-Sim:

1.Intervention
~~~~~~~~~~~~~~~~~~~~~
This is a Generic intervention that can be invoked several times or once when the activation condition is met based on the parameters passed.
This can be used to implement lockdows during an epidemic everytime the infected fraction of agent reaches a certain threshold.
There are 5 parameters that can be passed ``Intervention`` object:

    * ``interventionName``: This is the unique name of the intervention.
    * ``shouldActivateFunc``: This function which tells whether this intervention should be activated or not. The function should return a Boolean value.
    * ``shouldDeactivateFunc``: This function which tells whether this intervention should be deactivated. The function should return a Boolean value.
    * ``firstTimeActionFunc``: This is an optional function which will be executed at the start of the intervention
    * ``whenActiveActionFunc``: This is an optional function which will be executed per tick when intervention is active.

Example using the ``Intervention`` object:
````````````````````````````````````````````````````
.. code-block:: scala

    //This implements a lockdown when the nuber of agents in the infected compartment is greater than or equal to 2000
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
        (agent: Agent, context: Context) => {
        val isEssentialWorker = agent.asInstanceOf[Person].isEssentialWorker
        val isLockdown = context.activeInterventionNames.contains(interventionName)
        isLockdown && !isEssentialWorker// && agent.asInstanceOf[Person].isInfected
        },
        1
    )
    )
    }


2.IntervalBasedIntervention
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This can be used to invoke an intervention which starts and end at specific ticks. This can be used for giving relaxations in the epidemic regulations for a specified period of time, for example during a festival.

There are 5 paramertes that can be passed to the ``IntervalBasedIntervention`` object:

    * ``interventionName``: This is the unique intervention name.
    * ``startTick``: This ia an integer that specifies the start tick for intervention (inclusive); it should not be greater than endTick.
    * ``endTick``: This is an integer that specifies the end tick for the intervention (exclusive, intervention will not be active at "endTick".)
    * ``firstTimeActionFunc``: This is an optional function which gets executed when simulation starts.
    * ``whenActiveActionFunc``: This is an optional function which gets executed per tick when simulation is active.

Example suing the ``IntervalBasedIntervention`` object:
``````````````````````````````````````````````````````````````````

.. code-block:: scala

    //This implements a lockdown from the 20th to the 530th tick

    private def addLockdown(implicit context: Context): Unit = {

    val interventionName = "lockdown"
    val intervention = IntervalBasedIntervention(interventionName, 20, 530)

    val lockdownSchedule = (Day, Hour).add[House](0, 23)

    registerIntervention(intervention)
    registerSchedules(
    (

        lockdownSchedule,
        (agent: Agent, context: Context) => {
        val isEssentialWorker = agent.asInstanceOf[Person].isEssentialWorker
        val violateLockdown = agent.asInstanceOf[Person].violateLockdown
        val isLockdown = context.activeInterventionNames.contains(interventionName)
        val isSeverelyInfected = agent.asInstanceOf[Person].isSevereInfected
        isLockdown && !(isEssentialWorker || violateLockdown || isSeverelyInfected)
        },
        1

    )
    )
    }

3.OffsetBasedIntervention
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This can be used to invoke interventions that end after 'n' ticks. It gets invoked when the ``shouldActivateWhen`` function is true.
This can be used 

There are 5 parameters that can be passed to the ``OffsetBasedIntervention`` object:

    * ``interventionName``: This is the unique name of the intervention.
    * ``shouldActivateWhen``: This function decides when should the intervention be activated.
    * ``endAfterNTicks``: This is the offset 'n'; after n ticks from the start tick simulation will end.
    * ``firstTimeActionFunc``:This is an optional function which gets executed when simulation starts.
    * ``whenActiveActionFunc``: This is anoptional function which gets executed per tick when simulation is active.

4.SingleInvocationIntervention
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This can be used to create an intervention that will be invoked only once in the simulation.

There are 5 parameters that can be passed to the ``SingleInvocationIntervention`` object:

    * ``interventionName``: This is the unique name of the intervention.
    * ``shouldActivateFunc``: This function tells whether this intervention should be activated.
    * ``shouldDeactivateFunc``: This function tells whether this intervention should be deactivated.
    * ``firstTimeActionFunc``: This is an optional function which will be executed at the start of the intervention.
    * ``whenActiveActionFunc``: This is an optional function which will be executed per tick when intervention is active.

Example suing the ``SingleInvocationIntervention`` object:
``````````````````````````````````````````````````````````````````

.. code-block:: scala

    \\This starts a vaccination drive when there is a lockdown and continues till all the ingested population is vaccinated
    private def vaccination(implicit context: Context): Unit = {
    var interventionActivatedAt = 0
    val interventionName = "vaccination"
    val activationCondition = (context: Context) => {
    val result = context.activeInterventionNames.contains("lockdown")
    if (result) {
        vaccinationStarted = context.getCurrentStep
    }
    result
    }
    val firstTimeExecution = (context: Context) => interventionActivatedAt = context.getCurrentStep
    val deActivationCondition = (context: Context) => {
    vaccinesAdministered.get() >= ingestedPopulation
    }

    val intervention: Intervention = SingleInvocationIntervention(
    interventionName,
    activationCondition,
    deActivationCondition,
    firstTimeExecution,
    context => {
        val populationIterator: Iterator[GraphNode] = context.graphProvider.fetchNodes("Person")
        val numberOfVaccinesPerTick = Disease.vaccinationRate * ingestedPopulation * Disease.dt

        StreamUtil
        .create(populationIterator, parallel = true)
        .filter((node) => node.as[Person].shouldGetVaccine())
        .limit(numberOfVaccinesPerTick.toLong)
        .forEach(node => {
            val person = node.as[Person]

            person.updateParams(
            ("vaccinationStatus", true),
            ("betaMultiplier", person.betaMultiplier * Disease.vaccinatedBetaMultiplier),
            ("gamma", person.gamma * (1 + Disease.vaccinatedGammaFractionalIncrease))
            )
            vaccinesAdministered.getAndIncrement()
        })
    }
    )

    registerIntervention(intervention)
    }
