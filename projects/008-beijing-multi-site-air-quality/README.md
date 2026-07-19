# 008-beijing-multi-site-air-quality

# Summary

北京市12観測地点の大気汚染データを用いて、**Aotizhongxin 観測地点の3時間後の PM2.5 濃度**を予測した。

今回は平均値のみを予測する一般的な回帰モデルではなく、**NGBoost (Natural Gradient Boosting)** を用いて、予測値だけではなく**予測の不確実性（確率分布）**まで推定した。

---

# Dataset

**Beijing Multi-Site Air Quality Dataset**  
[UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/501/beijing+multi+site+air+quality+data)

## 対象地点

- Aotizhongxin

## 入力特徴量

- PM10
- SO2
- NO2
- CO
- O3
- 気温
- 気圧
- 露点温度
- 風向
- 風速
- 降水量
- 日時特徴量（month/day/hour の sin・cos）

## 目的変数

- **3時間後の Aotizhongxin PM2.5**

---

# Preprocessing

本データセットには数値・カテゴリの欠損値が存在するため、それぞれ異なる方法で補完を行った。

## Numerical Features

線形補完を使用。

```python
interpolate(method="linear")
```

先頭に連続して存在する欠損は、

- backward fill

で補完した。

## Categorical Features

風向（wd）はカテゴリ変数であるため、

- 最頻値補完
- category型へ変換

を行った。

## Datetime Features

日時から

- month
- day of year
- hour

を生成し、周期性を考慮するため

- sin
- cos

へ変換した。

---

# Feature Selection

EDAでは観測地点間で**相関係数が 1.0** の特徴量が複数確認された。

完全に重複する情報を避けるため、一方の特徴量のみを残して学習を行った。

また、他観測地点の PM2.5 は目的変数と同じ物理量であり、実運用では利用できないケースを想定し、特徴量から除外した。

---

# Model

今回は **NGBoost Regressor** を使用した。

NGBoost は通常の回帰モデルとは異なり、

- 平均値
- 分散

を同時に学習できるため、

> 「どのくらいの値になりそうか」

だけでなく、

> 「どれくらい不確実なのか」

まで推定できる。

---

# Result

| Metric | Score |
|--------|------:|
| MAE | 30.34 |
| RMSE | 48.11 |

---

# Prediction with Uncertainty

今回の特徴は、**平均予測だけでなく95%予測区間を可視化した点**である。

添付したグラフでは、

- **黒：実測値**
- **青：平均予測**
- **水色：95%予測区間**

を表している。

![NGboost prediction with 95% confidence interval](figs/ngboost_prediction.png)

観測値が大きく変動する時間帯では予測区間も広くなり、モデルが**「予測が難しい状況」であること**を表現できている。

一方で、比較的安定した時間帯では予測区間が狭くなり、予測に対する確信度が高いことが分かる。

このように、NGBoost は単一の予測値だけでは表現できない**予測の信頼性**を同時に提供できる点が特徴である。

---

# Findings

今回の実験で最も学びになったのは、

> **「予測値そのもの」だけではなく、「予測の不確実性」を扱えるモデルが存在することだった。**

一般的な回帰モデルでは、

```text
PM2.5 = 35
```

という一点予測しか得られない。

一方、NGBoostでは、

```text
PM2.5 は35付近だが、95%の確率で20〜55程度になる
```

というように、予測の幅まで表現できる。

これは、

- 大気汚染予測
- 気象予報
- 電力需要予測
- 在庫需要予測

など、将来値に不確実性が伴うタスクでは特に有用な考え方である。

**「どの値になるか」だけでなく、**

> **「どの程度その予測を信頼できるか」**

まで考慮できることが、NGBoost の大きな特徴である。  



---

# Future Work

今回はデフォルトパラメータの NGBoost を使用した。

今後は、

- Optuna によるハイパーパラメータ最適化
- LightGBM・CatBoostとの比較
- ラグ特徴量・移動平均、差分（傾き）など時系列特徴量の追加
- CRPS や Negative Log-Likelihood を用いた確率予測の評価

なども検討できる。

---

# Libraries

- pandas
- NumPy
- matplotlib
- seaborn
- category_encoders
- NGBoost
- scikit-learn

---

# Notes


**回帰モデルは「値を当てる」だけではない**

通常の回帰モデルは一点予測のみを返すが、NGBoostでは予測値と同時に予測分布を学習できるため、

- 予測値
- 予測区間
- 予測の信頼性

まで扱える。

実務では、予測値そのものだけでなく、

> **「この予測にはどのくらい自信があるのか」**

という情報が意思決定に重要になる場面も多い。

今回の分析を通して、**確率的予測（Probabilistic Forecasting）**という考え方を実践的に確認することができた。