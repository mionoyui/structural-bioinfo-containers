# BoltzGen

[BoltzGen](https://github.com/HannesStark/boltzgen): ユニバーサルバインダー設計（環状ペプチド・タンパク質）。

- License: MIT
- Image: `ghcr.io/mionoyui/boltzgen`
- Tool version: 0.3.1

## 前提条件

- NVIDIA driver >= 575（CUDA 13対応が必要。PyTorchが `nvidia-*-cu13` パッケージを要求するため）
- nvidia-container-toolkit インストール済み
- HuggingFaceモデル重みのキャッシュ用ディレクトリ（初回ダウンロード ~6GB）

## 使い方

```bash
# リポジトリルートから実行
mkdir -p ~/.cache/huggingface

# 環状ペプチドバインダー設計（GPU）
docker run --rm --gpus all \
  --shm-size=8g \
  -v ~/.cache/huggingface:/root/.cache/huggingface \
  -v $(pwd)/example/boltzgen:/work \
  ghcr.io/mionoyui/boltzgen:0.3.1 \
  boltzgen run /work/input/cyclicdesign.yaml \
    --output /work/results \
    --num_designs 3
```

## ローカルビルド

```bash
cd containers/boltzgen
pixi install

docker build -t boltzgen-local .

docker run --rm boltzgen-local boltzgen --help
```

## 入力フォーマット

YAMLファイルでデザイン仕様を記述する。ターゲット構造はCIF形式で指定。

### 環状ペプチド設計の例（`example/boltzgen/input/cyclicdesign.yaml`）

```yaml
entities:
  - protein:
      id: B
      sequence: 8..16    # 8〜16残基をランダムに設計
      cyclic: true       # 環状ペプチド

  - file:
      path: /work/input/8jjs.cif
      include:
        - chain:
            id: A
        - chain:
            id: C
      binding_types:
        - chain:
            id: A
            binding: 12,14,61,63,73,76,77,83,101,104,108  # 結合させたい残基番号
```

`--protocol` オプションで設計モードを選択できる（デフォルト: `protein-anything`）。

## パイプラインステップ

`boltzgen run` は以下の6ステップを順に実行する:

1. **design** — 拡散モデルによる構造生成
2. **inverse_folding** — 配列設計（ProteinMPNN的な逆問題折り畳み）
3. **folding** — 設計配列の再フォールディング検証
4. **design_folding** — デザイン構造の再フォールディング
5. **analysis** — ipTM・PAE等のスコアリング
6. **filtering** — フィルタリングとランキング

結果は `results/final_ranked_designs/` に出力される。

## モデル重みについて

初回実行時に HuggingFace (`boltzgen/boltzgen-1`) から自動ダウンロードされる。
キャッシュは `~/.cache/huggingface`（コンテナ内 `/root/.cache/huggingface`）に保存される。
