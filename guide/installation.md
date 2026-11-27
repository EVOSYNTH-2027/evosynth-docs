# Installation & Setup

Running the full EvoSynth suite involves three components: the ML engine (the `evosynth` command), the telemetry API server, and the React frontend dashboard. The engine also runs on its own; the server and dashboard are only needed for live monitoring.

## Prerequisites

- **Python 3.11+** (developed and tested on Python 3.14)
- **Node.js 18+** (for the Vite/React dashboard)

---

## 1. Install the EvoSynth Package

Navigate to the root of the `EvoSynth` directory and install the package. We highly recommend using a virtual environment.

```bash
cd EvoSynth

# Create virtual environment
python -m venv venv

# Activate it (Mac/Linux)
source venv/bin/activate
# Activate it (Windows)
# venv\Scripts\activate

# Install EvoSynth with all extras (NAS, baselines, tests)
pip install -e ".[all]"
```

This installs the `evosynth` command (also runnable as `python -m evosynth`). All dependencies are pinned in `pyproject.toml`.

Datasets are downloaded on first use into `.data/`, or all at once:
```bash
evosynth datasets download   # Covertype, HIGGS, Adult, Airlines, Poker Hand, MiniBooNE, Jannis, Fashion-MNIST
evosynth datasets list       # what is registered and cached
```

## 2. Install Frontend Dependencies

Next, navigate into the `dashboard` directory and install the Node packages.

```bash
cd dashboard
npm install
```

---

## Running the Suite

The engine checks whether the telemetry server is reachable and keeps checking during the run, so the server can be started before or after the engine. Start it first if you want to see the run from its first step.

### Terminal 1: The API Server
This acts as the Server-Sent Events (SSE) router.
```bash
cd EvoSynth
source venv/bin/activate
evosynth serve
```
*The server listens on `http://localhost:8000` by default (`settings/telemetry.toml`).*

### Terminal 2: The Telemetry Dashboard
This boots the React UI.
```bash
cd EvoSynth/dashboard
npm run dev
```
*Expected Output: `VITE ... ready` (Available at `http://localhost:5173`)*

### Terminal 3: Fire the Engine
This starts the UCB1 Multi-Armed Bandit meta-optimizer on the Covertype dataset.
```bash
cd EvoSynth
source venv/bin/activate
evosynth run                       # quick search + refit (200 evaluations)
evosynth run --full --dataset adult  # 500 evaluations on another dataset
```

Once the run starts, open `http://localhost:5173` in your browser to watch the live telemetry stream: islands, scores, promotions, SWAPs and injections.

Every run writes its results to `results/<experiment>/<dataset>/<variant>/seed_<k>/` (settings used, scores, full log). All settings live in the `settings/` folder and can be changed per run, e.g. `evosynth run --set bandit.c=0.2`.
