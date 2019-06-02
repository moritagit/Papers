# Style2Vec: Representation Learning for Fashion Items from Style Sets


## 概要

+ Paper: [Style2Vec: Representation Learning for Fashion Items from Style Sets](https://arxiv.org/abs/1708.04014)
+ Date: 2017-08-14
+ Authors: Hanbit Lee, Jinseok Seol, Sang-goo Lee
+ Japanese: [Paper Survey (shunk)](http://shunk031.me/paper-survey/summary/cv/Style2Vec-Representation-Learning-for-Fashion-Items-from-Style-Sets)
+ Model
    + Name: Style2Vec
    + Input: Image (64 x 64 x 3)
    + Output: Feature Vector (1,024 dim)
    + Network: VGGNet x 2 (for target and context)
    + Dataset: Polyvore
    + Loss: Log probability (SGNS)
    + Metric: cosine
+ Point of View
    + Word2Vec-Inspiredなモデル
        + word->item, sentence->style set (context->outfit)
        + Skipgram with Negative Sampling



## Related Work

+ Word2Vec
+ Skipgram
+ Negative Sampling


## Data

+ embedding
    Polyvoreから取ってきたもの
    + outfits: 297,083
    + each set: 2 to 4 categorically distinct items
    + items: 236,672
    + tops: 53,460
    + bottoms: 43,180
    + outers: 31,199
    + shoes: 77,981
    + dresses: 30,852
+ Style Classification
    + styleごとにPolyvoreで検索して取得
    + casual, punk, formal, business
    + train:test = 9:1



## Method

+ Word2Vec-Inspiredなモデル
    + 分布仮説（文脈が似ていれば意味も似ている）
    + outfit中の他のアイテムが似ていればitemも似ている
    + word->item, sentence->style set (context->outfit)
+ target item用とcontext items用の2種類のNetwork
+ あるoutfitで共起しているアイテムの値（確率）が大きくなるように学習
    + targetと他のアイテム(context)で内積を取ってsoftmax
    + 実際にはsoftmaxの計算コストが高いので，Negative Sampling
+ Skipgram with Negative Sampling


## Experiment

+ optimizer: Adam


## Result

+ t-SNE
    + 近いものは色や形が似ている
    + 似ていなくても，合うものが近くなっている
+ Analogy Test
    + word2vecでいう「王様 - 男 + 女 = 女王」
    + うまくいった例では，色や形に加えformalさなどのstyleも考慮できている
    + 人間の評価の結果，199/288 (69.1%)がOK（2人がOKしたもの）
+ Style Classification
    + casual, punk, formal, businessの4クラス分類
    + outfitをitemごとのembeddingの平均で表現
    + outfitのembeddingをMLP Classifierに入れて分類
    + Siamese CNN, DCGANとの比較（上回った）
