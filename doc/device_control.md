# Device Control (设备控制)

This document describes the API used to send control directives to a discovered device or scene.

---

## HTTP Request

*   **Method:** `POST`
*   **URL:** `https://(lambdaUrl)/dnaproxy/v2/control?license=`
*   **Content-Type:** `application/json`

### Headers

| Header Parameter | Type | Required | Description | Example |
| :--- | :--- | :--- | :--- | :--- |
| `CLOUD_SERVERS_DOMAIN` | `string` | **Required** | Target cluster domain used to route the request to the correct regional server:<br>• `cn`: China cluster<br>• `us`: USA cluster<br>• `eu`: Europe cluster | `cn` |

### Request Body Schema

*   **`directive`** (`object`, Required): The top-level container for the request.
    *   **`header`** (`object`, Required): Message metadata.
        *   `namespace` (`string`, Required): The capability interface namespace being controlled (e.g., `"DNA.PowerControl"`, `"DNA.ThermostatControl"`, or `"DNA.SceneControl"`).
        *   `name` (`string`, Required): The control action name (e.g., `"ChangePowerState"`, `"SetMode"`, or `"SetFixedTargetTemperature"`).
        *   `interfaceVersion` (`string`, Required): Target version (e.g., `"2"`).
        *   `messageId` (`string`, Required): Unique request identifier (UUID).
    *   **`endpoint`** (`object`, Required): The target device configuration.
        *   `scope` (`object`, Required): Access authorization scope.
            *   `type` (`string`, Required): The token type (e.g., `"BearerToken"`).
            *   `token` (`string`, Required): The access token obtained during [OAuth Token Exchange](file:///C:/Users/zacsa/Documents/antigravity/eager-pasteur/oauth_token.md).
        *   `endpointId` (`string`, Required): The unique ID of the target device (`endpointId` or `sceneId` from discovery).
        *   `cookie` (`object`, Required): The exact `cookie` object received for this device/scene during [Device Discovery](file:///C:/Users/zacsa/Documents/antigravity/eager-pasteur/device_discovery.md).
    *   **`payload`** (`object`, Required): Parameters specific to the action being performed (e.g., `{"powerState": "OFF"}`).

### Request Example

```http
POST https://(lambdaUrl)/dnaproxy/v2/control?license=
Content-Type: application/json
CLOUD_SERVERS_DOMAIN: cn

{
  "directive": {
    "header": {
      "namespace": "DNA.PowerControl",
      "name": "ChangePowerState",
      "interfaceVersion": "2",
      "messageId": "message_id_twice"
    },
    "endpoint": {
      "scope": {
        "type": "BearerToken",
        "token": "longTokenTwice"
      },
      "endpointId": "endpointThreeTimes",
      "cookie": {
        "Pid": null,
        "aeskeyToken": "tokenList_mentionedTwice",
        "bletoken": "bletoken_mentionedOnce",
        "did": "did_mentionedOnce_endsInMacAddr",
        "openmqtt": null,
        "room": "全屋",
        "sceneflag": null,
        "sdid": "endpointThreeTimes",
        "tokenlist": "[{\"devtype\":\"devTypeId_mentionedOnce\",\"key\":\"tokenList_mentionedTwice\",\"mac\":\"mentionedOnce_MacAddr\",\"room\":\"全屋\"}]",
        "vtdid": "vtdid_mentionedOnlyOnce",
        "pid": "pidMentionedOnlyOnce",
        "spid": "spidMentionedOnlyOnce",
        "userid": "",
        "devname": "全屋照明",
        "moduleid": "",
        "moduletype": "",
        "familyid": "",
        "familyname": "",
        "version": "v2",
        "isPreDefineCategory": null,
        "range": "{\"brightness\":{\"max\":100,\"min\":0},\"brightnessSteps\":{\"max\":100,\"min\":0},\"colortemp\":{\"max\":100,\"min\":1},\"colortempSteps\":{\"max\":100,\"min\":0}}",
        "shortaddr": "0",
        "blecategory": "RGBCWLIGHT",
        "bledevtype": "GROUP",
        "capDynamic": "false",
        "onlycommand": "",
        "bledevversion": "",
        "orifriendlyName": "全屋照明",
        "gatewayblesupport": "",
        "loopSceneGatewaydid": ""
      }
    },
    "payload": {
      "powerState": "OFF"
    }
  }
}
```

---

## HTTP Response

### Response Status
*   **Code:** `200 OK` (success)
*   **Content-Type:** `application/json`

### Response Body Schema

*   **`context`** (`object`): Represents the updated state properties resulting from the directive.
    *   **`properties`** (`array[object]`): List of updated attributes.
        *   `namespace` (`string`): The property's capability namespace.
        *   `name` (`string`): The name of the property that changed (e.g. `powerState`).
        *   `value` (`object`): A `Value` object representing the state details.
        *   `extend` (`string`): Extended metadata string.
        *   `timeOfSample` (`string`): ISO 8601 timestamp of when the value was recorded.
*   **`event`** (`object`): Response wrapper.
    *   **`header`** (`object`): Response metadata matching the request context.
    *   **`payload`** (`object`): Details of status (status `0` indicates success).
    *   **`endpoint`** (`object`): The targeted device info containing verification tokens and target ID.

#### Value Object Structure

| Parameter | Type | Required | Description | Example |
| :--- | :--- | :--- | :--- | :--- |
| `scale` | `string` | Yes | Unit or scale representation (often empty) | `""` |
| `scaleName` | `string` | Yes | Name of the scale (often empty) | `""` |
| `attributeName` | `string` | Yes | Name of the attribute being reported | `"powerState"` |
| `value` | `any` | Yes | The actual state value (e.g., `"OFF"`, or a numeric value) | `"OFF"` |
| `valueName` | `string` | Yes | Human-readable value name (often empty) | `""` |

### Response Example

```json
{
  "context": {
    "properties": [
      {
        "namespace": "DNA.PowerControl",
        "name": "powerState",
        "value": {
          "scale": "",
          "scaleName": "",
          "attributeName": "powerState",
          "value": "OFF",
          "valueName": ""
        },
        "extend": "",
        "timeOfSample": "2025-02-12T15:19:44.35Z"
      }
    ]
  },
  "event": {
    "header": {
      "namespace": "DNA.PowerControl",
      "name": "Response",
      "interfaceVersion": "2",
      "messageId": "message_id_twice"
    },
    "payload": {
      "status": 0,
      "type": ""
    },
    "scenes": null,
    "endpoints": null,
    "hiddenEndPoints": null,
    "eventEndPoints": null,
    "endpoint": {
      "scope": {
        "type": "BearerToken",
        "token": "longTokenTwice",
        "mtag": "",
        "spacemtag": "",
        "endpointid": ""
      },
      "endpointId": "endpointThreeTimes"
    }
  }
}
```
