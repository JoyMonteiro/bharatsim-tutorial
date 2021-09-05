BharatSim Documentation
========================

Welcome to this BharatSim Tutorial!

BharatSim is a collaborative project between `Ashoka University <https://www.ashoka.edu.in/>`_ and `Thoughtworks <https://www.thoughtworks.com/>`_, funded by the `Bill & Melinda Gates Foundation <https://www.gatesfoundation.org/>`_. Its vision is to build a distributed, multi-scale, agent-based model simulation framework for the scientific community, originally envisioned to help the scientific community track the spread of COVID -19 in India. Most real-world systems are complex and agent-based modeling allows researchers to create models with the required level of complexity at the level of individual agents in order to study a variety of different scenarios. These models are helpful in understanding emergent phenomena and would also help in testing different policy intervention scenarios. The framework has two components:

- **Simulation engine:** Given a :ref:`Synthetic Population`, the simulation engine can support India-wide simulations with multi-million agents, incorporating daily behaviours and other policy-level interventions. It uses network concepts such as nodes and relations to represent agents and their interactions, helping modellers to analyse the effect they have on the spread of the disease. The simulation engine is written in `Scala <https://en.wikipedia.org/wiki/Scala_(programming_language)>`_.

- **Visualization engine:** Given the customisable output that the simulatione engine can produce, the visualisation engine gives modellers the ability to  read the simulation output and chart it visually, as well as create different dashboards. Dashboards could have multiple kinds of charts simplifying the process of analysis, and crystallising insights.

In this tutorial documentation, we look be looking at BharatSim in detail, introducing the novice programmer to both components. The tutorial assumes a familiarity with (or at least an eagerness to learn) `Object-oriented Programming <https://en.wikipedia.org/wiki/Object-oriented_programming>`_. Prior familiarity with Scala or Java is desirable, but by no means a prerequisite.

.. toctree::
   :caption: Contents

   introduction

.. toctree::
   :caption: Epidemiology

   epidemiology
   
.. toctree::
   :caption: The Simulation Engine

   setup
   frameworkbasics
   firstprogram
   miscellaneous
