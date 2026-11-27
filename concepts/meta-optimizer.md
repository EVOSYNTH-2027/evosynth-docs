# The UCB1 Meta-Optimizer

Traditional hyperparameter tuning forces you to choose a single algorithm. EvoSynth treats the choice of algorithm itself as an optimization problem.

To solve this, EvoSynth frames the active algorithms as "slot machines" in a **Multi-Armed Bandit** scenario. It uses the **Upper Confidence Bound (UCB1)** algorithm to balance *Exploitation* (running the algorithm that currently has the highest composite score) with *Exploration* (giving underperforming algorithms a chance to prove themselves).

## The UCB1 Formula

For each algorithm $i$, EvoSynth calculates its UCB score at every scoring call:

$$ UCB_i = \bar{x}_i + c \sqrt{\frac{\ln(N)}{n_i}} $$

Where:
- $\bar{x}_i$ is the current [Composite Score](./composite-scoring.md) of algorithm $i$.
- $N$ is the total number of evaluations used so far.
- $n_i$ is the number of evaluations algorithm $i$ has used.
- $c$ is the exploration parameter ($c = 0.1$ by default).

## How it Works in EvoSynth

1. **Competition Phase:** During the first `t1_iters` iterations, the 4 core algorithms (GA, PSO, DE, CMA-ES) each run on their own Tier 1 island. The bandit then ranks them and promotes the top 2 to Tier 2.
2. **Scoring Phase:** Every $k$ Tier 2 iterations, the `CompositeScoringEngine` computes $\bar{x}_i$ from each algorithm's fitness, speed, efficiency, diversity and trend.
3. **Action Phase:** The Bandit updates the UCB scores. If the best Tier 1 algorithm's UCB score is higher than the weakest Tier 2 algorithm's by more than the `swap_margin` (0.10), the Meta-Engine swaps their positions, demoting the loser and promoting the winner. The newcomer must improve the global best within a short grace period, or the swap is rolled back.
