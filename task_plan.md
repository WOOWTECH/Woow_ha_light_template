# Task Plan: Template Light Composer Integration Testing

## Goal
Comprehensively test the `template_light_composer` Home Assistant custom integration on a Podman HA instance (port 18123) using demo entities. Cover all edge cases to validate production readiness.

## Environment
- **HA Instance**: localhost:18123 (Podman)
- **Credentials**: admin / admin123
- **Demo Integration**: https://www.home-assistant.io/integrations/demo/
- **Integration Source**: template_light_composer (cloned from GitHub)

## Phases

### Phase 1: Environment Setup `pending`
- [ ] Verify HA instance is running on port 18123
- [ ] Install the custom component into HA
- [ ] Enable demo integration for test entities
- [ ] Verify demo light entities are available

### Phase 2: Basic Functionality Tests `pending`
- [ ] T01: Create a basic template light (ON/OFF only)
- [ ] T02: Create a template light with brightness control
- [ ] T03: Create a template light with color temperature
- [ ] T04: Create a template light with all features (brightness + color temp)
- [ ] T05: Verify turn on / turn off actions work
- [ ] T06: Verify brightness control works
- [ ] T07: Verify color temperature control works

### Phase 3: Edge Case Tests `pending`
- [ ] T08: Entity with invalid/nonexistent source entity
- [ ] T09: Template rendering with unavailable entity
- [ ] T10: Boundary values — brightness 0 and 255
- [ ] T11: Boundary values — color temp at min (2500K) and max (6500K)
- [ ] T12: Duplicate unique_id creation attempt
- [ ] T13: Empty name / special characters in name
- [ ] T14: Config flow cancellation mid-step
- [ ] T15: State restoration after HA restart
- [ ] T16: Rapid state changes (flipping on/off quickly)
- [ ] T17: Availability template — entity goes unavailable
- [ ] T18: Action type: script execution
- [ ] T19: Action type: service call
- [ ] T20: Action type: scene activation

### Phase 4: Error Handling & Recovery `pending`
- [ ] T21: Remove source entity while integration is running
- [ ] T22: Integration unload/reload
- [ ] T23: Delete and re-add integration
- [ ] T24: Verify logs for error messages / warnings
- [ ] T25: Memory leak check (no growing listeners)

### Phase 5: Results & Report `pending`
- [ ] Compile test results
- [ ] Document issues found
- [ ] Provide go/no-go recommendation for release

## Errors Encountered
| Error | Attempt | Resolution |
|-------|---------|------------|
| (none yet) | | |
