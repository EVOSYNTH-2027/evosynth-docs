# Search Space (ConfigSpace)

Search spaces define the boundaries and data types of the hyperparameters being optimized. EvoSynth builds them with the `ConfigSpace` library from the ranges in `settings/search_spaces.toml`, so a range or a choice is changed there, not in code.

## The Tabular Search Space (8 Dimensions)

```python
from evosynth.search_spaces.tabular import get_tabular_search_space

space = get_tabular_search_space(include_model_type=True, include_fe_strategy=True)
```

| Dimension | Type | Range / choices |
| :--- | :--- | :--- |
| `colsample_bytree` | Float | 0.3 to 1.0 (also `max_features` for RandomForest / ExtraTrees) |
| `fe_strategy` | Categorical | `none`, `stats`, `interactions` |
| `learning_rate` | Float, log scale | 0.01 to 0.3 |
| `max_depth` | Integer | 3 to 10 |
| `model_type` | Categorical | `XGBoost`, `RandomForest`, `LightGBM`, `ExtraTrees` |
| `n_estimators` | Integer | 50 to 500 |
| `reg_alpha` | Float | 0.0 to 10.0 |
| `subsample` | Float | 0.5 to 1.0 |

ConfigSpace orders the dimensions alphabetically, as above; position $i$ of every latent vector is always the same hyperparameter.

## Decoding a Latent Vector

```python
from evosynth.core.decoding import decode_vector_to_dict

params = decode_vector_to_dict(vector, space)   # vector: 8 floats in [0, 1]
# {'colsample_bytree': 0.71, 'fe_strategy': 'interactions', 'learning_rate': 0.042, ...}
```

Floats are scaled linearly (or logarithmically) to their bounds, integers are scaled and rounded, and categoricals use equal bins: with $k$ choices, choice index $= \min(\lfloor z \cdot k \rfloor, k - 1)$.

The number of dimensions, `len(list(space.keys()))`, is what the Meta-Engine uses to size every random vector in the population.

## Other Spaces

- **Neural Architecture Search:** `evosynth.search_spaces.nas.get_image_nas_search_space()`: convolution channels, fully connected units, dropout, learning rate, optimizer and activation for a PyTorch CNN (7 dimensions).
- **Toy space:** `evosynth.search_spaces.toy.get_dummy_search_space()`: a 4-dimensional space used with a fast synthetic objective to test the engine.
