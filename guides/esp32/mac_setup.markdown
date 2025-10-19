---
layout: default
permalink: /guides/esp32/mac_setup
title: Mac向けセットアップ
---
# ATOM Matrix用 R2P2-ESP32 環境構築ガイド (macOS編)

このガイドでは、macOS環境でATOM MatrixにPicoRubyを動作させるためのR2P2-ESP32環境を構築する手順を説明します。

## 前提条件

- macOS 10.15 (Catalina) 以降
- ATOM Matrixデバイス  
- USB Type-Cケーブル

## 1.  macOS環境の準備

### 1.1 Xcodeコマンドラインツールの確認

まず、Xcodeコマンドラインツールがインストールされているか確認します。ターミナルで以下を実行

```bash
xcode-select --version
```

もし以下のようなエラーが表示される場合
```
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools)
```

Xcodeコマンドラインツールをインストールします
```bash
xcode-select --install
```

### 1.2 Homebrewの確認とインストール

Homebrewがインストールされているか確認
```bash
brew --version
```

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 1.3 開発ツールのインストール

Homebrewを使用してインストール

```bash
brew install cmake ninja dfu-util
```

**推奨**: ビルド速度向上のためccacheもインストール

```bash
brew install ccache
```


### 1.4 Python 3の確認

現在のPythonバージョンを確認

```bash
# デフォルトのPythonバージョン確認
python --version

# Python 3がインストールされているか確認
python3 --version
```

Python 3がインストールされていない場合

```bash
brew install python3
```


### 1.5 ESP-IDFのインストール

```bash
# 作業ディレクトリの作成
mkdir -p ~/esp
cd ~/esp

# ESP-IDF v5.4.2をクローン
git clone -b v5.4.2 --recursive https://github.com/espressif/esp-idf.git
```

**なぜ `--recursive` が必要？**
ESP-IDFは多くのサブモジュール（別のGitリポジトリ）を含んでいます。`--recursive`を付けないと、これらのサブモジュールがダウンロードされず、ビルド時にエラーになります。

ESP32用ツールのインストール

```bash
cd ~/esp/esp-idf
./install.sh esp32
```

### 1.6 環境変数の設定

一時的な設定（現在のターミナルセッションのみ）

```bash
. $HOME/esp/esp-idf/export.sh
```

**注意**: 先頭のドット(.)とパスの間にスペースが必要です！


### 1.7 デバイス接続の確認（オプション）

ATOM Matrixを接続して、シリアルポートを確認：

```bash
ls /dev/cu.*
```

以下のようなデバイスが表示されれば認識されています

```
/dev/cu.usbserial-XXXX
/dev/cu.wchusbserialXXXX
```

確認後、一旦USBケーブルを外しておきます（ビルド後に再接続）。


## 2. R2P2-ESP32のセットアップ

### 2.1 プロジェクトのクローン

```bash
cd ~/esp
git clone --recursive https://github.com/picoruby/R2P2-ESP32.git
cd R2P2-ESP32
```

**なぜ `--recursive` が必要？**
R2P2-ESP32はPicoRubyをサブモジュールとして含んでいます。`--recursive`を付けないと、PicoRuby本体がダウンロードされず、ビルドできません。


### 2.2 Rubyとbundlerの確認

macOSには標準でRubyがインストールされています


```bash
# Rubyバージョン確認
ruby --version

# bundlerのインストール（未インストールの場合）
sudo gem install bundler
```


## 2.3 初期セットアップとビルド

```bash
# ESP-IDF環境変数の読み込み（または get_idf コマンド）
. ~/esp/esp-idf/export.sh

# ESP32用の初期セットアップ
rake setup_esp32
```

このコマンドで以下が実行されます
- PicoRubyのビルドに必要なRuby依存関係のインストール
- PicoRubyのビルド
- ESP32ターゲットの設定


mruby Girls Matsue 1st のワークショップの場合は以下のファイルを編集して追加してください。
ワークショップで使うGemを追加します(LEDと加速度センサー用gem)

R2P2/components/picoruby-esp32/picoruby/build_config/xtensa-esp.rb

```ruby
  conf.gem core: 'picoruby-pwm'

  conf.gem github: 'ksbmyk/picoruby-ws2812', branch: 'main' # 追加
  conf.gem github: 'bash0C7/picoruby-mpu6886', branch: 'main' # 追加

  conf.picoruby(alloc_libc: false)
```

プロジェクトのビルド
```bash
rake build
```

ビルドが成功すると、`build`ディレクトリにファームウェアが生成されます。


# 3. デバイス接続と書き込み

### 3.1 ATOM Matrixの接続

USB Type-CケーブルでATOM MatrixをPCに接続します。

### 3.2 デバイスの確認

```bash
ls /dev/cu.*
```

以下のようなデバイスが表示されることを確認します。
- `/dev/cu.usbserial-XXXX`
- `/dev/cu.wchusbserialXXXX`


## 4. ファームウェアの書き込みと実行

### 4.1 書き込み

```bash
rake flash
```

以下のエラーが出た場合は

```
A fatal error occurred: Unable to verify flash chip connection
```

以下を実行してください

```bash
ESPBAUD=115200 rake flash
```

### 4.2 シリアルモニターの起動

```bash
rake monitor

# ポートを指定する場合
ESPPORT=/dev/cu.usbserial-XXXX rake monitor
```

### 4.3 一括実行

ビルド、書き込み、モニターを一括で実行

```bash
rake
```

## 5. PicoRuby Shellの確認

書き込みが成功すると、シリアルモニターに以下が表示されます

```
Starting shell...
PICORUBY
$>
```

これで、ATOM MatrixでPicoRubyコードを実行できる環境が整いました！

![Mac R2P2](/images/guides/mac_r2p2.png)

## トラブルシューティング

### Python関連のエラー

```bash
# Python3のパスを確認
which python3

# パスが正しくない場合
export PATH="/usr/local/opt/python@3/bin:$PATH"
```

### openssl関連のエラー

`rake setup_esp32` 実行中

```
ld: library not found for -lssl
clang: error: linker command failed with exit code 1 (use -v to see invocation)
rake aborted!
```

ld: library not found for -lssl というエラーは、clang が libssl（OpenSSLのライブラリ）を見つけられていないことを意味しています。これは pkg-config が正しく設定されていない、または ライブラリパスが通っていないのが原因です。


```
export LDFLAGS="-L$(brew --prefix openssl@3)/lib"
export CPPFLAGS="-I$(brew --prefix openssl@3)/include"
export PKG_CONFIG_PATH="$(brew --prefix openssl@3)/lib/pkgconfig"
```

再度ビルド

```bash
rake clean
rake setup_esp32
```

## 参考リンク

- [R2P2-ESP32 GitHub](https://github.com/picoruby/R2P2-ESP32)
- [ESP-IDF Documentation v5.4.2](https://docs.espressif.com/projects/esp-idf/en/v5.4.2/)
- [ATOM Matrix Documentation](https://docs.m5stack.com/en/core/atom_matrix)
