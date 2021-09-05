Exercises
=========

.. admonition:: Basic Exercises
  :class: error

   1.  Generalise the simple SIR model to an SEIR model, where the
       susceptibles move into an "exposed" compartment for some time before
       they start being infectious. In this compartment, they cannot infect
       others, nor can they get infected.
   2.  Generalise your SIR model to an SIRD model, where some fraction of
       the infected individuals (say, 1%) transit to the "Dead" compartment
       instead of recovering.
   3.  Generalise the simple SIR model to an SAIR model: infected
       individuals can either be Asymptomatic ($A$) or Symptomatic
       ($I$), and they can both recover from this state. You will need to
       define new parameters that determine the relative **fraction** of
       asymptomatics, for example.
   4.  For the simple SIR model (or any other model that you prefer),
       compute out the "residence time" distribution for the
       **susceptible** compartment. In other words, find all the times at
       which individual agents transition from $S\to I$, and plot a
       histogram.

.. admonition:: Intermediate Exercises
  :class: error

   1.  How does our simple SIR model scale up with the population $N$?
       You will need to study not only the epidemic curves, but also their
       **spread**. Run the SIR model for around 10 or 20 times for a given
       population size (say, 10,000 individuals), and record the output.
       Repeat the process for a population of 20,000, and so on, for as
       high as you can go. Next, find some way to quantify the **spread**
       of the epidemic curves (for example, the standard deviation in the
       number of recovered at the end of the epidemic). Plot a graph that
       shows how this "spread" varies with the initial population $N$.
   2.  Generalise the simple SIR model to include reinfections (the SIRS
       model): the recovered people do not stay recovered but -- at some
       rate $\zeta$ -- transit back into the susceptible compartment.
       Try to choose the parameters well such that an **endemic
       equilibrium** is reached, meaning that the disease never truly dies
       out, some small fraction of the population is always infected.
   3.  Introduce a lockdown to a simple SIR model. Say that only some
       fraction of the population (say, only the essential workers) will
       "violate" the lockdown and continue to go to work. Observe what
       happens to the total number of recovered at the end of the epidemic
       as you increase the duration of the lockdown.

.. admonition:: Advanced Exercises
  :class: error

   1.  Consider a simple model with multiple types of infected individuals
       (say, the SAIR model I described above). Introduce the possibility
       that one infected group is much more infectious than another
       infected group. In other words, say that Symptomatic individuals
       ("I") are 2 times more infectious than Asymptomatic individuals.
   2.  For the simple SIR model, in one of the earlier exercises we showed
       that the time that an individual spent in the infected compartment
       was an exponentially distributed Random Number with mean
       $1/\lambda_I$. Therefore, there should be another way to decide
       how long infected individuals stay in a compartment, instead of
       generating a probability of transitioning at every step $\Delta
       t$. When an individual transits from $S\to I$, choose a fixed
       amount of time randomly drawn from an exponential distribution of
       mean $\tau_I = 1/\lambda_I$. After this time, the individuals
       will exit from $I\to R$.
   3.  Compute the effective reproductive ratio ($R$) as a function of
       time. This number is defined as the average number of individuals a
       single infected individual is responsible for having infected. In
       order to do this, whenever an $S\to R$ transition occurs, choose
       one of the infected individuals who could be responsible for this an
       increment some parameter representing the the "number of people they
       have infected". At every time step, average this number out over the
       entire population, and finally plot a graph of this as a function of
       time.
