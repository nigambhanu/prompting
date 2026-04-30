# System Architecture — Analytics Platform

> **AI Agent Context File** — Include this in every agent/plan session so the AI understands the full system before generating code.

## Stack Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        DATA FLOW                                │
│                                                                 │
│  [Source Data] ──► [Bob ETL Job] ──► [Hydra DB] ──► [API] ──► [UI]
│                         │                │            │         │
│                    Python Scripts    File-based    Bottle /   React
│                    + Schedulers      Storage      Tornado    TypeScript
└─────────────────────────────────────────────────────────────────┘
```

## Component Responsibilities

### 1. Bob ETL Job (Data Layer)
- **Purpose**: Ingest, transform, and load raw data into Hydra
- **Language**: Python
- **Triggers**: Scheduled (cron / task queue)
- **Key patterns**: Extract → Validate → Transform → Load (EVTL)
- **Error handling**: Dead-letter queue or retry table in Hydra
- **Observability**: Log to structured JSON; emit job metrics

### 2. Hydra (Storage Layer)
- **Type**: File-based database
- **Access pattern**: Python SDK / direct file reads
- **Schema ownership**: ETL team owns schemas; API team is consumer
- **Partitioning**: Time-partitioned where applicable (YYYY/MM/DD)
- **Consistency**: Eventual — API must handle stale reads gracefully

### 3. Backend API (Service Layer)
- **Frameworks**: Python Bottle (lightweight routes) / Tornado (async/websocket)
- **Pattern**: RESTful JSON; async handlers for long-running queries
- **Auth**: [define your auth mechanism here]
- **Response envelope**:
  ```json
  { "status": "ok|error", "data": {}, "meta": { "ts": "", "version": "" } }
  ```
- **Error codes**: Use HTTP semantics; never 200 for errors

### 4. React TypeScript UI (Presentation Layer)
- **Framework**: React + TypeScript
- **State**: [Redux / Zustand / React Query — specify yours]
- **API calls**: Centralised in `/src/api/` — never inline fetch
- **Types**: Mirror backend response shapes exactly; use Zod for runtime validation
- **Data refresh**: Polling interval or websocket subscription (Tornado)

---

## Directory Conventions

```
/project-root
├── etl/                  # Bob job scripts
│   ├── jobs/             # Individual job definitions
│   ├── transforms/       # Reusable transform functions
│   └── tests/
├── api/                  # Bottle / Tornado backend
│   ├── routes/
│   ├── services/         # Business logic (no HTTP here)
│   ├── models/           # Hydra access objects
│   └── tests/
├── ui/                   # React TypeScript app
│   ├── src/
│   │   ├── api/          # API client layer
│   │   ├── components/
│   │   ├── pages/
│   │   ├── hooks/
│   │   ├── types/        # Shared TypeScript types
│   │   └── utils/
│   └── tests/
└── docs/                 # ADRs, skill MDs, runbooks
```

---

## Key Cross-Cutting Rules

1. **No business logic in ETL transforms** — transforms are pure data reshaping
2. **API services never import Hydra directly** — go through model layer
3. **UI components never call API directly** — use hooks in `/hooks/`
4. **All datetimes**: UTC storage, local display via UI
5. **Feature flags**: Controlled in API config, consumed by UI via `/api/config` endpoint

---

## AI Agent Instructions

When generating code for this project:
- Match the directory conventions above
- Follow the response envelope pattern for any new API endpoints
- TypeScript types must be generated from actual API response shapes
- ETL transforms must be pure functions (no side effects, no I/O)
- Always add a `# type: ignore` comment justification if bypassing mypy
