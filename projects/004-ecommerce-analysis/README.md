# 004-ecommerce-analysis

## Overview

[UCI Online Retail データセット](https://archive.ics.uci.edu/dataset/352/online+retail)を用いて、ECサイトの購買履歴を分析した。

本プロジェクトでは、

- データクリーニング
- RFM分析
- K-Meansによる顧客セグメンテーション
- マーケットバスケット分析（Association Rule Mining）

を実施し、顧客と商品の両面から購買傾向を分析した。

---

## Dataset

- UCI Online Retail Dataset
- 約54万件のEC購買データ

使用データ

- Invoice
- Customer ID
- StockCode
- Description
- Quantity
- UnitPrice
- InvoiceDate

---

## Analysis

### Data Cleaning

以下の前処理を実施した。

- 欠損値の除去
- キャンセル注文の除去
- UnitPriceとQuantityの負の値の除去
- 購入金額（Quantity × UnitPrice）の作成

---

### RFM Analysis

顧客ごとに

- Recency
- Frequency
- Monetary

を算出した。

Frequency と Monetary は一部の顧客に極端な値が存在したため、

- log変換
- StandardScalerによる標準化

を行った。

クラスタ数はエルボー法により検討し、本プロジェクトでは **K=4** を採用した。

顧客は以下の4つに分類された。

| Cluster | Characteristics |
|----------|-----------------|
| VIP Customers | 最近も購入しており、購入回数・購入金額ともに最も高い顧客 |
| Loyal Customers | 継続的に購入している優良顧客 |
| Regular Customers | 平均的な購入傾向を持つ顧客 |
| At-Risk Customers | 過去には購入していたが最近は購入が少ない顧客 |

経営方針や、他のKPIにもよるが、At-Risk Customersに対して離脱防止のクーポンなどを配布する施策などが
考えられる。

---

### Market Basket Analysis

Association Rule Mining（Apriori）を用いて、

商品の同時購入パターンを分析した。

メモリ使用量を抑えるため、

- 3〜5月
- 6〜8月
- 9〜11月
- 12〜2月

の4期間に分割して分析を行った。

---

## Interesting Findings

色違い・柄違いの商品を除外した上で確認すると、

以下のような興味深い組み合わせが確認できた。

| Association Rule | Possible Interpretation |
|------------------|-------------------------|
| ASSORTED COLOURS SILK FAN → PARTY BUNTING | 春〜夏のイベント・パーティー需要が考えられる |
| SMALL HEART MEASURING SPOONS → POPCORN HOLDER | ホームパーティー用途で同時購入されている可能性 |
| JUMBO BAG WOODLAND ANIMALS → RED TOADSTOOL LED NIGHT LIGHT | 森や動物をテーマとしたデザインの商品が一緒に購入される傾向 |

単なる色違い・柄違いだけでなく、  
商品の利用シーンやデザインテーマによる購買傾向が見られた。  
ecサイトの場合、カートの中身に応じて「こちらもいかがですか？」というポップアップなどを表示し、  
購入促進などに活用できる。

---

## Notes

- RFM分析では、対数変換と標準化を行うことでクラスタリングしやすい特徴量を作成できた。
- K-Meansによって顧客を特徴ごとに分類し、一般的なマーケティング施策へ結び付けられることを確認した。
- マーケットバスケット分析では、商品の用途や世界観が共通する商品が一緒に購入される傾向を確認できた。

---

## Other Ideas

今回は色違い・柄違いの商品を除外するために、
商品名の文字列・単語一致を利用した。

ただし、この方法には以下の課題がある。

- "bag" など一般的な単語まで一致と判定してしまう
- 意味的には異なる商品でも文字列が似ていると除外される
- antecedents に複数商品が含まれる場合も、単純な文字列比較のみで判定している

改善案としては、

- SentenceTransformer などで商品名をEmbedding化する
- コサイン類似度によって意味的な類似商品を判定する
- antecedents 内の商品ごとに個別比較を行う

ことで、色違い・シリーズ商品の除外精度を向上できる可能性がある。

本プロジェクトでは分析の流れを重視し、実装は行わなかったが、今後の改善案として検討できる。

## Tech Stack

- Python
- pandas
- numpy
- seaborn
- scikit-learn
- mlxtend