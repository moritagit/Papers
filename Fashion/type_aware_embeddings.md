# Learning Type-Aware Embeddings for Fashion Compatibility


## 概要

+ Paper: [Learning Type-Aware Embeddings for Fashion Compatibility](https://arxiv.org/abs/1803.09196)
+ Date: 2018-06-27
+ Authors: Mariya I. Vasileva, Bryan A. Plummer, Krishna Dusad, Shreya Rajpal, Ranjitha Kumar, David Forsyth
+ Accepted at ECCV 2018
+ Code: [mvasil/fashion-compatibility](https://github.com/mvasil/fashion-compatibility)
+ Japanese: [文献紹介_20180420_CSN _ Learning Type-Aware Embeddings for ...](https://speakerdeck.com/hrsma2i/wen-xian-shao-jie-20180420-csn-learning-type-aware-embeddings-for-fashion-compatibility)
+ Model
    + Name: Type-Aware Embeddings
    + Input: Labeled Image (224 x 224 x 3) and Description (for each item)
    + Output: Feature Vector ($2^{5--9}$ dim)
    + Network: ResNet-18 (pretrained on ImageNet) + simple NN
    + Dataset: Polyvore Outfits
    + Loss: Triplet (compatibility, similarity, VSE)
    + Metric: Euclid
+ Point of View
    + コーディネートのリコメンドの文脈
    + similarityとcompatibilityが考慮されたembeddingを得ようという試み
    + general embedding spaceとtype-specific embedding spaceを分けることで，「ある服に合うが見た目は異なる2種類の靴」といったものを探すことができるようになった
    + Polyvoreを拡張



## Introduction

+ 既存手法の問題点
    + 既存手法：CNNでembedして似ているものは近く，そうでないものは遠くする
    + 根本的な問題点：typeを考慮していない（靴と帽子が近くにembedされうる）
    + 生じる問題点：
        + compress variation: 多様性が失われる
        + improper triangles: 靴Aが帽子Bと，帽子Bが服Cと合うからといって，AとCが合うとは限らない（推移律が成立しない）
            + compativityは推移ではなく近接で表される
            + （つまりsimilarityとcompatibilityを同じ空間で表現しきるのは無理がある的な）
    + 評価方法 (from sec 5 Experiment Details)
        + [Learning Fashion Compatibility with Bidirectional LSTMs](https://arxiv.org/abs/1707.05691)について
        + negative outfitがtopsだけで構成されてたりする
        + fill-in-the-blankの候補もアイテムの種類を考慮してない
        + 合うかどうかの学習に至ってない
        + random samplingが原因
+ 提案手法
    + general embedding space
        + アイテムの類似度を測れる空間
        + 画像とそのテキストのsimilarityを考えてtrain
            + 同カテゴリを近く，異カテゴリを遠くするだけ
        + visual semantic lossも使う
            + 画像のembeddingをそのdescriptionのembeddingに近づける
            + あるitemのimage embeddingとそのdescriptionの間の距離を，他のitemのdescriptionのembeddingより近くするように
            + 意味的に近いアイテムを近くにembedできる
    + type-specific embedding space
        + 2 typeのアイテム間のcompatibilityを測れる空間
        + 合うアイテムが近接
        + 学習するのはgeneral->specificのprojection
    + Negative-sampling時にitemのカテゴリをちゃんと考慮



## Related Work

+ [Learning Visual Clothing Style with Heterogeneous Dyadic Co-occurrences](https://arxiv.org/abs/1509.07473)
+ [Conditional Similarity Networks](https://arxiv.org/abs/1603.07810)
+ [Learning Fashion Compatibility with Bidirectional LSTMs](https://arxiv.org/abs/1707.05691)



## Data

+ Polyvoreを拡張
    + outfits: 68,306
    + items: 365,054
+ test-train split
    + ある服がtrainにもtestにも出てくるような状況はOKか否かの考慮
    + Meryland Polyvore
        + 元々あったdataset
        + train:valid:test = 17,316:1,407:3,076
    + Polyvore Outfits
        + easier split
        + train:valid:test = 53,306:5,000:10,000
        + あるoutfitはtrain，valid，testのいずれかにしか現れないが，あるアイテムはtrainとtestの両方に現れうる
    + Polyvore Outfits-D
        + more difficult split
        + train:valid&test = 16,995:15,145
        + ある服はある分割にのみ現れる
        + Graph segmentation algorythmを用い，それぞれのアイテムをnodeとして，あるoutfitに同時に現れたアイテムをedgeでつないでいく
        + outfits: 32,140
        + items: 175,485
+ 結論としてはPolyvore Outfitsが最も良い（パフォーマンス的な問題）
+ Category (Type): 11個
    + {'bags', 'accessories', 'scarves', 'tops', 'shoes', 'jewellery', 'outerwear', 'sunglasses', 'all-body', 'bottoms', 'hats'}


## Methods

アイテムを$\bm{x}$，embeddingを$\bm{y}$，typeの総数を$T$とし，
$i$番目のtype$\tau$のアイテムを$\bm{x}_i^{(\tau)}$と表す。

$$\bm{y}_i = \bm{f} (\bm{x}_i;\ \bm{\theta})$$

Triplet Loss ($\mu$: margin)

$$l(i,\ j,\ k) = \max\{0,\ (d(i, j) - d(i,k) + \mu)\}$$


### Type Embedding

+ Euclid距離$d$がcompatibilityを表すように学習
    + 近いほど合うように
+ Triplets (set of images) = $\{ \bm{x}_i^{(u)},\ \bm{x}_j^{(v)},\ \bm{x}_k^{(v)} \}$
    + Anchor($i$): type $u$
    + Positive($j$)とNegative($k$): type $v$
+ type-specific embedding space
    + 2 typeのアイテムがあり，合うアイテム同士が近くにある空間
    + $M^{(u,\ v)}$
    + $P^{u \rightarrow (u,\ v)}$: $u$についてgeneral->$M^{(u,\ v)}$とするようなprojection
        + $d \times d$の行列
    + $z_i^{(u)} = P^{u \rightarrow (u,\ v)}(\bm{y}_i) = P^{u \rightarrow (u,\ v)}(\bm{f}(\bm{x}_i^{(u)};\ \bm{\theta}))$
    + 最小化したいもの：$|| z_i^{(u)} - z_j^{(v)} ||$
        + $\bm{x}_i^{(u)}$と$\bm{x}_j^{(v)}$が合う
    + $P$として次の2種類のものを考える
        + (a): $P^{u \rightarrow (u,\ v)} = P^{v \rightarrow (u,\ v)} = \mathrm{diag}(\bm{w}^{(u,\ v)}) \qquad (\bm{w}^{(u,\ v)} \in \mathbb{R}^d )$
        + (b): (a) + $\bm{w}^{(u,\ v)}$が2値のvectorで，gate functionのようにして$\bm{y}$からcompatibilityに関係のある成分だけ抽出するようなもの
    + sparseにしたいのでL1正則化を行う

Compatibility

$$ d_{ij}^{uv} = d (\bm{x}_i^{(u)},\ \bm{x}_j^{(v)},\ \bm{w}^{(u,\ v)}) = || \bm{z}_i^{(u)} \odot \bm{w}^{(u,\ v)} - \bm{z}_j^{(v)} \odot \bm{w}^{(u,\ v)} ||_2^2 $$

Triplet Loss ($\mu$: margin)

$$L_\mathrm{comp} (\bm{x}_i^{(u)},\ \bm{x}_j^{(v)},\ \bm{x}_k^{(v)},\ \bm{w}^{(u,\ v)}; \bm{\theta}) = \max\{0,\ (d_{ij}^{uv} - d_{ik}^{uv} + \mu) \}$$


### Constraints on the learned embedding

Text descriptionを$t_i^{(u)}$とし，$\bm{g} = \bm{g}(\bm{t};\ \bm{\phi})$でembedする。
Similarityを学習するlossは，

$$L_\mathrm{sim} = \lambda_1 l(\bm{x}_j^{(v)},\ \bm{x}_k^{(v)},\ \bm{x}_i^{(u)}) + \lambda_2 l(\bm{t}_j^{(v)},\ \bm{t}_k^{(v)},\ \bm{t}_i^{(u)})$$

Visual Semantic Embedding (VSE)を用い，visual-semantic space上でimage $\bm{x}_i^{(u)}$がdescription $\bm{t}_i^{(u)}$に近くなるようにする（$j,\ k$についても同様）。

$$L_{\mathrm{vse}\ i} = l(\bm{x}_i^{(u)},\ \bm{t}_i^{(u)},\ \bm{t}_j^{(v)}) + l(\bm{x}_i^{(u)},\ \bm{t}_i^{(u)},\ \bm{t}_k^{(v)})$$
$$L_\mathrm{vse} = L_{\mathrm{vse}\ i} + L_{\mathrm{vse}\ j} + L_{\mathrm{vse}\ k}$$


### All

$\bm{f}$をL2正規化して，全体のlossは，

$$L = L_\mathrm{comp} + L_\mathrm{sim} + \lambda_3 L_\mathrm{vse} + \lambda_4 L_\mathrm{L2} + \lambda_5 L_\mathrm{L1}$$


### Description

+ trainでのみ使用
    + 画像のsimilarityを学習させるためのもの
    + 見た目だけでなくその意味的な部分を考慮させるもの
    + general embeddingの正則化項のようなもの
+ compatibilityには直接は効いてない
    + generalに影響しているため，当然効いてはいるが



## Experiments

+ learning rate: 5e-5
+ batch size: 256
+ margin ($\mu$): 0.2
+ $lambda_3$: 5e-1 for Maryland, 5e-5 for the others
+ all the other $lambda$s: 5e-4
+ text representation: HGLMM Fisher vector encoding of word2vec
    + PCAで6000 dimに落としてから



## Result

+ Fashion Compatibility Task
    + outfitがcompatibleかどうかの2値分類
    + Polyvore内のoutfitは全てcompatible
    + negative samplingについてはDataで述べた通り
    + AUCで評価
+ fill-in-the-blank (FITB)
    + outfitに欠けてる1 itemを4択で選ぶ
    + accuracyで評価
+ MarylandのCompat. AUCでBiLSTMに劣ったが，あとは全て上回った
    + Polyvore Outfits-D
        + Compat. AUC: 0.84 (512 dim)
        + FITB ACC: 55.2 (512 dim)
+ t-SNE
    + generalもspecificも
    + Colorとshapeをいい感じに反映
