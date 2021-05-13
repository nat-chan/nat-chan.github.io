# ABC200-E Patisserie ABC 2
###### tags: `atcoder`
## 問題文
URL: [https://atcoder.jp/contests/abc200/tasks/abc200_e](https://atcoder.jp/contests/abc200/tasks/abc200_e)

$$
0 \le i,j,k < N \tag{1}
$$
この不等式を満たす$N^3$個の整数の3つ組$(i,j,k)$を以下の順序で並べる。

- $i+j+k$ が小さいものを、より左に並べる。
- ここまでで順序がつかなければ、$i$が小さいものを、より左に並べる。
- ここまでで順序がつかなければ、$j$が小さいものを、より左に並べる。

このとき、左から$K$番目にある整数の3つ組$(i,j,k)$を答えなさい。

※本来の問題文では変数の動く範囲が$1\le i,j,k \le N$である。
しかしながら`0-indexed`にすると実装上都合が良いのでこのように言い換えた。
元の問題に対しては今回得られた解$(i^\ast, j^\ast, k^\ast)$を用いて
$(i^\ast+1,j^\ast+1,k^\ast+1)$を答えればよい。
以降このように動く変数に対して得られた(固定された)変数を$\ast$をつけて区別する。

## 制約
- 入力は全て整数
- $1\le N \le 10^6$
- $1\le K \le N^3$

3つ組$(i,j,k)$を$K$番目に到達するまで小さい順に列挙する$O(N^3)の$愚直解がまず思いつく。
しかし、今回の制約では$O(N^3)$はおろか$O(N^2)$すら通らない。$O(N)$まで落とす方法を考える。

## 方針「階層ごとに解候補の個数を求め、取りつくす」

3つ組$(i,j,k)$は$i+j+k=l$の値によっていくつかの階層に分けられる。
この階層はいくつあるだろうか、まず$l$の取りうる値の範囲を考える。
$i=j=k=0$のとき、最小値$l=0$を取り、
$i=j=k=N-1$のとき、最大値$l=3N-3$を取る。
つまり$0\le l<3N-2$の間の$3N-2$個の値を$l$は漏れなく取りうる。

求めたい$K$番目の3つ組$(i,j,k)$がどの階層に属しているかを知りたい。
そのために、時間計算量$O(1)$で次の問いに答えることのできる3つのオラクルの存在を仮定する。

- ${\rm oracle_1}(l) :=$「$l$が固定された時、(1)を満たす3つ組$(i,j,k)$の取りうる数」
- ${\rm oracle_2}(l,i) :=$「$l,i$が固定された時、(1)を満たす3つ組$(i,j,k)$の取りうる数」
- ${\rm oracle_3}(l,i,j) :=$「$l,i,j$が固定された時、(1)を満たす3つ組$(i,j,k)$の取りうる数」

${\rm oracle_1}(l)$を用いて累積和の考え方を用いることで、
$$
\sum_{l=1}^{l^{\ast}-1}{\rm oracle_1}(l) < K \le \sum_{l=1}^{l^{\ast}}{\rm oracle_1}(l)
$$
を満たす$l^{\ast}$を時間計算量$O(3N-2)=O(N)$で求めることができる。
この結果から、$K$番目の3つ組$(i,j,k)$が階層$l^\ast$に属していることが分かる。

さて、$l^\ast=i+j+k$を満たす解候補は愚直解よりもさらに絞り込めた。しかしながら、
現時点で候補を列挙するには$i$と$j$の2つの変数を動かす必要があり、$O(N^2)$の時間計算量がかかってしまう。
そこでさらに${\rm oracle_2}(l, i)$を用いて$O(N)$の時間をかけて
より細かいふるいにかけるように解の候補を絞りこむ方針を取る。

以降の議論では$l^\ast$の階層の中で求める解が何番目であるかを知ればよい。
従って、$l=1,2\cdots l^\ast-1$の解候補はもう数えあげる必要が無いため次の操作を行っても問題ない。
$$
K' = K-\sum_{l=1}^{l^{\ast}-1}{\rm oracle_1}(l)
$$
以上の操作を行うアルゴリズムをPythonで書くと次のようになる。

```python
for l in range(0, 3*N-2):
    d = oracle1(l)
    if K-d <= 0:
        break
    K -= d
```

実際に${\rm oracle_2}(l, i)$を用いて階層$l^\ast$の中で$K'$番目の3つ組の$i$の値$i^\ast$を計算するのは以下のようになる。

$$
\sum_{i=1}^{i^{\ast}-1}{\rm oracle_2}(l, i) < K' \le \sum_{i=1}^{i^{\ast}}{\rm oracle_2}(l, i) \\
K'' = K'-\sum_{i=1}^{i^{\ast}-1}{\rm oracle_2}(l, i)
$$

```python
for i in range(0, N):
    d = oracle2(l, i)
    if K-d <= 0:
        break
    K -= d
```

最後に${\rm oracle_3}(l, i, j)$を用いて階層$l^\ast, i^\ast$の中で$K''$番目までの解候補をすべて取りつくす。
$$
\sum_{j=1}^{j^{\ast}-1}{\rm oracle_3}(l, i, j) < K'' \le \sum_{j=1}^{j^{\ast}}{\rm oracle_3}(l, i, j) \\
K''' = K''-\sum_{j=1}^{j^{\ast}-1}{\rm oracle_3}(l, i, j)
$$

このようにして得られた$l^\ast, i^\ast, j^\ast$を用いて
$$
k^\ast = l^\ast - i^\ast - j^\ast
$$
を計算することで求めたかった$K$番目の解$(i^\ast,j^\ast,k^\ast)$を答えることができる。

```python
for j in range(0, N):
    d = oracle3(l, i, j)
    if K-d <= 0:
        break
    K -= d
```

## オラクルの設計

<iframe src="../abc200_e_level.html" width=800 height=500></iframe>

### ${\rm oracle_1}(l)$

上記のグラフを動かしてほしい。🔵で示した区間ではピラミッド形になっており、
各階層$l=0,1,2\cdots$の値は$1,3,6\cdots$と一般項の簡単な三角数$\Sigma(l+1)$であることが分かる。

$$
\begin{align}
※\Sigma(n) = \sum_{i=1}^{n} i = \frac{n(n+1)}{2}
\end{align}
$$

🟢で示した区間では逆ピラミッド形になっており、逆順に減っていくため
三角数の一般項に対して $l \to 3N-3-l$
の変数変換をすることで$\Sigma(3N-2-l)$と得ることができる。

難しいのは🔴で示した区間である。
$(N,0,0)$のように区間$[0,N)$に収まらない数が出てきてしまうのだ。
そのような場合の数を考え、三角数から減ずる方針を取る。
まず補題として$i,j,k$の中で$N$以上の数は1種類のみである。
これは🔴で示した区間で最大の$l=2N-1$の時でも、$(N,N-1,0)$のような
例しか作れないことから分かる。
すなわち$N$以上の数として$i,j,k$のどれを選ぶかで3通りの自由度がある。
以降は$i$と固定して場合の数を数える。
$i$が取りうる値はNからlまでであり、$j$が取りうる値は$0$から$l-i$までの$l-i+1$通りである。
$i,j$が定まればこの範囲で$k$がただ一つに定まるので、求める場合の数は以下のようになる。

$$
\begin{align}
\sum_{i=N}^{l} l-i+1
&= \sum_{i=l}^{N} i-(l+1) \\
&= \sum_{i=1}^{N-l+1} i \\
&= \Sigma(l+1-N)
\end{align}
$$

以上の🔵🔴🟢の3つの区間の結果をまとめることで
${\rm oracle_1}(l)$は以下のように定まった。

$$
\begin{align}
{\rm oracle_1}(l) &= \begin{cases}
\Sigma(l+1)                & 0 \le l < N \\
\Sigma(l+1)-3\Sigma(l+1-N) & N \le l < 2N-2 \\
\Sigma(3N-2-l)             & 2N-2 \le l < 3N-2 \\
0                          & (otherwise)
\end{cases}
\end{align}
$$

### ${\rm oracle_2}(l, i)$
$l,i$の値が与えられているとき、$j$を定めれば対応する$k$はただ$1$つに定まる。
従って$j \in [L, U]$と上限と下限が求まれば、その区間の幅$\max(0, U-L+1)$個を答えればよい。
なぜ、$\max(0,\cdot)$で包んでいるのか、それは
$[L, U]=\varnothing$となる場合、
$L$と$U$の求め方によっては$U-L<0$となりうる為である。
実際に上限$U$を求める。$k=0$の時、最大値$j=l-i$を取る。ただし、$N-1$を上回ることはないので、

$$
\begin{align}
U = \min(N-1, l-i)
\end{align}
$$

となる。
また、下限$L$を求める。$k=N-1$の時、最小値$j=l-i-(N-1)$を取る。ただし、$0$を下回ることはないので、
$$
\begin{align}
L = \max(0, l-i-(N-1))
\end{align}
$$

以上の結果をまとめることで
${\rm oracle_2}(l, i)$は以下のように定まった。

$$
\begin{align}
{\rm oracle_2}(l, i) = \max(0, \min(N-1, l-i)-\max(0, l-i-(N-1))+1))
\end{align}
$$

### ${\rm oracle_3}(l, i, j)$
後は簡単である、与えられた$l,i,j$から$k=l-i-j$がただ$1$つ定まるため、
$k$が区間$[0,N)$に収まっているかをチェックし、収まっているなら$1$個、
収まっていなければ$0$個を答えればよい。

$$
\begin{align}
{\rm oracle_3}(l, i, j) = \begin{cases}
1 & 0 \le l-i-j < N \\
0 & (otherwise) \\
\end{cases}
\end{align}
$$

## 実装

以上の考察を踏まえPythonで実装したコードを以下に示す。

提出：[https://atcoder.jp/contests/abc200/submissions/22557207](https://atcoder.jp/contests/abc200/submissions/22557207)

```python
def sigma(n):
    return n*(n+1)//2

def oracle1(l):
    if l in range(0, N):
        return sigma(l+1)
    if l in range(N, 2*N-3+1):
        return sigma(l+1)-3*sigma(l+1-N)
    if l in range(2*N-3+1, 3*N-3+1):
        return sigma(3*N-2-l)
    return 0

def oracle2(l, i):
    L = max(0, l-i-(N-1))
    R = min(N-1, l-i)
    return max(0, R-L+1)

def oracle3(l, i, j):
    k = l - i - j
    if k in range(N):
        return 1
    return 0

N, K = map(int, input().split())
for l in range(0, 3*N-2):
    d = oracle1(l)
    if K-d <= 0:
        break
    K -= d
for i in range(N):
    d = oracle2(l, i)
    if K-d <= 0:
        break
    K -= d
for j in range(N):
    d = oracle3(l, i, j)
    if K-d <= 0:
        break
    K -= d
k = l - i - j
print(i+1, j+1, k+1)
```

また、グラフの作成に用いたコードも含めておく。

```python
import pandas as pd
import plotly.express as px
from itertools import product

N = 5

def level(l):
    if l in range(0, N):
        return "0≤l<N"
    if l in range(N, 2*N-2):
        return "N≤l<2N-2"
    if l in range(2*N-2, 3*N-2):
        return "2N-2≤l<3N-2"

df = pd.DataFrame(
    [
        [i,j,k,level(i+j+k)]
        for i,j,k in product(*[range(N)]*3)
    ],
    columns=['i','j','k','i+j+k=l']
)

fig = px.scatter_3d(
    df,
    x='i',
    y='j',
    z='k',
    color='i+j+k=l',
)

fig.update_layout(
    margin=dict(l=0,r=0,b=0,t=0),
    scene=dict(
        xaxis=dict(tickmode='linear',tick0=0,dtick=1),
        yaxis=dict(tickmode='linear',tick0=0,dtick=1),
        zaxis=dict(tickmode='linear',tick0=0,dtick=1),
    ),
    legend=dict(
        y=0.01,
        x=0.01,
    ),
)
fig.show()
```

## 関連する話題、

今回の問題は**K番目に小さい数を求める**という典型問題の一つで、
青diffの中でも
<span style="display: inline-block; border-radius: 50%; border-style: solid;border-width: 3px; margin-right: 5px; vertical-align: initial; height: 36px; width: 36px;border-color: #0000FF; background: linear-gradient(to top, #0000FF 0%, #0000FF 88.75%, rgba(0, 0, 0, 0) 88.75%, rgba(0, 0, 0, 0) 100%); "></span>
これくらいの難易度だったが、同様の考え方で解ける。緑diffの練習問題として
<span style="display: inline-block; border-radius: 50%; border-style: solid;border-width: 3px; margin-right: 5px; vertical-align: initial; height: 36px; width: 36px;border-color: #008000; background: linear-gradient(to top, #008000 0%, #008000 2%, rgba(0, 0, 0, 0) 2%, rgba(0, 0, 0, 0) 100%); "></span>


[abc061_c](https://atcoder.jp/contests/abc061/tasks/abc061_c)