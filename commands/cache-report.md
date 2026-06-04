# /cache-report

Generate a weekly or monthly cache efficiency report with cost estimates.

## Usage

```
/cache-report
/cache-report --period month
/cache-report --rate 0.016
```

## Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--period` | `week` | Report period: `week` or `month` |
| `--rate` | `0.008` | CI cost per minute in USD (default: GitHub Actions Linux runner) |
| `--compare` | true | Compare against previous period |

## What it does

Invokes `skills/cache-report` to:
1. Collect run history from `.turbo/runs/` or CI logs
2. Calculate total CI minutes spent vs saved by cache
3. Apply cost rate to estimate dollar savings
4. Compare hit rate vs previous period (IMPROVING / STABLE / DEGRADING)
5. List top tasks that ran uncached most often

## Example output

```
Cache Efficiency Report — Week of 2024-01-15

Hit Rate: 73.2% (+5.1pp vs last week) IMPROVING

This Week   Last Week
CI minutes spent:   92.4      112.0
CI minutes saved:   68.1       51.3
Estimated savings:  $0.54      $0.41

Top offenders (most uncached):
  build:app   18 uncached runs   avg 4m 12s
  test:e2e    14 uncached runs   avg 8m 05s
```
