# ProteinMPNN: protein sequence design
# https://github.com/dauparas/ProteinMPNN
# License: MIT
#
# Requirements:
#   - nvidia-container-toolkit on the host for GPU access
#   - NVIDIA driver >= 525 (CUDA 12)
#
# Usage:
#   docker run --rm --gpus all \
#     -v $(pwd)/example:/work \
#     ghcr.io/mionoyui/proteinmpnn:1.0.1 \
#     python /app/ProteinMPNN/protein_mpnn_run.py \
#       --pdb_path /work/input.pdb \
#       --out_folder /work/results \
#       --num_seq_per_target 10

# ── Stage 1: Build — resolve & install pixi environment ──────────────────────
FROM ghcr.io/prefix-dev/pixi:0.67.0 AS build

WORKDIR /app
COPY pixi.toml pixi.lock ./
RUN pixi install --locked

# ── Stage 2: Runtime ─────────────────────────────────────────────────────────
FROM debian:bookworm-slim

RUN apt-get update && apt-get install -y --no-install-recommends \
        libgomp1 \
        ca-certificates \
        git \
    && git clone --depth 1 --branch v1.0.1 https://github.com/dauparas/ProteinMPNN /app/ProteinMPNN \
    && rm -rf /app/ProteinMPNN/.git \
    && apt-get remove -y git \
    && apt-get autoremove -y \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY --from=build /app/.pixi/envs/default /app/.pixi/envs/default

ENV PATH=/app/.pixi/envs/default/bin:$PATH

CMD ["python", "/app/ProteinMPNN/protein_mpnn_run.py", "--help"]

LABEL org.opencontainers.image.source="https://github.com/mionoyui/structural-bioinfo-containers"
LABEL org.opencontainers.image.licenses="MIT"
LABEL org.opencontainers.image.description="ProteinMPNN: protein sequence design"
LABEL tool="proteinmpnn"
LABEL tool.version="1.0.1"
