# UI — React TypeScript Patterns & Skill Guide

> Skill file for AI agents and UI developers writing React TypeScript code.

## Layer Rules

```
Page Component  →  Feature Component  →  Hook  →  API Client  →  Backend
(routing/layout)   (UI logic)         (state)    (/src/api/)   (Bottle/Tornado)
```

Never call `fetch` directly in a component. Never put API URLs in JSX.

---

## Type Definitions — Mirror the Backend Envelope

```typescript
// src/types/api.ts
export interface ApiResponse<T> {
  status: "ok" | "error";
  data?: T;
  error?: { message: string; code: number };
  meta: { ts: number; version: string };
}

// src/types/metrics.ts
export interface MetricsSummary {
  total: number;
  count: number;
  date: string | null;
}
```

---

## API Client Layer

```typescript
// src/api/metricsApi.ts
import type { ApiResponse, MetricsSummary } from "../types";

const BASE = import.meta.env.VITE_API_BASE_URL;

export async function fetchMetrics(date: string): Promise<MetricsSummary> {
  const res = await fetch(`${BASE}/api/metrics/${date}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  
  const json: ApiResponse<MetricsSummary> = await res.json();
  if (json.status === "error") throw new Error(json.error?.message ?? "API error");
  if (!json.data) throw new Error("No data in response");
  
  return json.data;
}
```

---

## Data-Fetching Hook Pattern

```typescript
// src/hooks/useMetrics.ts
import { useState, useEffect } from "react";
import { fetchMetrics } from "../api/metricsApi";
import type { MetricsSummary } from "../types";

interface UseMetricsResult {
  data: MetricsSummary | null;
  isLoading: boolean;
  error: string | null;
  refetch: () => void;
}

export function useMetrics(date: string): UseMetricsResult {
  const [data, setData] = useState<MetricsSummary | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [tick, setTick] = useState(0);

  useEffect(() => {
    let cancelled = false;
    setIsLoading(true);
    setError(null);

    fetchMetrics(date)
      .then((d) => { if (!cancelled) setData(d); })
      .catch((e) => { if (!cancelled) setError(e.message); })
      .finally(() => { if (!cancelled) setIsLoading(false); });

    return () => { cancelled = true; };
  }, [date, tick]);

  return { data, isLoading, error, refetch: () => setTick((t) => t + 1) };
}
```

---

## Component Pattern

```tsx
// src/components/MetricsCard.tsx
import { useMetrics } from "../hooks/useMetrics";

interface MetricsCardProps {
  date: string;
}

export function MetricsCard({ date }: MetricsCardProps) {
  const { data, isLoading, error, refetch } = useMetrics(date);

  if (isLoading) return <div className="skeleton" aria-busy="true" />;
  if (error) return (
    <div className="error-state" role="alert">
      <p>{error}</p>
      <button onClick={refetch}>Retry</button>
    </div>
  );

  return (
    <article className="metrics-card">
      <h2>Metrics — {date}</h2>
      <dl>
        <dt>Total</dt><dd>{data?.total.toLocaleString()}</dd>
        <dt>Count</dt><dd>{data?.count.toLocaleString()}</dd>
      </dl>
    </article>
  );
}
```

---

## Naming Conventions

| Thing | Convention | Example |
|-------|-----------|---------|
| Components | PascalCase | `MetricsCard.tsx` |
| Hooks | camelCase, `use` prefix | `useMetrics.ts` |
| API functions | camelCase, verb prefix | `fetchMetrics`, `postReport` |
| Type interfaces | PascalCase | `MetricsSummary` |
| Constants | SCREAMING_SNAKE | `MAX_RETRIES` |

---

## Anti-Patterns

| ❌ Don't | ✅ Do instead |
|---------|-------------|
| `fetch` inside JSX | Use hook + API client |
| `any` type | Define exact types from API shape |
| Hardcode API URLs | Use `import.meta.env.VITE_API_BASE_URL` |
| State in page component | Move to hook |
| `console.log` in prod code | Use a logger util |

---

## AI Agent Prompt Snippet

```
Context: React TypeScript UI consuming a Python REST API.
Rules:
- API calls live in /src/api/ — never inline fetch in components
- All data fetching goes through custom hooks in /src/hooks/
- TypeScript types mirror the backend ApiResponse<T> envelope exactly
- Components receive data via props or hooks; zero direct API calls
- Always handle loading, error, and empty states explicitly
- Use discriminated unions for state: { status: 'loading' | 'error' | 'ok' }
```
