# GPU Fundamentals

Learn GPU programming fundamentals on the home RTX 5070. Work through
*[Programming Massively Parallel Processors](https://shop.elsevier.com/books/programming-massively-parallel-processors/hwu/978-0-443-43900-1)*,
by Hwu, Kirk, and Hajj; edit through VS Code Remote SSH and run CUDA, PyTorch,
profiling, and notebook workloads in the pinned NVIDIA NGC container.

## Start here

1. Follow the [home GPU workspace guide](docs/home/workspace.md).
2. Choose the current chapter in *Programming Massively Parallel Processors*.
3. Start with the related [curriculum](curriculum/README.md).

Open the configured server from the Mac:

```sh
infra/home/scripts/open-workspace
```

In the Remote SSH window, **Cmd+Shift+B** starts the runtime if necessary and
runs the active Python file on the RTX 5070. GPU code runs in the container,
not in the host's Pylance analysis environment.

## Useful places

- [Home GPU workspace](docs/home/workspace.md) — canonical editor, runtime,
  notebook, and Git workflow.
- [Home infrastructure](infra/home/) — pinned image, Compose service, scripts,
  observability, and operational rules.
- [Curriculum](curriculum/README.md) — exercises and artifacts.
- [Learning tools](learning-tools/README.md) — reusable interactive visualizations.
- [Today I Learned](TIL.md) — notes from the journey.
- [Scripts](scripts/README.md) — other local helpers.

## Future scale-up

The home workflow is intentionally primary. If an exercise later needs more
than 12 GB VRAM, a different accelerator, or multiple GPUs, adapt a Brev or
other cloud environment to the proven home interface. The existing Brev
automation remains under `infra/brev/` as a reference, but it is not used by
the default VS Code tasks.
