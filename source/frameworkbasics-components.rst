Components of the Simulation Engine
===================================

The following sections will explain the basic components and terminology of the BharatSim framework, using a simple but concrete example of a SIR model. Going through the instructions given in these sections, you should be able to write your own SIR model with agents, introduce network structures like homes and workplaces and allow agents to move between them, and define different disease compartments representing different disease states and transition agents between them.


Graphs, Nodes, and Relations
----------------------------

.. figure:: _static/images/FSM_network.png
  :align: center
  :width: 900px
  :figclass: align-center

  A sample Graph consisting of nodes such as Person, Offices and Houses which are connected by relations, and forming a Network.



The basic data structure in which BharatSim stores its data is a `Graph`. This `Graph` is composed of multiple ``Nodes``, each specified by a unique 64-bit id. Each of these ``Nodes`` can be connected to every other through a ``Relation``. Only a single ``Relation`` may exist between two ``Nodes``.

.. error::
  If multiple relations are defined between two ``Nodes``, then the simulation will not know which ``Relation`` to pick, and hence will throw an error.


Agents and network locations are both modelled as extensions of the ``Node`` class, while ``Relations`` are defined between them to create an underlying network structure. The simulation engine defines multiple functions that allow the modeler to create, access, update, and delete the ``Nodes`` and their ``Relations``. We will describe them in more detail below.


.. figure:: _static/images/FSM_class.png
  :align: center
  :width: 900px
  :figclass: align-center

  A modeler can define different extensions of the ``Node`` class to represent, for example, a ``Person``, a ``Home``, or a ``Work`` location.


Agents and Behaviours
---------------------

In Agent-Based Modelling, a system generally consists of a group of automatons that make decisions at every time-step, based on data from each other and the environment. These automata are called "agents". In BharatSim, agents can be modelled using the framework-defined ``Agent`` class, which is an extension of the ``Node`` class. To allow for heterogeneity present in real-world individuals, different instances of the ``Agent`` class can possess different user-defined attributes, like their age, occupation, vaccination status, and so on. Every class of agent will have to be registered in the ``Simulation`` using the framework-defined ``registerAgent`` function.

At each time step, an ``Agent`` is allowed to execute an action, known as a behaviour. The ``Agent`` class thus has a framework-defined ``addBehaviour`` function that can be used to execute an action at every time-step for every agent.

These behaviours can be modelled depending on the situation the agent finds itself in. For example, in the case of disease-modelling, one might use a behaviour to decide if an unvaccinated agent will get vaccinated on a specific day, based on  the result of a daily coin-toss.

In this way, the ``Agent`` with their behaviours mimics the actions of real people in a population. Depending on the level of heterogeneity introduced in the population by the modeller, these behaviours can be modelled as close to real-world actions as possible.

.. figure:: _static/images/FSM_Person.png
  :align: center
  :width: 900px
  :figclass: align-center

  Agents have can have custom Schedules and they can be made to execute actions through addBehaviour function.


Networks
--------

The ``Network`` class is another framework-defined extension of the ``Node`` class which can be used to model physical locations or contact-networks in a simulation.

In addition to the standard functions that the ``Node`` class provides, the ``Network`` class has a ``getContactProbability`` function which allows the programmer to model differential disease transmission in different network locations. For example, a crowded public-transport location might lead to a much higher probability of transmission of an infectious disease, when compared to an open office with very few employees.

The ``Network`` call can be extended by the modeller to describe different contact networks, such as homes, workplaces, and schools, for example. This thus allows multiple agents to begin interacting with each other. The interactions are governed using the ``addRelation`` function that establishes a relation between any two nodes. In this case, the nodes would be the ``Agent`` and the specific extension of the ``Network`` class. To illustrate the point, consider a simple model in which we define three types of network locations: a ``School``, an ``Office``, and a ``House``. Every agent in the population is assigned one of these, with multiple agents being assigned the same home, workplace, and school, based on real-world data. These agents are connected to these locations using user-defined relations. For example, we could say that a ``Person`` ``IS_EMPLOYED_BY`` a specific ``Office``, and that the ``Office`` is the ``EMPLOYS`` the ``Person``. Thus, the relations ``IS_EMPLOYED_BY`` and ``EMPLOYS`` connect specific ``Person`` and ``Office`` classes.

In this every ``Agent`` has a ``House`` with a unique ``id``, and therefore a contact network associated with their family -- i.e., the other agents who have been assigned the same ``House``. Similarly, this agent and all other agents who share the same ``Office`` ``id`` are assumed to work together, forming a professional network. In the next section, we will see how these agents can be made to spend different amount of times with these different networks, and how this can lead to more complex dynamics in the population.

.. figure:: _static/images/FSM_relations.png
  :align: center
  :width: 900px
  :figclass: align-center

  Illustration of a bidirectional relationship between two nodes.




Schedules
---------

In order to account for the movement of individuals between different networks, the BharatSim framework implements a ``Schedule`` which associates an ``Agent`` to a specific ``Network``. These schedules allow the modeller to decide what fraction of a unit of time (say, a day of 24 hours) the agent spends in each location. Each agent is assumed to follow this schedule, and this allows them to move between network locations over time, governed by their ``Schedule``.

Different agents can be given different schedules, based on their attributes or other factors. For example, if we define all individuals under the age of 18 to be schoolchildren and all those above the age of 18 to be working adults, we could assign all children to schools and all adults to workplaces, and give them different schedules based on this distinction. Taking another example, a modeller could assign agents an attribute ``is_employed`` which is set to ``true`` if the agent is employed. We could then define different schedules, an "employed" schedule and an "unemployed" schedule, requiring agents to follow the appropriate schedule based on their employment status. The condition for assigning an ``Agent`` a specific ``Schedule`` can thus be made as general or specific as required.

Additionally, the same agent can be assigned multiple schedules. Which schedule the agent follows at any given simulation tick is dictated by a "priority" parameter that is set when the ``Schedule`` is defined. With all else being the same, the ``Schedule`` with the higher priority is given precedence.


.. figure:: _static/images/FSM_customSch.png
  :align: center
  :width: 900px
  :figclass: align-center

  Different types of Persons can have their own schedule based on age, jobs, and socio-economic status.


Finite State Machine
--------------------

A Finite State Machine is a class of algorithms where an abstract machine can be in exactly one of the finite state at any given time. A "state" is defined as the explicit trait of the system and this can be changed after satisfying a said boolean condition. This change from one state to another state is called a transition, and the criteria for a transition between two different pair of states may vary based on modeller choices. This is best illustrated through an example of traffic lights.

.. list-table:: Traffic Lights
   :align: center
   :widths: 25 25 30
   :header-rows: 1

   * - Current State
     - Next State
     - Condition
   * - Green
     - Yellow
     - 120 seconds
   * - Yellow
     - Red
     - 20 seconds
   * - Red
     - Green
     - 120 seconds

The above table lists the "state" the system can be in and the possible transition condition that needs to be satisfied. Suppose the system just entered the ``Green`` state, then this implies that are after spending 120 seconds being ``Green``, the system will transition to the ``Yellow`` state.

BharatSim possesses a framework-defined Finite State Machine which allows users to define a ``State`` as an extension of the `Scala` programming language's `trait <https://docs.scala-lang.org/tour/traits.html>`_ datatype. Every distinct state in the model can be created as user-defined extensions of the ``State`` trait. These user-defined extensions can further allow for transitions between different states using the framework-defined ``addTransition`` function that every ``State``` possesses.

Every type of ``State`` should be registered in the ``Simulation`` using the framework-defined ``registerState`` function.



Stateful Agent
^^^^^^^^^^^^^^

In order to make use of the Finite State Machine, BharatSim defines an extension of the ``Agent`` class called the ``StatefulAgent`` class. Such agents possess an ``activeState`` which links them to one -- and only one -- instance of a ``State`` trait. Additionally, a ``StatefulAgent`` also possesses two important functions:

1. ``setInitialState``: which sets the initial state of a ``StatefulAgent``, and
2. ``fetchActiveState``: which returns the current state of the ``StatefulAgent``.

Like agents, a ``StatefulAgent`` also needs to be registered in the ``Simulation`` using the ``registerAgent`` function.

Actions
^^^^^^^

Additionally, in BharatSim, the ``State`` trait also posses certain "actions" that -- like behaviours -- are executed at every tick by every ``StatefulAgent`` associated with that ``State``.

1. ``enterAction``: is an action a ``StatefulAgent`` executes the moment they transition into a ``State`` for the first time.
2. ``perTickAction``: is an action executed by a ``StatefulAgent`` on every day that they are associated with that ``State``.

Additionally, similar to how agents have behaviours, states have "transitions". The ``State`` trait also allows for an ``addTransition`` function that can be used to check at every time-step if each ``StatefulAgent``  associated with that ``State`` is allowed to transition to any other state.
