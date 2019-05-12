# Adversarially Learned One-Class Classifier for Novelty Detection


## 概要

* Paper: [Adversarially Learned One-Class Classifier for Novelty Detection](https://arxiv.org/abs/1802.09088)
* Date: 2018-02-25
* Authors: Mohammad Sabokrou, Mohammad Khalooei, Mahmood Fathy, Ehsan Adeli
* Accepted at: CVPR2018 (poster)
* Code: [khalooei/ALOCC-CVPR2018](https://github.com/khalooei/ALOCC-CVPR2018)
* Japanese
    * [ディープラーニングによる異常検知手法ALOCCを実装した](https://qiita.com/kzkadc/items/334c3d85c2acab38f105)
    * [論文まとめ＆実装例：Adversarially Learned One-Class Classifier for Novelty Detection](https://qiita.com/masataka46/items/b167a89c11061eee607d)
    * [ALOCCを使った「文字画像」を判別する試み](https://devblog.thebase.in/entry/2018/11/07/111126)
    * [論文紹介：Adversarially Learned One-Class Classifier for Novelty Detection](https://www.slideshare.net/KazukiAdachi/adversarially-learned-oneclass-classifier-for-novelty-detection)
* Model
    * Name: ALOCC
    * Input: Image
    * Output: Reconstructor: Image (reconstructed), Discriminator: Prob. of real image
    * Network: CNN (without pooling layers)
    * Dataset: MNIST, Caltech-256, UCSD Ped2
    * Loss: Reconstrucing Loss (MSE) + mini-max
* keywords
    * novelty detection
    * one-class classification
    * GAN
* Point of View
    * 正常データのみ利用
    * AutoEncoder + GANのような構造
    * Reconstructor (AutoEncoder, Generator): ノイズを乗せた正常画像を元通りに復元する
    * Discriminator: 入力画像がRの出力か本物かを見分ける
    * 推論時はまずRに入れて再構成させたのちにDに入れて，その出力が0に近いほど異常
    * Rを通すことで知らない画像（正常画像以外）をうまく復元できず，異常をより際立たせる



### Introduction

* DLを用いたone-class分類で初のend-to-endモデル
    * one-class分類ではtrain時に負例が使えないため，end-to-endのモデルは組めていなかった
    * DCNNはend-to-endでかなりいい結果を残している
* Generator (Reconstructor)もDiscriminatorも両方用いる



## Dataset

* MNIST
    * 60,000枚の手書き数字データセット
* Caltech-256
    * 256カテゴリ
    * 30,607枚の画像
    * 各カテゴリに80--400枚が含まれる
* UCSD Ped2
    * 10 fps
    * 240 x 360
    * 歩行者が写っている



## Method

* AutoEncoder + GANのような構造
* Reconstructor (AutoEncoder, Generator, distortor): ノイズを乗せた正常画像を元通りに復元する
* Discriminator (detector): 入力画像がRの出力か本物かを見分ける
* 推論時はまずRに入れて再構成させたのちにDに入れて，その出力が1に近いほど異常
* Rを通すことで知らない画像（正常画像以外）をうまく復元できず，異常をより際立たせる
* 正例のみでtrainしたAutoEncoderの潜在変数zを用いた異常検知の手法はあったが，今回は再構成した画像を用いている

![model architecture](https://github.com/moritagit/Papers/blob/master/AnomalyDetection/result/figures/alloc_model.PNG "model")



## Experiment

* model
    * 安定化のためpooling層は使わない
* MNIST
    * 数字を1つ選んでtarget class (正例)とし，他の数字を負例とする
    * 負例は10%--50%とする
* Caltech-256
    * {1, 3, 5}カテゴリを正例とし，各カテゴリ最大150枚を用いる
    * 負例は50%とする
* UCSD Ped2
    * 車などを異常とみなす
    * 30 x 30のパッチに分割



## Result

* 評価指標：AUC, F1
* MNIST
    * LOF, DRAEと比較したが，D(R(X))が最高
    * 負例の数を増やしても，D(R(X))は他の手法に比べてロバスト
* Caltexh-256
    * 比較対象：CoP, REAPER, OutlierPursuit, LRR, DPCP, R-graph
    * 基本D(R(X))が1位，D(X)が2位
        * 正例が1カテゴリのときのAUCのみR-graphが1位
    * 正例を5カテゴリに増やしてもD(R(X))のAUC, F1は0.9を超えていてロバスト
* UCSD Ped2
    * 評価指標：EER (Equal Error Rate)
    * 1パッチでも異常とみなされたら，そのフレームは異常
    * SOTAとまではいかないが，タスク専用の設定でなく一般的なモデルでSOTA並のスコアを出せた
    * D(R(X)): 13%, D(X): 16%, SOTA (Deep-cascade): 9%

![result](https://github.com/moritagit/Papers/blob/master/AnomalyDetection/result/figures/alloc_result.PNG "result")
