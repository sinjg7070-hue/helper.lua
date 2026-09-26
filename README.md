-- 안전한 서비스 불러오기
local CoreGui = game:GetService("CoreGui")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
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
-- 💡 [유저별 맞춤형 키 시스템 설정]
-- ==========================================
local userKeys = {
    ["dambii522"] = "no.1keyap191929",
    ["zxxdaswo"] = "no.1keyap19293949",
    ["1CasaNova6974"] = "no.1keyap172737",
    ["dohunpoop"] = "dohunpoop_key12"
}

-- 12시간 인증 유지 파일 이름 (유저별로 구분)
local safePlayerName = localPlayer.Name:gsub("[^%w]", "_")
local authFileName = "WordHelper_Auth_" .. safePlayerName .. ".txt"

-- 12시간 유효성 검사 함수
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

-- 인증 정보 저장 함수 (12시간 = 43200초)
local function saveAuthSession()
    if writefile then
        pcall(function()
            local expireTime = os.time() + 43200
            writefile(authFileName, tostring(expireTime))
        end)
    end
end

-- ==========================================
-- 💡 [메인 헬퍼 UI 생성]
-- ==========================================
local titleFrame = Instance.new("TextButton")
titleFrame.Name = "TitleFrame"
titleFrame.Size = UDim2.new(0, 220, 0, 70)
titleFrame.Position = UDim2.new(0.75, 0, 0.1, 0)
titleFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
titleFrame.TextColor3 = Color3.fromRGB(255, 255, 255)
titleFrame.TextSize = 15
titleFrame.Font = Enum.Font.SourceSansBold
titleFrame.Text = "\n단어 맞히기 헬퍼 🖱️"
titleFrame.AutoButtonColor = false
titleFrame.Visible = checkSavedAuth() 
titleFrame.Parent = screenGui

local uiCornerBtn = Instance.new("UICorner")
uiCornerBtn.CornerRadius = UDim.new(0, 8)
uiCornerBtn.Parent = titleFrame

-- 개발자 표시 텍스트 라벨
local devLabel = Instance.new("TextLabel")
devLabel.Name = "DevLabel"
devLabel.Size = UDim2.new(1, 0, 0, 20)
devLabel.Position = UDim2.new(0, 0, 0, 5)
devLabel.BackgroundTransparency = 1
devLabel.TextColor3 = Color3.fromRGB(170, 170, 170)
devLabel.TextSize = 12
devLabel.Font = Enum.Font.SourceSansItalic
devLabel.Text = "스크립트 개발자 : 지환"
devLabel.Parent = titleFrame

-- 정답창 (TextLabel)
local answerLabel = Instance.new("TextLabel")
answerLabel.Name = "AnswerLabel"
answerLabel.Size = UDim2.new(0, 220, 0, 45)
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

-- ==========================================
-- 💡 [키 인증 프레임 생성 (인증 안 되었을 때만 표시)]
-- ==========================================
local keyFrame
if not titleFrame.Visible then
    keyFrame = Instance.new("Frame")
    keyFrame.Name = "KeySystemFrame"
    keyFrame.Size = UDim2.new(0, 300, 0, 215)
    keyFrame.Position = UDim2.new(0.5, -150, 0.4, -107)
    keyFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
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
    keyTitle.TextSize = 16
    keyTitle.Font = Enum.Font.SourceSansBold
    keyTitle.Text = "🔑 단어 헬퍼 전용 인증"
    keyTitle.Parent = keyFrame

    -- 키 입력 박스
    local keyBox = Instance.new("TextBox")
    keyBox.Size = UDim2.new(0, 260, 0, 35)
    keyBox.Position = UDim2.new(0.5, -130, 0, 45)
    keyBox.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    keyBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    keyBox.PlaceholderColor3 = Color3.fromRGB(150, 150, 150)
    keyBox.PlaceholderText = "비밀 키를 입력하세요..."
    keyBox.TextSize = 14
    keyBox.Font = Enum.Font.SourceSans
    keyBox.Text = ""
    keyBox.Parent = keyFrame

    local uiCornerBox = Instance.new("UICorner")
    uiCornerBox.CornerRadius = UDim.new(0, 6)
    uiCornerBox.Parent = keyBox

    -- 인증하기 버튼
    local submitBtn = Instance.new("TextButton")
    submitBtn.Size = UDim2.new(0, 260, 0, 32)
    submitBtn.Position = UDim2.new(0.5, -130, 0, 88)
    submitBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
    submitBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    submitBtn.TextSize = 14
    submitBtn.Font = Enum.Font.SourceSansBold
    submitBtn.Text = "인증하기"
    submitBtn.Parent = keyFrame

    local uiCornerSub = Instance.new("UICorner")
    uiCornerSub.CornerRadius = UDim.new(0, 6)
    uiCornerSub.Parent = submitBtn

    -- 전용 키 구매 버튼
    local buyBtn = Instance.new("TextButton")
    buyBtn.Size = UDim2.new(0, 260, 0, 32)
    buyBtn.Position = UDim2.new(0.5, -130, 0, 128)
    buyBtn.BackgroundColor3 = Color3.fromRGB(88, 101, 242) -- 디스코드 색상
    buyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    buyBtn.TextSize = 14
    buyBtn.Font = Enum.Font.SourceSansBold
    buyBtn.Text = "🛒 전용 키 구매"
    buyBtn.Parent = keyFrame

    local uiCornerBuy = Instance.new("UICorner")
    uiCornerBuy.CornerRadius = UDim.new(0, 6)
    uiCornerBuy.Parent = buyBtn

    -- 상태 안내 메시지 라벨
    local statusLabel = Instance.new("TextLabel")
    statusLabel.Size = UDim2.new(1, 0, 0, 25)
    statusLabel.Position = UDim2.new(0, 0, 0, 168)
    statusLabel.BackgroundTransparency = 1
    statusLabel.TextColor3 = Color3.fromRGB(255, 80, 80)
    statusLabel.TextSize = 12
    statusLabel.Font = Enum.Font.SourceSansItalic
    statusLabel.Text = ""
    statusLabel.Parent = keyFrame

    -- 키 구매 버튼 클릭 이벤트 (클립보드 복사)
    buyBtn.MouseButton1Click:Connect(function()
        local discordLink = "https://discord.gg/ZKenYVezV"
        if setclipboard then
            setclipboard(discordLink)
        elseif toclipboard then
            toclipboard(discordLink)
        end
        statusLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
        statusLabel.Text = "디스코드 방에 들어와 구매하세요"
    end)

    -- 키 검증 로직
    submitBtn.MouseButton1Click:Connect(function()
        local playerName = localPlayer.Name:gsub("^%s*(.-)%s*$", "%1")
        local enteredKey = keyBox.Text:gsub("^%s*(.-)%s*$", "%1")
        
        if userKeys[playerName] and userKeys[playerName] == enteredKey then
            saveAuthSession()
            statusLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
            statusLabel.Text = "인증 성공! 12시간 동안 유지됩니다."
            task.wait(0.8)
            keyFrame:Destroy()
            titleFrame.Visible = true
        else
            statusLabel.TextColor3 = Color3.fromRGB(255, 80, 80)
            statusLabel.Text = "권한이 없거나 잘못된 키입니다."
        end
    end)
end

-- ==========================================
-- 💡 [마우스 드래그 이동 로직]
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

-- 텍스트 순수 단어 검증 함수 (책, 읽는 등 불필요한 시스템 문구 강력 차단)
local function isValidWord(txt)
    if not txt or type(txt) ~= "string" then return false end
    txt = txt:gsub("^%s*(.-)%s*$", "%1")
    if #txt < 2 or #txt > 15 then return false end
    if tonumber(txt) ~= nil then return false end
    
    -- 제외할 키워드 필터 (책, 읽는, 대화, 시스템 텍스트 등 방지)
    local lowerTxt = txt:lower()
    if lowerTxt:find("책") or lowerTxt:find("읽는") or lowerTxt:find("read") or lowerTxt:find("book") or 
       lowerTxt:find("_") or lowerTxt:find("r$") or lowerTxt:find("robux") or 
       lowerTxt:find("대기") or lowerTxt:find("라운드") or lowerTxt:find("kucing") then 
        return false 
    end
    
    return true
end

-- 서버 통신 감지 (정답 추출)
pcall(function()
    for _, v in ipairs(ReplicatedStorage:GetDescendants()) do
        if v:IsA("RemoteEvent") or v:IsA("UnreliableRemoteEvent") then
            v.OnClientEvent:Connect(function(...)
                local args = {...}
                for _, arg in ipairs(args) do
                    if type(arg) == "string" and isValidWord(arg) then
                        answerLabel.Text = "정답: " .. arg
                    elseif type(arg) == "table" then
                        for _, subArg in pairs(arg) do
                            if type(subArg) == "string" and isValidWord(subArg) then
                                answerLabel.Text = "정답: " .. subArg
                            end
                        end
                    end
                end
            end)
        end
    end
end)

-- 백업 주기적 스캔
task.spawn(function()
    while true do
        task.wait(0.5)
        pcall(function()
            for _, obj in ipairs(ReplicatedStorage:GetDescendants()) do
                if obj:IsA("StringValue") then
                    local val = obj.Value
                    if isValidWord(val) then
                        answerLabel.Text = "정답: " .. val
                        break
                    end
                end
            end
        end)
    end
end)
