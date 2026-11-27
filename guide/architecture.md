# System Architecture

EvoSynth is built on a heavily decoupled, three-layer architecture. This ensures that the mathematical optimization algorithms are completely isolated from the machine learning evaluation logic, making the system highly extensible.

## The Three Layers

### 1. The Core Evaluator Layer (`src/evosynth/core`, `search_spaces`, `evaluators`, `datasets`)
This layer is responsible for defining what we are optimizing and how to score it.
- **Search Space:** Maps categorical, integer, and float hyperparameters to the Universal Latent Space $[0, 1]$.
- **Datasets:** A registry of real datasets (e.g. Covertype, HIGGS, Adult). Each has fixed held-out test rows that no search sees, and its own metric (accuracy or ROC AUC).
- **Evaluators:** Takes a decoded hyperparameter configuration, trains a machine learning model (e.g., XGBoost on the Covertype search rows), and returns its score via $k$-fold cross-validation.
- **Individual:** The foundational data structure representing a single "solution" (vector) moving through the space.

### 2. The Meta-Engine Layer (`src/evosynth/engine`)
The brains of EvoSynth. It orchestrates the algorithms, allocates budget, and manages the population.
- **DynamicOptimizer:** The main loop that initializes the algorithms and controls the flow of time (iterations/evaluations).
- **UCB1 Bandit:** The bandit that observes algorithm performance through the composite score and decides which algorithms are promoted to Tier 2 and when they are swapped, using the Upper Confidence Bound formula.
- **Tier Manager:** Splits the initial population into the Tier 1 islands and the Tier 2 reserve (stratified by fitness rank) and runs the elite exchange between the two Tier 2 islands.
- **EventReporter:** Serialises the engine's decisions and scores into events, writes them to the run's `events.jsonl`, and fires them over HTTP to the API server asynchronously.

### 3. The Worker Optimizers (`src/evosynth/optimizers`)
The mathematical worker bees. These algorithms have no concept of "Machine Learning" or "Hyperparameters." They simply receive $[0, 1]$ vectors, mathematically mutate them, and ask the Meta-Engine to evaluate them.
- **Genetic Algorithm (GA)**
- **Particle Swarm Optimization (PSO)**
- **Differential Evolution (DE)**
- **CMA-ES**

## Telemetry Flow (End-to-End)

The observability pipeline is designed to be completely non-blocking for the ML training loop:

1. The **Meta-Engine** makes a decision (e.g., "Swapping PSO with GA").
2. The **EventReporter** intercepts this, wraps it in JSON, and spawns a daemon thread to `POST /event`.
3. The **FastAPI Server** (started with `evosynth serve`, `src/evosynth/telemetry/server.py`) receives the event and pushes it to an `asyncio.Queue`.
4. The **React Dashboard** receives the event instantly via a Server-Sent Events (SSE) `GET /stream` connection and visually updates the UI.
