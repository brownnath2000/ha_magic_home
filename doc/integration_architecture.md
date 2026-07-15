# Integration Architecture & Structure

This document describes the structure of the `ha_magic_home` Home Assistant custom integration and how its modules coordinate to load configurations, authenticate with the BroadLink Magic Home API, discover endpoints, and register entities.

---

## 1. Directory Structure

```text
custom_components/ha_magic_home/
├── __init__.py          # Integration entry point (setup, unload, device discovery)
├── manifest.json        # Integration package declaration & requirements
├── config_flow.py       # Multi-step OAuth authentication wizard UI
├── light.py             # Light platform entity definitions (CCT, RGB, Brightness)
├── climate.py           # Thermostat/AC platform entity definitions
├── cover.py             # Curtain/Blinds platform entity definitions
├── translations/        # Translation files for the Config Flow UI
│   ├── en.json
│   └── zh-Hans.json
└── iot/                 # Core low-level Magic Home IoT wrapper
    ├── const.py         # Endpoints, domain configs, and capability maps
    ├── common.py        # HTTP POST control request and state reporting wrappers
    ├── device_class.py  # Pydantic data models for API JSON serialization
    └── iot_i18n.py      # Core translation wrapper
```

---

## 2. Component Collaboration Flow

### Step 1: Config Flow Authentication
1. **User Initiation**: The user sets up the integration in Home Assistant Settings.
2. **Cluster & Language selection (`config_flow.py`)**: The user selects their region (`cn`, `us`, `eu`) and UI language.
3. **Authentication**: The user inputs an authorization code (`code`) obtained from the BroadLink app login.
4. **Token Exchange**: `_validate_auth_code` sends a POST request to `/oauth/v2/token`, validating the code and fetching `access_token`, `refresh_token`, and `expires_in`.
5. **Entry Creation**: The credentials are saved to `core.config_entries`.

### Step 2: Integration Setup & Device Discovery (`__init__.py`)
1. **Entry Setup**: `async_setup_entry` is called by Home Assistant.
2. **Background Refresh**: A periodic task is scheduled to run every half of `expires_in` seconds, calling `refresh_token_handle` to refresh the access token.
3. **Device Discovery**: `async_get_devices` sends a POST to `/dnaproxy/v2/discover?license=` with the `CLOUD_SERVERS_DOMAIN` header.
4. **Data Deserialization**: The raw JSON response is parsed into the Pydantic `Discovery` model.
5. **Device Registration**: Endpoints are grouped by their display category (e.g. `LIGHT`, `AC`) in `hass.data[DOMAIN]['devices']`.
6. **Platform Forwarding**: `async_forward_entry_setups` is invoked, loading `light.py`, `climate.py`, and `cover.py`.

### Step 3: Entity Creation (`light.py`)
1. **Platform Setup**: `async_setup_entry` in `light.py` retrieves the `LIGHT` endpoint list.
2. **Instantiation**: Iterates over the endpoint list and instantiates `Light(LightEntity)` objects.
3. **Capability Mapping**: Maps supported properties (e.g., `colortemp`, `color`, `brightness`) to Home Assistant's supported color modes.
4. **State Reporting**: The light entity reports its state and parameters (`color_temp_kelvin`, `brightness`, `rgb_color`) to Home Assistant.

---

## 3. Controlling Entities

When a user triggers an action (e.g. turning off a light or changing brightness):
1. Home Assistant calls the platform's corresponding action method (e.g. `async_turn_on(**kwargs)`).
2. The entity translates Home Assistant parameters (e.g. translating standard 0-255 brightness to a percentage).
3. The platform delegates to `control_req(self, prop, value)` in `iot/common.py`.
4. `control_req` prepares the `ControlModel` payload, injects the authentication token and target cookie parameters, and dispatches the HTTP POST to `/dnaproxy/v2/control?license=`.
5. The cloud cluster receives the directive, controls the physical gateway/bulb, and returns the execution status.
