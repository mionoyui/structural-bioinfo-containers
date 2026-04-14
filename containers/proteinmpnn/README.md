# ProteinMPNN

[ProteinMPNN](https://github.com/dauparas/ProteinMPNN): タンパク質配列設計（逆問題折り畳み）。

- License: MIT
- Image: `ghcr.io/mionoyui/proteinmpnn`
- Tool version: 1.0.1

## 前提条件

- NVIDIA driver >= 525（CUDA 12対応）
- nvidia-container-toolkit インストール済み

## 使い方

```bash
# リポジトリルートから実行
# example/proteinmpnn/input/6MRR.pdb を入力として配列設計
mkdir -p example/proteinmpnn/results

docker run --rm --gpus all \
  --user $(id -u):$(id -g) \
  -v /etc/passwd:/etc/passwd:ro \
  -v $(pwd)/example/proteinmpnn/input:/input:ro \
  -v $(pwd)/example/proteinmpnn/results:/output \
  ghcr.io/mionoyui/proteinmpnn:1.0.1 \
  python /app/ProteinMPNN/protein_mpnn_run.py \
    --pdb_path /input/6MRR.pdb \
    --out_folder /output \
    --num_seq_per_target 2 \
    --sampling_temp "0.1" \
    --seed 37 \
    --batch_size 1
```

## ローカルビルド

```bash
cd containers/proteinmpnn
pixi install

docker build -t proteinmpnn-local .

docker run --rm proteinmpnn-local
```

## 入力フォーマット

PDB形式またはJSONL形式。詳細は [ProteinMPNNドキュメント](https://github.com/dauparas/ProteinMPNN) を参照。

## モデル重みについて

モデル重みはリポジトリにバンドル済みのため、別途ダウンロード不要。
ライセンス: MIT
