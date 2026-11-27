# Evaluators

The Evaluator layer bridges the gap between the mathematical latent space vectors $[0, 1]$ and actual Machine Learning models.

An evaluator is any object with an `.evaluate(individual)` method that returns a score (higher is better, in %). The engine never needs anything else from it.

## TabularEvaluator

The default production evaluator included in EvoSynth. It searches over the model family, a feature-engineering strategy and six hyperparameters on any registered tabular dataset.

### Usage
```python
from evosynth.evaluators.tabular import TabularEvaluator

evaluator = TabularEvaluator("covertype")            # or "higgs", "adult", "airlines", ...
score = evaluator.evaluate(individual)               # mean CV score in %
```

### Internal Mechanics
When `.evaluate(individual)` is called:
1. It decodes the individual's latent vector with the search space into actual hyperparameters (e.g., `{'model_type': 'LightGBM', 'fe_strategy': 'interactions', 'learning_rate': 0.1, 'max_depth': 6, ...}`).
2. It runs stratified $k$-fold cross-validation (2 folds by default) on the dataset's **search rows**. The dataset's held-out test rows are never used during search.
3. In each fold it applies the feature-engineering strategy (fitted on the training fold), builds the model (XGBoost, LightGBM, RandomForest or ExtraTrees), and trains it. Boosted models stop early on a slice of the training fold, never on the fold being scored.
4. It returns the mean score in the dataset's metric: accuracy for multiclass datasets, ROC AUC for binary ones. A fold whose model fails scores 0, and the failure is logged.

After the search, the best configurations are re-scored with 5-fold cross-validation, and the selected one is retrained on the dataset's training rows and scored once on its held-out test rows.

## ImageCNNEvaluator

Evaluates PyTorch CNN architectures and training hyperparameters on Fashion-MNIST, for neural architecture search (`evosynth nas`).

## DummyObjectiveFunction

A high-speed synthetic objective function used primarily for testing, debugging, and observing algorithm behavior without the overhead of training an ML model.

### Usage
```python
from evosynth.evaluators.toy import DummyObjectiveFunction

evaluator = DummyObjectiveFunction()
```

### Internal Mechanics
It decodes the vector into a small toy configuration (learning rate, number of layers, optimizer, dropout) and scores how close it is to a hidden "perfect" configuration, returning a mock accuracy of up to 100. It evaluates in microseconds, so the whole engine can be tested in seconds.
