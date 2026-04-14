# structural-bioinfo-containers

構造バイオインフォマティクスOSSツールのDockerコンテナ集。

パッケージ管理に [Pixi](https://pixi.sh) を採用。GPU使用時はホストの [nvidia-container-toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) が前提。

## 収録ツール

| ツール | カテゴリ | NVIDIAドライバー | ライセンス | イメージ | 詳細 |
|--------|---------|-------------|-----------|--------|------|
| [Boltz-2](https://github.com/jwohlwend/boltz) | 配列 -> 構造 | >= 575 (CUDA 13) | MIT | `ghcr.io/mionoyui/boltz` | [README](containers/boltz/README.md) |
| [ColabFold](https://github.com/sokrypton/ColabFold) | 配列 -> 構造 | >= 525 (CUDA 12) | MIT | `ghcr.io/mionoyui/colabfold` | [README](containers/colabfold/README.md) |
| [ProteinMPNN](https://github.com/dauparas/ProteinMPNN) | 構造 -> 配列 | >= 525 (CUDA 12) | MIT | `ghcr.io/mionoyui/proteinmpnn` | [README](containers/proteinmpnn/README.md) |
| [BoltzGen](https://github.com/HannesStark/boltzgen) | バインダー設計 | >= 575 (CUDA 13) | MIT | `ghcr.io/mionoyui/boltzgen` | [README](containers/boltzgen/README.md) |


## 前提条件

- Docker または Apptainer/Singularity
- GPU使用時: [nvidia-container-toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) インストール済み
- 各コンテナのビルド: [Pixi](https://pixi.sh) (`curl -fsSL https://pixi.sh/install.sh | bash`)

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
