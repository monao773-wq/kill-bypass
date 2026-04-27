-- [[ kill bypass ]]
local P = game:GetService("Players").LocalPlayer
local R = game:GetService("RunService")
local C = game:GetService("CoreGui")
local S = game:GetService("StarterGui")

-- 重複プロセスのクリーンアップ
if C:FindFirstChild("KillBypass_Ultimate_Omega") then C.KillBypass_Ultimate_Omega:Destroy() end

local G = Instance.new("ScreenGui", C); G.Name = "KillBypass_Ultimate_Omega"
local B = Instance.new("TextButton", G)
B.Size = UDim2.new(0, 250, 0, 60); B.Position = UDim2.new(0, 20, 0.8, 0)
B.Text = "kill bypass: OFF"; B.BackgroundColor3 = Color3.fromRGB(10, 10, 10); B.TextColor3 = Color3.new(1, 1, 1)
B.Font = Enum.Font.Code; B.TextSize = 22; B.BorderSizePixel = 4

local Active = false
local ABYSS_Y = -900000 -- 奈落の基準値
local GROUND_POS = Vector3.new(0, 50, 0)

-- [[ 死んだ時のみ右下に通知を出すロジック ]]
local function OnDeath()
    if not Active then return end
    task.spawn(function()
        S:SetCore("SendNotification", {
            Title = "🚨 DEATH DETECTED",
            Text = "Core was reset. Re-isolating limbs...",
            Duration = 5
        })
    end)
end

-- [[ キャラクターの物理解体と監視設定 ]]
local function Setup(char)
    if not Active then return end
    local hum = char:WaitForChild("Humanoid", 10)
    if hum then
        hum.Died:Connect(OnDeath)
        hum:ChangeState(Enum.HumanoidStateType.Physics)
        hum:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
    end
    -- 関節（Motor6D）を完全に破壊。Enabled = falseでは残像が残るためDestroy。
    for _, m in pairs(char:GetDescendants()) do
        if m:IsA("Motor6D") then m:Destroy() end
    end
end

P.CharacterAdded:Connect(Setup)

B.MouseButton1Click:Connect(function()
    Active = not Active
    if Active and P.Character then 
        GROUND_POS = P.Character.HumanoidRootPart.Position
        Setup(P.Character) 
    end
    B.Text = Active and "kill bypass: ON" or "kill bypass: OFF"
    B.BackgroundColor3 = Active and Color3.fromRGB(255, 0, 0) or Color3.fromRGB(10, 10, 10)
    B.TextColor3 = Active and Color3.new(0, 0, 0) or Color3.new(1, 1, 1)
end)

-- [[ 1. 不滅：核（RootPart）の隠蔽固定 ]]
R.Heartbeat:Connect(function()
    if not Active then return end
    local Char = P.Character
    local Hum = Char and Char:FindFirstChildOfClass("Humanoid")
    local Root = Char and Char:FindFirstChild("HumanoidRootPart")
    
    if Hum then
        -- サーバー側のキル判定を圧倒するパケット上書き
        for i = 1, 120 do
            Hum.Health = 100
            Hum:ChangeState(Enum.HumanoidStateType.Physics)
            Hum:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
        end
    end

    if Root then
        -- 核を足元50スタッド下に隔離。Anti-Voidに引っかからず、手も届かない。
        Root.CFrame = CFrame.new(GROUND_POS.X, GROUND_POS.Y - 50, GROUND_POS.Z)
        Root.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
    end
end)

-- [[ 2. 超空間分散：地上には「頭」のみ、体は10万スタッドずつ離してパージ ]]
R.PreRender:Connect(function()
    if not Active then return end
    local Char = P.Character
    local Cam = workspace.CurrentCamera
    if not Char or not Cam then return end

    -- カメラを地上の頭の位置に同期
    local Head = Char:FindFirstChild("Head")
    if Head then Cam.CameraSubject = Head end

    local parts = {}
    for _, p in pairs(Char:GetChildren()) do
        if p:IsA("BasePart") and p.Name ~= "HumanoidRootPart" then
            table.insert(parts, p)
        end
    end

    for i, p in ipairs(parts) do
        if p.Name == "Head" then
            -- 【頭】地上に維持しつつ、0.002秒間隔のワープで掴みを拒否
            local t = os.clock()
            if (t * 500) % 2 < 1 then
                p.CFrame = CFrame.new(GROUND_POS)
                p.RotVelocity = Vector3.new(1e10, 1e10, 1e10)
            else
                p.CFrame = CFrame.new(0, ABYSS_Y, 0)
            end
        else
            -- 【胴体・手足】10万スタッドずつ座標を離して奈落へパージ
            -- 胴体にルートパーツは付随せず、物理的に存在しないも同然の状態
            local offset = i * 100000
            p.CFrame = CFrame.new(offset, ABYSS_Y, offset)
            p.CanCollide = false
            p.CanTouch = false
            p.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
        end
    end
end)
