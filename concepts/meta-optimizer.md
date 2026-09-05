# The UCB1 Meta-Optimizer

Traditional hyperparameter tuning forces you to choose a single algorithm. EvoSynth treats the choice of algorithm itself as an optimization problem.

To solve this, EvoSynth frames the active algorithms as "slot machines" in a **Multi-Armed Bandit** scenario. It uses the **Upper Confidence Bound (UCB1)** algorithm to balance *Exploitation* (running the algorithm that currently has the highest composite score) with *Exploration* (giving underperforming algorithms a chance to prove themselves).

## The UCB1 Formula

For each algorithm $i$, EvoSynth calculates its UCB score at iteration $t$:

$$ UCB_i = \bar{x}_i + c \sqrt{\frac{\ln(t)}{n_i}} $$

Where:
- $\bar{x}_i$ is the historical average [Composite Score](./composite-scoring.md) of algorithm $i$.
- $t$ is the total number of iterations so far.
- $n_i$ is the number of times algorithm $i$ has been selected.
- $c$ is the exploration parameter (balancing factor).

## How it Works in EvoSynth

1. **Initialization Phase:** During the first `T1_iters` (e.g., 30 iterations), the bandit is strictly observing the 4 core algorithms (GA, PSO, DE, CMA-ES) in a round-robin or parallel fashion to gather baseline statistics.
2. **Scoring Phase:** Every $K$ iterations, the `CompositeScoringEngine` computes $\bar{x}_i$ based on the algorithm's fitness, speed, and diversity.
3. **Action Phase:** The Bandit updates the UCB scores. If a Tier 2 algorithm's UCB score falls significantly behind a Tier 1 algorithm's UCB score (exceeding the `swap_threshold`), the Meta-Engine actively swaps their positions, demoting the loser and promoting the winner.
