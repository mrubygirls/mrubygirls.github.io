---
layout: default
permalink: /guides/esp32/irb
title: IRB
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


## センサーを使ってみよう

### ボタン

irbを起動してください

```
b = GPIO.new(39, GPIO::IN)
I (22970) gpio: GPIO[39]| InputEn: 0| OutputEn: 0| OpenDrain: 0| Pullup: 1| Pulldown: 0| Intr:0 
=> #<GPIO:3ffb6738>
```

```
b.read
=> 1
```

Atom Matrixのボタンを押した状態でもう一度実行してください
```
b.read
=> 0
```

読み取った値がかわりました。
こうやって読み取れるということは「押したら○○する」といった制御ができるということになります。

- GPIO = マイコンが外の世界とやりとりする窓口で電気のON/OFFを読んだり、出したりできます。
- 39 → ATOM Matrixでボタンがつながってる場所
- GPIO::IN → 読み取り専用（INput = 入力）

これから先やっていくことの基本は全てこれです。
取得したり、指示を出したいセンサーと繋がるピン番号、それが何用かを指定し、値を読み取ったり書き込んだりしていきます。