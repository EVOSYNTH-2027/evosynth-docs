# CMA-ES

Covariance Matrix Adaptation Evolution Strategy (CMA-ES) is widely considered the state-of-the-art derivative-free optimizer for continuous domains. In EvoSynth, it is the heavy-lifter for late-stage exploitation.

## Mechanics

Unlike GA or PSO, which track a population of distinct points, CMA-ES tracks a multivariate normal distribution $\mathcal{N}(m, \sigma^2 C)$.

At each generation, it:

1. Samples a new population of vectors from the distribution.
2. Evaluates their fitness.
3. Updates the mean $m$ to move toward the best individuals.
4. Updates the covariance matrix $C$ to stretch the distribution along the paths of highest gradient descent.

## Why it Dominates

By adapting the covariance matrix, CMA-ES effectively learns the contour map (the Hessian matrix) of the fitness landscape without ever computing a gradient. If two hyperparameters are highly correlated (e.g., higher `max_depth` requires lower `learning_rate` in XGBoost), the covariance matrix stretches diagonally, allowing the algorithm to sample precisely along that ridge.

## The Wrapper Strategy

EvoSynth utilizes the standard `cma` Python library under the hood. However, because the Meta-Engine demands a strict `ask()/tell()` asynchronous interface to allow the UCB1 Bandit to pause and resume algorithms at will, EvoSynth wraps the `cma.CMAEvolutionStrategy` object.

The wrapper handles boundary penalties and maintains the distribution state across Tier demotions and promotions seamlessly.
