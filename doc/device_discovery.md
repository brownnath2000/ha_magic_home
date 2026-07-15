# Device Discovery (Device discovery)

This document describes the API used to discover scenes and devices (endpoints) associated with a user's account.

---

## HTTP Request

*   **Method:** `POST`
*   **URL:** `https://(lambdaUrl)/dnaproxy/v2/discover?license=`
*   **Content-Type:** `application/json`

### Headers

| Header Parameter | Type | Required | Description | Example |
| :--- | :--- | :--- | :--- | :--- |
| `CLOUD_SERVERS_DOMAIN` | `string` | **Required** | Target cluster domain used to route the request to the correct regional server:<br>• `cn`: China cluster<br>• `us`: USA cluster<br>• `eu`: Europe cluster | `cn` |

### Request Body Schema

*   **`directive`** (`object`, Required): The top-level container for the request.
    *   **`header`** (`object`, Required): Message metadata.
        *   `namespace` (`string`, Required): Must be `"DNA.Discovery"`.
        *   `name` (`string`, Required): Must be `"Discover"`.
        *   `interfaceVersion` (`string`, Required): Target version (e.g., `"2"`).
        *   `messageId` (`string`, Required): Unique request identifier (UUID).
    *   **`payload`** (`object`, Required): Parameters for device discovery.
        *   `scope` (`object`, Required): Access authorization scope.
            *   `type` (`string`, Required): The token type (e.g., `"BearerToken"`).
            *   `token` (`string`, Required): The access token obtained during [OAuth Token Exchange](file:///C:/Users/zacsa/Documents/antigravity/eager-pasteur/oauth_token.md).
        *   `options` (`object`, Optional): Additional options.
            *   `enableIntent` (`boolean`, Optional): Whether to enable intent mapping.
            *   `additionals` (`object`, Optional): Filter options.
                *   `familyids` (`array[string]`, Optional): List of family IDs to filter.
            *   `ignoreDevReachable` (`boolean`, Optional): Whether to ignore device reachability status.

### Request Example

```http
POST https://(lambdaUrl)/dnaproxy/v2/discover?license=
Content-Type: application/json
CLOUD_SERVERS_DOMAIN: cn

{
  "directive": {
    "header": {
      "namespace": "DNA.Discovery",
      "name": "Discover",
      "interfaceVersion": "2",
      "messageId": "messageId_mentionedOnce"
    },
    "payload": {
      "scope": {
        "type": "BearerToken",
        "token": "some-access-token"
      },
      "options": {
        "enableIntent": false,
        "additionals": {
          "familyids": ["xx", "xx"]
        },
        "ignoreDevReachable": false
      }
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

*   **`event`** (`object`, Required): Contains the response body.
    *   **`header`** (`object`, Required): Response metadata matching the request context.
    *   **`payload`** (`object`, Required): Details on status.
    *   **`scenes`** (`array[object]`, Required): List of discovered automation scenes.
    *   **`endpoints`** (`array[object]`, Required): List of discovered smart devices.

#### Scene Object Structure

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `sceneId` | `string` | Yes | Unique ID of the scene |
| `friendlyName` | `string` | Yes | Human-readable name of the scene (e.g. `"全开"`) |
| `icon` | `string` | Yes | URL to the scene icon |
| `manufacturerName` | `string` | Yes | Manufacturer name |
| `description` | `string` | Yes | Scene description |
| `displayCategories` | `array[string]` | Yes | Categories for UI categorization (e.g. `["SCENE_TRIGGER"]`) |
| `cookie` | `object` | Yes | Custom metadata for identifying and communicating with the scene / devices |
| `capabilities` | `array[object]` | Yes | Interfaces supported by the scene (e.g. `DNA.SceneControl`) |
| `floor` | `string` | Yes | Floor where the scene is located |
| `roomName` | `string` | Yes | Room name |
| `ignoreflag` | `boolean` | Yes | Flag to ignore this scene |

#### Endpoint (Device) Object Structure

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `endpointId` | `string` | Yes | Unique ID of the device |
| `parentId` | `string` | Yes | Parent device ID (for sub-devices/gateways) |
| `friendlyName` | `string` | Yes | Display name of the device (e.g. `"温控器 0c93"`) |
| `description` | `string` | Yes | Description |
| `manufacturerName`| `string` | Yes | Brand manufacturer (e.g., `BroadLink`) |
| `icon` | `string` | Yes | URL to device category icon image |
| `brand` | `string` | Yes | Brand identifier |
| `floor` | `string` | Yes | Floor |
| `roomName` | `string` | Yes | Room |
| `displayCategories`| `array[string]` | Yes | UI Category tag (e.g., `["AC"]` for Air Conditioner) |
| `cookie` | `object` | Yes | Device control mapping tokens (`did`, `pid`, `sdid`, `tokenlist`, `aeskeyToken`, etc.) |
| `isReachable` | `boolean` | Yes | True if the device is currently online/reachable |
| `capabilities` | `array[object]` | Yes | Control capabilities supported by this device (e.g., `DNA.PowerControl`, `DNA.ThermostatControl`) |
| `model` | `string` | Yes | Hardware model identifier |

---

### Response Example

```json
{
  "context": {
    "properties": null
  },
  "event": {
    "header": {
      "namespace": "DNA.Discovery",
      "name": "Response",
      "interfaceVersion": "2",
      "messageId": "messageId2_mentionedOnce"
    },
    "payload": {
      "status": 0,
      "type": ""
    },
    "scenes": [
      {
        "sceneId": "1",
        "friendlyName": "全开",
        "icon": "",
        "manufacturerName": "",
        "description": "",
        "displayCategories": [
          "SCENE_TRIGGER"
        ],
        "cookie": {
          "Pid": "pid_mentionedThreetimes",
          "aeskeyToken": "aeskeyToken_MentionedSixTimes",
          "bletoken": "b0a8fc7f",
          "did": "did_mentionedThrice",
          "openmqtt": "",
          "room": "全屋",
          "sceneflag": "blescene",
          "sdid": "1",
          "tokenlist": "[{\"devtype\":\"sameDevKeyAsControlMD_mentionedThreeTimes\",\"key\":\"aeskeyToken_MentionedSixTimes\",\"mac\":\"MACAddr_mentionedThreeTimes\",\"room\":\"全屋\"}]",
          "vtdid": "vtdid_mentionedThreeTimes"
        },
        "capabilities": [
          {
            "type": "DNAInterface",
            "interface": "DNA.SceneControl",
            "version": "2",
            "supportsDeactivation": false,
            "proactivelyReported": true
          }
        ],
        "floor": "",
        "roomName": "全屋",
        "ignoreflag": false
      },
      {
        "sceneId": "2",
        "friendlyName": "全关",
        "icon": "",
        "manufacturerName": "",
        "description": "",
        "displayCategories": [
          "SCENE_TRIGGER"
        ],
        "cookie": {
          "Pid": "pid_mentionedThreetimes",
          "aeskeyToken": "aeskeyToken_MentionedSixTimes",
          "bletoken": "b0a8fc7f",
          "did": "did_mentionedThrice",
          "openmqtt": "",
          "room": "全屋",
          "sceneflag": "blescene",
          "sdid": "2",
          "tokenlist": "[{\"devtype\":\"sameDevKeyAsControlMD_mentionedThreeTimes\",\"key\":\"aeskeyToken_MentionedSixTimes\",\"mac\":\"MACAddr_mentionedThreeTimes\",\"room\":\"全屋\"}]",
          "vtdid": "vtdid_mentionedThreeTimes"
        },
        "capabilities": [
          {
            "type": "DNAInterface",
            "interface": "DNA.SceneControl",
            "version": "2",
            "supportsDeactivation": false,
            "proactivelyReported": true
          }
        ],
        "floor": "",
        "roomName": "全屋",
        "ignoreflag": false
      }
    ],
    "endpoints": [
      {
        "endpointId": "sdid_mentionedTwice",
        "parentId": "",
        "friendlyName": "温控器 0c93",
        "description": "",
        "manufacturerName": "BroadLink",
        "icon": "https://ihcv0.ibroadlink.com/ec4appsysinfo/category2/AC.png",
        "brand": "bl",
        "floor": "",
        "roomName": "全屋",
        "displayCategories": [
          "AC"
        ],
        "displayCategories_v2": "tokenKey_mentionedOnce",
        "cookie": {
          "did": "did_mentionedThrice",
          "pid": "pid_mentionedThreetimes",
          "sdid": "sdid_mentionedTwice",
          "spid": "spid_mentionedOnce",
          "userid": "",
          "tokenlist": "[{\"devtype\":\"sameDevKeyAsControlMD_mentionedThreeTimes\",\"key\":\"aeskeyToken_MentionedSixTimes\",\"mac\":\"MACAddr_mentionedThreeTimes\",\"room\":\"全屋\"}]",
          "devname": "温控器 0c93",
          "moduleid": "",
          "moduletype": "",
          "familyid": "",
          "familyname": "",
          "version": "v2",
          "isPreDefineCategory": null,
          "aeskeyToken": "aeskeyToken_MentionedSixTimes",
          "range": "{\"fixedTargetTemperature\":{\"max\":32,\"min\":16}}",
          "shortaddr": "4",
          "bletoken": "b0a8fc7f",
          "blecategory": "BLEAC",
          "vtdid": "vtdid_mentionedThreeTimes",
          "room": "全屋",
          "capDynamic": "true",
          "onlycommand": "",
          "bledevversion": "4.1.0.72.15",
          "orifriendlyName": "温控器 0c93",
          "gatewayblesupport": "",
          "loopSceneGatewaydid": ""
        },
        "isReachable": true,
        "capabilities": [
          {
            "type": "DNAInterface",
            "interface": "DNA.PowerControl",
            "version": "2",
            "properties": {
              "supported": [
                {
                  "name": "powerState",
                  "enums": [
                    "OFF",
                    "ON"
                  ]
                }
              ],
              "proactivelyReported": true,
              "retrievable": false
            },
            "actions": {
              "supported": [
                {
                  "name": "ChangePowerState"
                }
              ]
            }
          },
          {
            "type": "DNAInterface",
            "interface": "DNA.ThermostatControl",
            "version": "2",
            "properties": {
              "supported": [
                {
                  "name": "mode",
                  "enums": [
                    "HEAT",
                    "VENT",
                    "AUTO",
                    "COLD",
                    "DEHUMI"
                  ],
                  "capabilityAvailableExp": {
                    "Expression": "modeenable",
                    "Value": {
                      "0": false,
                      "1": true,
                      "default": true
                    }
                  },
                  "propertyValueExp": {
                    "Expression": "modeset",
                    "Value": {
                      "1": "COLD,HEAT,VENT",
                      "default": "COLD,HEAT,VENT,AUTO,DEHUMI"
                    }
                  }
                },
                {
                  "name": "fixedTargetTemperature",
                  "range": {
                    "max": 32,
                    "min": 16
                  },
                  "capabilityAvailableExp": {
                    "Expression": "tempenable",
                    "Value": {
                      "0": false,
                      "1": true,
                      "default": true
                    }
                  },
                  "maxValueExp": "maxtemp * 1",
                  "minValueExp": "mintemp * 1"
                }
              ],
              "proactivelyReported": true,
              "retrievable": true
            },
            "actions": {
              "supported": [
                {
                  "name": "SetMode"
                },
                {
                  "name": "SetFixedTargetTemperature"
                }
              ]
            }
          },
          {
            "type": "DNAInterface",
            "interface": "DNA.WindSpeedControl",
            "version": "2",
            "properties": {
              "supported": [
                {
                  "name": "windSpeed",
                  "enums": [
                    "AUTO",
                    "HIGH",
                    "LOW",
                    "MEDIUM"
                  ],
                  "capabilityAvailableExp": {
                    "Expression": "windenable",
                    "Value": {
                      "0": false,
                      "1": true,
                      "default": true
                    }
                  }
                }
              ],
              "proactivelyReported": true,
              "retrievable": true
            },
            "actions": {
              "supported": [
                {
                  "name": "SetWindSpeed"
                }
              ]
            }
          }
        ],
        "additional": null,
        "sn": "",
        "fwversion": "",
        "model": "ble.conditioner"
      }
    ],
    "hiddenEndPoints": null,
    "eventEndPoints": null,
    "endpoint": {
      "scope": {
        "type": "",
        "token": "",
        "mtag": "",
        "spacemtag": "",
        "endpointid": ""
      },
      "endpointId": ""
    }
  }
}
```
