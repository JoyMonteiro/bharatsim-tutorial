Residence times in different compartments
=========================================

An interesting question to ask at this point is this: what is the probability distribution of times that individuals spend in the infected compartment? It turns out that in standard Markov-Chain Monte-Carlo simulations like the ones we study, the residence times are exponentially distributed. Let us try to explore why this is the case, and see if we can change our earlier algorithm to generalise to non-exponential waiting times.

We are discretising our system into little intervals of time-step $\Delta t$, so that the total time $T= N \Delta t$, where $N$ is the total number of steps. In our disease modelling simulations, we assume that at every step $\Delta t$, an individual has some probability of exiting the compartment they are in, which we will call $p$. (This value of $p$ will depend on many factors, including the number of individuals in that compartment at that time, and so on.)

In order to simulate a probability of $p$, we draw a random number $r$ from a uniform distribution and make the shift if $r<p$. This is just equivalent to saying: $$\begin{equation}\texttt{Make the shift with a probability p}.\end{equation}$$

This is the only role of the uniform distribution here.

Now, let's ask ourselves the following question: *what is the probability that an individual will leave a compartment at some time $t$?* We will call this probability $$\mathcal{P}\dd t,$$ since it's a probability distribution, and so we can't speak of something occuring exactly at time $t$, but rather in an interval of time between $t$ and $t+\dd t$.

In our scheme, $t= n \Delta t$, and so the question is equivalent to asking: what is the probability that the individual will leave exactly after $n$ steps. This is:

$$\text{Prob. that event occurs exactly after $n$ steps} = (1-p)^n p.$$

Using the fact that $p = \lambda \Delta t$, and that $\Delta t = T/N$, we can show that:
$$
\begin{align}
\mathcal{P}_{\Delta t}(t)\dd t &= (1 - \lambda \Delta t)^{t/\Delta t} \lambda \Delta t\\
&= \left(1 - \frac{\lambda T}{N} \right)^{t N/T} \lambda \Delta t
\end{align}
$$

To find the true probability density, we need to take the limit $N\to\infty$, i.e. $N/T \to \infty$, and so :

$$\mathcal{P}(t)\dd t = \lim_{N/T \to\infty} \left(1 - \frac{\lambda T}{N} \right)^{t N/T} \lambda \Delta t = \lambda e^{-\lambda t} \dd t, $$

so that the probability distribution is: $$\mathcal{P}(t) = \lambda e^{-\lambda t}.$$

.. admonition:: Exercise
   :class: error

    1.  Show that the probability that an individual exits the compartment
        \$I\$ exactly after some time \$t = n \\Delta t\$ is given by:
        \$\$\\mathcal{P}(t)\\,\\Delta t = p\_\\text{IR}
        (1-p\_\\text{IR})\^n.\$\$
    2.  Using the fact that \$p=\\lambda\_I\\,\\Delta t\$, and \$\\Delta t =
        T/N\_\\text{steps}\$ (Where \$T\$ is the total simulation time and
        \$N\_\\text{steps}\$ the total number of steps):
        \$\$\\mathcal{P}(t)\\,\\Delta t = \\left(1 - \\frac{\\lambda\_I
        T}{N\_\\text{steps}}\\right)\^{t N\_\\text{steps}/T}\\, \\lambda\_I
        \\, \\Delta t.\$\$
    3.  Next, take the limit \$N\_\\text{steps}\\to\\infty\$, and \$\\Delta
        t\\to 0\$, keeping \$N\_\\text{steps}\\,\\Delta t = T\$, and argue
        that \$\$\\mathcal{P}(t) = \\lambda\_I \\, e\^{-\\lambda\_I t}.\$\$
        The residence times in the infected compartment are exponentially
        distributed, with mean \$\\tau\_I = 1/\\lambda\_I\$! In other words,
        this is a succession of Poisson processes -- characteristic of
        Markov Chain Monte Carlo processes: happens every time the
        probability of a transition is independent of the history.
    4.  Can the same argument be used to say that the residence time in the
        susceptible compartment is \$\\tau\_S = 1/\\lambda\_S\$? Explain.
