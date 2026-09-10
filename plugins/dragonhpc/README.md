# DragonHPC

DragonHPC Python multiprocessing batch workload template for Terra. Launches a
Dragon frontend pod plus `nnodes` backend compute pods that form a distributed
runtime for Python multiprocessing, HPC, and LLM workloads.

## Launch Flow

The frontend pod runs `launch.sh`:

1. Verifies `DRAGON_ENTRYPOINT` is set (fails hard if missing)
2. `mkdir`/`cd` into the working directory (`DRAGON_WORKING_DIR`, default `/workspace`)
3. Clones `DRAGON_GIT_URL` into the working dir (optional)
4. Installs `DRAGON_REQUIREMENTS` via pip (optional)
5. Runs `dragon ${DRAGON_ENTRYPOINT}` — the Dragon runtime then creates the
   backend Job (one pod per node) and wires the overlay network

Backend pods run `backend.sh`: pip-install `dragonhpc`, then `exec dragon-backend`.

## Fields

- `nnodes` — number of backend Dragon compute nodes (backend Job parallelism/completions)
- `gpu` — attach a GPU; applies to backend compute pods (`runtimeClassName: nvidia`)
- `registry` / `repo` / `tag` — image used for both frontend and backend pods
- `cpu` / `memory` / `cpuLimit` / `memoryLimit` — per-pod resources

## Custom Environment Variables

| Name | Description |
|---|---|
| `DRAGON_ENTRYPOINT` | Python module/script to launch with dragon (required). Trailing args after the script path are forwarded to the child (word-split) |
| `DRAGON_WORKING_DIR` | Working directory for the launch script (default `/workspace`) |
| `DRAGON_GIT_URL` | Optional git repo to clone into the working dir |
| `DRAGON_REQUIREMENTS` | Path to requirements.txt to install before run |

## Runtime

Frontend and backend pods run under dedicated ServiceAccounts with
namespace-scoped Roles: frontend can create/watch Jobs and list/watch pods;
backend can get/list/patch its own pods. The embedded workload chart lives in
`scripts/chart/` and is rendered by Kuiper at launch time.