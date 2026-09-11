# Dragon-Jupyter

DragonHPC Jupyter Notebook workload template for Terra. Launches a persistent
Dragon frontend StatefulSet running a Jupyter Notebook server (via the
`dragon-jupyter` command) plus `nnodes` backend compute pods (Deployment) that
form a distributed runtime for Python multiprocessing, HPC, and LLM workloads.

## Launch Flow

The frontend StatefulSet runs `launch.sh`:

1. Installs Dragon runtime deps (native libs + `numpy`/`kubernetes`/`dragonhpc` pip packages)
2. Runs `dragon-jupyter` — starts a Jupyter Notebook server backed by the
   Dragon distributed runtime. Dragon then creates the backend Deployment
   (one pod per node) and wires the overlay network.

Backend pods run `backend.sh`: pip-install `dragonhpc`, then `exec dragon-backend`.

Unlike the batch-oriented `dragonhpc` plugin, the frontend is a StatefulSet so
the Jupyter Notebook server persists across restarts. The backend compute nodes
are a Deployment rather than a Job.

Notebooks are started and run inside Jupyter itself — work happens against the
Dragon distributed runtime that the frontend and backend pods form.

## Fields

- `nnodes` — number of backend Dragon compute nodes (Deployment replicas)
- `gpu` — attach a GPU; applies to backend compute pods (`runtimeClassName: nvidia`)
- `registry` / `repo` / `tag` — image used for both frontend and backend pods
  (defaults to `quay.io/jupyter/datascience-notebook:lab-4.4.9`)
- `cpu` / `memory` / `cpuLimit` / `memoryLimit` — per-pod resources

## Runtime

Frontend and backend pods run as root (`securityContext: runAsUser: 0`) under
dedicated ServiceAccounts with
namespace-scoped Roles: frontend can create/watch Deployments and list/watch
pods; backend can get/list/patch its own pods. The Jupyter server is exposed
via a ClusterIP Service and nginx Ingress with Hubble authentication at
`/<namespace>/jupyter/<name>/`. The embedded workload chart lives in
`scripts/chart/` and is rendered by Kuiper at launch time.