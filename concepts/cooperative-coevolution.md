# Cooperative Co-evolution

If four algorithms are running at the same time, it would be inefficient for them to be completely isolated. If one island finds an incredibly good configuration, its partner should know about it.

EvoSynth implements a **two-tier Island Model with Elite Sharing**.

## The Tier System

The evaluated initial population (N configurations) is split by fitness rank, so every group gets a similar mix of good and weak members:
- **Tier 1 (The Competition):** 40% of N, split into four equal islands, one each for GA, PSO, DE and CMA-ES. They compete for a fixed number of iterations.
- **Tier 2 reserve:** the other 60% of N, kept in two equal groups.

After the competition, the UCB1 bandit promotes the two best algorithms. Each gets a **Tier 2 island** made of its own Tier 1 island plus one reserve group, so most of the remaining budget goes to them. The two other algorithms keep running in Tier 1 with their small islands.

## Checkpoints

Every $k$ Tier 2 iterations the controller holds a checkpoint (with automatic $k$, Tier 2 always reaches at least two):

1. It rescores all islands with the composite score and UCB1.
2. **SWAP:** if the best Tier 1 algorithm beats the weakest Tier 2 algorithm by more than 0.10, they change places. The newcomer gets a 5-iteration grace period; if it doesn't improve the global best, the controller rolls back.
3. **Diversity injection:** an island that has stopped improving gets fresh random configurations in place of its worst members.
4. **Elite sharing:** the two Tier 2 islands trade elites.

## Elite Sharing

The exchange is pairwise: island A's worst 20% are replaced by island B's best 20%, and island B's worst 20% by island A's best 20%.

### Why do this?
By injecting the partner's elites into an island, you "warm seed" it.
For example, if CMA-ES is struggling to find the right basin of attraction, receiving the elites found by its partner pulls its search distribution (which restarts around the new population) towards a highly profitable area of the latent space. Each algorithm updates its own state when it receives members: PSO resets the replaced particles, CMA-ES restarts its distribution.
