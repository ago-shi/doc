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
$ vault secrets enable <path>
```

#### シークレットエンジンの無効化(削除)
```
$ vault secrets disable <path>
```