## <宿題１：ハッシュテーブル>
キーの何文字目かの記録をするcountを掛けることにより、アナグラム同士が同じハッシュ値を持つことを避けようと試みた。
> ex) alice -> org(‘a’)*1 + org(‘l’)*2 + org(‘i’)*3 + org(‘c’)*4 + org(‘e’)*e5

しかしここに素数をかけようと何をしようと実行時間が10秒に達することが多かった為、ビットシフトとの掛け合わせを試したところ大きく短縮することができた。\
いずれの案でも複数のテストケースに生まれる差への解決策、また素数の利用よりビットシフトがより効果的な理由は思いついていない。\
以下は上記の計算を試した際の実行時間を秒単位で記した表で、いくつか実行に時間がかかったケースから最長で合った57番目のケースをピックアップしている。

 ハッシュ値 \ テストケース        | 1      | 40    | 57     | 99   
 ------------------------------ | ------ | ----- | ------ | ----- 
 hash += ord(i) * count         | 0.0826 | 4.764 | 13.213 | 12.672
 hash += ord(i) << idx          | 0.0462 | 0.447 | 3.474  | 1.268
 hash += ord(i) * count << idx  | 0.0534 | 0.090 | 2.820  | 0.184
 hash += ord(i) *10^(count-1)   | 0.0380 | 0.026 | 1.432  | 0.030
 
追記) キーの各桁に対して 10^(count-1) を掛けてみたら一番早く計算が終わった…代わりに、長いキーではハッシュ値の桁数が大きくなってしまうのではないかと懸念もしている


 ## <宿題２：ハッシュテーブル vs 木構造>
 ##### 木構造が好まれるわけ
 ・ハッシュはメモリの使用量が多いから\
衝突を減らすことを目的にテーブルのサイズがある程度確保されている為、余分なメモリまで消費することになる\
・木構造では関連性のあるデータを扱いやすいから\
ハッシュテーブルは検索等が容易である反面、一つ一つのデータの関連性を無視している為にランダムに格納されてしまう。そのためデータ同士の関連性を残したい場合には木構造のほうが適している。
・実行時間が不安定だから
ハッシュ値を工夫しようと、多くの衝突が起こる可能性自体は存在してしまう。

## <宿題３：キャッシュ>
直近に訪問したサイトを先頭、一番過去に訪問したサイトを末尾に置く双方向連結リストをハッシュテーブルと組み合わせてキャッシュを作る。
```
Node(){
url = https://...
page = ...
previous = ...
next = ...
}
```
以上の要素を持つノードとして、アクセスしたサイトを記録し双方向連結リストを作成する。これにより、訪問順を保管しつつ、キャッシュ内に訪問歴のない場合は新たに先頭へ追加し、末尾のサイトの記録を取り出せるようになる。この際headとtailとなるノードも記録しておく。また、各ノードをハッシュテーブルにも入れることによって、キャッシュ内の記録の有無をO(1)で確認可能にする。\
サイトのURLをキーとして扱い、ハッシュテーブルにて一致するキーがあるかを検索する。そののちの動作は以下のようになる。
### ①アクセス歴のない場合
新たなノードを作成し、ページの情報を記録する。そのうえにheadをこの新ノードに向け、nextを現headとして記録する。同様に現headのpreviousを新ノードへと変更する。その後tailのノードを取り出し、同一のキー(URL)を持つをハッシュで検索して削除、新ノードを追加。
### ②アクセス歴のある場合 / 再訪問
ハッシュ内の該当するノードに対し、'node.previous.next = node.next'、'node.next.previous = node.previous'として前後をリンクさせる。それから該当ノードがheadとなるように、①と同じようにしてpreviousとnextを変更していく。(既にheadである場合は変更なし)　