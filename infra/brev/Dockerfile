ARG NGC_PYTORCH_IMAGE=nvcr.io/nvidia/pytorch:24.07-py3
FROM ${NGC_PYTORCH_IMAGE}

# Keep the pinned NGC runtime while giving the workspace owner a real home.
RUN groupadd --gid 1000 ubuntu 2>/dev/null || true \
    && if ! id -u ubuntu >/dev/null 2>&1; then \
         useradd --uid 1000 --gid 1000 --create-home --shell /bin/bash ubuntu; \
       fi \
    && install -d --owner=1000 --group=1000 --mode=0755 /home/ubuntu

# Keep native-extension notebooks on the same host compiler across NGC bases.
RUN apt-get update \
    && apt-get install --yes --no-install-recommends gcc-11 g++-11 \
    && rm -rf /var/lib/apt/lists/*

# Required by notebook cells that capture native CUDA/C++ output.
RUN python -m pip install --no-cache-dir wurlitzer==3.1.1
