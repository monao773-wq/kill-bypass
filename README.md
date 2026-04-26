-- [[ Kill Bypass (Singularity Edition) ]]
local P = game:GetService("Players").LocalPlayer
local R = game:GetService("RunService")
local C = game:GetService("CoreGui")
local S = game:GetService("StarterGui")

-- プロセスの重複排除
if C:FindFirstChild("KillBypass_Singularity") then C.KillBypass_Singularity:Destroy() end

local G = Instance.new("ScreenGui", C); G.Name = "KillBypass_Singularity"
local B = Instance.new("TextButton", G)
B.Size = UDim2.new(0, 240, 0, 60); B.Position = UDim2.new(0, 10, 0.8, 0)
B.Text = "Kill Bypass: READY"; B.BackgroundColor3 = Color3.fromRGB(0, 0, 0); B.TextColor3 = Color3.new(0, 1, 1)
B.Font = Enum.Font.Code; B.TextSize = 20; B.BorderSizePixel = 2

local Active = false
local ABYSS_Y = -950000 -- キル判定を無視する深淵
local WORLD_RADIUS = 80000 -- 相手のカメラをバグらせる爆速移動範囲

local function Notify(msg)
    S:SetCore("SendNotification", {
        Title = "Kill Bypass";
        Text = msg;
        Duration = 3;
    })
end

B.MouseButton1Click:Connect(function()
    Active = not Active
    B.Text = Active and "Kill Bypass: ON" or "Kill Bypass: READY"
    B.BackgroundColor3 = Active and Color3.fromRGB(50, 0, 100) or Color3.fromRGB(0, 0, 0)
    Notify(Active and "Bypass Protocol: ONLINE" or "Bypass: Standby")
end)

-- [[ 1. 論理防御：超高密度ラグドール・ループ ]]
R.PreSimulation:Connect(function()
    if not Active then return end
    local Char = P.Character
    local Hum = Char and Char:FindFirstChildOfClass("Humanoid")
    if Hum then
        -- 死亡・場外・掴みステートを1フレーム120連で上書き
        task.spawn(function()
            for i = 1, 120 do 
                Hum.Health = 100
                Hum:ChangeState(Enum.HumanoidStateType.Physics) -- ヒント：ラグドールループ
                Hum:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
                Hum:SetStateEnabled(Enum.HumanoidStateType.GettingUp, false)
            end
        end)
    end
end)

-- [[ 2. 物理反撃：アンチ・グラブ ＆ クライアント・クラッシャー ]]
R.Stepped:Connect(function()
    if not Active then return end
    local Char = P.Character
    if not Char then return end
    
    -- 関節の切断（相手が掴もうとした瞬間にパーツを分解して逃がす）
    for _, j in pairs(Char:GetDescendants()) do
        if j:IsA("Motor6D") or j:IsA("Weld") then j.Enabled = false end
    end
    
    local parts = {}
    for _, p in pairs(Char:GetChildren()) do if p:IsA("BasePart") then table.insert(parts, p) end end

    local t = os.clock()
    -- カオス座標：スパイラル移動でエイムとキルオーラを無力化
    local posX = math.sin(t * 120) * WORLD_RADIUS
    local posZ = math.cos(t * 120) * WORLD_RADIUS
    
    for i, part in ipairs(parts) do
        part.CanTouch = true -- カウンター有効化
        
        -- 相手をバグらせる物理エネルギー (1e25: クラッシュ性能重視)
        part.AssemblyLinearVelocity = Vector3.new(1e25, 1e25, 1e25)
        part.RotVelocity = Vector3.new(1e25, 1e25, 1e25)

        if part.Name == "HumanoidRootPart" then
            -- 【核】命の隔離。視点はここ。
            part.CFrame = CFrame.new(posX, ABYSS_Y, posZ)
        else
            -- 【囮】地上と奈落を爆速スイッチ。掴んだ瞬間に相手の手をバグらせる。
            if (t * 250) % 2 < 1 then
                part.CFrame = CFrame.new(posX + (i*5), 45, posZ + (i*5)) 
            else
                part.CFrame = CFrame.new(posX, ABYSS_Y + 500, posZ)
            end
        end
    end
end)

-- 万が一の死亡通知
P.CharacterAppearanceLoaded:Connect(function(char)
    local hum = char:WaitForChild("Humanoid")
    hum.Died:Connect(function()
        if Active then Notify("CRITICAL: Death Bypass Failed. Re-activating...") end
    end)
end)
