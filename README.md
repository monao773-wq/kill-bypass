-- [[ FINAL ASCENSION V19: THE SINGULARITY (Ultimate Optimization) ]]
local P = game:GetService("Players").LocalPlayer
local R = game:GetService("RunService")
local C = game:GetService("CoreGui")

-- 既存プロセスの完全抹消
if C:FindFirstChild("TheSingularity_V19") then C.TheSingularity_V19:Destroy() end

local G = Instance.new("ScreenGui", C); G.Name = "TheSingularity_V19"
local B = Instance.new("TextButton", G)
B.Size = UDim2.new(0, 280, 0, 70); B.Position = UDim2.new(0, 10, 0.8, 0)
B.Text = "SINGULARITY: STABLE"; B.BackgroundColor3 = Color3.fromRGB(0, 0, 0); B.TextColor3 = Color3.new(0, 1, 1)
B.Font = Enum.Font.Code; B.TextSize = 20; B.BorderSizePixel = 3

local Active = false
local ABYSS_LIMIT = -999999999 -- 物理限界深度
local DOMAIN_EXPANSION = 150000 -- マップの概念を破壊するテレポート範囲

B.MouseButton1Click:Connect(function()
    Active = not Active
    B.Text = Active and "SINGULARITY: OVERRIDING" or "SINGULARITY: STABLE"
    B.BackgroundColor3 = Active and Color3.fromRGB(50, 0, 100) or Color3.fromRGB(0, 0, 0)
    B.TextColor3 = Active and Color3.new(1, 1, 1) or Color3.new(0, 1, 1)
end)

-- [[ 1. 論理不滅コア：マルチスレッド・ラグドール・インジェクション ]]
-- サーバー側のキルパケットが到着する前に、クライアントが「無効」を叩き込む
R.PreSimulation:Connect(function()
    if not Active then return end
    local Char = P.Character
    local Hum = Char and Char:FindFirstChildOfClass("Humanoid")
    if Hum then
        task.spawn(function()
            -- 1フレームにつき120連ループ（有料スクリプトのパケット密度を圧倒）
            for i = 1, 120 do 
                Hum.Health = 100
                Hum:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
                -- 核心のラグドール・ループ
                Hum:ChangeState(Enum.HumanoidStateType.Physics)
                -- 起き上がり（整合性チェック）を完全に破壊
                Hum:SetStateEnabled(Enum.HumanoidStateType.GettingUp, false)
            end
        end)
    end
end)

-- [[ 2. 物理反撃コア：アンチネットワーク・オーバーロード ]]
R.Stepped:Connect(function()
    if not Active then return end
    local Char = P.Character
    if not Char then return end
    
    -- 関節の物理的完全解体
    for _, j in pairs(Char:GetDescendants()) do
        if j:IsA("Motor6D") or j:IsA("Weld") or j:IsA("BallSocketConstraint") then
            j.Enabled = false
        end
    end
    
    local parts = {}
    for _, p in pairs(Char:GetChildren()) do if p:IsA("BasePart") then table.insert(parts, p) end end

    local t = os.clock()
    -- 予測不能なカオス・スパイラル座標（エイム・キラー）
    local angle = t * 150
    local radius = DOMAIN_EXPANSION * (0.5 + 0.5 * math.sin(t * 5))
    local posX = math.cos(angle) * radius
    local posZ = math.sin(angle) * radius
    
    for i, part in ipairs(parts) do
        -- 所有権の奪取と相手への過負荷攻撃
        part.CanTouch = true
        part.CanQuery = true
        
        -- 物理演算の限界（1e38）を全方向に放射
        part.AssemblyLinearVelocity = Vector3.new(1e38, 1e38, 1e38)
        part.RotVelocity = Vector3.new(1e38, 1e38, 1e38)

        if part.Name == "HumanoidRootPart" then
            -- 【核】命の特異点。極限奈落。
            part.CFrame = CFrame.new(posX, ABYSS_LIMIT, posZ)
        else
            -- 【高頻度囮】0.01秒単位で地上と奈落をスイッチ。
            -- 相手がクリックした瞬間に座標をNaNへ飛ばし、相手の武器の計算をバグらせる。
            if (t * 100) % 2 < 1 then
                part.CFrame = CFrame.new(posX, 50, posZ) 
            else
                part.CFrame = CFrame.new(posX, ABYSS_LIMIT + 500, posZ)
            end
        end
    end
end)
