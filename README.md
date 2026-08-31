# Terra-DragonHPC

Juno Innovations Terra Plugins accelerated with DragonHPC.

This repository contains Terra plugins optimized for DragonHPC accelerated computing plugins.
Plugins are consumed by Genesis at workload launch time via Kuiper, which renders
`scripts/chart/` at that time.

## Plugin Types

This repo supports three plugin types:

- **Namespaced** — deploys a workload into a user's project namespace
- **Cluster-level** — cluster-wide operator via ArgoCD Application, installed into `argocd` namespace
- **Workload template** — consumed by Genesis/Kuiper; installs a ConfigMap carrying an embedded Helm chart (`scripts/chart/`). Kuiper renders this chart at workload launch time using field values from `templates/metadata.yaml`.

## Quick Start

```sh
# Create a new workload template plugin
make new-plugin

# Follow interactive prompts for plugin name, type, and (for workloads) UI category

# After editing scripts/, repackage:
make package <plugin-name>

# Verify all packages are up to date
make verify

# Check ConfigMap size limit
make check-size <plugin-name>
```

## Critical Rules

1. **Repackage after changing `scripts/`** — `make package <plugin>` regenerates ConfigMaps. Skipping it deploys stale scripts.
2. **1MiB ConfigMap limit** — `make check-size <plugin>` warns at 900KB, errors at 1MiB.
3. **`metadata.yaml` field names must match `scripts/chart/values.yaml` keys** — misaligned names cause Helm rendering failures.
4. **Never edit generated files** — `packaged-scripts*.yaml` are always overwritten by `make package`.
5. **`terra.yaml` fields are install-time only** — shown in Terra app store; become Helm values at ArgoCD sync time.
6. **No workstation affinity/tolerances by default** — DragonHPC base scaffold omits these; sites may add their own per-plugin.

## Make Targets

| Target | Usage |
|--------|-------|
| `make new-plugin` | Interactive plugin scaffolding |
| `make package <name>` | Repackage scripts/ into ConfigMap YAML |
| `make verify` | Check all plugins have up-to-date packages |
| `make check-size <name>` | Check packaged size vs 1MiB limit |
| `make watch <name>` | Auto-repackage on scripts/ changes |
| `make lint` | Helm lint all charts |
| `make test <name>` | Deploy to local Kind cluster via ArgoCD |
| `make docs` | Serve docs site locally |

## Repository Structure

- `plugins/` — Directory containing all Terra plugins (populated via `make new-plugin`)
- `template/` — Scaffolding templates for all plugin types (namespaced, cluster, workload)
- `AGENTS.md` — Agent guidance rules
- `Makefile` — Build and packaging targets
- `README.md` — This file