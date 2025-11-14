---
layout: default
permalink: /guides/esp32/led
title: LED
---

# LED

Atom Matrixには25個のLEDが組み込まれています

このLEDはWS2812というチップが使われていて、少ない配線と簡単なプログラムで、非常にカラフルでダイナミックな光の表現を可能にする、マイコン工作において最も人気のあるLEDチップです

## 1. LED配置図

25個のLEDは、ジグザグ状に配置されていて、一本の線で繋がっています
```
 0  1  2  3  4
 9  8  7  6  5
10 11 12 13 14
19 18 17 16 15
20 21 22 23 24
```

## 2. gemのインストール

WS2812をESP32で使うために必要なことがあるのですが、今回は、あらかじめ、簡単に利用できるようにしたgemを使います


PCで `R2P2-ESP32/components/picoruby-esp32/picoruby/build_config/xtensa-esp.rb`
この設定ファイルを開いて以下を追加してください

```ruby
  conf.gem core: 'picoruby-pwm'

  conf.gem github: 'ksbmyk/picoruby-ws2812', branch: 'main' # 追加

  conf.picoruby(alloc_libc: false)
```

シリアルモニタを立ち上げている場合は一度 `Ctl+]` で終了してください

追加した設定ファイルの状態で、再度buildして、追加したgemを取り込みます。

```
rake clean build
```

この内容でマイコンに書き込みます。

```
ESPBAUD=115200 rake flash
```

再度シリアルモニターを立ち上げます。
```
rake monitor
```

## 3. IRBで点灯

まずは1つ点灯させます。
これは最初にインストールしたWS2812 gemを読み込むためのコードです。WS2812というLEDを簡単に制御できるgemを使います。

シリアルモニターで、irbを起動してください

```
$> irb
irb>
```

以下を打ち込んでください。先ほど追加したgemを使うので `require 'ws2812'` としています。
27というのは、AtomMatrixでLEDが繋がっているpinの番号です。

```ruby
require 'ws2812'

rmt = RMTDriver.new(27)
led = WS2812.new(rmt)

led.show_rgb([255, 0, 0])
```

左上、0番目のLEDが赤になりました

show_hexメソッドを使うと16進でも色を指定できます
```ruby
led.show_hex(0xFF0000)
```


好きな色に変えてみましょう
```
# RGB値で指定
led.show_rgb([0, 255, 0])    # 緑
led.show_rgb([0, 0, 255])    # 青
led.show_rgb([255, 255, 0])  # 黄色
led.show_rgb([255, 0, 255])  # マゼンタ

# または16進数で
led.show_hex(0x00FF00)  # 緑
led.show_hex(0x0000FF)  # 青
```

複数点灯させます。指定した前から順番に0番目、2番目と解釈されていきます。

```
# RGB値で指定
led.show_rgb([255, 0, 0], [0, 255, 0], [0, 0, 255])

# または16進数で
led.show_hex(0xFF0000, 0x00FF00, 0x0000FF)
```

黒を指定することで消灯させることができます。
```
# RGB値で指定
led.show_rgb([0, 0, 0], [0, 0, 0], [0, 0, 0])

# または16進数で
led.show_hex(0x000000, 0x000000, 0x000000)
```

## 4. ファイルを送る

長いコードをirbで書くのは大変なので（現在esp32用のR2P2ではカーソルキーが使えなかったりコピー＆ペーストができなかったりします）
手元でファイルを作り、それを実行します

一度、irbを `exit` で終了し、シリアルモニターを `Ctr+]` で終了してください。

PCで以下の内容のファイルを作り、 `R2P2-ESP32/storage/home/` の下に、 `square.rb` という名前で保存してください

```ruby
require 'ws2812'

rmt = RMTDriver.new(27)
led = WS2812.new(rmt)

# 配列を使って全LEDを制御
pixels = [ # 全て消灯
  [0, 0, 0], [0, 0, 0], [0, 0, 0], [0, 0, 0], [0, 0, 0],
  [0, 0, 0], [0, 0, 0], [0, 0, 0], [0, 0, 0], [0, 0, 0],
  [0, 0, 0], [0, 0, 0], [0, 0, 0], [0, 0, 0], [0, 0, 0],
  [0, 0, 0], [0, 0, 0], [0, 0, 0], [0, 0, 0], [0, 0, 0],
  [0, 0, 0], [0, 0, 0], [0, 0, 0], [0, 0, 0], [0, 0, 0],
]

# 四隅だけ光らせる
pixels[0] = [255, 0, 0]    # 左上
pixels[4] = [0, 255, 0]    # 右上
pixels[20] = [0, 0, 255]   # 左下
pixels[24] = [255, 255, 0] # 右下

led.show_rgb(*pixels)
```


PCに保存したファイルをマイコンに転送します

```
ESPBAUD=115200 rake flash
```

もう一度シリアルモニターを立ち上げます


```
rake monitor
```

シリアルモニターで `ls` を実行すると先ほど転送したファイルが見えます

```
$> ls
square.rb
```


このファイルを実行します

```
./square.rb
```


四隅が違う色で光りました


## Tips

ファイルを続けて実行すると、以下のようなメッセージが表示されることがあります

```
W (xxxxx) rmt: GPIO 27 is not usable, maybe conflict with others
```

これは警告メッセージですが、LEDの制御には問題ありません。気にせず次のステップに進んでください

