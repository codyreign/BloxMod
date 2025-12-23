# Examples

## Elevator (button-controlled, sliding doors, inside/outside buttons)
```
local SPEED = 6
local LIFT = 25

local CAB_WIDTH = 10
local CAB_DEPTH = 8
local CAB_HEIGHT = 8
local WALL_THICK = 0.5
local DOOR_WIDTH = 3
local DOOR_HEIGHT = 6
local DOOR_OPEN = 2.2

local MOVE_SND = 1298733270
local DOOR_SND = 132683375031276

state.parts = state.parts or {}
state.goUp = state.goUp or false
state.moving = state.moving or false
state.progress = state.progress or 0
state.moveSound = state.moveSound or nil
state.doorOpen = state.doorOpen or true
state.baseCf = state.baseCf or nil

local function v3(x, y, z) return { x = x, y = y, z = z } end
local function clamp(v, lo, hi)
    if v < lo then return lo end
    if v > hi then return hi end
    return v
end

local function setMat(part, mat)
    part:setMaterial(mat)
end

local function add(name, part, ox, oy, oz)
    part._offsetVec = v3(ox, oy, oz)
    state.parts[name] = part
end

local function buildCab(baseCf)
    add("floor", chip.spawn({
        size = v3(CAB_WIDTH, 1, CAB_DEPTH),
        position = chip.toWorld(baseCf, v3(0, 0, 0)),
        anchored = true,
        canCollide = true,
        color = v3(0.2, 0.6, 1),
    }), 0, 0, 0)
    setMat(state.parts.floor, "Metal")

    add("ceil", chip.spawn({
        size = v3(CAB_WIDTH, 1, CAB_DEPTH),
        position = chip.toWorld(baseCf, v3(0, CAB_HEIGHT, 0)),
        anchored = true,
        canCollide = true,
        color = v3(0.2, 0.6, 1),
    }), 0, CAB_HEIGHT, 0)
    setMat(state.parts.ceil, "Metal")

    add("back", chip.spawn({
        size = v3(CAB_WIDTH, CAB_HEIGHT, WALL_THICK),
        position = chip.toWorld(baseCf, v3(0, CAB_HEIGHT / 2, -(CAB_DEPTH / 2) + (WALL_THICK / 2))),
        anchored = true,
        canCollide = true,
        color = v3(0.15, 0.5, 0.9),
    }), 0, CAB_HEIGHT / 2, -(CAB_DEPTH / 2) + (WALL_THICK / 2))
    setMat(state.parts.back, "Metal")

    add("left", chip.spawn({
        size = v3(WALL_THICK, CAB_HEIGHT, CAB_DEPTH),
        position = chip.toWorld(baseCf, v3(-(CAB_WIDTH / 2) + (WALL_THICK / 2), CAB_HEIGHT / 2, 0)),
        anchored = true,
        canCollide = true,
        color = v3(0.15, 0.5, 0.9),
    }), -(CAB_WIDTH / 2) + (WALL_THICK / 2), CAB_HEIGHT / 2, 0)
    setMat(state.parts.left, "Metal")

    add("right", chip.spawn({
        size = v3(WALL_THICK, CAB_HEIGHT, CAB_DEPTH),
        position = chip.toWorld(baseCf, v3((CAB_WIDTH / 2) - (WALL_THICK / 2), CAB_HEIGHT / 2, 0)),
        anchored = true,
        canCollide = true,
        color = v3(0.15, 0.5, 0.9),
    }), (CAB_WIDTH / 2) - (WALL_THICK / 2), CAB_HEIGHT / 2, 0)
    setMat(state.parts.right, "Metal")

    add("frontLeft", chip.spawn({
        size = v3((CAB_WIDTH - DOOR_WIDTH) / 2, CAB_HEIGHT, WALL_THICK),
        position = chip.toWorld(baseCf, v3(-(DOOR_WIDTH / 2) - ((CAB_WIDTH - DOOR_WIDTH) / 4), CAB_HEIGHT / 2, (CAB_DEPTH / 2) - (WALL_THICK / 2))),
        anchored = true,
        canCollide = true,
        color = v3(0.15, 0.5, 0.9),
    }), -(DOOR_WIDTH / 2) - ((CAB_WIDTH - DOOR_WIDTH) / 4), CAB_HEIGHT / 2, (CAB_DEPTH / 2) - (WALL_THICK / 2))
    setMat(state.parts.frontLeft, "Metal")

    add("frontRight", chip.spawn({
        size = v3((CAB_WIDTH - DOOR_WIDTH) / 2, CAB_HEIGHT, WALL_THICK),
        position = chip.toWorld(baseCf, v3((DOOR_WIDTH / 2) + ((CAB_WIDTH - DOOR_WIDTH) / 4), CAB_HEIGHT / 2, (CAB_DEPTH / 2) - (WALL_THICK / 2))),
        anchored = true,
        canCollide = true,
        color = v3(0.15, 0.5, 0.9),
    }), (DOOR_WIDTH / 2) + ((CAB_WIDTH - DOOR_WIDTH) / 4), CAB_HEIGHT / 2, (CAB_DEPTH / 2) - (WALL_THICK / 2))
    setMat(state.parts.frontRight, "Metal")

    add("frontTop", chip.spawn({
        size = v3(DOOR_WIDTH, CAB_HEIGHT - DOOR_HEIGHT, WALL_THICK),
        position = chip.toWorld(baseCf, v3(0, DOOR_HEIGHT + ((CAB_HEIGHT - DOOR_HEIGHT) / 2), (CAB_DEPTH / 2) - (WALL_THICK / 2))),
        anchored = true,
        canCollide = true,
        color = v3(0.15, 0.5, 0.9),
    }), 0, DOOR_HEIGHT + ((CAB_HEIGHT - DOOR_HEIGHT) / 2), (CAB_DEPTH / 2) - (WALL_THICK / 2))
    setMat(state.parts.frontTop, "Metal")

    add("doorL", chip.spawn({
        size = v3(DOOR_WIDTH / 2, DOOR_HEIGHT, WALL_THICK),
        position = chip.toWorld(baseCf, v3(-(DOOR_WIDTH / 4), DOOR_HEIGHT / 2, (CAB_DEPTH / 2) - (WALL_THICK / 2))),
        anchored = true,
        canCollide = true,
        color = v3(0.9, 0.9, 0.9),
    }), -(DOOR_WIDTH / 4), DOOR_HEIGHT / 2, (CAB_DEPTH / 2) - (WALL_THICK / 2))
    setMat(state.parts.doorL, "SmoothPlastic")

    add("doorR", chip.spawn({
        size = v3(DOOR_WIDTH / 2, DOOR_HEIGHT, WALL_THICK),
        position = chip.toWorld(baseCf, v3((DOOR_WIDTH / 4), DOOR_HEIGHT / 2, (CAB_DEPTH / 2) - (WALL_THICK / 2))),
        anchored = true,
        canCollide = true,
        color = v3(0.9, 0.9, 0.9),
    }), (DOOR_WIDTH / 4), DOOR_HEIGHT / 2, (CAB_DEPTH / 2) - (WALL_THICK / 2))
    setMat(state.parts.doorR, "SmoothPlastic")

    add("button", chip.spawn({
        size = v3(0.6, 0.6, 0.2),
        position = chip.toWorld(baseCf, v3((CAB_WIDTH / 2) - 0.4, 3, 1)),
        anchored = true,
        canCollide = false,
        color = v3(1, 0.6, 0.2),
    }), (CAB_WIDTH / 2) - 0.4, 3, 1)
    setMat(state.parts.button, "Neon")

    add("extButton", chip.spawn({
        size = v3(0.6, 0.6, 0.2),
        position = chip.toWorld(baseCf, v3(3, 3, (CAB_DEPTH / 2) + 0.4)),
        anchored = true,
        canCollide = false,
        color = v3(1, 0.8, 0.2),
    }), 3, 3, (CAB_DEPTH / 2) + 0.4)
    setMat(state.parts.extButton, "Neon")

    add("doorBtnInside", chip.spawn({
        size = v3(0.6, 0.6, 0.2),
        position = chip.toWorld(baseCf, v3(-3, 3, 1)),
        anchored = true,
        canCollide = false,
        color = v3(0.2, 1, 0.4),
    }), -3, 3, 1)
    setMat(state.parts.doorBtnInside, "Neon")
end

local function rebuildTargets(baseCf, liftY)
    for _, part in pairs(state.parts) do
        local o = part._offsetVec
        part._target = v3(o.x, o.y + liftY, o.z)
    end
end

local function applyTargets(baseCf)
    for _, part in pairs(state.parts) do
        chip.setCFrameAtOffset(part, baseCf, part._target)
    end
end

local function setDoor(open)
    local offset = open and DOOR_OPEN or 0
    state.parts.doorL._offsetVec.x = -(DOOR_WIDTH / 4) - offset
    state.parts.doorR._offsetVec.x = (DOOR_WIDTH / 4) + offset
    state.doorOpen = open
    state.parts.doorL:playSound(DOOR_SND, 0.7, 1, false)
end

chip.on("init", function()
    state.baseCf = chip.getChipCFrame()
    buildCab(state.baseCf)
    rebuildTargets(state.baseCf, 0)
    applyTargets(state.baseCf)
    setDoor(true)

    state.parts.button:onClick(function()
        state.goUp = not state.goUp
        state.moving = true
        state.progress = clamp(state.progress, 0, 1)
        setDoor(false)
        if state.moveSound then state.moveSound:Destroy() end
        state.moveSound = state.parts.floor:playSound(MOVE_SND, 0.6, 1, true)
    end)

    state.parts.extButton:onClick(function()
        if state.moving then return end
        setDoor(not state.doorOpen)
        rebuildTargets(state.baseCf, LIFT * state.progress)
        applyTargets(state.baseCf)
    end)

    state.parts.doorBtnInside:onClick(function()
        if state.moving then return end
        setDoor(not state.doorOpen)
        rebuildTargets(state.baseCf, LIFT * state.progress)
        applyTargets(state.baseCf)
    end)
end)

chip.on("tick", function(dt)
    if not state.moving then return end

    local step = (SPEED * dt) / LIFT
    state.progress = clamp(state.progress + (state.goUp and step or -step), 0, 1)

    rebuildTargets(state.baseCf, LIFT * state.progress)
    applyTargets(state.baseCf)

    if state.progress == 0 or state.progress == 1 then
        state.moving = false
        if state.moveSound then
            state.moveSound.Looped = false
            state.moveSound:Stop()
            state.moveSound:Destroy()
            state.moveSound = nil
        end
        setDoor(true)
    end
end)
```

## Blue Pet Follower (side + ground)
```
local FOLLOW_SPEED = 10
local ROAM_SPEED = 3
local FOLLOW_RADIUS = 30
local ROAM_RADIUS = 6
local SIDE_OFFSET = 5

state.t = state.t or 0
state.follower = state.follower or nil
state.home = state.home or nil

local function dist(ax, ay, az, bx, by, bz)
    local dx = bx - ax
    local dy = by - ay
    local dz = bz - az
    return math.sqrt(dx * dx + dy * dy + dz * dz)
end

local function moveTowards(part, tx, ty, tz, speed, dt)
    local cf = chip.getPosition(part)
    local pos = cf.Position
    local px, py, pz = pos.X, pos.Y, pos.Z

    local dx = tx - px
    local dy = ty - py
    local dz = tz - pz
    local d = math.sqrt(dx * dx + dy * dy + dz * dz)
    if d <= 0.01 then return end

    local step = speed * dt
    if step >= d then
        part:setPosition(tx, ty, tz)
        return
    end

    local nx = dx / d
    local ny = dy / d
    local nz = dz / d
    part:setPosition(px + nx * step, py + ny * step, pz + nz * step)
end

local function getNearestPlayer()
    local chipPos = chip.getChipPosition()
    if not chipPos then return nil end
    local players = chip.getPlayersInBox(chipPos, { x = 200, y = 200, z = 200 })
    return players[1]
end

chip.on("init", function()
    local chipPos = chip.getChipPosition()
    state.home = { x = chipPos.x, y = chipPos.y + 1, z = chipPos.z }

    state.follower = chip.spawn({
        size = { x = 1, y = 1, z = 1 },
        position = state.home,
        anchored = true,
        canCollide = false,
        color = { r = 0.2, g = 0.6, b = 1 },
    })
end)

chip.on("tick", function(dt)
    if not state.follower then return end
    state.t = state.t + dt

    local player = getNearestPlayer()
    if player then
        local p = player:getPosition()
        if p then
            local chipPos = chip.getChipPosition()
            local d = dist(p.x, p.y, p.z, chipPos.x, chipPos.y, chipPos.z)

            if d > FOLLOW_RADIUS then
                local dx = p.x - chipPos.x
                local dz = p.z - chipPos.z
                local len = math.sqrt(dx * dx + dz * dz)
                if len < 0.01 then len = 1 end

                local sx = -dz / len
                local sz = dx / len

                local tx = p.x + sx * SIDE_OFFSET
                local tz = p.z + sz * SIDE_OFFSET
                local ty = p.y + 1

                moveTowards(state.follower, tx, ty, tz, FOLLOW_SPEED, dt)
                return
            end
        end
    end

    local hx, hy, hz = state.home.x, state.home.y, state.home.z
    local rx = hx + math.cos(state.t) * ROAM_RADIUS
    local rz = hz + math.sin(state.t) * ROAM_RADIUS
    local ry = hy
    moveTowards(state.follower, rx, ry, rz, ROAM_SPEED, dt)
end)
```
