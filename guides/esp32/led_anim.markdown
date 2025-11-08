---
layout: default
permalink: /guides/esp32/led_anim
title: LEDアニメーション
---

# LEDアニメーション

![LED](/images/guides/atom_led_anim.gif)

Rubyのロゴが上下に動くアニメーションを作ります
パラパラ漫画の原理で、LEDの表示を少しずつ変えて動いているように見せます。まずは動かしてみて、その後、色を変えたり、動きを変えたり、自分だけのパターンを作ってみましょう

## 1. gemのインストール

他のステップでまだインストールしていない場合は行ってください


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
rake flash
```

再度シリアルモニターを立ち上げます。
```
rake monitor
```

## 2. Rubyロゴを表示

```ruby
require 'ws2812'

rmt = RMTDriver.new(27)
led = WS2812.new(rmt)

# Rubyロゴを1つの位置に表示
pixels = [
  0x000000, 0xFF0000, 0xFF0000, 0xFF0000, 0x000000,
  0xFF0000, 0xFF0000, 0xFF0000, 0xFF0000, 0xFF0000,
  0x000000, 0xFF0000, 0xFF0000, 0xFF0000, 0x000000,
  0x000000, 0x000000, 0xFF0000, 0x000000, 0x000000,
  0x000000, 0x000000, 0x000000, 0x000000, 0x000000
]
led.show_hex(*pixels)
```

## 3. 上下に動かす

3回上下に動かしてみます
```ruby
require 'ws2812'

rmt = RMTDriver.new(27)
led = WS2812.new(rmt)

# Rubyロゴを1つの位置に表示
up_pixels = [
  0x000000, 0xFF0000, 0xFF0000, 0xFF0000, 0x000000,
  0xFF0000, 0xFF0000, 0xFF0000, 0xFF0000, 0xFF0000,
  0x000000, 0xFF0000, 0xFF0000, 0xFF0000, 0x000000,
  0x000000, 0x000000, 0xFF0000, 0x000000, 0x000000,
  0x000000, 0x000000, 0x000000, 0x000000, 0x000000
]

down_pixels = [
  0x000000, 0x000000, 0x000000, 0x000000, 0x000000,
  0x000000, 0xFF0000, 0xFF0000, 0xFF0000, 0x000000,
  0xFF0000, 0xFF0000, 0xFF0000, 0xFF0000, 0xFF0000,
  0x000000, 0xFF0000, 0xFF0000, 0xFF0000, 0x000000,
  0x000000, 0x000000, 0xFF0000, 0x000000, 0x000000
]

3.times do
  led.show_hex(*up_pixels)
  sleep 0.3
  led.show_hex(*down_pixels)
  sleep 0.3
end
```

動く部分をループにしてみて、繰り返し、ボタンで止めるようにします

```ruby
require 'ws2812'

button = GPIO.new(39, GPIO::IN)
rmt = RMTDriver.new(27)
led = WS2812.new(rmt)

# Rubyロゴを1つの位置に表示
up_pixels = [
  0x000000, 0xFF0000, 0xFF0000, 0xFF0000, 0x000000,
  0xFF0000, 0xFF0000, 0xFF0000, 0xFF0000, 0xFF0000,
  0x000000, 0xFF0000, 0xFF0000, 0xFF0000, 0x000000,
  0x000000, 0x000000, 0xFF0000, 0x000000, 0x000000,
  0x000000, 0x000000, 0x000000, 0x000000, 0x000000
]

down_pixels = [
  0x000000, 0x000000, 0x000000, 0x000000, 0x000000,
  0x000000, 0xFF0000, 0xFF0000, 0xFF0000, 0x000000,
  0xFF0000, 0xFF0000, 0xFF0000, 0xFF0000, 0xFF0000,
  0x000000, 0xFF0000, 0xFF0000, 0xFF0000, 0x000000,
  0x000000, 0x000000, 0xFF0000, 0x000000, 0x000000
]

loop do
  break if button.read == 0
  led.show_hex(*up_pixels)
  sleep 0.3
  led.show_hex(*down_pixels)
  sleep 0.3
end

# 最後に消灯
led.show_hex(*Array.new(25, 0x000000))
```

## 4. カスタマイズしよう

好きなアニメーションを作って表示してみましょう

- たとえば
  - 色を変える（赤→青、緑、紫など）
  - 速度を変える（sleep 0.3 → 速く/遅く）
  - オリジナルパターン作成：自分のイニシャル、ハート、星など
  - 複数パターンの組み合わせ：2つのパターンを交互に表示
  - 動きのバリエーション：回転、波、フェード
  - インタラクティブ要素：ボタンで動きを変える


## 5. 作品ができたら

`app.rb` という名前のファイルがあれば、電源が入るとそのプログラムが自動的に実行されます

`storage/home/` に `app.rb` という名前で保存して、 `rake flash` で書き込んでみよう


## Tips
動かす部分を動的に書くこともできます

```ruby
require 'ws2812'

button = GPIO.new(39, GPIO::IN)
rmt = RMTDriver.new(27)
led = WS2812.new(rmt)

# Rubyパターン
ruby = [
  [0x000000, 0xFF0000, 0xFF0000, 0xFF0000, 0x000000],
  [0xFF0000, 0xFF0000, 0xFF0000, 0xFF0000, 0xFF0000],
  [0x000000, 0xFF0000, 0xFF0000, 0xFF0000, 0x000000],
  [0x000000, 0x000000, 0xFF0000, 0x000000, 0x000000]
]

def draw_pattern(led, pattern, start_row, rows)
  pixels = Array.new(25, 0x000000)
  
  rows.times do |y|
    actual_row = start_row + y
    if actual_row >= 0 && actual_row < 5
      5.times do |x|
        index = actual_row * 5 + x
        pixels[index] = pattern[y][x]
      end
    end
  end
  
  led.show_hex(*pixels)
end

# Rubyロゴを上下に移動
loop do
  break if button.read == 0
  
  # 上から下へ
  0.upto(1) do |row|
    draw_pattern(led, ruby, row, 4)
    sleep 0.3
  end
  
  # 下から上へ
  1.downto(0) do |row|
    draw_pattern(led, ruby, row, 4)
    sleep 0.3
  end
end

# 最後に消灯
led.show_hex(*Array.new(25, 0x000000))
```