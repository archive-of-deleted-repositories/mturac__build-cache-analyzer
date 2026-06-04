# cache-analyze

Analyze build cache hit rates and identify cache-busting patterns.

## Steps

### 1. Detect Build System

Check project root for config files:

| Build System | Config File | Cache Location |
|---|---|---|
| Turborepo | `turbo.json` | `.turbo/` |
| Nx | `nx.json` | `.nx/cache/` |
| Next.js | `next.config.js` | `.next/cache/` |
| Webpack | `webpack.config.js` | `.cache/` |
| Vite | `vite.config.ts` | `node_modules/.vite/` |
| Gradle | `build.gradle` | `.gradle/caches/` |

A project may use multiple systems simultaneously.

### 2. Turborepo Analysis

Read `.turbo/runs/*.json` log files:
- Extract per-task: status (HIT/MISS), hash, duration, dependencies
- Accumulate: `task -> { hits, misses, total_duration_ms }`
- Fallback: parse `turbo run` stderr for `[cached]` / `[noncached]` markers

### 3. Nx Analysis

Read `.nx/cache/` directory entries:
- For each task in `nx.json` targetDefaults: check if cached output exists
- Compare cache entry timestamp vs source file mtimes
- HIT = cache newer than all source inputs; MISS = any source newer

Parse `nx run-many --output-style=compact` stdout for cache status lines.

### 4. Webpack / Vite Analysis

For each build output file:
- Find corresponding source files via import graph
- Compare modification timestamps
- source mtime > output mtime → MISS; otherwise HIT
- For Webpack persistent cache: check `cache.identifier` and `cache.version` fields in config

### 5. Calculate Hit Rate

```
hit_rate = (total_hits / total_runs) * 100
```

Compute per-task rates separately. Flag any task below 50% as a priority.

### 6. Find Cache Busters

```
for each source file in git history (last 30 days or last 50 commits):
  count = number of commits touching the file
  correlate with cache MISS timestamps
```

Classify cache busters:
- **Lock files** — `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`
- **Generated files** committed to repo
- **Files outside declared inputs** that silently affect output

### 7. Output

```markdown
## Cache Analysis Results

**Overall Hit Rate:** 73.2%

### Tasks with Worst Hit Rates
| Task       | Hit Rate | Runs | Hits | Misses |
|------------|----------|------|------|--------|
| build:app  |    45.0% |   20 |    9 |     11 |
| test:unit  |    62.5% |   16 |   10 |      6 |
| lint       |    89.0% |   19 |   17 |      2 |

### Top 10 Cache-Busting Files
| File                | Times Changed | Misses Caused |
|---------------------|---------------|---------------|
| package-lock.json   |            18 |            15 |
| src/utils/shared.ts |            12 |            12 |
| .env.production     |             8 |             8 |

Run `/cache-optimize` for specific fixes.
```
