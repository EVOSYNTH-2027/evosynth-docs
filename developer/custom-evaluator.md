# Custom Evaluator

EvoSynth is currently configured to tune an XGBoost model on the Breast Cancer dataset. However, you can swap this out to tune **any** machine learning model (PyTorch, TensorFlow, Scikit-Learn) by writing a custom Evaluator.

## The `BaseEvaluator` Interface

Create a new class that inherits from `BaseEvaluator`:

```python
from automl_engine.core.evaluator import BaseEvaluator
from automl_engine.core.search_space import ConfigSpace
import torch
import torch.nn as nn

class PyTorchEvaluator(BaseEvaluator):
    def __init__(self):
        super().__init__()
        # Load your dataset here
        self.X, self.y = load_my_data()
        
    def evaluate(self, latent_vector: list[float]) -> float:
        """
        Takes a [0, 1] vector, decodes it, trains a model, and returns a scalar score.
        EvoSynth seeks to MAXIMIZE this score.
        """
        # 1. Decode the [0, 1] vector into real hyperparameters
        params = ConfigSpace.decode(latent_vector)
        
        # 2. Build your model using the params
        learning_rate = params['learning_rate']
        hidden_size = params['hidden_size']
        
        model = SimpleNeuralNet(hidden_size=hidden_size)
        optimizer = torch.optim.Adam(model.parameters(), lr=learning_rate)
        
        # 3. Train the model (Pseudo-code)
        train_model(model, optimizer, self.X, self.y)
        
        # 4. Calculate validation accuracy (or 1 - loss)
        accuracy = evaluate_model(model, self.X, self.y)
        
        return accuracy
```

## Updating the Search Space

If you write a custom evaluator, you must also update the `ConfigSpace` in `automl_engine/core/search_space.py` to match the hyperparameters your model expects.

```python
class ConfigSpace:
    BOUNDS = [
        (0.0001, 0.1),  # learning_rate
        (32, 512),      # hidden_size
    ]
    TYPES = ['float', 'int']
    CATEGORIES = {}
```

## Hooking it Up

Finally, pass your new evaluator into the `DynamicOptimizer` in `main.py`:

```python
from my_custom_evaluator import PyTorchEvaluator

evaluator = PyTorchEvaluator()
controller = DynamicOptimizer(evaluator=evaluator, budget=5000)
controller.optimize()
```
