# Wine Quality

## Overview

ワインの化学成分データを用いて、以下の2つのタスクに取り組んだ。

* ワイン品質の予測（回帰）
* 赤ワイン・白ワインの分類（分類）

同一データセットに対して異なる目的変数を設定し、モデル性能や重要特徴量の違いを比較した。

---

## Dataset

[Wine Quality Dataset](https://archive.ics.uci.edu/dataset/186/wine+quality)

ワインの化学成分（アルコール度数、酸度、糖度など）から、品質やワイン種別を予測するデータセットを利用した。

---

## Approach

### EDA

* 特徴量の分布の確認
* データ型の確認
* 目的変数の分布確認

### Task 1 : Wine Quality Prediction

品質スコアを予測する回帰モデルを構築した。

使用モデル

* LightGBM Regressor
* SVR

評価指標

* RMSE

### Task 2 : Red / White Classification

赤ワイン・白ワインを分類するモデルを構築した。

使用モデル

* LightGBM Classifier
* SVC

評価指標

* Accuracy
* F1 Score

---

## Result

### Quality Prediction (Regression)

| Model    |  RMSE |
| -------- | ----: |
| LightGBM | 0.637 |
| SVR      | 0.678 |

LightGBM の方が高い予測性能を示した。

### Red / White Classification

| Model    | Accuracy |    F1 |
| -------- | -------: | ----: |
| LightGBM |    0.993 | 0.995 |
| SVC      |    0.993 | 0.995 |

両モデルとも非常に高い分類性能を示した。

---

## Feature Importance

### Classification

赤ワイン・白ワインの分類では、以下の特徴量が強く寄与した。

* chlorides
* total_sulfur_dioxide
* sulphates
* volatile_acidity

特に chlorides の寄与が大きく、ワイン種別の判別に重要な特徴量であることが分かった。

### Quality Prediction

品質予測では、以下の特徴量が強く寄与した。

* alcohol
* volatile_acidity
* free_sulfur_dioxide
* sulphates

特に alcohol が最も重要な特徴量となった。

---

## Analysis

同一のデータセットを利用していても、予測対象によって重要な特徴量が大きく変化することを確認できた。

赤ワイン・白ワインの分類では chlorides が最も重要な特徴量となった一方で、  
品質予測では alcohol が最も重要な特徴量となった。

これは、ワインの種類を見分けるための情報と、ワインの品質を評価するための情報が必ずしも一致しないことを示している。

---

## Notes

今回の分析では、同じデータセットでも目的変数によってモデルの見ている情報が変わることを確認した。

分類と回帰の両方を実施することで、予測対象に応じて重要特徴量が変化することを確認できた。

また、モデル性能だけでなく特徴量重要度を確認することで、
モデルがどのような根拠で予測しているかを理解する重要性を再認識した。

---

