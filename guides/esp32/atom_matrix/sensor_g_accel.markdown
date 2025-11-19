---
layout: default
permalink: /guides/esp32/atom_matrix/sensor_g_accel
title: 6軸モーションセンサー
---

# 6軸モーションセンサー

Atom MatrixにはMPU6886という6軸モーションセンサーが搭載されています

## 1. 6軸モーションセンサとは

6軸モーションセンサーとは、「加速度センサー（3軸）」と「ジャイロスコープ（3軸）」の合計6つの動きを測定できる機能を一つのパッケージにまとめたセンサーの総称です。

加速度センサーは、デバイスがどちらに傾いているか（床に対して水平か、垂直か）を、地球の重力（1G）の力を利用して測ったり、デバイスがどれくらいの勢いで動き出したか、または止まったかを測れます

ジャイロスコープは、デバイスが1秒間に何度の速さで回転したか（または向きを変えたか）を正確に測ります。

両方の情報を組み合わせることで、「今、どの方向に、どれくらいのスピードで傾きながら回転しているか」という、複雑で正確な動きの情報を得ることができます

## 2. gemのインストール

MPU6886をESP32で使うために必要なことがあるのですが、今回は、あらかじめ、簡単に利用できるようにしたgemを使います

PCで `R2P2-ESP32/components/picoruby-esp32/picoruby/build_config/xtensa-esp.rb`
この設定ファイルを開いて以下を追加してください

```ruby
  conf.gem core: 'picoruby-pwm'

  conf.gem github: 'bash0C7/picoruby-mpu6886', branch: 'main' # 追加

  conf.picoruby(alloc_libc: false)
```

シリアルモニタを立ち上げている場合は一度 `Ctl+]` で終了してください

追加した設定ファイルの状態で、再度buildして、追加したgemを取り込みます

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

## 3. 値をとる

実際に加速度センサーをさわってみましょう。

一度シリアルモニターを終了して

シリアルモニターを `Ctr+]` で終了して
以下のファイルを作って、保存して `storage/home/` 配下に、`accel.rb` という名前で置いてください

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

`ESPBAUD=115200 rake flash`で書き込み、`rake monitor`でシリアルモニターを立ち上げてください


シリアルモニターで実行します
```
./accel.rb
```

右に傾けるとXが負の数、左に傾けると正の数、奥に傾けるとYが負の数、手前に傾けると正の数、真上に加速すると、Zが増加、真下に加速するとZが減少します

確かめたら、ボタンを押して終了してください

## 4. 閾値

状態を判定するには閾値を設けます。たとえば閾値が0.3以上なら傾いていると判定します

シリアルモニターを `Ctr+]` で終了して
以下のファイルを作って、保存して `storage/home/` 配下に、`tilt.rb` という名前で置いてください


```ruby
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

`ESPBAUD=115200 rake flash`で書き込み、`rake monitor`でシリアルモニターを立ち上げてください



シリアルモニターで実行します
```
./tilt.rb
```

今回は0.3としていますが、それは使うセンサーや目的次第です。少しでも傾けたら方向を判定したいなら閾値0.1とか、大きく傾けた時にはじめて判定したければ0.5など自分の実現したいことに合わせて設定します
