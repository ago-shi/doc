## proxmoxのリポジトリ有効化
## ansible playbook化済み
proxmoxはデフォルトでは有償のリポジトリが有効になっており、アップデートに失敗する。
無償版のリポジトリに切り替えることでアップデート可能にする。

### GUIでの設定方法
GUIの場合、ホスト名->アップデート->リポジトリと辿って設定する。
以下を無効化
- ceph-squid enterprise
- pve pve-enterprise

以下を追加することで有効化
- ceph-squid no-subscription
- pve pve-no-subscription

設定変更後は再起動が必要っぽい。

### CUIでの設定方法
CUIの場合は、/etc/apt/sources.list.d/配下のファイルを更新して設定する。

無効化。Enabled: falseを追記することで無効になる模様。
```
# cat /etc/apt/sources.list.d/pve-enterprise.sources
Types: deb
URIs: https://enterprise.proxmox.com/debian/pve
Suites: trixie
Components: pve-enterprise
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
Enabled: false

# cat /etc/apt/sources.list.d/ceph.sources
Types: deb
URIs: https://enterprise.proxmox.com/debian/ceph-squid
Suites: trixie
Components: enterprise
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
Enabled: false

Types: deb
URIs: http://download.proxmox.com/debian/ceph-squid
Suites: trixie
Components: no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
```

有効化。新しいファイルに定義を記述するか、既存ファイルに追記すれば良い模様。
```
# cat /etc/apt/sources.list.d/proxmox.sources
Types: deb
URIs: http://download.proxmox.com/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg

# cat /etc/apt/sources.list.d/ceph.sources
Types: deb
URIs: https://enterprise.proxmox.com/debian/ceph-squid
Suites: trixie
Components: enterprise
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
Enabled: false

Types: deb
URIs: http://download.proxmox.com/debian/ceph-squid
Suites: trixie
Components: no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
```

リポジトリキャッシュを更新する。
```
# apt update
```

### アップグレード
GUIでも出来る。CUIの場合はaptコマンドでOK。
```
# apt update
# apt upgrade -y
```