# Live Telemetry Dashboard

EvoSynth ships with a beautifully designed, high-performance React dashboard that allows you to peer inside the "black box" of the hyperparameter optimization process in real-time.

## Tech Stack

The dashboard is built using modern, fast web technologies:
- **React 18** (UI Components)
- **Vite** (Rapid development server and building)
- **Tailwind CSS** (Utility-first styling with a custom dark theme)
- **Recharts** (Declarative charts for telemetry visualization)
- **Server-Sent Events (SSE)** (Native browser API for real-time one-way data streaming)

## Why a Dashboard?

When running a 10-hour hyperparameter search, printing `Iteration 500/2000...` to the console is unacceptable. You need to know:
1. Is the model actively improving, or has it stagnated?
2. Which algorithm is currently winning the Multi-Armed Bandit competition?
3. Is CMA-ES actually finding better solutions than GA, or did GA get lucky early on?
4. What is the current configuration of the XGBoost model?

The dashboard answers all of these questions instantaneously without pausing or slowing down the Python engine.
