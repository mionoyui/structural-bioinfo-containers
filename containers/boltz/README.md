# boltz

[Boltz-2](https://github.com/jwohlwend/boltz): biomolecular structure prediction.

- License: MIT
- Image: `ghcr.io/mionoyui/boltz`
- Tool version: 2.2.1

## 前提条件

- NVIDIA driver >= 575（CUDA 13対応が必要。PyTorchが `nvidia-*-cu13` パッケージを要求するため）
- nvidia-container-toolkit インストール済み

## 使い方

```bash
# リポジトリルートから実行
mkdir -p ~/.boltz ~/.cache/triton example/boltz/results

docker run --rm --gpus all \
  --shm-size=8g \
  --user $(id -u):$(id -g) \
  -v /etc/passwd:/etc/passwd:ro \
  -e NUMBA_CACHE_DIR=/tmp \
  -e TRITON_CACHE_DIR=/models/triton \
  -e TORCHINDUCTOR_CACHE_DIR=/tmp \
  -e XDG_CACHE_HOME=/tmp \
  -v ~/.boltz:/models/boltz \
  -v ~/.cache/triton:/models/triton \
  -v $(pwd)/example/boltz/input:/input:ro \
  -v $(pwd)/example/boltz/results:/output \
  ghcr.io/mionoyui/boltz:2.2.1 \
  boltz predict /input/input.fasta \
    --out_dir /output \
    --cache /models/boltz \
    --use_msa_server
```

## ローカルビルド

```bash
# pixi.lock の生成（初回 or pixi.toml 変更後）
cd containers/boltz
pixi install

# Docker イメージのビルド
docker build -t boltz-local .

# 動作確認
docker run --rm boltz-local boltz predict --help
```

## 入力フォーマット

FASTAまたはYAML形式。詳細は [Boltz-2ドキュメント](https://github.com/jwohlwend/boltz) を参照。

## Boltzのモデル重みについて

初回実行時に `~/.boltz`（コンテナ内 `/models/boltz`）へ自動ダウンロードされます。
