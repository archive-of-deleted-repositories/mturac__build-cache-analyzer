# cache-report

Generate weekly or monthly cache efficiency reports with cost estimates.

## Steps

### 1. Collect Run History

Source data (in priority order):
- `.turbo/runs/*.json` — Turborepo run logs with timestamps and durations
- CI provider API (GitHub Actions, CircleCI, GitLab CI) — if token available
- Local git log + build output timestamps as fallback

Group runs by calendar week or month depending on `--period` flag (default: week).

### 2. Calculate Efficiency Metrics

For each period:
```
total_task_runs       = all task executions
cached_runs           = runs with HIT status
hit_rate              = cached_runs / total_task_runs * 100

time_per_cached_run   = average duration of cached runs (near 0 for turbo)
time_per_fresh_run    = average duration of uncached runs

ci_minutes_spent      = sum(actual duration of all runs)
ci_minutes_saved      = cached_runs * average_fresh_run_duration
```

### 3. Cost Estimate

Apply team rate to minutes saved:

```
default_rate = $0.008 per CI minute (GitHub Actions Linux runner)
estimated_savings = ci_minutes_saved * rate
```

If `--rate` flag provided, use that value instead.

### 4. Trend Analysis

Compare current period vs previous period:
- hit_rate delta: +/- percentage points
- Label: IMPROVING (delta > +2pp), STABLE (-2pp to +2pp), DEGRADING (delta < -2pp)

### 5. Top Offenders

List tasks sorted by uncached runs descending:
```
| Task       | Uncached Runs | % of Total Misses | Avg Duration |
|------------|--------------|-------------------|--------------|
| build:app  |           42 |              38%  |       4m 12s |
| test:e2e   |           31 |              28%  |       8m 05s |
```

### 6. Output

```markdown
## Cache Efficiency Report — Week of 2024-01-15

**Hit Rate:** 73.2% (+5.1pp vs last week) ✓ IMPROVING

### Summary
| Metric                | This Week | Last Week |
|-----------------------|-----------|-----------|
| Total task runs       |       148 |       132 |
| Cache hits            |       108 |         89 |
| CI minutes spent      |      92.4 |      112.0 |
| CI minutes saved      |      68.1 |       51.3 |
| **Estimated savings** |  **$0.54** |  **$0.41** |

### Top Offenders (most uncached)
| Task       | Uncached Runs | Avg Duration |
|------------|--------------|--------------|
| build:app  |           18 |       4m 12s |
| test:e2e   |           14 |       8m 05s |
| type-check |            8 |       1m 30s |

Run `/cache-optimize` to address the top offenders.
```
