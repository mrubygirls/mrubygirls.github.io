---
layout: default
permalink: /guides/esp32/wsl_setup
title: Windows(WSL)向けセットアップ
---
# Windows(WSL)向けセットアップ
このガイドでは、Windows環境のWSL (Windows Subsystem for Linux) を使用して、ATOM MatrixでPicoRubyを動作させるためのR2P2-ESP32環境を構築する手順を説明します。

## 前提
WSLを利用します。過去利用したことがあって以下のものがインストール済みの場合は次のステップに進んでください。

- WSLの導入が済んでいる
    - まだの場合は[Rails Girls Guides Setup for Windows 1.WSLの導入]( https://railsgirls.jp/install/windows-wsl) の手順を行ってください
- Rubyのインストールが済んでいる
    - まだの場合は[Rails Girls Guides Setup for Windows 2. Rubyのインストール]( https://railsgirls.jp/install/windows-wsl) の手順を行ってください

## ESP-IDF のインストール
ESP-IDFはESP32の開発環境です。

## 1. WSL環境の準備
### 1.1 WSLの起動
ターミナルを開きます（スタートメニューで「Powershell」と検索）
```powershell
wsl
```
プロンプトが以下のように変わります
* Windows: `PS C:\Users\ユーザー名>`
* WSL: `ユーザー名@PC名:~$`

### 1.2 必要なパッケージのインストール
WSL内で以下のコマンドを実行：
```bash
sudo apt-get update
sudo apt-get install git wget flex bison gperf python3 python3-pip python3-venv cmake ninja-build ccache libffi-dev libssl-dev dfu-util libusb-1.0-0
```

1.3 ESP-IDFのインストール
ESP32用のツールをインストールします

作業ディレクトリの作成
```bash
mkdir -p ~/esp
cd ~/esp
```
ESP-IDF v5.4をクローン
```
git clone -b v5.4 --recursive https://github.com/espressif/esp-idf.git
```

ESP32用ツールのインストール
```bash
cd ~/esp/esp-idf
./install.sh esp32
```


ESP-IDFの環境変数を読み込みます
```bash
. $HOME/esp/esp-idf/export.sh
```


### 1.4 usbipd-winのインストール

WSLからUSBデバイスを使用するため、*別のターミナル* でPowerShellを開き、以下を実行

usbipd-winがインストールされているか確認
```powershell
usbipd --version
```

インストールされていない場合
```powershell
winget install --interactive --exact dorssel.usbipd-win
```

![install](/images/guides/install_usbipd_win.png)

「Install」を押す

![completed](/images/guides/completed_usbipd_win.png)

「Close」を押す

### 1.5 USBデバイスの認識確認
ここで一旦**ATOM Matrixを接続して**動作確認をします

ATOM MatrixをPCに接続して、PowerShellで以下を実行

```powershell
usbipd list
```

出力例
```
BUSID  VID:PID    DEVICE                           STATE
1-1    0403:6001  USB Serial Converter             Not shared
```

`USB Serial Converter` が表示されることを確認します。

🛠️ デバイスが表示されない場合
* 別のUSB Type-Cケーブルを試す（データ転送対応か確認）
* 別のUSBポートを試す
* デバイスマネージャーで「ポート (COM と LPT)」を確認

確認できたら、一旦USB接続を外し、PowerShellは開いたままにしておきます（後で使用します）。


## 2. R2P2-ESP32のセットアップ

### 2.1 プロジェクトのクローン

WSLのターミナルで
```bash
cd ~/esp
git clone --recursive https://github.com/picoruby/R2P2-ESP32.git
cd R2P2-ESP32
```
* :memo: なぜ --recursive が必要？
     * R2P2-ESP32はPicoRubyをサブモジュールとして含んでいます。--recursiveを付けないと、PicoRuby本体がダウンロードされず、ビルドできません。

### 2.2 初期セットアップとビルド

```bash
# ESP32用の初期セットアップ
rake setup_esp32
```

このコマンドで以下が実行されます
- PicoRubyのビルドに必要なRuby依存関係のインストール
- PicoRubyのビルド  
- ESP32ターゲットの設定

### 2.3 プロジェクトのビルド

mruby Girls Matsue 1st のワークショップの場合は以下のファイルを編集して追加してください。
ワークショップで使うGemを追加します(LEDと加速度センサー用gem)

R2P2/components/picoruby-esp32/picoruby/build_config/xtensa-esp.rb

```ruby
  conf.gem core: 'picoruby-pwm'

  conf.gem github: 'ksbmyk/picoruby-ws2812', branch: 'main' # 追加
  conf.gem github: 'bash0C7/picoruby-mpu6886', branch: 'main' # 追加

  conf.picoruby(alloc_libc: false)
```


```bash
rake build
```

ビルドが成功すると、`build`ディレクトリにファームウェアが生成されます。

## 3. デバイス接続と書き込み
USBでAtomMatrixをパソコンと繋いでください
### 3.1 USBデバイスのバインド（Windows側）

**PowerShellを管理者権限で新しく開き**、以下を実行

```powershell
usbipd bind --busid 1-1
```

※ BUSIDは[1.5]で確認した、USB Serial ConverterのBUSID値を使用します(上記例では1-1)
（わからない場合はもう一度 `usbipd list` で確認できます）


### 3.2 WSLへのアタッチ（Windows側）

通常権限のPowerShellで､以下を実行

```powershell
usbipd attach --wsl --busid 1-1
```

※ BUSIDは1.5で確認した値を使用

成功すると以下のようなメッセージが表示されます
```
usbipd: info: Using WSL distribution 'Ubuntu' to attach
```

### 3.3 デバイスの確認（WSL側）

WSLのターミナルで､以下を実行

```bash
ls /dev/tty*
```

`/dev/ttyUSB0`などが表示されることを確認します。

## 4. ファームウェアの書き込みと実行

### 4.1 書き込み
プログラム環境とファイルをATOM Matrixに転送します。

WSLのターミナルで､以下を実行
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
ATOM Matrixからの出力メッセージを表示し、コマンドを送信できる通信画面(=シリアルモニタ)を起動します。以下を実装します。

```bash
rake monitor
```

### 4.3 一括実行

ビルド、書き込み、モニターを一括で実行

```bash
rake
```

## 5. PicoRuby Shellの確認

書き込みが成功すると、シリアルモニターに以下が表示されます：

![WSL R2P2](/images/guides/wsl_r2p2.png)

これで、ATOM MatrixでPicoRubyコードを実行できる環境が整いました！

終了するときは  `Ctr + ]` で終わってください

## トラブルシューティング

### rake flashに失敗する
```
rake flash --trace
** Invoke flash (first_time)
** Execute flash
idf.py flash
rake aborted!
Command failed with status (127): [idf.py flash]
```

ESP-IDF環境変数の再読み込みが必要。別ターミナルを起動した等 export.sh  がなくなっている場合がある。ので `~/esp/R2P2-ESP32` 配下で実行する。

```
. ~/esp/esp-idf/export.sh
```

### attachに失敗する
```
usbipd attach --wsl --busid 1-1
usbipd: info: Using WSL distribution 'Ubuntu' to attach; the device will be available in all WSL 2 distributions.
usbipd: info: Loading vhci_hcd module.
usbipd: info: Detected networking mode 'nat'.
usbipd: info: Using IP address 172.20.80.1 to reach the host.
```

管理者権限で実行できていないので、PowerShellを管理者権限で実行し直してください。

### ポートが見つからない場合

```
No serial ports found. Connect a device, or use '-p PORT' option
```

対処法：
1. USBデバイスがWSLにアタッチされているか確認
2. `/dev/ttyUSB0`が存在するか確認
3. ポートを明示的に指定：
   ```bash
   idf.py -p /dev/ttyUSB0 flash
   ```

### WSLでのパーミッションエラー

```bash
# 実行権限の付与
chmod +x ~/esp/esp-idf/install.sh
chmod +x ~/esp/esp-idf/export.sh
```


## 参考リンク

- [R2P2-ESP32 GitHub](https://github.com/picoruby/R2P2-ESP32)
- [ESP-IDF Documentation](https://docs.espressif.com/projects/esp-idf/en/v5.4/esp32/get-started/index.html)
- [WSL USB接続ガイド (Microsoft Learn)](https://learn.microsoft.com/ja-jp/windows/wsl/connect-usb)
