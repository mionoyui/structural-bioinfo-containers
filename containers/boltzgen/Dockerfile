# BoltzGen: universal binder design
# https://github.com/HannesStark/boltzgen
# License: MIT
#
# Requirements:
#   - nvidia-container-toolkit on the host for GPU access
#   - NVIDIA driver >= 575 (CUDA 13 required; PyTorch pulls nvidia-*-cu13 packages)
#   - Mount a cache directory for model weights (~6GB): -v ~/.cache/huggingface:/root/.cache/huggingface
#
# Usage:
#   docker run --rm --gpus all \
#     --shm-size=8g \
#     -v ~/.cache/huggingface:/root/.cache/huggingface \
#     -v $(pwd)/example/boltzgen:/work \
#     ghcr.io/mionoyui/boltzgen:0.3.1 \
#     boltzgen --help

# ── Stage 1: Build — resolve & install pixi environment ──────────────────────
FROM ghcr.io/prefix-dev/pixi:0.67.0 AS build

WORKDIR /app
COPY pixi.toml pixi.lock ./
RUN pixi install --locked

# ── Stage 2: Runtime ─────────────────────────────────────────────────────────
FROM debian:bookworm-slim

# Model weights are downloaded to ~/.cache/huggingface (i.e. /root/.cache/huggingface in container).
# Mount the host cache to avoid re-downloading: -v ~/.cache/huggingface:/root/.cache/huggingface
VOLUME /root/.cache/huggingface

RUN apt-get update && apt-get install -y --no-install-recommends \
        libgomp1 \
        libglib2.0-0 \
        ca-certificates \
        gcc \
        libc6-dev \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY --from=build /app/.pixi/envs/default /app/.pixi/envs/default

ENV PATH=/app/.pixi/envs/default/bin:$PATH
ENV LD_LIBRARY_PATH=/app/.pixi/envs/default/lib/python3.11/site-packages/nvidia/cu13/lib:/app/.pixi/envs/default/lib/python3.11/site-packages/nvidia/cuda_nvrtc/lib
ENV HF_HOME=/root/.cache/huggingface

CMD ["boltzgen", "--help"]

LABEL org.opencontainers.image.source="https://github.com/mionoyui/structural-bioinfo-containers"
LABEL org.opencontainers.image.licenses="MIT"
LABEL org.opencontainers.image.description="BoltzGen: universal binder design"
LABEL tool="boltzgen"
LABEL tool.version="0.3.1"
