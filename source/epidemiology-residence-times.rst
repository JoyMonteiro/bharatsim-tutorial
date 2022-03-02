Residence times in different compartments
=========================================

How long does an individual spend in an infected state or compartment? This duration, also known as residence time, isn’t a fixed value, so it’s important to know the distribution of residence times. In Markov-Chain-Monte-Carlo simulations like ours, the residence times are exponentially distributed. First we will explain why the distribution is exponential, which can allow us to change our algorithm for non-exponential cases.

We discretise time into small intervals or time-steps of $\Delta t$, so that the total time $T = N \Delta t$, where $N$ is the total number of time-steps. In every small step $\Delta t$ an individual in our disease model has a small probability, $p$, of exiting the compartment. The value of $p$ depends on factors such as the number of individuals in that compartment at that time, and so on.

To simulate $p$, we draw a random number $r$ from a uniform distribution. If $r < p$, the individual exits the compartment. This is equivalent to saying: $$\begin{equation}\texttt{The probability of exiting the compartment is p}.\end{equation}$$

This is the only role of the uniform distribution here.

Now, *what is the probability that an individual will leave a compartment at some time $t$?* In our scheme, this is equivalent to asking, what is the probability of the event occurring between $t$ and $t + \Delta t$. We will call this probability $P(t)\Delta t$. We’ve also discretized time into units of $\Delta t$, so we can define $t = n\Delta t$, where $n$ is the number of steps of $\Delta t$.

This allows us to reframe the problem as, *what is the probability that the individual will leave the compartment after exactly $n$ steps?* This is given by:

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
