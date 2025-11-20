## k8sクラスタの構成

次の構成要素から成り立っている。
- controle plane
	- kubeapi-server
		- クラスタに対するリクエストを受け付ける。
		- etcdやkubeletなどの他のコンポーネントへ指示する。
	- etcd
		- クラスタのメタデータを管理する。
	- kube controller manager
		- クラスタを監視し望ましい状態を維持する。
	- kube scheduler
		- woker nodeをランク付けする。
		- 一定の条件に基づき、どのnodeにpodを作成するかを決定する。
	- kubelet
		- systemdサービスとして存在する。
		- kubeapi-serverからの指示を受けてpodsを作成する。
	- kube-proxy
		- クラスタ内部ネットワークのプロキシとして各ノードに配置される。
- worker node
	- kubelet
		- systemdサービスとして存在する。
		- kubeapi-serverからの指示を受けてpodsを作成する。
	- kube-proxy
		- クラスタ内部ネットワークのプロキシとして各ノードに配置される。


(参考)kubeadmで構築したクラスタの構成要素
``` kubeadmで構築したクラスタの構成要素
$ kubectl get pods -n kube-system
NAME                                      READY   STATUS    RESTARTS         AGE
## ここでは割愛
cilium-56twn                              1/1     Running   3 (115d ago)     78d
cilium-cthc6                              1/1     Running   102 (9d ago)     9d
cilium-envoy-54pwl                        1/1     Running   94 (9d ago)      9d
cilium-envoy-97bwn                        1/1     Running   2 (115d ago)     77d
cilium-envoy-cjscg                        1/1     Running   242              152d
cilium-envoy-nqz55                        1/1     Running   247              152d
cilium-envoy-q42hc                        1/1     Running   260 (115d ago)   152d
cilium-envoy-sx4dq                        1/1     Running   3 (115d ago)     78d
cilium-kqtxd                              1/1     Running   2 (115d ago)     77d
cilium-operator-799f498c8-4hsv5           1/1     Running   6 (115d ago)     77d
cilium-pzcl5                              1/1     Running   159              152d
cilium-qtvg5                              1/1     Running   138 (115d ago)   152d
cilium-xdmrk                              1/1     Running   158              152d
## ここでは割愛
coredns-668d6bf9bc-v6m49                  1/1     Running   2 (115d ago)     101d
coredns-668d6bf9bc-5f44n                  1/1     Running   0                101d
## etcd control planeのノード単位に存在
etcd-u6s-master01                         1/1     Running   2 (115d ago)     77d
etcd-u6s-master02                         1/1     Running   4 (115d ago)     78d
etcd-u6s-master03                         1/1     Running   282 (115d ago)   152d
## kube-apiserver control planeのノード単位に存在
kube-apiserver-u6s-master01               1/1     Running   23 (9d ago)      77d
kube-apiserver-u6s-master02               1/1     Running   7 (115d ago)     78d
kube-apiserver-u6s-master03               1/1     Running   284 (115d ago)   152d
## kube-controller-manager control planeのノード単位に存在
kube-controller-manager-u6s-master01      1/1     Running   21 (9d ago)      77d
kube-controller-manager-u6s-master02      1/1     Running   4 (115d ago)     77d
kube-controller-manager-u6s-master03      1/1     Running   299 (115d ago)   152d
## kube-proxy 全ノードに存在
kube-proxy-4gzrg                          1/1     Running   228              152d
kube-proxy-bj5d8                          1/1     Running   225 (115d ago)   152d
kube-proxy-dwcjq                          1/1     Running   92 (9d ago)      9d
kube-proxy-nrq9t                          1/1     Running   3 (115d ago)     78d
kube-proxy-zznst                          1/1     Running   226              152d
kube-proxy-zzw7c                          1/1     Running   2 (115d ago)     77d
## kube-scheduler control planeのノード単位に存在
kube-scheduler-u6s-master01               1/1     Running   29 (115d ago)    77d
kube-scheduler-u6s-master02               1/1     Running   5 (5d15h ago)    78d
kube-scheduler-u6s-master03               1/1     Running   307 (115d ago)   152d
## ここでは割愛
kube-vip-cloud-provider-58577b454-c7q7h   1/1     Running   7 (5d15h ago)    77d
kube-vip-ds-7m8dh                         1/1     Running   67 (4d11h ago)   77d
kube-vip-ds-8xkb9                         1/1     Running   71 (31h ago)     78d
kube-vip-ds-9shhq                         1/1     Running   180 (18h ago)    151d
## kubeletはsystemdサービスとして作成するため、podとして存在しない。
```