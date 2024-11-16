local players = game:GetService("Players")
local localPlayer = players.LocalPlayer
local camera = game:GetService("Workspace").CurrentCamera
local userInputService = game:GetService("UserInputService")
local runService = game:GetService("RunService")

local aimEnabled, aimToggle, chamsEnabled = false, false, false
local trackedPlayer = nil
local aimKey = Enum.KeyCode.LeftAlt
local aimRadius = 300
local chamsObjects = {}

-- Функция создания GUI
local function createUIElement(className, properties, parent)
    local element = Instance.new(className, parent)
    for prop, value in pairs(properties) do
        element[prop] = value
    end
    return element
end

-- Создание интерфейса
local screenGui = createUIElement("ScreenGui", { Name = "AimbotChamsMenu", Parent = localPlayer:WaitForChild("PlayerGui") })
local mainFrame = createUIElement("Frame", {
    Size = UDim2.new(0, 200, 0, 230),
    Position = UDim2.new(0.5, -100, 0.4, -75),
    BackgroundColor3 = Color3.fromRGB(50, 50, 50),
    Visible = false,
    Active = true,
    Draggable = true,
}, screenGui)

createUIElement("TextLabel", {
    Size = UDim2.new(1, 0, 0, 30),
    Position = UDim2.new(0, 0, 0, -30),
    Text = "Aimbot & CHAMS",
    BackgroundTransparency = 1,
    TextColor3 = Color3.fromRGB(255, 255, 255),
    Font = Enum.Font.SourceSans,
    TextSize = 18,
}, mainFrame)

local aimButton = createUIElement("TextButton", {
    Size = UDim2.new(1, -20, 0, 30),
    Position = UDim2.new(0, 10, 0, 10),
    Text = "Toggle Aimbot (OFF)",
    BackgroundColor3 = Color3.fromRGB(100, 100, 100),
}, mainFrame)

local chamsButton = createUIElement("TextButton", {
    Size = UDim2.new(1, -20, 0, 30),
    Position = UDim2.new(0, 10, 0, 50),
    Text = "Toggle CHAMS (OFF)",
    BackgroundColor3 = Color3.fromRGB(100, 100, 100),
}, mainFrame)

local radiusSliderLabel = createUIElement("TextLabel", {
    Size = UDim2.new(1, -20, 0, 20),
    Position = UDim2.new(0, 10, 0, 90),
    Text = "Aimbot Radius: " .. aimRadius,
    BackgroundTransparency = 1,
    TextColor3 = Color3.fromRGB(255, 255, 255),
    Font = Enum.Font.SourceSans,
    TextSize = 14,
}, mainFrame)

local increaseRadiusButton = createUIElement("TextButton", {
    Size = UDim2.new(0.5, -10, 0, 30),
    Position = UDim2.new(0, 10, 0, 120),
    Text = "+",
    BackgroundColor3 = Color3.fromRGB(100, 100, 100),
}, mainFrame)

local decreaseRadiusButton = createUIElement("TextButton", {
    Size = UDim2.new(0.5, -10, 0, 30),
    Position = UDim2.new(0.5, 0, 0, 120),
    Text = "-",
    BackgroundColor3 = Color3.fromRGB(100, 100, 100),
}, mainFrame)

local closeButton = createUIElement("TextButton", {
    Size = UDim2.new(1, -20, 0, 30),
    Position = UDim2.new(0, 10, 0, 200),
    Text = "Script by Alowyyy",
    BackgroundColor3 = Color3.fromRGB(100, 100, 100),
}, mainFrame)

-- Функции для работы с CHAMS
local function manageChams(player, action)
    if player == localPlayer or not player.Character then return end
    if action == "add" and not chamsObjects[player] then
        local teamColor = player.Team and player.Team.TeamColor.Color or Color3.fromRGB(255, 0, 0)
        chamsObjects[player] = {}
        for _, part in ipairs(player.Character:GetChildren()) do
            if part:IsA("BasePart") then
                local adornment = Instance.new("BoxHandleAdornment")
                adornment.Size = part.Size
                adornment.AlwaysOnTop = true
                adornment.ZIndex = 5
                adornment.Adornee = part
                adornment.Color3 = teamColor
                adornment.Transparency = 0.5
                adornment.Parent = part
                table.insert(chamsObjects[player], adornment)
            end
        end
    elseif action == "remove" and chamsObjects[player] then
        for _, adornment in ipairs(chamsObjects[player]) do
            adornment:Destroy()
        end
        chamsObjects[player] = nil
    end
end

-- Aimbot функции
local function aimAtHead(target)
    if target and target:FindFirstChild("Head") then
        local head = target.Head
        local adjustedPosition = head.Position + Vector3.new(0, head.Size.Y / 3, 0)
        camera.CFrame = CFrame.new(camera.CFrame.Position, adjustedPosition)
    end
end

local function getClosestPlayer()
    local closestPlayer, shortestDistance = nil, math.huge
    for _, player in ipairs(players:GetPlayers()) do
        if player ~= localPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local screenPoint, onScreen = camera:WorldToScreenPoint(player.Character.Head.Position)
            local distance = (Vector2.new(screenPoint.X, screenPoint.Y) - Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)).Magnitude
            if onScreen and distance < shortestDistance and distance <= aimRadius then
                closestPlayer = player
                shortestDistance = distance
            end
        end
    end
    return closestPlayer
end

-- Обработчики событий
aimButton.MouseButton1Click:Connect(function()
    aimToggle = not aimToggle
    aimButton.Text = "Toggle Aimbot (" .. (aimToggle and "ON" or "OFF") .. ")"
end)

chamsButton.MouseButton1Click:Connect(function()
    chamsEnabled = not chamsEnabled
    chamsButton.Text = "Toggle CHAMS (" .. (chamsEnabled and "ON" or "OFF") .. ")"
end)

increaseRadiusButton.MouseButton1Click:Connect(function()
    aimRadius = aimRadius + 50
    radiusSliderLabel.Text = "Aimbot Radius: " .. aimRadius
end)

decreaseRadiusButton.MouseButton1Click:Connect(function()
    aimRadius = math.max(50, aimRadius - 50)
    radiusSliderLabel.Text = "Aimbot Radius: " .. aimRadius
end)

userInputService.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.X then
        mainFrame.Visible = not mainFrame.Visible
    elseif input.KeyCode == aimKey and aimToggle then
        if trackedPlayer then
            trackedPlayer = nil
            aimEnabled = false
        else
            trackedPlayer = getClosestPlayer()
            aimEnabled = trackedPlayer ~= nil
        end
    end
end)

runService.RenderStepped:Connect(function()
    if chamsEnabled then
        for _, player in ipairs(players:GetPlayers()) do
            manageChams(player, "add")
        end
    else
        for _, player in ipairs(players:GetPlayers()) do
            manageChams(player, "remove")
        end
    end

    if aimEnabled and trackedPlayer and trackedPlayer.Character then
        aimAtHead(trackedPlayer.Character)
    end
end)
