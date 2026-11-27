# Composite Scoring Engine

When the UCB1 Multi-Armed Bandit is deciding which algorithm to reward, looking purely at the "Best Accuracy" is fundamentally flawed.

If Algorithm A finds a 98% accuracy after a long time, and Algorithm B finds 97.9% in a fraction of the evaluation time, which one is better? If Algorithm C has collapsed into a local minimum and hasn't improved in many iterations, should it be rewarded?

To fix this, EvoSynth scores algorithms with a **5-Metric Composite Score**.

## The Metrics

At every scoring call (after the Tier 1 competition and at every Tier 2 checkpoint), the `CompositeScoringEngine` measures each algorithm's island:

### 1. Fitness ($w=0.30$)
The best score in the island, in the dataset's metric (accuracy or ROC AUC).

### 2. Speed ($w=0.25$)
How much the algorithm's best score improved since its last scoring call. Algorithms that are actively climbing are rewarded over algorithms that are stagnating, even if their fitness is slightly lower.

### 3. Efficiency ($w=0.15$)
That improvement divided by the seconds of evaluation time the algorithm used since its last scoring call: progress per unit of compute.

### 4. Diversity ($w=0.15$)
The mean pairwise distance between the island's configurations in the latent space, divided by $\sqrt{d}$. Algorithms that are still exploring are rewarded, which discourages premature convergence.

### 5. Trend ($w=0.15$)
The slope of the algorithm's best score over its last 3 scoring calls: sustained momentum rather than one lucky jump.

## Normalization and Aggregation

At each scoring call, the engine computes these 5 metrics for all active islands and min-max normalizes each metric across them (a metric on which all islands tie gives each 0.5).

Finally, it computes the weighted sum:

$Score = (0.30 * Fitness) + (0.25 * Speed) + (0.15 * Efficiency) + (0.15 * Diversity) + (0.15 * Trend)$

This $Score$ is what is fed into the UCB1 Bandit, which adds an exploration bonus and ranks the algorithms for every promotion and SWAP decision. The weights are settings (`[composite]` in `settings/engine.toml`).
