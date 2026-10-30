# Home GPU server

The default GPU development target is `gpu-home`, with the checkout at
`/home/trey/workspace/gpu-fundamentals`. Use VS Code Remote SSH for editing and
the Compose runtime in this directory for every CUDA, PyTorch, Triton, notebook,
and profiling command. Do not run GPU code in the host analysis environment.

Jupyter and monitoring endpoints must remain bound to remote loopback. Never
open router ports or expose tokenless Jupyter directly to LAN or Tailscale.
Git is the only source synchronization mechanism; preserve tracked and
untracked learner work before pulling or changing branches.

The runtime must remain reproducible on the RTX 5070. Add operating-system or
runtime Python dependencies to `infra/home/Dockerfile`, then rebuild. Notebook
cells must not use `sudo` or install system packages. Detect the active GPU
compute capability instead of hard-coding an SM target.

Brev and other cloud targets are out of scope unless the user explicitly asks
to scale up. When asked, read `infra/brev/AGENTS.md`, preserve its cloud image
pin and cost controls, and adapt the cloud target to the established home
commands rather than weakening the home workflow.

## Platform versus workload ownership

The GPU Server Home Lab repository at
`/Users/treyshanks/workspace/gpu-server-home-lab` owns shared Ubuntu host
configuration, NVIDIA drivers, Docker/container runtime integration, access,
backups and shared operations tooling. Request a reviewed infrastructure change
there when a workload needs a host capability; do not directly change host
packages, services, drivers, storage, Docker daemon settings or network policy
as part of a curriculum task.

This curriculum owns its source, notebooks, tests, workload Dockerfile,
dependency locks and project-scoped Compose configuration. Add libraries to
those reproducible workload definitions; do not install them into host Python.
Keep its writable paths, volumes and container lifecycle separate from
Observable Recs. Never operate another project's containers or perform global
Docker cleanup. Coordinate GPU benchmarks, training and profiling with other
active lab work; do not assume a shared GPU provides performance isolation.

The user has chosen AGENTS.md guidance for responsibility boundaries for now.
Do not introduce separate accounts, rootless Docker, socket restrictions or a
custom agent security harness unless separately requested. These rules guide
agent behavior; they do not enforce OS-level isolation. Git is the only way to
propagate this instruction update to the server; do not copy it over SSH or
pull into a dirty remote checkout without review.
