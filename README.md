-- [[ kill bypass ]]
local P = game:GetService("Players").LocalPlayer
local R = game:GetService("RunService")
local C = game:GetService("CoreGui")
local S = game:GetService("StarterGui")

-- 既存プロセスのクリーンアップ
if C:FindFirstChild("KillBypass_Final_Solution") then C.KillBypass_Final_Solution:Destroy() end

-- GUIの構築
local G = Instance.new("ScreenGui", C); G.Name = "KillBypass_Final_Solution"
local B = Instance.new("TextButton", G)
B.Size = UDim2.new(0, 250, 0, 60); B.Position = UDim2.new(0, 20, 0.8, 0)
B.Text = "kill bypass: OFF"; B.BackgroundColor3 = Color3.fromRGB(20, 20, 20); B.TextColor3 = Color3.new(1, 1, 1)
B.Font = Enum.Font.Code; B.TextSize = 22; B.BorderSizePixel = 4

local Active = false
local HOME_POS = Vector3.new(0, 50, 0)

-- [[ 死んだ時のみ右下に通知を出す専用ロジック ]]
local function OnDeath()
    if not Active then return end
    task.spawn(function()
        S:SetCore("SendNotification", {
            Title = "🚨 DEATH DETECTED",
            Text = "Character has been reset. Re-applying bypass...",
            Duration = 5
        })
    end)
end

-- [[ キャラクター設定：ラグドール化と監視 ]]
local function Setup(char)
    if not Active then return end
    
    local hum = char:WaitForChild("Humanoid", 10)
    if hum then
        -- 死亡イベントの紐付け
        hum.Died:Connect(OnDeath)
        hum:ChangeState(Enum.HumanoidStateType.Physics)
        hum:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
    end

    -- 核の消失を監視（サーバーによる消去対策）
    char.ChildRemoved:Connect(function(child)
        if Active and child.Name == "HumanoidRootPart" then OnDeath() end
    end)

    -- 関節をパージ（ラグドール状態の生成）
    for _, m in pairs(char:GetDescendants()) do
        if m:IsA("Motor6D") then m.Enabled = false end
    end
end

P.CharacterAdded:Connect(Setup)

B.MouseButton1Click:Connect(function()
    Active = not Active
    if Active then
        if P.Character then 
            HOME_POS = P.Character.HumanoidRootPart.Position
            Setup(P.Character) 
        end
        B.Text = "kill bypass: ON"; B.BackgroundColor3 = Color3.fromRGB(255, 0, 0); B.TextColor3 = Color3.new(0, 0, 0)
    else
        B.Text = "kill bypass: OFF"; B.BackgroundColor3 = Color3.fromRGB(20, 20, 20); B.TextColor3 = Color3.new(1, 1, 1)
    end
end)

-- [[ 物理演算：不滅と超広範囲分散の並列処理 ]]
R.Heartbeat:Connect(function()
    if not Active then return end
    local Char = P.Character
    local Hum = Char and Char:FindFirstChildOfClass("Humanoid")
    local Root = Char and Char:FindFirstChild("HumanoidRootPart")
    
    if Hum then
        -- 死の判定をパケットレベルで上書き（120連）
        for i = 1, 120 do
            Hum.Health = 100
            Hum:ChangeState(Enum.HumanoidStateType.Physics)
            Hum:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
        end
    end

    if Root then
        -- 「The Worst」のAnti-Voidに消されない地上高度に固定
        Root.CFrame = CFrame.new(HOME_POS.X, HOME_POS.Y + 3, HOME_POS.Z)
        Root.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
    end
end)

R.PreRender:Connect(function()
    if not Active then return end
    local Char = P.Character
    local Cam = workspace.CurrentCamera
    if not Char or not Cam then return end

    -- カメラを地上の中心位置に固定
    local anchor = Cam:FindFirstChild("FinalAnchor") or Instance.new("Part", Cam)
    anchor.Name = "FinalAnchor"; anchor.Transparency = 1; anchor.Anchored = true; anchor.CanCollide = false
    anchor.CFrame = CFrame.new(HOME_POS)
    Cam.CameraSubject = anchor

    -- 【超広範囲・不規則・カオス旋回】
    local parts = {}
    for _, p in pairs(Char:GetChildren()) do 
        if p:IsA("BasePart") and p.Name ~= "HumanoidRootPart" then table.insert(parts, p) end 
    end

    for i, p in ipairs(parts) do
        local t = os.clock()
        -- 半径250〜350スタッドを不規則に伸縮しながら高速回転
        local angle = (t * 24) + (i * (math.pi * 2 / #parts))
        local radius = 300 + math.sin(t * 6 + i) * 100
        local height = math.sin(t * 12 + i) * 35 + 15
        
        local pos = HOME_POS + Vector3.new(
            math.cos(angle) * radius,
            height,
            math.sin(angle) * radius
        )

        p.CanCollide = false
        p.CFrame = CFrame.new(pos)
        -- 物理カウンター
        p.RotVelocity = Vector3.new(1e10, 1e10, 1e10)
    end
end)
