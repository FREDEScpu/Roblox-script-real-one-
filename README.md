-- Roblox script real one
-- This script is for educational and entertainment purposes only; such scripts violate Roblox rules and provide an unfair advantage. I am not responsible for anything that happens to the account of anyone using this script.



local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")
local camera = workspace.CurrentCamera

-- Değişkenler
local aimlockEnabled = false
local speedEnabled = false
local invisflingEnabled = false
local invisibleEnabled = false
local noclipEnabled = false
local flyEnabled = false

local aimTarget = nil
local flySpeed = 50
local walkSpeedBackup = humanoid.WalkSpeed

-- NPC tespiti için
local function isNPC(character)
    if character:IsA("Model") and character:FindFirstChild("Humanoid") then
        if not game.Players:GetPlayerFromCharacter(character) then
            return true
        end
    end
    return false
end

-- En yakın hedefi bul (oyuncu veya NPC)
local function getClosestTarget()
    local nearest = nil
    local nearestDist = math.huge
    local pos = rootPart.Position
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("Model") and v:FindFirstChild("HumanoidRootPart") and v ~= character then
            local isPlayer = game.Players:GetPlayerFromCharacter(v)
            if isPlayer or isNPC(v) then
                local dist = (v.HumanoidRootPart.Position - pos).Magnitude
                if dist < nearestDist and dist < 200 then -- 200 çalışma mesafesi
                    nearest = v
                    nearestDist = dist
                end
            end
        end
    end
    return nearest
end

-- Aimlock fonksiyonu
local function aimlockLoop()
    while aimlockEnabled do
        if aimTarget and aimTarget:FindFirstChild("HumanoidRootPart") then
            local targetPos = aimTarget.HumanoidRootPart.Position
            camera.CFrame = CFrame.new(camera.CFrame.Position, targetPos)
        else
            aimTarget = getClosestTarget()
        end
        task.wait()
    end
end

-- Fly fonksiyonu
local function flyLoop()
    while flyEnabled do
        if character and rootPart then
            local moveDirection = humanoid.MoveDirection
            local forward = camera.CFrame.LookVector * moveDirection.Z
            local right = camera.CFrame.RightVector * moveDirection.X
            local up = Vector3.new(0, 1, 0) * moveDirection.Y
            rootPart.Velocity = (forward + right + up) * flySpeed
        end
        task.wait()
    end
end

-- Noclip fonksiyonu
local function noclipLoop()
    while noclipEnabled do
        if character then
            for _, part in ipairs(character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end
        task.wait()
    end
end

-- Invisfling fonksiyonu (etrafındaki her şeyi fırlat)
local function invisflingLoop()
    while invisflingEnabled do
        if character then
            -- Kendini görünmez yap
            for _, part in ipairs(character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.Transparency = 1
                end
            end
            -- Etraftaki tüm karakterleri fırlat
            local pos = rootPart.Position
            for _, v in ipairs(workspace:GetDescendants()) do
                if v:IsA("Model") and v:FindFirstChild("HumanoidRootPart") and v ~= character then
                    local isPlayer = game.Players:GetPlayerFromCharacter(v)
                    if isPlayer or isNPC(v) then
                        local hrp = v.HumanoidRootPart
                        local dist = (hrp.Position - pos).Magnitude
                        if dist < 50 then -- 50 birim içindekiler
                            hrp.Velocity = Vector3.new(0, 150, 0) + (hrp.Position - pos).Unit * 50
                        end
                    end
                end
            end
        end
        task.wait(0.1) -- 0.1 saniye aralık
    end
end

-- NPC throw (hedefteki NPC'yi fırlat)
local function throwNPC()
    if aimTarget and isNPC(aimTarget) and aimTarget:FindFirstChild("HumanoidRootPart") then
        local hrp = aimTarget.HumanoidRootPart
        hrp.Velocity = Vector3.new(0, 100, 0) + (hrp.Position - rootPart.Position).Unit * 80
        sendMessage("🗿 NPC fırlatıldı!")
    else
        sendMessage("❌ Hedefte NPC yok veya hedef seçili değil.")
    end
end

-- Invisible (tam görünmezlik) - ayrı bir toggle
local function setInvisible(state)
    if state then
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.Transparency = 1
            end
        end
    else
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.Transparency = 0
            end
        end
    end
end

-- Speed (40) toggle
local function setSpeed(state)
    if state then
        humanoid.WalkSpeed = 40
    else
        humanoid.WalkSpeed = 16 -- varsayılan
    end
end

-- GUI oluşturma
local function createGUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Parent = player:WaitForChild("PlayerGui")

    local mainFrame = Instance.new("Frame")
    mainFrame.Parent = screenGui
    mainFrame.Size = UDim2.new(0, 200, 0, 300)
    mainFrame.Position = UDim2.new(1, -210, 0, 10)
    mainFrame.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
    mainFrame.BackgroundTransparency = 0.3
    mainFrame.BorderSizePixel = 2
    mainFrame.BorderColor3 = Color3.new(0.5, 0.5, 0.5)

    local title = Instance.new("TextLabel")
    title.Parent = mainFrame
    title.Size = UDim2.new(1, 0, 0, 25)
    title.Text = "🥖 Bread's Cheat"
    title.TextColor3 = Color3.new(1, 1, 1)
    title.BackgroundTransparency = 1
    title.Font = Enum.Font.SourceSansBold
    title.TextScaled = true

    local function createButton(parent, name, yPos, toggleFunc)
        local btn = Instance.new("TextButton")
        btn.Parent = parent
        btn.Size = UDim2.new(0.9, 0, 0, 25)
        btn.Position = UDim2.new(0.05, 0, 0, yPos)
        btn.Text = name .. " [KAPALI]"
        btn.TextColor3 = Color3.new(1, 1, 1)
        btn.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
        btn.BorderColor3 = Color3.new(0.3, 0.3, 0.3)
        btn.Font = Enum.Font.SourceSans
        btn.TextScaled = true
        local state = false
        btn.MouseButton1Click:Connect(function()
            state = not state
            btn.Text = name .. (state and " [AÇIK]" or " [KAPALI]")
            toggleFunc(state)
        end)
        return btn
    end

    -- Butonlar
    local aimBtn = createButton(mainFrame, "Aimlock", 30, function(state)
        aimlockEnabled = state
        if state then
            aimTarget = getClosestTarget()
            spawn(aimlockLoop)
        end
    end)

    local speedBtn = createButton(mainFrame, "Speed (40)", 60, function(state)
        speedEnabled = state
        setSpeed(state)
    end)

    local invisflingBtn = createButton(mainFrame, "Invisfling", 90, function(state)
        invisflingEnabled = state
        if state then
            spawn(invisflingLoop)
        else
            -- Görünmezliği kapatma (eğer invisfling kapanırsa, invisible ayrı olduğu için sadece fling durur, görünmezlik kalabilir, ama biz invisfling kapanınca görünür yapalım mı? Hayır, invisible ayrı toggle, onu karıştırmayalım.)
            -- Ama invisfling açıkken karakter görünmez oluyor, kapatınca eski haline dönmeli mi? Invisible ayrı olduğu için, eğer invisible kapalıysa görünür yapalım.
            if not invisibleEnabled then
                setInvisible(false)
            end
        end
    end)

    local invisibleBtn = createButton(mainFrame, "Invisible", 120, function(state)
        invisibleEnabled = state
        setInvisible(state)
    end)

    local noclipBtn = createButton(mainFrame, "Noclip", 150, function(state)
        noclipEnabled = state
        if state then
            spawn(noclipLoop)
        else
            -- CanCollide'ı eski haline döndür
            if character then
                for _, part in ipairs(character:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = true
                    end
                end
            end
        end
    end)

    local flyBtn = createButton(mainFrame, "Fly", 180, function(state)
        flyEnabled = state
        if state then
            spawn(flyLoop)
        else
            if character and rootPart then
                rootPart.Velocity = Vector3.new(0, 0, 0)
            end
        end
    end)

    -- NPC throw için özel buton (toggle değil, tek kullanımlık)
    local npcBtn = Instance.new("TextButton")
    npcBtn.Parent = mainFrame
    npcBtn.Size = UDim2.new(0.9, 0, 0, 25)
    npcBtn.Position = UDim2.new(0.05, 0, 0, 215)
    npcBtn.Text = "🗿 NPC Fırlat"
    npcBtn.TextColor3 = Color3.new(1, 0.7, 0)
    npcBtn.BackgroundColor3 = Color3.new(0.3, 0.1, 0)
    npcBtn.BorderColor3 = Color3.new(0.5, 0.2, 0)
    npcBtn.Font = Enum.Font.SourceSansBold
    npcBtn.TextScaled = true
    npcBtn.MouseButton1Click:Connect(throwNPC)

    -- Kapat butonu
    local closeBtn = Instance.new("TextButton")
    closeBtn.Parent = mainFrame
    closeBtn.Size = UDim2.new(0.4, 0, 0, 25)
    closeBtn.Position = UDim2.new(0.3, 0, 0, 250)
    closeBtn.Text = "✕ Kapat"
    closeBtn.TextColor3 = Color3.new(1, 0.2, 0.2)
    closeBtn.BackgroundColor3 = Color3.new(0.3, 0, 0)
    closeBtn.BorderColor3 = Color3.new(0.5, 0, 0)
    closeBtn.Font = Enum.Font.SourceSansBold
    closeBtn.TextScaled = true
    closeBtn.MouseButton1Click:Connect(function()
        mainFrame.Visible = not mainFrame.Visible
    end)

    -- Açma butonu (eğer kapatıldıysa geri açmak için)
    local openBtn = Instance.new("TextButton")
    openBtn.Parent = screenGui
    openBtn.Size = UDim2.new(0, 40, 0, 40)
    openBtn.Position = UDim2.new(1, -50, 0, 10)
    openBtn.Text = "⚙️"
    openBtn.TextColor3 = Color3.new(1, 1, 1)
    openBtn.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
    openBtn.BackgroundTransparency = 0.5
    openBtn.BorderSizePixel = 0
    openBtn.Font = Enum.Font.SourceSansBold
    openBtn.TextScaled = true
    openBtn.Visible = false
    openBtn.MouseButton1Click:Connect(function()
        mainFrame.Visible = true
        openBtn.Visible = false
    end)

    -- mainFrame görünürlüğü değişince openBtn'ı kontrol et
    mainFrame:GetPropertyChangedSignal("Visible"):Connect(function()
        openBtn.Visible = not mainFrame.Visible
    end)

    sendMessage("✅ Cheat menüsü yüklendi! Sağ alt köşedeki ⚙️ ile aç/kapat.")
end

-- Mesaj fonksiyonu (GUI üzerinden bilgi vermek için)
local function sendMessage(msg)
    game:GetService("StarterGui"):SetCore("ChatMakeSystemMessage", {
        Text = "[BREAD] " .. msg,
        Color = Color3.new(0.3, 1, 0.3)
    })
end

-- Başlat
createGUI()

-- Karakter yenilendiğinde ayarları sıfırla
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = character:WaitForChild("Humanoid")
    rootPart = character:WaitForChild("HumanoidRootPart")
    -- Tüm toggle'ları kapat (GUI'deki durumlar güncellenmez, ama kullanıcı tekrar tıklarsa düzelir)
    aimlockEnabled = false
    speedEnabled = false
    invisflingEnabled = false
    invisibleEnabled = false
    noclipEnabled = false
    flyEnabled = false
    setSpeed(false)
    setInvisible(false)
    sendMessage("🔄 Karakter yenilendi, efektler sıfırlandı.")
end)
