## 認証メソッド

#### 認証メソッドの一覧化
```
$ vault auth list
Path         Type          Accessor                    Description                Version
----         ----          --------                    -----------                -------
token/       token         auth_token_bc6e4a6f         token based credentials    n/a
```

#### 認証メソッドの有効化
```
$ vault auth enable <type>
```
> [!NOTE]  
> \<type\>は以下がある。(他にもある。)
> - token 
> - userpass
> - kubernetes

#### 認証メソッドに名前(path)をつけて有効化
```
$ vault auth enable -path=<path> <type>
```
#### 認証メソッドの無効化(削除)
```
$ vault auth disable <path>
```

## シークレットエンジン全般
#### シークレットエンジンの有効化
```
$ vault secrets enable <path> <engine>
```
> [!NOTE]
> \<engine\>は以下がある。(他にもある。)
> - kv-v2
> - pki
> - ssh

#### シークレットエンジンの無効化(削除)
```
$ vault secrets disable <path>
```

## kvsエンジン
#### kvsに登録されたデータ一覧
```
$ vault kv list <path>
```
#### kvsへのkey/value登録
```
$ vault kv put <path>/<data name> <KEY1>="<Value1>" <KEY2>="<Value2>"
``` 
#### kvsからkey/value情報を取得
```
$ vault kv get <path>/<data name>
```

## pkiエンジン
#### 認証局の発行者を確認する
```
$ vault read <path>/issuer/default
## ex) $ vault read pki_int/issuer/default
## 参照できる情報
ca_chain
certificate
crl_distribution_points           []
issuer_id                         f67ca50d-0ebc-9e78-6487-ab7842a3429b
issuer_name                       n/a
issuing_certificates              []
key_id                            d8f87b5b-fb76-7873-e2db-db07983c614b
leaf_not_after_behavior           err
manual_chain                      <nil>
ocsp_servers                      []
revocation_signature_algorithm    n/a
revoked                           false
usage                             crl-signing,issuing-certificates,ocsp-signing,read-only
```

> [!NOTE]
> jsonで出力する方法  
> $ vault read pki_int/issuer/default --format json

