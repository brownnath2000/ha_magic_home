# Color Modes Best Practices for Light Entities

This document describes the validation rules, structures, and implementation details for light entities in Home Assistant, focusing on the transition from legacy brightness/color configurations to the modern `ColorMode` specification.

---

## 1. Core Color Mode Rules

Home Assistant requires light entities to report their capabilities (`supported_color_modes`) and their active state (`color_mode`) explicitly. Failure to comply with these validation schemas will raise severe `HomeAssistantError` exceptions during entity registration or state updates.

### Supported Color Modes vs Active Color Mode
*   **`supported_color_modes`**: A set of `ColorMode` values declaring what control types the hardware supports (e.g. `{ColorMode.COLOR_TEMP, ColorMode.RGB}`).
*   **`color_mode`**: A single `ColorMode` representing the *active* state. It must be a member of the declared `supported_color_modes` set.

---

## 2. Validation Constraints

### Rule A: Color Mode Mutually Exclusive with Redundant Brightness
*   **The Constraint**: If a light entity supports complex modes like `ColorMode.COLOR_TEMP` or `ColorMode.RGB`, it **implicitly** supports brightness control. You **must not** include `ColorMode.BRIGHTNESS` in `supported_color_modes` along with these complex modes.
*   **Legacy Pitfall**:
    ```python
    # ❌ INVALID - Raises HomeAssistantError during state calculations
    self._attr_supported_color_modes = {ColorMode.COLOR_TEMP, ColorMode.BRIGHTNESS}
    ```
*   **Modern Implementation**:
    ```python
    # ✅ VALID - ColorMode.BRIGHTNESS is filtered out because COLOR_TEMP implies it
    self._attr_supported_color_modes = {ColorMode.COLOR_TEMP}
    ```

### Rule B: No Empty Supported Modes
*   **The Constraint**: `supported_color_modes` must never be empty or null if the light is being registered as a modern entity. If a light supports nothing except simple On/Off toggling (no brightness, temperature, or color adjustments), it must explicitly declare `ColorMode.ONOFF`.
*   **Implementation**:
    ```python
    if not self._attr_supported_color_modes:
        self._attr_supported_color_modes.add(ColorMode.ONOFF)
    ```

### Rule C: Mandatory Active Color Mode
*   **The Constraint**: If `supported_color_modes` is populated, the light **must** report a non-null `color_mode` when its state is computed. Returning `None` will crash the entity state writer.
*   **Implementation**:
    ```python
    @property
    def color_mode(self) -> ColorMode:
        """Return the active color mode of the light."""
        return self._attr_color_mode
    ```

---

## 3. Reference Implementation Patterns

Here is the correct initialization and state handling pattern for a Home Assistant Light entity:

### Entity Initialization (`__init__`)
```python
from homeassistant.components.light import ColorMode, LightEntity

class CustomLight(LightEntity):
    def __init__(self, device):
        self._attr_supported_color_modes = set()
        
        # Populate capabilities based on device features
        if device.supports_color_temp:
            self._attr_supported_color_modes.add(ColorMode.COLOR_TEMP)
        if device.supports_rgb:
            self._attr_supported_color_modes.add(ColorMode.RGB)
        if device.supports_brightness:
            self._attr_supported_color_modes.add(ColorMode.BRIGHTNESS)
            
        # 1. Enforce Mutual Exclusion
        if len(self._attr_supported_color_modes) > 1 and ColorMode.BRIGHTNESS in self._attr_supported_color_modes:
            self._attr_supported_color_modes.remove(ColorMode.BRIGHTNESS)
            
        # 2. Fallback to ONOFF if empty
        if not self._attr_supported_color_modes:
            self._attr_supported_color_modes.add(ColorMode.ONOFF)
            
        # 3. Establish initial active mode
        if ColorMode.COLOR_TEMP in self._attr_supported_color_modes:
            self._attr_color_mode = ColorMode.COLOR_TEMP
        elif ColorMode.RGB in self._attr_supported_color_modes:
            self._attr_color_mode = ColorMode.RGB
        elif ColorMode.BRIGHTNESS in self._attr_supported_color_modes:
            self._attr_color_mode = ColorMode.BRIGHTNESS
        else:
            self._attr_color_mode = ColorMode.ONOFF
```

### State Modification (`async_turn_on`)
```python
    async def async_turn_on(self, **kwargs):
        if ATTR_COLOR_TEMP_KELVIN in kwargs:
            self._attr_color_temp_kelvin = kwargs[ATTR_COLOR_TEMP_KELVIN]
            if ColorMode.COLOR_TEMP in self._attr_supported_color_modes:
                self._attr_color_mode = ColorMode.COLOR_TEMP
                
        if ATTR_RGB_COLOR in kwargs:
            self._attr_rgb_color = kwargs[ATTR_RGB_COLOR]
            if ColorMode.RGB in self._attr_supported_color_modes:
                self._attr_color_mode = ColorMode.RGB
                
        if ATTR_BRIGHTNESS in kwargs:
            self._attr_brightness = kwargs[ATTR_BRIGHTNESS]
```
