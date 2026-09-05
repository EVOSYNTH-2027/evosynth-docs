# Universal Latent Space

One of the biggest challenges in applying purely mathematical optimizers (like Particle Swarm Optimization) to Machine Learning hyperparameter tuning is the presence of mixed data types.

A typical XGBoost search space looks like this:
- `learning_rate`: Float (0.01 to 0.3)
- `max_depth`: Integer (3 to 10)
- `booster`: Categorical (`gbtree`, `gblinear`, `dart`)

How can a particle calculate a "velocity" through a categorical dimension like `booster`?

## The [0, 1] Mapping Engine

EvoSynth solves this by completely abstracting the hyperparameter space away from the algorithms.

Every single Individual in EvoSynth is represented strictly as an $n$-dimensional vector of floats, bounded strictly between $[0.0, 1.0]$. 

When an algorithm mutates a vector, it might create a point like:
`[0.15, 0.99, 0.45, ...]`

### The Decoding Process

Before the model is trained, the `Evaluator` layer intercepts this vector and decodes it using the `ConfigSpace` definition:

1. **Floats:** Linearly (or logarithmically) scaled from $[0, 1]$ to the target bounds.
2. **Integers:** Scaled to the bounds and rounded to the nearest whole number.
3. **Categoricals:** The $[0, 1]$ range is divided evenly into bins. E.g., if there are 3 choices, $0.0 - 0.33$ maps to choice 1, $0.33 - 0.66$ maps to choice 2, etc.

Because the algorithms only ever see smooth $[0, 1]$ floats, they can easily compute Euclidean distances, velocities, and covariance matrices without crashing on categorical boundaries.
