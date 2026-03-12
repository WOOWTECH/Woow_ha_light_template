# Findings: Template Light Composer v1.0.0 Testing

## Test Environment

- **Home Assistant**: 2025.4.2 (podman container)
- **Platform**: Linux 6.17.0-14-generic
- **Test Date**: 2026-03-12
- **Test Method**: Automated REST API test suite (27 test cases)
- **Test Entities**: `input_boolean.test_light_state`, `input_number.test_brightness` (0-255), `input_number.test_color_temp` (2500-6500)

## Test Results: 27/27 PASS

| Phase | Tests | Result |
|-------|-------|--------|
| Basic Functionality | T01-T07 (7 tests) | 7/7 PASS |
| Edge Cases | T10a-T18 (7 tests) | 7/7 PASS |
| Turn On with Parameters | T19-T21 (4 tests) | 4/4 PASS |
| Integration Lifecycle | T22-T24 (3 tests) | 3/3 PASS |
| Action Edge Cases | T25-T30 (6 tests) | 6/6 PASS |

### Test Coverage Detail

**Basic Functionality (T01-T07)**
- T01: Entity exists with correct type (`light.`)
- T02: Attributes correct (color_temp mode, kelvin range 2500-6500)
- T03: State tracks `input_boolean` reactively
- T04: Brightness tracks `input_number` reactively
- T05: Color temp tracks `input_number` reactively
- T06: `light.turn_on` service fires turn-on action
- T07: `light.turn_off` service fires turn-off action

**Edge Cases (T10-T18)**
- T10a/T10b: Brightness boundary values (0 and 255)
- T11a/T11b: Color temp boundary values (2500K and 6500K)
- T16a: Rapid state changes — 10 rapid toggles, final state correct
- T16b: Rapid brightness sweep — sequential brightness changes converge
- T18: Extra state attributes exposed (level_entity, state_entity, etc.)

**Turn On with Parameters (T19-T21)**
- T19: `light.turn_on` with `brightness` parameter — sets brightness AND turns on
- T20: `light.turn_on` with `color_temp_kelvin` parameter — sets color_temp AND turns on
- T20b: `light.turn_on` with BOTH brightness + color_temp — all three actions fire correctly
- T21: Turn off/on cycle preserves brightness value

**Integration Lifecycle (T22-T24)**
- T22: Integration reload — entity survives (`homeassistant.reload_config_entry`)
- T23: State restoration after reload — brightness/color_temp restored
- T24: Entity functional post-reload — service calls still work

**Action Edge Cases (T25-T30)**
- T25: Turn on when already on (idempotent, no errors)
- T26: Turn off when already off (idempotent, no errors)
- T27: Set brightness while off, verify value applied on turn-on
- T28: Color temp consistent through on/off cycle
- T29: TRANSITION feature flag supported (`supported_features` includes transition bit)
- T30: `color_mode` reports `color_temp` when light is ON

## Issues Found & Fixed During Testing

### Bug 1: Config Flow — Action targets not collected (Critical)
- **Root Cause**: `async_step_actions` used progressive disclosure that required multiple form round-trips. Target selectors only appeared when user_input already contained action type, but the form was submitted before targets could be selected.
- **Fix**: Rewrote `async_step_actions` to show both type AND target fields upfront on first render.
- **File**: `config_flow.py`

### Bug 2: Action execution — Missing entity_id (Critical)
- **Root Cause**: Service calls to `input_boolean.turn_on` / `input_number.set_value` lacked the required `entity_id` parameter.
- **Fix**: Added `_get_target_entity_for_action()` method and auto-populate logic in `_execute_action()` to infer entity_id from the action role (turn_on/off → state_entity, set_level → level_entity, set_temperature → temperature_entity).
- **File**: `light.py`

### Bug 3: Action execution — Missing value for input_number.set_value (High)
- **Root Cause**: When calling `input_number.set_value`, the `value` parameter was not included in service_data.
- **Fix**: Auto-populate `value` from kwargs based on action_key (brightness for set_level, color_temp_kelvin for set_temperature).
- **File**: `light.py`

### Bug 4: Duplicate kwargs — TypeError (High)
- **Root Cause**: `async_turn_on` passed `brightness=brightness` explicitly AND via `**kwargs`, causing "got multiple values for keyword argument" error.
- **Fix**: Removed explicit keyword arguments, relying on `**kwargs` passthrough.
- **File**: `light.py`

### Bug 5: Turn on with params doesn't turn on (High)
- **Root Cause**: When `light.turn_on` was called with brightness/color_temp kwargs, the specialized actions (set_level/set_temperature) handled the values but the turn_on action was skipped due to `handled_by_specialized` guard.
- **Fix**: Always execute the turn_on action regardless of whether specialized actions ran.
- **File**: `light.py`

## HA Logs Analysis

- **No errors** from `template_light_composer` in HA logs
- **No warnings** specific to the integration (only standard "custom integration not tested" notice)
- **No exceptions** during any test phase including rapid state changes and reload cycles

## Recommendation: GO for Production Release

All 27 test cases pass across basic functionality, edge cases, parameter handling, lifecycle management, and action edge cases. The 5 bugs discovered during testing have been fixed and verified. No errors in HA logs. The integration handles boundary values, rapid state changes, idempotent operations, and reload/restore cycles correctly.

### Caveats
1. Testing was performed with `input_boolean`/`input_number` entities only. Script and scene action types were not tested with live entities (config flow accepts them but no runtime test).
2. Testing used a single config entry. Multi-instance behavior was not tested.
3. No long-term stability test (hours/days of continuous operation).
4. `availability_template` Jinja2 rendering was configured but not stress-tested with complex templates.
