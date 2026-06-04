# cache-optimize

Suggest specific, actionable cache optimizations based on analysis results.

## Steps

### 1. Turborepo: Audit `inputs` Configuration

Read `turbo.json` for each pipeline task's `inputs` array.

**Overly broad inputs (every file busts cache):**
```json
"build": { "inputs": ["**/*"] }
```
Fix — narrow to actual source dependencies:
```json
"build": {
  "inputs": ["src/**", "tsconfig.json", "package.json", "!src/**/*.test.*", "!src/**/*.spec.*"]
}
```

**Inputs too narrow (missing implicit deps):**
- For each recent cache MISS, diff files changed vs declared inputs
- Any changed file NOT in inputs that correlates with the miss → add to inputs

### 2. Nx: Audit `implicitDependencies`

Check `nx.json` and per-project `project.json`.

**Overly broad:**
```json
{ "implicitDependencies": ["*"] }
```
Fix — list only actual deps:
```json
{ "implicitDependencies": ["shared-utils", "config"] }
```

**Generate `.nxignore`** for files that never affect build output:
```
*.md
.github/**
docs/**
*.log
coverage/**
```

### 3. Detect Non-Deterministic Outputs

Scan output dirs for patterns that break cache determinism:

| Pattern | Regex | Fix |
|---|---|---|
| Timestamps | `\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}` | Normalize or strip |
| Absolute paths | `/home/[a-z]+/` or `C:\\Users\\` | Use relative paths |
| Random IDs | unique hex per build in non-hash context | Content-hash instead |
| Env-specific values | `staging.example.com`, `PROD_SECRET` | Inject at runtime |
| Build machine PIDs | `PID: \d+` | Remove from output |

Detection method: build twice with identical inputs, byte-diff outputs. Any difference = non-determinism.

### 4. Auto-Generated Files in Outputs

Check `outputs` config for each task. Flag any file that:
- Contains a current timestamp in its header
- Is regenerated identically each run but recorded as output
- Is a `.d.ts` declaration that includes machine-local paths

Recommendation: exclude via `!path/to/generated/**` in `outputs`.

### 5. Output — Prioritized Fix List

```markdown
## Cache Optimization Recommendations

### High Impact (+25% estimated hit rate)

1. **Narrow Turborepo `build` inputs**
   File: `turbo.json`
   Change: `"inputs": ["**/*"]` → `["src/**", "tsconfig.json", "!**/*.test.*"]`

2. **Add `.nxignore` for docs/**
   Cause: 34% of misses correlated with docs/ changes
   Fix: create `.nxignore` containing `docs/**`

### Medium Impact (+10% estimated hit rate)

3. **Remove absolute paths from Webpack output**
   File: `webpack.config.js`
   Change: set `output.pathinfo: false` and `devtool: false` in production

4. **Add pnpm-lock.yaml to Turborepo `inputs` for install task only**
   Currently causes full rebuild on every lock update across all tasks
```
