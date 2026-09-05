# Composite Scoring Engine

When the UCB1 Multi-Armed Bandit is deciding which algorithm to reward, looking purely at the "Best Accuracy" is fundamentally flawed.

If Algorithm A finds a 98% accuracy in 10 minutes, and Algorithm B finds a 97.9% accuracy in 2 seconds, which one is better? If Algorithm C has collapsed into a local minimum and hasn't moved in 50 iterations, should it be rewarded?

To fix this, EvoSynth scores algorithms based on a **5-Metric Composite Score**.

## The Metrics

The `CompositeScoringEngine` computes a normalized $[0, 1]$ scalar for each algorithm based on:

### 1. Absolute Fitness ($w=0.4$)
The raw accuracy (or loss) of the absolute best individual the algorithm has ever found.

### 2. Recent Improvement Rate ($w=0.3$)
How much has the algorithm improved over the last $K$ iterations? Algorithms that are actively climbing gradients are rewarded over algorithms that are stagnating, even if their absolute fitness is slightly lower.

### 3. Population Diversity ($w=0.1$)
Calculated as the variance of the population vectors in the latent space. Algorithms with high diversity are rewarded to encourage global exploration and prevent premature convergence.

### 4. Convergence Speed ($w=0.1$)
How quickly the algorithm reached its current best fitness.

### 5. Historical Success Rate ($w=0.1$)
A long-term memory of how often this algorithm has been the "Global Best" throughout the entire run.

## Normalization and Aggregation

Every iteration, the engine computes these 5 metrics for all active algorithms.
It then Min-Max normalizes each metric across the active algorithms.

Finally, it computes the weighted sum:

$Score = (0.4 * Fitness) + (0.3 * ImpRate) + (0.1 * Div) + (0.1 * Speed) + (0.1 * History)$

This $Score$ is what is fed into the UCB1 Bandit to update the expected value of the algorithm "slot machine."
