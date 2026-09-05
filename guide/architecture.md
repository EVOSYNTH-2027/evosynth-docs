# System Architecture

EvoSynth is built on a heavily decoupled, three-layer architecture. This ensures that the mathematical optimization algorithms are completely isolated from the machine learning evaluation logic, making the system highly extensible.

## The Three Layers

### 1. The Core Evaluator Layer (`automl_engine/core`)
This layer is responsible for defining what we are optimizing and how to score it.
- **Search Space:** Maps categorical, integer, and float hyperparameters to the Universal Latent Space $[0, 1]$.
- **Evaluators:** Takes a decoded hyperparameter configuration, trains a machine learning model (e.g., XGBoost on the Breast Cancer dataset), and returns an accuracy score via $k$-fold cross-validation.
- **Individual:** The foundational data structure representing a single "solution" (vector) moving through the space.

### 2. The Meta-Engine Layer (`automl_engine/engine`)
The brains of EvoSynth. It orchestrates the algorithms, allocates budget, and manages the population.
- **DynamicOptimizer:** The main loop that initializes the algorithms and controls the flow of time (iterations/evaluations).
- **UCB1 Bandit:** The reinforcement learning agent that observes algorithm performance and decides which algorithm deserves more CPU time based on the Upper Confidence Bound formula.
- **Tier Manager:** Handles the Cooperative Co-evolution island model, promoting algorithms to Tier 2 (active focus) or demoting them to Tier 1 (background exploration).
- **EventReporter:** Hooks deeply into the Python `sys.stdout` stream to serialize internal states and fire them over HTTP to the API server asynchronously.

### 3. The Worker Optimizers (`automl_engine/optimizers`)
The mathematical worker bees. These algorithms have no concept of "Machine Learning" or "Hyperparameters." They simply receive $[0, 1]$ vectors, mathematically mutate them, and ask the Meta-Engine to evaluate them.
- **Genetic Algorithm (GA)**
- **Particle Swarm Optimization (PSO)**
- **Differential Evolution (DE)**
- **CMA-ES**

## Telemetry Flow (End-to-End)

The observability pipeline is designed to be completely non-blocking for the ML training loop:

1. The **Meta-Engine** makes a decision (e.g., "Swapping PSO with GA").
2. The **EventReporter** intercepts this, wraps it in JSON, and spawns a daemon thread to `POST /event`.
3. The **FastAPI Server** (`api_server.py`) receives the event and pushes it to an `asyncio.Queue`.
4. The **React Dashboard** receives the event instantly via a Server-Sent Events (SSE) `GET /stream` connection and visually updates the UI.
