# eth0-bridge

`eth0-bridge` は、Linux 上で `eth0` をブリッジインターフェース `br0` に自動参加／切り離しするための監視スクリプトです。

systemd timer により定期的に状態を確認し、以下の条件に応じて `eth0` を `br0` に接続または切断します。

---

# 機能

* `eth0` のリンク状態を監視
* `eth0` が未設定または APIPA アドレス（169.254.x.x）の場合はブリッジ参加
* `eth0` と `br0` が同一サブネットの場合はブリッジ参加
* `eth0` と `br0` が異なるサブネットの場合はブリッジから切断
* `eth0` のリンクダウン時は自動的にブリッジから切断
* systemd timer により定期実行
* syslog(journalctl)へログ出力

---

# 構成ファイル

| ファイル                         | 内容          |
| ---------------------------- | ----------- |
| eth0-bridge-cehck3.sh        | メイン監視スクリプト  |
| eth0-bridge-check.service    | systemdサービス |
| eth0-bridge-check.timer      | 定期起動タイマー    |
| install.sh                   | インストールスクリプト |
| *.orign                      | 旧版バックアップ    |
| *.20260523_01, *.20260605_01 | 改修履歴保存版     |

---

# 動作条件

監視対象：

* ブリッジインターフェース

```
br0
```

* Ethernetインターフェース

```
eth0
```

---

# 動作ロジック

## 1. br0 の存在確認

存在しない場合

```
Bridge br0 not found
```

で終了します。

---

## 2. eth0 の存在確認

存在しない場合

```
Interface eth0 not found
```

で終了します。

---

## 3. リンク状態確認

リンクダウンの場合

```
eth0 link down
```

として、

```
ip link set eth0 nomaster
```

を実行してブリッジから切り離します。

---

## 4. IPアドレス判定

### eth0 にIPが無い

```
eth0 has no IPv4
```

→ ブリッジ参加

### eth0 が APIPA

```
169.254.x.x
```

→ ブリッジ参加

### br0 にIPが無い

```
br0 has no IPv4
```

→ ブリッジしない

### eth0 と br0 が同一 /24 サブネット

例

```
eth0 : 192.168.1.10/24
br0  : 192.168.1.1/24
```

→ ブリッジ参加

### 異なるサブネット

例

```
eth0 : 192.168.10.5/24
br0  : 192.168.1.1/24
```

→ ブリッジから切断

---

# インストール

```bash
sudo ./install.sh
```

実行される内容：

```bash
sudo cp eth0-bridge-cehck3.sh /usr/local/bin/eth0-bridge-check.sh
sudo cp eth0-bridge-check.service /etc/systemd/system/
sudo cp eth0-bridge-check.timer /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now eth0-bridge-check.timer
```

---

# 実行周期

タイマー設定：

```ini
[Timer]
OnBootSec=10
OnUnitActiveSec=5
```

* 起動後10秒で開始
* 以後5秒ごとに状態確認

---

# ログ確認

journalctl で確認できます。

```bash
journalctl -t eth0-bridge-cehck
```

または

```bash
journalctl -u eth0-bridge-check.service
```

---

# 手動実行

```bash
sudo /usr/local/bin/eth0-bridge-check.sh
```

---

# 想定用途

* Raspberry Pi
* Wi-Fi HaLow ゲートウェイ
* 災害通信システム
* 自動ネットワーク切替
* DHCP未取得時のフェイルオーバー
* APIPA状態からの自動復旧
* `eth0` と `br0` の動的ブリッジ制御

---

# 動作環境

* Linux
* systemd
* iproute2
* bridge-utils（または bridge コマンド）

---

# ライセンス

利用・改変は自己責任で行ってください。

本ソフトウェアの使用によって発生したいかなる損害についても、作者は責任を負いません。

