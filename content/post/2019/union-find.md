+++
title = "Union FindのメモとGoによる実装"
date = 2019-03-12T17:50:37+09:00
categories = ["技術"]
tags = ["競プロ", "golang", "データ構造"]
draft = false
description = "union findのまとめ"
+++



[AtCoder Beginners Content 120](https://atcoder.jp/contests/abc120/tasks)のD問題でUnionFindを使う問題が出題されたので学習した流れと実装をメモ．



# 問題

以下，[問題ページ（D: Decayed Bridges）](https://atcoder.jp/contests/abc120/tasks/abc120_d)より引用．

**問題文**:

> $N$ 個の島と $M$ 本の橋があります。
>
> $i$ 番目の橋は $A_i$ 番目の島と $B_i$ 番目の島を繋いでおり、双方向に行き来可能です。
>
> はじめ、どの 2 つの島についてもいくつかの橋を渡って互いに行き来できます。調査の結果、老朽化のためこれら $M$ 本の橋は 1 番目の橋から順に全て崩落することがわかりました。
>
> 「いくつかの橋を渡って互いに行き来できなくなった 2 つの島の組$ (a,b) (a<b) $の数」を**不便さ**と呼ぶことにします。
>
> 各 $i (1\leq i \leq M)$ について、$i$ 番目の橋が崩落した直後の不便さを求めてください。

<br>



**制約**: 

> 入力は全て整数である

> - $2\leq N \leq 10^5$
> - $1 \leq M \leq 10^5$
> - $1 \leq A_i \lt B_i \leq N$
> - $(A_i, B_i)$の組はすべて異なる
> - 初期状態における不便さは0である



# 全探索による解法

今回の問題は$O(NM)$が通らないので全探索は無理なのですが，そもそもグラフの問題をきちんと解いたことがなかったので，まずは素直に実装してみた．前から順番に橋を落としていき，毎回独立に0から隣接行列を計算して到達可能でない島の数を数えています．

<script src="https://gist.github.com/raahii/652851fb1f45ab0b365145cabe588c36.js"></script>

重要なのは新しい隣接情報が与えられたときに，隣の隣の島の情報もきちんと反映させることで，今回は一つ隣の島の隣接情報を拾う処理を$N-1$回繰り返すことでそれを実現しています．

この方法で書くにしてももっと効率よく書けそうな気もしますが，計算量は$MN^4​$でとにかく全く間に合いません．



# 代表頂点を使ってグループを管理する

全探索では駄目なので工夫が必要なのですが，まずグラフからノードを削除していく方針ではなく，グラフを0から順に構築していくように見ていく方針を取ることでより簡単に問題が解けます．確かに，隣接している島の情報を消していく操作は，その隣接に依存している他の隣接情報を解決する必要があり筋が悪そうです．

よって，まず不便さの初期値を$N(N-1)/2$とし，与えられた隣接情報を後ろからみて追加していくことで各不便さを求め，それを逆順に出力するようにします．

またここで，行き来可能な島の集合というのは，グラフにおけるグループだと考えることができます．このグループ管理を各頂点が所属するグループの代表頂点の番号を保存することで行うことにします．

こうすることで，2つの頂点が同じグループに属しているかどうかは**代表頂点の相違**で判断することができます．またこれは実装上，単なる配列の要素参照となり高速に実現できます．

これを使うと，新たな橋の隣接情報が与えられた時，2つの頂点が元々同じグループであれば不便さに変化はなし，違うグループであれば2つのグループが併合することになるので`お互いの要素数の積`だけ不便さが減少することになります．



<script src="https://gist.github.com/raahii/930f1a6509cfdccfb99143d54e8e564b.js"></script>



しかしながらこの方法でも，2つのグループを併合する際に，要素の移動（$O(N)$）が発生してしまうため，全体の計算量は$O(MN)$となりTLEとなってしいます．



# Union Find

そこでUnion Findが登場します．実はグループの管理の仕方自体は変わりません．変更点は，先程起きたグループの併合における計算量の問題を，代表頂点の代表頂点を書き換えることによって$O(1)$で行うところにあります．

ただし，この変更により，ある頂点の代表頂点は単に配列の要素を参照するのではなく，親の親…という風に再帰的に探索する必要がでてきます．これは特に要素1のグループが1つの階層構造を持ってしまった場合（全グループが縦に並ぶイメージ）に，探索が$O(N)​$かかってしまうため結局問題となります．よって，以下の2つを行うことでこれを解決します．

- 代表頂点の探索処理と同時に，その処理に関わった全てのノードを直接根につなぎ直す（多段になっている階層構造を解消する）
- 2つのグループを併合する時に，要素数の大きい側に少ない側を併合するようにする．

これにより，Union Findの計算量は少なくとも$O(\text{log}N)$未満となるようです．重要な制約として，グループの併合はできても分割はできないことが挙げられます．最終的なコードは以下の通りになります．

<script src="https://gist.github.com/raahii/f456ddf6109b2121dfd6f451d0476d6c.js"></script>



# 参考

- [Union-Find木の解説と例題 - Qiita](https://qiita.com/ofutonfuton/items/c17dfd33fc542c222396)
- [B: Union Find - AtCoder Typical Contest 001 | AtCoder](https://atc001.contest.atcoder.jp/tasks/unionfind_a)

- [Union Find - kumilog.net](https://www.kumilog.net/entry/union-find)