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
