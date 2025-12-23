# Chip Runtime API

This document describes the API available to user code running inside a Chip.
The runtime is sandboxed and does not expose Roblox instances or `require`.

## Global Objects

### `chip`
Main interface for event binding, logging, and spawning allowed objects.

#### `chip.on(eventName, callback)`
Registers a handler for a supported event.
- `eventName` (string): Event to subscribe to.
- `callback` (function): Called with event arguments.

Supported events:
- `"init"( )`: Fired once after compile/validate before any other events.
- `"tick"(dt)`: Fired at most `MaxTicksPerSecond`. `dt` is seconds since last tick.
- `"message"(msg)`: Fired when the owner sends a message via the editor (future).

Notes:
- No re-entrant callbacks. Events are serialized per chip.
- If a handler throws a runtime error, the chip halts.

#### `chip.log(message)`
Logs a message to the owner’s output panel.

#### `chip.warn(message)`
Logs a warning to the owner’s output panel.

#### `chip.error(message)`
Raises a runtime error and halts the chip.

#### `chip.spawn(partSpec)`
Creates a new part owned by the chip runtime.
- `partSpec` (table): Optional configuration.
  - `size` (table): `{x, y, z}` in studs, defaults to `{1,1,1}`.
  - `color` (table): `{r, g, b}` in 0..1, defaults to `{1,1,1}`.
  - `anchored` (bool): Defaults `true`.
  - `position` (table): `{x, y, z}` world position.

Returns: `handle` (table) with methods:
- `handle:setPosition(x, y, z)`
- `handle:setColor(r, g, b)`
- `handle:setSize(x, y, z)`
- `handle:destroy()`

Notes:
- All spawned parts are cleaned up on stop.
- Spawn limits enforced by `MaxSpawnedInstances`.

#### `chip.time()`
Returns the chip runtime time in seconds since `init`.

#### `chip.rand()`
Deterministic pseudo-random number in `[0, 1)` based on chip seed.

## Global Tables

### `state`
Persistent per-runtime state table. It resets when the chip is removed.
Example:
```
state.timer = (state.timer or 0) + dt
```

## Budgets and Limits

Budgets are defined in `ChipShared.Budgets` and enforced by the runtime:
- `MaxTicksPerSecond`
- `MaxInstructionsPerTick`
- `MaxTotalInstructions`
- `MaxRuntimeSeconds`
- `MaxSpawnedInstances`
- `MaxLogsPerSecond`
- `MaxEventQueue`

When a budget is exceeded, the chip halts and performs cleanup.

## Error Handling

Runtime errors halt the chip immediately. Errors are shown to the owner.

## Examples

Tick log once per second:
```
local state = {}

chip.on("tick", function(dt)
    state.timer = (state.timer or 0) + dt
    if state.timer >= 1 then
        state.timer = 0
        chip.log("hello")
    end
end)
```
