## Ingressを使う前提条件

Ingressコントローラが実装されている必要がある。
そのうえで、Ingressリソースをデプロイする。

## Ingressコントローラの実行

EKSやGCEなど、k8sサービスではデフォルトで実行済み。
自前k8sではコントロールをクラスタに追加し、実行する必要がある。

クラスタでIngressリソースが使えるか確認する。
ingressclassesとingressesが存在すれば使える。
```
$ kubectl api-resources | grep -e ^NAME -e networking

NAME                                SHORTNAMES                          APIVERSION                        NAMESPACED   KIND
ingressclasses                                                          networking.k8s.io/v1              false        IngressClass
ingresses                           ing                                 networking.k8s.io/v1              true         Ingress
networkpolicies                     netpol                              networking.k8s.io/v1              true         NetworkPolicy
```

クラスタでingressコントロール実行されているか確認する。
実行されていない場合は"No resources found"が表示される。
```
$ kubectl get ingressclasses   ## namespaceの指定は不要。api-resourcesでNAMESPACED=falseのため。

No resources found
```


コントローラの