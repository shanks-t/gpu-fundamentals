# Home GPU development workspace

The RTX 5070 server is the canonical environment for GPU Fundamentals. VS Code
edits the host checkout through Remote SSH, while Docker Compose supplies the
pinned CUDA, PyTorch, compiler, and Jupyter runtime.

| Concern | Location |
| --- | --- |
| SSH target | `gpu-home` |
| Host checkout | `/home/trey/workspace/gpu-fundamentals` |
| Container checkout | `/workspace` |
| Pylance interpreter | `/home/trey/.venvs/gpu-fundamentals-analysis/bin/python` |
| Jupyter | Remote loopback `127.0.0.1:8889` |
| Model/framework caches | `/home/trey/.cache/gpu-lab` |

Git is the only source synchronization mechanism. Do not use Dev Containers,
rsync, or copy-based synchronization.

## Open the workspace

From the Mac clone, use the one-command launcher:

```sh
infra/home/scripts/open-workspace
```

It connects to `gpu-home`, refreshes the uv-managed analysis environment,
ensures the GPU runtime is healthy, reports the remote Git state, and opens VS
Code at the server checkout. It does not pull, commit, or overwrite files.

The direct equivalent is:

```sh
code --remote ssh-remote+gpu-home /home/trey/workspace/gpu-fundamentals
```

## Run exercises

In the Remote SSH window, open a Python file and press **Cmd+Shift+B**. The
default **GPU: Run Current Python File** task starts Jupyter if needed and runs
the active file inside the GPU container.

Use **Tasks: Run Task** for:

- **GPU: Start Runtime**
- **GPU: Run Current Python File**
- **GPU: Build and Run Current Standalone CUDA File**
- **GPU: CUDA and PyTorch Check**
- **GPU: Open Runtime Shell**
- **GPU: Stop Runtime**

From the remote terminal, the same operations are:

```sh
infra/home/scripts/start-runtime
infra/home/scripts/run-gpu curriculum/gpu-mode-lecture-001/pytorch_square.py
infra/home/scripts/run-gpu \
  curriculum/gpu-mode-lecture-002/vector_addition/vector_addition.cu
infra/home/scripts/stop-runtime
```

`run-gpu` accepts a workspace `.py` file or a standalone `.cu` file. It starts
the runtime automatically, detects the RTX 5070's compute capability, and keeps
generated CUDA executables outside the checkout.

## Run notebooks

Start the runtime with **GPU: Start Runtime**, then open an `.ipynb`. Choose
**Select Kernel**, **Existing Jupyter Server**, and enter:

```text
http://127.0.0.1:8889
```

VS Code forwards this remote loopback port over SSH. The notebook kernel runs
inside the NGC container; the selected host interpreter is only for Pylance.
The connectivity notebook at `infra/home/connectivity-test.ipynb`
reports the active GPU and CUDA runtime.

## Runtime contract

The home image is pinned to:

```text
nvcr.io/nvidia/pytorch:25.01-py3@sha256:96990c82825613c3bdeebb66675c7c91b0123f64a5895623316dc5b824e0d7a9
```

It is validated with driver 595.84, CUDA 12.8, PyTorch
`2.6.0a0+ecf3bae40a.nv25.01`, compute capability 12.0, and GCC/G++ 11.5.0.
Jupyter and all observability services remain loopback-only.

After changing `infra/home/Dockerfile` or `infra/home/compose.yaml`, rebuild
deliberately:

```sh
infra/home/scripts/compose build jupyter
infra/home/scripts/start-runtime
```

## Source control

The server checkout is independent of the Mac checkout. Before pulling, inspect
and preserve notebook work:

```sh
git status
git diff
git pull --ff-only
```

Commit and push completed work from whichever checkout owns the edit. Large
models, image layers, datasets, and profiler traces are host-local artifacts.

## Future cloud extension

First optimize and stabilize the commands above on `gpu-home`. If a workload
outgrows 12 GB VRAM or needs multiple GPUs, adapt a Brev or other cloud target
to this interface: Remote SSH, a host Git checkout, `start-runtime`, `run-gpu`,
loopback Jupyter, and explicit lifecycle controls. The older Brev implementation
and its preserved 24.07 image pin remain under `infra/brev/` as reference; they
are not part of the default workflow.
