# Refresh OAuth Token Request (刷新token请求)

This document describes the API to refresh an expired access token using a refresh token.

---

## HTTP Request

*   **Method:** `POST`
*   **URL:** `https://(lambdaUrl)/oauth/v2/token`

### Headers

| Header Parameter | Type | Required | Description | Example |
| :--- | :--- | :--- | :--- | :--- |
| `CLOUD_SERVERS_DOMAIN` | `string` | **Required** | Target cluster domain used to route the request to the correct regional server:<br>• `cn`: China cluster<br>• `us`: USA cluster<br>• `eu`: Europe cluster | `cn` |

### Query Parameters

| Parameter | Type | Required/Optional | Description | Example         |
| :--- | :--- | :--- | :--- |:----------------|
| `grant_type` | `string` | **Required** | Fixed value: `refresh_token` | `refresh_token` |
| `refresh_token` | `string` | **Required** | The refresh token returned by the request to obtain the token | `_____`         |

### Request Example

```http
POST https://(lambdaUrl)/oauth/v2/token?grant_type=refresh_token&refresh_token=______
CLOUD_SERVERS_DOMAIN: cn
```

---

## HTTP Response

### Response Status
*   **Code:** `200 OK` (success)
*   **Content-Type:** `application/json`

### Response Body Fields

| Parameter | Type | Required/Optional | Description |
| :--- | :--- | :--- | :--- |
| `access_token` | `string` | Required | The new access token |
| `expires_in` | `integer` | Required | The lifetime in seconds of the access token |
| `refresh_token` | `string` | Required | The new refresh token |
| `token_type` | `string` | Required | The type of token returned. Typically `Bearer` |

### Response Example

```json
{
    "access_token": "iM-nK1t_Sw6yyqBk3fAGyw",
    "expires_in": 7200,
    "refresh_token": "cwey5p6RTXa_PuasoLAhSw",
    "token_type": "Bearer"
}
```
