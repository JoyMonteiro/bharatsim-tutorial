BharatSim Documentation
========================

Welcome to this BharatSim Tutorial!

BharatSim is a collaborative project between `Ashoka University <https://www.ashoka.edu.in/>`_ and `Thoughtworks <https://www.thoughtworks.com/>`_, funded by the `Bill & Melinda Gates Foundation <https://www.gatesfoundation.org/>`_. Its vision is to build a simulation framework that is distributed, multi-scale, and agent-based for use by the scientific community. It was originally designed to run decision-critical scenarios for India during the COVID-19 pandemic. Real-world systems involve interactions between individuals with different attributes (age, weight, etc.) and geographies. These interactions lead to emergent phenomena, while events like pandemics affect individuals according to their attributes. Existing simple models are ill-suited for prediction and analysis of such complex real-world phenomena. Agent-based modeling accounts for individual differences and allows us to simulate scenarios of varying complexity. These simulations can guide policy level interventions (eg. lockdowns). The framework has two components:


- **Simulation engine:** Given a :ref:`Synthetic Population`, simulation engine can support India-wide simulations with multi-million agents, incorporating daily behaviours and policy-level interventions. It structures this information by treating agents, locations, and their relations like nodes and edges on a network. This way modellers can analyse how population structure affects the spread of the disease. The simulation engine is written in `Scala <https://en.wikipedia.org/wiki/Scala_(programming_language)>`_.

- **Visualization engine:** A key feature of BharatSim is the dashboard visualisation that allows you to view multiple types of results at the same time. The simulation engine can generate customizable outputs, which the visualisation engine allows you to plot and arrange for viewing and presentation. This customizable dashboard system is flexible in order to suit the user’s analytical and insight formation process.


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
   Optimization
   otherexamples
