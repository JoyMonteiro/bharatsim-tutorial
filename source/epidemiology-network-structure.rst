
Introducing a network structure
===============================

Consider the semi-realistic scenario where agents can move between different locations like “work” and “home”. We can build a network that accounts for the relationship between agents and locations over time.

.. figure:: _static/images/epidemiology_network_schematic.png
    :align: center
    :alt: A schematic of a model network
    :width: 600px
    :figclass: align-center

By adding this complexity, we can see that the number of individuals at any given location is not fixed. So we modify our algorithm such that we treat each location as a well-mixed case looping over the instantaneous population at the given location. The modified algorithm is shown in the box below. 

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
