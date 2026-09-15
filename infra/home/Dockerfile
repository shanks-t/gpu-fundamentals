ARG NGC_PYTORCH_IMAGE=nvcr.io/nvidia/pytorch:25.01-py3@sha256:96990c82825613c3bdeebb66675c7c91b0123f64a5895623316dc5b824e0d7a9
FROM ${NGC_PYTORCH_IMAGE}

RUN groupadd --gid 1000 ubuntu 2>/dev/null || true \
    && if ! id -u ubuntu >/dev/null 2>&1; then \
         useradd --uid 1000 --gid 1000 --create-home --shell /bin/bash ubuntu; \
       fi \
    && install -d --owner=1000 --group=1000 --mode=0755 /home/ubuntu

# Lecture notebooks compile PyTorch extensions with this stable host compiler.
RUN apt-get update \
    && apt-get install --yes --no-install-recommends gcc-11 g++-11 \
    && rm -rf /var/lib/apt/lists/*

RUN python -m pip install --no-cache-dir wurlitzer==3.1.1
