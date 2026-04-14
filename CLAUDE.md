# CLAUDE.md

このリポジトリでの作業方針。

## プロジェクト概要

構造バイオインフォマティクスツールのDockerコンテナ集。Pixi でパッケージ管理し、マルチステージビルドで最小ランタイムイメージを生成する。

## Dockerfileパターン

全コンテナで共通のマルチステージビルドを使う:

```dockerfile
# Stage 1: pixi install
FROM ghcr.io/prefix-dev/pixi:0.67.0 AS build
WORKDIR /app
COPY pixi.toml pixi.lock ./   # or pyproject.toml pixi.lock
RUN pixi install --locked

# Stage 2: runtime
FROM debian:bookworm-slim
COPY --from=build /app/.pixi/envs/default /app/.pixi/envs/default
ENV PATH=/app/.pixi/envs/default/bin:$PATH
```

- ベースイメージは必ずバージョンを固定する（`latest` 禁止）
- `pixi.lock` は必ずコミットする（再現性の保証）
- `pixi install --locked` でビルドする

## バージョン管理

- ツールバージョンは `versions.json` で一元管理
- Dockerfile の `LABEL tool.version` と `versions.json` は常に一致させる
- 依存ライブラリのバージョンは `pixi.lock` が保証するので Dockerfile には書かない

## CUDA バージョンの扱い

- **PyPI経由のPyTorch**（boltz, boltzgen）: `nvidia-*-cu13` を自動で引くため CUDA 13 固定、ドライバー >= 575 必須
- **conda経由のPyTorch**（proteinmpnn）: `pytorch-cuda = "12.*"` でCUDA 12に固定可能
- **JAX cuda12**（colabfold）: PyPI `jax[cuda12]` でCUDA 12、ドライバー >= 525

## LD_LIBRARY_PATH

PyPI の nvidia-* パッケージはライブラリを site-packages 以下にバンドルする。ランタイムで明示的に指定が必要:

- cu13系（boltz/boltzgen）: `nvidia/cu13/lib` + `nvidia/cuda_nvrtc/lib`
- cu12系（colabfold）: `nvidia/cublas/lib`, `nvidia/cudnn/lib` など個別ディレクトリを全て列挙

## 既知の問題と対処

### colabfold: TF + JAX の CUDA プラグイン競合（segfault）

TF と JAX が同一プロセスで CUDA プラグインを二重登録するとセグフォルトが起きる。
対処: `colabfold_batch` を wrapper スクリプトで置き換えて、TF を先に CPU で初期化する。

```python
#!/usr/bin/env python3
import os, sys
os.environ["CUDA_VISIBLE_DEVICES"] = ""
import tensorflow
del os.environ["CUDA_VISIBLE_DEVICES"]
from colabfold.batch import main
sys.exit(main())
```

### colabfold: WSL2 での GPU 予測

以下の環境変数が必要（`.bashrc` に書いてあっても Docker には伝わらない）:

```
TF_FORCE_UNIFIED_MEMORY=1
XLA_PYTHON_CLIENT_MEM_FRACTION=4.0
XLA_PYTHON_CLIENT_ALLOCATOR=platform
TF_FORCE_GPU_ALLOW_GROWTH=true
```

### boltz/boltzgen: triton JIT コンパイル

`gcc` と `libc6-dev` が runtime イメージに必要。

## ディレクトリ構成

```
containers/{tool}/
  Dockerfile
  pixi.toml (or pyproject.toml)
  pixi.lock
  README.md

example/{tool}/
  input/   # サンプル入力ファイル
  results/ # .gitignore で除外済み
```

## CI/CD

`.github/workflows/build.yml` が `containers/**` の変更を検知して、変更されたコンテナだけ GHCR にビルド&プッシュする。バージョンタグは `versions.json` から読む。
