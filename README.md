# Front Door WAF Triage Workbook

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fwinticloud%2Fazure-frontdoor-waf-triage%2Fmain%2FFrontDoorWAFTriageWorkbook_ARM.json)

This workbook visualizes Azure Front Door WAF rule violations and helps triage them to facilitate tuning the WAF against valid traffic.

It is designed to parse WAF logs from **Azure Front Door Premium** with a WAF Policy, sending diagnostics to a **Log Analytics workspace** via `AzureDiagnostics`.

## Prerequisites

- Azure Front Door **Premium** with a WAF Policy attached (Premium is required for managed rulesets such as `Microsoft_DefaultRuleSet` and `Microsoft_BotManagerRuleSet`)
- Diagnostic settings on the Front Door profile sending the following log categories to a Log Analytics workspace:
  - `FrontDoorWebApplicationFirewallLog`
  - `FrontDoorAccessLog`

## What it does

The workbook provides a multi-step triage flow:

1. **Select WAF scope** — pick your Front Door profile and WAF policy
2. **Triage by Rule** — drill down from triggered rules → affected hosts → impacted URLs → individual requests, with links to open the raw logs in Log Analytics
3. **Triage by URL** — start from a hostname, browse flagged paths, and trace back to the rules that fired

## Deploy

Click the **Deploy to Azure** button above, then provide:

| Parameter | Description |
|---|---|
| `workbookDisplayName` | Display name shown in Azure Portal (default: `Front Door WAF Triage`) |
| `workbookSourceId` | Full resource ID of the Log Analytics workspace to associate the workbook with |
| `workbookId` | A new GUID for the workbook resource |

### Example `workbookSourceId`

```
/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.OperationalInsights/workspaces/<workspace-name>
```

## Files

| File | Description |
|---|---|
| `FrontDoorWAFTriageWorkbook_ARM.json` | ARM template containing the full workbook definition |
| `parametersFile.json` | Example parameters file for CLI deployment |

## CLI deployment

```bash
GUID=$(uuidgen | tr '[:upper:]' '[:lower:]')
az deployment group create \
  --resource-group <your-resource-group> \
  --template-file FrontDoorWAFTriageWorkbook_ARM.json \
  --parameters parametersFile.json workbookId="$GUID"
```

## Schema notes

This workbook targets the `AzureDiagnostics` table. Key column names for Front Door (different from Application Gateway):

| Column | Description |
|---|---|
| `policy_s` | WAF policy name |
| `ruleName_s` | Full rule name, e.g. `Microsoft_DefaultRuleSet-2.1-SQLI-942430` |
| `trackingReference_s` | Unique request reference (join key between WAF and access logs) |
| `host_s` | Request hostname (WAF log) |
| `hostName_s` | Request hostname (access log) |
| `action_s` | WAF action: `Log`, `Block`, `AnomalyScoring` |

## Adapted from

Original [Application Gateway WAF Triage Workbook](https://github.com/Azure/Azure-Network-Security/tree/master/Azure%20WAF/Workbook%20-%20AppGw%20WAF%20Triage%20Workbook) by Christof Claessens and Camila Martins (Microsoft).
