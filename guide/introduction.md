# Introduction to EvoSynth

**EvoSynth** is an advanced, academic-grade Automated Machine Learning (AutoML) framework. It was designed from the ground up to solve a fundamental flaw in traditional hyperparameter optimization: the reliance on a single, static search algorithm.

## The Problem

When tuning complex machine learning models (like XGBoost, Random Forests, or deep neural networks), data scientists typically choose one algorithm—like Grid Search, Bayesian Optimization, or a Genetic Algorithm—and run it until the budget is exhausted. 

However, different algorithms excel at different phases of the search:
- **Swarm Intelligence (PSO, CMA-ES)** is incredibly fast at converging on a local optimum.
- **Genetic Algorithms (GA)** excel at global exploration and escaping local minima.

If you lock yourself into one algorithm, you either converge too early (stagnation) or explore too broadly (inefficiency).

## The EvoSynth Solution

EvoSynth operates as a **Meta-Optimizer**. Instead of picking one algorithm, it runs multiple evolutionary algorithms simultaneously.

Using a **Multi-Armed Bandit (UCB1)** controller, EvoSynth observes the performance of each algorithm in real-time. It dynamically routes the computational budget (i.e., CPU time) to the algorithm that is currently performing the best, while reserving a small budget to explore algorithms that might find a breakthrough later.

### Key Innovations

1. **Universal Latent Space:** Hyperparameters are often mixed (learning rate is a float, max depth is an integer, booster type is a categorical string). EvoSynth maps all of these into a unified $[0, 1]$ continuous vector space. This allows purely mathematical algorithms (like Particle Swarm) to fly through categorical dimensions seamlessly.
2. **Cooperative Co-evolution:** The population of algorithms is divided into a strict hierarchy (Tier 1 and Tier 2). Elite individuals are periodically shared across islands to guarantee global knowledge transfer.
3. **Glass-Box Telemetry:** Machine learning shouldn't be a black box. EvoSynth streams its internal decision-making process (Bandit UCB scores, algorithm swaps, diversity metrics) to a beautiful React dashboard in real-time via Server-Sent Events (SSE).

## Next Steps

To understand how these pieces fit together, check out the [System Architecture](./architecture.md).
