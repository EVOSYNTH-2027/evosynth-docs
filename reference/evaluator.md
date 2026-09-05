# Evaluators

The Evaluator layer bridges the gap between the mathematical latent space vectors $[0, 1]$ and actual Machine Learning models.

Every evaluator must inherit from `BaseEvaluator`.

## XGBoostEvaluator

The default production evaluator included in EvoSynth.

### Usage
```python
from automl_engine.core.evaluator import XGBoostEvaluator

evaluator = XGBoostEvaluator()
```

### Internal Mechanics
When `.evaluate(latent_vector)` is called:
1. It passes the `latent_vector` to the `ConfigSpace` to decode it into a dictionary of actual hyperparameters (e.g., `{'learning_rate': 0.1, 'max_depth': 6, 'booster': 'gbtree'}`).
2. It initializes an `xgboost.XGBClassifier` using these parameters.
3. It performs a 5-fold cross-validation on the built-in Breast Cancer dataset.
4. It returns the mean accuracy score (a float between `0.0` and `1.0`).

## DummyObjectiveFunction

A high-speed synthetic objective function used primarily for testing, debugging, and observing algorithm behavior without the overhead of training an ML model.

### Usage
```python
from automl_engine.core.evaluator import DummyObjectiveFunction

evaluator = DummyObjectiveFunction()
```

### Internal Mechanics
It implements the classic **Rastrigin Function**—a non-convex function often used as a performance test problem for optimization algorithms because it has a large number of local minima. 

It takes the $[0, 1]$ vector, maps it to $[-5.12, 5.12]$, calculates the Rastrigin score, and normalizes the return value so that finding the global minimum (0) returns a fitness of `1.0`.
