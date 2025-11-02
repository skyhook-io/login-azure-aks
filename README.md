# Skyhook / login-azure-aks

Unified Azure login with optional **AKS kubecontext** and **ACR registry** login.

- ✅ Works with **Service Principal** (client secret **or** OIDC federated credentials)
- ✅ Works with **Managed Identity** (self-hosted runners)
- ☸️ Optional **AKS** kube context setup
- 🐳 Optional **ACR** registry login

> This action is designed to be used directly, or as a building block in `skyhook-io/cloud-login`.

---

## Usage

### 1) Service Principal (OIDC) + AKS + ACR

```yaml
permissions:
  id-token: write
  contents: read

- uses: skyhook-io/login-azure-aks@v1
  with:
    subscription_id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
    resource_group: my-rg
    cluster_name: aks-prod
    # OIDC: provide client_id + tenant_id, omit client_secret
    client_id: ${{ vars.AZURE_CLIENT_ID }}
    tenant_id: ${{ vars.AZURE_TENANT_ID }}
    enable_acr_login: true
    acr_registry: myacrname
```

### 2) Service Principal (client secret) — registry only

```yaml
- uses: skyhook-io/login-azure-aks@v1
  with:
    subscription_id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
    client_id: ${{ secrets.AZURE_CLIENT_ID }}
    client_secret: ${{ secrets.AZURE_CLIENT_SECRET }}
    tenant_id: ${{ secrets.AZURE_TENANT_ID }}
    enable_acr_login: true
    acr_registry: myacrname
```

### 3) Managed Identity (self-hosted) + AKS

```yaml
- uses: skyhook-io/login-azure-aks@v1
  with:
    subscription_id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
    resource_group: my-rg
    cluster_name: aks-staging
```

---

## Inputs

| Input | Description | Required |
|---|---|---|
| `subscription_id` | Azure subscription ID | ✅ for all auth modes |
| `resource_group` | Resource group containing the AKS cluster | ✅ if `cluster_name` is set |
| `cluster_name` | AKS cluster name (to configure kubecontext) | ❌ |
| `client_id` | Service Principal client ID (SP/ OIDC) | ❌ |
| `client_secret` | Service Principal client secret (SP w/ secret) | ❌ |
| `tenant_id` | Azure tenant ID (SP/ OIDC) | ❌ |
| `enable_acr_login` | `true`/`false` to login to ACR | ❌ (default `false`) |
| `acr_registry` | ACR registry name (`myacrname`) | ✅ if `enable_acr_login=true` |

**Auth modes**

- **Service Principal (OIDC):** set `client_id` and `tenant_id`, omit `client_secret`, and ensure `permissions: id-token: write`.
- **Service Principal (secret):** set `client_id`, `client_secret`, `tenant_id`.
- **Managed Identity:** omit SP fields; runner must have MI and access to the subscription.

---

## Outputs

| Output | Description |
|---|---|
| `subscription_id` | Resolved subscription ID |
| `kubectl_context` | Current kubectl context (if `cluster_name` set) |
| `registry_url` | `myacrname.azurecr.io` if ACR login was requested |

---

## Requirements & roles

- **AKS:** the identity must have AKS “cluster user” permissions to fetch kubeconfig.
- **ACR:** `AcrPull` (and `AcrPush` if you push).
- Tools expected on runner: `az`, `kubectl` (present on GitHub-hosted Ubuntu runners).

---
