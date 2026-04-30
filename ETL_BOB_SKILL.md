# ETL — Bob Job Patterns & Skill Guide

> Skill file for AI agents generating or modifying ETL/Bob job code.

## Job Anatomy

Every Bob job follows the EVTL pattern:

```
Extract  →  Validate  →  Transform  →  Load
   │             │            │           │
 Source       Schema       Pure Fn     Hydra
 I/O only    checks      no I/O       write
```

---

## Standard Job Template

```python
# jobs/your_job_name.py
"""
Job: <job_name>
Schedule: <cron expression>
Source: <data source description>
Target: Hydra table `<table_name>`, partition by date
Owner: <team/person>
"""

import logging
from datetime import date
from typing import Any
import pandas as pd

from transforms.<module> import transform_records
from models.hydra_writer import write_partition
from utils.validation import validate_schema
from utils.metrics import emit_job_metric

logger = logging.getLogger(__name__)

SCHEMA = {
    "id": "int64",
    "created_at": "datetime64[ns]",
    "value": "float64",
}

def extract(run_date: date) -> list[dict[str, Any]]:
    """Pull raw records for run_date from source system."""
    # TODO: implement source-specific extraction
    raise NotImplementedError

def run(run_date: date | None = None) -> dict:
    """Entry point. Returns job result summary."""
    run_date = run_date or date.today()
    logger.info(f"[{__name__}] Starting for {run_date}")

    # 1. Extract
    raw = extract(run_date)
    logger.info(f"Extracted {len(raw)} records")

    # 2. Validate raw
    assert raw, f"No records extracted for {run_date}"

    # 3. Transform (pure function — no I/O)
    df = transform_records(raw)

    # 4. Validate schema
    validate_schema(df, SCHEMA)

    # 5. Load
    partition = str(run_date)
    write_partition("your_table_name", partition, df)

    result = {"date": partition, "rows_written": len(df)}
    emit_job_metric(__name__, result)
    logger.info(f"Done: {result}")
    return result


if __name__ == "__main__":
    run()
```

---

## Transform Functions — Rules

```python
# transforms/revenue.py
# Rule: pure functions only. No I/O, no DB calls, no logging side effects.

import pandas as pd

def transform_records(raw: list[dict]) -> pd.DataFrame:
    df = pd.DataFrame(raw)
    df["created_at"] = pd.to_datetime(df["created_at"], utc=True)
    df["value"] = pd.to_numeric(df["value"], errors="coerce").fillna(0.0)
    return df[["id", "created_at", "value"]]  # explicit column select
```

---

## Error Handling Patterns

```python
# Retry-able extraction error — let scheduler handle retry
raise ConnectionError(f"Source unavailable: {e}")

# Data quality error — stop job, alert
raise ValueError(f"Schema mismatch on column 'id': {actual} vs expected int64")

# Partial data warning — write what we have, log gap
logger.warning(f"Only {len(df)} of expected ~{EXPECTED_COUNT} records")
```

---

## Testing Pattern

```python
# tests/test_transform_revenue.py
import pytest
from transforms.revenue import transform_records

def test_transform_happy_path():
    raw = [{"id": 1, "created_at": "2024-01-01T00:00:00Z", "value": "42.5"}]
    df = transform_records(raw)
    assert df["value"].iloc[0] == 42.5
    assert df["id"].iloc[0] == 1

def test_transform_null_value_defaults_to_zero():
    raw = [{"id": 2, "created_at": "2024-01-01T00:00:00Z", "value": None}]
    df = transform_records(raw)
    assert df["value"].iloc[0] == 0.0
```

---

## AI Agent Prompt Snippet

```
Context: We use Bob ETL jobs to load into Hydra.
Rules:
- All jobs follow Extract → Validate → Transform → Load
- Transforms are PURE functions (no I/O, no side effects)
- Validate schema before writing to Hydra
- Emit a job metric dict at the end of each run
- Use structured logging with logger.info / logger.warning / logger.error
- Entry point is always run(run_date) for testability
```
