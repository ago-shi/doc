## terraformでproxmoxへVMをプロビジョニングする方法

### 基本のtfファイル
```
# VM リソース定義
resource "proxmox_vm_qemu" "vm_from_template" {
  name         = "from-terraform"
  vmid         = 110
  target_node  = "proxmox01"
  clone        = "goss-test"
  full_clone   = true

  memory       = 2048
  cpu {
    cores      = 2
    type       = "host"
  }
  scsihw       = "virtio-scsi-single"

  network {
    id        = 0
    model     = "virtio"
    bridge    = "vmbr0"
    firewall  = true
  }
}
```

---

## 🧾 Terraform構成の詳細解説（Proxmox VMクローン）

このTerraformファイルは、Proxmox環境で既存テンプレートVM（`goss-test`）から**フルクローン**を作成するための定義です。構成の各要素とその設計意図、Terraform適用後の起動トラブルへの対策まで、以下に詳しく解説します。

---

### 🖥️ VMの基本情報

```hcl
resource "proxmox_vm_qemu" "vm_from_template" {
  name         = "from-terraform"
  vmid         = 110
  target_node  = "proxmox01"
```

- `resource "proxmox_vm_qemu"`: ProxmoxのQEMUベースVMを定義するTerraformリソース。
- `name`: 作成されるVMの名前。Proxmox上で表示される識別名。
- `vmid`: VMの一意なID。クラスタ内で重複しないように指定。
- `target_node`: このVMを配置するProxmoxノード。クラスタ環境で特定ノードに配置する際に使用。

---

### 🧬 クローン設定

```hcl
  clone        = "goss-test"
  full_clone   = true
```

- `clone`: `"goss-test"`という既存のテンプレートVMからクローンを作成。
- `full_clone`: フルクローンを指定。テンプレートから完全に独立したVMが作成され、元のテンプレートに依存しません。

#### 💡 なぜ `disk` ブロックを省略しているのか？

この構成では `disk` ブロックを明示的に記述していません。その理由は以下の通りです：

- テンプレートVM（`goss-test`）にすでに適切なディスクレイアウト（OSディスク、サイズ、ストレージタイプなど）が設定されている。
- `full_clone = true` によって、テンプレートのディスクが完全にコピーされるため、Terraform側で再定義する必要がない。
- **明示的に `disk` を定義すると、テンプレートの構成と競合する可能性があり、再現性や保守性が低下する。**
- **さらに、Proxmoxの挙動として、テンプレート由来のディスクとは別に新しい未使用ディスクが作成されてしまうことがある。**  
  Terraformが新規ディスクを追加しようとする一方で、テンプレート由来のディスクが既に存在しているため、Proxmox上で「未使用ディスク」として残ってしまう。これはストレージの無駄遣いや構成の混乱を招く原因になります。

---

### 🧠 リソース設定

```hcl
  memory       = 2048
  cpu {
    cores      = 2
    type       = "host"
  }
  scsihw       = "virtio-scsi-single"
```

- `memory`: VMに割り当てるメモリ容量（MB）。この場合は2GB。
- `cpu`:  
  - `cores`: 仮想CPUのコア数（2コア）。  
  - `type`: `"host"`を指定することで、ホストCPUと同じ命令セットを使用。パフォーマンス重視の設定。
- `scsihw`: SCSIコントローラーのタイプ。`virtio-scsi-single`はVirtIOベースで、パフォーマンスと互換性のバランスが良い。

---

### 🌐 ネットワーク設定

```hcl
  network {
    id        = 0
    model     = "virtio"
    bridge    = "vmbr0"
    firewall  = true
  }
}
```

- `id`: 最初のネットワークインターフェース。
- `model`: `"virtio"`を指定することで、高性能な仮想NICを使用。
- `bridge`: `"vmbr0"`はProxmoxの仮想ブリッジ。外部ネットワークとの通信に使われる。
- `firewall`: Proxmoxのファイアウォール機能を有効化。

---

### ⚠️ Terraform適用後にVMが起動しない問題とその対策

#### 問題の概要

Terraformでこの構成を `apply` しても、VMが**正常に起動しない**ことがあります。具体的には：

- クローンされたVMがProxmox上に存在するが、OSがブートしない。
- コンソールに「No bootable device」などのエラーが表示される。
- Proxmox GUI上で確認すると、ディスクは存在しているが、**boot順序や設定が不完全**。

#### 原因

Terraformの `proxmox_vm_qemu` プロバイダーは、テンプレート由来のディスクをクローンする際に、**bootデバイスの設定を正しく引き継がない**ことがあります。特に `boot: order=scsi0` のような設定が欠落していると、VMは起動できません。

#### ✅ 対策：`qm set` コマンドで明示的にboot設定とディスクマッピングを追加

Terraform適用後に、Proxmox CLIで以下のように設定することで、VMが正常に起動するようになります：

```bash
qm set 110 --scsi0 vmimage-zfs:vm-110-disk-0
qm set 110 --boot order=scsi0
```

- `--scsi0`: VMのOSディスクを明示的に `scsi0` にマッピング。Terraformが自動で設定しない場合に必要。
- `--boot order=scsi0`: `scsi0` を最初のブートデバイスとして指定。

この2つの設定により、Proxmoxは `vm-110-disk-0` を正しく認識し、OSが起動可能になります。Terraform単体では不足しがちな**起動構成の補完**として、非常に有効です。

---

## ✅ まとめ

このTerraform構成は、テンプレートの信頼性を活かしつつ、Terraformによる再現性と管理性を最大限に引き出す設計です。特に以下の点が重要です：

| 設計ポイント | 意図 |
|--------------|------|
| `disk` ブロック省略 | テンプレートの構成を忠実に再現し、未使用ディスクの生成を防ぐ |
| `full_clone = true` | テンプレートから完全に独立したVMを作成 |
| `qm set` によるboot設定 | Terraformの限界を補完し、VMの起動を保証 |
| `qm set` によるscsiマッピング | ディスク認識の明示化で起動失敗を防止 |

