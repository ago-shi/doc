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
#### pkiエンジンの最大TTLを設定する。
roleや証明書のTTLはここで設定したTTL以上にすることは出来ない。
```
$ vault secrets tune -max-lease-ttl=<TTL> <pkiパス>
```
#### 認証局のデフォルト発行者を確認する
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
> $ vault read pki_int/issuer/default -format json
#### 発行者に名前をつける
```
###  発行者名を後付けする方法
## 発行者一覧確認(発行者識別子的なもので一覧化される)
$ vault list <path>/issuers
Keys
----
c1231ab9-4f01-0687-758c-76acfca139f4
f67ca50d-0ebc-9e78-6487-ab7842a3429b

## 発行者識別子を指定して発行者名を書き込む
$ vault write <path>/issuer/f67ca50d-0ebc-9e78-6487-ab7842a3429b issuer_name=<任意の発行者名>
```
#### サーバ証明書を発行する
```
### 鍵ペアをvaultで作成し、証明書を作成する方法
## formatの指定はpath指定前が良い。後に書くとjsonファイルにwarningメッセージが記載される。(影響はない)
## ファイルにリダイレクトしないと標準出力に表示される。
## ttlを指定しないとroleに定義されたdefault ttlが設定される。
$ vault write -format=json <path>/issue/<role名> common_name="<FQDN>" ttl="<TTL(24h)>" > <FQDN>.json
```
#### 中間証明書を削除する
```
## 発行者一覧を確認
$ vault list <path>/issuers
Keys
----
3f04ec8d-fa7a-1c68-af77-ecac46126529
52597ba9-5afa-aeb3-3a3e-be0741579e40

## 削除(一覧確認とパスが異なるので注意。issuerとissuersで違う。)
$ vault delete <path>/issuer/<issuer_id>
```
#### 証明書の最大TTL/デフォルトTTLを設定する(変更する)
roleに対する操作になる
```
## roleの確認
$ vault list <path>/roles
Keys
----
uws-dot-lan

## roleの設定確認
$ vault read <path>/roles/<role名>

## roleの設定変更
$ vault write <path>/roles/<role名> max_ttl="<TTL>" ttl="<TTL>"
## (注意) 設定変更は全パラメータ上書きになるため、default値から変更しているパラメータは全て指定する必要あり。
```
#### 有効期限が切れた証明書を認証局から削除する
```
$ vault write pki_int/tidy tidy_cert_store=true tidy_revoked_certs=true safty_buffer="<buffer time>"
```
#### 証明書の失効
```
## 認証局のCA証明書の場合、発行者から除外する必要がある
$ vault delete <path>/issuer/<発行者名>
or
$ vault delete <path>/issuer/<発行者id>

## 証明書を失効させる
$ vault write <path>/revoke serial_number=<シリアル番号>
or
$ vault write <path>/revoke certificate=<証明書名>
```
## (etc)openssl チート
#### 証明書の発行者と主体者を確認する
```
$ openssl x509 -noout -subject -subject_hash -issuer -issuer_hash -in <証明書>
serial=6BC92AE76A60149A122C72F53D5EE7B638F31183
cfb07e43    #主体者
1e3552d1    #発行者
```

## 参考
https://developer.hashicorp.com/vault/api-docs/secret/pki