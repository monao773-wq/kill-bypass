-- [[ Kill Bypass (Ultimate Edition) ]]
local P = game:GetService("Players").LocalPlayer
local R = game:GetService("RunService")
local C = game:GetService("CoreGui")
local S = game:GetService("StarterGui")

-- 重複起動の防止
if C:FindFirstChild("KillBypass_Ultimate") then C.KillBypass_Ultimate:Destroy() end

local G = Instance.new("ScreenGui", C); G.Name = "KillBypass_Ultimate"
local B = Instance.new("TextButton", G)
B.Size = UDim2.new(0, 220, 0, 55); B.Position = UDim2.new(0, 15, 0.8, 0)
B.Text = "Kill Bypass: READY"; B.BackgroundColor3 = Color3.fromRGB(15, 15, 15); B.TextColor3 = Color3.new(1, 1, 1)
B.Font = Enum.Font.Code; B.TextSize = 19; B.BorderSizePixel = 2

local Active = false
local ABYSS_Y = -900000 
local WORLD_DOMAIN = 55000 

-- 高精度システム通知
local function Notify(msg)
    S:SetCore("SendNotification", {
        Title = "Kill Bypass";
        Text = msg;
        Duration = 4;
        Button1 = "OK";
    })
end

B.MouseButton1Click:Connect(function()
    Active = not Active
    B.Text = Active and "Kill Bypass: ON" or "Kill Bypass: OFF"
    B.BackgroundColor3 = Active and Color3.fromRGB(0, 180, 100) or Color3.fromRGB(15, 15, 15)
    Notify(Active and "Immortal Protocol Activated" or "Bypass Standby")
end)

-- [[ 1. 論理防御：超高密度ラグドール・インジェクション ]]
R.PreSimulation:Connect(function()
    if not Active then return end
    local Char = P.Character
    local Hum = Char and Char:FindFirstChildOfClass("Humanoid")
    if Hum then
        -- 死亡判定をリアルタイム監視して通知
        if Hum.Health <= 0 then
            Notify("CRITICAL: Death Detected. Resetting...")
        end
        
        task.spawn(function()
            -- 120回ループで「死」を物理的に塗りつぶす
            for i = 1, 120 do 
                Hum.Health = 100
                Hum:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
                -- 核心：ラグドール固定ループ
                Hum:ChangeState(Enum.HumanoidStateType.Physics)
                Hum:SetStateEnabled(Enum.HumanoidStateType.GettingUp, false)
            end
        end)
    end
end)

-- [[ 2. 物理防御 ＆ 反撃：カオス・ベクトル・フィールド ]]
R.Stepped:Connect(function()
    if not Active then return end
    local Char = P.Character
    if not Char then return end
    
    -- 関節を完全に解体
    for _, j in pairs(Char:GetDescendants()) do
        if j:IsA("Motor6D") or j:IsA("Weld") then j.Enabled = false end
    end
    
    local parts = {}
    for _, p in pairs(Char:GetChildren()) do if p:IsA("BasePart") then table.insert(parts, p) end end

    local t = os.clock()
    -- カオス座標：相手のエイムを粉砕する不規則スパイラル
    local angle = t * 120
    local posX = math.sin(angle) * WORLD_DOMAIN
    local posZ = math.cos(angle) * WORLD_DOMAIN
    
    for i, part in ipairs(parts) do
        part.CanTouch = true -- カウンターを有効化
        
        -- 相手をフリーズさせる物理エネルギー (1e20: 描画安定化Ver)
        part.AssemblyLinearVelocity = Vector3.new(1e20, 1e20, 1e20)
        part.RotVelocity = Vector3.new(1e20, 1e20, 1e20)

        if part.Name == "HumanoidRootPart" then
            -- 【核】命の隔離
            part.CFrame = CFrame.new(posX, ABYSS_Y, posZ)
        else
            -- 【トラップ】0.01秒ごとに地上と奈落を反転
            -- 攻撃してきた相手の座標を強制的にバグらせて弾き飛ばす
            if (t * 200) % 2 < 1 then
                part.CFrame = CFrame.new(posX + (i*10), 40, posZ + (i*10)) 
            else
                part.CFrame = CFrame.new(posX, ABYSS_Y + 1000, posZ)
            end
        end
    end
end)
