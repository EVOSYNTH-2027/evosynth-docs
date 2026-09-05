---
layout: home

hero:
  name: EvoSynth
  text: Dynamic Meta-Optimizer for AutoML
  tagline: Multi-Armed Bandit strategy selection with cooperative co-evolution, real-time telemetry, and a universal latent space for mixed-type hyperparameter tuning.
  image:
    src: /evo.png
    alt: EvoSynth Neural and DNA graphic
  actions:
    - theme: brand
      text: Get Started
      link: /guide/introduction
    - theme: alt
      text: View Algorithms
      link: /algorithms/genetic-algorithm
    - theme: alt
      text: GitHub
      link: https://github.com/EVOSYNTH-2027

features:
  - title: UCB1 Multi-Armed Bandit
    details: Frames algorithm selection as a bandit problem. Dynamically allocates computational budget to the strategy delivering the highest composite reward, balancing exploitation and exploration.
    link: /concepts/meta-optimizer
    linkText: Learn more
  - title: Universal Latent Space
    details: Maps all hyperparameters (continuous, integer, categorical) into a unified [0, 1] continuous vector space, enabling mathematical optimizers like PSO to operate natively on mixed-type search spaces.
    link: /concepts/latent-space
    linkText: Learn more
  - title: Cooperative Co-evolution
    details: Structures the population into competing Tier 1 islands and a Tier 2 working pool. Elite individuals are periodically shared between islands to prevent genetic stagnation.
    link: /concepts/cooperative-coevolution
    linkText: Learn more
  - title: 4 Optimization Algorithms
    details: Ships with Genetic Algorithm (GA), Particle Swarm Optimization (PSO), Differential Evolution (DE), and CMA-ES. All follow a decoupled ask/tell interface for easy extensibility.
    link: /algorithms/genetic-algorithm
    linkText: Explore algorithms
  - title: Real-Time Telemetry Dashboard
    details: A React + Vite + Tailwind CSS dashboard that streams live optimization data via Server-Sent Events (SSE). Watch strategy switches, accuracy tracking, and tier state changes in real-time.
    link: /dashboard/overview
    linkText: View dashboard
  - title: Extensible Architecture
    details: Add new optimization algorithms, swap the ML evaluator (XGBoost, Random Forest, Neural Networks), or distribute evaluations across cloud workers with minimal code changes.
    link: /developer/adding-optimizer
    linkText: Developer guide
---
