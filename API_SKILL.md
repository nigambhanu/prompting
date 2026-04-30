# Backend API — Bottle / Tornado Patterns & Skill Guide

> Skill file for AI agents and backend developers writing API routes and services.

## Architecture Rule

```
Route Handler  →  Service Layer  →  Model Layer  →  Hydra
(HTTP only)      (business logic)   (data access)   (storage)
```

No route should contain business logic. No service should contain HTTP logic.

---

## Response Envelope (ALWAYS use this)

```python
# utils/response.py
import json
import time

def ok(data, meta: dict | None = None):
    return json.dumps({
        "status": "ok",
        "data": data,
        "meta": {"ts": time.time(), "version": "1.0", **(meta or {})}
    })

def error(message: str, code: int = 400):
    return json.dumps({
        "status": "error",
        "error": {"message": message, "code": code},
        "meta": {"ts": time.time()}
    })
```

---

## Bottle Route Pattern

```python
# routes/metrics.py
import bottle
from services.metrics_service import MetricsService
from utils.response import ok, error

app = bottle.Bottle()
svc = MetricsService()

@app.get("/api/metrics/<date>")
def get_metrics(date: str):
    try:
        result = svc.get_for_date(date)
        bottle.response.content_type = "application/json"
        return ok(result)
    except ValueError as e:
        bottle.response.status = 400
        bottle.response.content_type = "application/json"
        return error(str(e))
    except Exception as e:
        bottle.response.status = 500
        bottle.response.content_type = "application/json"
        return error("Internal server error")
```

---

## Tornado Async Handler Pattern

```python
# routes/metrics_async.py
import tornado.web
from services.metrics_service import MetricsService
from utils.response import ok, error
import json

class MetricsHandler(tornado.web.RequestHandler):
    def initialize(self, service: MetricsService):
        self.svc = service

    async def get(self, date: str):
        try:
            result = await self.svc.get_for_date_async(date)
            self.set_header("Content-Type", "application/json")
            self.write(ok(result))
        except ValueError as e:
            self.set_status(400)
            self.write(error(str(e), 400))
        except Exception:
            self.set_status(500)
            self.write(error("Internal server error", 500))
```

---

## Service Layer Pattern

```python
# services/metrics_service.py
from models.metrics_model import MetricsModel

class MetricsService:
    def __init__(self):
        self.model = MetricsModel()

    def get_for_date(self, date: str) -> dict:
        if not self._valid_date(date):
            raise ValueError(f"Invalid date format: {date}. Use YYYY-MM-DD")
        df = self.model.load(date)
        return self._to_summary(df)

    def _valid_date(self, date: str) -> bool:
        import re
        return bool(re.match(r"^\d{4}-\d{2}-\d{2}$", date))

    def _to_summary(self, df) -> dict:
        return {
            "total": float(df["value"].sum()),
            "count": int(len(df)),
            "date": df["date"].iloc[0] if len(df) else None,
        }
```

---

## Model Layer Pattern

```python
# models/metrics_model.py
import config
from utils.hydra_client import HydraClient

class MetricsModel:
    TABLE = "daily_metrics"

    def __init__(self):
        self.db = HydraClient(config.HYDRA_PATH)

    def load(self, partition: str):
        return self.db.read(self.TABLE, partition=partition)
```

---

## Anti-Patterns

| ❌ Don't | ✅ Do instead |
|---------|-------------|
| Return raw dict without envelope | Always use `ok()` / `error()` |
| Put query logic in route handler | Use service + model layers |
| Catch all exceptions silently | Log the exception before returning 500 |
| Return 200 for errors | Use correct HTTP status codes |
| Access Hydra directly in route | Go through model layer |

---

## AI Agent Prompt Snippet

```
Context: Backend uses Python Bottle (sync) and Tornado (async).
Rules:
- Always return the standard JSON envelope: {status, data, meta} or {status, error, meta}
- Three layers: Route (HTTP) → Service (logic) → Model (Hydra access)
- Services raise ValueError for bad input, Exception for system errors
- Routes catch exceptions and map to HTTP status codes
- Tornado handlers use async/await; Bottle handlers are sync
```
