local player = game.Players.LocalPlayer
local camera = workspace.CurrentCamera
local virtualInput = game:GetService("VirtualInputManager")

local function equipPistol()
    local backpack = player:WaitForChild("Backpack")
    local pistol = backpack:FindFirstChild("Pistol")
    if pistol then
        pistol.Parent = player.Character
        return player.Character:FindFirstChild("Pistol")
    end
    return nil
end

-- Raycast para checar se o NPC pode ser atingido
local function canHit(head)
    local origin = camera.CFrame.Position
    local direction = (head.Position - origin).Unit * (head.Position - origin).Magnitude
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    raycastParams.FilterDescendantsInstances = {player.Character}
    local result = workspace:Raycast(origin, direction, raycastParams)
    if result and result.Instance then
        -- Só acerta se a primeira coisa atingida for a cabeça do NPC
        return result.Instance == head
    end
    return false
end

local function getAliveNPCs()
    local npcs = {}
    for _, npc in pairs(workspace:GetChildren()) do
        if npc:IsA("Model") and npc ~= player.Character and npc:FindFirstChild("Head") and npc:FindFirstChildOfClass("Humanoid") then
            local humanoid = npc:FindFirstChildOfClass("Humanoid")
            if humanoid.Health > 0 and canHit(npc.Head) then
                table.insert(npcs, npc)
            end
        end
    end
    return npcs
end

local function fireAt(position)
    local screenPoint = camera:WorldToViewportPoint(position)
    virtualInput:SendMouseButtonEvent(screenPoint.X, screenPoint.Y, 0, true, game, 0)
    virtualInput:SendMouseButtonEvent(screenPoint.X, screenPoint.Y, 0, false, game, 0)
end

local function aimAndShootAtNPC(npc)
    local head = npc:FindFirstChild("Head")
    if not head then return end
    local humanoid = npc:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end

    local died = false
    local conn
    conn = humanoid.Died:Connect(function()
        died = true
        if conn then conn:Disconnect() end
    end)

    while humanoid.Health > 0 and not died and canHit(head) do
        fireAt(head.Position)
        task.wait(0.01)
    end
end

task.spawn(function()
    while true do
        local pistol = equipPistol()
        if pistol then
            local npcs = getAliveNPCs()
            for _, npc in ipairs(npcs) do
                aimAndShootAtNPC(npc)
            end
        end
        task.wait(0.1)
    end
end)
