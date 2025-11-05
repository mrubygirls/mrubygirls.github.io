---
layout: default
permalink: /guides/esp32/tilt_led
title: デジタル水準器
---


# デジタル水準器

傾けた方向にLEDが光る、水準器のようなものを作ります

加速度センサーで傾きを検知して、その方向をLEDで表示します。段階的に作っていくので、センサーの値がどう変わるか、それをどうLEDに反映させるかを理解しながら進められます

完成したら、色を変えたり、振ったら光るようにしたり、自由にアレンジしてみましょう！


## 1. gemのインストール

もし、他のステップでgemをインストールしていない場合は行ってください

PCで `R2P2-ESP32/components/picoruby-esp32/picoruby/build_config/xtensa-esp.rb`
この設定ファイルを開いて以下を追加してください

```ruby
  conf.gem core: 'picoruby-pwm'

  conf.gem github: 'ksbmyk/picoruby-ws2812', branch: 'main' # 追加
  conf.gem github: 'bash0C7/picoruby-mpu6886', branch: 'main' # 追加

  conf.picoruby(alloc_libc: false)
```

シリアルモニタを立ち上げている場合は一度 `Ctl+]` で終了してください

追加した設定ファイルの状態で、再度buildして、追加したgemを取り込みます

```
rake clean build
```

この内容でマイコンに書き込みます

```
rake flash
```

再度シリアルモニターを立ち上げます
```
rake monitor
```

## 2. 値をとる

PCで以下の内容のファイルを作り、 `R2P2-ESP32/storage/home/` の下に、 `tlit_led.rb` という名前で保存してください

```ruby
require 'i2c'
require 'mpu6886'

i2c = I2C.new(
  unit: :ESP32_I2C0,
  frequency: 100_000,
  sda_pin: 25,
  scl_pin: 21
)

mpu = MPU6886.new(i2c)
button = GPIO.new(39, GPIO::IN)

puts "---"

loop do
  break if button.read == 0 

  accel = mpu.acceleration
  
  if accel[:x] > 0.3
    puts "右に傾いています！ X軸: #{accel[:x]}"
  elsif accel[:x] < -0.3
    puts "左に傾いています！ X軸: #{accel[:x]}"
  end
  
  if accel[:y] > 0.3
    puts "前に傾いています！ Y軸: #{accel[:y]}"
  elsif accel[:y] < -0.3
    puts "後ろに傾いています！ Y軸: #{accel[:y]}"
  end
  
  puts "加速度 X:#{accel[:x]} Y:#{accel[:y]} Z:#{accel[:z]} (G単位)"
  puts "---"
  
  sleep_ms 200
end
```

シリアルモニターを `Ctr+]` で終了し、 `rake flash` で書き込み、 `rake monitor` でシリアルモニタで立ち上げ

転送したファイルを実行してください
```
./tlit_led.rb
```

かたむけてみてください

## 3 中央が光るようにする

tlit_led.rb に変更を加えて、中央が光るようにしましょう

```ruby
require 'i2c'
require 'mpu6886'
require 'ws2812'

i2c = I2C.new(
  unit: :ESP32_I2C0,
  frequency: 100_000,
  sda_pin: 25,
  scl_pin: 21
)
mpu = MPU6886.new(i2c)
button = GPIO.new(39, GPIO::IN)

rmt = RMTDriver.new(27)
led = WS2812.new(rmt)
# 最初に全消灯
led.show_hex(*Array.new(25, 0x000000))

loop do
  break if button.read == 0 

  accel = mpu.acceleration
  
  pixels = Array.new(25, 0x000000)
  # 中心のみ緑で光らせる
  pixels[12] = 0x001E00
  led.show_hex(*pixels)

  if accel[:x] > 0.3
    puts "右に傾いています！ X軸: #{accel[:x]}"
  elsif accel[:x] < -0.3
    puts "左に傾いています！ X軸: #{accel[:x]}"
  end
  
  if accel[:y] > 0.3
    puts "前に傾いています！ Y軸: #{accel[:y]}"
  elsif accel[:y] < -0.3
    puts "後ろに傾いています！ Y軸: #{accel[:y]}"
  end
  
  sleep_ms 200
end

led.show_hex(*Array.new(25, 0x000000))
```


シリアルモニターを `Ctr+]` で終了し、 `rake flash` で書き込み、 `rake monitor` でシリアルモニタで立ち上げ

転送したファイルを実行してください
```
./tlit_led.rb
```

中央が光りました。ボタンを押すと消えます

## 4. X軸（左右）の傾きで光らせる

tlit_led.rb に変更を加えて、傾けたら光るようにしてみましょう

```ruby
require 'i2c'
require 'mpu6886'
require 'ws2812'

i2c = I2C.new(
  unit: :ESP32_I2C0,
  frequency: 100_000,
  sda_pin: 25,
  scl_pin: 21
)
mpu = MPU6886.new(i2c)
button = GPIO.new(39, GPIO::IN)

rmt = RMTDriver.new(27)
led = WS2812.new(rmt)
led.show_hex(*Array.new(25, 0x000000))

loop do
  break if button.read == 0 

  accel = mpu.acceleration
  
  pixels = Array.new(25, 0x000000)
  pixels[12] = 0x001E00

  if accel[:x] > 0.3
    # 右に傾いた
    pixels[13] = 0x1E0000
    if accel[:x] > 0.6
      pixels[14] = 0x1E0000
    end
  elsif accel[:x] < -0.3
    # 左に傾いた
    pixels[11] = 0x1E0000
    if accel[:x] < -0.6
      pixels[10] = 0x1E0000
    end
  end
  
  if accel[:y] > 0.3
    pixels[17] = 0x00001E
    if accel[:y] > 0.6
      pixels[22] = 0x00001E
    end
  elsif accel[:y] < -0.3
    pixels[7] = 0x00001E
    if accel[:y] < -0.6
      pixels[2] = 0x00001E
    end
  end

  led.show_hex(*pixels)
  
  sleep_ms 200
end

led.show_hex(*Array.new(25, 0x000000))
```

シリアルモニターを `Ctr+]` で終了し、 `rake flash` で書き込み、 `rake monitor` でシリアルモニタで立ち上げ

転送したファイルを実行してください
```
./tlit_led.rb
```

前後左右に傾けると光ります


## 5. カスタマイズしてみよう

好きにカスタマイズしてみましょう

- たとえば
  - 傾けた方向全体を光らせる
  - 傾きの強さでグラデーション
  - 振ったら（加速度が大きい時）全点灯
  - 水平な時だけ光るパターン
  - デジタルサイコロ

## 6. 作品ができたら

`app.rb` という名前のファイルがあれば、電源が入るとそのプログラムが自動的に実行されます

`storage/home/` に `app.rb` という名前で保存して、 `rake flash` で書き込んでみよう
