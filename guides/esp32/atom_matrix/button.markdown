---
layout: default
permalink: /guides/esp32/atom_matrix/button
title: ワークショップガイド
---

# ボタン

Atom Matrixにはボタンが内臓されています。ディスプレイ部分についています。試しに軽く押してみてください

これから、このボタンを使っていきます

## 1. ボタンの値を読み取る

シリアルモニターで、irbを起動してください

```
$> irb
irb>
```

以下を入力してください
```
b = GPIO.new(39, GPIO::IN)
```

このような結果になります

```
b = GPIO.new(39, GPIO::IN)
I (22970) gpio: GPIO[39]| InputEn: 0| OutputEn: 0| OpenDrain: 0| Pullup: 1| Pulldown: 0| Intr:0 
=> #<GPIO:3ffb6738>
```

これでボタンを読み取る準備ができたので読み取ってみます

```
b.read
=> 1
```

Atom Matrixのボタンを押した状態でもう一度実行してください
```
b.read
=> 0
```

読み取った値がかわりました

こうやって読み取れるということは「押したら○○する」といった制御ができるということです


## 2. ボタンで止める

```ruby
loop do
  puts b.read
  break if b.read == 0
  sleep 1
end
```

これを実行するとボタンを押すまで1が出力され続け、ボタンを押すとループが終了できます
```
irb> loop do 
irb*  puts b.read
irb*  break if b.read == 0
irb*  sleep 1
irb* end
1
1
1
1
1
1
0
=> nil
```

- GPIO = マイコンが外の世界とやりとりする窓口で電気のON/OFFを読んだり、出したりできます。
- 39 → ATOM Matrixでボタンがつながってる場所
- GPIO::IN → 読み取り専用（Input = 入力）

これから先やっていくことの基本は全てこれです。
取得したり、指示を出したいセンサーと繋がるピン番号、それが何用かを指定し、値を読み取ったり書き込んだりしていきます。


一旦irbを終了します
```
exit
```
