
# 005-nasa-randomized-battery-usage

## Overview

リチウムイオンバッテリーの劣化データを用いて、LSTMによる SOH（State of Health）予測を行った。

本プロジェクトでは、

- LSTMのベースモデル
- 移動平均・傾き特徴量を追加したモデル

を比較し、

**LSTMにおいて、特徴量設計とモデル構造のどちらが予測性能へ大きく影響するのか**

を検証した。

---

## Dataset

[Kaggle : Lithium-ion Battery Degradation Dataset](https://www.kaggle.com/datasets/programmer3/lithium-ion-battery-degradation-dataset/data)


入力特徴量

- Voltage
- Current
- Temperature

目的変数

- SOH (State of Health)

※本データセットは、[NASAが提供している元の充放電データ](https://www.nasa.gov/intelligent-systems-division/discovery-and-systems-health/pcoe/pcoe-data-set-repository/)ではなく、各サイクルを集約した特徴量で構成されている。

---

## Experiments

### Base Model

入力特徴量

- Voltage
- Current
- Temperature

を用いてLSTMを学習した。

以下のパラメータを変更しながら比較を行った。

- LSTM Unit数
- Time Step
- Epoch数

なお、上記３つのうち、２つのパラメータの値を固定し、残り１つのパラメータを変更して検証をおこなった。
そのため、精度向上のための組み合わせを探る、というよりもそれぞれのパラメータの検証を行っている。

---

### Feature Engineering

一般的な時系列特徴量として、

各入力特徴量に対して

- 移動平均
- 傾き（差分）

を追加した。

その結果、

入力特徴量は

- Base：6特徴量
- Feature Added：18特徴量

となった。

移動平均や差分の計算により発生する欠損値は `fillna(0)` により補完し、LSTM の Masking レイヤーによって 0 を学習対象から除外する簡易実装とした。

---

## Result

### Base Model

Best MAE

**15.50**

### Feature Added Model

Best MAE

**17.78**

### Optuna

参考として 、Feature Added Modelに対して、Optuna による簡易的なハイパーパラメータ探索を実施した。

Best MAE

**13.99**

---

## Findings

本データセットでは、

移動平均や傾きなどの特徴量を追加しても精度改善は限定的だった。

一方で、

LSTM Unit数を変更した際には MAE が大きく改善しており、

**特徴量設計よりも、LSTMのモデル構造やネットワーク容量の方が予測性能へ与える影響が大きい**

ことを確認した。  
（ただし、あくまで本データセットでの検証結果であることは留意したい。）

また、

Time Step を短く設定した場合は精度が低下する傾向が見られた。

LSTM は比較的長い系列情報を利用することで特徴を学習するモデルであり、

バッテリー劣化も数サイクルだけでは変化が小さいため、

ある程度まとまった時系列情報を入力することが重要である可能性が示唆された。

---

## Discussion

本データセットは、

元の充放電データではなく、

各サイクルごとの平均値として整理されたデータセットである。

そのため、

LSTM が本来得意とする細かな時系列変化の情報は一部失われている可能性がある。

今回追加した

- 移動平均
- 傾き

といった特徴量は、

一般的な時系列特徴量として簡易的に追加したものであるが、

元データが既にサイクル単位で集約されているため、

追加できる情報量は限定的であり、

精度向上へ大きく寄与しなかった可能性が考えられる。

---

## Notes

- 深層学習では特徴量設計だけでなく、モデル構造が予測性能へ大きく影響することを確認した。
- LSTMでは Time Step や Unit数などの設計が重要であることを確認した。
- 時系列データでは、データセットの前処理方法や情報量によって特徴量追加の効果が変化する可能性があることを学んだ。
- Optuna により、少ない実装でハイパーパラメータ探索を行えることを確認した。

---

## Tech Stack

- Python
- pandas
- numpy
- TensorFlow / Keras
- scikit-learn
- Optuna