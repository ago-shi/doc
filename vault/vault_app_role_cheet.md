# AppRole認証チーと

## 1. AppRole認証有効化
```bash
vault auth enable -path=dev/approle approle
```

## 2. AppRole用roleを作成
```bash
vault write auth/dev/approle/role/dev-server \
  token_policies="dev-server" \              ## 特定サーバ用のpolicyを指定する
  secret_id_bound_cidrs="ip-address/32" \    ## approle認証するサーバのipアドレスを指定する
  token_ttl=6h \
  token_max_ttl=12h \
  secret_id_ttl=6h \
  secret_id_num_uses=2
```

## 3. RoleID / SecretIDを取得(vaultサーバやansibleを想定)

RoleID取得
```bash
## vaultコマンド版
vault read -field=role_id auth/dev/approle/role/dev-server/role-id

## curlコマンド版
curl -sS \
  -H "X-Vault-Token: $VAULT_TOKEN" \
  $VAULT_ADDR/v1/auth/dev/approle/role/dev-server/role-id
  | jq -r '.data.role_id'
```

SecretID取得
```bash
## vaultコマンド版
vault write -field=secret_id auth/dev/approle/role/dev-server/secret-id \
  cidr_list="ip-address/32"       ## approle認証するサーバのipアドレスを指定する

## curlコマンド版
curl -s \
  -H "X-Vault-Token: $VAULT_TOKEN" \
  -H "Content-Type: application/json" \
  -X POST \
  --data '{"cidr_list":"ip-address/32"}' \  ## approle認証するサーバのipアドレスを指定する
  $VAULT_ADDR/v1/auth/dev/approle/role/dev-server/secret-id \
  | jq -r '.data.secret_id'
```

## 4. AppRoleログイン(Vaultクライアントサーバを想定)
```bash
VAULT_TOKEN=$(curl -s \
  --request POST \
  --data @- \
  $VAULT_ADDR/v1/dev/auth/approle/login <<EOF
> {
>    "role_id": "$ROLE_ID",
>    "secret_id": "$SECRET_ID"
> }
> EOF
  | jq -r '.auth.client_token')
```