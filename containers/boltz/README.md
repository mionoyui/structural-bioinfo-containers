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
# ~/.boltz がなければ先に作成（初回のみ）
mkdir -p ~/.boltz
git clone structural-bioinfo-containers
cd structural-bioinfo-containers
# リポジトリルートから実行
# 構造予測（GPU）
# ~/.boltz にすでにモデルがあればダウンロードをスキップ
docker run --rm --gpus all \
  --shm-size=8g \
  -v ~/.boltz:/root/.boltz \
  -v $(pwd)/example/boltz:/work \
  ghcr.io/mionoyui/boltz:2.2.1 \
  boltz predict /work/input/input.fasta --out_dir /work/results --use_msa_server
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

初回実行時に `~/.boltz`（コンテナ内 `/root/.boltz`）へ自動ダウンロードされます。
