# DynamicOptimizer (The Meta-Engine)

The `DynamicOptimizer` class is the central orchestrator of EvoSynth. It manages the UCB1 Bandit, controls the Tier system, and handles the main event loop.

## Class Signature

```python
from evosynth.engine.controller import DynamicOptimizer

class DynamicOptimizer:
    def __init__(
        self,
        search_space,
        evaluator,
        budget: int = None,      # 2000
        pop_size: int = None,    # 100
        t1_iters: int = None,    # 30
        k_iters: int = None,     # 10
        ...
    )
```

Every argument left as `None` takes its value from the `settings/` folder (`[engine]`, `[safeguards]`, `[run]`); the defaults are shown above. The command presets use smaller values, e.g. `evosynth run --full` uses a budget of 500 and a population of 40.

## Constructor Parameters

- `search_space` *(ConfigSpace)*: the space every latent vector is decoded with.
- `evaluator`: an object with an `.evaluate(individual)` method returning a score in % (e.g., `TabularEvaluator`).
- `budget` *(int)*: The maximum number of model evaluations allowed across all algorithms.
- `pop_size` *(int)*: The initial population size N. 40% of it seeds the four Tier 1 islands and 60% is kept as the Tier 2 reserve; each Tier 1 island needs at least 4 members, so N must be at least 40.
- `t1_iters` *(int)*: The number of iterations of the Tier 1 competition, after which the top 2 algorithms are promoted.
- `k_iters` *(int)*: The interval between Tier 2 checkpoints. It is lowered automatically when the budget would otherwise allow fewer than two checkpoints.

## Public Methods

### `.run()`
Starts the optimization loop.

**Returns:** the best `Individual` found (its `.vector`, `.fitness` and `.source_algorithm`).

After `.run()`, the optimizer also exposes every evaluated individual (`.evaluated`), every scoring call with each island's metrics and UCB score (`.scoring_history`), and every promotion, SWAP, rollback, injection and elite exchange (`.decisions`).

## Internal Flow

When `.run()` is called, the controller executes the following loop:
1. **Evaluate Initial Population:** Generates `pop_size` random vectors and evaluates them in parallel threads.
2. **Tier Split:** Splits the population by fitness rank so every group gets a similar mix: 40% into four equal Tier 1 islands (GA, PSO, DE, CMA-ES) and 60% into two reserve groups.
3. **Tier 1 Competition:** Runs all four algorithms for `t1_iters` iterations.
4. **Promotion:** Scores the islands (composite score + UCB1) and promotes the top 2 algorithms to Tier 2, each with its Tier 1 island plus one reserve group. The other two keep running in Tier 1.
5. **Main Loop:** Runs the Tier 2 and remaining Tier 1 islands. Every $k$ iterations: re-computes the composite scores, updates the UCB Bandit, potentially swaps algorithm tiers (with a grace period and rollback), injects fresh configurations into stagnant islands, and exchanges elites between the two Tier 2 islands.
6. **Termination:** Stops when the `budget` is exhausted and returns the best individual. The `evosynth run` command then re-scores the top configurations with 5-fold CV, retrains the selected one and scores it on held-out test data.
