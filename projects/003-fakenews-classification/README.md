# Fake News Detection

## Overview

日本語フェイクニュースデータセットを用いて、ニュース記事の文章からフェイクニュースを判定する分類モデルを構築した。

本プロジェクトでは、古典的な自然言語処理手法である TF-IDF と、事前学習済み SentenceTransformer による埋め込み表現を比較し、それぞれの特徴量表現が分類性能に与える影響を検証した。

また、特徴量だけでなくモデルも変更しながら比較を行い、どの組み合わせが本データセットに適しているかを分析した。

---

## Dataset

[Japanese Fake News Dataset](https://github.com/tanreinama/Japanese-Fakenews-Dataset/tree/master)

ニュース記事中のフェイク部分と実ニュース部分がアノテーションされたデータセットを利用した。

本検証では、リークを避けるため以下の列は学習には使用していない。

* nchar_real
* nchar_fake

モデル入力には記事本文（context）のみを使用した。

---

## Approach

### EDA

* 文章長の分布確認
* 記事のフェイク率（fake_ratio）の分布確認

### Feature Extraction

#### TF-IDF

当初は TfidfVectorizer を使用し一部のデータを用いて動作確認を行なったが、  
全データで実行した際にメモリ不足が発生した。

そのため、

* HashingVectorizer
* TfidfTransformer

を組み合わせた構成へ変更した。

#### Sentence Embedding

SentenceTransformer を利用して文章をベクトル化した。

大量データを扱うため、エンコード時には batch_size を指定し、メモリ使用量を抑えながら埋め込みを生成した。

---

## Models

### TF-IDF

* HashingVectorizer
* TfidfTransformer
* Logistic Regression
* LightGBM

### Sentence Embedding

* SentenceTransformer
* Logistic Regression
* LightGBM

---

## Result

| Method                                    | Accuracy | F1 (weighted) |
| ----------------------------------------- | -------- | ------------- |
| TF-IDF + Logistic Regression              | 0.682    | 0.679         |
| TF-IDF + LightGBM                         | 0.791    | 0.558         |
| SentenceTransformer + Logistic Regression | 0.606    | 0.604         |
| SentenceTransformer + LightGBM            | 0.560    | 0.558         |

Accuracy では **TF-IDF + LightGBM** が最も高い性能を示した。

F1では、**TF-IDF + Logistic Regression** が最も高い性能を示した。

一方で、SentenceTransformer を利用したモデルは期待したほど性能が伸びず、  
TF-IDF ベースの特徴量表現の方が本データセットとの相性が良い結果となった。

---

## Error Analysis

記事中のフェイク文字数割合を表す以下の指標を利用して分析を行った。

```text
fake_ratio = nchar_fake / context_len
```

学習には利用していないが、誤分類分析のために使用した。

分析の結果、

* fake_ratio が 0 または 1 に近い記事
* フェイク要素または実ニュース要素が明確な記事

は比較的正しく分類できる傾向があった。

一方で、

* フェイク要素と実ニュース要素が混在する記事

では誤分類が増加した。

このことから、フェイクニュース判定において曖昧な記事ほど分類が難しくなることが確認できた。

---

## Analysis

当初は SentenceTransformer を利用した埋め込み表現の方が高い性能を示すと予想していた。

しかし実際には、TF-IDF を利用したモデルの方が高い精度となった。

TF-IDF は学習データから語彙情報を構築するため、データセット固有の単語や表現パターンを捉えやすい。

一方で SentenceTransformer は汎用的な事前学習済みモデルであり、文章全体の意味を圧縮した埋め込み表現を生成する。

今回のタスクでは文章全体の意味理解よりも、特定の単語や表現パターンが判定に有効であった可能性が考えられる。

また、本データセットのフェイク文章は GPT-2 ベースの生成モデルによって作成されているため、生成モデル特有の語彙や表現の癖を TF-IDF が捉えた可能性も考えられる。

さらに、TF-IDF は今回の学習データから特徴量空間を構築しているのに対し、SentenceTransformer は事前学習済みモデルをそのまま利用している。

そのため、データセット固有の語彙や表現パターンへの適応という観点でも、TF-IDF が有利に働いた可能性がある。

---

## Notes

今回の検証を通じて、高性能な事前学習モデルを利用すれば必ず性能が向上するわけではないことを学んだ。

モデルやライブラリの新しさだけで手法を選択するのではなく、ツールはあくまでツールであり、データセットやタスクの特性に応じて適切な特徴量表現を選択することが重要である。

また、分類性能の比較だけでなく、誤分類サンプルを分析することで、モデルが苦手とするデータの特徴を把握できることを確認した。

さらに、実運用を想定した場合には精度だけでなくメモリ使用量も重要であり、  
TfidfVectorizer から HashingVectorizer + TfidfTransformer への変更や、  
SentenceTransformer の batch_size 調整など、リソース制約を考慮した実装の重要性も学ぶことができた。

---

## Tech Stack

* Python
* pandas
* numpy
* scikit-learn
* LightGBM
* SentenceTransformers
* Matplotlib
* Seaborn
