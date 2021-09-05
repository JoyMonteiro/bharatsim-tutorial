
Introducing a network structure
===============================

We can now add an additional layer of complexity, by introducing
relations between the agents through their home and work locations.

.. figure:: _static/images/epidemiology_network_schematic.png
    :align: center
    :alt: A schematic of a model network
    :width: 600px
    :figclass: align-center

In such a model, individuals can move between their home and their work
locations, and as a result the total number of individuals in any given
location is not fixed. As a result, we now need to go over every
location, and treat it as a well-mixed location. The algorithm now
becomes:

.. admonition:: Algorithm
   :class: tip

    1.  Divide the total time into steps of $\Delta t$, and at every
        time-step we loop over all agents.
    2.  If the agent is susceptible, we compute the number of infected
        individuals who could potentially infect them ("$I$") in their
        current location. Then, with some probability $$p_\text{SI} =
        \lambda_S\frac{I}{N}\Delta t,$$ we transition them to the
        infected compartment.
    3.  If the agent is already infected, we transition them to the
        recovered compartment with a probability $$p_\text{IR} =
        \lambda_I\,\Delta t.$$
    4.  If they have recovered, do nothing.
    5.  Repeat the entire process until there are no more infected
        individuals, or the total time has elapsed.
