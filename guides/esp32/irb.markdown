---
layout: default
permalink: /guides/esp32/irb
title: irb
---


## irbを動かしてみよう

`rake monitor` で R2P2を起動し

irbを実行します。
irbとは「対話モード」打ったらすぐ結果が返ってきます。

起動
```
$> irb
irb>
```

irbが起動したら、先頭が `irb>` となります


計算
```
irb> 1+1
=> 2
```

Array
```
[1,2,3].first
=> 1
```

繰り返し
```
3.times {puts "Hi"}
Hi
Hi
Hi
=> 3
```

```
5.times { |i| puts 5-i; sleep 1}
5
4
3
2
1
=> 5
```

### もし `NoMethodError` エラーが出たら？

たとえば配列は扱えますが、 `Array#flatten` は使えません

```
[[1,2],[3,4]].flatten
=> #<NoMethodError:3ffcc9c4>
```

ATOM Matrixのメモリは約0.5MB、みなさんのPCの一万分の1以下です。
PicoRubyはマイコン用のRubyなので、厳選したよく使う機能だけが入ってます。

Rubyのメソッドのうち何が使えるかは実装を見ることでもわかりますが、気軽にirbで事項してみて、使える/使えないをためしてみるのがおすすめです。

### irbの終わり方

```
exit
```
