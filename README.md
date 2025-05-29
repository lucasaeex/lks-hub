--éolkshub
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

local settings = {
    espEnabled = false,
    teamCheck = true,
    invisibilityEnabled = false,
}

local originalAcessories = {}
local originalClothing = {}

-- UI
local gui = Instance.new("ScreenGui")
gui.Name = "ESPInvisGui"
gui.ResetOnSpawn = false
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local function createButton(text, posY, color)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 160, 0, 30)
    btn.Position = UDim2.new(0, 10, 0, posY)
    btn.BackgroundColor3 = color
    btn.Text = text
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 14
    btn.Parent = gui
    return btn
end

local btnEnableESP = createButton("Enable ESP", 10, Color3.fromRGB(30,180,70))
local btnDisableESP = createButton("Disable ESP", 50, Color3.fromRGB(200,50,50))
local btnEnableInvis = createButton("Enable Invisibility", 90, Color3.fromRGB(100,100,255))
local btnDisableInvis = createButton("Disable Invisibility", 130, Color3.fromRGB(255,100,100))

-- ESP
local espObjects = {}

local function createESP(player)
    if player == LocalPlayer then return end

    local function onCharacter(character)
        local head = character:WaitForChild("Head", 5)
        if not head then return end

        local billboard = Instance.new("BillboardGui")
        billboard.Name = "ESP"
        billboard.Size = UDim2.new(0, 100, 0, 20)
        billboard.Adornee = head
        billboard.AlwaysOnTop = true
        billboard.StudsOffset = Vector3.new(0, 2, 0)
        billboard.Parent = head

        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1, 0, 1, 0)
        label.BackgroundTransparency = 1
        label.Text = player.Name
        label.TextColor3 = Color3.fromRGB(255,0,0)
        label.TextScaled = true
        label.TextStrokeTransparency = 0
        label.Font = Enum.Font.SourceSansBold
        label.Parent = billboard

        espObjects[player] = billboard

        RunService.RenderStepped:Connect(function()
            if not settings.espEnabled or not character or not character:FindFirstChild("Head") then
                billboard.Enabled = false
                return
            end
            if settings.teamCheck and player.Team == LocalPlayer.Team then
                billboard.Enabled = false
                return
            end
            billboard.Enabled = true
        end)
    end

    if player.Character then
        onCharacter(player.Character)
    end
    player.CharacterAdded:Connect(onCharacter)
end

-- Aplicar ESP a todos
for _, p in ipairs(Players:GetPlayers()) do
    createESP(p)
end
Players.PlayerAdded:Connect(createESP)

-- Botões ESP
btnEnableESP.MouseButton1Click:Connect(function()
    settings.espEnabled = true
end)

btnDisableESP.MouseButton1Click:Connect(function()
    settings.espEnabled = false
    for _, esp in pairs(espObjects) do
        esp.Enabled = false
    end
end)

-- Invisibilidade com remoção de acessórios
local function setInvisibility(state)
    local char = LocalPlayer.Character
    if not char then return end

    if state then
        -- Salvar acessórios e roupas
        originalAcessories = {}
        originalClothing = {}

        for _, obj in ipairs(char:GetChildren()) do
            if obj:IsA("Accessory") then
                table.insert(originalAcessories, obj:Clone())
                obj:Destroy()
            elseif obj:IsA("Shirt") or obj:IsA("Pants") then
                table.insert(originalClothing, obj:Clone())
                obj:Destroy()
            end
        end

        -- Tornar partes invisíveis
        for _, part in ipairs(char:GetDescendants()) do
            if part:IsA("BasePart") or part:IsA("Decal") then
                part.Transparency = 1
            elseif part:IsA("ParticleEmitter") or part:IsA("Trail") then
                part.Enabled = false
            end
        end

    else
        -- Restaurar partes visuais
        for _, part in ipairs(char:GetDescendants()) do
            if part:IsA("BasePart") or part:IsA("Decal") then
                part.Transparency = 0
            elseif part:IsA("ParticleEmitter") or part:IsA("Trail") then
                part.Enabled = true
            end
        end

        -- Restaurar acessórios e roupas
        for _, acc in ipairs(originalAcessories) do
            acc.Parent = char
        end
        for _, cloth in ipairs(originalClothing) do
            cloth.Parent = char
        end

        originalAcessories = {}
        originalClothing = {}
    end
end

btnEnableInvis.MouseButton1Click:Connect(function()
    settings.invisibilityEnabled = true
    setInvisibility(true)
end)

btnDisableInvis.MouseButton1Click:Connect(function()
    settings.invisibilityEnabled = false
    setInvisibility(false)
end)
