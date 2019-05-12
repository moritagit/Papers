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
    * Input:
    * Output:
    * Network:
    * Dataset:
    * Loss:
* Point of View
    * 正常データのみ利用
    * AutoEncoder + GANのような構造
    * Reconstructor (AutoEncoder, Generator): ノイズを乗せた正常画像を元通りに復元する
    * Discriminator: 入力画像がRの出力か本物かを見分ける
    * 推論時はまずRに入れて再構成させたのちにDに入れて，その出力が1に近いほど異常
    * Rを通すことで知らない画像（正常画像以外）をうまく復元できず，異常をより際立たせる




### Related Work



## Dataset



## Method



## Experiment

* model
* optimizer



## Result

* 評価指標：
