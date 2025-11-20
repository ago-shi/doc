## 言語を日本語へ変更
isoインストール時に日本語を選択していてもproxmox自体は英語設定になっている。これを日本語化する。
```
root@proxmox01:/etc/network# echo $LANG
en_US.UTF-8

root@proxmox01:/etc/network# dpkg-reconfigure locales
## GUIライクな画面が出てくる
## en_US.UTF-8に*がついているので、spaceキーを押して消す。
## ja_JP.UTF-8に*をつける。(spaceキーを押して付ける。)

Generating locales (this might take a while)...
  ja_JP.UTF-8... done
Generation complete.

root@proxmox01:/etc/network# source /etc/default/locale
root@proxmox01:/etc/network# echo $?
0
```

## IPv6無効化
## ansible playbook化済み
```
# vi /etc/sysctl.conf
## 以下を追記する。
net.ipv6.conf.all.disable_ipv6 = 1

# 設定の反映
# sysctl -p
```

## 参考
[設定](https://www.yuuronacademy.com/ja/posts/2025/2/16/)