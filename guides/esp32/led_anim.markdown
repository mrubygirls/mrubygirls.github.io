---
layout: default
permalink: /guides/esp32/led_anim
title: LEDアニメーション
---

## LEDアニメーション

Rubyのロゴが上下に動くアニメーションを作ります。
パラパラ漫画の原理で、LEDの表示を少しずつ変えて動いているように見せます。まずは動かしてみて、その後、色を変えたり、動きを変えたり、自分だけのパターンを作ってみましょう。
プログラムでアニメーションを制御する面白さを体験してください！

### Rubyロゴを表示

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

### 上下に動かす

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
  0x000000, 0xFF0000, 0xFF0000, 0xFF0000, 0x000000,
  0xFF0000, 0xFF0000, 0xFF0000, 0xFF0000, 0xFF0000,
  0x000000, 0xFF0000, 0xFF0000, 0xFF0000, 0x000000,
  0x000000, 0x000000, 0xFF0000, 0x000000, 0x000000,
  0x000000, 0x000000, 0x000000, 0x000000, 0x000000
]

3.times do
  led.show_hex(*up_pixels)
  sleep 0.3
  led.show_hex(*down_pixels)
  sleep 0.3
end
```

動く部分をメソッドにしてみて、繰り返し、ボタンで止めるようにします

```ruby
require 'ws2812'

button = GPIO.new(39, GPIO::IN)
rmt = RMTDriver.new(27)
led = WS2812.new(rmt)

# Rubyパターン
pixels = [
  [0x000000, 0xFF0000, 0xFF0000, 0xFF0000, 0x000000],
  [0xFF0000, 0xFF0000, 0xFF0000, 0xFF0000, 0xFF0000],
  [0x000000, 0xFF0000, 0xFF0000, 0xFF0000, 0x000000],
  [0x000000, 0x000000, 0xFF0000, 0x000000, 0x000000]
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

では、これをカスタマイズして、

色を変える（赤→青、緑、紫など）
速度を変える（sleep 0.3 → 速く/遅く）

オリジナルパターン作成：自分のイニシャル、ハート、星など
複数パターンの組み合わせ：2つのパターンを交互に表示
動きのバリエーション：回転、波、フェード
インタラクティブ要素：ボタンで動きを変える

### Tips
動かす部分を動的に書くこともできます

```
require 'ws2812'

button = GPIO.new(39, GPIO::IN)
rmt = RMTDriver.new(27)
led = WS2812.new(rmt)

# Rubyパターン
pixels = [
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