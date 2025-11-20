## 🛠️ cloud-init構成メモ

### 📌 概要
- **目的**: VM構築時の初期設定をcloud-initで自動化する。cloud-initのセットアップ方法を記録する。
- **対象環境**: VM基盤 proxmox / OS Almalinux10
- **適用タイミング**: VM初回起動時 / 再構成時

---

### 🧩 構成要素一覧

| 機能カテゴリ       | 実現内容                             | 備考・注意点                 |
|-------------------|-------------------------------------|-----------------------------|
| IPアドレス設定     | static IPを設定する                | 他サーバと重複しないように注意  |
| DNS設定           | DNSサーバとsearchドメインを設定する  |                              |
| ホスト名設定       | ホスト名を設定する                  |                              |
| ssh root公開鍵設定 | vaultからroot公開鍵をコピー         |                               |
| ユーザー設定       | ansibleユーザーを作成する           | uid / gidは全サーバ共通にする   |

---

### 📝 cloud-init YAML例（抜粋）

```yaml
#cloud-config
users:
  - name: shigeru
    sudo: ALL=(ALL) NOPASSWD:ALL
    ssh_authorized_keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC...

packages:
  - htop
  - git

write_files:
  - path: /etc/motd
    content: |
      Welcome to your new VM, Shigeru!
    permissions: '0644'

runcmd:
  - [ systemctl, restart, ssh ]
```

---

### 🧪 テスト・検証ポイント

- [ ] cloud-initログ（`/var/log/cloud-init.log`）でエラー確認
- [ ] `cloud-init status --long` で完了状態を確認
- [ ] 期待通りのユーザー・ファイル・サービスが反映されているか

---

### 📚 参考リンク

- [cloud-init公式ドキュメント](https://cloudinit.readthedocs.io/en/latest/)
- [Ubuntu cloud-init examples](https://ubuntu.com/server/docs/cloud-init)

---

このフォーマットをベースに、目的ごとにテンプレート化していくと再利用性も高まるよ。もしTerraformやProxmoxとの連携も含めたいなら、そのセクションも追加できるよ。どうまとめたいか、もう少し教えてくれたらさらに最適化できる！
