-- ======================================================================
-- 📨 DUNGEON QUEST - AUTO SPAM JOIN REQUEST (SMART COOLDOWN EDITION)
-- ======================================================================

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui", 15)
local Remotes = ReplicatedStorage:FindFirstChild("remotes") or ReplicatedStorage:WaitForChild("remotes", 10)

-- Đọc cấu hình từ getgenv() hoặc _G
local function GetConfig(key, default)
    if getgenv and getgenv()[key] ~= nil then
        return getgenv()[key]
    elseif _G and _G[key] ~= nil then
        return _G[key]
    end
    return default
end

local TargetUser = GetConfig("TargetUser", "090114755999")
local AutoSpam   = true
if getgenv and getgenv().AutoSpam ~= nil then
    AutoSpam = getgenv().AutoSpam
end

local DelayTime  = GetConfig("Delay", 4.5) -- ⚠️ Server Dungeon Quest cooldown ~4.5 - 5s

local requestCount = 0
local lastStatus = "Sẵn sàng"

-- ======================================================================
-- 🔔 HÀM HIỆN THÔNG BÁO CHUẨN CỦA GAME (ALERT BOX)
-- ======================================================================
local function ShowInGameAlert(text)
    pcall(function()
        local alertBox = PlayerGui:FindFirstChild("alertBox")
        if alertBox and alertBox:FindFirstChild("Frame") then
            local alertTemplate = (ReplicatedStorage:FindFirstChild("ui") and ReplicatedStorage.ui:FindFirstChild("alert")) 
                               or ReplicatedStorage:FindFirstChild("alert")
            if alertTemplate then
                local clone = alertTemplate:Clone()
                local lbl = clone:FindFirstChild("TextLabel")
                if lbl then
                    lbl.Text = tostring(text)
                end
                clone.Parent = alertBox.Frame
            end
        end
    end)
end

-- ======================================================================
-- ⚡ VÒNG LẶP AUTO SPAM REQUEST (TỰ ĐỘNG BÙ COOLDOWN SERVER)
-- ======================================================================
task.spawn(function()
    while true do
        TargetUser = GetConfig("TargetUser", TargetUser)
        if getgenv and getgenv().AutoSpam ~= nil then
            AutoSpam = getgenv().AutoSpam
        end
        
        if AutoSpam and TargetUser and TargetUser ~= "" then
            pcall(function()
                if Remotes and Remotes:FindFirstChild("sendJoinRequest") then
                    local ok, isSent, serverMsg = pcall(function()
                        return Remotes.sendJoinRequest:InvokeServer(TargetUser)
                    end)
                    
                    if ok then
                        local displayMsg = serverMsg or (isSent and "Your request has been sent." or "Couldn't send request.")
                        
                        if isSent then
                            requestCount = requestCount + 1
                            lastStatus = "✅ [#" .. tostring(requestCount) .. "] " .. tostring(displayMsg)
                            ShowInGameAlert(displayMsg)
                            task.wait(DelayTime or 4.5)
                        else
                            lastStatus = "⏳ " .. tostring(displayMsg)
                            -- Nếu Server báo đang chờ cooldown, đợi thêm 2 giây
                            if tostring(displayMsg):find("wait a moment") then
                                task.wait(2.0)
                            else
                                task.wait(DelayTime or 4.5)
                            end
                        end
                    else
                        lastStatus = "❌ Lỗi kết nối Remote"
                        task.wait(3.0)
                    end
                else
                    task.wait(2.0)
                end
            end)
        else
            task.wait(1.0)
        end
    end
end)

-- ======================================================================
-- 🖥️ MINI GUI HIỂN THỊ TRẠNG THÁI & PHẢN HỒI THẬT
-- ======================================================================
if PlayerGui:FindFirstChild("DQ_JoinSpammerUI") then
    PlayerGui.DQ_JoinSpammerUI:Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "DQ_JoinSpammerUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = PlayerGui

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 320, 0, 195)
MainFrame.Position = UDim2.new(0.5, -160, 0.2, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(22, 24, 30)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0, 10)
Corner.Parent = MainFrame

local Stroke = Instance.new("UIStroke")
Stroke.Color = Color3.fromRGB(99, 102, 241)
Stroke.Thickness = 1.5
Stroke.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -40, 0, 35)
Title.Position = UDim2.new(0, 12, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "📨 Auto Join Spammer (Anti-Cooldown)"
Title.TextColor3 = Color3.fromRGB(240, 240, 245)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 13
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = MainFrame

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 26, 0, 26)
CloseBtn.Position = UDim2.new(1, -32, 0, 5)
CloseBtn.BackgroundColor3 = Color3.fromRGB(35, 38, 48)
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.fromRGB(180, 180, 190)
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 12
CloseBtn.Parent = MainFrame

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 6)
CloseCorner.Parent = CloseBtn

CloseBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

local UserBox = Instance.new("TextBox")
UserBox.Size = UDim2.new(1, -24, 0, 34)
UserBox.Position = UDim2.new(0, 12, 0, 40)
UserBox.BackgroundColor3 = Color3.fromRGB(30, 33, 44)
UserBox.TextColor3 = Color3.fromRGB(255, 255, 255)
UserBox.Text = tostring(TargetUser)
UserBox.PlaceholderText = "Nhập username mục tiêu..."
UserBox.PlaceholderColor3 = Color3.fromRGB(130, 135, 150)
UserBox.Font = Enum.Font.Gotham
UserBox.TextSize = 13
UserBox.ClearTextOnFocus = false
UserBox.Parent = MainFrame

local UserBoxCorner = Instance.new("UICorner")
UserBoxCorner.CornerRadius = UDim.new(0, 6)
UserBoxCorner.Parent = UserBox

UserBox.FocusLost:Connect(function()
    TargetUser = UserBox.Text:gsub("%s+", "")
    if getgenv then getgenv().TargetUser = TargetUser end
    _G.TargetUser = TargetUser
    requestCount = 0
    lastStatus = "Đã đổi sang: @" .. TargetUser
end)

local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Size = UDim2.new(1, -24, 0, 34)
ToggleBtn.Position = UDim2.new(0, 12, 0, 82)
ToggleBtn.BackgroundColor3 = AutoSpam and Color3.fromRGB(34, 197, 94) or Color3.fromRGB(239, 68, 68)
ToggleBtn.Text = AutoSpam and "🟢 ĐANG BẬT TỰ ĐỘNG GỬI" or "🔴 ĐÃ TẮT (BẤM ĐỂ BẬT)"
ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleBtn.Font = Enum.Font.GothamBold
ToggleBtn.TextSize = 13
ToggleBtn.Parent = MainFrame

local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(0, 6)
ToggleCorner.Parent = ToggleBtn

ToggleBtn.MouseButton1Click:Connect(function()
    AutoSpam = not AutoSpam
    if getgenv then getgenv().AutoSpam = AutoSpam end
    _G.AutoSpam = AutoSpam
    
    if AutoSpam then
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
        ToggleBtn.Text = "🟢 ĐANG BẬT TỰ ĐỘNG GỬI"
        requestCount = 0
        lastStatus = "Bắt đầu gửi tới @" .. tostring(TargetUser)
    else
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
        ToggleBtn.Text = "🔴 ĐÃ TẮT (BẤM ĐỂ BẬT)"
        lastStatus = "Đã tạm dừng"
    end
end)

local StatusLabel = Instance.new("TextLabel")
StatusLabel.Size = UDim2.new(1, -24, 0, 50)
StatusLabel.Position = UDim2.new(0, 12, 0, 124)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Text = "Trạng thái: " .. lastStatus .. "\n(Phím tắt UI: Right Control)"
StatusLabel.TextColor3 = Color3.fromRGB(150, 160, 185)
StatusLabel.Font = Enum.Font.Gotham
StatusLabel.TextSize = 11
StatusLabel.TextWrapped = true
StatusLabel.TextXAlignment = Enum.TextXAlignment.Left
StatusLabel.Parent = MainFrame

task.spawn(function()
    while true do
        task.wait(0.3)
        if StatusLabel and StatusLabel.Parent then
            StatusLabel.Text = "Trạng thái: " .. lastStatus .. "\n(Phím tắt UI: Right Control)"
        end
        if UserBox and UserBox.Parent and not UserBox:IsFocused() then
            if UserBox.Text ~= TargetUser then
                UserBox.Text = tostring(TargetUser)
            end
        end
    end
end)

-- Dragging
local dragging, dragInput, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)
MainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

UserInputService.InputBegan:Connect(function(input, gpe)
    if not gpe and input.KeyCode == Enum.KeyCode.RightControl then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

print("📨 [Auto Join Spammer] Ready! Target: @" .. tostring(TargetUser))