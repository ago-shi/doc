## 🧰 `qm` コマンド概要

`qm` は Proxmox VE における **QEMU/KVM 仮想マシンの管理CLI** です。VMの作成、設定変更、起動、停止、削除などを行えます。

基本構文：
```bash
qm <サブコマンド> <vmid> [オプション]
```

---

## 🔧 `qm set` — VMの設定変更

VMの構成を変更するためのコマンド。**既存VMに対して設定を追加・更新**します。

### 基本構文
```bash
qm set <vmid> --<オプション>=<値>
```

### よく使うオプション一覧

| オプション | 説明 | 例 |
|-----------|------|-----|
| `--name` | VMの表示名を変更 | `--name my-vm` |
| `--memory` | メモリサイズ（MB） | `--memory 4096` |
| `--cores` | CPUコア数 | `--cores 2` |
| `--cpu` | CPUタイプ（例：host, kvm64） | `--cpu host` |
| `--scsihw` | SCSIコントローラータイプ | `--scsihw virtio-scsi-single` |
| `--scsi0` | ディスクの割り当て | `--scsi0 local-lvm:vm-110-disk-0` |
| `--boot` | ブート順序の指定 | `--boot order=scsi0` |
| `--net0` | ネットワーク設定 | `--net0 virtio,bridge=vmbr0,firewall=1` |
| `--agent` | QEMU Guest Agentの有効化 | `--agent enabled=1` |
| `--onboot` | ホスト起動時にVMを自動起動 | `--onboot 1` |
| `--serial0` | シリアルポート設定 | `--serial0 socket` |

### 使用例
```bash
qm set 110 --memory 2048 --cores 2 --scsi0 vmimage-zfs:vm-110-disk-0 --boot order=scsi0
```

---

## 📋 `qm config` — VMの現在の構成を表示

VMの設定内容を確認するためのコマンド。**pending（未反映）設定も含めて表示**されます。

### 基本構文
```bash
qm config <vmid> [--current] [--snapshot <name>]
```

### オプション一覧

| オプション | 説明 |
|-----------|------|
| `--current` | 現在の反映済み設定のみ表示（pendingは除外） |
| `--snapshot <name>` | 指定したスナップショットの設定を表示 |

### 使用例
```bash
qm config 110
qm config 110 --current
qm config 110 --snapshot pre-upgrade
```

---

## 🧠 実運用での補足

- `qm set` は **VMが停止していなくても実行可能**ですが、一部の変更（ディスク追加など）は停止中の方が安全です。
- `qm config` は **Terraform適用後の確認や差分チェック**に非常に便利です。
- `qm set` を使って `scsi0` や `boot` を明示的に設定することで、Terraform適用後の起動失敗を防げます。

---

