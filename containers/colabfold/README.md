# ColabFold

[ColabFold](https://github.com/sokrypton/ColabFold): AlphaFold2を高速MSAで動かすタンパク質構造予測ツール。

- License: MIT
- Image: `ghcr.io/mionoyui/colabfold`
- Tool version: 1.6.1

## 前提条件

- NVIDIA driver >= 525（CUDA 12対応）
- nvidia-container-toolkit インストール済み
- モデル重みのキャッシュ用ディレクトリ（初回ダウンロード ~4GB）

## 使い方

```bash
# リポジトリルートから実行
mkdir -p ~/.cache/colabfold ~/.cache/jax_cache

# 構造予測（初回実行時に重みを自動ダウンロード）
docker run --rm --gpus all \
  --shm-size=8g \
  -v ~/.cache/colabfold:/root/.cache/colabfold \
  -v ~/.cache/jax_cache:/root/.cache/jax_cache \
  -e JAX_COMPILATION_CACHE_DIR=/root/.cache/jax_cache \
  -v $(pwd)/example/colabfold:/work \
  ghcr.io/mionoyui/colabfold:1.6.1 \
  colabfold_batch /work/input/1BJP_1.fasta /work/results
```

> **注:** JAXのXLAコンパイルキャッシュを `~/.cache/jax_cache` にマウントすることで、2回目以降の起動時にJITコンパイルをスキップできる。初回のみコンパイルが走る。

### WSL2 (Windows) での実行

WSL2環境ではGPUメモリ管理の都合上、追加の環境変数が必要:

```bash
docker run --rm --gpus all \
  --shm-size=8g \
  -e TF_FORCE_UNIFIED_MEMORY=1 \
  -e XLA_PYTHON_CLIENT_MEM_FRACTION=4.0 \
  -e XLA_PYTHON_CLIENT_ALLOCATOR=platform \
  -e TF_FORCE_GPU_ALLOW_GROWTH=true \
  -e JAX_COMPILATION_CACHE_DIR=/root/.cache/jax_cache \
  -v ~/.cache/colabfold:/root/.cache/colabfold \
  -v ~/.cache/jax_cache:/root/.cache/jax_cache \
  -v $(pwd)/example/colabfold:/work \
  ghcr.io/mionoyui/colabfold:1.6.1 \
  colabfold_batch /work/input/1BJP_1.fasta /work/results
```

## ローカルビルド

```bash
cd containers/colabfold
pixi install

docker build -t colabfold-local .

docker run --rm colabfold-local colabfold_batch --help
```

## 入力フォーマット

FASTAまたはA3M形式。モノマー・マルチマー両対応。詳細は [ColabFoldドキュメント](https://github.com/sokrypton/ColabFold) を参照。

## モデル重みについて

初回実行時に `~/.cache/colabfold`（コンテナ内 `/root/.cache/colabfold`）へ自動ダウンロードされる。
ローカルに localcolabfold をインストール済みであれば同じキャッシュをそのまま共有できる。
