# Fashion Style in 128 Floats: Joint Ranking and Classification using Weak Data for Feature Extraction


## 概要

+ Paper: [Fashion Style in 128 Floats: Joint Ranking and Classification using Weak Data for Feature Extraction]()
+ Date: 2016
+ Authors: Simo-Serra Edgar, Hiroshi Ishikawa
+ Code: [bobbens/cvpr2016_stylenet](https://github.com/bobbens/cvpr2016_stylenehttps://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/Simo-Serra_Fashion_Style_in_CVPR_2016_paper.pdft)
+ Japanese:
    + [ランキングロスと分類ロスにもとづくファッションデータの特徴抽出](https://esslab.jp/~ess/ja/research/stylenet/)
    + [【論文紹介】Fashion Style in 128 Floats: Joint Ranking and Classification using Weak Data for Feature Extraction (CVPR 2016)](https://www.slideshare.net/HirokatsuKataoka/fashion-style-in-128-floats-joint-ranking-and-classification-using-weak-data-for-feature-extraction-cvpr-2016)
+ Model
    + Name: StyleNet
    + Input: Fasion Snap image (Aspect Ratio = 3:4)
    + Output: Feature Vector (128 dim)
    + Network: CNN-Based (論文中のTable 1)
    + Dataset: Fashion 144k, Hispter Wars
    + Loss: Triplet + Classify
    + Metric: Euclid
+ Point of View
    + Webから手に入る弱教師ありのデータを使用
    + compactでかついい特徴量を，比較的小さめのモデル・データで学習
    + Euclid距離で効率的に比較ができる


## Related Work

+ FaceNet
+ 差分：
    + 正規化を距離に対して行っている
    + 弱（半）教師あり学習
        + Webから大量の画像を入手して利用


## Data

+ Train: Fashion 144k
+ Test: Hisper Wars, Fashion 144k
+ Fashion 144k
    + 6000 imagesで学習させた分類器で不要な画像をcleaningする
        + train:test = 1:1で，VGG16 on ImageNetをfine-tuneして使用
    + ラベル
        + color
            + color-garmentの形式
            + 3180種類
            + $T_1$
        + weakly-annotated
            + colorとgarmentに分割
            + 123種類
            + classificationに用いる
            + $T_2$
+ train:valid = 80554:8948 = 9:1


## Method

+ 弱教師ありのデータから，RankingとClassificationにより特徴学習
+ trainが終われば識別器なしで使える
+ 3枚の画像（Triplets）を入力
    + Reference ($I$), Similar ($I_+$), Dissimilar ($I_-$)
    + 類似・非類似はラベルデータから判断
+ Ranking Loss + 識別Lossを最小化
+ Ranking Loss:
    + 特徴抽出器
    + 類似・非類似画像との距離の関数(Intersection over Union: IoU)
    + to learn discriminative feature representation
+ Classification Loss:
    + 分類器(特徴抽出器に接ぎ木)
        + BN + ReLU + FC(128) + FC(tags)
        + 123 tags
    + Cross-Entropy Loss
    + 弱教師ありを補うようなイメージ
    + more representativeに(to learn useful feature maps)


### Joint Ranking and Classification

手法の詳細について。
画像を$I$，ラベルを$\bm{y}$，得られた特徴ベクトルを$\bm{f}$と表す。


#### Tripletsの組み合わせ方

類似画像かどうかをラベルに基づいて考え，その関数を$r = r(\bm{y}_1, \bm{y}_2)$とする。
いま，全てのラベルを0か1で表すこととし，$|\bm{y}|$はそのラベルのうち1のものの数を表すとする。
このとき，
2つのラベル$\bm{a},\ \bm{b}$の間の類似度を表す関数$r$をIntersection of Union (IoU)と呼び，
次のように定義する。

$$r(\bm{a}, \bm{b}) = \frac{|\bm{a \land b}|}{|\bm{a \lor b}|}$$

以上より，2つの閾値$\tau_s,\ \tau_d$を用いることで，類似・非類似の分類を以下のように行うことができる。

+ similar   : $r(\bm{y}, \bm{y}_+) > \tau_s$
+ dissimilar: $r(\bm{y}_-, \bm{y}) < \tau_d$


#### Ranking Loss

Reference--Similar間の距離$d_+$が，Reference--Dissimilar間の距離$d_-$より小さくなるように学習させたい。
距離についてはEuclid距離を考えるが，正規化を特徴ベクトルについてでなく距離のペアについて行う（soft constraint）。
これにより，RankingとClassificationを同時に学習できる。
正規化はsoftmaxにより行う。

$$d_\mathrm{n} = ||\bm{f}_- - \bm{f}||_2, \quad d_\mathrm{p} = ||\bm{f}_+ - \bm{f}||_2$$
$$d_- = \frac{\exp(d_\mathrm{n})}{\exp(d_\mathrm{n}) + \exp(d_\mathrm{p})}$$
$$d_+ = \frac{\exp(d_\mathrm{p})}{\exp(d_\mathrm{n}) + \exp(d_\mathrm{p})}$$

これより，$d_+,\ d_-$ともに0～1の範囲にある。
以上より，$d_+$を小さく，$d_-$を大きくするようなRanking Loss $I_\mathrm{R}$は次のようになる。

$$I_\mathrm{R} (d_+,\ d_-) = 0.5((d_+)^2 + (1 - d_-)^2) = (d_+)^2$$


#### Classification Loss

非類似画像の特徴ベクトル$\bm{f}_-$のみを用いて分類タスクを解かせる。
一つひとつのラベルは2値なので，$T$個のラベルに対しそれぞれ$(x_0,\ x_1)$という分類結果が得られる。
つまり全体では次のようになる。

$$X_- = (X^t_-)_{t \in T}, \quad X^t_- = (X^t_{-, 0}, X^t_{-, 1})$$

これより，Classification Loss $I_\mathrm{C}$は，
multi-label cross entropyを用いて次のようになる。

$$I_\mathrm{C} (X_-,\ \bm{y}_-) = \frac{1}{|T|} \sum_{t \in T} l_\times (X^t_-,\ y^t_-)$$
$$l_\times (x,\ y) = -x_y + \log(\exp(x_0) + \exp(x_1))$$



#### All

$$I (d_+,\ d_-,\ X_-,\ \bm{y}_-) = I_\mathrm{R} (d_+,\ d_-) + \alpha I_\mathrm{C} (X_-,\ \bm{y}_-)$$

ここで，$\alpha$はLossの重みづけである。



## Experiment

+ optimizer: Adadelta
+ No image cropping, momentum, weight decay
+ 画像の正規化（平均引いて分散で割る）は行った
+ Batch Size
    + Classification: 32
    + Joint: 16
+ Threshold
    + $\tau_s = 0.75$
    + $\tau_d = 0.01$
+ Classification Loss Weight: $\alpha = 0.01$
+ 最初からJointだと学習できない
    + 最初は特徴抽出器に分類用のFC層を付けて学習
        + 収束したら初期化は完了
    + 新しい分類器をつけてjointで学習



## Result

+ Hisper Wars
    + 5-way clothing style classification
    + 39,168-dimのモデル，ImageNetとPlaceでpretrainしたVGGと比較
    + それぞれのモデルで得られたfeatureをSVMで分類
    + acc, pre, rec, iou全てで他を上回った
+ Fashion 144k
    + おしゃれ度を1--10で評価
    + Trainに使ってないデータで評価
    + SVMで分類
    + 4096dimに負けるとこもあるが128dimでは強い
+ Visualizing the Style Descriptor
    + 2画像からoutfitの似た部分をheat mapのような感じで出力
    + 他のモデルよりもちゃんと出せていそう
+ t-SNE
    + 背景に対してはrobustでoutfitのみを考慮している
    + 似たoutfitが近くにいて，遷移もちゃんとしている
