The SIR Model
-------------

.. figure:: _static/images/epidemiology_SIR_disease_progression.png
    :align: center
    :alt: The disease progression in the SIR Model
    :figclass: align-center

The SIR model is so named because in it individuals can be in one of
three compartments, (S)usceptible, (I)nfected, and (R)ecovered, at any
given time. Infected individuals can "infect" the Susceptible, causing
them to transition to the Infected compartment, from which they can
eventually recover. In what follows we will use the letters $S$,
$I$, and $R$ to represent both the individual compartments as well
as the numbers in each of these compartments. The context should make it
clear what we are referring to. In addition, we make some assumptions:

1.  Susceptible individuals can only leave this compartment if they have
    been infected. (We ignore birth and death rates.)
2.  The rate of change of $S(t)$, the number of susceptibles, depends
    on the number of individuals currently susceptible, the number
    currently infected, and the amount of contact between susceptibles
    and infecteds.
3.  We also assume that some fixed fraction (say, $\gamma$) of the
    infected group will recover on any given day. Keep in mind that when
    we say someone has "Recovered", what we mean (in this model) is that
    they are no longer infectious or susceptible to the disease and
    therefore cannot contribute to its spread.

Mean-field or "Well-mixed" solution to this model
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This system is modelled by a set of coupled non-linear first-order
differential equations that should remind some of you of the "rate"
equations you've seen in Chemistry courses.
$$
\begin{equation}
\begin{aligned} \dv{S}{t} &= - \beta\, S(t)\, I(t)\\[10pt]
\dv{I}{t} &= + \beta \, S(t) \, I(t) - \gamma \, I(t)\\[10pt]
\dv{R}{t} &= + \gamma \, I(t) \end{aligned} \tag{1} \label{1}
\end{equation}
$$

.. admonition:: Exercise
   :class: error

    1.  Why is there a negative sign in the equation for $S(t)$?
    2.  Convince yourself that $\beta$ represents the average number of
        contacts per unit time between an infected individual and a
        susceptible individual.
    3.  Show that the total population $N = S+I+R$ is fixed (i.e. that
        this is a **closed system**). This is often a very good check to see
        if your simulation is running correctly!

On some inspection, you should be able to realise that parameter
$\beta$ is a little different from the parameter $\gamma$.
$\gamma$ is quite simply the fraction of infected individuals who
will recover on any given day. $\beta$, on the other hand multiplies
a non-linear term, and represents the probability that an infected
individual comes in contact with a susceptible individual. As a result,
it is actually a composite object $$\text{Transmission coefficient
}\beta = \text{Contact probability } \times \text{ Transmission
rate},$$ in other words each infected individual meets some fraction
of the total number of susceptibles and infects them at some
"transmission rate". Now, this "contact probability" decides how the
transmission of a disease scales up with the population. Two specific
types of "scaling" can be chosen, depending on the type of disease.

1.  **Contact probability = 1.** In this case, we assume that infected
    individuals meet all the susceptibles in one day. Therefore, the
    larger the population, the faster the spread of disease. This can be
    used to model a large number of people crowded in an enclosed space.
    Such a scaling is often appropriate in the case of plant and animal
    diseases. This is known as *density dependent scaling*.

2.  **Contact probability = $\\mathbf{1/N}$.** In this case, we assume
    the infected individual meets a constant number of people (say
    $\lambda_S$) every day. Of these, the fraction of susceptibles
    is $S/N$. This assumption holds for most human diseases, where
    contact is determined by social factors. This is known as *frequency dependent scaling*.

The distinction between these two types of scaling only occurs when the
total population size is not fixed, since otherwise the factor of $N$
could simply be absorbed into the definition of $\beta$. In what
follows, we will be using **frequency dependent scaling**. As a result,
Equation (\ref{1}) above can be rewritten as: $$\begin{equation}
\begin{aligned} \dv{S}{t} &= - \frac{\lambda_S}{N} S I\\[10pt]
\dv{I}{t} &= + \frac{\lambda_S}{N} S I - \lambda_I I\\[10pt]
\dv{R}{t} &= + \lambda_I I \end{aligned} \tag{2} \label{2}
\end{equation}$$

.. admonition:: Exercise
   :class: error

    1.  Show that, because of the extra factor of $N$, the solutions to
        this differential equation will be *independent* of $N$. (**Hint:**
        Rewrite the equations in terms of $s(t) = S(t)/N$, $i(t) =
        I(t)/N$, and $r(t) = R(t)/N$.)
    2.  It turns out that finding the exact solution to this equation as a
        function of time is not possible. However, an implicit solution can be found. By dividing the first and third equations, show that $$s(t) =
        s(0)\,e^{-R_0 r(t)},$$ where the quantity $R_0$ is called
        the **reproductive ratio**. Show that $R_0$ is independent of
        $N$.
    3.  Now, repeat the previous question for the system denoted by Equation
        (\ref{1}). Show that in that case, $R_0$ depends on the total
        population $N$.

The above differential equations represented in Equation (\\ref{2}) can
be easily solved numerically to obtain solutions that look like this:

.. figure:: _static/images/epidemiology_SIR_deterministic.png
    :align: center
    :alt: The deterministic solution to the differential equations of the SIR Model
    :width: 600px
    :figclass: align-center

Of course, these solutions are **deterministic**. The reason for this is
that we assume that a fixed number of individuals transition every day
between compartments, which in turn is a result of using a **well-mixed
population**.

.. note::
  A well-mixed population is one in which any infected
  individual has a probability of contacting any susceptible individual
  that can be approximated reasonably well by the **average** probability
  of susceptible-infected interaction. This is often the most problematic
  assumption, but is easily relaxed in more complex models.

Stochastic solutions to this model
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

But what happens when you cannot approximate the entire population to be
"well-mixed"? In particular, the population could comprise many
individuals who move between their homes and work-places and other
locations. In this case, the number of contacts that an individual agent
has would not necessarily be comparable to those of another agent. For
example, some agents might work in high-density workplaces and come in
contact with many more individuals and spread the disease at a faster
rate than others. Furthermore, we might want to include a heterogeneity
in the population: some agents might intrinsically be more likely to get
infected than others. And lastly we might also want to model different
interventions like a lockdown where only certain agents are allowed to
move, and not others.

For all of the above cases, the well-mixed system is a very bad
approximation, since it assumes that all individuals are identical and indistinguishable. As a result, we need to look at a different approach. One
way to do this is to go down to the level of the individual agents, and
incorporate as much heterogeneity as we want at this level, and then
imagine that these agents move about an infect each other at constant
rates due to their contacts with each other.

This is a far more resource-intensive process, as we now have to keep
track of each individual agent, what their attributes are, and who they
can infect. However, the types of questions that such models can answer
are much more extensive.

The most famous method to solve such systems is known as the `Gillespie
Algorithm <https://en.wikipedia.org/wiki/Gillespie_algorithm>`_. We will
be using a much simpler discrete time approximation of this method.
Below we will start off by considering all the individuals in a single
location, so that everyone is in contact with everyone else. However, in
the next section, we will relax this assumption and allow for networks
of individuals to be formed. The basic idea is as follows:

.. admonition:: Algorithm
   :class: tip

    1.  Divide the total time into steps of \$\\Delta t\$, and at every
        time-step we loop over all agents.
    2.  If the agent is susceptible, we compute the number of infected
        individuals who could potentially infect them ("\$I\$"). Then, with
        some probability \$\$p\_\\text{SI} = \\lambda\_S\\frac{I}{N}\\Delta
        t,\$\$ we transition them to the infected compartment.
    3.  If the agent is already infected, we transition them to the
        recovered compartment with a probability \$\$p\_\\text{IR} =
        \\lambda\_I\\,\\Delta t.\$\$
    4.  If they have recovered, do nothing.
    5.  Repeat the entire process until there are no more infected
        individuals, or the total time has elapsed.

The results of such a stochastic simulation can be seen in the figure
below. There are faint curves represent a large number of stochastic
realisations of the same process, given the same initial conditions. As
you can see, the progress of the disease is no longer deterministic.
However (if all agents are in a single location and therefore
interacting with each other) the average over all of these stochastic
runs will result in the "well-mixed" result. This could serve as another
potential check to your code.

.. figure:: _static/images/epidemiology_SIR_stochastic.png
    :align: center
    :alt: The stochastic solutions to the differential equations of the SIR Model
    :width: 600px
    :figclass: align-center
