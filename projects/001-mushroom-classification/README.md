# Mushroom Classification

## Overview

毒キノコデータセットを用いて、キノコが食用か毒かを分類する機械学習モデルを構築した。

LightGBM、CatBoost、SVM の3種類のモデルを比較し、それぞれの性能や特徴量の影響を確認した。

---

## Dataset

[Mushroom dataset from the UCI ML repository](https://huggingface.co/datasets/mstz/mushroom)

各キノコの形状や色、匂いなどの特徴量から、毒キノコか否かを分類するデータセットを利用した。

---

## Approach

### EDA

* 欠損値の確認
* カテゴリ値の確認
* データ型の確認

### Modeling

以下のモデルを比較した。

* LightGBM
* CatBoost
* SVM

カテゴリ変数を One-Hot Encoding で変換し学習を実施した。

評価指標は以下を使用した。

* Accuracy
* Precision
* Recall
* F1 Score

---

## Result

| Model    | Accuracy | Precision | Recall |   F1 |
| -------- | -------: | --------: | -----: | ---: |
| LightGBM |     1.00 |      1.00 |   1.00 | 1.00 |
| CatBoost |     1.00 |      1.00 |   1.00 | 1.00 |
| SVM      |     1.00 |      1.00 |   1.00 | 1.00 |

全てのモデルで完全分類となった。

---

## Analysis

当初は特徴量 `odor` が目的変数と強い関係を持っていることから、高い精度の主な要因であると考えた。

しかし `odor` を除外して再学習した場合でも、全てのモデルで Accuracy=1.00 を維持した。

特徴量重要度を確認すると、`gill_spacing`、`has_bruises`、`population`、`spore_print_color` など複数の特徴量が予測に大きく寄与していた。

このことから、本データセットは単一の特徴量だけでなく、複数の特徴量が目的変数と強く関連しており、比較的容易な分類問題であることが分かった。

---

## Notes

今回の分析では、高い評価指標が得られた際に、結果だけでなくその理由を調査する重要性を学んだ。

モデル学習後に特徴量重要度やクロス集計を確認したことで、モデル性能の高さだけでなく、データセット自体が持つ構造を理解することができた。

また、EDAでは欠損値やデータ型の確認だけでなく、各特徴量と目的変数の関係を確認することの重要性を改めて認識した。

機械学習ではモデル選択やチューニングだけでなく、データそのものを理解することが重要であると感じた。

---