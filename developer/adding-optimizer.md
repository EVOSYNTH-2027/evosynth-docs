# Adding a New Optimizer

EvoSynth's decoupled architecture makes it incredibly easy to add entirely new mathematical optimization algorithms without touching the machine learning code.

Because of the **Universal Latent Space**, your new algorithm only needs to know how to mutate a vector of floats between $0.0$ and $1.0$.

## The `BaseOptimizer` Interface

To add a new algorithm, create a new file in `automl_engine/optimizers/` (e.g., `my_optimizer.py`). 
Your class must inherit from `BaseOptimizer` and implement the following interface:

```python
from typing import List, Tuple
from automl_engine.core.evaluator import Individual
from automl_engine.optimizers.base import BaseOptimizer

class MyNewOptimizer(BaseOptimizer):
    def __init__(self, name: str, dimensions: int, pop_size: int):
        super().__init__(name, dimensions, pop_size)
        # Initialize your algorithm's specific state here
        
    def ask(self) -> List[Individual]:
        """
        Generate and return a list of completely new Individuals 
        that need to be evaluated by the Meta-Engine.
        
        The vector of each Individual must be in the range [0.0, 1.0].
        """
        pass
        
    def tell(self, evaluated_individuals: List[Individual]) -> None:
        """
        The Meta-Engine returns the Individuals you yielded in ask(), 
        but now they have their .fitness values populated.
        
        Update your algorithm's internal state (e.g., velocity, covariance) here.
        """
        pass
        
    def inject_elite(self, elite: Individual) -> None:
        """
        The Meta-Engine is forcibly injecting a highly fit individual 
        from another algorithm into your population.
        
        Handle this appropriately (e.g., replace your worst individual, 
        or update your Global Best).
        """
        pass
```

## The Ask/Tell Paradigm

Notice that your algorithm does **not** call the evaluator itself. 

It yields proposed solutions via `ask()`. The Meta-Engine evaluates them, and returns them via `tell()`. This asynchronous design is what allows the UCB1 Bandit to pause algorithms, dynamically alter their population sizes based on Tier status, and inject elites mid-flight!

## Registering the Optimizer

Once your class is written, simply import it and add it to the active algorithm list in `main.py`:

```python
from automl_engine.optimizers.my_optimizer import MyNewOptimizer

# Inside main.py
algorithms = [
    GeneticAlgorithm(name="GA", dimensions=dims, pop_size=0),
    MyNewOptimizer(name="MyOpt", dimensions=dims, pop_size=0)
]
```
*(Note: `pop_size=0` is correct. The `DynamicOptimizer` will dynamically assign the population size when it boots).*
