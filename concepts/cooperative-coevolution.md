# Cooperative Co-evolution

If four algorithms are running at the same time, it would be inefficient for them to be completely isolated. If the Genetic Algorithm finds an incredibly good configuration, the Particle Swarm should know about it.

EvoSynth implements an **Island Model with Elite Migration**.

## The Tier System

The algorithms are physically divided into two groups:
- **Tier 2 (The Active Workers):** The algorithms that currently have the highest composite score. The majority of the computational budget is funneled here.
- **Tier 1 (The Explorers):** The underperforming algorithms. They receive a much smaller trickle of budget to continue background exploration.

## Elite Sharing

Every $N$ iterations, a Migration event occurs.

1. The Meta-Engine pauses all algorithms.
2. It requests the *Best Individual (Elite)* from every algorithm across all Tiers.
3. It takes the absolute global best Elite, and forcibly injects it into the population of the underperforming algorithms.

### Why do this?
By injecting the global elite into a struggling algorithm, you "warm seed" it. 
For example, if CMA-ES is struggling to find the right basin of attraction, injecting the elite from GA immediately pulls the CMA-ES covariance matrix towards a highly profitable area of the latent space, massively accelerating convergence.
