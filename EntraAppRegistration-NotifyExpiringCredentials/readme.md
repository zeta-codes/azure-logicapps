# Notify Expiring Entra App Credentials with Azure Logic Apps

Proactively alerts app owners when **Microsoft Entra application secrets/certificates** are expired or expiring soon. Runs daily, scans app registrations via **Microsoft Graph**, and emails owners (and optionally a shared/admin mailbox) with a compact report.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fzeta-codes%2Fazure-logicapps%2Frefs%2Fheads%2Fmain%2FEntraAppRegistration-NotifyExpiringCredentials%2Fsrc%2Fazuredeploy.json)

---

## What you get

- **Daily checks** (06:00 “W. Europe Standard Time” by default)
- **Owner notifications** for expired / expiring (within threshold) secrets & certs
- **Optional summary** to a shared mailbox
- **Secure secret retrieval** from Azure Key Vault
- **Scales** across large tenants with Graph paging

---

## How it works (high level)

- **Recurrence** trigger runs daily
- **Key Vault** connector retrieves the Graph client secret securely
- **Microsoft Graph API** lists apps, owners, and credentials
- **Pagination** follows `@odata.nextLink` until all apps are processed
- **Email** goes to owners; optional **summary** goes to the shared mailbox

---

## Prerequisites

- **Entra app registration** (for Graph, client credentials flow) with **Application** permissions in Microsoft Graph (grant admin consent):  
  - `Application.Read.All`  
  - `Directory.Read.All`
- **Azure Key Vault** with a secret that stores the **Graph client secret value**

---

## Quick Deploy

1. Click **Deploy to Azure** above
2. Choose **Subscription** and **Resource Group**
3. Fill in the parameters (see below)
4. Click **Review + create → Create**
5. Complete the **Post-deploy** steps below.

---

### Parameters

| Parameter | Example / Default | Description |
|---|---|---|
| `logicApp_name` | `NotifyEntraAppRegSecretExpiration` | Logic App name |
| `keyVault_connection_name` | `keyvault-conn` | Key Vault connector resource name |
| `office365_connection_name` | `office365-conn` | Office 365 Outlook connector resource name |
| `keyVault_name` | `my-kv-prod` | Azure Key Vault name |
| `tenant_id` | `00000000-0000-0000-0000-000000000000` | Entra tenant ID |
| `client_id` | `00000000-0000-0000-0000-000000000000` | App registration client ID used for Graph |
| `sharedMailbox` | `alerts@contoso.com` (or empty) | Optional admin/monitoring mailbox for summaries |
| `threshold_days` | `30` | Days ahead to treat as “expiring soon” |
| `graph_client_secret_name` | `Graph-Client-Secret` | **Key Vault secret name** holding the Graph client secret **value** |
> Tip: You can also deploy with a parameters file (`azuredeploy.parameters.json`)

---

## Post-deploy steps

1. **Grant** the Logic App access to Key Vault using either: 
    - RBAC (recommended): assign **Key Vault Secrets User** to the Logic App’s System Assigned Managed Identity at the Key Vault scope.
    - Access Policies: grant **Get** on secrets.
2. Authorize the Office 365 Outlook connection  
    - Logic App → Connections → open office365-conn → **Authorize** → sign in with the account that should send mail.

---

## Troubleshooting

- **Graph 401/403** → Verify admin consent, `tenant_id`/`client_id`, and the **Key Vault secret value**.
- **Key Vault 403** → Managed identity is missing Key Vault rights (assign RBAC role or access policy).
- **Office 365 send fails** → Re-authorize the connector; verify mailbox permissions.
