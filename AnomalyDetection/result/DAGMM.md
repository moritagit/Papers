# Deep Autoencoding Gaussian Mixture Model for Unsupervised Anomaly Detection


## 概要

* Paper: [Deep Autoencoding Gaussian Mixture Model for Unsupervised Anomaly Detection](https://openreview.net/forum?id=BJJLHbb0-)
* Date: 2018-02-22
* Authors: Bo Zong, Qi Song, Martin Renqiang Min, Wei Cheng, Cristian Lumezanu, Daeki Cho, Haifeng Chen
* Accepted at: ICLR2018 Poster
* Code: [danieltan07/dagmm](https://github.com/danieltan07/dagmm)
* Japanese
    * [異常値検出手法 DAGMM の実装](https://qiita.com/ToshihiroNakae/items/5863fd70ed9afc5fd6cd)
        * [DAGMM Tensorflow implementation](https://github.com/tnakae/DAGMM)
    * [Chainerで異常検知手法DAGMMを実装してみた](https://qiita.com/KaijiS/items/94f880ce066dab053c8f)
    * [深層自己符号化器＋混合ガウスモデルによる教師なし異常検知](https://www.slideshare.net/ChihiroKusunoki/ss-98822807)
    * [論文読みメモ： 深層自己符号化器＋混合ガウスモデルによる教師なし異常検知（その1）](http://cookie-box.hatenablog.com/entry/2018/05/18/004544)
* Model
    * Name: DAGMM
    * Input:
    * Output:
    * Network:
    * Dataset:
    * Loss:
* Keywords
* Point of View



### Introduction



### Related Work



## Dataset



## Method

1. Compression Network : AutoEncoderの潜在変数と再構築誤差をconcatしたものを出力する．
2. Estimation Network : 上記の出力を元に，混合ガウスモデル内の各クラスに対し，どれくらいの確率で分類されるかを出力するモデル
3. エネルギー計算 : 1の出力と2の出力より，GMMに対する負の対数尤度を計算する(論文中ではこれをエネルギーと読んでいる)


## Experiment

* model
* optimizer



## Result

* 評価指標：
