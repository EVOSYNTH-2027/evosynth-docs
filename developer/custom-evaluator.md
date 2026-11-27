# Custom Evaluator

EvoSynth ships with an evaluator that tunes XGBoost, LightGBM, RandomForest and ExtraTrees on registered tabular datasets (Covertype, HIGGS, Adult and others). However, you can swap this out to tune **any** machine learning model (PyTorch, TensorFlow, Scikit-Learn) by writing a custom Evaluator.

## The Evaluator Interface

An evaluator needs only a `.space` (a `ConfigSpace` search space) and an `.evaluate(individual)` method:

```python
import ConfigSpace as CS
import ConfigSpace.hyperparameters as CSH
import torch

from evosynth.core.decoding import decode_vector_to_dict


class PyTorchEvaluator:
    def __init__(self):
        # Define the hyperparameters your model expects
        self.space = CS.ConfigurationSpace()
        self.space.add_hyperparameters([
            CSH.UniformFloatHyperparameter("learning_rate", lower=1e-4, upper=1e-1, log=True),
            CSH.UniformIntegerHyperparameter("hidden_size", lower=32, upper=512),
        ])
        # Load your dataset here
        self.X, self.y = load_my_data()

    def evaluate(self, individual) -> float:
        """
        Takes an Individual holding a [0, 1] vector, decodes it, trains a model, and returns a
        scalar score. EvoSynth seeks to MAXIMIZE this score (use %, e.g. 0-100).
        """
        # 1. Decode the [0, 1] vector into real hyperparameters
        params = decode_vector_to_dict(individual.vector, self.space)

        # 2. Build your model using the params
        model = SimpleNeuralNet(hidden_size=params["hidden_size"])
        optimizer = torch.optim.Adam(model.parameters(), lr=params["learning_rate"])

        # 3. Train the model (Pseudo-code)
        train_model(model, optimizer, self.X, self.y)

        # 4. Calculate validation accuracy (or 100 - loss)
        return evaluate_model(model, self.X, self.y)
```

`decode_vector_to_dict` handles log scales, integers and categorical bins exactly as EvoSynth's own evaluators do, so every optimizer works on your space unchanged.

## Hooking it Up

Pass your evaluator and its search space into the `DynamicOptimizer`:

```python
from evosynth.engine.controller import DynamicOptimizer
from my_custom_evaluator import PyTorchEvaluator

evaluator = PyTorchEvaluator()
controller = DynamicOptimizer(search_space=evaluator.space, evaluator=evaluator, budget=500, pop_size=40)
best = controller.run()
print(best.fitness, decode_vector_to_dict(best.vector, evaluator.space))
```
