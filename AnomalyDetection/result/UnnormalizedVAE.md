# 深層生成モデルによる非正則化異常度を用いた工業製品の異常検知


## 概要

* Paper: [深層生成モデルによる非正則化異常度を用いた工業製品の異常検知](https://confit.atlas.jp/guide/event-img/jsai2018/2A1-03/public/pdf?type=in)
* Date: 2018
* Authors: 立花亮介, 松原崇, 上原邦昭 (神戸大学 大学院 システム情報学研究科)
* Accepted at 人工知能学会
* Japanese
    * [Variational Autoencoderを使った画像の異常検知　前編](https://qiita.com/shinmura0/items/811d01384e20bfd1e035)
    * [Variational Autoencoderを使った画像の異常検知　後編　（塩尻MLもくもく会＃7）](https://qiita.com/shinmura0/items/6572d782ad21b15b004e)
    * [Keras VAEの画像異常検出を理解する](http://cedro3.com/ai/keras-vae-anomaly/)
* Model
    * Name: Unnormalized VAE
    * Input: Screw image (640 x 480 -> 96 x 96)
    * Output: Abnormal score
    * Network: CNN
    * Dataset: The screw dataset
    * Loss: ELBo
* Point of View
    * VAEやGMMにおいて異常度を表す損失関数の値から，正則化項として効くものを排除し，再構成誤差やマハラノビス距離となる項のみを残した
    * これにより，出現頻度の異なる要素間での，異常度の値を調整



## Dataset

* The screw dataset
    * アイシン・エィ・ダブリュ株式会社
* 640 x 480 gray scale images
* train : test = 12,406 : 995
* (test) normal : abnormal = 888 : 107
* trainのabnormalの率は不明（testと同一と仮定）



## Method

* 異常度の算出において，損失関数の値から正則化項として働くものを引いたものを用いた
    * 再構成誤差やマハラノビス距離
* 画像に写るものは要素に分解可能
    * 数字なら1は直線のみだが9は直線と円からなるなど
* 異なる種類の要素グループ間でそのデータ数に違いがある
* そのデータ数の違いをならさないと，頻度の高い要素において異常となる距離が，他の要素だと正常，といったアンバランスなことが起こる
* 元の画像方16ピクセルごとにパッチを切り出し，そのパッチ全てのうちで1つでも異常度が閾値を超えたら，その画像は異常とする



## Experiment

* model
    * encoder: 5 CNN + FC
        * 開始チャンネル数Nc: {16, 32, 64}
        * kernel 4x4, stride 2, channel Nc x 2^{n-1}
        * Batch Normalization
        * ReLU
        * 出力（潜在次元数）: {2, 5, 10, 20, 50, 100}
* optimizer: Adam
* weight decay: 0.9999



## Result

* ROC-AUCで評価
* 非正則化VAEが最も高い値に
* ヒートマップ
