# atmaCup11 18位/614チーム 解法・ふりかえり
###### tags: `compe`

https://www.guruguru.science/competitions/17/discussions/bc7672a8-b015-494e-9105-ce4102735cbe/

## 自己紹介
はじめましてℕと申します。非kagglerです。AtCoder水色です。大学院生です。

Twitter [@mathbbN](https://twitter.com/mathbbN) やっています。フォローしてください。

コンペ参加の経験は↓の1回だけです。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">D社さんから賞状と賞金、ビールが贈られてきた。(ありがとうございます) <a href="https://t.co/wvsyD0Cur6">pic.twitter.com/wvsyD0Cur6</a></p>&mdash; ℕ (@mathbbN) <a href="https://twitter.com/mathbbN/status/1339760077658443776?ref_src=twsrc%5Etfw">December 18, 2020</a></blockquote>

![](https://i.imgur.com/vbP6qDo.jpg)

画像生成の研究をしていることが、多少、有利に働いたかもしれません。
<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">学習時にヒントカラー与えすぎてて線画から認識した領域を重視してない傾向にあったので．与える確率を1/4にしたら服や髪で別の色指定しても破綻しなくなった．�� <a href="https://t.co/KGKFQSDKYW">pic.twitter.com/KGKFQSDKYW</a></p>&mdash; ℕ (@mathbbN) <a href="https://twitter.com/mathbbN/status/1256863572727959553?ref_src=twsrc%5Etfw">May 3, 2020</a></blockquote>

![](https://i.imgur.com/sHdrsfG.png)

## 18位解法

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">【18位解法】Pretext taskとしてResNet34を3x3ジグソーパズルを解かせた後に、転移学習をtarget、onehot、sorting_date、word2vecの埋め込みの4タスク適用し、得られた特徴量を勾配ブースティング木でスタッキング、アンサンブルてsubしました。<a href="https://twitter.com/hashtag/atmaCup?src=hash&amp;ref_src=twsrc%5Etfw">#atmaCup</a></p>&mdash; ℕ (@mathbbN) <a href="https://twitter.com/mathbbN/status/1418143919628423171?ref_src=twsrc%5Etfw">July 22, 2021</a></blockquote> 

## 再現実験用のリンク(GitHub)

| ソースコードのリポジトリ | [**https://github.com/nat-chan/atmaCup11**](https://github.com/nat-chan/atmaCup11) |
| -------- | -------- |

### モデル重みや前処理画像のリンク

| DLリンク | size | 含まれるファイル | 
| ---- | ---- | ---- |
| [**atma11_deleteweights.zip**](https://www.kde.cs.tsukuba.ac.jp/~natsuki/atma11_deleteweights.zip) | 40M | 🗹特徴量<br>☐モデル重み<br>☐前処理済み画像 |
| [**atma11_deleteimages.zip**](https://www.kde.cs.tsukuba.ac.jp/~natsuki/atma11_deleteimages.zip) | 49G | 🗹特徴量<br>🗹モデル重み<br>☐前処理済み画像 |

前処理済み画像が必要な方は、後述するflip,flopの組み合わせでimagemagickを用いて変換するシェルスクリプトを用意しているので、そちらをお使いください。

## 3foldCV+Train自動化パイプライン
https://github.com/nat-chan/atmaCup11/blob/main/tmuxinator/fold0123.yml

僕の学習・評価パイプラインが独自性があると思うのでまず紹介させてください。

| 1列目の責務 | 2列目の責務 | 3列目の責務 |
| --- | --- | ---- |
| `train`する| `loss`の下がりを監視 | `epoch数.pth`が書き出されたら`evaluate`する |

|        |     | 
| --- | --- |
| 1行目のtrain/testデータ | `3fold0` |
| 2行目のtrain/testデータ | `3fold1` |
| 3行目のtrain/testデータ | `3fold2` |
| 4行目のtrain/testデータ | `all` |

![](https://i.imgur.com/qNgC7T1.gif)

`tmuxinator`を使った割と特殊な方法ですがなんでやったかお気持ちを述べておきます。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">train1epochごとにevaluateするコードの方が多いと思うんだけど、プロセス分けたくないですか…。trainが止まるのは死んでも避けたくて、モデルのsaveをwatchdogしてevaluateした方がリスクが少ないしtrainは速く終わると信じてる。</p>&mdash; ℕ (@mathbbN) <a href="https://twitter.com/mathbbN/status/1418214065411743750?ref_src=twsrc%5Etfw">July 22, 2021</a></blockquote> 


## やった実験

### Cross Validation のやり方

[StratifiedGroupKFold](https://www.guruguru.science/competitions/17/discussions/593ddbfc-da15-4ba7-9482-7e89e0b99a96/)を3foldでやりました。

ただしskleanの実装だと次のように数が1ずれてしまいました。

| train | test |
| ----- | ---- |
| 2624 | 1313 |
| 2625 | 1312 |
| 2625 | 1312 |

2624 = 2^5 × 41 と 都合が良いので

- trainとtestに同じart_seriesが入らないように分ける
- targetの比率を合わせる

を守りながら一例だけtrainからtestに移ってもらいました。

https://github.com/nat-chan/atmaCup11/blob/main/SGKFold.py

かなり厳しいCVをしたのですべてのsubが

SG3FoldCV > PublicLB ≒ PrivateLB

の不等式を満たしており、良いCVを回せたなと満足しています。

### 画像の前処理

今回のコンペはデータ数が少なすぎるなと感じたので、flip,flop,rotateの組み合わせで8倍にデータ数を増やして画像を保存しておきました。

https://github.com/nat-chan/atmaCup11/blob/main/preprocessing/hvr.sh

以降、やけにepoch少ないなと感じたら8倍換算でお考え下さい。

### BatchSizeの選択

![](https://i.imgur.com/KRwNNqJ.png)


デカければデカいほど学習が速く回りますが、デカすぎるとGPUのメモリに乗り切らなくなったり、性能が悪化します。

倍々にして調べて行って最もepoch間の分散の小さく、5epochでのRMSEも小さいBatchSize=256を選択しました。（以降の実験はすべてこれで固定。）


### ResNet34 vs EfficientNetB0

![](https://i.imgur.com/qyfeV1U.png)

5epoch目をみてResNet34を選択しました。

### [3x3ジグゾーパズル](https://www.guruguru.science/competitions/17/discussions/fe12e2a4-d6ba-47c7-b2eb-740c409fd2f9/)のPretext taskは効果あるのか？
https://github.com/nat-chan/atmaCup11/blob/main/preprocessing/j3.py
https://github.com/nat-chan/atmaCup11/blob/main/data/atma11j4_dataset.py

![](https://i.imgur.com/VZBdLql.png)

めっちゃ効果ありました。以降すべてのタスクにこの学習済み重みを用います。

### 転移学習時にどの重みを固定or動かすべきか

![](https://i.imgur.com/sCyCXWD.png)


<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">ResNet34の転移学習をする際、破壊的忘却を防ぐべく、layer1,2,3,4のうち、ボトム側のlayer3,4のみrequire_grad=Trueして学習する対照実験をするべきだった。最終日は勾配ブースティング木でアンサンブルすると決めてたので手戻りを避けたが。1→2epochのみ3foldCVが悪化していたので未練が残る。</p>&mdash; ℕ (@mathbbN) <a href="https://twitter.com/mathbbN/status/1418581404456128516?ref_src=twsrc%5Etfw">July 23, 2021</a></blockquote> 

layer3,layer4,fcのみ動かす場合はコンペ終了後の実験結果です。

1epoch目はもっとも低いRMSEがでています。しかし、5epochまで回すとすべての重みを動かした場合に負けていました。

やはり、最終的にスタッキングする際のモデルのバリアンスが上がるという利点もあるので、すべての重みを動かして良かったと思います。


### 勾配ブースティング木に食わせる用のサブタスク3つ

#### 今までは画像からtargetを単純に予測するタスク
https://github.com/nat-chan/atmaCup11/blob/main/data/atma11simple_dataset.py

でしたが、後に勾配ブースティング木にモデルの出力を食わせてスタッキング（アンサンブルの一種）するために以下の3つのサブタスクを学習させました。

#### 画像からsortingdateを予測するタスク
https://github.com/nat-chan/atmaCup11/blob/main/data/atma11sortingdate_dataset.py
![](https://i.imgur.com/VD7IRuc.png)


#### 画像からtargetをonehotで分類するタスク
https://github.com/nat-chan/atmaCup11/blob/main/data/atma11onehot_dataset.py
![](https://i.imgur.com/6hw8Xxy.png)


#### 画像からword2vec埋め込みを予測するタスク
- https://github.com/nat-chan/atmaCup11/blob/main/data/atma11materialstechniques_dataset.py
- https://github.com/nat-chan/atmaCup11/blob/main/word2vec.py
![](https://i.imgur.com/bfPFmxM.png)


## LightGBMでスタッキング
https://github.com/nat-chan/atmaCup11/blob/main/ensemble.ipynb
これら4つのモデルの出力を特徴量として勾配ブースティング木を学習しました。(なお、GPU時間が余っていたので10epochまで学習しときました。)

flip,flop,rotateでデータ数を8倍にしてあるので特徴量かさましにmean,var,max,minを使いました。meanの場合はTest Time Augmentationと一致すると思います。

- onehotのtargetの予測が0.94を超えていたら強制的にその年代にする。
- スタッキング結果と本来のtarget予測値を0.5の比率で混ぜ合わせる。

この2つの後処理のハイパラも3foldCVで手で2分探索で最適化して、一番CV良かったのとpublicLBが良かった2つを最終submitionとなりました。

## タイムライン (Submissionスコアもここに書きました)

### 6月22日（最終日）

| diff | コメント | SG3FoldCV | PublicLB | PrivateLB |
| --- | --- | --- | --- | --- |
| [ffc7200](https://github.com/nat-chan/atmaCup11/commit/ffc7200) | 余計なセルを消した |
| [afc7f28](https://github.com/nat-chan/atmaCup11/commit/afc7f28) | いいかんじ | **0.7174** | 0.6857 | 0.6754 |
| [73d5129](https://github.com/nat-chan/atmaCup11/commit/73d5129) | 同期をとる |
| [a202e4d](https://github.com/nat-chan/atmaCup11/commit/a202e4d) | とりあえずj4足したら悪くなった？ |
| [3933451](https://github.com/nat-chan/atmaCup11/commit/3933451) | いい感じになってきた | 0.7180 | 0.6844 | **0.6748** |
| [bcf528c](https://github.com/nat-chan/atmaCup11/commit/bcf528c) | 良さげなパラメタ引っ張ってきた |
| [0203137](https://github.com/nat-chan/atmaCup11/commit/0203137) | lru_chacheして無駄にIOしない |
| [5e0e053](https://github.com/nat-chan/atmaCup11/commit/5e0e053) | pklよめた |
| [50f7eb2](https://github.com/nat-chan/atmaCup11/commit/50f7eb2) | よめる |

### 6月21日

| diff | コメント | SG3FoldCV | PublicLB | PrivateLB |
| --- | --- | -- | -- | -- |
| [195e456](https://github.com/nat-chan/atmaCup11/commit/195e456) | さっきまで読めてたのになぜか読めないなんで？ |
| [e1fdd76](https://github.com/nat-chan/atmaCup11/commit/e1fdd76) | やっぱりnotebookで比較 |
| [6913b88](https://github.com/nat-chan/atmaCup11/commit/6913b88) | epochでのCVを表示するように変更 |
| [3852675](https://github.com/nat-chan/atmaCup11/commit/3852675) | evalをgpu4567でもできるように |
| [c5c6417](https://github.com/nat-chan/atmaCup11/commit/c5c6417) | ensembleの途中 |
| [38ff3d8](https://github.com/nat-chan/atmaCup11/commit/38ff3d8) | epoch=conf2のfeatureをpickleでdump |
| [d15ea47](https://github.com/nat-chan/atmaCup11/commit/d15ea47) | Merge branch 'main' of github.com:nat-chan/atmaCup11 into  main |
| [03c4287](https://github.com/nat-chan/atmaCup11/commit/03c4287) | epoch10で3foldもtrainをevalする |
| [fdff7d5](https://github.com/nat-chan/atmaCup11/commit/fdff7d5) | アンサンブル用に依存を追加 |
| [007680d](https://github.com/nat-chan/atmaCup11/commit/007680d) | j4e5のepoch5時点でのsub及びCVを作成 **5epochからの転移学習**| 0.7346 | 0.6947 | 0.6835 |
| [0d7a725](https://github.com/nat-chan/atmaCup11/commit/0d7a725) | materialsとtechniquesをwv2vをで20にするdataset,parameter |
| [dcf5640](https://github.com/nat-chan/atmaCup11/commit/dcf5640) | j4e5のパラメタ追加、5epochまで回して1epochと比較する |
| [8b56a3c](https://github.com/nat-chan/atmaCup11/commit/8b56a3c) | 6から10epochまで学習をresumeするパイプライン |
| [7d1b6ef](https://github.com/nat-chan/atmaCup11/commit/7d1b6ef) | 試しにsubmittion.csvを作るファイル。**1epochからの転移学習完了** | 0.7474 | 0.7114 | 0.6989 |
| [79fbc85](https://github.com/nat-chan/atmaCup11/commit/79fbc85) | sortingdateをのデータセットとパラメタ、5epochじゃ足りないかも |
| [bef8008](https://github.com/nat-chan/atmaCup11/commit/bef8008) | targetをonehot vectorで比較するようにする |
| [d6731b8](https://github.com/nat-chan/atmaCup11/commit/d6731b8) | 転移学習ただし最終層のみ動かすはうまく行かない |
| [992523f](https://github.com/nat-chan/atmaCup11/commit/992523f) | EfficentNetB0を試したがあんま良くない |
| [8034bbf](https://github.com/nat-chan/atmaCup11/commit/8034bbf) | j4のresume、5epochまで回す |
| [9cfe196](https://github.com/nat-chan/atmaCup11/commit/9cfe196) | 3foldとtrainを並行して4gpuで回すためのパラメータ |
| [5cf72a0](https://github.com/nat-chan/atmaCup11/commit/5cf72a0) | CV可視化はpyで管理 |
| [ddd848f](https://github.com/nat-chan/atmaCup11/commit/ddd848f) | 3foldとtrainを4つのgpuで並行してまわす |
| [6139111](https://github.com/nat-chan/atmaCup11/commit/6139111) | out_featuresが複数の時に対応 |

### 6月20日

| diff | コメント |
| --- | --- |
| [adb1bd5](https://github.com/nat-chan/atmaCup11/commit/adb1bd5) | もうぜんぶなんだよ、5iterで十分j4nofreezeのが優秀 |
| [9dbb56d](https://github.com/nat-chan/atmaCup11/commit/9dbb56d) | j3途中までとj4のnofreezeでの比較 **事前学習の3x3ジグソーパズルが完了**|
| [1d1b8bc](https://github.com/nat-chan/atmaCup11/commit/1d1b8bc) | RMSEとfeatureをtrain,test両方書き出すのに合わせてファイル名を変更した |
| [f00c53a](https://github.com/nat-chan/atmaCup11/commit/f00c53a) | 結局使わなかった |
| [865768b](https://github.com/nat-chan/atmaCup11/commit/865768b) | 3時間で1epoch終わるj4データセットを追加、依存の整理 |
| [523073b](https://github.com/nat-chan/atmaCup11/commit/523073b) | 学習止まっててショック、parameterをtransfer_weightsしたら継続できた。netEとnetDの依存を切る |

### 6月19日

| diff | コメント |
| --- | --- |
| [c28b7ae](https://github.com/nat-chan/atmaCup11/commit/c28b7ae) | resnet32model,simple,j3, 3-flod, validateの実装 |
| [4cd4b75](https://github.com/nat-chan/atmaCup11/commit/4cd4b75) | netGをResNetにすることでtrainが動いた、multi | GPUもばっちり |

### 6月18日

| diff | コメント |
| --- | --- |
| [e023943](https://github.com/nat-chan/atmaCup11/commit/e023943) | 各epochごとにdatasetをGPUに配る前にdeterministicにshuffleして他の疑似乱数生成には影響を与えない |
| [621890d](https://github.com/nat-chan/atmaCup11/commit/621890d) | make_deterministic |
| [0bedc38](https://github.com/nat-chan/atmaCup11/commit/0bedc38) | atma11simple_datasetがdry-runできることを確かめた |
| [9e80052](https://github.com/nat-chan/atmaCup11/commit/9e80052) | vscodeのmypyやdebug等の設定 |
| [c1222e2](https://github.com/nat-chan/atmaCup11/commit/c1222e2) | triviaな変更 |

### 6月17日

| diff | コメント |
| --- | --- |
| [4364703](https://github.com/nat-chan/atmaCup11/commit/4364703) | 3foldでtrainの長さがbatchsize64の倍数になるように調整 |
| [ad02361](https://github.com/nat-chan/atmaCup11/commit/ad02361) | どっちみちoptで指定することになるからdataディレクトリの衝突を避けてsymlink貼るのを辞めた |

### 6月16日

| diff | コメント |
| --- | --- |
| [eeae68c](https://github.com/nat-chan/atmaCup11/commit/eeae68c) | dataを.gitignoreする |
| [1c00551](https://github.com/nat-chan/atmaCup11/commit/1c00551) | 前処理scriptをpreprocessingに移動、データセットが動くのを確かめた |

### 6月14日

| diff | コメント |
| --- | --- |
| [41ec383](https://github.com/nat-chan/atmaCup11/commit/41ec383) | 講座2のCamGRAD追加 |
| [23c2e56](https://github.com/nat-chan/atmaCup11/commit/23c2e56) | seabornとかtorchsummaryとか足した |
| [3a95e43](https://github.com/nat-chan/atmaCup11/commit/3a95e43) | SimSiam用のlightlyをdependenciesに追加した |

### 6月13日

| diff | コメント |
| --- | --- |
| [8bd6d09](https://github.com/nat-chan/atmaCup11/commit/8bd6d09) | 研究のコードベースから移植できるように依存ライブラリを増やした。seaborn等、discussionのコードが動くようにライブラリを増やした。 |
| [e4a3d83](https://github.com/nat-chan/atmaCup11/commit/e4a3d83) | StratifiedGroupKFoldを理解した、これからart_series_をgroupにしで分割するようにする |

### 6月12日

| diff | コメント |
| --- | --- |
| [22ae03f](https://github.com/nat-chan/atmaCup11/commit/22ae03f) | word2vecで埋め込み表現得られることを確認 |
| [9e10738](https://github.com/nat-chan/atmaCup11/commit/9e10738) | StratifiedGroupKFoldが使えるnightly sklearnと1.9の最新pytorch |

### 6月10日

| diff | コメント |
| --- | --- |
| [b886438](https://github.com/nat-chan/atmaCup11/commit/b886438) | lightlyよさげだけどpytorchに依存してるのでちゃんとenvironment.yml書かなきゃだめそう |
| [694de56](https://github.com/nat-chan/atmaCup11/commit/694de56) | Initial | commit |

## ふりかえり

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">初期に実装したパイプラインは3foldCVが下にサチる5epochまで4タスクの並列学習・評価に4時間かかる。アンサンブルのハイパラチューニングが終わって最終subができた時点でコンペ終了まで10時間あったから、手戻りを恐れず、学習し直せば、間に合った筈、敢闘賞目指せたのではないかという未練がある。</p>&mdash; ℕ (@mathbbN) <a href="https://twitter.com/mathbbN/status/1418583008739692547?ref_src=twsrc%5Etfw">July 23, 2021</a></blockquote>

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">ResNet34の転移学習をする際、破壊的忘却を防ぐべく、layer1,2,3,4のうち、ボトム側のlayer3,4のみrequire_grad=Trueして学習する対照実験をするべきだった。最終日は勾配ブースティング木でアンサンブルすると決めてたので手戻りを避けたが。1→2epochのみ3foldCVが悪化していたので未練が残る。</p>&mdash; ℕ (@mathbbN) <a href="https://twitter.com/mathbbN/status/1418581404456128516?ref_src=twsrc%5Etfw">July 23, 2021</a></blockquote>

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">今回のコンペもtrain時のlossの下がり方の管理を自前実装していて、wandb使っときゃ良かったなとなり。実装量が少なくなるライブラリは是非とも使いたいが、しかしながらhydraくらいFatなframeworkだとディレクトリ構造・実験パイプラインを相手側に合わせる必要があり、僕の好みじゃない。</p>&mdash; ℕ (@mathbbN) <a href="https://twitter.com/mathbbN/status/1418579194880004105?ref_src=twsrc%5Etfw">July 23, 2021</a></blockquote>


<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">csvをpd.DataFrameで扱うのが定石なのだろうが、特徴量エンジニアリングするにあたってmodelの出力をidをkeyとするDefaultDict[str, List[np.ndarray]]にappendしていってpickleで保存してしまうのが一番取り扱い安かったな…</p>&mdash; ℕ (@mathbbN) <a href="https://twitter.com/mathbbN/status/1418195514944757764?ref_src=twsrc%5Etfw">July 22, 2021</a></blockquote> 

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">実験の結果とかモデルの重みとか全部zipして無限容量のGoogleDriveに投げてるんだけどgdriveって奴がコマンドラインから叩くだけで便利だった。conda install -c conda-forge gdriveで入る。</p>&mdash; ℕ (@mathbbN) <a href="https://twitter.com/mathbbN/status/1418463034796765192?ref_src=twsrc%5Etfw">July 23, 2021</a></blockquote>

P.S. `gdrive`は使えなくなったので`rclone`がおすすめです。
