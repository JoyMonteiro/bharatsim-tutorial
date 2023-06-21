Writing your First Program
==========================

This section is a detailed guide for a novice user on how to build a `SIR Model <https://bharatsim.readthedocs.io/en/latest/epidemiology-sir-model.html>`_ in BharatSim from scratch. Going through the instructions given in these sections, you should be able to write your own SIR model with agents, introduce network structures like homes and workplaces and allow agents to move between them, and define different disease compartments representing different disease states and transition agents between them.

Any model built on BharatSim contains various classes which are essentially different extensions of the ``Node`` class. To build a SIR model from scratch, one needs to define these classes and the properties and relationships associated with them. In what follows, we will be modelling the individuals of our population as extensions of the ``Agent`` class, and the locations that they frequent as extensions of the ``Network`` class.

.. note::
   Before we move on, make sure you have gone through the sections on :ref:`Agents and Behaviours` and :ref:`Networks`, since certain ideas in the sections below are explained in greater detail there.

Different agents come in contact with each other based on the underlying contact network structure of the population. For simplicity, we begin our model assuming that every agent is in contact with every other agent, which is equivalent to all of them being in the same "location". However, we will relax this very quickly to account for a more realistic contact network strucure. Finally, we will describe BharatSim's Finite State Machine (FSM) and rewrite our model using it, with the help of the framework-defined `StatefulAgent` class.

.. toctree:: 
   :titlesonly:

   firstprogram-single-location-sir
   firstprogram-disease-dynamics
   firstprogram-expanding-network
   firstprogram-social-interventions
   firstprogram-FSM