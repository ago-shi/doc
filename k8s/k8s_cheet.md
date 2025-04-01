## vault secret operatorの設定
#### vault kubernete認証メソッド有効化
```
$ vault auth enable -path=<path> kubernetes
```

#### vaultへ対象k8sクラスタ登録
```
$ vault write auth/<path>/config \
kubernetes_host=https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT

$ vault read auth/<path>/config
```

#### k8sへservice account作成
```
$ kubectl create sa <service account name> -n <name space>
```

#### vaultへpolicyを設定(k8sからのシークレットのアクセス権を設定)
```
$ vault policy write <policy名> - <<EOF
path "<secret path>" {
    capabilities = ["read"]
}
```

#### vaultへroleを設定
roleへpolicy, namespace, service accountを紐づける。
```
$ vault write auth/<path>/role/<role名> \
bound_service_account_names=<service account> \
bound_service_account_namespaces=<name space> \
policies=<policy名> \
ttl=24h
## ttlはk8s認証後に発行されるtokenの有効期間
```
#### role一覧
```
$ vault list auth/<path>/role
Keys
----
app
```

#### roleの内容確認
```
$ vault read auth/<path>/role/<role名>
Key                                         Value
---                                         -----
alias_name_source                           serviceaccount_uid
bound_service_account_names                 [infra]
bound_service_account_namespace_selector    n/a
bound_service_account_namespaces            [u6s-infra]
policies                                    [u6s-master-policy]
token_bound_cidrs                           []
token_explicit_max_ttl                      0s
token_max_ttl                               0s
token_no_default_policy                     false
token_num_uses                              0
token_period                                0s
token_policies                              [u6s-master-policy]
token_ttl                                   24h
token_type                                  default
ttl                                         24h
```

#### vault connectionリソース作成
```manifest: yaml
---
apiVersion: secrets.hashicorp.com/v1beta1
kind: VaultConnection
metadata:
  name: <vault-connectionリソース名>
  namespace: <namespace名>
spec:
  address: "<vaultのエンドポイント(例https://vault~~~)>"
  caCertSecretRef: <vault api>
```