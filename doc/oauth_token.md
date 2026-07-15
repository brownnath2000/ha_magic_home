# Get OAuth Token Request (获取token请求)

This document describes the API to request an OAuth 2.0 access token using an authorization code.

---

## HTTP Request

*   **Method:** `POST`
*   **URL:** `https://(lambdaUrl)/oauth/v2/token`

### Headers

| Header Parameter | Type | Required | Description | Example |
| :--- | :--- | :--- | :--- | :--- |
| `CLOUD_SERVERS_DOMAIN` | `string` | **Required** | Target cluster domain used to route the request to the correct regional server:<br>• `cn`: China cluster<br>• `us`: USA cluster<br>• `eu`: Europe cluster | `cn` |

### Query Parameters

| Parameter | Type | Required | Description | Example |
| :--- | :--- | :--- | :--- | :--- |
| `grant_type` | `string` | **Required** | The authorization flow type. Must be set to `authorization_code` | `authorization_code` |
| `code` | `string` | **Required** | User login authorization code | `(code)` |
| `client_id` | `string` | Optional | Client ID (often empty in third-party service setups) | `""` |
| `redirect_uri` | `string` | Optional | Redirect URI used during code authorization | `http://homeassistant.local:8123` |

### Request Body Schema

*   **`auth_code`** (`string`, Required): The authorization code matching the `code` query parameter.

### Request Example

```http
POST https://(lambdaUrl)/oauth/v2/token?grant_type=authorization_code&code=some-auth-code&client_id=&redirect_uri=http://homeassistant.local:8123
Content-Type: application/json
CLOUD_SERVERS_DOMAIN: cn

{
  "auth_code": "some-auth-code"
}
```

---

## HTTP Response

### Response Status
*   **Code:** `200 OK` (success)
*   **Content-Type:** `application/json`

### Response Body Fields

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `access_token` | `string` | **Required** | The access token used to authenticate subsequent API requests |
| `expires_in` | `integer` | **Required** | The lifetime in seconds of the access token (e.g., 7200 seconds) |
| `refresh_token` | `string` | **Required** | The token used to refresh expired access tokens |
| `token_type` | `string` | **Required** | The type of token returned. Typically `Bearer` |

### Response Example

```json
{
    "access_token": "_______",
    "expires_in": 7200,
    "refresh_token": "_____________",
    "token_type": "Bearer"
}
```

---

## Related Endpoints

*   [Refresh OAuth Token Request](file:///C:/Users/zacsa/Documents/antigravity/eager-pasteur/refresh_token.md)

