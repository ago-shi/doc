# vault sshエンジンチート

## 1. vaultサーバ側の初期設定(クライアントCA用)

### 1-1. SSH Secret engine(クライアントCA用)
```bash
vault secrets enable -path=ssh/dev/clien-ca ssh
```

### 1-2. CAキー生成
```bash
vault write ssh/dev/client-ca/config/ca generate_signing_key=true
```

sshクライアント用ssh ca公開鍵の取得(vaultクライアントで実行する想定)
```bash
# vaultコマンド
vault read -field=public_key ssh/dev/client-ca/config/ca > ssh_client-ca.pub

# curlコマンド(REST API)
curl \
--header "X-Vault-Token: $VAULT_TOKEN" \
--request GET \
$VAULT_ADDR/v1/ssh/dev/client-ca/config/ca \
| jq -r '.data.public_key' \
> ssh_client-ca.pub
```

## 2. SSHクライアント証明書利用サーバ追加時のvault設定

### 2-1. 署名ポリシーの作成
```bash
vault write ssh/dev/client-ca/roles/dev-server \
key_type=ca \
allow_user_certificates=true
allowed_users="user01,user02,user03" \
default_user="user01" \
ttl=6h \e
max_ttl=12h
```

### 2-2. SSHクライアント用vault policyの設定
ポリシー用のhclファイルが既にある場合は追記書きする。
```bash
cat << EOF >> dev-server.hcl
> path "ssh/dev/client-ca/sign/dev-server" {
>   capabilities = ["update"]
> }
>
> path "ssh/dev/client-ca/config/ca" {
>   capabilities = ["read"]
> }
> EOF

vault policy write dev-server dev-server.hcl
```

## 3. SSHサーバ側の設定

### 3-1. SSHクライアントルート公開鍵の取得
```bash
# curlコマンド(REST API)
curl \
--header "X-Vault-Token: $VAULT_TOKEN" \
--request GET \
$VAULT_ADDR/v1/ssh/dev/client-ca/config/ca \
| jq -r '.data.public_key' \
> ssh_client-ca.pub
```

### 3-2. ルート公開鍵の配置
```bash
sudo mkdir -p /etc/ssh/ca
sudo cp ssh_client-ca.pub /etc/ssh/ca/ssh_client-ca.pub
sudo chmod 644 /etc/ssh/ca/ssh_client-ca.pub
```

### 3-3. sshd_configにCA公開鍵を設定
```bash
sudo vi /etc/ssh/sshd_config

### 以下、sshd_configファイルへ記載
TrustedUserCAKeys /etc/ssh/ca/ssh_client-ca.pub
PasswordAuthentication no
PubkeyAuthentication yes

### sshd_configファイルを更新したらsshdを再起動
sudo systemctl restart sshd
```

## 4. SSHクライアント側の設定

### 4-1. 公開鍵・秘密鍵の生成
```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519
```

### 4-2. 公開鍵を署名する(SSHクライアント証明書発行)
```bash
PUBKEY=$(cat ~/.ssh/id_ed25519.pub)

curl \
  --header "X-Vault-Token: $VAULT_TOKEN" \
  --header "Content-Type: application/json" \
  --request POST \
  --data "{\"public_key\": \"${PUBKEY}\"}" \
  $VAULT_ADDR/v1/ssh/dev/client-ca/sign/dev-server \
  > signed_key.json

jq -r '.data.signed.key' signed_key.json > ~/.ssh/id_ed25519-cert.pub
```

### 4-3. SSHクライアント設定
```bash
vi ~/.ssh/config

### 以下をconfigファイルへ記載
Host dev-server
  IdentityFile ~/.ssh/id_ed25519
  CertificateFile ~/.ssh/id_ed25519-cert.pub
```


## 5. vaultサーバ側の初期設定(ホストCA用)

### 5-1. SSH Secret engine(ホストCA用)
```bash
vault secrets enable -path=ssh/dev/host-ca ssh
```

### 5-2. CAキー生成
```bash
vault write ssh/dev/host-ca/config/ca generate_signing_key=true
```

sshクライアント用ssh ca公開鍵の取得(vaultクライアントで実行する想定)
```bash
# vaultコマンド
vault read -field=public_key ssh/dev/host-ca/config/ca > ssh_host-ca.pub

# curlコマンド(REST API)
curl \
--header "X-Vault-Token: $VAULT_TOKEN" \
--request GET \
$VAULT_ADDR/v1/ssh/dev/host-ca/config/ca \
| jq -r '.data.public_key' \
> ssh_host-ca.pub
```

## 6. SSHサーバ証明書利用サーバ追加時のvalut設定

### 6-1. 署名ポリシーの作成
```bash
vault write ssh/dev/host-ca/dev-uws-lan \
key_type=ca \
allow_host_certificates=true \
allowed_domains="dev.uws.lan" \
allow_subdomains=true \
ttl=8760h
```

### 6-2. SSHサーバ用vault policyの設定
ポリシー用のhclファイルが既にある場合は追記書きする。
```bash
cat << EOF >> dev-server.hcl
> path "ssh/dev/host-ca/sign/dev-uws-lan" {
>   capabilities = ["update"]
> }
> 
> path "ssh/dev/host-ca/config/ca" {
>   capabilities = ["read"]
> }
> EOF

vault policy write dev-server dev-server.hcl
```

## 7. SSHクライアント側の設定

### 7-1. SSHサーバCA公開鍵の取得
```bash
# curlコマンド(REST API)
curl \
--header "X-Vault-Token: $VAULT_TOKEN" \
--request GET \
$VAULT_ADDR/v1/ssh/dev/host-ca/config/ca \
| jq -r '.data.public_key' \
> ssh_host-ca.pub
```

### 7-2. ルート公開鍵の配置
```bash
echo "@cert-authority *.dev.uws.lan $(cat ssh_host-ca.pub)" \
>> ~/.ssh/known_hosts
```

## 8. SSHサーバ側の設定

### 8-1. ホスト鍵の確認
```bash
ls -l /etc/ssh/
/etc/ssh/ssh_host_ed25519_key
/etc/ssh/ssh_host_ed25519_key.pub
```

### 8-2. 公開鍵を署名する(SSHホスト証明書発行)
```bash
HOST_PUBKEY=$(cat /etc/ssh/ssh_host_ed25519_key.pub)

curl \
  --header "X-Vault-Token: $VAULT_TOKEN" \
  --header "Content-Type: application/json" \
  --request POST \
  --data "{\"public_key\": \"${HOST_PUBKEY}\", \"cert_type\": \"host\"}" \
  $VAULT_ADDR/v1/ssh/dev/host-ca/sign/dev-uws-dev \
  > signed_host.json

jq -r '.data.signed_key' signed_host.json > /etc/ssh/id_host_ed25519_key-cert.pub

sudo chown root:root /etc/ssh/id_host_ed25519_key-cert.pub
sudo chmod 600 /etc/ssh/id_host_ed25519_key-cert.pub
```

### 8-3. sshd_config設定
```bash
vi ~/etc/ssh/sshd_config

### 以下をsshd_configファイルへ記載
HostKey /etc/ssh/ssh_host_ed25519_key
HostCertificate /etc/ssh_host_ed25519_key-cert.pub

### sshd_configファイルを更新したらsshdを再起動
sudo systemctl restart sshd
```