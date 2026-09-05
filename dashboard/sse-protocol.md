# The SSE Protocol

EvoSynth utilizes **Server-Sent Events (SSE)** for unidirectional, real-time data streaming from the Python backend to the React frontend.

Unlike WebSockets, which are bi-directional and complex to manage, SSE operates entirely over standard HTTP. The dashboard opens a single persistent `GET` request to `/stream`, and the server holds the connection open, pushing textual data as it arrives.

## The Event Lifecycle

1. **Python `sys.stdout` Interception:** The `EventReporter` class in the Meta-Engine redirects `stdout`. Whenever the engine prints a log, the reporter parses it to see if it contains a parseable event (like `[EVENT]`).
2. **FastAPI `POST /event`:** If an event is found, a daemon thread fires an HTTP POST request containing a JSON payload to the FastAPI server.
3. **Asyncio Queue:** FastAPI receives the POST and pushes the JSON string into an `asyncio.Queue`.
4. **SSE `GET /stream` Generator:** A background async generator infinitely yields data from the queue to connected HTTP clients using the `text/event-stream` format.

## Event Payloads

The React dashboard expects JSON payloads with the following structure:

### Initialization Event
Fired when the engine boots. Contains the budget and search space definitions.
```json
{
  "type": "init",
  "data": {
    "dimensions": 7,
    "budget": 2000,
    "algorithms": ["GA", "PSO", "DE", "CMAES"]
  }
}
```

### Iteration Tick
Fired every $K$ evaluations. Updates the charts and tier statuses.
```json
{
  "type": "tick",
  "data": {
    "iteration": 5,
    "global_best": 98.07,
    "algorithms": {
      "GA": { "best": 97.89, "ucb_score": 0.5 },
      "CMAES": { "best": 98.42, "ucb_score": 0.95 }
    },
    "tier1": ["GA", "PSO"],
    "tier2": ["DE", "CMAES"]
  }
}
```

### Final Result
Fired when the budget is exhausted.
```json
{
  "type": "finish",
  "data": {
    "winning_algorithm": "DE",
    "final_accuracy": 98.59,
    "best_params": {
      "learning_rate": 0.1704,
      "max_depth": 9,
      ...
    }
  }
}
```
