# Interpreting the Dashboard

When you open the React dashboard during a run (start `evosynth serve`, then the dashboard, then a run), it shows five panels. All of them are fed by the [SSE stream](./sse-protocol.md); nothing is polled.

## 1. Header and progress bar
The header shows the current workflow step ("Step 3: Tier 1 Competition", "Step 5: Tier 2 …") and an **Evaluations** bar (`evaluations used / budget`). The budget is shared by all four optimizers, so the bar tells you how much of the whole run is spent, not how much one optimizer used.

## 2. Tier Architecture
Two rows of chips: **Tier 1** (the competition arena) and **Tier 2** (the working population).
- After the Tier 1 tournament, the two families with the highest UCB scores move to Tier 2.
- A **SWAP** moves a surging Tier 1 family into Tier 2 and the weaker Tier 2 family down. It happens only when the challenger's UCB score exceeds the weaker Tier 2 family's by more than 0.10.
- A **Rollback** moves them back if the promoted family does not improve the global best within its 5-iteration grace period.

Chips changing is the visible sign of the meta-optimizer at work.

## 3. Live Best Accuracy
A leaderboard of each family's best cross-validated score so far (accuracy or ROC AUC, depending on the dataset), with the leader marked.
- **A family that stops moving** has stagnated. In Tier 1 its island gets 50% fresh random members at once; in Tier 2, 20% after three stagnant checkpoints.
- **A Tier 2 family that jumps to the other's score** has usually just received its elites: at each checkpoint the two Tier 2 islands trade their best 20% (each island's worst 20% are replaced by the other's best). Elites are shared only between the two Tier 2 islands, never with Tier 1.
- The best score in the search is a 2-fold CV score on the search rows. The final model's held-out test score is printed at the end of the run, after the top 5 configurations are confirmed with 5-fold CV and the selected one is refitted.

## 4. Switch Feed
One card per meta-decision, newest first: **Promotion**, **Swap**, **Rollback**, **Injection** and **Elite Sharing**, each with the families involved and the reason. This is the audit trail of the run; the same decisions are saved in the run folder.

## 5. Terminal
The full console output, including the scoring table at each checkpoint:

| Column | Meaning |
|---|---|
| Best Acc | the family's best CV score |
| Diversity | mean pairwise distance between the island's latent vectors, divided by √d (0 = collapsed, 1 = maximally spread). Values near 0.05 mean the island has converged onto one region |
| Composite | `0.30·fitness + 0.25·speed + 0.15·efficiency + 0.15·diversity + 0.15·trend`, each term min–max normalised across the families (see [Composite Scoring](../concepts/composite-scoring.md)) |
| Bonus | the UCB1 exploration bonus `0.1·sqrt(ln N / n)`; larger for families that have used fewer evaluations |
| UCB Score | composite + bonus; the number promotions and swaps are decided on |

**Reading it:** a family can rank first with a lower best score if it is still improving quickly and keeps a diverse island. In a real Covertype run, DE was promoted ahead of CMA-ES this way (composite 0.961 against 0.417) although CMA-ES had the higher best.
