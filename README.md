# OnePassword Items Helm Chart

このHelmチャートは、Kubernetesクラスター内で1Password Operatorを使用して、各種サービスの認証情報を管理するためのOnePasswordItemリソースを作成します。

## 概要

このチャートは以下のサービスの認証情報を1Passwordから取得し、Kubernetesシークレットとして作成します：

- **Cloudflare Tunnel Ingress Controller**: Cloudflareトンネルの設定情報
- **External DNS**: CloudflareのAPIトークン
- **Loki**: MinIOストレージの認証情報
- **Mimir**: MinIOストレージの認証情報
- **MinIO Tenant**: ストレージ設定とユーザー認証情報

## 前提条件

- Kubernetesクラスター（バージョン1.20以上）
- [1Password Operator](https://github.com/1Password/onepassword-operator)がクラスターにインストールされている
- 1Password Connect Serverが設定されている
- 必要な認証情報が1Passwordのvault「Kubernetes」に保存されている

## インストール

### Helmリポジトリからのインストール

```bash
helm install onepassword-items ./onepassworditems
```

### カスタム値でのインストール

```bash
helm install onepassword-items ./onepassworditems -f values.yaml
```

## 1Passwordのアイテム構成

以下のアイテムが1Passwordの「Kubernetes」vaultに存在する必要があります：

| アイテム名 | 説明 | 使用するサービス |
|-----------|------|-----------------|
| `cf-ingress-controller-secrets` | Cloudflareトンネルの設定情報 | Cloudflare Tunnel Ingress Controller |
| `cloudflare-api-token` | CloudflareのAPIトークン | External DNS |
| `minio-secrets` | MinIOの認証情報 | Loki, Mimir |
| `minio_storage-configuration` | MinIOストレージ設定 | MinIO Tenant |
| `minio_storage-user` | MinIOユーザー認証情報 | MinIO Tenant |

## 作成されるリソース

このチャートは以下のOnePasswordItemリソースを作成します：

### Cloudflare Tunnel Ingress Controller
- **Namespace**: `cloudflare-tunnel-ingress-controller`
- **Secret**: `cf-ingress-controller-secrets`

### External DNS
- **Namespace**: `external-dns`
- **Secret**: `cloudflare-api-token`

### Loki
- **Namespace**: `loki`
- **Secret**: `minio-secrets`

### Mimir
- **Namespace**: `mimir`
- **Secret**: `minio-secrets`

### MinIO Tenant
- **Namespace**: `minio-tenant`
- **Secrets**: `storage-configuration`, `storage-user`

## アンインストール

```bash
helm uninstall onepassword-items
```

## トラブルシューティング

### OnePasswordItemが作成されない場合

1. 1Password Operatorが正しくインストールされているか確認
2. 1Password Connect Serverが稼働しているか確認
3. 指定されたvaultとアイテムが1Passwordに存在するか確認

### シークレットが作成されない場合

```bash
# OnePasswordItemの状態を確認
kubectl get onepassworditem -A

# 特定のOnePasswordItemの詳細を確認
kubectl describe onepassworditem <item-name> -n <namespace>

# 1Password Operatorのログを確認
kubectl logs -n 1password-operator deployment/onepassword-operator
```

## セキュリティについて

- このチャートは機密情報を直接含まず、1Passwordからの参照のみを作成します
- 実際の認証情報は1Passwordで安全に管理されます
- Kubernetesシークレットは各サービスの名前空間に分離されます
