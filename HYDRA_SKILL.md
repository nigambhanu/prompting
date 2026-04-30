# Hydra DB — Patterns & Skill Guide

> **Skill file for AI agents and developers.** Reference when reading/writing to Hydra.

## What is Hydra (in this project)

Hydra is a **file-based database** — data lives as structured files on disk (Parquet, CSV, JSON lines, or binary). Think of it as an organised filesystem with a query layer on top.

---

## Core Read Patterns

### Basic Query
```python
import hydra  # or your internal Hydra SDK

def get_dataset(name: str, partition: str | None = None):
    """Load a dataset, optionally scoped to a partition."""
    db = hydra.open(config.HYDRA_PATH)
    table = db.table(name)
    if partition:
        table = table.partition(partition)
    return table.to_pandas()
```

### Time-Partitioned Read
```python
def get_range(table_name: str, start_date: str, end_date: str):
    """Read data between two dates (YYYY-MM-DD)."""
    db = hydra.open(config.HYDRA_PATH)
    return (
        db.table(table_name)
          .filter(date__gte=start_date, date__lte=end_date)
          .to_pandas()
    )
```

### Stale-Read Guard (API layer)
```python
import time

MAX_STALENESS_SECONDS = 300  # 5 minutes

def read_with_staleness_check(table_name: str) -> dict:
    db = hydra.open(config.HYDRA_PATH)
    meta = db.table(table_name).metadata()
    age = time.time() - meta["last_written_at"]
    if age > MAX_STALENESS_SECONDS:
        logger.warning(f"Stale data: {table_name} is {age:.0f}s old")
    return db.table(table_name).to_dict()
```

---

## Core Write Patterns (ETL only)

### Atomic Write (Bob Job)
```python
def write_partition(table_name: str, partition: str, df):
    """Write a partition atomically — write to .tmp then rename."""
    db = hydra.open(config.HYDRA_PATH)
    db.table(table_name).write(
        df,
        partition=partition,
        mode="overwrite",   # always full partition replace
        validate_schema=True
    )
```

### Schema Validation Before Write
```python
EXPECTED_COLUMNS = {"id": "int64", "ts": "datetime64[ns]", "value": "float64"}

def validate_schema(df):
    for col, dtype in EXPECTED_COLUMNS.items():
        assert col in df.columns, f"Missing column: {col}"
        assert str(df[col].dtype) == dtype, f"Wrong type for {col}: {df[col].dtype}"
```

---

## Anti-Patterns — Never Do These

| ❌ Anti-pattern | ✅ Correct approach |
|----------------|-------------------|
| Read entire table in API request | Filter by partition/date first |
| Write from API layer | ETL job owns all writes |
| Partial partition write | Always write complete partitions |
| Raw file path access in UI code | Go through API endpoint |
| Ignore `last_written_at` | Always check staleness in API |

---

## Naming Conventions

```
Tables:     snake_case           e.g.  daily_revenue_summary
Partitions: YYYY-MM-DD           e.g.  2024-03-15
Columns:    snake_case           e.g.  customer_id, created_at
Temp files: <name>.tmp.parquet   (auto-cleaned by write layer)
```

---

## AI Agent Prompt Snippets

When asking AI to write Hydra code, prepend:

```
Context: Hydra is a file-based DB accessed via Python SDK.
- Always filter by partition before loading to pandas
- Writes are ETL-only; never write from API or UI layers
- Use atomic write pattern (overwrite full partition)
- Check data staleness; log warnings if > 5 min old
```
