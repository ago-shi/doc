## LVMによるディスクサイズの変更
proxmoxをインストールするとデフォルトでdiskパーティション、LVMが作成される。  
LVMはrootデバイス、swapデバイス、dataデバイスに分かれており、これもデフォルトで作成される。  
各デバイスの容量を変更する場合は、インストール後にLVMを変更する必要がある。  
ここではLMV変更方法を記録する。  

### ディスク確認
lsblkでディスク構成を確認する。
```
# lsblk
NAME                     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                       8:0    0   1.8T  0 disk
├── sda1                  8:1    0  1007K  0 part
├── sda2                  8:2    0     1G  0 part /boot/efi
├── sda3                  8:3    0   1.8T  0 part
│   ├── pve-swap        252:0    0     8G  0 lvm  [SWAP]
│   ├── pve-root        252:1    0    96G  0 lvm  /
│   ├── pve-data_tmeta  252:2    0  15.9G  0 lvm
│   │   └── pve-data    252:4    0   1.7T  0 lvm
│   └── pve-data_tdata  252:3    0   1.7T  0 lvm
│       └── pve-data    252:4    0   1.7T  0 lvm
└── sdb                   8:16   0   1.9T  0 disk
```

fdiskで認識しているディスク一覧を確認する。
```
# fdisk -l
Disk /dev/sda: 1.82 TiB, 2000398934016 bytes, 3907029168 sectors
~略~

## proxmoxのデフォルトのdiskパーティション構成
Device       Start        End    Sectors  Size Type
/dev/sda1       34       2047       2014 1007K BIOS boot
/dev/sda2     2048    2099199    2097152    1G EFI System
/dev/sda3  2099200 3907029134 3904929935  1.8T Linux LVM

Disk /dev/sdb: 1.86 TiB, 2048408248320 bytes, 4000797360 sectors
~略~

## lsblkのpve-swapはOSからは/dev/mapper/pve-swapと認識されている。
Disk /dev/mapper/pve-swap: 8 GiB, 8589934592 bytes, 16777216 sectors

## lsblkのpve-rootはOSからは/dev/mapper/rootと認識されている。
Disk /dev/mapper/pve-root: 96 GiB, 103079215104 bytes, 201326592 sectors
```
 
今回はisoイメージファイルの保管領域を確保するために次の通り容量を変更する。  
pve-root:  96GB -> 1.8TB  # ここがisoイメージファイル保管領域を兼ねている。  
pve-data: 1.7TB -> 0B     # VMイメージは別のSSDに保存するため消してしまう！


### LV(pve-data)の削除
pve-dataを削除する。  
！！これをやるとVMイメージの保管領域がなくなるので、別のディスクを用意しておく必要あり！！
```
# lvremove /dcev/pve/data -y
```

:::message

対象のLVがファイルシステムを使っており、容量縮小したい場合はfsckを実行し、resize2fsでファイルシステムを縮小後に、lvreduceでlvを縮小する。
```
# e2fsck -f /dev/pve/root
# resize2fs /dev/pve/root 25G
# lvreduce --size 25G /dev/pve/root
```
:::

### LV(pve-root)の拡張
pve-rootを空き容量分拡張する。
```
# lvresize -l +100%FREE /dev/pve/root
# resize2fs /dev/pve/root
```

### proxmoxのWeb UIの編集
Web UI上、削除したデバイス=ストレージが残っているため、削除する。  
(削除しなくても影響はないと思われる。)