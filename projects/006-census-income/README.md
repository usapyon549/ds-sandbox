# 006-census-income

Adult Census Income データセットを用いて、高所得者（>$50K）の二値分類を行った。

本プロジェクトでは、LightGBM をベースモデルとし、KNNImputer による欠損値補完を行った場合との性能を比較した。

---

## Dataset

* UCI Machine Learning Repository
* [Adult (Census Income) Dataset](https://archive.ics.uci.edu/dataset/2/adult)

目的変数

* income (>50K / <=50K)

---

## Objective

Adult データセットは、前処理の教材として広く利用されているデータセットである。

今回はモデル自体の改善ではなく、

**KNNImputer による欠損値補完が分類性能へどの程度影響するか**

をテーマとして検証した。

---

## Methods

### Baseline

* 欠損値は LightGBM にそのまま入力
* Ordinal Encoding
* LightGBM

### KNNImputer Model

前処理として

* Ordinal Encoding
* StandardScaler
* KNNImputer
* LightGBM

を適用し、ベースモデルとの性能を比較した。

---

## Results

| Model      |   Accuracy |  Precision |     Recall |
| ---------- | ---------: | ---------: | ---------: |
| Baseline   |     0.8749 |     0.7835 |     0.6595 |
| KNNImputer | **0.8758** | **0.7845** | **0.6634** |

KNNImputer を適用したモデルは全指標でわずかに改善したものの、性能差は限定的であった。

---

## Discussion

今回のデータセットでは、KNNImputer による改善はごく小さい結果となった。

考えられる理由として、以下が挙げられる。

* 欠損値の多くがカテゴリ変数であり、KNNImputer が本来得意とする数値データと異なるため、補完を十分に活用できなかった可能性
* KNNImputer の適用のためにカテゴリ変数を数値へエンコードし、さらに標準化を行ったことで、本来のカテゴリ情報が距離計算へ適切に反映されなかった可能性
* Notebook をシンプルに保つため全特徴量へ StandardScaler を適用したが、歪みの大きい数値特徴量に対する対数変換などは実施しておらず、距離計算に適した前処理ではなかった可能性

一方で、LightGBM は欠損値を内部で処理できるため、今回のようなケースでは欠損値補完による改善効果が限定的であった可能性もある。

---

## Notes

今回の検証を通じて、

**「前処理として有名な手法だから使う」のではなく、データの特徴に応じて適切な前処理を選択することが重要である**

ことを改めて確認した。

ツールはあくまでツールであり、

* データの型
* 欠損の性質
* モデルの特性

を踏まえた上で前処理を設計することが重要であると確認した。

---

## Tech Stack

* Python
* pandas
* NumPy
* scikit-learn
* LightGBM
* KNNImputer
* Matplotlib
* Seaborn



