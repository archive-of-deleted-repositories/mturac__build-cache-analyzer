# /cache-optimize

Suggest and apply cache optimizations for this project's build system.

## Usage

```
/cache-optimize
/cache-optimize --apply
/cache-optimize --system turbo
```

## Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--apply` | false | Write suggested fixes directly to config files (asks for confirmation) |
| `--system` | auto-detect | Target a specific build system |
| `--dry-run` | false | Show diffs without writing anything |

## What it does

Invokes `skills/cache-optimize` to:
1. Audit `inputs`/`outputs` config for the detected build system
2. Detect non-deterministic outputs (timestamps, absolute paths, random IDs)
3. Find generated files incorrectly included in tracked outputs
4. Produce a prioritized fix list with estimated hit rate improvement per fix

Without `--apply`, prints recommendations only. With `--apply`, shows a diff and asks for confirmation before modifying `turbo.json`, `nx.json`, or `.nxignore`.

## Example output

```
Cache Optimization Recommendations

HIGH IMPACT (+25% hit rate):
  1. Narrow turbo.json `build.inputs` — currently ["**/*"]
     Suggested: ["src/**", "tsconfig.json", "!**/*.test.*"]

  2. Create .nxignore — docs/ changes causing 34% of all misses

MEDIUM IMPACT (+10% hit rate):
  3. Set webpack output.pathinfo: false to remove absolute paths from bundles

Run with --apply to write these changes.
```
