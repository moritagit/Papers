# Learning Deep Features for One-Class Classification


## 概要

* Paper: [Learning Deep Features for One-Class Classification](https://arxiv.org/abs/1801.05365)
* Date: 2018-01-16
* Authors: Pramuditha Perera, Vishal M. Patel
* Accepted at:
* Code: [PramuPerera/DeepOneClass](https://github.com/PramuPerera/DeepOneClass)
* Japanese
    * [ディープラーニングを使った画像の異常検知　－論文と実装－](https://qiita.com/shinmura0/items/cfb51f66b2d172f2403b)
    * [がんばる人のための画像検査機](https://qiita.com/shinmura0/items/7f4298b75d6b788bba80)
* Model
    * Name: DOC
    * Input: Image
    * Output: Feature vector
    * Network: AlexNet, VGG16
    * Dataset: 1001 Abnormal Objects Dataset, Caltech 256, UMDAA-02
    * Loss: Compactness Loss (Center Loss) + Descriptiveness Loss (Cross-Entropy)
* Keywords
    * one-class classification
    * anomaly detection
    * novelty detection
* Point of View
    * one-class分類におけるfeature learningの手法
    * 正例のみのデータに外部のデータを加えてtrain
        * クラス内の距離を小さく（compactに），クラス間の距離を大きく（descriptiveに）
        * ほぼcenter lossでは...?



### Introduction

* Deep Learningによる分類（の訓練）には次のものが必要
    * 複数クラスのデータ (multiple classes)
    * 膨大な量のデータ (extremely large number of training samples)
* 上2つの満たされ方次第で取るべき手法は変わる
    * a) Multiple classes, many training samples
        * end-to-endでtrain
        * パラメータは乱数で初期化
    * b) Multiple classes, low to medium number of training samples
        * 特徴量抽出器にpre-trained modelを使用
        * データがかなり少ないときはFC層のみをtrain
        * データが多少はあるときはFC層に近いConv層もtrain
        * fine-tuning
    * c) Single class or no training data:
        * 特徴量抽出器にpre-trained modelを使用
        * pre-trainedはscratchでもfine-tuningでもいいが，どのみち外部データで行われる
* 本論文ではc)の単一クラスのみの場合を考える
* Compactness Loss (one-class dataset)とDescriptiveness Loss(external multi-class dataset)を用いてtrain



## Dataset

* 1001 Abnormal Objects Dataset
    * 1,001 abnormal images (6 classes)
        * Airplane, Boat, Car, Chair, Motorbike, Sofa
    * 各クラス最低100枚はある
    * 元々PASCALにあったもの
* ImageNet
    * 256 classes, 30,706 images
* Caltech 256
* UMDAA-02


## Method

* 既存手法の問題点
    * a): pre-trained modelはdescriptivenessは保証してもcompactnessは保証しない
    * b): 外部の大規模データセットでpre-trainedされたモデルに対し，2クラス分類でfine-tuningした場合は，alienクラスがうまく表現されるように学習されていない限り（そのデータにalienクラスに近いものをを含まない限り），descriptivenessに対して悪く働く
        * 例えば，椅子の中でも普通のものと一風変わったものを分類したい，といった場合，ImageNetでpre-trainしたモデルなどでは，椅子というだけでかなり近いところにembedされてしまうという問題がある
    * c): single classでfine-tuningするとcompactだがdescriptiveでなくなる
* a)が一番マシだが，これにcompactnessを加えられれば強いのでは
* Compactness
    * 同クラス（target class, 正例）のfeatureは近くにあってほしい
    * クラス内距離を小さくする
        * ここではeuclid距離を用いる
    * クラス内の分散を小さくすればよい
    * batchの分散を損失として考えればよい
* Discriptiveness
    * 異クラスのfeatureは遠くあってほしい
    * クラス間距離を大きくする
    * Reference dataset (external multi-class dataset)を導入
        * 単一クラスしかない場合，クラス間距離は学習できない
    * 通常の分類タスクをReference dataで行う
    * cross-entropyを用いる
* Siamese Netと同様に，targetとreferenceをパラメータをshareしたCNNに入れる
    * というか完全にcenter lossでは...?
* テスト時には，target classの集合が与えられ，それと新しいデータとの距離を測り，その値と閾値を比べることで分類する

Compactness Lossを$L_\mathrm{C}$，
Discriptiveness Lossを$L_\mathrm{D}$，
target classのtraining dataを$t$，
reference dataを$r$とすると，
損失関数は次のようになる。

$$L = L_\mathrm{D} (r) + \lambda L_\mathrm{C} (t)$$
$$L_\mathrm{C} = \frac{1}{nk} \sum_{i=1}^{n} z_i^\mathrm{T} z_i$$
$$z_i = x_i - m_i, \qquad m_i = \frac{1}{n-1} \sum_{j \neq i} x_j$$
$$z_i = x_i - \frac{1}{n-1} \sum_{j=1}^{n} x_j + \frac{1}{n-1} x_i = \frac{n}{n-1} \left(x_i - \frac{1}{n} \sum_{j=1}^{n} x_j \right) = \frac{n}{n-1} (x_i - m)$$
$$z_i^\mathrm{T} z_i = \frac{n^2}{(n-1)^2} (x_i - m)^\mathrm{T} (x_i-m) = \frac{n^2}{(n-1)^2} \sigma_i^2$$
$$\therefore L_\mathrm{C} = \frac{1}{nk} \sum_{i=1}^{n} \frac{n^2}{(n-1)^2} \sigma_i^2$$

![model](https://github.com/moritagit/Papers/blob/master/AnomalyDetection/result/figures/doc_model.PNG "model")



## Result

* 評価指標：AUC, EER, ROC
* DOC with VGG16
    * 1001 Abnormal Objects, Caltech 256, UDMAA-02で最高性能
    * Caltech 256で星条旗をtargetにしたときのAUCは0.999

![result](https://github.com/moritagit/Papers/blob/master/AnomalyDetection/result/figures/doc_result.PNG "result")
