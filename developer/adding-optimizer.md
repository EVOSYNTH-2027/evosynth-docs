# Adding a New Optimizer

EvoSynth's decoupled architecture makes it easy to add entirely new optimization algorithms without touching the machine learning code.

Because of the **Universal Latent Space**, your new algorithm only needs to know how to mutate a vector of floats between $0.0$ and $1.0$.

## The `BaseOptimizer` Interface

To add a new algorithm, create a new file in `src/evosynth/optimizers/` (e.g., `my_optimizer.py`).
Your class must inherit from `BaseOptimizer` and implement the following interface:

```python
from evosynth.optimizers.base import BaseOptimizer


class MyNewOptimizer(BaseOptimizer):
    # BaseOptimizer.__init__(search_space, pop_size, rng) sets self.dimensions,
    # self.population and self.rng (this optimizer's own seeded random generator).

    def ask(self, num_samples):
        """
        Return `num_samples` new vectors (numpy arrays) to be evaluated by the Meta-Engine.
        Every value must be in the range [0.0, 1.0]. Use self.rng for all randomness,
        so a run is reproducible from its seed.
        """
        return [self.rng.random(self.dimensions) for _ in range(num_samples)]

    def tell(self, evaluated_individuals):
        """
        The Meta-Engine returns Individuals for the vectors you proposed in ask(),
        now with their .fitness values populated.

        Update your algorithm's internal state (e.g., velocity, covariance) here.
        """
        ...

    def inject(self, individuals):
        """
        Optional. The Meta-Engine is placing already-evaluated individuals into your
        population (elites from the partner island, or fresh random configurations
        when your island stagnates). The default replaces your worst members; override
        it if your algorithm keeps extra state (PSO resets particles, CMA-ES restarts).
        """
        super().inject(individuals)
```

Put the algorithm's constants (rates, coefficients) in `settings/optimizers.toml` and read them with `evosynth.config.get_settings()`.

## The Ask/Tell Paradigm

Notice that your algorithm does **not** call the evaluator itself.

It yields proposed solutions via `ask()`. The Meta-Engine evaluates them (in parallel threads) and returns them via `tell()`. This design is what allows the controller to score algorithms against each other, swap them between tiers, and inject elites mid-flight.

## Registering the Optimizer

Once your class is written, add it to the optimizer factory in `src/evosynth/engine/controller.py`:

```python
from evosynth.optimizers.my_optimizer import MyNewOptimizer

def _make_optimizer(algo_key, space, pop_size, rng=None):
    ...
    if algo_key == 'MYOPT': return MyNewOptimizer(space, pop_size, rng=rng)
```

Then add `'MYOPT'` to the controller's per-family bookkeeping (`algo_evals`, `cooldowns`, `_algo_stats`). Tier 1 currently has one island for each of the four families, so a fifth family also needs a fifth Tier 1 island.
