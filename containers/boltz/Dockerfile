# Boltz-2: biomolecular structure prediction
# https://github.com/jwohlwend/boltz
# License: MIT
#
# Requirements:
#   - nvidia-container-toolkit on the host for GPU access
#   - NVIDIA driver >= 575 (CUDA 13 required; PyTorch pulls nvidia-*-cu13 packages)
#   - Mount a cache directory for model weights: -v ~/.boltz:/root/.boltz
#
# Usage:
#   docker run --rm --gpus all \
#     -v ~/.boltz:/root/.boltz \
#     -v $(pwd):/work \
#     ghcr.io/mionoyui/boltz:2.2.1 \
#     boltz predict /work/input.fasta --out_dir /work/results --accelerator gpu

# ── Stage 1: Build — resolve & install pixi environment ──────────────────────
FROM ghcr.io/prefix-dev/pixi:0.67.0 AS build

WORKDIR /app
COPY pixi.toml pixi.lock ./
RUN pixi install --locked

# ── Stage 2: Runtime ─────────────────────────────────────────────────────────
# debian:bookworm-slim (Debian 12 LTS) — stable base, no CUDA image needed;
# GPU is provided by nvidia-container-toolkit on the host.
FROM debian:bookworm-slim

# Boltz downloads model weights to ~/.boltz (i.e. /root/.boltz in container).
# Mount the host cache to avoid re-downloading: -v ~/.boltz:/root/.boltz
VOLUME /root/.boltz

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

CMD ["boltz", "predict", "--help"]

LABEL org.opencontainers.image.source="https://github.com/mionoyui/structural-bioinfo-containers"
LABEL org.opencontainers.image.licenses="MIT"
LABEL org.opencontainers.image.description="Boltz-2: biomolecular structure prediction"
LABEL tool="boltz"
LABEL tool.version="2.2.1"
