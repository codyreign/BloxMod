# Chip Runtime API

This document describes the API available to user code running inside a Chip.
The runtime is sandboxed and does not expose Roblox instances or `require`.

## Global Objects

### `chip`
Main interface for event binding, logging, spawning allowed objects, and querying players.

#### `chip.on(eventName, callback)`
Registers a handler for a supported event.
- `eventName` (string): Event to subscribe to.
- `callback` (function): Called with event arguments.

Supported events:
- `"init"( )`: Fired once after compile/validate before any other events.
- `"tick"(dt)`: Fired at most `MaxTicksPerSecond`. `dt` is seconds since last tick.
- `"touch"(hit)`: Fired when the chip part is touched (raw hit instance, legacy).
- `"playerEntered"(player, sensor)`: Fired when a player enters a sensor part.
- `"playerLeft"(player, sensor)`: Fired when a player leaves a sensor part.

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
  - `size` / `Size`: `{x, y, z}` in studs, defaults to `{1,1,1}`.
  - `color` / `Color`: `{r, g, b}` in 0..1, defaults to `{0.31,0.71,1}`.
  - `position` / `Position`: `{x, y, z}` world position.
  - `anchored` / `Anchored`: boolean, defaults `true`.
  - `canCollide` / `CanCollide`: boolean, defaults `true`.
  - `sensor` / `Sensor`: boolean, enables player enter/leave events.

Returns: `handle` (table) with methods:
- `handle:setPosition(x, y, z)` or `handle:setPosition({x,y,z})`
- `handle:setColor(r, g, b)` or `handle:setColor({r,g,b})`
- `handle:setSize(x, y, z)` or `handle:setSize({x,y,z})`
- `handle:moveTo(x, y, z, speed)` or `handle:moveTo({x,y,z}, speed)`
- `handle:onTouched(callback)`
- `handle:setSensor(true/false)`
- `handle:destroy()`

Notes:
- All spawned parts are cleaned up on stop.
- Spawn limits enforced by `MaxSpawnedInstances`.

#### `chip.newPart(partSpec)`
Alias of `chip.spawn`.

#### `chip.setPosition(handle, cframeOrPosition)`
Legacy helper for moving parts. Accepts a part handle and a `CFrame` or `{x,y,z}`.

#### `chip.getPosition(handle)`
Returns the `CFrame` of a part handle.

#### `chip.destroy(handle)`
Destroys a spawned part handle.

#### `chip.getPlayersInBox(center, size)`
Returns a list of player handles inside a box.
- `center`: `{x,y,z}`
- `size`: `{x,y,z}`

#### `chip.getChipPosition()`
Returns `{x,y,z}` for the chip model position.

#### `chip.getChipCFrame()`
Returns the chip model `CFrame`.

#### `chip.setChipVisible(visible)`
Shows or hides the chip part. `visible` is boolean.

#### `chip.setChipCollidable(collide)`
Enables or disables collisions for the chip part.

## Global Tables

### `state`
Persistent per-runtime state table. It resets when the chip is removed.

## Player Handle
Player handles are safe wrappers; they do not expose Roblox Instances.

Methods:
- `player:getName()`
- `player:getUserId()`
- `player:isAlive()`
- `player:getPosition()` returns `{x,y,z}` or `nil`
- `player:getHumanoidRoot()` same as `getPosition()`

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

Elevator platform (sensor + moveTo):
```
local platform = chip.spawn({
    size = {x = 8, y = 1, z = 8},
    position = {x = 0, y = 3, z = 0},
    anchored = true,
    canCollide = true,
    sensor = true,
})

local up = {x = 0, y = 20, z = 0}
local down = {x = 0, y = 3, z = 0}

chip.on("playerEntered", function(player, sensor)
    if sensor == platform then
        platform:moveTo(up, 8)
    end
end)

chip.on("playerLeft", function(player, sensor)
    if sensor == platform then
        platform:moveTo(down, 8)
    end
end)
```

Tick log once per second:
```
chip.on("tick", function(dt)
    state.timer = (state.timer or 0) + dt
    if state.timer >= 1 then
        state.timer = 0
        chip.log("hello")
    end
end)
```
