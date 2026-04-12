# structural-bioinfo-containers

構造バイオインフォマティクスOSSツールのDockerコンテナ集。

パッケージ管理に [Pixi](https://pixi.sh) を採用。GPU使用時はホストの [nvidia-container-toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) が前提。

## 収録ツール

| ツール | カテゴリ | ライセンス | イメージ |
|--------|---------|-----------|--------|
| [Boltz-2](https://github.com/jwohlwend/boltz) | 配列 -> 構造 | MIT | `ghcr.io/mionoyui/boltz` |
| [ColabFold](https://github.com/sokrypton/ColabFold) | 配列 -> 構造 | MIT | `ghcr.io/mionoyui/colabfold` |
| [ProteinMPNN](https://github.com/dauparas/ProteinMPNN) | 構造 -> 配列 | MIT | `ghcr.io/mionoyui/proteinmpnn` |
| [BoltzGen](https://github.com/HannesStark/boltzgen) | バインダー設計 | MIT | `ghcr.io/mionoyui/boltzgen` |


## 前提条件

- Docker または Apptainer/Singularity
- GPU使用時: [nvidia-container-toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) インストール済み
- 各コンテナのビルド: [Pixi](https://pixi.sh) (`curl -fsSL https://pixi.sh/install.sh | bash`)

## クイックスタート

### Boltz-2 で構造予測

```bash
# モデル重みのキャッシュ用ディレクトリを用意
mkdir -p ~/.boltz

# 構造予測（初回実行時に重みを自動ダウンロード）
docker run --rm --gpus all \
  --shm-size=8g \
  -v ~/.boltz:/root/.boltz \
  -v $(pwd)/example/boltz:/work \
  ghcr.io/mionoyui/boltz:2.2.1 \
  boltz predict /work/input/input.fasta --out_dir /work/results --accelerator gpu --use_msa_server
```

### ProteinMPNN で配列設計

```bash
docker run --rm --gpus all \
  -v $(pwd)/example/proteinmpnn:/work \
  ghcr.io/mionoyui/proteinmpnn:1.0.1 \
  python /app/ProteinMPNN/protein_mpnn_run.py \
    --pdb_path /work/input/6MRR.pdb \
    --out_folder /work/results \
    --num_seq_per_target 2 \
    --sampling_temp "0.1" \
    --seed 37 \
    --batch_size 1
```

### BoltzGen で環状ペプチドバインダー設計

```bash
mkdir -p ~/.cache/huggingface

docker run --rm --gpus all \
  --shm-size=8g \
  -v ~/.cache/huggingface:/root/.cache/huggingface \
  -v $(pwd)/example/boltzgen:/work \
  ghcr.io/mionoyui/boltzgen:0.3.1 \
  boltzgen run /work/input/cyclicdesign.yaml \
    --output /work/results \
    --num_designs 3
```

## ディレクトリ構成

```
structural-bioinfo-containers/
├── containers/     # ツールごとの独立したDockerfile + pixi.toml
├── example/        # 動作確認用サンプル入力
├── docs/           # ドキュメント（ライセンス注意事項、GPU設定など）
├── versions.json   # ツールバージョンの唯一の情報源（CIが参照）
└── LICENSE         # MIT
```

## バージョン管理方針

- **Dockerfileのベースイメージは必ずバージョンを固定する**（`latest` タグは使用しない）
- 依存関係が許す範囲で、なるべく最新の安定版を使用する
- ツールのバージョンは `versions.json` で一元管理し、Dockerfile・CIの両方がここを参照する
- `pixi.lock` を必ずコミットし、環境の完全な再現性を保証する

## 開発

```bash
# 開発環境セットアップ
pixi install

# ローカルビルド（例: boltz）
docker build -t boltz-local containers/boltz/

# 動作確認
docker run --rm boltz-local boltz --help
```

## ライセンス

このリポジトリのDockerfile・設定コード: MIT License
各ツールのライセンスは [docs/license_notes.md](docs/license_notes.md) を参照。
