Writing your First Program
==========================

This section is a detailed guide for a novice user on how to build a `SIR Model <https://bharatsim.readthedocs.io/en/latest/epidemiology-sir-model.html>`_ in BharatSim from scratch. By the end of this section, you should be able to write and execute a SIR model by yourself.

Any model built on this framework contains various classes which are essentially extensions of different Nodes. To build a SIR model from scratch, one needs to define the said classes and the properties and relationships associated with them. In the basic SIR there are two main components:

1. :ref:`Agent and Behaviours`
2. :ref:`Network`

Agents are simply the individuals in the population and the Network is a class that defines a node that multiple agents frequent. Network classes can be used to model -- for example -- physical locations where different Agents come in contact with one another.

Here, we begin writing our program assuming that every agent is in contact with every other agent, which is equivalent to all of them being in the same "location".

.. toctree:: 
   :titlesonly:

   firstprogram-single-location-sir
   firstprogram-disease-dynamics
   firstprogram-expanding-network
   firstprogram-social-interventions
   firstprogram-FSM