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
mkdir -p ~/.cache/colabfold example/colabfold/results

# 構造予測（初回実行時に重みを自動ダウンロード）
docker run --rm --gpus all \
  --shm-size=8g \
  --user $(id -u):$(id -g) \
  -v /etc/passwd:/etc/passwd:ro \
  -e TF_FORCE_UNIFIED_MEMORY=1 \
  -e XLA_PYTHON_CLIENT_MEM_FRACTION=4.0 \
  -e XLA_PYTHON_CLIENT_ALLOCATOR=platform \
  -e TF_FORCE_GPU_ALLOW_GROWTH=true \
  -e JAX_COMPILATION_CACHE_DIR=/models/colabfold/jax_cache \
  -v ~/.cache/colabfold:/models/colabfold \
  -v $(pwd)/example/colabfold/input:/input:ro \
  -v $(pwd)/example/colabfold/results:/output \
  ghcr.io/mionoyui/colabfold:1.6.1 \
  colabfold_batch \
    --data /models/colabfold \
    /input/1BJP_1.fasta \
    /output
```

> **注:** `JAX_COMPILATION_CACHE_DIR` を `/models/colabfold/jax_cache` に設定することで XLA JIT コンパイル結果が永続化され、2回目以降の起動が速くなる。

### WSL2 (Windows) での実行

上記コマンドにWSL2固有の追加設定は不要。`TF_FORCE_UNIFIED_MEMORY` などの環境変数は既に含まれている。

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

初回実行時に `~/.cache/colabfold`（コンテナ内 `/models/colabfold`）へ自動ダウンロードされる。
ローカルに localcolabfold をインストール済みであれば同じキャッシュをそのまま共有できる。
