# YAMADAHUB-- ==========================================
-- 1. ログイン（ローダー）部分（ユーザー名登録制）
-- ==========================================
local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local localPlayer = Players.LocalPlayer

-- ▼ ここに許可したいプレイヤーの名前を登録してください（30人）
local ALLOWED_USERNAMES = {
    "couseicousei",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
    "",
}

-- ユーザー名がリストに含まれているかチェックする関数
local function isAuthorized()
    local myName = localPlayer.Name
    for _, name in ipairs(ALLOWED_USERNAMES) do
        if myName == name then
            return true
        end
    end
    return false
end

-- すでに起動していれば既存のUIを削除
pcall(function()
    if CoreGui:FindFirstChild("YadaHubMobile") then
        CoreGui.YadaHubMobile:Destroy()
    end
end)

-- 起動時にユーザー名チェック
if not isAuthorized() then
    -- 許可されていない場合の警告UI
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "YadaHubMobile"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = CoreGui

    local keyFrame = Instance.new("Frame")
    keyFrame.Size = UDim2.new(0, 300, 0, 140)
    keyFrame.Position = UDim2.new(0.5, -150, 0.5, -70)
    keyFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    keyFrame.Parent = screenGui
    Instance.new("UICorner", keyFrame).CornerRadius = UDim.new(0, 8)

    local keyTitle = Instance.new("TextLabel")
    keyTitle.Size = UDim2.new(1, 0, 0, 35)
    keyTitle.Position = UDim2.new(0, 0, 0, 15)
    keyTitle.BackgroundTransparency = 1
    keyTitle.Text = "山田ハブ - アクセス拒否"
    keyTitle.TextColor3 = Color3.fromRGB(255, 80, 80)
    keyTitle.TextSize = 14
    keyTitle.Font = Enum.Font.Code
    keyTitle.Parent = keyFrame

    local errorLabel = Instance.new("TextLabel")
    errorLabel.Size = UDim2.new(1, -20, 0, 40)
    errorLabel.Position = UDim2.new(0, 10, 0, 55)
    errorLabel.BackgroundTransparency = 1
    errorLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    errorLabel.TextSize = 11
    errorLabel.Font = Enum.Font.Code
    errorLabel.Text = "あなたのアカウント名はこのハブの利用が許可されていません。"
    errorLabel.TextWrapped = true
    errorLabel.Parent = keyFrame

    local closeKeyButton = Instance.new("TextButton")
    closeKeyButton.Size = UDim2.new(0, 120, 0, 28)
    closeKeyButton.Position = UDim2.new(0.5, -60, 0, 100)
    closeKeyButton.BackgroundColor3 = Color3.fromRGB(120, 40, 40)
    closeKeyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeKeyButton.Text = "閉じる"
    closeKeyButton.Parent = keyFrame
    Instance.new("UICorner", closeKeyButton).CornerRadius = UDim.new(0, 5)

    closeKeyButton.MouseButton1Click:Connect(function()
        screenGui:Destroy()
    end)
    
    return -- 許可されていない場合はここで処理をストップ
end


-- ==========================================
-- 2. 認証成功したあとに実行されるメインスクリプト
-- ==========================================
print("認証成功！メインスクリプトをロードします...")

-- Services
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Stats = game:GetService("Stats")
local HttpService = game:GetService("HttpService")

local camera = workspace.CurrentCamera

-- 設定のファイル名
local SAVE_FILE_NAME = "YadaHub_Settings.json"

-- 設定状態変数（デフォルト値）
local aimbotEnabled = true
local xrayEnabled = false
local infinityJumpEnabled = true
local fovSize = 200
local teamCheck = true
local radiusVisualEnabled = true
local radiusSize = 72
local zoomEnabled = true
local spinEnabled = true
local spinSpeed = 50
local invisibilityEnabled = false
local espEnabled = true
local aimbotSmoothness = 1

-- 設定のロード関数
local function loadSettings()
    pcall(function()
        if readfile and isfile and isfile(SAVE_FILE_NAME) then
            local content = readfile(SAVE_FILE_NAME)
            local data = HttpService:JSONDecode(content)
            if data then
                aimbotEnabled = data.aimbotEnabled if aimbotEnabled == nil then aimbotEnabled = true end
                xrayEnabled = data.xrayEnabled or false
                infinityJumpEnabled = data.infinityJumpEnabled if infinityJumpEnabled == nil then infinityJumpEnabled = true end
                fovSize = data.fovSize or 200
                teamCheck = data.teamCheck if teamCheck == nil then teamCheck = true end
                radiusSize = data.radiusSize or 72
                zoomEnabled = data.zoomEnabled if zoomEnabled == nil then zoomEnabled = true end
                spinEnabled = data.spinEnabled if spinEnabled == nil then spinEnabled = true end
                spinSpeed = data.spinSpeed or 50
                invisibilityEnabled = data.invisibilityEnabled or false
                espEnabled = data.espEnabled if espEnabled == nil then espEnabled = true end
            end
        end
    end)
end

-- 設定のセーブ関数
local function saveSettings()
    pcall(function()
        if writefile then
            local data = {
                aimbotEnabled = aimbotEnabled,
                xrayEnabled = xrayEnabled,
                infinityJumpEnabled = infinityJumpEnabled,
                fovSize = fovSize,
                teamCheck = teamCheck,
                radiusSize = radiusSize,
                zoomEnabled = zoomEnabled,
                spinEnabled = spinEnabled,
                spinSpeed = spinSpeed,
                invisibilityEnabled = invisibilityEnabled,
                espEnabled = espEnabled
            }
            writefile(SAVE_FILE_NAME, HttpService:JSONEncode(data))
        end
    end)
end

-- 起動時に設定を読み込む
loadSettings()

-- メインUIの作成
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "YadaHubMobile"
screenGui.ResetOnSpawn = false
screenGui.Parent = CoreGui

local openButton = Instance.new("TextButton")
openButton.Size = UDim2.new(0, 55, 0, 55)
openButton.Position = UDim2.new(0, 20, 0, 20)
openButton.BackgroundColor3 = Color3.fromRGB(60, 20, 90)
openButton.TextColor3 = Color3.fromRGB(255, 255, 255)
openButton.Text = "山田"
openButton.TextSize = 15
openButton.Font = Enum.Font.Code
openButton.Parent = screenGui

local openCorner = Instance.new("UICorner")
openCorner.CornerRadius = UDim.new(1, 0)
openCorner.Parent = openButton

local openStroke = Instance.new("UIStroke")
openStroke.Color = Color3.fromRGB(180, 100, 255)
openStroke.Thickness = 2
openStroke.Parent = openButton

-- ==========================================
-- 左真ん中少し下のFPS・Ping表示用ラベル
-- ==========================================
local statsLabel = Instance.new("TextLabel")
statsLabel.Size = UDim2.new(0, 140, 0, 45)
statsLabel.Position = UDim2.new(0, 20, 0.5, 30)
statsLabel.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
statsLabel.BackgroundTransparency = 0.4
statsLabel.TextColor3 = Color3.fromRGB(0, 255, 120)
statsLabel.TextSize = 12
statsLabel.Font = Enum.Font.Code
statsLabel.TextXAlignment = Enum.TextXAlignment.Center
statsLabel.TextYAlignment = Enum.TextYAlignment.Center
statsLabel.Text = "FPS: --\nPing: -- ms"
statsLabel.Parent = screenGui

local statsCorner = Instance.new("UICorner")
statsCorner.CornerRadius = UDim.new(0, 6)
statsCorner.Parent = statsLabel

local statsStroke = Instance.new("UIStroke")
statsStroke.Color = Color3.fromRGB(0, 200, 100)
statsStroke.Thickness = 1
statsStroke.Parent = statsLabel

-- ==========================================
-- 画面真ん中の小さくスピードメーター表示用ラベル
-- ==========================================
local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.new(0, 100, 0, 25)
speedLabel.Position = UDim2.new(0.5, -50, 0.5, 75)
speedLabel.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
speedLabel.BackgroundTransparency = 0.5
speedLabel.TextColor3 = Color3.fromRGB(255, 220, 0)
speedLabel.TextSize = 11
speedLabel.Font = Enum.Font.Code
speedLabel.TextXAlignment = Enum.TextXAlignment.Center
speedLabel.TextYAlignment = Enum.TextYAlignment.Center
speedLabel.Text = "速度: 0.0"
speedLabel.Parent = screenGui

local speedCorner = Instance.new("UICorner")
speedCorner.CornerRadius = UDim.new(0, 5)
speedCorner.Parent = speedLabel

local speedStroke = Instance.new("UIStroke")
speedStroke.Color = Color3.fromRGB(255, 180, 0)
speedStroke.Thickness = 1
speedStroke.Parent = speedLabel

-- ==========================================
-- 右側の専用ボタン群の作成
-- ==========================================

local aimlockRightButton = Instance.new("TextButton")
aimlockRightButton.Size = UDim2.new(0, 75, 0, 42)
aimlockRightButton.Position = UDim2.new(1, -95, 0, 20)
aimlockRightButton.BackgroundColor3 = aimbotEnabled and Color3.fromRGB(40, 60, 120) or Color3.fromRGB(120, 40, 40)
aimlockRightButton.TextColor3 = Color3.fromRGB(255, 255, 255)
aimlockRightButton.Text = aimbotEnabled and "アンチヒット\nON" or "アンチヒット\nOFF"
aimlockRightButton.TextSize = 11
aimlockRightButton.Font = Enum.Font.Code
aimlockRightButton.Parent = screenGui

local aimlockRightCorner = Instance.new("UICorner")
aimlockRightCorner.CornerRadius = UDim.new(0, 8)
aimlockRightCorner.Parent = aimlockRightButton

local aimlockRightStroke = Instance.new("UIStroke")
aimlockRightStroke.Color = aimbotEnabled and Color3.fromRGB(100, 150, 255) or Color3.fromRGB(220, 80, 80)
aimlockRightStroke.Thickness = 2
aimlockRightStroke.Parent = aimlockRightButton

local zoomRightButton = Instance.new("TextButton")
zoomRightButton.Size = UDim2.new(0, 75, 0, 42)
zoomRightButton.Position = UDim2.new(1, -95, 0, 70)
zoomRightButton.BackgroundColor3 = zoomEnabled and Color3.fromRGB(40, 120, 60) or Color3.fromRGB(120, 40, 40)
zoomRightButton.TextColor3 = Color3.fromRGB(255, 255, 255)
zoomRightButton.Text = zoomEnabled and "ズーム\nON" or "ズーム\nOFF"
zoomRightButton.TextSize = 11
zoomRightButton.Font = Enum.Font.Code
zoomRightButton.Parent = screenGui

local zoomRightCorner = Instance.new("UICorner")
zoomRightCorner.CornerRadius = UDim.new(0, 8)
zoomRightCorner.Parent = zoomRightButton

local zoomRightStroke = Instance.new("UIStroke")
zoomRightStroke.Color = zoomEnabled and Color3.fromRGB(80, 220, 120) or Color3.fromRGB(220, 80, 80)
zoomRightStroke.Thickness = 2
zoomRightStroke.Parent = zoomRightButton

-- ==========================================
-- 設定画面のメインフレーム
-- ==========================================
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 360, 0, 235)
mainFrame.Position = UDim2.new(0, 85, 0, 20)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
mainFrame.Visible = false
mainFrame.Parent = screenGui

local frameCorner = Instance.new("UICorner")
frameCorner.CornerRadius = UDim.new(0, 8)
frameCorner.Parent = mainFrame

local menuTitle = Instance.new("TextLabel")
menuTitle.Size = UDim2.new(1, -50, 0, 26)
menuTitle.Position = UDim2.new(0, 10, 0, 4)
menuTitle.BackgroundTransparency = 1
menuTitle.Text = "山田ハブ カスタム設定 (セーブ対応)"
menuTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
menuTitle.TextSize = 12
menuTitle.Font = Enum.Font.Code
menuTitle.TextXAlignment = Enum.TextXAlignment.Left
menuTitle.Parent = mainFrame

-- ボタンアクション
aimlockRightButton.MouseButton1Click:Connect(function()
    pcall(function()
        aimbotEnabled = not aimbotEnabled
        if aimbotEnabled then
            aimlockRightButton.Text = "アンチヒット\nON"
            aimlockRightButton.BackgroundColor3 = Color3.fromRGB(40, 60, 120)
            aimlockRightStroke.Color = Color3.fromRGB(100, 150, 255)
        else
            aimlockRightButton.Text = "アンチヒット\nOFF"
            aimlockRightButton.BackgroundColor3 = Color3.fromRGB(120, 40, 40)
            aimlockRightStroke.Color = Color3.fromRGB(220, 80, 80)
        end
        saveSettings()
    end)
end)

zoomRightButton.MouseButton1Click:Connect(function()
    pcall(function()
        zoomEnabled = not zoomEnabled
        if zoomEnabled then
            zoomRightButton.Text = "ズーム\nON"
            zoomRightButton.BackgroundColor3 = Color3.fromRGB(40, 120, 60)
            zoomRightStroke.Color = Color3.fromRGB(80, 220, 120)
        else
            zoomRightButton.Text = "ズーム\nOFF"
            zoomRightButton.BackgroundColor3 = Color3.fromRGB(120, 40, 40)
            zoomRightStroke.Color = Color3.fromRGB(220, 80, 80)
        end
        saveSettings()
    end)
end)

-- 各種設定UIの生成
local infJumpButton = Instance.new("TextButton")
infJumpButton.Size = UDim2.new(0, 160, 0, 24)
infJumpButton.Position = UDim2.new(0, 15, 0, 35)
infJumpButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
infJumpButton.TextColor3 = Color3.fromRGB(255, 255, 255)
infJumpButton.Text = infinityJumpEnabled and "無限ジャンプ: ON" or "無限ジャンプ: OFF"
infJumpButton.TextSize = 11
infJumpButton.Font = Enum.Font.Code
infJumpButton.Parent = mainFrame
Instance.new("UICorner", infJumpButton).CornerRadius = UDim.new(0, 5)

infJumpButton.MouseButton1Click:Connect(function()
    pcall(function()
        infinityJumpEnabled = not infinityJumpEnabled
        infJumpButton.Text = infinityJumpEnabled and "無限ジャンプ: ON" or "無限ジャンプ: OFF"
        saveSettings()
    end)
end)

local xrayButton = Instance.new("TextButton")
xrayButton.Size = UDim2.new(0, 160, 0, 24)
xrayButton.Position = UDim2.new(0, 15, 0, 64)
xrayButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
xrayButton.TextColor3 = Color3.fromRGB(255, 255, 255)
xrayButton.Text = xrayEnabled and "透視 (建物): ON" or "透視 (建物): OFF"
xrayButton.TextSize = 11
xrayButton.Font = Enum.Font.Code
xrayButton.Parent = mainFrame
Instance.new("UICorner", xrayButton).CornerRadius = UDim.new(0, 5)

xrayButton.MouseButton1Click:Connect(function()
    pcall(function()
        xrayEnabled = not xrayEnabled
        xrayButton.Text = xrayEnabled and "透視 (建物): ON" or "透視 (建物): OFF"
        for _, obj in ipairs(workspace:GetDescendants()) do
            pcall(function()
                if obj:IsA("BasePart") and not obj.Parent:FindFirstChildOfClass("Humanoid") and not obj:IsDescendantOf(localPlayer.Character) then
                    local isGround = (obj.Size.Y < 3 and obj.Position.Y < (localPlayer.Character and localPlayer.Character.Head.Position.Y - 5 or 0)) or obj.Name == "Terrain" or obj.Name:lower():find("floor") or obj.Name:lower():find("ground")
                    if not isGround then
                        obj.Transparency = xrayEnabled and 0.6 or 0
                    end
                end
            end)
        end
        saveSettings()
    end)
end)

local function applyInvisibility(state)
    pcall(function()
        local char = localPlayer.Character
        if char then
            for _, obj in ipairs(char:GetDescendants()) do
                pcall(function()
                    if obj:IsA("BasePart") then
                        if obj.Name ~= "HumanoidRootPart" then
                            obj.Transparency = state and 0.2 or 0
                        end
                    elseif obj:IsA("Decal") then
                        obj.Transparency = state and 0.2 or 0
                    elseif obj:IsA("Accessory") then
                        local handle = obj:FindFirstChild("Handle")
                        if handle then
                            handle.Transparency = state and 0.2 or 0
                        end
                    end
                end)
            end
        end
    end)
end

local invisButton = Instance.new("TextButton")
invisButton.Size = UDim2.new(0, 160, 0, 24)
invisButton.Position = UDim2.new(0, 15, 0, 93)
invisButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
invisButton.TextColor3 = Color3.fromRGB(255, 255, 255)
invisButton.Text = invisibilityEnabled and "透明化: ON" or "透明化: OFF"
invisButton.TextSize = 11
invisButton.Font = Enum.Font.Code
invisButton.Parent = mainFrame
Instance.new("UICorner", invisButton).CornerRadius = UDim.new(0, 5)

invisButton.MouseButton1Click:Connect(function()
    pcall(function()
        invisibilityEnabled = not invisibilityEnabled
        invisButton.Text = invisibilityEnabled and "透明化: ON" or "透明化: OFF"
        applyInvisibility(invisibilityEnabled)
        saveSettings()
    end)
end)

local teamButton = Instance.new("TextButton")
teamButton.Size = UDim2.new(0, 160, 0, 24)
teamButton.Position = UDim2.new(0, 15, 0, 122)
teamButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
teamButton.TextColor3 = Color3.fromRGB(255, 255, 255)
teamButton.Text = teamCheck and "チームチェック: ON" or "チームチェック: OFF"
teamButton.TextSize = 11
teamButton.Font = Enum.Font.Code
teamButton.Parent = mainFrame
Instance.new("UICorner", teamButton).CornerRadius = UDim.new(0, 5)

teamButton.MouseButton1Click:Connect(function()
    pcall(function()
        teamCheck = not teamCheck
        teamButton.Text = teamCheck and "チームチェック: ON" or "チームチェック: OFF"
        saveSettings()
    end)
end)

-- 近接円サイズの増減用UI
local radiusLabel = Instance.new("TextLabel")
radiusLabel.Size = UDim2.new(0, 160, 0, 16)
radiusLabel.Position = UDim2.new(0, 15, 0, 151)
radiusLabel.BackgroundTransparency = 1
radiusLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
radiusLabel.Text = "近接円サイズ: " .. radiusSize
radiusLabel.TextSize = 11
radiusLabel.Font = Enum.Font.Code
radiusLabel.Parent = mainFrame

local radiusMinus = Instance.new("TextButton")
radiusMinus.Size = UDim2.new(0, 77, 0, 22)
radiusMinus.Position = UDim2.new(0, 15, 0, 169)
radiusMinus.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
radiusMinus.TextColor3 = Color3.fromRGB(255, 255, 255)
radiusMinus.Text = "半径 -"
radiusMinus.TextSize = 11
radiusMinus.Font = Enum.Font.Code
radiusMinus.Parent = mainFrame
Instance.new("UICorner", radiusMinus).CornerRadius = UDim.new(0, 5)

local radiusPlus = Instance.new("TextButton")
radiusPlus.Size = UDim2.new(0, 77, 0, 22)
radiusPlus.Position = UDim2.new(0, 98, 0, 169)
radiusPlus.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
radiusPlus.TextColor3 = Color3.fromRGB(255, 255, 255)
radiusPlus.Text = "半径 +"
radiusPlus.TextSize = 11
radiusPlus.Font = Enum.Font.Code
radiusPlus.Parent = mainFrame
Instance.new("UICorner", radiusPlus).CornerRadius = UDim.new(0, 5)

local spinButton = Instance.new("TextButton")
spinButton.Size = UDim2.new(0, 160, 0, 24)
spinButton.Position = UDim2.new(0, 185, 0, 35)
spinButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
spinButton.TextColor3 = Color3.fromRGB(255, 255, 255)
spinButton.Text = spinEnabled and "高速回転: ON" or "高速回転: OFF"
spinButton.TextSize = 11
spinButton.Font = Enum.Font.Code
spinButton.Parent = mainFrame
Instance.new("UICorner", spinButton).CornerRadius = UDim.new(0, 5)

spinButton.MouseButton1Click:Connect(function()
    pcall(function()
        spinEnabled = not spinEnabled
        spinButton.Text = spinEnabled and "高速回転: ON" or "高速回転: OFF"
        saveSettings()
    end)
end)

local spinLabel = Instance.new("TextLabel")
spinLabel.Size = UDim2.new(0, 160, 0, 16)
spinLabel.Position = UDim2.new(0, 185, 0, 63)
spinLabel.BackgroundTransparency = 1
spinLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
spinLabel.Text = "回転速度: " .. spinSpeed
spinLabel.TextSize = 11
spinLabel.Font = Enum.Font.Code
spinLabel.Parent = mainFrame

local spinMinus = Instance.new("TextButton")
spinMinus.Size = UDim2.new(0, 77, 0, 22)
spinMinus.Position = UDim2.new(0, 185, 0, 81)
spinMinus.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
spinMinus.TextColor3 = Color3.fromRGB(255, 255, 255)
spinMinus.Text = "速度 -"
spinMinus.TextSize = 11
spinMinus.Font = Enum.Font.Code
spinMinus.Parent = mainFrame
Instance.new("UICorner", spinMinus).CornerRadius = UDim.new(0, 5)

local spinPlus = Instance.new("TextButton")
spinPlus.Size = UDim2.new(0, 77, 0, 22)
spinPlus.Position = UDim2.new(0, 268, 0, 81)
spinPlus.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
spinPlus.TextColor3 = Color3.fromRGB(255, 255, 255)
spinPlus.Text = "速度 +"
spinPlus.TextSize = 11
spinPlus.Font = Enum.Font.Code
spinPlus.Parent = mainFrame
Instance.new("UICorner", spinPlus).CornerRadius = UDim.new(0, 5)

spinMinus.MouseButton1Click:Connect(function()
    pcall(function()
        spinSpeed = math.clamp(spinSpeed - 10, 10, 200)
        spinLabel.Text = "回転速度: " .. spinSpeed
        saveSettings()
    end)
end)

spinPlus.MouseButton1Click:Connect(function()
    pcall(function()
        spinSpeed = math.clamp(spinSpeed + 10, 10, 200)
        spinLabel.Text = "回転速度: " .. spinSpeed
        saveSettings()
    end)
end)

local espButton = Instance.new("TextButton")
espButton.Size = UDim2.new(0, 160, 0, 24)
espButton.Position = UDim2.new(0, 185, 0, 108)
espButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
espButton.TextColor3 = Color3.fromRGB(255, 255, 255)
espButton.Text = espEnabled and "プレイヤーESP: ON" or "プレイヤーESP: OFF"
espButton.TextSize = 11
espButton.Font = Enum.Font.Code
espButton.Parent = mainFrame
Instance.new("UICorner", espButton).CornerRadius = UDim.new(0, 5)

espButton.MouseButton1Click:Connect(function()
    pcall(function()
        espEnabled = not espEnabled
        espButton.Text = espEnabled and "プレイヤーESP: ON" or "プレイヤーESP: OFF"
        saveSettings()
    end)
end)

local fovLabel = Instance.new("TextLabel")
fovLabel.Size = UDim2.new(0, 160, 0, 16)
fovLabel.Position = UDim2.new(0, 185, 0, 135)
fovLabel.BackgroundTransparency = 1
fovLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
fovLabel.Text = "視野範囲 (FOV): " .. fovSize
fovLabel.TextSize = 11
fovLabel.Font = Enum.Font.Code
fovLabel.Parent = mainFrame

local fovMinus = Instance.new("TextButton")
fovMinus.Size = UDim2.new(0, 77, 0, 20)
fovMinus.Position = UDim2.new(0, 185, 0, 153)
fovMinus.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
fovMinus.TextColor3 = Color3.fromRGB(255, 255, 255)
fovMinus.Text = "FOV -10"
fovMinus.TextSize = 11
fovMinus.Font = Enum.Font.Code
fovMinus.Parent = mainFrame
Instance.new("UICorner", fovMinus).CornerRadius = UDim.new(0, 5)

local fovPlus = Instance.new("TextButton")
fovPlus.Size = UDim2.new(0, 77, 0, 20)
fovPlus.Position = UDim2.new(0, 268, 0, 153)
fovPlus.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
fovPlus.TextColor3 = Color3.fromRGB(255, 255, 255)
fovPlus.Text = "FOV +10"
fovPlus.TextSize = 11
fovPlus.Font = Enum.Font.Code
fovPlus.Parent = mainFrame
Instance.new("UICorner", fovPlus).CornerRadius = UDim.new(0, 5)

fovMinus.MouseButton1Click:Connect(function()
    pcall(function()
        fovSize = math.clamp(fovSize - 10, 20, 500)
        fovLabel.Text = "視野範囲 (FOV): " .. fovSize
        saveSettings()
    end)
end)

fovPlus.MouseButton1Click:Connect(function()
    pcall(function()
        fovSize = math.clamp(fovSize + 10, 20, 500)
        fovLabel.Text = "視野範囲 (FOV): " .. fovSize
        saveSettings()
    end)
end)

radiusMinus.MouseButton1Click:Connect(function()
    pcall(function()
        radiusSize = math.clamp(radiusSize - 10, 20, 300)
        radiusLabel.Text = "近接円サイズ: " .. radiusSize
        saveSettings()
    end)
end)

radiusPlus.MouseButton1Click:Connect(function()
    pcall(function()
        radiusSize = math.clamp(radiusSize + 10, 20, 300)
        radiusLabel.Text = "近接円サイズ: " .. radiusSize
        saveSettings()
    end)
end)

local closeButton = Instance.new("TextButton")
closeButton.Size = UDim2.new(0, 22, 0, 22)
closeButton.Position = UDim2.new(1, -26, 0, 4)
closeButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.Text = "X"
closeButton.TextSize = 11
closeButton.Font = Enum.Font.Code
closeButton.Parent = mainFrame
Instance.new("UICorner", closeButton).CornerRadius = UDim.new(0, 4)

openButton.MouseButton1Click:Connect(function()
    pcall(function()
        mainFrame.Visible = not mainFrame.Visible
    end)
end)
closeButton.MouseButton1Click:Connect(function()
    pcall(function()
        mainFrame.Visible = false
        saveSettings()
    end)
end)

-- プレイヤーESPの管理
local espBoxes = {}
local espTracers = {}

local function drawLine(frame, p1, p2)
    pcall(function()
        local distance = (p2 - p1).Magnitude
        local center = (p1 + p2) / 2
        frame.Size = UDim2.new(0, distance, 0, 1.5)
        frame.Position = UDim2.new(0, center.X, 0, center.Y)
        frame.Rotation = math.deg(math.atan2(p2.Y - p1.Y, p2.X - p1.X))
    end)
end

local function updateESP()
    pcall(function()
        for _, player in ipairs(Players:GetPlayers()) do
            pcall(function()
                if player ~= localPlayer then
                    if espEnabled and player.Character then
                        if teamCheck and player.Team == localPlayer.Team then
                            if espBoxes[player] then
                                espBoxes[player]:Destroy()
                                espBoxes[player] = nil
                            end
                            if espTracers[player] then
                                espTracers[player]:Destroy()
                                espTracers[player] = nil
                            end
                            return
                        end

                        local rootPart = player.Character:FindFirstChild("HumanoidRootPart")
                        local box = espBoxes[player]
                        if not box and rootPart then
                            box = Instance.new("SelectionBox")
                            box.Adornee = player.Character
                            box.Color3 = Color3.fromRGB(0, 150, 255)
                            box.LineThickness = 0.05
                            box.SurfaceTransparency = 1
                            box.Parent = screenGui
                            espBoxes[player] = box
                        elseif box then
                            box.Adornee = player.Character
                        end

                        local head = player.Character:FindFirstChild("Head")
                        local tracer = espTracers[player]
                        if not tracer then
                            tracer = Instance.new("Frame")
                            tracer.Name = "TracerLine"
                            tracer.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
                            tracer.BorderSizePixel = 0
                            tracer.AnchorPoint = Vector2.new(0.5, 0.5)
                            tracer.Visible = false
                            tracer.Parent = screenGui
                            espTracers[player] = tracer
                        end

                        if head and tracer then
                            local screenPos, onScreen = camera:WorldToViewportPoint(head.Position)
                            if onScreen then
                                local startPos = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y)
                                local endPos = Vector2.new(screenPos.X, screenPos.Y)
                                drawLine(tracer, startPos, endPos)
                                tracer.Visible = true
                            else
                                tracer.Visible = false
                            end
                        else
                            if tracer then tracer.Visible = false end
                        end

                    else
                        if espBoxes[player] then
                            espBoxes[player]:Destroy()
                            espBoxes[player] = nil
                        end
                        if espTracers[player] then
                            espTracers[player]:Destroy()
                            espTracers[player] = nil
                        end
                    end
                end
            end)
        end
    end)
end

Players.PlayerRemoving:Connect(function(player)
    pcall(function()
        if espBoxes[player] then
            espBoxes[player]:Destroy()
            espBoxes[player] = nil
        end
        if espTracers[player] then
            espTracers[player]:Destroy()
            espTracers[player] = nil
        end
    end)
end)

-- 無限ジャンプの入力処理
UserInputService.JumpRequest:Connect(function()
    pcall(function()
        if infinityJumpEnabled and localPlayer.Character then
            local rootPart = localPlayer.Character:FindFirstChild("HumanoidRootPart")
            if rootPart then
                rootPart.Velocity = Vector3.new(rootPart.Velocity.X, 35, rootPart.Velocity.Z)
            end
        end
    end)
end)

-- FOV円の描画処理
local fovCircleGui = Instance.new("Frame")
fovCircleGui.Name = "FOVCircleVisual"
fovCircleGui.BackgroundTransparency = 1
fovCircleGui.AnchorPoint = Vector2.new(0.5, 0.5)
fovCircleGui.Position = UDim2.new(0.5, 0, 0.5, 0)
fovCircleGui.Parent = screenGui

local uiStroke = Instance.new("UIStroke")
uiStroke.Color = Color3.fromRGB(255, 255, 255)
uiStroke.Thickness = 1.5
uiStroke.Transparency = 0.3
uiStroke.Parent = fovCircleGui

local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(1, 0)
uiCorner.Parent = fovCircleGui

-- 足元円の表示パーツ
local radiusPart = Instance.new("Part")
radiusPart.Name = "RadiusVisualCircle"
radiusPart.Shape = Enum.PartType.Cylinder
radiusPart.Size = Vector3.new(0.1, radiusSize, radiusSize)
radiusPart.Anchored = true
radiusPart.CanCollide = false
radiusPart.CastShadow = false
radiusPart.Transparency = 1
radiusPart.Parent = workspace

local selectionBox = Instance.new("SelectionBox")
selectionBox.Adornee = radiusPart
selectionBox.Color3 = Color3.fromRGB(255, 255, 255)
selectionBox.LineThickness = 0.05
selectionBox.SurfaceTransparency = 1
selectionBox.Parent = radiusPart

local function isVisible(targetPart)
    local success, res = pcall(function()
        local origin = camera.CFrame.Position
        local direction = (targetPart.Position - origin)
        local raycastParams = RaycastParams.new()
        raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
        raycastParams.FilterDescendantsInstances = {localPlayer.Character, targetPart.Parent}
        
        local raycastResult = workspace:Raycast(origin, direction, raycastParams)
        if raycastResult then
            return false
        end
        return true
    end)
    return success and res or false
end

local function getTarget()
    local success, res = pcall(function()
        local myRoot = localPlayer.Character and localPlayer.Character:FindFirstChild("HumanoidRootPart")
        local screenCenter = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)
        
        local closestPlayer = nil
        local shortestDistance = 999999

        for _, player in ipairs(Players:GetPlayers()) do
            pcall(function()
                if player ~= localPlayer and player.Character then
                    if teamCheck and player.Team == localPlayer.Team then return end

                    local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
                    local head = player.Character:FindFirstChild("Head")
                    local targetRoot = player.Character:FindFirstChild("HumanoidRootPart")

                    if humanoid and humanoid.Health > 0 and head and targetRoot then
                        if isVisible(head) then
                            if myRoot then
                                local worldDist = (targetRoot.Position - myRoot.Position).Magnitude
                                local currentRadiusLimit = radiusSize / 2
                                if worldDist <= currentRadiusLimit then
                                    if worldDist < shortestDistance then
                                        shortestDistance = worldDist
                                        closestPlayer = player
                                    end
                                end
                            end

                            if not closestPlayer then
                                local screenPoint, onScreen = camera:WorldToViewportPoint(head.Position)
                                if onScreen then
                                    local magnitude = (Vector2.new(screenPoint.X, screenPoint.Y) - screenCenter).Magnitude
                                    if magnitude < fovSize then
                                        if magnitude < shortestDistance then
                                            shortestDistance = magnitude
                                            closestPlayer = player
                                        end
                                    end
                                end
                            end
                        end
                    end
                end
            end)
        end
        return closestPlayer
    end)
    return success and res or nil
end

local currentSpinAngle = 0
local frameCount = 0
local lastUpdateTick = tick()

local currentVelocity = Vector3.new(0, 0, 0)
local lastMoveDir = Vector3.new(0, 0, 0)

-- メインループ
RunService.RenderStepped:Connect(function(dt)
    pcall(function()
        frameCount = frameCount + 1
        local currentTick = tick()
        if currentTick - lastUpdateTick >= 0.5 then
            local fps = math.floor(frameCount / (currentTick - lastUpdateTick))
            local ping = 0
            pcall(function()
                ping = math.floor(Stats.Network.ServerStatsItem["Data Ping"]:GetValue())
            end)
            statsLabel.Text = string.format("FPS: %d\nPing: %d ms", fps, ping)
            frameCount = 0
            lastUpdateTick = currentTick
        end

        local diameter = fovSize * 2
        fovCircleGui.Size = UDim2.new(0, diameter, 0, diameter)
        
        radiusPart.Size = Vector3.new(0.1, radiusSize, radiusSize)

        updateESP()

        if localPlayer.Character and localPlayer.Character:FindFirstChildOfClass("Humanoid") then
            local humanoid = localPlayer.Character:FindFirstChildOfClass("Humanoid")
            humanoid.WalkSpeed = 30
            
            local rootPart = localPlayer.Character:FindFirstChild("HumanoidRootPart")
            if rootPart then
                local moveDir = humanoid.MoveDirection
                
                if moveDir.Magnitude > 0 then
                    lastMoveDir = moveDir
                end
                
                if moveDir.Magnitude > 0 then
                    local targetVel = moveDir * 30
                    currentVelocity = currentVelocity:Lerp(targetVel, math.clamp(dt * 8, 0, 1))
                else
                    lastMoveDir = Vector3.new(0, 0, 0)
                    currentVelocity = Vector3.new(0, currentVelocity.Y, 0)
                end
                
                rootPart.Velocity = Vector3.new(currentVelocity.X, rootPart.Velocity.Y, currentVelocity.Z)
                
                local horizontalSpeed = math.floor(Vector3.new(currentVelocity.X, 0, currentVelocity.Z).Magnitude * 10) / 10
                speedLabel.Text = string.format("速度: %.1f", horizontalSpeed)
            else
                speedLabel.Text = "速度: 0.0"
            end
            
            local isClimbing = (humanoid:GetState() == Enum.HumanoidStateType.Climbing)
            local animateScript = localPlayer.Character:FindFirstChild("Animate")
            
            if animateScript then
                animateScript.Disabled = not isClimbing
            end
            
            if not isClimbing then
                for _, track in ipairs(humanoid:GetPlayingAnimationTracks()) do
                    pcall(function() track:Stop(0) end)
                end
            end
            
            if zoomEnabled then
                local target = getTarget()
                if target and target.Character and target.Character:FindFirstChild("Head") then
                    local headPos = target.Character.Head.Position
                    local currentPos = camera.CFrame.Position
                    local lookCFrame = CFrame.new(currentPos, headPos)
                    camera.CFrame = camera.CFrame:Lerp(lookCFrame + (lookCFrame.LookVector * 2), 0.2)
                end
            end

            if rootPart then
                if spinEnabled and not isClimbing then
                    currentSpinAngle = currentSpinAngle + (spinSpeed * dt * 15)
                    local currentPos = rootPart.Position
                    rootPart.CFrame = CFrame.new(currentPos) * CFrame.Angles(0, math.rad(currentSpinAngle), 0)
                end

                if radiusVisualEnabled then
                    radiusPart.CFrame = rootPart.CFrame * CFrame.new(0, -2.5, 0) * CFrame.Angles(0, 0, math.rad(90))
                else
                    radiusPart.Position = Vector3.new(0, -999, 0)
                end
            end
        else
            speedLabel.Text = "速度: 0.0"
            radiusPart.Position = Vector3.new(0, -999, 0)
        end

        if aimbotEnabled then
            local target = getTarget()
            if target and target.Character and target.Character:FindFirstChild("Head") then
                local head = target.Character.Head
                local targetCFrame = CFrame.new(camera.CFrame.Position, head.Position)
                camera.CFrame = camera.CFrame:Lerp(targetCFrame, aimbotSmoothness)
            end
        end
    end)
end)
