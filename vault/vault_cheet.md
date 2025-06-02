## vaultエンジン
#### エンジンの一覧化
```
$ vault secrets list
## 詳細表示(エンジン全体のmax ttl設定など)
$ vault secrets list -detailed
```
#### エンジンの設定変更
エンジン全体のmax ttl設定
```
$ vault secrets tune -max-lease-ttl=<TTL> <パス名>
```

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
$ vault secrets enable -path=<path> <engine>
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

#### kvsからkey/valueを削除する
```
$ vault kv metadata <path>/<data name>
```

## vault API
#### curlステートメントを忘れた時のコマンド
```
$ vault status -output-curl-string
curl -H "X-Vault-Token: $(vault print token)" -H "X-Vault-Request: true" http://localhost:8200/v1/sys/seal-status
```

## 参考
https://developer.hashicorp.com/vault/api-docs/secret/pki