local Players = game:GetService("Players")

local UserInputService = game:GetService("UserInputService")

local TweenService = game:GetService("TweenService")

local CoreGui = game:GetService("CoreGui")

local HttpService = game:GetService("HttpService")

local RunService = game:GetService("RunService")

local player = Players.LocalPlayer

local ConfigFile = "VantaLaggerConfig.json"

-- PODERES DEL KILL HUB: 23 - 32 - 70 - 90

local NIVELES = {

    Low     = { poder = 23 },

    Mid     = { poder = 32 },

    High    = { poder = 70 },

    Ultra   = { poder = 90 }

}

local keybind = Enum.KeyCode.H

local laggerActive = false

local lagThread = nil

local nivelActual = "Low"

local ventanaBloqueada = false

local ultraConfirmado = false

local UI_CONFIG = {

    MainBg       = Color3.fromRGB(10, 10, 10),

    TextColor    = Color3.fromRGB(255, 255, 255),

    ButtonInact  = Color3.fromRGB(25, 25, 25),

    ButtonAct    = Color3.fromRGB(255, 255, 255),

    ToggleOff    = Color3.fromRGB(30, 30, 30),

    ToggleOn     = Color3.fromRGB(255, 255, 255),

    Font         = Enum.Font.GothamBold,

    BorderColor  = Color3.fromRGB(80, 80, 80),

}

local function SaveConfig()

    local data = {

        Nivel = nivelActual,

        Bloqueado = ventanaBloqueada,

        PosX = mainFrame and mainFrame.Position.X.Scale or 0.15,

        PosY = mainFrame and mainFrame.Position.Y.Scale or 0.5,

        OffsetX = mainFrame and mainFrame.Position.X.Offset or 0,

        OffsetY = mainFrame and mainFrame.Position.Y.Offset or -50

    }

    pcall(function() writefile(ConfigFile, HttpService:JSONEncode(data)) end)

end

local function LoadConfig()

    local loadedPos = {ScaleX = 0.15, ScaleY = 0.5, OffsetX = 0, OffsetY = -50}

    if pcall(isfile, ConfigFile) and isfile(ConfigFile) then

        pcall(function()

            local data = HttpService:JSONDecode(readfile(ConfigFile))

            nivelActual = data.Nivel or "Low"

            ventanaBloqueada = data.Bloqueado or false

            loadedPos.ScaleX = data.PosX or 0.15

            loadedPos.ScaleY = data.PosY or 0.5

            loadedPos.OffsetX = data.OffsetX or 0

            loadedPos.OffsetY = data.OffsetY or -50

        end)

    end

    return loadedPos

end

local savedPos = LoadConfig()

local function bomb(poder)

    local main, spam = {}, {{}}

    local z = spam[1]

    for i = 1, 25 do local t = {} table.insert(z, t) z = t end

    local max = math.min(12000, poder * 50)

    for i = 1, max do table.insert(main, spam) end

    pcall(function() game:GetService("RobloxReplicatedStorage").SetPlayerBlockList:FireServer(main) end)

end

local toggleBall, toggleContainer, btnLow, btnMid, btnHigh, btnUltra, lockToggleContainer, lockToggleBall, lockToggleClick

local titleLabel, textEnable, textLagger, toggleClick

local mainFrame, glitchOverlay, scanline1, scanline2

local powerIndicator, statusDot

local screenGui, logoImage

local popupAbierto = false

local function mostrarConfirmacionUltra(callback)

    if popupAbierto then return end

    popupAbierto = true

    local overlay = Instance.new("Frame")

    overlay.Name = "UltraOverlay"

    overlay.BackgroundColor3 = Color3.fromRGB(0, 0, 0)

    overlay.BackgroundTransparency = 0.65

    overlay.Size = UDim2.new(1, 0, 1, 0)

    overlay.Position = UDim2.new(0, 0, 0, 0)

    overlay.ZIndex = 98

    overlay.Parent = screenGui

    local confirmFrame = Instance.new("Frame")

    confirmFrame.Name = "UltraConfirm"

    confirmFrame.BackgroundColor3 = Color3.fromRGB(5, 5, 5)

    confirmFrame.BorderSizePixel = 0

    confirmFrame.Size = UDim2.new(0, 240, 0, 100)

    confirmFrame.Position = UDim2.new(0.5, -120, 0.5, -50)

    confirmFrame.ZIndex = 100

    confirmFrame.Parent = screenGui

    confirmFrame.ClipsDescendants = true

    Instance.new("UICorner", confirmFrame).CornerRadius = UDim.new(0, 10)

    

    local confirmStroke = Instance.new("UIStroke", confirmFrame)

    confirmStroke.Color = Color3.fromRGB(255, 255, 255)

    confirmStroke.Thickness = 2

    confirmStroke.Transparency = 0.3

    local warningIcon = Instance.new("TextLabel", confirmFrame)

    warningIcon.BackgroundTransparency = 1

    warningIcon.Size = UDim2.new(1, 0, 0, 24)

    warningIcon.Position = UDim2.new(0, 0, 0, 10)

    warningIcon.Font = Enum.Font.GothamBlack

    warningIcon.Text = "[ ! ] ULTRA MODE [ ! ]"

    warningIcon.TextColor3 = Color3.fromRGB(255, 255, 255)

    warningIcon.TextSize = 15

    warningIcon.ZIndex = 101

    local warningText = Instance.new("TextLabel", confirmFrame)

    warningText.BackgroundTransparency = 1

    warningText.Size = UDim2.new(1, -30, 0, 18)

    warningText.Position = UDim2.new(0, 15, 0, 36)

    warningText.Font = UI_CONFIG.Font

    warningText.Text = "Puede crashear tu juego y servidor."

    warningText.TextColor3 = Color3.fromRGB(200, 200, 200)

    warningText.TextSize = 9

    warningText.ZIndex = 101

    local warningText2 = Instance.new("TextLabel", confirmFrame)

    warningText2.BackgroundTransparency = 1

    warningText2.Size = UDim2.new(1, -30, 0, 16)

    warningText2.Position = UDim2.new(0, 15, 0, 52)

    warningText2.Font = UI_CONFIG.Font

    warningText2.Text = "Estas seguro de activar ULTRA?"

    warningText2.TextColor3 = Color3.fromRGB(160, 160, 160)

    warningText2.TextSize = 9

    warningText2.ZIndex = 101

    local btnYes = Instance.new("TextButton", confirmFrame)

    btnYes.BackgroundColor3 = Color3.fromRGB(255, 255, 255)

    btnYes.Size = UDim2.new(0, 90, 0, 26)

    btnYes.Position = UDim2.new(0, 25, 0, 68)

    btnYes.Font = UI_CONFIG.Font

    btnYes.Text = "SI, ACTIVAR"

    btnYes.TextColor3 = Color3.fromRGB(0, 0, 0)

    btnYes.TextSize = 11

    btnYes.AutoButtonColor = false

    btnYes.ZIndex = 101

    Instance.new("UICorner", btnYes).CornerRadius = UDim.new(0, 5)

    local btnNo = Instance.new("TextButton", confirmFrame)

    btnNo.BackgroundColor3 = Color3.fromRGB(40, 40, 40)

    btnNo.Size = UDim2.new(0, 90, 0, 26)

    btnNo.Position = UDim2.new(0, 125, 0, 68)

    btnNo.Font = UI_CONFIG.Font

    btnNo.Text = "NO, CANCELAR"

    btnNo.TextColor3 = Color3.fromRGB(255, 255, 255)

    btnNo.TextSize = 11

    btnNo.AutoButtonColor = false

    btnNo.ZIndex = 101

    Instance.new("UICorner", btnNo).CornerRadius = UDim.new(0, 5)

    local function cerrarPopup()

        confirmFrame:Destroy()

        overlay:Destroy()

        popupAbierto = false

    end

    btnYes.MouseEnter:Connect(function()

        TweenService:Create(btnYes, TweenInfo.new(0.2), {

            BackgroundColor3 = Color3.fromRGB(200, 200, 200),

            Size = UDim2.new(0, 95, 0, 28)

        }):Play()

    end)

    btnYes.MouseLeave:Connect(function()

        TweenService:Create(btnYes, TweenInfo.new(0.2), {

            BackgroundColor3 = Color3.fromRGB(255, 255, 255),

            Size = UDim2.new(0, 90, 0, 26)

        }):Play()

    end)

    btnNo.MouseEnter:Connect(function()

        TweenService:Create(btnNo, TweenInfo.new(0.2), {

            BackgroundColor3 = Color3.fromRGB(60, 60, 60),

            Size = UDim2.new(0, 95, 0, 28)

        }):Play()

    end)

    btnNo.MouseLeave:Connect(function()

        TweenService:Create(btnNo, TweenInfo.new(0.2), {

            BackgroundColor3 = Color3.fromRGB(40, 40, 40),

            Size = UDim2.new(0, 90, 0, 26)

        }):Play()

    end)

    btnYes.MouseButton1Click:Connect(function()

        cerrarPopup()

        ultraConfirmado = true

        callback(true)

    end)

    btnNo.MouseButton1Click:Connect(function()

        cerrarPopup()

        ultraConfirmado = false

        callback(false)

    end)

end

local function animarBoton(btn)

    TweenService:Create(btn, TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {

        Size = UDim2.new(0, btn.Size.X.Offset - 3, 0, btn.Size.Y.Offset - 3)

    }):Play()

    task.wait(0.1)

    TweenService:Create(btn, TweenInfo.new(0.2, Enum.EasingStyle.Elastic, Enum.EasingDirection.Out), {

        Size = UDim2.new(0, btn.Size.X.Offset + 3, 0, btn.Size.Y.Offset + 3)

    }):Play()

    task.wait(0.15)

    TweenService:Create(btn, TweenInfo.new(0.15, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {

        Size = UDim2.new(0, btn.Size.X.Offset, 0, btn.Size.Y.Offset)

    }):Play()

end

local function actualizarBotonesNivel()

    if nivelActual == "Low" then

        btnLow.BackgroundColor3 = UI_CONFIG.ButtonAct

        btnLow.TextColor3 = Color3.fromRGB(0, 0, 0)

        btnLow.BorderSizePixel = 0

    else

        btnLow.BackgroundColor3 = UI_CONFIG.ButtonInact

        btnLow.TextColor3 = Color3.fromRGB(255, 255, 255)

        btnLow.BorderSizePixel = 1

        btnLow.BorderColor3 = UI_CONFIG.BorderColor

    end

    if nivelActual == "Mid" then

        btnMid.BackgroundColor3 = UI_CONFIG.ButtonAct

        btnMid.TextColor3 = Color3.fromRGB(0, 0, 0)

        btnMid.BorderSizePixel = 0

    else

        btnMid.BackgroundColor3 = UI_CONFIG.ButtonInact

        btnMid.TextColor3 = Color3.fromRGB(255, 255, 255)

        btnMid.BorderSizePixel = 1

        btnMid.BorderColor3 = UI_CONFIG.BorderColor

    end

    if nivelActual == "High" then

        btnHigh.BackgroundColor3 = UI_CONFIG.ButtonAct

        btnHigh.TextColor3 = Color3.fromRGB(0, 0, 0)

        btnHigh.BorderSizePixel = 0

    else

        btnHigh.BackgroundColor3 = UI_CONFIG.ButtonInact

        btnHigh.TextColor3 = Color3.fromRGB(255, 255, 255)

        btnHigh.BorderSizePixel = 1

        btnHigh.BorderColor3 = UI_CONFIG.BorderColor

    end

    if nivelActual == "Ultra" then

        btnUltra.BackgroundColor3 = Color3.fromRGB(0, 0, 0)

        btnUltra.TextColor3 = Color3.fromRGB(255, 255, 255)

        btnUltra.BorderSizePixel = 1

        btnUltra.BorderColor3 = Color3.fromRGB(255, 255, 255)

    else

        btnUltra.BackgroundColor3 = UI_CONFIG.ButtonInact

        btnUltra.TextColor3 = Color3.fromRGB(255, 255, 255)

        btnUltra.BorderSizePixel = 1

        btnUltra.BorderColor3 = UI_CONFIG.BorderColor

    end

end

local function actualizarLockToggle()

    if lockToggleContainer then

        lockToggleContainer.BackgroundColor3 = ventanaBloqueada and UI_CONFIG.ToggleOn or UI_CONFIG.ToggleOff

    end

    if lockToggleBall then

        lockToggleBall.BackgroundColor3 = ventanaBloqueada and Color3.fromRGB(0, 0, 0) or Color3.fromRGB(255, 255, 255)

        if ventanaBloqueada then

            lockToggleBall.Position = UDim2.new(1, -14, 0.5, -7)

        else

            lockToggleBall.Position = UDim2.new(0, 2, 0.5, -7)

        end

    end

    if lockToggleClick then

        lockToggleClick.Text = ventanaBloqueada and "LOCK" or "UNLOCK"

        if ventanaBloqueada then

            lockToggleClick.BackgroundColor3 = Color3.fromRGB(255, 255, 255)

            lockToggleClick.TextColor3 = Color3.fromRGB(0, 0, 0)

        else

            lockToggleClick.BackgroundColor3 = Color3.fromRGB(40, 40, 40)

            lockToggleClick.TextColor3 = Color3.fromRGB(255, 255, 255)

        end

    end

end

local function actualizarSwitch()

    if toggleContainer then

        toggleContainer.BackgroundColor3 = laggerActive and UI_CONFIG.ToggleOn or UI_CONFIG.ToggleOff

    end

    if toggleBall then

        toggleBall.BackgroundColor3 = laggerActive and Color3.fromRGB(0, 0, 0) or Color3.fromRGB(255, 255, 255)

        if laggerActive then

            toggleBall.Position = UDim2.new(1, -18, 0.5, -9)

        else

            toggleBall.Position = UDim2.new(0, 3, 0.5, -9)

        end

    end

    if toggleClick then

        toggleClick.Text = laggerActive and "ON" or "OFF"

        if laggerActive then

            toggleClick.BackgroundColor3 = Color3.fromRGB(255, 255, 255)

            toggleClick.TextColor3 = Color3.fromRGB(0, 0, 0)

        else

            toggleClick.BackgroundColor3 = Color3.fromRGB(40, 40, 40)

            toggleClick.TextColor3 = Color3.fromRGB(255, 255, 255)

        end

    end

    if powerIndicator then

        if laggerActive then

            powerIndicator.Text = nivelActual:upper()

            powerIndicator.TextColor3 = Color3.fromRGB(255, 255, 255)

        else

            powerIndicator.Text = "STANDBY"

            powerIndicator.TextColor3 = Color3.fromRGB(120, 120, 120)

        end

    end

    if statusDot then

        if laggerActive and nivelActual == "Ultra" then

            statusDot.BackgroundColor3 = Color3.fromRGB(0, 0, 0)

        else

            statusDot.BackgroundColor3 = laggerActive and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(80, 80, 80)

        end

    end

end

local function triggerGlitch()

    if not mainFrame or not glitchOverlay then return end

    

    local duracion = 0.05 + math.random() * 0.1

    local intensidad = math.random(1, 3)

    

    glitchOverlay.Visible = true

    glitchOverlay.ImageTransparency = 0.7 + math.random() * 0.2

    

    local offsetX = (math.random() - 0.5) * intensidad * 4

    local offsetY = (math.random() - 0.5) * intensidad * 3

    

    TweenService:Create(mainFrame, TweenInfo.new(0.02, Enum.EasingStyle.Linear), {

        Position = UDim2.new(mainFrame.Position.X.Scale, mainFrame.Position.X.Offset + offsetX, mainFrame.Position.Y.Scale, mainFrame.Position.Y.Offset + offsetY)

    }):Play()

    

    if scanline1 then

        scanline1.Visible = math.random() < 0.5

        scanline1.Position = UDim2.new(0, 0, math.random(), 0)

    end

    if scanline2 then

        scanline2.Visible = math.random() < 0.5

        scanline2.Position = UDim2.new(0, 0, math.random(), 0)

    end

    

    if titleLabel and math.random() < 0.3 then

        local glitchTexts = {"VANTA L4GGER", "V4NTA LAGGER", "VANTA LAGG3R", "V@NTA LAGGER"}

        titleLabel.Text = glitchTexts[math.random(1, #glitchTexts)]

        task.delay(0.08, function()

            if titleLabel then titleLabel.Text = "VANTA LAGGER" end

        end)

    end

    

    task.delay(duracion, function()

        if mainFrame then

            TweenService:Create(mainFrame, TweenInfo.new(0.05, Enum.EasingStyle.Linear), {

                Position = UDim2.new(mainFrame.Position.X.Scale, mainFrame.Position.X.Offset - offsetX, mainFrame.Position.Y.Scale, mainFrame.Position.Y.Offset - offsetY)

            }):Play()

        end

        if glitchOverlay then

            glitchOverlay.Visible = false

        end

        if scanline1 then scanline1.Visible = false end

        if scanline2 then scanline2.Visible = false end

    end)

end

local function toggleLagger()

    laggerActive = not laggerActive

    local targetPos = laggerActive and UDim2.new(1, -18, 0.5, -9) or UDim2.new(0, 3, 0.5, -9)

    local targetColor = laggerActive and UI_CONFIG.ToggleOn or UI_CONFIG.ToggleOff

    TweenService:Create(toggleBall, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {

        Position = targetPos,

        BackgroundColor3 = laggerActive and Color3.fromRGB(0, 0, 0) or Color3.fromRGB(255, 255, 255)

    }):Play()

    TweenService:Create(toggleContainer, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {

        BackgroundColor3 = targetColor

    }):Play()

    toggleClick.Text = laggerActive and "ON" or "OFF"

    if laggerActive then

        toggleClick.BackgroundColor3 = Color3.fromRGB(255, 255, 255)

        toggleClick.TextColor3 = Color3.fromRGB(0, 0, 0)

        TweenService:Create(toggleClick, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {

            TextSize = 11

        }):Play()

        task.wait(0.15)

        TweenService:Create(toggleClick, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {

            TextSize = 9

        }):Play()

    else

        toggleClick.BackgroundColor3 = Color3.fromRGB(40, 40, 40)

        toggleClick.TextColor3 = Color3.fromRGB(255, 255, 255)

    end

    actualizarSwitch()

    if laggerActive then

        if lagThread then task.cancel(lagThread) end

        lagThread = task.spawn(function()

            while laggerActive do

                pcall(function() game:GetService("NetworkClient"):SetOutgoingKBPSLimit(80000) end)

                bomb(NIVELES[nivelActual].poder)

                task.wait(0.18)

            end

        end)

    else

        if lagThread then task.cancel(lagThread); lagThread = nil end

    end

end

local function cambiarNivel(nuevoNivel)

    if nuevoNivel == "Ultra" and not ultraConfirmado then

        mostrarConfirmacionUltra(function(acepto)

            if acepto then

                nivelActual = "Ultra"

                actualizarBotonesNivel()

                actualizarSwitch()

                SaveConfig()

                if laggerActive then

                    if lagThread then task.cancel(lagThread) end

                    lagThread = task.spawn(function()

                        while laggerActive do

                            pcall(function() game:GetService("NetworkClient"):SetOutgoingKBPSLimit(80000) end)

                            bomb(NIVELES[nivelActual].poder)

                            task.wait(0.18)

                        end

                    end)

                end

            end

        end)

        return

    end

    

    nivelActual = nuevoNivel

    actualizarBotonesNivel()

    actualizarSwitch()

    SaveConfig()

    if laggerActive then

        if lagThread then task.cancel(lagThread) end

        lagThread = task.spawn(function()

            while laggerActive do

                pcall(function() game:GetService("NetworkClient"):SetOutgoingKBPSLimit(80000) end)

                bomb(NIVELES[nivelActual].poder)

                task.wait(0.18)

            end

        end)

    end

end

if CoreGui:FindFirstChild("VantaLagger_UI") then CoreGui.VantaLagger_UI:Destroy() end

screenGui = Instance.new("ScreenGui")

screenGui.Name = "VantaLagger_UI"

screenGui.Parent = CoreGui

screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

screenGui.ResetOnSpawn = false

screenGui.IgnoreGuiInset = true

local mainFrame = Instance.new("Frame")

mainFrame.Name = "MainFrame"

mainFrame.BackgroundColor3 = UI_CONFIG.MainBg

mainFrame.BackgroundTransparency = 1

mainFrame.BorderSizePixel = 0

mainFrame.Size = UDim2.new(0, 200, 0, 120)

mainFrame.Position = UDim2.new(savedPos.ScaleX, savedPos.OffsetX, savedPos.ScaleY, savedPos.OffsetY)

mainFrame.Parent = screenGui

mainFrame.ClipsDescendants = true

logoImage = Instance.new("ImageLabel")

logoImage.Name = "LogoBackground"

logoImage.BackgroundTransparency = 1

logoImage.Size = UDim2.new(1, 0, 1, 0)

logoImage.Position = UDim2.new(0, 0, 0, 0)

logoImage.Image = "rbxassetid://80772694108292"

logoImage.ScaleType = Enum.ScaleType.Crop

logoImage.ZIndex = 0

logoImage.Parent = mainFrame

Instance.new("UICorner", logoImage).CornerRadius = UDim.new(0, 6)

local bgOverlay = Instance.new("Frame")

bgOverlay.BackgroundColor3 = Color3.fromRGB(0, 0, 0)

bgOverlay.BackgroundTransparency = 0.55

bgOverlay.BorderSizePixel = 0

bgOverlay.Size = UDim2.new(1, 0, 1, 0)

bgOverlay.ZIndex = 2

bgOverlay.Parent = mainFrame

Instance.new("UICorner", bgOverlay).CornerRadius = UDim.new(0, 6)

local mainStroke = Instance.new("UIStroke", mainFrame)

mainStroke.Color = Color3.fromRGB(80, 80, 80)

mainStroke.Thickness = 1

mainStroke.Transparency = 0.5

Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 6)

glitchOverlay = Instance.new("ImageLabel")

glitchOverlay.Name = "GlitchOverlay"

glitchOverlay.BackgroundTransparency = 1

glitchOverlay.Size = UDim2.new(1, 0, 1, 0)

glitchOverlay.Image = "rbxassetid://16255699706"

glitchOverlay.ImageTransparency = 1

glitchOverlay.ScaleType = Enum.ScaleType.Stretch

glitchOverlay.ZIndex = 10

glitchOverlay.Visible = false

glitchOverlay.Parent = mainFrame

scanline1 = Instance.new("Frame")

scanline1.Name = "Scanline1"

scanline1.BackgroundColor3 = Color3.fromRGB(255, 255, 255)

scanline1.BackgroundTransparency = 0.9

scanline1.Size = UDim2.new(1, 0, 0, 2)

scanline1.BorderSizePixel = 0

scanline1.ZIndex = 9

scanline1.Visible = false

scanline1.Parent = mainFrame

scanline2 = Instance.new("Frame")

scanline2.Name = "Scanline2"

scanline2.BackgroundColor3 = Color3.fromRGB(0, 0, 0)

scanline2.BackgroundTransparency = 0.85

scanline2.Size = UDim2.new(1, 0, 0, 3)

scanline2.BorderSizePixel = 0

scanline2.ZIndex = 9

scanline2.Visible = false

scanline2.Parent = mainFrame

task.spawn(function()

    while true do

        task.wait(math.random(3, 8))

        triggerGlitch()

    end

end)

local topLine = Instance.new("Frame")

topLine.BackgroundColor3 = Color3.fromRGB(255, 255, 255)

topLine.BackgroundTransparency = 0.85

topLine.BorderSizePixel = 0

topLine.Size = UDim2.new(1, -20, 0, 1)

topLine.Position = UDim2.new(0, 10, 0, 26)

topLine.ZIndex = 3

topLine.Parent = mainFrame

local midLine = Instance.new("Frame")

midLine.BackgroundColor3 = Color3.fromRGB(255, 255, 255)

midLine.BackgroundTransparency = 0.9

midLine.BorderSizePixel = 0

midLine.Size = UDim2.new(1, -20, 0, 1)

midLine.Position = UDim2.new(0, 10, 0, 56)

midLine.ZIndex = 3

midLine.Parent = mainFrame

local bottomLine = Instance.new("Frame")

bottomLine.BackgroundColor3 = Color3.fromRGB(255, 255, 255)

bottomLine.BackgroundTransparency = 0.9

bottomLine.BorderSizePixel = 0

bottomLine.Size = UDim2.new(1, -20, 0, 1)

bottomLine.Position = UDim2.new(0, 10, 0, 78)

bottomLine.ZIndex = 3

bottomLine.Parent = mainFrame

titleLabel = Instance.new("TextLabel", mainFrame)

titleLabel.BackgroundTransparency = 1

titleLabel.Position = UDim2.new(0, 12, 0, 0)

titleLabel.Size = UDim2.new(1, -80, 0, 26)

titleLabel.Font = Enum.Font.GothamBlack

titleLabel.Text = "VANTA LAGGER"

titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)

titleLabel.TextSize = 13

titleLabel.TextXAlignment = Enum.TextXAlignment.Left

titleLabel.TextYAlignment = Enum.TextYAlignment.Center

titleLabel.ZIndex = 3

statusDot = Instance.new("Frame")

statusDot.BackgroundColor3 = Color3.fromRGB(80, 80, 80)

statusDot.Size = UDim2.new(0, 5, 0, 5)

statusDot.Position = UDim2.new(0, 2, 0, 10)

statusDot.BorderSizePixel = 0

statusDot.ZIndex = 3

statusDot.Parent = titleLabel

Instance.new("UICorner", statusDot).CornerRadius = UDim.new(1, 0)

local grisSuave = Color3.fromRGB(200, 200, 200)

local blancoPuro = Color3.fromRGB(255, 255, 255)

task.spawn(function()

    while true do

        TweenService:Create(titleLabel, TweenInfo.new(0.4, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {

            TextColor3 = grisSuave,

            TextSize = 13

        }):Play()

        task.wait(0.4)

        TweenService:Create(titleLabel, TweenInfo.new(0.4, Enum.EasingStyle.Sine, Enum.EasingDirection.In), {

            TextColor3 = blancoPuro,

            TextSize = 14

        }):Play()

        task.wait(0.4)

    end

end)

lockToggleContainer = Instance.new("Frame", mainFrame)

lockToggleContainer.BackgroundColor3 = UI_CONFIG.ToggleOff

lockToggleContainer.Position = UDim2.new(1, -60, 0, 7)

lockToggleContainer.Size = UDim2.new(0, 50, 0, 16)

lockToggleContainer.ZIndex = 3

Instance.new("UICorner", lockToggleContainer).CornerRadius = UDim.new(1,0)

lockToggleBall = Instance.new("Frame", lockToggleContainer)

lockToggleBall.BackgroundColor3 = Color3.fromRGB(255, 255, 255)

lockToggleBall.Size = UDim2.new(0, 14, 0, 14)

lockToggleBall.Position = UDim2.new(0, 2, 0.5, -7)

lockToggleBall.ZIndex = 4

Instance.new("UICorner", lockToggleBall).CornerRadius = UDim.new(1,0)

lockToggleClick = Instance.new("TextButton", lockToggleContainer)

lockToggleClick.BackgroundTransparency = 0

lockToggleClick.BackgroundColor3 = Color3.fromRGB(40, 40, 40)

lockToggleClick.Size = UDim2.new(1,0,1,0)

lockToggleClick.ZIndex = 5

lockToggleClick.Font = UI_CONFIG.Font

lockToggleClick.Text = "UNLOCK"

lockToggleClick.TextSize = 7

lockToggleClick.TextColor3 = Color3.fromRGB(255, 255, 255)

lockToggleClick.TextXAlignment = Enum.TextXAlignment.Center

lockToggleClick.TextYAlignment = Enum.TextYAlignment.Center

lockToggleClick.MouseButton1Click:Connect(function()

    ventanaBloqueada = not ventanaBloqueada

    actualizarLockToggle()

    SaveConfig()

end)

lockToggleClick.AutoButtonColor = false

Instance.new("UICorner", lockToggleClick).CornerRadius = UDim.new(1,0)

textEnable = Instance.new("TextLabel", mainFrame)

textEnable.BackgroundTransparency = 1

textEnable.Position = UDim2.new(0, 10, 0, 32)

textEnable.Size = UDim2.new(0, 38, 0, 16)

textEnable.Font = UI_CONFIG.Font

textEnable.Text = "ENABLE"

textEnable.TextColor3 = UI_CONFIG.TextColor

textEnable.TextSize = 9

textEnable.TextXAlignment = Enum.TextXAlignment.Left

textEnable.ZIndex = 3

textLagger = Instance.new("TextLabel", mainFrame)

textLagger.BackgroundTransparency = 1

textLagger.Position = UDim2.new(0, 50, 0, 32)

textLagger.Size = UDim2.new(0, 42, 0, 16)

textLagger.Font = UI_CONFIG.Font

textLagger.Text = "LAGGER"

textLagger.TextColor3 = UI_CONFIG.TextColor

textLagger.TextSize = 9

textLagger.TextXAlignment = Enum.TextXAlignment.Left

textLagger.ZIndex = 3

toggleContainer = Instance.new("Frame", mainFrame)

toggleContainer.BackgroundColor3 = UI_CONFIG.ToggleOff

toggleContainer.Position = UDim2.new(1, -60, 0, 30)

toggleContainer.Size = UDim2.new(0, 50, 0, 20)

toggleContainer.ZIndex = 3

Instance.new("UICorner", toggleContainer).CornerRadius = UDim.new(1,0)

toggleBall = Instance.new("Frame", toggleContainer)

toggleBall.BackgroundColor3 = Color3.fromRGB(255, 255, 255)

toggleBall.Size = UDim2.new(0, 18, 0, 18)

toggleBall.Position = UDim2.new(0, 2, 0.5, -9)

toggleBall.ZIndex = 4

Instance.new("UICorner", toggleBall).CornerRadius = UDim.new(1,0)

toggleClick = Instance.new("TextButton", toggleContainer)

toggleClick.BackgroundTransparency = 0

toggleClick.BackgroundColor3 = Color3.fromRGB(40, 40, 40)

toggleClick.Size = UDim2.new(1,0,1,0)

toggleClick.ZIndex = 5

toggleClick.Font = UI_CONFIG.Font

toggleClick.Text = "OFF"

toggleClick.TextSize = 9

toggleClick.TextColor3 = Color3.fromRGB(255, 255, 255)

toggleClick.TextXAlignment = Enum.TextXAlignment.Center

toggleClick.TextYAlignment = Enum.TextYAlignment.Center

toggleClick.MouseButton1Click:Connect(toggleLagger)

toggleClick.AutoButtonColor = false

Instance.new("UICorner", toggleClick).CornerRadius = UDim.new(1,0)

task.spawn(function()

    while true do

        if laggerActive then

            TweenService:Create(toggleContainer, TweenInfo.new(0.8, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {

                Size = UDim2.new(0, 52, 0, 22)

            }):Play()

            task.wait(0.8)

            TweenService:Create(toggleContainer, TweenInfo.new(0.8, Enum.EasingStyle.Sine, Enum.EasingDirection.In), {

                Size = UDim2.new(0, 50, 0, 20)

            }):Play()

            task.wait(0.8)

        else

            task.wait(1)

        end

    end

end)

powerIndicator = Instance.new("TextLabel", mainFrame)

powerIndicator.BackgroundTransparency = 1

powerIndicator.Position = UDim2.new(0, 10, 0, 60)

powerIndicator.Size = UDim2.new(0, 80, 0, 14)

powerIndicator.Font = UI_CONFIG.Font

powerIndicator.Text = "STANDBY"

powerIndicator.TextColor3 = Color3.fromRGB(120, 120, 120)

powerIndicator.TextSize = 8

powerIndicator.TextXAlignment = Enum.TextXAlignment.Left

powerIndicator.ZIndex = 3

local btnY = 84

local btnW = 42

local btnH = 22

local espaciado = 5

local margenIzq = 9

local function aplicarEfectoHover(btn, colorActivo)

    btn.MouseEnter:Connect(function()

        TweenService:Create(btn, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {

            BackgroundColor3 = colorActivo or UI_CONFIG.ButtonAct,

            TextColor3 = Color3.fromRGB(0, 0, 0),

            Size = UDim2.new(0, btnW + 4, 0, btnH + 4)

        }):Play()

    end)

    btn.MouseLeave:Connect(function()

        actualizarBotonesNivel()

        TweenService:Create(btn, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {

            Size = UDim2.new(0, btnW, 0, btnH)

        }):Play()

    end)

end

btnLow = Instance.new("TextButton", mainFrame)

btnLow.Size = UDim2.new(0, btnW, 0, btnH)

btnLow.Position = UDim2.new(0, margenIzq, 0, btnY)

btnLow.Font = UI_CONFIG.Font

btnLow.Text = "LOW"

btnLow.TextColor3 = Color3.fromRGB(255, 255, 255)

btnLow.TextSize = 9

btnLow.AutoButtonColor = false

btnLow.BackgroundColor3 = UI_CONFIG.ButtonInact

btnLow.BorderSizePixel = 1

btnLow.BorderColor3 = UI_CONFIG.BorderColor

btnLow.ZIndex = 3

Instance.new("UICorner", btnLow).CornerRadius = UDim.new(0, 4)

btnLow.MouseButton1Click:Connect(function()

    cambiarNivel("Low")

    animarBoton(btnLow)

end)

aplicarEfectoHover(btnLow)

btnMid = Instance.new("TextButton", mainFrame)

btnMid.Size = UDim2.new(0, btnW, 0, btnH)

btnMid.Position = UDim2.new(0, margenIzq + btnW + espaciado, 0, btnY)

btnMid.Font = UI_CONFIG.Font

btnMid.Text = "MID"

btnMid.TextColor3 = Color3.fromRGB(255, 255, 255)

btnMid.TextSize = 9

btnMid.AutoButtonColor = false

btnMid.BackgroundColor3 = UI_CONFIG.ButtonInact

btnMid.BorderSizePixel = 1

btnMid.BorderColor3 = UI_CONFIG.BorderColor

btnMid.ZIndex = 3

Instance.new("UICorner", btnMid).CornerRadius = UDim.new(0, 4)

btnMid.MouseButton1Click:Connect(function()

    cambiarNivel("Mid")

    animarBoton(btnMid)

end)

aplicarEfectoHover(btnMid)

btnHigh = Instance.new("TextButton", mainFrame)

btnHigh.Size = UDim2.new(0, btnW, 0, btnH)

btnHigh.Position = UDim2.new(0, margenIzq + (btnW + espaciado) * 2, 0, btnY)

btnHigh.Font = UI_CONFIG.Font

btnHigh.Text = "HIGH"

btnHigh.TextColor3 = Color3.fromRGB(255, 255, 255)

btnHigh.TextSize = 9

btnHigh.AutoButtonColor = false

btnHigh.BackgroundColor3 = UI_CONFIG.ButtonInact

btnHigh.BorderSizePixel = 1

btnHigh.BorderColor3 = UI_CONFIG.BorderColor

btnHigh.ZIndex = 3

Instance.new("UICorner", btnHigh).CornerRadius = UDim.new(0, 4)

btnHigh.MouseButton1Click:Connect(function()

    cambiarNivel("High")

    animarBoton(btnHigh)

end)

aplicarEfectoHover(btnHigh)

btnUltra = Instance.new("TextButton", mainFrame)

btnUltra.Size = UDim2.new(0, btnW + 4, 0, btnH)

btnUltra.Position = UDim2.new(0, margenIzq + (btnW + espaciado) * 3, 0, btnY)

btnUltra.Font = UI_CONFIG.Font

btnUltra.Text = "ULTRA"

btnUltra.TextColor3 = Color3.fromRGB(255, 255, 255)

btnUltra.TextSize = 8

btnUltra.AutoButtonColor = false

btnUltra.BackgroundColor3 = UI_CONFIG.ButtonInact

btnUltra.BorderSizePixel = 1

btnUltra.BorderColor3 = UI_CONFIG.BorderColor

btnUltra.ZIndex = 3

Instance.new("UICorner", btnUltra).CornerRadius = UDim.new(0, 4)

btnUltra.MouseButton1Click:Connect(function()

    if nivelActual == "Ultra" then return end

    cambiarNivel("Ultra")

    animarBoton(btnUltra)

end)

aplicarEfectoHover(btnUltra, Color3.fromRGB(0, 0, 0))

actualizarBotonesNivel()

actualizarSwitch()

actualizarLockToggle()

local isDragging, dragStart, startPos = false, nil, nil

mainFrame.InputBegan:Connect(function(input)

    if ventanaBloqueada then return end

    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then

        isDragging = true

        dragStart = input.Position

        startPos = mainFrame.Position

        TweenService:Create(mainFrame, TweenInfo.new(0.15, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {

            Size = UDim2.new(0, 204, 0, 124)

        }):Play()

    end

end)

UserInputService.InputChanged:Connect(function(input)

    if not isDragging or ventanaBloqueada then return end

    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then

        local delta = input.Position - dragStart

        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)

    end

end)

mainFrame.InputEnded:Connect(function(input)

    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then

        isDragging = false

        TweenService:Create(mainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Elastic, Enum.EasingDirection.Out), {

            Size = UDim2.new(0, 200, 0, 120)

        }):Play()

        SaveConfig()

    end

end)

UserInputService.InputBegan:Connect(function(input, gp)

    if gp then return end

    if input.KeyCode == keybind then

        toggleLagger()

    end

end)
