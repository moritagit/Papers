# AVID: Adversarial Visual Irregularity Detection


## 概要

* Paper: [AVID: Adversarial Visual Irregularity Detection](https://arxiv.org/abs/1805.09521)
* Date: 2018-05-24
* Authors: Mohammad Sabokrou, Masoud Pourreza, Mohsen Fayyaz, Rahim Entezari, Mahmood Fathy, Jürgen Gall, Ehsan Adeli
* Accepted at:
* Code:
* Japanese
    * [[DL輪読会]AVID:Adversarial Visual Irregularity Detection](https://www.slideshare.net/DeepLearningJP2016/dlavidadversarial-visual-irregularity-detection)
    * [[DL Hacks]AVID: Adversarial Visual Irregularity Detection](https://www.slideshare.net/DeepLearningJP2016/dl-hacksavid-adversarial-visual-irregularity-detection)
* Model
    * Name: AVID
    * Input: 64 x 64
    * Output: 11 x 11
    * Network: U-Net (Inpainting, Generator) + FCN (Discriminator)
    * Dataset:
    * Loss:
* Keywords
* Point of View
    * videoについての異常検知
    * end-to-endで異常検知し，異常箇所をみつけるモデル
    * pixel-level（true positive大）とpatch-level（false positive小）のいいとこどり
    * Inpainting (Generator)：U-Netによりpixel-levelでirregularなものを除去するモデルで，ノイズ除去もする。再構成誤差が大きいとき異常。
    * Detector (Discriminator)：FCNでpatch-levelに見る（入力は丸ごと）。1 patchでも出力が閾値より小さいとき異常。
    * 上2つの異常な領域の積集合を，このモデル全体の判断した異常な領域とする



### Introduction

* end-to-endで異常検知し，異常箇所をみつけるモデルで，patch-levelでないのは初。
* 既存手法はGとDの一方しか使わなくて計算時間の割に合わないし，patch-basedだった
* pixel-levelでvideoの異常箇所を見つけようとすると，true positiveは大きくなるが，false positive errorが大きくなる
* detection rateは犠牲になるが，patchを大きくしてfalse positive errorを下げるモデルはある
* pixel-levelとpatch-level双方を用いたモデルを考案
* pixel-level: Inpaint (Generator), 異常の除去
* patch-level: Detector, patch-levelで異常を検知
* これらをadversarialにtrain



## Dataset

* UCSD
* UMN
    * 3シーン
    * 人が歩いている映像
    * 突然走り出すのでそこが異常
* MNIST



## Method

* 目標: irregularity detection & localization
* I (Inpaint the video): 入力画像からpixel-wise irregularityを除き，Dが正常画像だと勘違いさせるように学習
    * encoder-decoder model (U-Net)
    * 同時にdenosiningも行う
    * AutoEncoderを正例のみでtrainすると負例をうまく再構成できなくなるのを利用
    * pixel-levelなのでtrue positiveは大きくなるが，false positive errorが大きくなってしまう
        * 異常でないものまで指摘しまくってしまう
* D (Detect): 各patchが異常かどうかを予測
    * FCN (Fully Convolutional Neural Networks) (without pooling and FC layers)
    * 正例を1と予測
    * patch-levelなのでfalse positive errorは下がるがdetection error rateが大きくなる
        * 異常の見落としが増える
* IとDはend-to-end・adversarial・self-supervisedにtrainされる（GAN-inspired model）
* Iでpixel-levelにirregularityな箇所がわかり，また，Dがpatch-levelで異常とみなした箇所も得られるので，これらの積集合となる部分が異常な箇所となる
    * 積集合を取ることで，それぞれの欠点を補い合える
* video（時間軸）の処理
    * 一般にはLSTMや3D kernelを用いるが，計算コストがかさむ
    * 1フレーム前との平均を，現在・2フレーム前・4フレーム前の3つについて取り，RGBのように3チャンネルにして入力とすればよい

![model](https://github.com/moritagit/Papers/blob/master/AnomalyDetection/result/figures/AVID_model.PNG "model")



## Experiment

* IR-MNIST
    * 自作
    * 121枚を11x11に並べたもの
    * trainには3を含まない
        * 3が異常とみなされる
    * train : test = 5,000: 1,000



## Result

* 評価指標：EER, AUC
* UCSD
    * Iによって車や自転車は消える
    * AVIDはSOTAとはならなかった
* UMN
    * SOTA
* IR-MNIST

![result UCSD](https://github.com/moritagit/Papers/blob/master/AnomalyDetection/result/figures/AVID_result_UCSD.PNG "result_UCSD")

![result MNIST](https://github.com/moritagit/Papers/blob/master/AnomalyDetection/result/figures/AVID_result_MNIST.PNG "result_MNIST")
