# The SSE Protocol

EvoSynth utilizes **Server-Sent Events (SSE)** for unidirectional, real-time data streaming from the Python backend to the React frontend.

Unlike WebSockets, which are bi-directional and complex to manage, SSE operates entirely over standard HTTP. The dashboard opens a single persistent `GET` request to `/stream`, and the server holds the connection open, pushing textual data as it arrives.

## The Event Lifecycle

1. **Engine events:** As the Meta-Engine works, the `EventReporter` turns each decision and score into a small JSON event (`reporter.emit(type, data)`). Every event is also appended to the run folder's `events.jsonl`, so a run can be inspected afterwards without the dashboard.
2. **FastAPI `POST /event`:** If the telemetry server is reachable (the reporter re-checks periodically, so the server can be started mid-run), a daemon thread POSTs the event to it without blocking the search.
3. **Asyncio Queue:** FastAPI receives the POST and pushes the JSON string into an `asyncio.Queue`.
4. **SSE `GET /stream` Generator:** A background async generator yields data from the queue to connected HTTP clients using the `text/event-stream` format.

## Event Payloads

Every event has a `type` and a `data` object.

### `step`
Fired when the engine enters a new stage (search-space definition, initial population, tier split, Tier 1 competition, promotion, Tier 2 loop).
```json
{ "type": "step", "data": { "num": 2, "name": "Tier Split (4-Way)" } }
```

### `progress`
The evaluation budget used so far.
```json
{ "type": "progress", "data": { "evals": 120, "budget": 500 } }
```

### `metric`
Fired whenever an algorithm finds a new best score.
```json
{ "type": "metric", "data": { "algo": "PSO", "accuracy": 86.42 } }
```

### `composite`
One per island at every scoring call: its UCB score and diversity.
```json
{ "type": "composite", "data": { "algo": "T1_PSO", "composite": 0.721, "diversity": 0.337 } }
```

### `tier_state`
Which algorithms are in which tier, after every promotion, SWAP or rollback.
```json
{ "type": "tier_state", "data": { "T1": ["DE", "GA"], "T2": ["PSO", "CMAES"] } }
```

### `switch`
A meta-controller intervention: `Promotion`, `Swap`, `Rollback`, `Injection` or `Elite Sharing`.
```json
{ "type": "switch", "data": { "action": "Injection", "algo": "T2_GA", "details": "Replaced bottom 3 due to stagnation" } }
```

### `log`
Lines of the engine's console output, for the dashboard's log panel.
