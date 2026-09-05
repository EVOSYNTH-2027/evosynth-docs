# Installation & Setup

Running EvoSynth requires booting three separate components: the ML backend engine, the telemetry API server, and the React frontend dashboard.

## Prerequisites

- **Python 3.9+** (For the machine learning engine)
- **Node.js 18+** (For the Vite/React dashboard)

---

## 1. Install Backend Dependencies

Navigate to the root of the `EvoSynth` directory and install the required Python packages. We highly recommend using a virtual environment.

```bash
cd EvoSynth

# Create virtual environment
python -m venv venv

# Activate it (Mac/Linux)
source venv/bin/activate
# Activate it (Windows)
# venv\Scripts\activate

# Install requirements
pip install -r requirements.txt
```

*(Note: Ensure your `requirements.txt` contains `fastapi`, `uvicorn`, `xgboost`, `scikit-learn`, `ConfigSpace`, `numpy`, `cma`, and `requests`.)*

## 2. Install Frontend Dependencies

Next, navigate into the `dashboard` directory and install the Node packages.

```bash
cd dashboard
npm install
```

---

## Running the Suite

Because the backend engine explicitly checks if the telemetry server is alive before it starts broadcasting, **you must start the API server first**.

### Terminal 1: The API Server
This acts as the Server-Sent Events (SSE) router.
```bash
cd EvoSynth
source venv/bin/activate
python api_server.py
```
*Expected Output: `Starting API Server on http://localhost:8000`*

### Terminal 2: The Telemetry Dashboard
This boots the React UI.
```bash
cd EvoSynth/dashboard
npm run dev
```
*Expected Output: `VITE v4.x.x ready in xxx ms` (Available at `http://localhost:5173`)*

### Terminal 3: Fire the Engine
This triggers the UCB1 Multi-Armed Bandit meta-optimizer to begin evaluating XGBoost models.
```bash
cd EvoSynth
source venv/bin/activate
python main.py
```

Once `main.py` is running, open `http://localhost:5173` in your browser. You will instantly see the live telemetry stream populate the glass-box dashboard!
