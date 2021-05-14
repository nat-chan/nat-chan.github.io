# ABC186-D Sum of difference
###### tags: `atcoder`
## 問題文
URL: https://atcoder.jp/contests/abc186/tasks/abc186_d

$N$個の整数$A_i,\cdots A_N$が与えられます。
$1\le i< j\le N$を満たす全ての$i,j$の組についての$|A_i-A_j|$の和を求めてください。
すなわち
\begin{align*}
\sum_{i=1}^{N-1}\sum_{j=i+1}^{N}|A_i-A_j|
\end{align*}
を求めてください。

## 制約
- $2\le N \le 2\times 10^5$
- $|A_i| \le 10^8$
- $A_i$は整数である

$|A_i-A_j|$の計算が$O(1)$かかるとすると愚直に計算すると
\begin{align*}
\sum_{i=1}^{N-1}\sum_{j=i+1}^{N}O(1) 
&=
O(\frac{N(N-1)}{2})
\\
&= O(N^2)
\end{align*}
の計算量がかかってしまう。
$1$秒間にコンピュータが計算できる量は$10^8$程度だと言われているので、
今回の制約では$O(N^2)$は通らない。上手く$O(N\log N)$や$O(N)$に落とすことを考える。

## ポイント1、Aをソートしても和が変わらない
分かりやすく例示するため$|A_i-A_j|$の二重和の$i$番目を$\circ$、$j$番目を$\times$のテーブルで表記する関数$g$を定義しておく。具体例として以下の4つの区間の和を考える。
### 1つめ、上左区間(Up Right)の二重和$UR$
今回求めたい$UR$は$g$を用いると以下のように表示できる。
$$
\begin{align*}
UR &= 
g\left(
\begin{array}{c|ccccc}
i＼j & 1 & 2 & 3 & .. & N \\
\hline
1&
\circ&\times&\times&\times&\times\\
2 &
&\circ&\times&\times&\times\\
3 &
&&\circ&\times&\times\\
..  &
&&&\circ&\times\\
N&
&&&&\circ\\
\end{array}
\right)
\\ &=
\sum_{i=1}^N\sum_{j=1}^N 
\begin{cases}
    |A_i-A_j| & (i< j) \\
    0         & (otherwise)
\end{cases}
\\ &=
\sum_{i=1}^{N-1}\sum_{j=i+1}^{N}|A_i-A_j|
\end{align*}
$$

### 2つめ、下右区間(Down Left)の二重和$DL$
一般に$2$変数関数$f(x,y)$の変数を入れ替えた$f(y,x)$のグラフは元のグラフと直線$y=x$で線対称なグラフとなる。
今回の被和分関数$f(i,j)=|A_i-A_j|=|A_j-A_i|$は$i,j$の入れ替えに対称であるので対角線$i=j$に対して線対称なグラフとなる。
別の言い方をすると行列 $ \lbrace\lbrace |A_i-A_j|\rbrace_{j=1}^N\rbrace_{i=1}^N$ は転置に対称な対称行列である。
$DL$の$i,j$を入れ替えると$UR$になるため、$DL=UR$が成り立つ。
 $$
\begin{align*}
DL &= 
g\left(
\begin{array}{c|ccccc}
i＼j & 1 & 2 & 3 & .. & N \\
\hline
1&
\circ&\\
2 &
\times&\circ\\
3 &
\times&\times&\circ\\
..  &
\times&\times&\times&\circ\\
N&
\times&\times&\times&\times&\circ\\
\end{array}
\right)
\\ &=
\sum_{i=1}^N\sum_{j=1}^N 
\begin{cases}
    |A_i-A_j| & (i>j) \\
    0         & (otherwise)
\end{cases}
\\ &=
\sum_{i=1}^N\sum_{j=1}^N 
\begin{cases}
    |A_j-A_i| & (j>i) \\
    0         & (otherwise)
\end{cases}
\\ &= UR
\end{align*}
$$

### 3つめ、対角線上(Diagonal)の二重和$DI$
$$
\begin{align*}
DI &= 
g\left(
\begin{array}{c|ccccc}
i＼j & 1 & 2 & 3 & .. & N \\
\hline
1&
\otimes \\
2 &
&\otimes \\
3 &
&&\otimes \\
..  &
&&&\otimes \\
N&
&&&&\otimes\\
\end{array}
\right)
\\ &=
\sum_{i=1}^N\sum_{j=1}^N 
\begin{cases}
    |A_i-A_j| & (i=j) \\
    0         & (otherwise)
\end{cases}
\\ &=
\sum_{i=1}^N |A_i-A_i|
\\ &= 0
\end{align*}
$$

### 4つめ、全区間(ALL)の二重和$AL$
$$
\begin{align*}
AL
&=
g\left(
\begin{array}{c|ccccc}
i＼j & 1 & 2 & 3 & .. & N \\
\hline
1&
\otimes&\times&\times&\times&\times\\
2 &
\times&\otimes&\times&\times&\times\\
3 &
\times&\times&\otimes&\times&\times\\
..  &
\times&\times&\times&\otimes&\times\\
N&
\times&\times&\times&\times&\otimes\\
\end{array}
\right)
\\ &=
\sum_{i=1}^N\sum_{j=1}^N  |A_i-A_j|
\end{align*}
$$
### 証明
$i=1,2,\dots,N$、$j=1,2,\dots,N$と動くとき、$(i,j)$の集合は$i<j$、$i>j$、$i=j$の関係で不足なく重複なく分割される。従って以下の関係式が成り立つ。
$$
\begin{align*}
AL &= UR+DL+DI \\
UR &= DL \\
DI &= 0 \\
∴UR &= \frac{AL}{2}
\end{align*}
$$
足し算$+$は交換法則$a+b=b+a$が成り立つため
$$
\begin{align*}
AL &=
\sum_{i=1}^N\sum_{j=1}^N  |A_i-A_j|
\end{align*}
$$
は$A$の順序を任意に入れ替えても一致する。上記の関係式より$UR$も同様である。
## ポイント２、Aが単調増加なら絶対値が外れる
上記の議論より以降$A$はsortされていると仮定してよい。
quick sortは$O(N\log N)$の計算量で済むので今回の制約を満たす。
このとき$A$は次の広義単調増加性を満たす。
$$
\begin{align*}
i \le j ⇒ A_i \le A_j
\end{align*}
$$
$A_i - A_j \le 0$のとき絶対値を取ると符号が反転するので、
$$
\begin{align*}
UR
 &=
\sum_{i=1}^{N-1}\sum_{j=i+1}^{N}
|A_i-A_j|
\\ &=
\sum_{i=1}^{N-1}\sum_{j=i+1}^{N}
\begin{cases}
    -A_i+A_j & (A_i\le A_j) \\
    A_i-A_j & (A_i>A_j) \\
\end{cases}
\\ &=
\sum_{i=1}^{N-1}\sum_{j=i+1}^{N}
-A_i+A_j
\end{align*}
$$
このように二重和から絶対値を外すことができた。
## ポイント３、二重和を一重和に落とす。
上記の議論より$i,j$が動くとき$A_i$が常にマイナス、$A_j$が常にプラスの符号が付いて足し合わされることが分かった。これら２つの部分和をそれぞれ求めていく。
まず$i$を固定したときに$A_i$が何回足し合わされたかを調べる。
これは$g$関数で表示したときに各行で$\circ$の右にある$\times$が何個あるか数え上げれば分かる。
$$
\begin{align*}
UR &= 
g\left(
\begin{array}{c|ccccc}
i＼j & 1 & 2 & 3 & j & N \\
\hline
1&
\circ_{N-1}&\{\times&\times&\times&\times\}\\
2 &
&\circ_{N-2}&\{\times&\times&\times\}\\
3 &
&&\circ_{N-3}&\{\times&\times\}\\
i  &
&&&\circ_{N-i}&\{\times\}\\
N&
&&&&\circ_{N-N}\\
\end{array}
\right)
\end{align*}
$$
$i$行目は$(N-i)$回$A_i$が足し合わされることが分かったので前半のマイナス項は以下のように一重和に落とせる。
$$
\begin{align*}
UR &= -
\sum_{i=1}^{N-1}\sum_{j=i+1}^{N}
A_i
 +
\sum_{i=1}^{N-1}\sum_{j=i+1}^{N}
A_j
\\ &= -
\sum_{i=1}^{N-1} A_i
\sum_{j=i+1}^{N} 1
 +
\sum_{i=1}^{N-1}\sum_{j=i+1}^{N}
A_j
\\ &= -
\sum_{i=1}^{N-1} A_i
(N-i)
 +
\sum_{i=1}^{N-1}\sum_{j=i+1}^{N}
A_j
\end{align*}
$$
次に$j$を固定したときに$A_j$が何回足し合わされたかを調べる。
これも$g$関数で表示したときに各列で$\circ$の上にある$\times$が何個あるか数え上げれば分かる。
$$
\begin{align*}
UR &= 
g\left(
\begin{array}{c|ccccc}
i＼j & 1 & 2 & 3 & j & N \\
\hline
1&
\overset{0}{\circ}&\underbrace{\overbrace{\times}}&\overbrace{\times}&\overbrace{\times}&\overbrace{\times}\\
2 &
&\overset{1}{\circ}&\underbrace{\times}&\times&\times\\
3 &
&&\overset{2}{\circ}&\underbrace{\times}&\times\\
i  &
&&&\overset{j-1}{\circ}&\underbrace{\times}\\
N&
&&&&\overset{N-1}{\circ}\\
\end{array}
\right)
\end{align*}
$$

$j$行目は$(j-1)$回$A_j$が足し合わされることが分かったので後半のプラス項も以下のように一重和に落とせる。
$$
\begin{align*}
UR &= -
\sum_{i=1}^{N-1} A_i
(N-i)
 +
\sum_{j=2}^{N}\sum_{i=1}^{j-1}
A_j
\\ &= -
\sum_{i=1}^{N-1} A_i
(N-i)
 +
\sum_{j=2}^{N}
A_j
\sum_{i=1}^{j-1}
1
\\ &= -
\sum_{i=1}^{N-1} A_i
(N-i)
 +
\sum_{j=2}^{N}
A_j(j-1)
\\ &=
-\sum_{i=1}^{N} A_i(N-i)
+\sum_{i=1}^{N} A_i(i-1)
\\ &=
\sum_{i=1}^{N} A_i(2i-N-1)
\end{align*}
$$

これで求めたい和の計算量を$O(N^2)$から$O(N)$へと落とすことができた。全体の計算量はquick sortがボトルネックとなり$O(N\log N)$となる。
## Python3で実装
配列の添え字が$0$から始まることと`range(1,N)`が$1,2,\dots,N-1$まで動き$N$を含まないことに注意して実装する。Python3はfor文が遅いのでコンテスト中はPyPy3で提出したが、Python3でも最悪`198 ms`と余裕をもって`AC`で通ることを確認してある。
```python linenums="1"
N = int(input())
A = list(map(int, input().split()))
A.sort()
print(sum((2*i-N+1)*A[i] for i in range(N)))
```