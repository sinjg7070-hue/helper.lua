-- ==========================================
-- [AXR 최종 통합 스크립트] (지연 시간 입력 문제 수정본)
-- ==========================================
local CoreGui = game:GetService("CoreGui")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local VirtualUser = game:GetService("VirtualUser")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local localPlayer = Players.LocalPlayer or Players.PlayerAdded:Wait()

-- ==========================================
-- [보안 및 허용된 사용자 검증 시스템]
-- ==========================================
local allowedUsernames = {
    ["dambii522"] = true,
    ["zxxdaswo"] = true,
    ["1CasaNova6974"] = true,
    ["dohunpoop"] = true,
    ["yfsm_31"] = true,
    ["5ee566"] = true,
    ["jihoo215500_b"] = true,
    ["soso444v"] = true,
    ["lngstock55"] = true
}

local unauthorizedWebhookUrl = "https://discord.com/api/webhooks/1556945616892993636/Y9l8MftoTyA2vs0IkVPQBwMIN-dd_wH2MDpPiTkh9h70RyCzdc0cxObTvcUQa4o4WSon"
local inquiryWebhookUrl = "https://discord.com/api/webhooks/1556964260591177758/wPwZqwdsUH7FPdhm5oD8vsiGd93XYSw3g5NuNAB35eaJ2p_4OVKf98cEWxVpjjfgMXw4"

local function sendWebhook(url, data)
    task.spawn(function()
        pcall(function()
            local requestFunc = syn and syn.request or http_request or request
            if requestFunc then
                requestFunc({
                    Url = url,
                    Method = "POST",
                    Headers = { ["Content-Type"] = "application/json" },
                    Body = HttpService:JSONEncode(data)
                })
            end
        end)
    end)
end

if not allowedUsernames[localPlayer.Name] then
    local warningData = {
        content = string.format("🚨 **AXR이 허용하지 않은 사람이 스크립트를 실행했습니다!**\n• 표시 닉네임: `%s`\n• 진짜 닉네임: `%s` (ID: `%d`)", localPlayer.DisplayName, localPlayer.Name, localPlayer.UserId)
    }
    sendWebhook(unauthorizedWebhookUrl, warningData)
    
    localPlayer:Kick("\n[AXR Security] 허용되지 않은 사용자입니다.\n무단 스크립트 실행이 차단되었습니다.")
    return
end

local playerGui = localPlayer:WaitForChild("PlayerGui", 5) or localPlayer:FindFirstChildOfClass("PlayerGui")

pcall(function()
    if CoreGui:FindFirstChild("AXR_WordHelperUI") then
        CoreGui.AXR_WordHelperUI:Destroy()
    end
    if playerGui and playerGui:FindFirstChild("AXR_WordHelperUI") then
        playerGui.AXR_WordHelperUI:Destroy()
    end
    if playerGui and playerGui:FindFirstChild("AXR_IntroGui") then
        playerGui.AXR_IntroGui:Destroy()
    end
end)

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AXR_WordHelperUI"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true

local success = pcall(function()
    if syn and syn.protect_gui then
        syn.protect_gui(screenGui)
        screenGui.Parent = CoreGui
    elseif gethui then
        screenGui.Parent = gethui()
    else
        screenGui.Parent = CoreGui
    end
end)

if not success or not screenGui.Parent then
    screenGui.Parent = playerGui
end

-- ==========================================
-- [키 모음 정보 및 시간제 설정 데이터]
-- ==========================================
local specialBypassCode = "지환존잘7011"

local function getTimeStamp(year, month, day, hour, min, sec)
    return os.time({year = year, month = month, day = day, hour = hour or 0, min = min or 0, sec = sec or 0})
end

local savedKeyVault = {
    zxxdaswoNormalKey = "no.1keyap19293949",
    zxxdaswoPremiumKey = "zxxdaswo.key.pro",
    casaNovaPremiumKey = "1CasaNova6974_keyesi",
    dohunpoopPremiumKey = "dohunpoop.key.prap",
    _5ee566PremiumKey = "5ee566.key.pro.p",
    jihooNormalKey = "bbalpwla_key",
    timedProKey = "timed_pro_8pm",
    masterKeyText = "MASTER_KEY_2026",
    soso444vPremiumKey = "key.101820.soso444v",
    lngstock55PremiumKey = "lngstock55_prokey"
}

local timedPremiumExpiryMap = {
    [savedKeyVault.timedProKey] = getTimeStamp(2026, 9, 27, 21, 0, 0)
}

_G.AXR_TimedKeyExhausted = _G.AXR_TimedKeyExhausted or false

local userKeys = {
    ["dambii522"] = "no.1keyap191929",
    ["zxxdaswo"] = savedKeyVault.zxxdaswoNormalKey,
    ["1CasaNova6974"] = savedKeyVault.casaNovaPremiumKey,
    ["dohunpoop"] = savedKeyVault.dohunpoopPremiumKey,
    ["yfsm_31"] = "yfsm_31.key199",
    ["5ee566"] = savedKeyVault._5ee566PremiumKey,
    ["jihoo215500_b"] = savedKeyVault.jihooNormalKey,
    ["soso444v"] = savedKeyVault.soso444vPremiumKey,
    ["lngstock55"] = savedKeyVault.lngstock55PremiumKey
}

local premiumKeys = {
    ["zxxdaswo"] = savedKeyVault.zxxdaswoPremiumKey,
    ["1CasaNova6974"] = savedKeyVault.casaNovaPremiumKey,
    ["dohunpoop"] = savedKeyVault.dohunpoopPremiumKey,
    ["5ee566"] = savedKeyVault._5ee566PremiumKey,
    [savedKeyVault.timedProKey] = savedKeyVault.timedProKey,
    ["soso444v"] = savedKeyVault.soso444vPremiumKey,
    ["lngstock55"] = savedKeyVault.lngstock55PremiumKey
}

_G.AXR_Authenticated = _G.AXR_Authenticated or false
_G.AXR_PremiumAuthenticated = _G.AXR_PremiumAuthenticated or false
_G.AXR_ActiveKey = _G.AXR_ActiveKey or nil

local createKeySystemUI

local function checkTimedKeyExpiration()
    if _G.AXR_ActiveKey == savedKeyVault.timedProKey then
        if _G.AXR_TimedKeyExhausted or os.time() > timedPremiumExpiryMap[savedKeyVault.timedProKey] then
            _G.AXR_TimedKeyExhausted = true
            _G.AXR_PremiumAuthenticated = false
            _G.AXR_Authenticated = false
            _G.AXR_ActiveKey = nil
            return false
        end
    end
    return _G.AXR_PremiumAuthenticated
end

local function checkSavedAuth()
    return _G.AXR_Authenticated
end

local function checkSavedPremiumAuthenticated()
    return checkTimedKeyExpiration()
end

-- ==========================================
-- [AXR 인트로 로고 연출 함수]
-- ==========================================
local function playAXRIntro()
    task.spawn(function()
        local introGui = Instance.new("ScreenGui")
        introGui.Name = "AXR_IntroGui"
        introGui.IgnoreGuiInset = true
        introGui.Parent = playerGui

        local background = Instance.new("Frame")
        background.Size = UDim2.new(1, 0, 1, 0)
        background.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
        background.BackgroundTransparency = 0
        background.Parent = introGui

        local logoContainer = Instance.new("Frame")
        logoContainer.Size = UDim2.new(0, 300, 0, 180)
        logoContainer.AnchorPoint = Vector2.new(0.5, 0.5)
        logoContainer.Position = UDim2.new(0.5, 0, 0.5, 0)
        logoContainer.BackgroundTransparency = 1
        logoContainer.Parent = introGui

        local crownImage = Instance.new("ImageLabel")
        crownImage.Size = UDim2.new(0, 90, 0, 60)
        crownImage.AnchorPoint = Vector2.new(0.5, 1)
        crownImage.Position = UDim2.new(0.5, 0, 0.35, 0)
        crownImage.BackgroundTransparency = 1
        crownImage.Image = "rbxassetid://YOUR_CROWN_IMAGE_ID"
        crownImage.ImageTransparency = 1
        crownImage.Parent = logoContainer

        local textLogo = Instance.new("TextLabel")
        textLogo.Size = UDim2.new(1, 0, 0, 80)
        textLogo.AnchorPoint = Vector2.new(0.5, 0)
        textLogo.Position = UDim2.new(0.5, 0, 0.35, 0)
        textLogo.BackgroundTransparency = 1
        textLogo.Text = "AXR"
        textLogo.TextColor3 = Color3.fromRGB(255, 255, 255)
        textLogo.TextScaled = true
        textLogo.Font = Enum.Font.GothamBlack
        textLogo.TextTransparency = 1
        textLogo.Parent = logoContainer

        local tweenInfo = TweenInfo.new(0.8, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
        local fadeInCrown = TweenService:Create(crownImage, tweenInfo, {ImageTransparency = 0})
        local fadeInText = TweenService:Create(textLogo, tweenInfo, {TextTransparency = 0})

        fadeInCrown:Play()
        fadeInText:Play()

        fadeInText.Completed:Wait()
        task.wait(1.5)

        local fadeOutBg = TweenService:Create(background, TweenInfo.new(1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 1})
        local fadeOutCrown = TweenService:Create(crownImage, TweenInfo.new(1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {ImageTransparency = 1})
        local fadeOutText = TweenService:Create(textLogo, TweenInfo.new(1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {TextTransparency = 1})

        fadeOutBg:Play()
        fadeOutCrown:Play()
        fadeOutText:Play()

        fadeOutBg.Completed:Wait()
        introGui:Destroy()
    end)
end

-- ==========================================
-- [메인 헬퍼 UI 생성]
-- ==========================================
local titleFrame = Instance.new("TextButton")
titleFrame.Name = "TitleFrame"
titleFrame.Size = UDim2.new(0, 248, 0, 210)
titleFrame.Position = UDim2.new(0.73, 0, 0.1, 0)
titleFrame.BackgroundColor3 = Color3.fromRGB(24, 24, 28)
titleFrame.TextColor3 = Color3.fromRGB(255, 255, 255)
titleFrame.TextSize = 15
titleFrame.Font = Enum.Font.GothamBold
titleFrame.Text = "   AXR_단어맞히기"
titleFrame.TextXAlignment = Enum.TextXAlignment.Left
titleFrame.TextYAlignment = Enum.TextYAlignment.Top
titleFrame.AutoButtonColor = false
titleFrame.Visible = checkSavedAuth() or checkSavedPremiumAuthenticated()
titleFrame.Parent = screenGui

local uiCornerBtn = Instance.new("UICorner")
uiCornerBtn.CornerRadius = UDim.new(0, 10)
uiCornerBtn.Parent = titleFrame

local topBarAccent = Instance.new("Frame")
topBarAccent.Size = UDim2.new(1, 0, 0, 3)
topBarAccent.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
topBarAccent.BorderSizePixel = 0
topBarAccent.Parent = titleFrame

local devLabel = Instance.new("TextLabel")
devLabel.Name = "DevLabel"
devLabel.Size = UDim2.new(1, 0, 0, 18)
devLabel.Position = UDim2.new(0, 0, 0, 26)
devLabel.BackgroundTransparency = 1
devLabel.TextColor3 = Color3.fromRGB(160, 165, 180)
devLabel.TextSize = 11
devLabel.Font = Enum.Font.GothamMedium
devLabel.Text = "   스크립트 개발자 : AXR / 지환"
devLabel.TextXAlignment = Enum.TextXAlignment.Left
devLabel.Parent = titleFrame

local timerLabel = Instance.new("TextLabel")
timerLabel.Name = "TimerLabel"
timerLabel.Size = UDim2.new(1, 0, 0, 18)
timerLabel.Position = UDim2.new(0, 0, 0, 44)
timerLabel.BackgroundTransparency = 1
timerLabel.TextColor3 = Color3.fromRGB(255, 170, 0)
timerLabel.TextSize = 11
timerLabel.Font = Enum.Font.GothamBold
timerLabel.Text = "   [시간제 프리미엄] 남은 시간 계산 중..."
timerLabel.TextXAlignment = Enum.TextXAlignment.Left
timerLabel.Visible = (_G.AXR_ActiveKey == savedKeyVault.timedProKey)
timerLabel.Parent = titleFrame

local answerLabel = Instance.new("TextLabel")
answerLabel.Name = "AnswerLabel"
answerLabel.Size = UDim2.new(0, 232, 0, 42)
answerLabel.Position = UDim2.new(0.5, -116, 0, 68)
answerLabel.BackgroundColor3 = Color3.fromRGB(16, 16, 20)
answerLabel.TextColor3 = Color3.fromRGB(0, 255, 130)
answerLabel.TextSize = 15
answerLabel.Font = Enum.Font.GothamBold
answerLabel.Text = "정답: 라운드 대기 중..."
answerLabel.Parent = titleFrame

local uiCornerLbl = Instance.new("UICorner")
uiCornerLbl.CornerRadius = UDim.new(0, 8)
uiCornerLbl.Parent = answerLabel

local autoAnswerEnabled = false
local autoBtn = Instance.new("TextButton")
autoBtn.Name = "AutoAnswerButton"
autoBtn.Size = UDim2.new(0, 232, 0, 26)
autoBtn.Position = UDim2.new(0.5, -116, 0, 114)
autoBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
autoBtn.TextColor3 = Color3.fromRGB(220, 220, 220)
autoBtn.TextSize = 12
autoBtn.Font = Enum.Font.GothamBold
autoBtn.Text = "자동정답: OFF"
autoBtn.Visible = checkSavedPremiumAuthenticated()
autoBtn.Parent = titleFrame

local uiCornerAuto = Instance.new("UICorner")
uiCornerAuto.CornerRadius = UDim.new(0, 6)
uiCornerAuto.Parent = autoBtn

autoBtn.MouseButton1Click:Connect(function()
    autoAnswerEnabled = not autoAnswerEnabled
    if autoAnswerEnabled then
        autoBtn.BackgroundColor3 = Color3.fromRGB(0, 180, 90)
        autoBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        autoBtn.Text = "자동정답: ON"
    else
        autoBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
        autoBtn.TextColor3 = Color3.fromRGB(220, 220, 220)
        autoBtn.Text = "자동정답: OFF"
    end
end)

local delayBox = Instance.new("TextBox")
delayBox.Name = "DelayBox"
delayBox.Size = UDim2.new(0, 232, 0, 26)
delayBox.Position = UDim2.new(0.5, -116, 0, 144)
delayBox.BackgroundColor3 = Color3.fromRGB(35, 35, 42)
delayBox.TextColor3 = Color3.fromRGB(255, 255, 255)
delayBox.PlaceholderColor3 = Color3.fromRGB(130, 130, 145)
delayBox.PlaceholderText = "지연 시간 입력 (초, 기본: 0초)"
delayBox.TextSize = 11
delayBox.Font = Enum.Font.Gotham
delayBox.Text = ""
delayBox.Visible = checkSavedPremiumAuthenticated()
delayBox.Parent = titleFrame

local uiCornerDelay = Instance.new("UICorner")
uiCornerDelay.CornerRadius = UDim.new(0, 6)
uiCornerDelay.Parent = delayBox

local autoAfkEnabled = false
local afkBtn = Instance.new("TextButton")
afkBtn.Name = "AutoAfkButton"
afkBtn.Size = UDim2.new(0, 232, 0, 26)
afkBtn.Position = UDim2.new(0.5, -116, 0, 174)
afkBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
afkBtn.TextColor3 = Color3.fromRGB(220, 220, 220)
afkBtn.TextSize = 12
afkBtn.Font = Enum.Font.GothamBold
afkBtn.Text = "자동 AFK 방지: OFF"
afkBtn.Visible = checkSavedPremiumAuthenticated()
afkBtn.Parent = titleFrame

local uiCornerAfk = Instance.new("UICorner")
uiCornerAfk.CornerRadius = UDim.new(0, 6)
uiCornerAfk.Parent = afkBtn

afkBtn.MouseButton1Click:Connect(function()
    autoAfkEnabled = not autoAfkEnabled
    if autoAfkEnabled then
        afkBtn.BackgroundColor3 = Color3.fromRGB(0, 180, 90)
        afkBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        afkBtn.Text = "자동 AFK 방지: ON"
    else
        afkBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
        afkBtn.TextColor3 = Color3.fromRGB(220, 220, 220)
        afkBtn.Text = "자동 AFK 방지: OFF"
    end
end)

task.spawn(function()
    while true do
        task.wait(1)
        local isTimed = (_G.AXR_ActiveKey == savedKeyVault.timedProKey)
        timerLabel.Visible = isTimed

        if isTimed then
            local expiryTime = timedPremiumExpiryMap[savedKeyVault.timedProKey] or 0
            local remainSec = expiryTime - os.time()
            if remainSec <= 0 or _G.AXR_TimedKeyExhausted then
                _G.AXR_TimedKeyExhausted = true
                _G.AXR_Authenticated = false
                _G.AXR_PremiumAuthenticated = false
                _G.AXR_ActiveKey = nil
                titleFrame.Visible = false
                if createKeySystemUI then createKeySystemUI() end
                break
            else
                local mm = math.floor(remainSec / 60)
                local ss = remainSec % 60
                timerLabel.Text = string.format("   남은 시간: %02d분 %02d초 (만료)", mm, ss)
            end
        end

        local isPrem = checkSavedPremiumAuthenticated()
        autoBtn.Visible = isPrem
        delayBox.Visible = isPrem
        afkBtn.Visible = isPrem
        if not isPrem and autoAnswerEnabled then
            autoAnswerEnabled = false
            autoBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
            autoBtn.Text = "자동정답: OFF"
        end
    end
end)

task.spawn(function()
    while true do
        task.wait(50)
        if autoAfkEnabled and checkSavedPremiumAuthenticated() then
            pcall(function()
                if VirtualUser then
                    VirtualUser:CaptureController()
                    VirtualUser:ClickButton2(Vector2.new(0,0))
                end
            end)
        end
    end
end)

local function updatePremiumUIVisibility(isVisible)
    autoBtn.Visible = isVisible
    delayBox.Visible = isVisible
    afkBtn.Visible = isVisible
    timerLabel.Visible = (_G.AXR_ActiveKey == savedKeyVault.timedProKey)
end

-- ==========================================
-- [문의하기 UI 생성 함수]
-- ==========================================
local function createInquiryUI(settingsFrame)
    settingsFrame.Visible = false

    local inquiryFrame = Instance.new("Frame")
    inquiryFrame.Size = UDim2.new(0, 360, 0, 310)
    inquiryFrame.Position = UDim2.new(0.5, -180, 0.4, -155)
    inquiryFrame.BackgroundColor3 = Color3.fromRGB(24, 24, 28)
    inquiryFrame.BorderSizePixel = 0
    inquiryFrame.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = inquiryFrame

    local accentLine = Instance.new("Frame")
    accentLine.Size = UDim2.new(1, 0, 0, 3)
    accentLine.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
    accentLine.BorderSizePixel = 0
    accentLine.Parent = inquiryFrame

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 40)
    title.BackgroundTransparency = 1
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.TextSize = 15
    title.Font = Enum.Font.GothamBold
    title.Text = "  개발자에게 문의하기"
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = inquiryFrame

    local noticeLbl = Instance.new("TextLabel")
    noticeLbl.Size = UDim2.new(0, 330, 0, 30)
    noticeLbl.Position = UDim2.new(0.5, -165, 0, 42)
    noticeLbl.BackgroundTransparency = 1
    noticeLbl.TextColor3 = Color3.fromRGB(255, 170, 0)
    noticeLbl.TextSize = 11
    noticeLbl.Font = Enum.Font.GothamMedium
    noticeLbl.Text = "⚠️ 주의: 장난 및 도배성 문의는 개발자에게 실시간 알림이 가므로 자제해 주세요!"
    noticeLbl.TextWrapped = true
    noticeLbl.TextXAlignment = Enum.TextXAlignment.Left
    noticeLbl.Parent = inquiryFrame

    local contentBox = Instance.new("TextBox")
    contentBox.Size = UDim2.new(0, 330, 0, 45)
    contentBox.Position = UDim2.new(0.5, -165, 0, 78)
    contentBox.BackgroundColor3 = Color3.fromRGB(16, 16, 20)
    contentBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    contentBox.PlaceholderColor3 = Color3.fromRGB(130, 130, 145)
    contentBox.PlaceholderText = "개발자에게 전달할 메시지를 입력하세요..."
    contentBox.TextSize = 12
    contentBox.Font = Enum.Font.Gotham
    contentBox.Text = ""
    contentBox.ClearTextOnFocus = false
    contentBox.Parent = inquiryFrame

    local c1 = Instance.new("UICorner")
    c1.CornerRadius = UDim.new(0, 6)
    c1.Parent = contentBox

    local discordBox = Instance.new("TextBox")
    discordBox.Size = UDim2.new(0, 330, 0, 45)
    discordBox.Position = UDim2.new(0.5, -165, 0, 130)
    discordBox.BackgroundColor3 = Color3.fromRGB(16, 16, 20)
    discordBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    discordBox.PlaceholderColor3 = Color3.fromRGB(130, 130, 145)
    discordBox.PlaceholderText = "예: OOOO#0 또는 본인 디스코드 닉네임"
    discordBox.TextSize = 12
    discordBox.Font = Enum.Font.Gotham
    discordBox.Text = ""
    discordBox.ClearTextOnFocus = false
    discordBox.Parent = inquiryFrame

    local c2 = Instance.new("UICorner")
    c2.CornerRadius = UDim.new(0, 6)
    c2.Parent = discordBox

    local statusLbl = Instance.new("TextLabel")
    statusLbl.Size = UDim2.new(1, 0, 0, 20)
    statusLbl.Position = UDim2.new(0, 0, 0, 180)
    statusLbl.BackgroundTransparency = 1
    statusLbl.TextColor3 = Color3.fromRGB(200, 200, 200)
    statusLbl.TextSize = 11
    statusLbl.Font = Enum.Font.GothamMedium
    statusLbl.Text = ""
    statusLbl.Parent = inquiryFrame

    local sendBtn = Instance.new("TextButton")
    sendBtn.Size = UDim2.new(0, 330, 0, 38)
    sendBtn.Position = UDim2.new(0.5, -165, 0, 205)
    sendBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 85)
    sendBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    sendBtn.TextSize = 13
    sendBtn.Font = Enum.Font.GothamBold
    sendBtn.Text = "문의 내용 보내기"
    sendBtn.Parent = inquiryFrame

    local c3 = Instance.new("UICorner")
    c3.CornerRadius = UDim.new(0, 6)
    c3.Parent = sendBtn

    local backBtn = Instance.new("TextButton")
    backBtn.Size = UDim2.new(0, 330, 0, 32)
    backBtn.Position = UDim2.new(0.5, -165, 0, 250)
    backBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 70)
    backBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    backBtn.TextSize = 12
    backBtn.Font = Enum.Font.GothamBold
    backBtn.Text = "돌아가기"
    backBtn.Parent = inquiryFrame

    local c4 = Instance.new("UICorner")
    c4.CornerRadius = UDim.new(0, 6)
    c4.Parent = backBtn

    sendBtn.MouseButton1Click:Connect(function()
        local inquiryText = contentBox.Text:gsub("^%s*(.-)%s*$", "%1")
        local discordTag = discordBox.Text:gsub("^%s*(.-)%s*$", "%1")

        if inquiryText == "" then
            statusLbl.TextColor3 = Color3.fromRGB(255, 80, 80)
            statusLbl.Text = "문의할 내용을 입력해주세요."
            return
        end
        if discordTag == "" then
            statusLbl.TextColor3 = Color3.fromRGB(255, 80, 80)
            statusLbl.Text = "디스코드 표시 닉네임을 입력해주세요."
            return
        end

        local payload = {
            content = string.format("📩 **새로운 개발자 문의가 도착했습니다!**\n• 로블록스 닉네임: `%s`\n• 표시 닉네임: `%s`\n• 고유 숫자 ID: `%d`\n• 디스코드 표시 닉네임: `%s`\n• 문의 내용:\n> %s", 
                localPlayer.Name, localPlayer.DisplayName, localPlayer.UserId, discordTag, inquiryText)
        }

        sendWebhook(inquiryWebhookUrl, payload)
        statusLbl.TextColor3 = Color3.fromRGB(50, 255, 50)
        statusLbl.Text = "문의가 성공적으로 전송되었습니다!"
        task.wait(1.5)
        inquiryFrame:Destroy()
        settingsFrame.Visible = true
    end)

    backBtn.MouseButton1Click:Connect(function()
        inquiryFrame:Destroy()
        settingsFrame.Visible = true
    end)
end

-- ==========================================
-- [설정 창 생성 함수]
-- ==========================================
local function createSettingsUI()
    titleFrame.Visible = false

    local settingsFrame = Instance.new("Frame")
    settingsFrame.Size = UDim2.new(0, 270, 0, 225)
    settingsFrame.Position = UDim2.new(0.5, -135, 0.4, -112)
    settingsFrame.BackgroundColor3 = Color3.fromRGB(24, 24, 28)
    settingsFrame.BorderSizePixel = 0
    settingsFrame.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = settingsFrame

    local accentLine = Instance.new("Frame")
    accentLine.Size = UDim2.new(1, 0, 0, 3)
    accentLine.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
    accentLine.BorderSizePixel = 0
    accentLine.Parent = settingsFrame

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 40)
    title.BackgroundTransparency = 1
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.TextSize = 15
    title.Font = Enum.Font.GothamBold
    title.Text = "   ⚙️ AXR 헬퍼 설정"
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = settingsFrame

    local resetKeyBtn = Instance.new("TextButton")
    resetKeyBtn.Size = UDim2.new(0, 230, 0, 34)
    resetKeyBtn.Position = UDim2.new(0.5, -115, 0, 48)
    resetKeyBtn.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
    resetKeyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    resetKeyBtn.TextSize = 12
    resetKeyBtn.Font = Enum.Font.GothamBold
    resetKeyBtn.Text = "키 시스템 초기화"
    resetKeyBtn.Parent = settingsFrame

    local uiCornerReset = Instance.new("UICorner")
    uiCornerReset.CornerRadius = UDim.new(0, 6)
    uiCornerReset.Parent = resetKeyBtn

    local inquiryBtn = Instance.new("TextButton")
    inquiryBtn.Size = UDim2.new(0, 230, 0, 34)
    inquiryBtn.Position = UDim2.new(0.5, -115, 0, 88)
    inquiryBtn.BackgroundColor3 = Color3.fromRGB(88, 101, 242)
    inquiryBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    inquiryBtn.TextSize = 12
    inquiryBtn.Font = Enum.Font.GothamBold
    inquiryBtn.Text = "문의하기"
    inquiryBtn.Parent = settingsFrame

    local uiCornerInquiry = Instance.new("UICorner")
    uiCornerInquiry.CornerRadius = UDim.new(0, 6)
    uiCornerInquiry.Parent = inquiryBtn

    local destroyScriptBtn = Instance.new("TextButton")
    destroyScriptBtn.Size = UDim2.new(0, 230, 0, 34)
    destroyScriptBtn.Position = UDim2.new(0.5, -115, 0, 128)
    destroyScriptBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 70)
    destroyScriptBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    destroyScriptBtn.TextSize = 12
    destroyScriptBtn.Font = Enum.Font.GothamBold
    destroyScriptBtn.Text = "스크립트 완전히 삭제"
    destroyScriptBtn.Parent = settingsFrame

    local uiCornerDestroy = Instance.new("UICorner")
    uiCornerDestroy.CornerRadius = UDim.new(0, 6)
    uiCornerDestroy.Parent = destroyScriptBtn

    local closeSettingsBtn = Instance.new("TextButton")
    closeSettingsBtn.Size = UDim2.new(0, 230, 0, 32)
    closeSettingsBtn.Position = UDim2.new(0.5, -115, 0, 174)
    closeSettingsBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 52)
    closeSettingsBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
    closeSettingsBtn.TextSize = 12
    closeSettingsBtn.Font = Enum.Font.GothamBold
    closeSettingsBtn.Text = "닫기 (메인 창 복귀)"
    closeSettingsBtn.Parent = settingsFrame

    local uiCornerClose = Instance.new("UICorner")
    uiCornerClose.CornerRadius = UDim.new(0, 6)
    uiCornerClose.Parent = closeSettingsBtn

    resetKeyBtn.MouseButton1Click:Connect(function()
        _G.AXR_Authenticated = false
        _G.AXR_PremiumAuthenticated = false
        _G.AXR_ActiveKey = nil
        updatePremiumUIVisibility(false)
        settingsFrame:Destroy()
        if createKeySystemUI then createKeySystemUI() end
    end)

    inquiryBtn.MouseButton1Click:Connect(function()
        createInquiryUI(settingsFrame)
    end)

    destroyScriptBtn.MouseButton1Click:Connect(function()
        pcall(function()
            if screenGui then
                screenGui:Destroy()
            end
        end)
    end)

    closeSettingsBtn.MouseButton1Click:Connect(function()
        settingsFrame:Destroy()
        if checkSavedAuth() or checkSavedPremiumAuthenticated() then
            titleFrame.Visible = true
        end
    end)
end

local settingsIconBtn = Instance.new("TextButton")
settingsIconBtn.Name = "SettingsIconButton"
settingsIconBtn.Size = UDim2.new(0, 26, 0, 26)
settingsIconBtn.Position = UDim2.new(1, -30, 0, 6)
settingsIconBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
settingsIconBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
settingsIconBtn.TextSize = 13
settingsIconBtn.Font = Enum.Font.GothamBold
settingsIconBtn.Text = "⚙"
settingsIconBtn.Parent = titleFrame

local uiCornerSettingsIcon = Instance.new("UICorner")
uiCornerSettingsIcon.CornerRadius = UDim.new(0, 6)
uiCornerSettingsIcon.Parent = settingsIconBtn

settingsIconBtn.MouseButton1Click:Connect(function()
    createSettingsUI()
end)

-- ==========================================
-- [자동 정답 입력 및 최신 큐 관리 로직]
-- ==========================================
local currentAnswer = ""
local latestInputThread = nil -- 이전 지연 대기를 취소하기 위한 스레드 변수

local function triggerAutoInput(word)
    if not checkSavedPremiumAuthenticated() or not autoAnswerEnabled then return end
    
    -- 이미 대기 중인 지연 입력 작업이 있다면 즉시 취소하여 엉뚱한 이전 단어가 입력되는 것 방지
    if latestInputThread then
        task.cancel(latestInputThread)
        latestInputThread = nil
    end

    latestInputThread = task.spawn(function()
        local delayVal = tonumber(delayBox.Text) or 0
        if delayVal > 0 then
            task.wait(delayVal)
        end
        
        if not autoAnswerEnabled then return end

        local targetBox = nil
        local focusedGui = UserInputService:GetFocusedTextBox()
        if focusedGui and focusedGui:IsA("TextBox") then
            targetBox = focusedGui
        else
            if playerGui then
                for _, descendant in ipairs(playerGui:GetDescendants()) do
                    if descendant:IsA("TextBox") and descendant.Visible and descendant.AbsoluteSize.X > 0 then
                        local phText = (descendant.PlaceholderText or ""):lower()
                        local txt = (descendant.Text or ""):lower()
                        if phText:find("입력") or phText:find("단어") or phText:find("여기에") or phText:find("chat") or
                           txt:find("입력") or txt:find("단어") or txt:find("여기에") then
                            targetBox = descendant
                            break
                        end
                    end
                end
            end
            if not targetBox and playerGui then
                for _, descendant in ipairs(playerGui:GetDescendants()) do
                    if descendant:IsA("TextBox") and descendant.Visible and descendant.AbsoluteSize.X > 50 then
                        targetBox = descendant
                        break
                    end
                end
            end
        end

        if targetBox then
            targetBox.Text = word
            task.spawn(function()
                targetBox:CaptureFocus()
                task.wait(0.04)
                if VirtualInputManager then
                    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.Return, false, game)
                    task.wait(0.03)
                    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.Return, false, game)
                end
            end)
        end
        latestInputThread = nil
    end)
end

local function isValidWord(txt)
    if not txt or type(txt) ~= "string" then return false end
    txt = txt:gsub("^%s*(.-)%s*$", "%1")
    if txt == "" then return false end
    if txt:find("#") or txt:find("_") then return false end
    if txt:find("%s") then return false end
    if #txt > 25 then return false end
    if tonumber(txt) ~= nil then return false end
    
    local lowerTxt = txt:lower()
    if lowerTxt == "total" or lowerTxt:find("total") or lowerTxt == "설정" or lowerTxt == "옵션" or lowerTxt == "메뉴" or lowerTxt == "상점" or lowerTxt == "정보" or lowerTxt == "선택됨" then
        return false
    end
    if lowerTxt:match("^cl") or lowerTxt:match("^gui") or lowerTxt:match("^rem") or lowerTxt:match("^http") then
        return false
    end
    return true
end

local function checkRoundReset(txt)
    if not txt or type(txt) ~= "string" then return false end
    local low = txt:lower()
    if low:find("대기") or low:find("라운드") or low:find("시작") or low:find("끝") or low:find("종료") or low:find("ready") or low:find("wait") or low:find("end") or low:find("over") or low:find("finish") then
        return true
    end
    return false
end

local function processValue(txt)
    if not txt or type(txt) ~= "string" then return end
    txt = txt:gsub("^%s*(.-)%s*$", "%1")
    if checkRoundReset(txt) then
        if currentAnswer ~= "RESET" then
            currentAnswer = "RESET"
            answerLabel.Text = "정답: 라운드 대기 중..."
        end
    elseif isValidWord(txt) then
        if txt ~= currentAnswer then
            currentAnswer = txt
            answerLabel.Text = "정답: " .. txt
            triggerAutoInput(txt)
        end
    end
end

pcall(function()
    local function hookEvent(v)
        if v:IsA("RemoteEvent") or v:IsA("UnreliableRemoteEvent") then
            v.OnClientEvent:Connect(function(...)
                local args = {...}
                for _, arg in ipairs(args) do
                    if type(arg) == "string" then processValue(arg)
                    elseif type(arg) == "table" then
                        for _, subArg in pairs(arg) do
                            if type(subArg) == "string" then processValue(subArg) end
                        end
                    end
                end
            end)
        end
    end
    for _, v in ipairs(ReplicatedStorage:GetDescendants()) do hookEvent(v) end
    ReplicatedStorage.DescendantAdded:Connect(hookEvent)
end)

pcall(function()
    for _, obj in ipairs(ReplicatedStorage:GetDescendants()) do
        if obj:IsA("StringValue") or obj:IsA("TextValue") then
            processValue(obj.Value)
            obj.Changed:Connect(processValue)
        end
    end
    ReplicatedStorage.DescendantAdded:Connect(function(obj)
        if obj:IsA("StringValue") or obj:IsA("TextValue") then
            obj.Changed:Connect(processValue)
        end
    end)
end)

-- ==========================================
-- [패치노트 및 인증 시스템 UI]
-- ==========================================
local function createPatchNotesUI(keyFrame)
    local patchFrame = Instance.new("Frame")
    patchFrame.Size = UDim2.new(0, 320, 0, 330)
    patchFrame.Position = UDim2.new(0.5, -160, 0.4, -165)
    patchFrame.BackgroundColor3 = Color3.fromRGB(24, 24, 28)
    patchFrame.BorderSizePixel = 0
    patchFrame.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = patchFrame

    local line = Instance.new("Frame")
    line.Size = UDim2.new(1, 0, 0, 3)
    line.BackgroundColor3 = Color3.fromRGB(70, 130, 180)
    line.BorderSizePixel = 0
    line.Parent = patchFrame

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 42)
    title.BackgroundTransparency = 1
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.TextSize = 15
    title.Font = Enum.Font.GothamBold
    title.Text = "  📜 AXR 패치노트 & 업데이트"
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = patchFrame

    local contentBox = Instance.new("TextLabel")
    contentBox.Size = UDim2.new(0, 288, 0, 210)
    contentBox.Position = UDim2.new(0.5, -144, 0, 46)
    contentBox.BackgroundColor3 = Color3.fromRGB(16, 16, 20)
    contentBox.TextColor3 = Color3.fromRGB(210, 210, 220)
    contentBox.TextSize = 12
    contentBox.Font = Enum.Font.Gotham
    contentBox.TextXAlignment = Enum.TextXAlignment.Left
    contentBox.TextYAlignment = Enum.TextYAlignment.Top
    contentBox.TextWrapped = true
    contentBox.Text = [[
[ AXR v2.3 패치 내역 ]
• 지연 시간 변경 시 이전 단어가 밀려서 입력되던 버그 수정 (최신 정답 우선 입력 큐 적용)
• lngstock55 사용자 프리미엄 권한 및 전용 키 등록 완료
• 허용되지 않은 사용자 실행 시 웹훅 경고 및 즉시 킥 처리 보안 유지
]]
    contentBox.Parent = patchFrame

    local boxCorner = Instance.new("UICorner")
    boxCorner.CornerRadius = UDim.new(0, 6)
    boxCorner.Parent = contentBox

    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 288, 0, 32)
    closeBtn.Position = UDim2.new(0.5, -144, 0, 268)
    closeBtn.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.TextSize = 13
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.Text = "닫기"
    closeBtn.Parent = patchFrame

    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 6)
    btnCorner.Parent = closeBtn

    closeBtn.MouseButton1Click:Connect(function()
        patchFrame:Destroy()
        if keyFrame then
            keyFrame.Visible = true
        end
    end)
end

local function createKeyInfoResultUI(specialFrame)
    local infoFrame = Instance.new("Frame")
    infoFrame.Size = UDim2.new(0, 360, 0, 310)
    infoFrame.Position = UDim2.new(0.5, -180, 0.4, -155)
    infoFrame.BackgroundColor3 = Color3.fromRGB(24, 24, 28)
    infoFrame.BorderSizePixel = 0
    infoFrame.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = infoFrame

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 42)
    title.BackgroundTransparency = 1
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.TextSize = 16
    title.Font = Enum.Font.GothamBold
    title.Text = "  AXR 저장된 키 모음 정보"
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = infoFrame

    local normalKeyBtn = Instance.new("TextButton")
    normalKeyBtn.Size = UDim2.new(0, 320, 0, 42)
    normalKeyBtn.Position = UDim2.new(0.5, -160, 0, 48)
    normalKeyBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 215)
    normalKeyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    normalKeyBtn.TextSize = 12
    normalKeyBtn.Font = Enum.Font.GothamBold
    normalKeyBtn.Text = "기본 키로 적용 및 실행\n(" .. savedKeyVault.zxxdaswoNormalKey .. ")"
    normalKeyBtn.Parent = infoFrame

    local c1 = Instance.new("UICorner")
    c1.CornerRadius = UDim.new(0, 6)
    c1.Parent = normalKeyBtn

    local premiumKeyBtn = Instance.new("TextButton")
    premiumKeyBtn.Size = UDim2.new(0, 320, 0, 42)
    premiumKeyBtn.Position = UDim2.new(0.5, -160, 0, 98)
    premiumKeyBtn.BackgroundColor3 = Color3.fromRGB(230, 130, 0)
    premiumKeyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    premiumKeyBtn.TextSize = 12
    premiumKeyBtn.Font = Enum.Font.GothamBold
    premiumKeyBtn.Text = "프리미엄 키로 적용 및 실행\n(" .. savedKeyVault.zxxdaswoPremiumKey .. ")"
    premiumKeyBtn.Parent = infoFrame

    local c2 = Instance.new("UICorner")
    c2.CornerRadius = UDim.new(0, 6)
    c2.Parent = premiumKeyBtn

    local masterKeyBtn = Instance.new("TextButton")
    masterKeyBtn.Size = UDim2.new(0, 320, 0, 42)
    masterKeyBtn.Position = UDim2.new(0.5, -160, 0, 148)
    masterKeyBtn.BackgroundColor3 = Color3.fromRGB(219, 112, 147)
    masterKeyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    masterKeyBtn.TextSize = 12
    masterKeyBtn.Font = Enum.Font.GothamBold
    masterKeyBtn.Text = "마스터 키로 적용 및 실행\n(" .. savedKeyVault.masterKeyText .. ")"
    masterKeyBtn.Parent = infoFrame

    local c3 = Instance.new("UICorner")
    c3.CornerRadius = UDim.new(0, 6)
    c3.Parent = masterKeyBtn

    normalKeyBtn.MouseButton1Click:Connect(function()
        _G.AXR_Authenticated = true
        _G.AXR_PremiumAuthenticated = false
        _G.AXR_ActiveKey = nil
        infoFrame:Destroy()
        if specialFrame then specialFrame:Destroy() end
        playAXRIntro()
        titleFrame.Visible = true
        updatePremiumUIVisibility(false)
    end)

    premiumKeyBtn.MouseButton1Click:Connect(function()
        _G.AXR_Authenticated = true
        _G.AXR_PremiumAuthenticated = true
        _G.AXR_ActiveKey = savedKeyVault.zxxdaswoPremiumKey
        infoFrame:Destroy()
        if specialFrame then specialFrame:Destroy() end
        playAXRIntro()
        titleFrame.Visible = true
        updatePremiumUIVisibility(true)
    end)

    masterKeyBtn.MouseButton1Click:Connect(function()
        _G.AXR_Authenticated = true
        _G.AXR_PremiumAuthenticated = true
        _G.AXR_ActiveKey = savedKeyVault.masterKeyText
        infoFrame:Destroy()
        if specialFrame then specialFrame:Destroy() end
        playAXRIntro()
        titleFrame.Visible = true
        updatePremiumUIVisibility(true)
    end)
end

local function createSpecialCodeUI(keyFrame)
    local specialFrame = Instance.new("Frame")
    specialFrame.Size = UDim2.new(0, 300, 0, 185)
    specialFrame.Position = UDim2.new(0.5, -150, 0.4, -92)
    specialFrame.BackgroundColor3 = Color3.fromRGB(24, 24, 28)
    specialFrame.BorderSizePixel = 0
    specialFrame.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = specialFrame

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 38)
    title.BackgroundTransparency = 1
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.TextSize = 14
    title.Font = Enum.Font.GothamBold
    title.Text = "  스크 개발자 / 허용한 친구 코드"
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = specialFrame

    local codeBox = Instance.new("TextBox")
    codeBox.Size = UDim2.new(0, 264, 0, 34)
    codeBox.Position = UDim2.new(0.5, -132, 0, 44)
    codeBox.BackgroundColor3 = Color3.fromRGB(16, 16, 20)
    codeBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    codeBox.PlaceholderColor3 = Color3.fromRGB(130, 130, 145)
    codeBox.PlaceholderText = "코드를 입력하세요..."
    codeBox.TextSize = 13
    codeBox.Font = Enum.Font.Gotham
    codeBox.Text = ""
    codeBox.Parent = specialFrame

    local boxCorner = Instance.new("UICorner")
    boxCorner.CornerRadius = UDim.new(0, 6)
    boxCorner.Parent = codeBox

    local statusLbl = Instance.new("TextLabel")
    statusLbl.Size = UDim2.new(1, 0, 0, 25)
    statusLbl.Position = UDim2.new(0, 0, 0, 84)
    statusLbl.BackgroundTransparency = 1
    statusLbl.TextColor3 = Color3.fromRGB(255, 80, 80)
    statusLbl.TextSize = 12
    statusLbl.Font = Enum.Font.GothamMedium
    statusLbl.Text = ""
    statusLbl.Parent = specialFrame

    local submitCodeBtn = Instance.new("TextButton")
    submitCodeBtn.Size = UDim2.new(0, 128, 0, 32)
    submitCodeBtn.Position = UDim2.new(0.5, -132, 0, 122)
    submitCodeBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 85)
    submitCodeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    submitCodeBtn.TextSize = 13
    submitCodeBtn.Font = Enum.Font.GothamBold
    submitCodeBtn.Text = "확인"
    submitCodeBtn.Parent = specialFrame

    local btnCorner1 = Instance.new("UICorner")
    btnCorner1.CornerRadius = UDim.new(0, 6)
    btnCorner1.Parent = submitCodeBtn

    local cancelCodeBtn = Instance.new("TextButton")
    cancelCodeBtn.Size = UDim2.new(0, 128, 0, 32)
    cancelCodeBtn.Position = UDim2.new(0.5, 4, 0, 122)
    cancelCodeBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 70)
    cancelCodeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    cancelCodeBtn.TextSize = 13
    cancelCodeBtn.Font = Enum.Font.GothamBold
    cancelCodeBtn.Text = "취소"
    cancelCodeBtn.Parent = specialFrame

    local btnCorner2 = Instance.new("UICorner")
    btnCorner2.CornerRadius = UDim.new(0, 6)
    btnCorner2.Parent = cancelCodeBtn

    cancelCodeBtn.MouseButton1Click:Connect(function()
        specialFrame:Destroy()
        keyFrame.Visible = true
    end)

    submitCodeBtn.MouseButton1Click:Connect(function()
        local entered = codeBox.Text:gsub("^%s*(.-)%s*$", "%1")
        if entered == specialBypassCode then
            statusLbl.TextColor3 = Color3.fromRGB(50, 255, 50)
            statusLbl.Text = "인증 성공!"
            task.wait(0.4)
            createKeyInfoResultUI(specialFrame)
            if keyFrame then keyFrame:Destroy() end
        else
            statusLbl.TextColor3 = Color3.fromRGB(255, 80, 80)
            statusLbl.Text = "코드가 일치하지 않습니다."
        end
    end)
end

-- ==========================================
-- [메인 키 시스템 UI]
-- ==========================================
createKeySystemUI = function()
    local keyFrame = Instance.new("Frame")
    keyFrame.Name = "KeySystemFrame"
    keyFrame.Size = UDim2.new(0, 300, 0, 340)
    keyFrame.Position = UDim2.new(0.5, -150, 0.4, -170)
    keyFrame.BackgroundColor3 = Color3.fromRGB(24, 24, 28)
    keyFrame.BorderSizePixel = 0
    keyFrame.Visible = true
    keyFrame.Parent = screenGui

    local uiCornerKey = Instance.new("UICorner")
    uiCornerKey.CornerRadius = UDim.new(0, 10)
    uiCornerKey.Parent = keyFrame

    local keyTitle = Instance.new("TextLabel")
    keyTitle.Size = UDim2.new(1, 0, 0, 40)
    keyTitle.BackgroundTransparency = 1
    keyTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    keyTitle.TextSize = 15
    keyTitle.Font = Enum.Font.GothamBold
    keyTitle.Text = "  AXR 전용 인증 시스템"
    keyTitle.TextXAlignment = Enum.TextXAlignment.Left
    keyTitle.Parent = keyFrame

    local keyBox = Instance.new("TextBox")
    keyBox.Size = UDim2.new(0, 268, 0, 32)
    keyBox.Position = UDim2.new(0.5, -134, 0, 42)
    keyBox.BackgroundColor3 = Color3.fromRGB(16, 16, 20)
    keyBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    keyBox.PlaceholderColor3 = Color3.fromRGB(130, 130, 145)
    keyBox.PlaceholderText = "비밀 키를 입력하세요..."
    keyBox.TextSize = 12
    keyBox.Font = Enum.Font.Gotham
    keyBox.Text = ""
    keyBox.Parent = keyFrame

    local uiCornerBox = Instance.new("UICorner")
    uiCornerBox.CornerRadius = UDim.new(0, 6)
    uiCornerBox.Parent = keyBox

    local submitBtn = Instance.new("TextButton")
    submitBtn.Size = UDim2.new(0, 268, 0, 30)
    submitBtn.Position = UDim2.new(0.5, -134, 0, 80)
    submitBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
    submitBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    submitBtn.TextSize = 12
    submitBtn.Font = Enum.Font.GothamBold
    submitBtn.Text = "인증하기"
    submitBtn.Parent = keyFrame

    local uiCornerSub = Instance.new("UICorner")
    uiCornerSub.CornerRadius = UDim.new(0, 6)
    uiCornerSub.Parent = submitBtn

    local buyBtn = Instance.new("TextButton")
    buyBtn.Size = UDim2.new(0, 268, 0, 28)
    buyBtn.Position = UDim2.new(0.5, -134, 0, 118)
    buyBtn.BackgroundColor3 = Color3.fromRGB(88, 101, 242)
    buyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    buyBtn.TextSize = 12
    buyBtn.Font = Enum.Font.GothamBold
    buyBtn.Text = "전용 키 구매"
    buyBtn.Parent = keyFrame

    local uiCornerBuy = Instance.new("UICorner")
    uiCornerBuy.CornerRadius = UDim.new(0, 6)
    uiCornerBuy.Parent = buyBtn

    local buyProBtn = Instance.new("TextButton")
    buyProBtn.Size = UDim2.new(0, 268, 0, 28)
    buyProBtn.Position = UDim2.new(0.5, -134, 0, 152)
    buyProBtn.BackgroundColor3 = Color3.fromRGB(255, 140, 0)
    buyProBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    buyProBtn.TextSize = 12
    buyProBtn.Font = Enum.Font.GothamBold
    buyProBtn.Text = "프리미엄 전용 키 구매"
    buyProBtn.Parent = keyFrame

    local uiCornerBuyPro = Instance.new("UICorner")
    uiCornerBuyPro.CornerRadius = UDim.new(0, 6)
    uiCornerBuyPro.Parent = buyProBtn

    local devFriendBtn = Instance.new("TextButton")
    devFriendBtn.Size = UDim2.new(0, 268, 0, 28)
    devFriendBtn.Position = UDim2.new(0.5, -134, 0, 186)
    devFriendBtn.BackgroundColor3 = Color3.fromRGB(120, 60, 180)
    devFriendBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    devFriendBtn.TextSize = 11
    devFriendBtn.Font = Enum.Font.GothamBold
    devFriendBtn.Text = "스크 개발자 전용 또는 허용한 친구"
    devFriendBtn.Parent = keyFrame

    local uiCornerDevFriend = Instance.new("UICorner")
    uiCornerDevFriend.CornerRadius = UDim.new(0, 6)
    uiCornerDevFriend.Parent = devFriendBtn

    local patchNoteBtn = Instance.new("TextButton")
    patchNoteBtn.Size = UDim2.new(0, 268, 0, 28)
    patchNoteBtn.Position = UDim2.new(0.5, -134, 0, 220)
    patchNoteBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 70)
    patchNoteBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    patchNoteBtn.TextSize = 12
    patchNoteBtn.Font = Enum.Font.GothamBold
    patchNoteBtn.Text = "📜 AXR 패치노트 및 업데이트 확인"
    patchNoteBtn.Parent = keyFrame

    local uiCornerPatch = Instance.new("UICorner")
    uiCornerPatch.CornerRadius = UDim.new(0, 6)
    uiCornerPatch.Parent = patchNoteBtn

    local statusLabel = Instance.new("TextLabel")
    statusLabel.Size = UDim2.new(1, 0, 0, 25)
    statusLabel.Position = UDim2.new(0, 0, 0, 256)
    statusLabel.BackgroundTransparency = 1
    statusLabel.TextColor3 = Color3.fromRGB(255, 80, 80)
    statusLabel.TextSize = 12
    statusLabel.Font = Enum.Font.GothamMedium
    statusLabel.Text = ""
    statusLabel.Parent = keyFrame

    local function copyDiscordLink()
        local discordLink = "https://discord.gg/ZKenYVezV"
        pcall(function()
            if setclipboard then setclipboard(discordLink) end
        end)
        statusLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
        statusLabel.Text = "디스코드 링크가 복사되었습니다!"
    end

    buyBtn.MouseButton1Click:Connect(copyDiscordLink)
    buyProBtn.MouseButton1Click:Connect(copyDiscordLink)

    devFriendBtn.MouseButton1Click:Connect(function()
        keyFrame.Visible = false
        createSpecialCodeUI(keyFrame)
    end)

    patchNoteBtn.MouseButton1Click:Connect(function()
        keyFrame.Visible = false
        createPatchNotesUI(keyFrame)
    end)

    submitBtn.MouseButton1Click:Connect(function()
        local playerName = localPlayer.Name:gsub("^%s*(.-)%s*$", "%1")
        local enteredKey = keyBox.Text:gsub("^%s*(.-)%s*$", "%1")
        
        if enteredKey == savedKeyVault.timedProKey then
            if _G.AXR_TimedKeyExhausted or os.time() > timedPremiumExpiryMap[savedKeyVault.timedProKey] then
                _G.AXR_TimedKeyExhausted = true
                statusLabel.TextColor3 = Color3.fromRGB(255, 80, 80)
                statusLabel.Text = "만료되었거나 이미 사용된 시간제 키입니다."
                return
            end
            statusLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
            statusLabel.Text = "시간제 프리미엄 키 인증 성공!"
            task.wait(0.6)
            
            _G.AXR_Authenticated = true
            _G.AXR_PremiumAuthenticated = true
            _G.AXR_ActiveKey = enteredKey
            
            keyFrame:Destroy()
            playAXRIntro()
            titleFrame.Visible = true
            updatePremiumUIVisibility(true)

        elseif premiumKeys[playerName] and premiumKeys[playerName] == enteredKey then
            statusLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
            statusLabel.Text = "프리미엄 키 인증 성공! 환영합니다."
            task.wait(0.6)
            keyFrame:Destroy()
            playAXRIntro()
            _G.AXR_Authenticated = true
            _G.AXR_PremiumAuthenticated = true
            _G.AXR_ActiveKey = enteredKey
            titleFrame.Visible = true
            updatePremiumUIVisibility(true)

        elseif userKeys[playerName] and userKeys[playerName] == enteredKey then
            statusLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
            statusLabel.Text = "일반 키 인증 성공! 환영합니다."
            task.wait(0.6)
            keyFrame:Destroy()
            playAXRIntro()
            _G.AXR_Authenticated = true
            _G.AXR_PremiumAuthenticated = false
            _G.AXR_ActiveKey = enteredKey
            titleFrame.Visible = true
            updatePremiumUIVisibility(false)

        else
            statusLabel.TextColor3 = Color3.fromRGB(255, 80, 80)
            statusLabel.Text = "권한이 없거나 잘못된 키입니다."
        end
    end)
end

if not titleFrame.Visible then
    createKeySystemUI()
end

-- 드래그 이동 로직
local dragging, dragStart, startPos = false, nil, nil
titleFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = titleFrame.Position
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        titleFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
