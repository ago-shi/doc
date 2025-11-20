## クラスタノードの削除

#### control planeノード削除前の準備
削除対象がcontrol planeの場合、etcd pod内に削除対象ノードのデータが登録されているため、データを削除する。
```
$ kubectl get pods -n kube-system | grep etcd
kube-system   etcd-u6s-master01        1/1     Running   0    19m   ## -> 削除対象
kube-system   etcd-u6s-master02        1/1     Running   4    2d2h
kube-system   etcd-u6s-master03        1/1     Running   4    2d2h

# 削除しないControl Planeで起動しているetcdへアクセスし削除対象ノードの設定を削除する
$ kubectl -n kube-system exec -it etcd-u6s-master02 -- sh
sh-5.2# etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/peer.crt --key=/etc/kubernetes/pki/etcd/peer.key member list 

812dd29a7ca1c57b, started, u6s-master01, https://192.168.1.40:2380, https://192.168.1.40:2379, false  ## -> 削除対象
97528f60b79d4a62, started, u6s-master03, https://192.168.1.38:2380, https://192.168.1.38:2379, false
cb79e24bfe09ba1a, started, u6s-master02, https://192.168.1.39:2380, https://192.168.1.39:2379, false

# 削除したいノードのメンバーを削除
sh-5.2# etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/peer.crt --key=/etc/kubernetes/pki/etcd/peer.key member remove 812dd29a7ca1c57b
Member 812dd29a7ca1c57b removed from cluster a4ba3ddb5d7dadaf
sh-5.2# exit
```
#### クラスタからノードを削除する
対象ノードをドレインする。削除の準備と思っておけばよい。
```
## ノード障害で既に停止しているような場合はやらなくてもよい
$ kubectl drain <削除対象ノードのFQDN> --ignore-daemonsets --delete-emptydir-data --force

## statusが"SchedulingDisabled"になっていることを確認
$ kubectl get nodes <ノードのホスト名>
NAME               STATUS                     ROLES    AGE    VERSION
k8s.node           Ready,SchedulingDisabled   <none>   9m6s   v1.30.1
```

対象ノードを削除する。
```
$ kubectl delete node <ノードのホスト名>
```

## クラスタの追加

#### クラスタノード用サーバ構築
以下のroleまで完了させる。(- role: linuxK8sBuildは実行しない。)  
https://github.com/ago-shi/uws/blob/main/u6s-cluster.yml  
    - role: ubuntuBasicSetup    
    - role: linuxK8sSetup  

#### 証明書類の複製
!!! 新規ノードを追加する場合は不要な手順 !!!
!!! 既存ノードを再構築した場合のみ実行すれば良い !!!
クラスタの既存ノードから証明書、鍵をコピーする。
```
$ cd /etc/kubernetes/pki
$ sudo tar czvf kube-certs.tar.gz sa.pub sa.key ca.crt ca.key front-proxy-ca.crt front-proxy-ca.key etcd/ca.crt etcd/ca.key
$ sudo scp kube-certs.tar.gz ansible@<追加予定ノード>:/tmp
$ sudo rm kube-certs.tar.gz
```

#### 既存ノードでjoinコマンドを出力
!!! クラスタ上の既存ノードで実行すること !!!  
追加するノードで実行するコマンドを既存ノード上で出力する。
tokenが動的に生成されるため、ノード追加の都度実行する必要がある。
```
$ sudo kubeadm token create --print-join-command
kubeadm join u6s-master:6443 --token h8bpe5.XXXXXXXXXXXX --discovery-token-ca-cert-hash sha256:XXXXXXXXXXXXX
```

#### ノード追加
!!! 追加予定のノードで実行すること !!!  

追加するノードで証明書類を展開する。
!!! 証明書類の展開は新規ノードを追加する場合は不要な手順 !!!
!!! 既存ノードを再構築した場合のみ実行すれば良い !!!
```
$ sudo mkdir /etc/kubernetes/pki
$ sudo tar zxvf /tmp/kube-certs.tar.gz -C /etc/kubernetes/pki
```

クラスターへ追加する。既存ノードで出力したjoinコマンドを使う。  

!!! worker nodeを追加する場合のコマンド !!!
```
$ sudo kubeadm join u6s-master:6443 --token h8bpe5.XXXXXXXXXXXX \
--discovery-token-ca-cert-hash sha256:XXXXXXXXXXXXX \
```

!!! control planeを追加する場合のコマンド !!!  
"--control-plane"オプションを付与すること。
```
$ sudo kubeadm join u6s-master:6443 --token h8bpe5.XXXXXXXXXXXX \
--discovery-token-ca-cert-hash sha256:XXXXXXXXXXXXX \
--control-plane
```

## 補足1: kube-systemが不安定になるケースあり
ノード追加後、podの状態を暫く監視しておいた方が良い。  
追加したノードのpodでcrash loop backoffを繰り返すことがあったりする。(おま環なのかもしれない。)  
不安定な状態が続いたら追加したノードをrebootする。
```
kubectl get pods -n kube-system                      u6s-master02: Sun Mar  9 18:32:49 2025

NAME                                   READY   STATUS             RESTARTS          AGE
cilium-envoy-cjscg                     1/1     Running            241 (51d ago)     52d
cilium-envoy-nqz55                     1/1     Running            247 (51d ago)     52d
cilium-envoy-pxk46                     0/1     CrashLoopBackOff   14 (94s ago)      54m
cilium-envoy-q42hc                     1/1     Running            258 (193d ago)    52d
cilium-envoy-xtpks                     1/1     Running            259 (193d ago)    52d
cilium-fs8d9                           1/1     Running            14 (5m26s ago)    54m
cilium-operator-799f498c8-kvm7w        1/1     Running            0                 6h9m
cilium-pzcl5                           1/1     Running            158 (51d ago)     52d
cilium-qtvg5                           1/1     Running            136 (193d ago)    52d
cilium-tr9rj                           1/1     Running            146 (193d ago)    52d
cilium-xdmrk                           1/1     Running            158 (51d ago)     52d
coredns-668d6bf9bc-5f44n               1/1     Running            0                 17h
coredns-668d6bf9bc-v6m49               1/1     Running            0                 17h
etcd-u6s-master01                      1/1     Running            15 (5m46s ago)    54m
etcd-u6s-master02                      1/1     Running            273 (193d ago)    52d
etcd-u6s-master03                      1/1     Running            280 (193d ago)    52d
kube-apiserver-u6s-master01            0/1     CrashLoopBackOff   14 (98s ago)      54m
kube-apiserver-u6s-master02            1/1     Running            275 (18h ago)     52d
kube-apiserver-u6s-master03            1/1     Running            281 (193d ago)    52d
kube-controller-manager-u6s-master01   0/1     CrashLoopBackOff   13 (101s ago)     54m
kube-controller-manager-u6s-master02   1/1     Running            287 (18h ago)     52d
kube-controller-manager-u6s-master03   1/1     Running            294 (6h31m ago)   52d
kube-proxy-4gzrg                       1/1     Running            227 (51d ago)     52d
kube-proxy-8f72s                       1/1     Running            228 (193d ago)    52d
kube-proxy-9xdkx                       0/1     CrashLoopBackOff   13 (80s ago)      54m
kube-proxy-bj5d8                       1/1     Running            223 (193d ago)    52d
kube-proxy-zznst                       1/1     Running            226 (51d ago)     52d
kube-scheduler-u6s-master01            0/1     CrashLoopBackOff   18 (14s ago)      54m
kube-scheduler-u6s-master02            1/1     Running            294 (6h34m ago)   52d
kube-scheduler-u6s-master03            1/1     Running            302 (6h30m ago)   52d
kube-vip-ds-9shhq                      1/1     Running            67 (6h32m ago)    51d
kube-vip-ds-b2nw4                      1/1     Running            13 (5m45s ago)    52m
kube-vip-ds-kvtbl                      1/1     Running            66 (53m ago)      51d
```

## 補足2: error statusでpodが起動しないケースあり
補足1の対応をしたり、ノードを単純リブートした時にstatusがerrorになることがある。  
原因はリブート前のコンテナが残ってしまっていることの様。
```
kubectl describe pod kube-scheduler-u6s-master02 -n kube-system
Warning  FailedCreatePodSandBox  3m52s (x73 over 18m)  kubelet  Failed to create pod sandbox: rpc error: code = Unknown desc = failed to reserve sandbox name "kube-scheduler-u6s-master02_kube-system_33e007e87732f7e8b1e7e4e06bd779b6_1": name "kube-scheduler-u6s-master02_kube-system_33e007e87732f7e8b1e7e4e06bd779b6_1" is reserved for "a767a9335301691a6ea2568f93992b69a3ac3ec20fae34b379bfb4f6807d252e"
```
対応としてはerrorが出ているpodが動作しているノードで、タスク又はコンテナを削除すればよい。  
```
$ sudo ctr -n k8s.io t ls | grep a767a9335301691a6ea2568f93992b69a3ac3ec20fae34b379bfb4f6807d252e
## あったらタスクがゾンビ化しているので削除する。
$ sudo ctr -n k8s.io t kill {task id}

$ sudo ctr -n k8s.io c ls | grep a767a9335301691a6ea2568f93992b69a3ac3ec20fae34b379bfb4f6807d252e
## あったらコンテナがゾンビ化しているので削除する。
$ sudo ctr -n k8s.io c rm {container id}

## タスク又はコンテナを削除したらcontainerdを再起動する。
$ sudo systemctl restart containerd.service
```
(参考)https://kubeedge.io/docs/faq/setup/

## ノード停止

### アンスケジュール(cordon)
```
$ kubectl cordon <<node name>>

## STATUSがSchedulingDisabledになる。
$ kubectl get nodes
NAME           STATUS                     ROLES           AGE    VERSION
u6s-worker01   Ready,SchedulingDisabled   <none>          220d   v1.32.0
u6s-worker02   Ready,SchedulingDisabled   <none>          220d   v1.32.0
u6s-worker03   Ready,SchedulingDisabled   <none>          77d    v1.32.5
```