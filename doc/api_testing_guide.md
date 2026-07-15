# API Testing & Verification Guide

This document describes how to test and verify the account linkage, token exchange, token refresh, and device discovery API endpoints for the BroadLink Magic Home API integration using standard development tools (`PowerShell` or `curl`).

---

## 1. Environment Configurations

All testing directives should point to the target cluster gateway URL. Set the routing header based on the target region:

*   **Cluster Endpoint Base URL**: `https://ha-magic-home-mfdddeskcg.cn-hangzhou.fcapp.run`
*   **Routing Header (`CLOUD_SERVERS_DOMAIN`)**:
    *   `cn`: China Cluster
    *   `us`: United States Cluster
    *   `eu`: Europe Cluster

---

## 2. Testing Directives

### A. Exchange Authorization Code for Access Token
Use this request to exchange the single-use authorization code (`code`) obtained from the app linkage flow for an `access_token` and `refresh_token`.

#### PowerShell Command
```powershell
$code = "YOUR_AUTHORIZATION_CODE"
$body = @{ auth_code = $code } | ConvertTo-Json

Invoke-RestMethod -Method Post `
  -Uri "https://ha-magic-home-mfdddeskcg.cn-hangzhou.fcapp.run/oauth/v2/token?grant_type=authorization_code&code=$code&client_id=&redirect_uri=http://homeassistant.local:8123" `
  -Headers @{ "CLOUD_SERVERS_DOMAIN" = "eu" } `
  -ContentType "application/json" `
  -Body $body | ConvertTo-Json -Depth 10
```

#### curl Command
```bash
curl -X POST "https://ha-magic-home-mfdddeskcg.cn-hangzhou.fcapp.run/oauth/v2/token?grant_type=authorization_code&code=YOUR_AUTHORIZATION_CODE&client_id=&redirect_uri=http://homeassistant.local:8123" \
  -H "Content-Type: application/json" \
  -H "CLOUD_SERVERS_DOMAIN: eu" \
  -d '{"auth_code": "YOUR_AUTHORIZATION_CODE"}'
```

---

### B. Refresh Expired Access Token
Use this request to exchange a `refresh_token` for a new `access_token` when the lifetime has expired.

#### PowerShell Command
```powershell
$refreshToken = "YOUR_REFRESH_TOKEN"

Invoke-RestMethod -Method Post `
  -Uri "https://ha-magic-home-mfdddeskcg.cn-hangzhou.fcapp.run/oauth/v2/token?grant_type=refresh_token&refresh_token=$refreshToken" `
  -Headers @{ "CLOUD_SERVERS_DOMAIN" = "eu" } `
  -ContentType "application/json" | ConvertTo-Json -Depth 10
```

#### curl Command
```bash
curl -X POST "https://ha-magic-home-mfdddeskcg.cn-hangzhou.fcapp.run/oauth/v2/token?grant_type=refresh_token&refresh_token=YOUR_REFRESH_TOKEN" \
  -H "Content-Type: application/json" \
  -H "CLOUD_SERVERS_DOMAIN: eu"
```

---

### C. Run Device Discovery
Test device discovery by passing the active `access_token` in the JSON scope payload.

#### PowerShell Command
```powershell
$token = "YOUR_ACCESS_TOKEN"
$body = @{
    directive = @{
        header = @{
            namespace = "DNA.Discovery"
            name = "Discover"
            interfaceVersion = "2"
            messageId = [Guid]::NewGuid().ToString()
        }
        payload = @{
            scope = @{
                type = "BearerToken"
                token = $token
            }
        }
    }
} | ConvertTo-Json -Depth 5

Invoke-RestMethod -Method Post `
  -Uri "https://ha-magic-home-mfdddeskcg.cn-hangzhou.fcapp.run/dnaproxy/v2/discover?license=" `
  -Headers @{ "CLOUD_SERVERS_DOMAIN" = "eu" } `
  -ContentType "application/json" `
  -Body $body | ConvertTo-Json -Depth 10
```

#### curl Command
```bash
curl -X POST "https://ha-magic-home-mfdddeskcg.cn-hangzhou.fcapp.run/dnaproxy/v2/discover?license=" \
  -H "Content-Type: application/json" \
  -H "CLOUD_SERVERS_DOMAIN: eu" \
  -d '{
    "directive": {
      "header": {
        "namespace": "DNA.Discovery",
        "name": "Discover",
        "interfaceVersion": "2",
        "messageId": "12345678-1234-1234-1234-123456789012"
      },
      "payload": {
        "scope": {
          "type": "BearerToken",
          "token": "YOUR_ACCESS_TOKEN"
        }
      }
    }
  }'
```
