# 📘 Kali Linux × Bluetooth（BLE キーボード）完全まとめ【保存版】
以下は、Kali・Debian 系で
Vial/BMP キーボード、BLE HID（HID over GATT）機器を安定接続するための必須手順です。

## 1. 初期設定（最初の一回だけ）
#### (1) UserspaceHID を有効にする（BLE HID 用）
```
sudo nano /etc/bluetooth/input.conf

## 以下を追記、または修正
[General]
UserspaceHID=true
```
```
sudo systemctl restart bluetooth
```

#### (2) bluetoothctl を適切なモードで使用する
```
bluetoothctl
power on
agent KeyboardOnly
default-agent
pairable on
discoverable on
scan on
```

キーボードが見つかったら：
```
pair <MACアドレス>
trust <MACアドレス>
connect <MACアドレス>
```

例：
```
pair C5:80:DE:B3:CE:92
trust C5:80:DE:B3:CE:92
connect C5:80:DE:B3:CE:92
```

#### (3) BlueZ の reconnect 設定（自動接続率向上）
```
sudo nano /etc/bluetooth/main.conf

## 以下を追記（または存在を確認）：
[Policy]
ReconnectAttempts=7
ReconnectIntervals=1,2,4,8,16,32,64
```

#### (4) （推奨）experimental を有効にして BLE を安定化
```
sudo nano /etc/systemd/system/bluetooth.target.wants/bluetooth.service

## ExecStart= の行を以下に変更：
ExecStart=/usr/lib/bluetooth/bluetoothd --experimental
```

反映：
```
sudo systemctl daemon-reload
sudo systemctl restart bluetooth
```

## 2. キーボード側で必要な対策（Vial/BMP 系）
#### (1) BLE プロファイル切り替え
（例）Fn+1 / Fn+2 / Fn+3
→ 新しいプロファイルは必ず空いているためペアリング成功率が上がる。

#### (2) ペアリング情報クリア

過去のホスト情報が残っていると Linux の pairing を拒否する。

（よくある方法）  
Fn + Del（長押し）  
Fn + B  
“BT Clear” キー

RESET / BOOT 長押し  
※キーボードのビルドに依存

## 3. 再起動後の自動接続（Auto Reconnect）の条件
必須条件（これができていれば通常は毎回自動接続する）  
pair 済み  
trust 済み  

ReconnectAttempts が設定されている

キーボードが 広告 (advertising) を出している
　※多くの BLE キーボードは「キーを押すと広告が始まる」

## 4. 自動接続しない場合の対処
#### ◆ 対策1：キーボードを一度操作する

BLE は省電力のため広告を止める場合がある  
→ キーを1回押せば広告開始 → Linuxが自動接続

#### ◆ 対策2：プロファイル切り替え

Fn+1 / Fn+2 などで広告が再開される

#### ◆ 対策3：BlueZ experimental の有効化

BLE HID の安定性が激増する

#### ◆ 対策4：bluez-tools を使う（強制自動接続）
```
sudo apt install bluez-tools
bt-device --set <MAC> Trusted 1
```

## 5. よく出るエラーと原因
❌ org.bluez.Error.AuthenticationRejected

デバイス側が認証を拒否

ほぼ 100% → キーボード側に古いペアリング情報がある

BLE プロファイル切り替え or ペアリングクリアで解決

❌ 接続できなくなる／毎回やり直しになる

BlueZ の HID over GATT が experimental を必要とする  
→ --experimental で改善

❌ ペアリングできても入力できない

UserspaceHID=true が必要  
→ /etc/bluetooth/input.conf

## 6. 最もシンプルな使用手順（保存用）

/etc/bluetooth/input.conf に UserspaceHID を書く

/etc/bluetooth/main.conf に Reconnect 設定

bluetoothd --experimental を有効化

bluetoothctl で
```
agent KeyboardOnly
pair <MAC>
trust <MAC>
connect <MAC>
```

再起動後は自動接続（またはキーボードを一度押せば接続）

## 7. 再接続の流れ（実際の動き）
1. PC起動
2. bluetoothd起動
3. trust済デバイスへ reconnect 試行
4. キーボードが広告を出していれば即接続
5. 出していなければ、キー操作で広告開始→自動接続
