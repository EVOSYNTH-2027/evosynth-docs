# Search Space (ConfigSpace)

The `ConfigSpace` class is responsible for defining the boundaries and data types of the hyperparameters being optimized.

## Definition Example

To define the XGBoost search space, EvoSynth uses the following configuration:

```python
class ConfigSpace:
    # Defining the XGBoost Search Space (7 Dimensions)
    BOUNDS = [
        (0, 2),            # 0: booster (Categorical: 0=gbtree, 1=gblinear, 2=dart)
        (3, 10),           # 1: max_depth (Integer)
        (0.01, 0.3),       # 2: learning_rate (Float)
        (50, 500),         # 3: n_estimators (Integer)
        (0.5, 1.0),        # 4: subsample (Float)
        (0.5, 1.0),        # 5: colsample_bytree (Float)
        (0.0, 5.0)         # 6: reg_alpha (Float)
    ]
    
    TYPES = [
        'cat',      # booster
        'int',      # max_depth
        'float',    # learning_rate
        'int',      # n_estimators
        'float',    # subsample
        'float',    # colsample_bytree
        'float'     # reg_alpha
    ]
    
    CATEGORIES = {
        0: ['gbtree', 'gblinear', 'dart']
    }
```

## Methods

### `.decode(latent_vector)`
Accepts an $n$-dimensional list or numpy array where every element is between $0.0$ and $1.0$.

Returns a dictionary mapping the hyperparameter names to their real values (e.g., `'learning_rate': 0.15`).

### `.get_dimensions()`
Returns the number of dimensions in the search space (e.g., `7`). This is used by the Meta-Engine to initialize the sizes of the random vectors in the population.
