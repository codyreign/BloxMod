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
