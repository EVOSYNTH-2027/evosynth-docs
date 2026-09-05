# DynamicOptimizer (The Meta-Engine)

The `DynamicOptimizer` class is the central orchestrator of EvoSynth. It manages the UCB1 Bandit, controls the Tier system, and handles the main event loop.

## Class Signature

```python
class DynamicOptimizer:
    def __init__(
        self,
        evaluator: BaseEvaluator,
        budget: int = 2000,
        pop_size: int = 100,
        t1_iters: int = 30
    )
```

## Constructor Parameters

- `evaluator` *(BaseEvaluator)*: An instance of an evaluator (e.g., `XGBoostEvaluator`) that implements a `.evaluate(latent_vector)` method.
- `budget` *(int)*: The maximum number of model evaluations allowed across all algorithms. Default is `2000`.
- `pop_size` *(int)*: The global population size. This is automatically partitioned among the active algorithms based on their tier. Default is `100`.
- `t1_iters` *(int)*: The number of iterations in the "Initialization Phase" where all algorithms compete equally before the first Tier split occurs. Default is `30`.

## Public Methods

### `.optimize()`
Starts the optimization loop.

**Returns:**
- A tuple: `(best_latent_vector, best_decoded_hyperparameters, best_accuracy_score)`

## Internal Flow

When `.optimize()` is called, the controller executes the following loop:
1. **Initialize Algorithms:** Boots GA, PSO, DE, and CMA-ES.
2. **Evaluate Initial Population:** Generates `pop_size` random vectors and evaluates them to establish a baseline.
3. **Initialization Phase:** Runs all algorithms for `t1_iters` to collect UCB1 statistics.
4. **Tier Split:** Promotes the Top 2 algorithms to Tier 2 (receiving 60% of `pop_size`), and demotes the bottom 2 to Tier 1 (receiving 40% of `pop_size`).
5. **Main Loop:** Continues running algorithms according to their Tier budgets. Every $K$ iterations, re-computes the composite scores, updates the UCB Bandit, handles Elite Migration, and potentially swaps algorithm tiers.
6. **Termination:** Breaks the loop and returns the absolute best configuration when the `budget` is exhausted.
