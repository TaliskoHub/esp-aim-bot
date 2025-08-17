--[[
# TALISKOHUB - Interface Bonita (Estilo Latvix) + Aimbot + ESP (SEM BOTÕES DECORATIVOS)
- Clique na bolinha preta para abrir/fechar o menu.
- Apenas os botões "Aimbot" e "Esp Player" aparecem e funcionam.
- Script Roblox Lua em formato .md (markdown comentado).
--]]

-- Serviços
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

--[[
========================
==   GUI PRINCIPAL    ==
========================
]]

local gui = Instance.new("ScreenGui")
gui.Name = "TaliskoHubGUI"
gui.Parent = (pcall(function() return game.CoreGui end) and game.CoreGui) or LocalPlayer:WaitForChild("PlayerGui")

local openBtn = Instance.new("TextButton")
openBtn.Name = "OpenBtn"
openBtn.Size = UDim2.new(0, 35, 0, 35)
openBtn.Position = UDim2.new(0, 10, 0.5, -18)
openBtn.BackgroundColor3 = Color3.new(0,0,0)
openBtn.Text = ""
openBtn.BorderSizePixel = 0
openBtn.AutoButtonColor = true
openBtn.Parent = gui
openBtn.BackgroundTransparency = 0.15
openBtn.ZIndex = 100

local uicorner = Instance.new("UICorner", openBtn)
uicorner.CornerRadius = UDim.new(1,0)

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 265, 0, 140)
mainFrame.Position = UDim2.new(0, 55, 0.5, -70)
mainFrame.BackgroundColor3 = Color3.fromRGB(38,38,38)
mainFrame.Visible = false
mainFrame.Parent = gui
mainFrame.ZIndex = 101
mainFrame.BorderSizePixel = 0

local corner = Instance.new("UICorner", mainFrame)
corner.CornerRadius = UDim.new(0, 8)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 30)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundTransparency = 1
title.Text = "TALISKOHUB"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 22
title.Parent = mainFrame
title.ZIndex = 102

local mainLabel = Instance.new("TextLabel")
mainLabel.Size = UDim2.new(1, -20, 0, 20)
mainLabel.Position = UDim2.new(0, 10, 0, 35)
mainLabel.BackgroundTransparency = 1
mainLabel.Text = "Main"
mainLabel.TextColor3 = Color3.fromRGB(220,220,220)
mainLabel.Font = Enum.Font.GothamSemibold
mainLabel.TextXAlignment = Enum.TextXAlignment.Left
mainLabel.TextSize = 18
mainLabel.Parent = mainFrame
mainLabel.ZIndex = 102

local function makeButton(txt, order)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -30, 0, 38)
    btn.Position = UDim2.new(0, 15, 0, 65+(order-1)*45)
    btn.BackgroundColor3 = Color3.fromRGB(53,53,53)
    btn.Text = txt
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 18
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = true
    btn.ZIndex = 103
    local c = Instance.new("UICorner", btn)
    c.CornerRadius = UDim.new(0,5)
    btn.Parent = mainFrame
    return btn
end

local aimbotBtn = makeButton("Aimbot", 1)
local espBtn = makeButton("Esp Player", 2)

--[[
========================
==   MENU TOGGLE      ==
========================
]]
openBtn.MouseButton1Click:Connect(function()
    mainFrame.Visible = not mainFrame.Visible
end)

local dragging = false
local dragInput, dragStart, startPos

mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

mainFrame.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

--[[
========================
==      AIMBOT        ==
========================
]]

local aimbotON = false

local function getClosestPlayer()
    local closest, dist = nil, math.huge
    for _,plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("Head") then
            local head = plr.Character.Head
            local pos, visible = workspace.CurrentCamera:WorldToViewportPoint(head.Position)
            if visible then
                local mouseDist = (Vector2.new(pos.X, pos.Y) - Vector2.new(Mouse.X, Mouse.Y)).Magnitude
                if mouseDist < dist then
                    dist = mouseDist
                    closest = head
                end
            end
        end
    end
    if dist < 200 then
        return closest
    end
end

local aimConnection

local function activateAimbot()
    if aimConnection then aimConnection:Disconnect() end
    aimConnection = RunService.RenderStepped:Connect(function()
        local target = getClosestPlayer()
        if aimbotON and target then
            local cam = workspace.CurrentCamera
            local pos = cam:WorldToViewportPoint(target.Position)
            mousemoverel((pos.X-Mouse.X)*0.1, (pos.Y-Mouse.Y)*0.1)
        end
    end)
end

local function deactivateAimbot()
    if aimConnection then aimConnection:Disconnect() end
end

aimbotBtn.MouseButton1Click:Connect(function()
    aimbotON = not aimbotON
    if aimbotON then
        aimbotBtn.BackgroundColor3 = Color3.fromRGB(50,200,50)
        aimbotBtn.Text = "Aimbot (ON)"
        activateAimbot()
    else
        aimbotBtn.BackgroundColor3 = Color3.fromRGB(53,53,53)
        aimbotBtn.Text = "Aimbot"
        deactivateAimbot()
    end
end)

--[[
========================
==      ESP PLAYER    ==
========================
]]

local espON = false
local highlights = {}

local function addESP(plr)
    if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
        local old = plr.Character:FindFirstChild("TaliskoHighlight")
        if old then old:Destroy() end

        local highlight = Instance.new("Highlight")
        highlight.Name = "TaliskoHighlight"
        highlight.FillColor = Color3.fromRGB(255,0,0)
        highlight.OutlineColor = Color3.fromRGB(255,0,0)
        highlight.FillTransparency = 0.5
        highlight.OutlineTransparency = 0
        highlight.Adornee = plr.Character
        highlight.Parent = plr.Character
        highlights[plr] = highlight
    end
end

local function removeESP()
    for plr, highlight in pairs(highlights) do
        if highlight and highlight.Parent then
            highlight:Destroy()
        end
    end
    highlights = {}
end

local function updateESP()
    if espON then
        for _,plr in pairs(Players:GetPlayers()) do
            addESP(plr)
        end
    else
        removeESP()
    end
end

Players.PlayerAdded:Connect(function(plr)
    plr.CharacterAdded:Connect(function()
        wait(1)
        if espON then addESP(plr) end
    end)
end)

for _, plr in ipairs(Players:GetPlayers()) do
    plr.CharacterAdded:Connect(function()
        wait(1)
        if espON then addESP(plr) end
    end)
end

espBtn.MouseButton1Click:Connect(function()
    espON = not espON
    if espON then
        espBtn.BackgroundColor3 = Color3.fromRGB(50,200,50)
        espBtn.Text = "Esp Player (ON)"
        updateESP()
    else
        espBtn.BackgroundColor3 = Color3.fromRGB(53,53,53)
        espBtn.Text = "Esp Player"
        updateESP()
    end
end)

--[[
========================
==      FIM           ==
========================
]]

LocalPlayer.OnRemoving:Connect(function()
    gui:Destroy()
end)
