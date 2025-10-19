---
layout: default
permalink: /guides/esp32/accel
title: 加速度センサー
---

## 加速度センサー

Atom Matrixには加速度センサーが内臓されています

### 加速度センサーとは
皆さんのスマホには加速度センサーが入っています。スマホを縦から横に傾けると、画面が自動で回転しますよね？あれは加速度センサーが「今、どっちに傾いているか」を検知しているからです。今日使うATOM Matrixにも加速度センサーが入っています。スマホのものとは製品は違いますが、「傾きや動きを数値で検知する」という役割は同じです。


### 使ってみる

実際に加速度センサーをさわってみましょう。

一度シリアルモニターを終了して

シリアルモニターを `Ctr+]` で終了して
以下のファイルを作って、保存して `home/strage/` 配下に、`accel.rb` という名前で置いてください

```
require 'i2c'
require 'mpu6886'

# センサー初期化
i2c = I2C.new(unit: :ESP32_I2C0, frequency: 100_000, sda_pin: 25, scl_pin: 21)
mpu = MPU6886.new(i2c)

# ボタン設定
button = GPIO.new(39, GPIO::IN)

loop do
  break if button.read == 0 # ボタンを押すと終了
  
  accel = mpu.acceleration
  puts "X:#{accel[:x]}  Y:#{accel[:y]}  Z:#{accel[:z]}"
  
  sleep_ms 300
end
```


`rake flash`で書き込み、`rake monitor`でシリアルモニターを立ち上げてください


シリアルモニターで実行します
```
./accel.rb
```

右に傾けるとXが負の数、左に傾けると正の数、奥に傾けるとYが負の数、手前に傾けると正の数、真上に加速すると、Zが増加、真下に加速するとZが減少します

確かめたら、ボタンを押して終了してください

### 閾値

状態を判定するには閾値を設けます。たとえば閾値が0.3以上なら傾いていると判定します

シリアルモニターを `Ctr+]` で終了して
以下のファイルを作って、保存して `home/strage/` 配下に、`tilt.rb` という名前で置いてください


```ruby tilt.rb
require 'i2c'
require 'mpu6886'

puts "傾き検知プログラム開始"

# ATOM Matrix用のI2C設定
i2c = I2C.new(
  unit: :ESP32_I2C0,
  frequency: 100_000,
  sda_pin: 25,
  scl_pin: 21
)

# MPU6886センサーを初期化
mpu = MPU6886.new(i2c)
# ボタン設定
button = GPIO.new(39, GPIO::IN)

puts "---"

# センサーデータを繰り返し読み取る
loop do
  break if button.read == 0 # ボタンを押したら終了

  # 加速度を取得（単位：G、重力加速度）
  accel = mpu.acceleration
  
  # X軸方向の傾きを判定（閾値：0.3G）
  if accel[:x] > 0.3
    puts "右に傾いています！ X軸: #{accel[:x]}"
  elsif accel[:x] < -0.3
    puts "左に傾いています！ X軸: #{accel[:x]}"
  end
  
  # Y軸方向の傾きを判定（閾値：0.3G）
  if accel[:y] > 0.3
    puts "前に傾いています！ Y軸: #{accel[:y]}"
  elsif accel[:y] < -0.3
    puts "後ろに傾いています！ Y軸: #{accel[:y]}"
  end
  
  # 詳細な数値も表示
  puts "加速度 X:#{accel[:x]} Y:#{accel[:y]} Z:#{accel[:z]} (G単位)"
  puts "---"
  
  # 200ミリ秒待機
  sleep_ms 200
end
```

`rake flash`で書き込み、`rake monitor`でシリアルモニターを立ち上げてください



シリアルモニターで実行します
```
./tilt.rb
```

今回は0.3としていますが、それは使うセンサーや目的次第です。少しでも傾けたら方向を判定したいなら閾値0.1とか、大きく傾けた時にはじめて判定したければ0.5とか自分の実現したいことに合わせて設定します。
