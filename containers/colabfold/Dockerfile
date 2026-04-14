# ColabFold: fast protein structure prediction
# https://github.com/sokrypton/ColabFold
# License: MIT
#
# Requirements:
#   - nvidia-container-toolkit on the host for GPU access
#   - NVIDIA driver >= 525 (CUDA 12)
#   - Mount a cache directory for model weights (~4GB): -v ~/.cache/colabfold:/root/.cache/colabfold
#
# Usage:
#   docker run --rm --gpus all \
#     --shm-size=8g \
#     -v ~/.cache/colabfold:/root/.cache/colabfold \
#     -v ~/.cache/jax_cache:/root/.cache/jax_cache \
#     -e JAX_COMPILATION_CACHE_DIR=/root/.cache/jax_cache \
#     -v $(pwd)/example/colabfold:/work \
#     ghcr.io/mionoyui/colabfold:1.6.1 \
#     colabfold_batch /work/input/1BJP_1.fasta /work/results
#
# On WSL2, add these flags for GPU memory management:
#   -e TF_FORCE_UNIFIED_MEMORY=1
#   -e XLA_PYTHON_CLIENT_MEM_FRACTION=4.0
#   -e XLA_PYTHON_CLIENT_ALLOCATOR=platform
#   -e TF_FORCE_GPU_ALLOW_GROWTH=true

# ── Stage 1: Build — resolve & install pixi environment ──────────────────────
FROM ghcr.io/prefix-dev/pixi:0.67.0 AS build

RUN apt-get update && apt-get install -y --no-install-recommends \
        git \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY pyproject.toml pixi.lock ./
RUN pixi install --locked

# ── Stage 2: Runtime ─────────────────────────────────────────────────────────
FROM debian:bookworm-slim

# Model weights are downloaded to ~/.cache/colabfold (i.e. /root/.cache/colabfold in container).
# Mount the host cache to avoid re-downloading: -v ~/.cache/colabfold:/root/.cache/colabfold
VOLUME /root/.cache/colabfold

RUN apt-get update && apt-get install -y --no-install-recommends \
        libgomp1 \
        libglib2.0-0 \
        ca-certificates \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY --from=build /app/.pixi/envs/default /app/.pixi/envs/default

# Wrapper script: initialize TF on CPU before JAX claims GPU CUDA plugins.
# Without this, TF and JAX both register CUDA plugins causing a segfault at JIT time.
RUN printf '#!/usr/bin/env python3\nimport os, sys\nos.environ["CUDA_VISIBLE_DEVICES"] = ""\nimport tensorflow\ndel os.environ["CUDA_VISIBLE_DEVICES"]\nfrom colabfold.batch import main\nsys.exit(main())\n' \
    > /app/.pixi/envs/default/bin/colabfold_batch && chmod +x /app/.pixi/envs/default/bin/colabfold_batch

ENV PATH=/app/.pixi/envs/default/bin:$PATH
ENV LD_LIBRARY_PATH=/app/.pixi/envs/default/lib/python3.12/site-packages/nvidia/cublas/lib:/app/.pixi/envs/default/lib/python3.12/site-packages/nvidia/cudnn/lib:/app/.pixi/envs/default/lib/python3.12/site-packages/nvidia/cufft/lib:/app/.pixi/envs/default/lib/python3.12/site-packages/nvidia/cusolver/lib:/app/.pixi/envs/default/lib/python3.12/site-packages/nvidia/cusparse/lib:/app/.pixi/envs/default/lib/python3.12/site-packages/nvidia/nccl/lib:/app/.pixi/envs/default/lib/python3.12/site-packages/nvidia/nvjitlink/lib:/app/.pixi/envs/default/lib/python3.12/site-packages/nvidia/cuda_nvrtc/lib:/app/.pixi/envs/default/lib/python3.12/site-packages/nvidia/cuda_runtime/lib:/app/.pixi/envs/default/lib/python3.12/site-packages/nvidia/cuda_cupti/lib
ENV XDG_CACHE_HOME=/root/.cache
ENV MPLBACKEND=Agg
ENV MPLCONFIGDIR=/root/.cache/matplotlib

CMD ["colabfold_batch", "--help"]

LABEL org.opencontainers.image.source="https://github.com/mionoyui/structural-bioinfo-containers"
LABEL org.opencontainers.image.licenses="MIT"
LABEL org.opencontainers.image.description="ColabFold: fast protein structure prediction"
LABEL tool="colabfold"
LABEL tool.version="1.6.1"
