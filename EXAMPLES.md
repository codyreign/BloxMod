# Examples

## Elevator (step on to go up, step off to return)
```
state.platform = nil
state.sensor = nil
state.playersOn = 0
state.isUp = false
state.isMoving = false
state.moveTimer = 0

local SPEED = 6
local LIFT = 17

chip.on("init", function()
    local chipPos = chip.getChipPosition()
    local downPos = { x = chipPos.x, y = chipPos.y + 2, z = chipPos.z }
    local upPos   = { x = downPos.x, y = downPos.y + LIFT, z = downPos.z }

    state.downPos = downPos
    state.upPos = upPos

    state.platform = chip.spawn({
        size = { x = 8, y = 1, z = 8 },
        position = downPos,
        anchored = true,
        canCollide = true,
        color = { r = 0.31, g = 0.71, b = 1 },
    })

    state.sensor = chip.spawn({
        size = { x = 7.5, y = 4, z = 7.5 },
        position = { x = downPos.x, y = downPos.y + 2, z = downPos.z },
        anchored = true,
        canCollide = false,
        sensor = true,
    })
    state.sensor:setVisible(false)
end)

local function moveBoth(pos)
    state.isMoving = true
    state.moveTimer = 0
    state.platform:moveTo(pos, SPEED)
    state.sensor:moveTo({ x = pos.x, y = pos.y + 2, z = pos.z }, SPEED)
end

chip.on("tick", function(dt)
    if not state.isMoving then return end
    state.moveTimer = state.moveTimer + dt
    if state.moveTimer >= (LIFT / SPEED) then
        state.isMoving = false
        state.moveTimer = 0
    end
end)

chip.on("playerEntered", function(player, hitSensor)
    if hitSensor ~= state.sensor then return end
    state.playersOn = state.playersOn + 1
    if state.isMoving or state.isUp then return end
    state.isUp = true
    moveBoth(state.upPos)
end)

chip.on("playerLeft", function(player, hitSensor)
    if hitSensor ~= state.sensor then return end
    state.playersOn = state.playersOn - 1
    if state.playersOn < 0 then state.playersOn = 0 end
    if state.playersOn > 0 or state.isMoving or not state.isUp then return end
    state.isUp = false
    moveBoth(state.downPos)
end)
```
