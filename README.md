-- 안전한 서비스 불러오기
local CoreGui = game:GetService("CoreGui")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local RunService = game:GetService("RunService")
local localPlayer = Players.LocalPlayer or Players.PlayerAdded:Wait()

local playerGui = localPlayer:WaitForChild("PlayerGui", 5) or localPlayer:FindFirstChildOfClass("PlayerGui")

-- 기존 GUI 제거 (중복 방지)
pcall(function()
    if CoreGui:FindFirstChild("WordGameHelperUI") then
        CoreGui.WordGameHelperUI:Destroy()
    end
    if playerGui and playerGui:FindFirstChild("WordGameHelperUI") then
        playerGui.WordGameHelperUI:Destroy()
    end
end)

-- ScreenGui 생성
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "WordGameHelperUI"
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
-- [유저별 맞춤형 키 및 프리미엄 키 시스템 설정]
-- ==========================================
local userKeys = {
    ["dambii522"] = "no.1keyap191929",
    ["zxxdaswo"] = "no.1keyap19293949",
    ["1CasaNova6974"] = "no.1keyap172737",
    ["dohunpoop"] = "dohunpoop_key12"
}

-- 프리미엄 전용 키 목록 (지정한 유저와 키 매칭)
local premiumKeys = {
    ["zxxdaswo"] = "zxxdaswo.key.pro"
}

-- 12시간 인증 유지 파일 이름 (유저별로 구분)
local safePlayerName = localPlayer.Name:gsub("[^%w]", "_")
local authFileName = "WordHelper_Auth_" .. safePlayerName .. ".txt"
local premiumAuthFileName = "WordHelper_PremiumAuth_" .. safePlayerName .. ".txt"

-- 일반 인증 유효성 검사 함수
local function checkSavedAuth()
    if writefile and readfile and isfile and isfile(authFileName) then
        local success, data = pcall(function()
            return tonumber(readfile(authFileName))
        end)
        if success and data then
            if os.time() < data then
                return true
            end
        end
    end
    return false
end

-- 프리미엄 인증 유효성 검사 함수
local function checkSavedPremiumAuth()
    if writefile and readfile and isfile and isfile(premiumAuthFileName) then
        local success, data = pcall(function()
            return tonumber(readfile(premiumAuthFileName))
        end)
        if success and data then
            if os.time() < data then
                return true
            end
        end
    end
    return false
end

-- 인증 정보 저장 함수 (12시간 = 43200초)
local function saveAuthSession()
    if writefile then
        pcall(function()
            local expireTime = os.time() + 43200
            writefile(authFileName, tostring(expireTime))
        end)
    end
end

local function savePremiumAuthSession()
    if writefile then
        pcall(function()
            local expireTime = os.time() + 43200
            writefile(authFileName, tostring(expireTime))
            writefile(premiumAuthFileName, tostring(expireTime))
        end)
    end
end

-- ==========================================
-- [메인 헬퍼 UI 생성]
-- ==========================================
local titleFrame = Instance.new("TextButton")
titleFrame.Name = "TitleFrame"
titleFrame.Size = UDim2.new(0, 240, 0, 75)
titleFrame.Position = UDim2.new(0.73, 0, 0.1, 0)
titleFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
titleFrame.TextColor3 = Color3.fromRGB(255, 255, 255)
titleFrame.TextSize = 15
titleFrame.Font = Enum.Font.SourceSansBold
titleFrame.Text = "단어 맞히기 헬퍼"
titleFrame.TextYAlignment = Enum.TextYAlignment.Top
titleFrame.AutoButtonColor = false
titleFrame.Visible = checkSavedAuth() or checkSavedPremiumAuth()
titleFrame.Parent = screenGui

local uiCornerBtn = Instance.new("UICorner")
uiCornerBtn.CornerRadius = UDim.new(0, 8)
uiCornerBtn.Parent = titleFrame

-- 개발자 표시 텍스트 라벨
local devLabel = Instance.new("TextLabel")
devLabel.Name = "DevLabel"
devLabel.Size = UDim2.new(1, 0, 0, 20)
devLabel.Position = UDim2.new(0, 0, 0, 22)
devLabel.BackgroundTransparency = 1
devLabel.TextColor3 = Color3.fromRGB(170, 170, 170)
devLabel.TextSize = 12
devLabel.Font = Enum.Font.SourceSansItalic
devLabel.Text = "스크립트 개발자 : 지환"
devLabel.Parent = titleFrame

-- 자동 정답 토글 버튼
local autoAnswerEnabled = false
local autoBtn = Instance.new("TextButton")
autoBtn.Name = "AutoAnswerButton"
autoBtn.Size = UDim2.new(0, 85, 0, 24)
autoBtn.Position = UDim2.new(1, -90, 0, 45)
autoBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
autoBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
autoBtn.TextSize = 11
autoBtn.Font = Enum.Font.SourceSansBold
autoBtn.Text = "자동정답: OFF"
autoBtn.Parent = titleFrame

local uiCornerAuto = Instance.new("UICorner")
uiCornerAuto.CornerRadius = UDim.new(0, 5)
uiCornerAuto.Parent = autoBtn

autoBtn.MouseButton1Click:Connect(function()
    autoAnswerEnabled = not autoAnswerEnabled
    if autoAnswerEnabled then
        autoBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 85)
        autoBtn.Text = "자동정답: ON"
    else
        autoBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
        autoBtn.Text = "자동정답: OFF"
    end
end)

-- 정답창 (TextLabel) - 초기 텍스트를 '정답: 라운드 대기 중...'으로 수정
local answerLabel = Instance.new("TextLabel")
answerLabel.Name = "AnswerLabel"
answerLabel.Size = UDim2.new(0, 240, 0, 45)
answerLabel.Position = UDim2.new(0, 0, 1, 5)
answerLabel.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
answerLabel.BackgroundTransparency = 0.2
answerLabel.TextColor3 = Color3.fromRGB(0, 255, 128)
answerLabel.TextSize = 18
answerLabel.Font = Enum.Font.SourceSansBold
answerLabel.Text = "정답: 라운드 대기 중..."
answerLabel.Visible = true
answerLabel.Parent = titleFrame

local uiCornerLbl = Instance.new("UICorner")
uiCornerLbl.CornerRadius = UDim.new(0, 8)
uiCornerLbl.Parent = answerLabel

-- 키 시스템 초기화 버튼
local resetKeyBtn = Instance.new("TextButton")
resetKeyBtn.Name = "ResetKeyButton"
resetKeyBtn.Size = UDim2.new(0, 240, 0, 30)
resetKeyBtn.Position = UDim2.new(0, 0, 1, 10)
resetKeyBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
resetKeyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
resetKeyBtn.TextSize = 13
resetKeyBtn.Font = Enum.Font.SourceSansBold
resetKeyBtn.Text = "키 시스템 초기화"
resetKeyBtn.Visible = true
resetKeyBtn.Parent = answerLabel

local uiCornerReset = Instance.new("UICorner")
uiCornerReset.CornerRadius = UDim.new(0, 6)
uiCornerReset.Parent = resetKeyBtn

-- 스크립트 삭제 버튼
local destroyScriptBtn = Instance.new("TextButton")
destroyScriptBtn.Name = "DestroyScriptButton"
destroyScriptBtn.Size = UDim2.new(0, 240, 0, 30)
destroyScriptBtn.Position = UDim2.new(0, 0, 1, 8)
destroyScriptBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
destroyScriptBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
destroyScriptBtn.TextSize = 13
destroyScriptBtn.Font = Enum.Font.SourceSansBold
destroyScriptBtn.Text = "스크립트 삭제"
destroyScriptBtn.Visible = true
destroyScriptBtn.Parent = resetKeyBtn

local uiCornerDestroy = Instance.new("UICorner")
uiCornerDestroy.CornerRadius = UDim.new(0, 6)
uiCornerDestroy.Parent = destroyScriptBtn

destroyScriptBtn.MouseButton1Click:Connect(function()
    pcall(function()
        if screenGui then
            screenGui:Destroy()
        end
    end)
end)

-- ==========================================
-- [프리미엄 전용: 상대방 제어용 원격 입력창 생성]
-- ==========================================
local remoteInputBox = Instance.new("TextBox")
remoteInputBox.Name = "RemoteInputBox"
remoteInputBox.Size = UDim2.new(0, 240, 0, 32)
remoteInputBox.Position = UDim2.new(0, 0, 1, 8)
remoteInputBox.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
remoteInputBox.TextColor3 = Color3.fromRGB(255, 255, 255)
remoteInputBox.PlaceholderColor3 = Color3.fromRGB(160, 160, 160)
remoteInputBox.PlaceholderText = "[프리미엄 전용] 상대에게 단어 전송..."
remoteInputBox.TextSize = 12
remoteInputBox.Font = Enum.Font.SourceSansBold
remoteInputBox.Text = ""
-- 프리미엄 인증을 완료한 유저에게만 노출
remoteInputBox.Visible = checkSavedPremiumAuth()
remoteInputBox.Parent = destroyScriptBtn

local uiCornerRemote = Instance.new("UICorner")
uiCornerRemote.CornerRadius = UDim.new(0, 6)
uiCornerRemote.Parent = remoteInputBox

-- ==========================================
-- [자동 정답 입력 로직]
-- ==========================================
local function triggerAutoInput(word)
    if not autoAnswerEnabled then return end
    pcall(function()
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
                        if phText:find("입력") or phText:find("단어") or phText:find("여기에") or 
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
                targetBox.CaptureFocus()
                task.wait(0.05)
                local clicked = false
                local searchContainers = { targetBox.Parent, playerGui }
                for _, container in ipairs(searchContainers) do
                    if container and not clicked then
                        for _, child in ipairs(container:GetDescendants()) do
                            if (child:IsA("TextButton") or child:IsA("ImageButton")) and child ~= targetBox then
                                local col = child.BackgroundColor3
                                local nameLower = child.Name:lower()
                                if (col.G > col.R and col.G > col.B and col.G > 120) or 
                                   nameLower:find("send") or nameLower:find("submit") or nameLower:find("btn") or nameLower:find("enter") then
                                    local absPos = child.AbsolutePosition
                                    local absSize = child.AbsoluteSize
                                    if absSize.X > 10 and absSize.Y > 10 then
                                        local clickX = absPos.X + (absSize.X / 2)
                                        local clickY = absPos.Y + (absSize.Y / 2) + 36
                                        if VirtualInputManager then
                                            VirtualInputManager:SendMouseButtonEvent(clickX, clickY, 0, true, game, 0)
                                            task.wait(0.03)
                                            VirtualInputManager:SendMouseButtonEvent(clickX, clickY, 0, false, game, 0)
                                            clicked = true
                                        end
                                    end
                                    if clicked then break end
                                end
                            end
                        end
                    end
                    if clicked then break end
                end
                if not clicked and VirtualInputManager then
                    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.Return, false, game)
                    task.wait(0.02)
                    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.Return, false, game)
                end
            end)
        end
    end)
end

-- ==========================================
-- [키 인증 프레임 생성 함수]
-- ==========================================
local function createKeySystemUI()
    local keyFrame = Instance.new("Frame")
    keyFrame.Name = "KeySystemFrame"
    keyFrame.Size = UDim2.new(0, 300, 0, 260)
    keyFrame.Position = UDim2.new(0.5, -150, 0.4, -130)
    keyFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    keyFrame.BorderSizePixel = 0
    keyFrame.Visible = true
    keyFrame.Parent = screenGui

    local uiCornerKey = Instance.new("UICorner")
    uiCornerKey.CornerRadius = UDim.new(0, 10)
    uiCornerKey.Parent = keyFrame

    local keyTitle = Instance.new("TextLabel")
    keyTitle.Size = UDim2.new(1, 0, 0, 35)
    keyTitle.BackgroundTransparency = 1
    keyTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    keyTitle.TextSize = 16
    keyTitle.Font = Enum.Font.SourceSansBold
    keyTitle.Text = "단어 헬퍼 전용 인증"
    keyTitle.Parent = keyFrame

    local keyBox = Instance.new("TextBox")
    keyBox.Size = UDim2.new(0, 260, 0, 32)
    keyBox.Position = UDim2.new(0.5, -130, 0, 38)
    keyBox.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    keyBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    keyBox.PlaceholderColor3 = Color3.fromRGB(150, 150, 150)
    keyBox.PlaceholderText = "비밀 키를 입력하세요..."
    keyBox.TextSize = 13
    keyBox.Font = Enum.Font.SourceSans
    keyBox.Text = ""
    keyBox.Parent = keyFrame

    local uiCornerBox = Instance.new("UICorner")
    uiCornerBox.CornerRadius = UDim.new(0, 6)
    uiCornerBox.Parent = keyBox

    local submitBtn = Instance.new("TextButton")
    submitBtn.Size = UDim2.new(0, 260, 0, 30)
    submitBtn.Position = UDim2.new(0.5, -130, 0, 76)
    submitBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
    submitBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    submitBtn.TextSize = 13
    submitBtn.Font = Enum.Font.SourceSansBold
    submitBtn.Text = "인증하기"
    submitBtn.Parent = keyFrame

    local uiCornerSub = Instance.new("UICorner")
    uiCornerSub.CornerRadius = UDim.new(0, 6)
    uiCornerSub.Parent = submitBtn

    local buyBtn = Instance.new("TextButton")
    buyBtn.Size = UDim2.new(0, 260, 0, 28)
    buyBtn.Position = UDim2.new(0.5, -130, 0, 112)
    buyBtn.BackgroundColor3 = Color3.fromRGB(88, 101, 242)
    buyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    buyBtn.TextSize = 12
    buyBtn.Font = Enum.Font.SourceSansBold
    buyBtn.Text = "전용 키 구매"
    buyBtn.Parent = keyFrame

    local uiCornerBuy = Instance.new("UICorner")
    uiCornerBuy.CornerRadius = UDim.new(0, 6)
    uiCornerBuy.Parent = buyBtn

    local buyProBtn = Instance.new("TextButton")
    buyProBtn.Size = UDim2.new(0, 260, 0, 28)
    buyProBtn.Position = UDim2.new(0.5, -130, 0, 146)
    buyProBtn.BackgroundColor3 = Color3.fromRGB(255, 140, 0)
    buyProBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    buyProBtn.TextSize = 12
    buyProBtn.Font = Enum.Font.SourceSansBold
    buyProBtn.Text = "프리미엄 전용 키 구매"
    buyProBtn.Parent = keyFrame

    local uiCornerBuyPro = Instance.new("UICorner")
    uiCornerBuyPro.CornerRadius = UDim.new(0, 6)
    uiCornerBuyPro.Parent = buyProBtn

    local statusLabel = Instance.new("TextLabel")
    statusLabel.Size = UDim2.new(1, 0, 0, 25)
    statusLabel.Position = UDim2.new(0, 0, 0, 180)
    statusLabel.BackgroundTransparency = 1
    statusLabel.TextColor3 = Color3.fromRGB(255, 80, 80)
    statusLabel.TextSize = 12
    statusLabel.Font = Enum.Font.SourceSansItalic
    statusLabel.Text = ""
    statusLabel.Parent = keyFrame

    local function copyDiscordLink()
        local discordLink = "https://discord.gg/ZKenYVezV"
        if setclipboard then
            setclipboard(discordLink)
        elseif toclipboard then
            toclipboard(discordLink)
        end
        statusLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
        statusLabel.Text = "디스코드 방에 들어와 구매하세요"
    end

    buyBtn.MouseButton1Click:Connect(copyDiscordLink)
    buyProBtn.MouseButton1Click:Connect(copyDiscordLink)

    submitBtn.MouseButton1Click:Connect(function()
        local playerName = localPlayer.Name:gsub("^%s*(.-)%s*$", "%1")
        local enteredKey = keyBox.Text:gsub("^%s*(.-)%s*$", "%1")
        
        -- 프리미엄 키 검증
        if premiumKeys[playerName] and premiumKeys[playerName] == enteredKey then
            savePremiumAuthSession()
            statusLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
            statusLabel.Text = "프리미엄 인증 성공! 12시간 유지."
            task.wait(0.8)
            keyFrame:Destroy()
            titleFrame.Visible = true
            remoteInputBox.Visible = true
        -- 일반 키 검증
        elseif userKeys[playerName] and userKeys[playerName] == enteredKey then
            saveAuthSession()
            statusLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
            statusLabel.Text = "인증 성공! 12시간 동안 유지됩니다."
            task.wait(0.8)
            keyFrame:Destroy()
            titleFrame.Visible = true
            remoteInputBox.Visible = false
        else
            statusLabel.TextColor3 = Color3.fromRGB(255, 80, 80)
            statusLabel.Text = "권한이 없거나 잘못된 키입니다."
        end
    end)
end

if not titleFrame.Visible then
    createKeySystemUI()
end

resetKeyBtn.MouseButton1Click:Connect(function()
    pcall(function()
        if delfile and isfile then
            if isfile(authFileName) then delfile(authFileName) end
            if isfile(premiumAuthFileName) then delfile(premiumAuthFileName) end
        end
    end)
    titleFrame.Visible = false
    remoteInputBox.Visible = false
    createKeySystemUI()
end)

-- ==========================================
-- [마우스 드래그 이동 로직]
-- ==========================================
local dragging = false
local dragInput, dragStart, startPos

titleFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = titleFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        titleFrame.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y
        )
    end
end)

-- ==========================================
-- [머리 위 무지개 헬퍼 유저 표시 시스템]
-- ==========================================
local function setupRainbowTag(character)
    if not character then return end
    local head = character:WaitForChild("Head", 5)
    if not head then return end

    if head:FindFirstChild("WordHelperRainbowTag") then return end

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "WordHelperRainbowTag"
    billboard.Size = UDim2.new(0, 200, 0, 50)
    billboard.StudsOffset = Vector3.new(0, 2.5, 0)
    billboard.AlwaysOnTop = true
    billboard.Adornee = head
    billboard.Parent = head

    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(1, 0, 1, 0)
    textLabel.BackgroundTransparency = 1
    textLabel.Text = "✨ 단어 헬퍼 스크립트 사용자 ✨"
    textLabel.TextSize = 14
    textLabel.Font = Enum.Font.SourceSansBold
    textLabel.Parent = billboard

    task.spawn(function()
        local hue = 0
        while billboard and billboard.Parent do
            hue = (hue + 0.01) % 1
            textLabel.TextColor3 = Color3.fromHSV(hue, 1, 1)
            task.wait(0.03)
        end
    end)
end

-- ==========================================
-- [스크립트 사용자 간 양방향 통신 네트워크]
-- ==========================================
local remoteFolderName = "WordHelperSyncNetwork"
local syncFolder = ReplicatedStorage:FindFirstChild(remoteFolderName)
if not syncFolder then
    pcall(function()
        syncFolder = Instance.new("Folder")
        syncFolder.Name = remoteFolderName
        syncFolder.Parent = ReplicatedStorage
    end)
end

local remoteEvent = syncFolder:FindFirstChild("RemoteWordEvent")
if not remoteEvent then
    pcall(function()
        remoteEvent = Instance.new("RemoteEvent")
        remoteEvent.Name = "RemoteWordEvent"
        remoteEvent.Parent = syncFolder
    end)
end

-- 프리미엄 인증된 유저만 전송 가능
remoteInputBox.FocusLost:Connect(function(enterPressed)
    if enterPressed and checkSavedPremiumAuth() then
        local typedWord = remoteInputBox.Text:gsub("^%s*(.-)%s*$", "%1")
        if typedWord ~= "" and remoteEvent then
            pcall(function()
                remoteEvent:FireServer(typedWord)
            end)
            answerLabel.Text = "프리미엄 전송됨: " .. typedWord
            remoteInputBox.Text = ""
        end
    end
end)

-- 단어 수신
if remoteEvent then
    remoteEvent.OnClientEvent:Connect(function(senderName, word)
        if senderName ~= localPlayer.Name then
            answerLabel.Text = "[" .. senderName .. "] 힌트: " .. word
            triggerAutoInput(word)
        end
    end)
end

local function registerMyPresence()
    pcall(function()
        local mySignal = syncFolder:FindFirstChild(localPlayer.Name)
        if not mySignal then
            mySignal = Instance.new("BoolValue")
            mySignal.Name = localPlayer.Name
            mySignal.Value = true
            mySignal.Parent = syncFolder
        end

        for _, p in ipairs(Players:GetPlayers()) do
            if syncFolder:FindFirstChild(p.Name) then
                if p.Character then
                    setupRainbowTag(p.Character)
                end
            end
        end

        Players.PlayerAdded:Connect(function(p)
            p.CharacterAdded:Connect(function(char)
                task.wait(1)
                if syncFolder:FindFirstChild(p.Name) then
                    setupRainbowTag(char)
                end
            end)
        end)

        syncFolder.ChildAdded:Connect(function(child)
            local targetPlayer = Players:FindFirstChild(child.Name)
            if targetPlayer and targetPlayer.Character then
                setupRainbowTag(targetPlayer.Character)
            end
        end)
    end)
end

registerMyPresence()

-- ==========================================
-- [단어 검증 및 정답 추출 로직]
-- ==========================================
local function isValidWord(txt)
    if not txt or type(txt) ~= "string" then return false end
    txt = txt:gsub("^%s*(.-)%s*$", "%1")
    
    if txt:find("%s") then return false end
    if #txt < 1 or #txt > 15 then return false end
    if tonumber(txt) ~= nil then return false end
    
    local lowerTxt = txt:lower()
    if lowerTxt:find("책") or lowerTxt:find("읽는") or lowerTxt:find("read") or lowerTxt:find("book") or 
       lowerTxt:find("_") or lowerTxt:find("r$") or lowerTxt:find("robux") or 
       lowerTxt:find("대기") or lowerTxt:find("라운드") or lowerTxt:find("kucing") then 
        return false 
    end
    
    return true
end

local function checkRoundReset(txt)
    if not txt or type(txt) ~= "string" then return false end
    local low = txt:lower()
    if low:find("대기") or low:find("라운드") or low:find("시작") or low:find("ready") or low:find("wait") or low:find("end") then
        return true
    end
    return false
end

pcall(function()
    for _, v in ipairs(ReplicatedStorage:GetDescendants()) do
        if v:IsA("RemoteEvent") or v:IsA("UnreliableRemoteEvent") and v.Name ~= "RemoteWordEvent" then
            v.OnClientEvent:Connect(function(...)
                local args = {...}
                for _, arg in ipairs(args) do
                    if type(arg) == "string" then
                        if checkRoundReset(arg) then
                            answerLabel.Text = "정답: 라운드 대기 중..."
                        elseif isValidWord(arg) then
                            answerLabel.Text = "정답: " .. arg
                            triggerAutoInput(arg)
                        end
                    elseif type(arg) == "table" then
                        for _, subArg in pairs(arg) do
                            if type(subArg) == "string" then
                                if checkRoundReset(subArg) then
                                    answerLabel.Text = "정답: 라운드 대기 중..."
                                elseif isValidWord(subArg) then
                                    answerLabel.Text = "정답: " .. subArg
                                    triggerAutoInput(subArg)
                                end
                            end
                        end
                    end
                end
            end)
        end
    end
end)

task.spawn(function()
    while true do
        task.wait(0.5)
        pcall(function()
            for _, obj in ipairs(ReplicatedStorage:GetDescendants()) do
                if obj:IsA("StringValue") then
                    local val = obj.Value
                    if checkRoundReset(val) then
                        answerLabel.Text = "정답: 라운드 대기 중..."
                        break
                    elseif isValidWord(val) then
                        answerLabel.Text = "정답: " .. val
                        triggerAutoInput(val)
                        break
                    end
                end
            end
        end)
    end
end)
