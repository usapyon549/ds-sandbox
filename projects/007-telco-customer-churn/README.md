# 007-telco-customer-churn

通信サービスの契約者データを用いて、顧客解約（Customer Churn）を予測した。

LightGBM による分類モデルを構築し、  
Feature Importance・SHAP・SHAP Dependence Plot を用いて  
モデルの予測根拠を分析した。

# Dataset

- **Dataset:** [Telco Customer Churn Dataset](https://www.kaggle.com/datasets/yeanzc/telco-customer-churn-ibm-dataset)
- **Target:** Customer Churn

# Approach

- 欠損値処理
- カテゴリ変数のエンコーディング
- LightGBM による分類モデル
- Feature Importance による重要特徴量の確認
- SHAP Summary Plot によるモデル解釈
- SHAP Dependence Plot による特徴量間の関係の可視化

# Result

Feature Importance では **Contract** が最も重要な特徴量となった。

SHAP の分析からも、

- 長期契約ほど解約しにくい
- 利用期間（Tenure）が長い顧客ほど解約しにくい
- 月額料金（Monthly Charges）が高い顧客ほど解約しやすい
- Online Security を契約している顧客は解約しにくい

という傾向を確認できた。

# Findings

Feature Importance は「どの特徴量が重要か」を把握することはできるが、予測へどのような方向に影響したかまでは分からない。

一方、SHAP を利用することで、

- 長期契約は解約確率を下げる
- 月単位契約は解約確率を上げる
- 月額料金が高いほど解約方向へ寄与する

など、特徴量が予測へ与える影響の方向まで解釈できた。

さらに、SHAP Dependence Plot を用いて最も重要だった **Contract** を確認したところ、

- Month-to-month は解約方向へ大きく寄与
- One year はやや継続方向へ寄与
- Two year はさらに継続方向へ寄与

という関係が可視化された。

また、`interaction_index` を指定しない設定では相互作用が最も強い特徴量が自動で選択されるが、 **Tenure Months** が選択された。  
これより、Contract と Tenure Months に強い相互作用があることを確認した。

契約期間（Contract）と利用期間（Tenure）はどちらも継続利用に関係する特徴量であり、この結果はデータの性質とも整合していた。

# Improvement

今回はベースラインとして LightGBM を用いた。

今後は以下のような改善も考えられる。

- Optuna によるハイパーパラメータ最適化
- CatBoost・XGBoost との比較
- lightGBMの学習の際に不均衡データ対応（is_unbalance=Trueなど）  
- 予測確率を利用した解約リスク順位付け
- SHAP Interaction Values を利用した特徴量間の相互作用の詳細分析

# Takeaways

今回最も大きな学びは、Feature Importance・SHAP・Dependence Plot はそれぞれ役割が異なるという点である。

- **Feature Importance**：重要な特徴量を把握する
- **SHAP Summary Plot**：予測へ与える影響の方向を把握する
- **SHAP Dependence Plot**：特定の特徴量と他特徴量との関係を詳しく確認する

予測性能だけでなく、「なぜその予測になったのか」を段階的に説明できることが、  
実務で機械学習モデルを活用する上でも重要であることを確認した。