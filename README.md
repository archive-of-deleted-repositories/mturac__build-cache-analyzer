# build-cache-analyzer

Claude Code plugin that analyzes build cache hit rates across CI runs, finds what's busting your cache, and optimizes build times.

Supports Turborepo, Nx, Webpack, Vite, Next.js, and Gradle.

## Install

```bash
claude plugin install build-cache-analyzer
```

## Commands

| Command | Description |
|---------|-------------|
| `/cache-analyze` | Analyze current cache state and hit rates |
| `/cache-optimize` | Get specific fix suggestions for your build config |
| `/cache-report` | Weekly/monthly efficiency report with cost estimates |

## Example Report

```
Cache Analysis Results

Overall Hit Rate: 73.2%

Tasks with worst hit rates:
  build:app   45.0%  (9 hits / 20 runs)
  test:unit   62.5% (10 hits / 16 runs)
  lint        89.0% (17 hits / 19 runs)

Top 10 cache-busting files:
  package-lock.json    18 changes → 15 misses
  src/utils/shared.ts  12 changes → 12 misses
  .env.production       8 changes →  8 misses

Cache Optimization Recommendations

HIGH IMPACT (+25% hit rate):
  1. Narrow turbo.json build.inputs from ["**/*"] to ["src/**", "tsconfig.json"]
  2. Create .nxignore — docs/ changes causing 34% of all misses

Run /cache-optimize --apply to write these fixes.
```

## Skills

- `cache-analyze` — detects build system, parses cache logs, identifies busters
- `cache-optimize` — audits inputs/outputs config, detects non-determinism, generates fixes
- `cache-report` — weekly/monthly report with CI minute savings and cost estimates
