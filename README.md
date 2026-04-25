-- [[ kill bypass ]] --
local P = game.Players.LocalPlayer
local R = game:GetService("RunService")
local C = game:GetService("CoreGui")

-- 既存のUIを削除してクリーンアップ
if C:FindFirstChild("kill_bypass_gui") then C.kill_bypass_gui:Destroy() end

-- UI構築
local G = Instance.new("ScreenGui", C); G.Name = "kill_bypass_gui"
local B = Instance.new("TextButton", G)
B.Name = "kill bypass"
B.Size = UDim2.new(0, 180, 0, 50)
B.Position = UDim2.new(0, 10, 0.85, 0)
B.Text = "kill bypass"
B.BackgroundColor3 = Color3.new(0, 0, 0)
B.TextColor3 = Color3.new(1, 1, 1)
B.BorderSizePixel = 2

local Active = false
local HouseCoords = {}

-- マップ内の「家」を特定して座標をリスト化
local function ScanForSafeHouses()
    HouseCoords = {}
    for _, v in pairs(workspace:GetDescendants()) do
        if v:IsA("BasePart") and (v.Name:lower():find("floor") or v.Name:lower():find("wall")) then
            if v.Parent:IsA("Model") and (v.Parent.Name:lower():find("house") or v.Parent.Name:lower():find("home")) then
                table.insert(HouseCoords, v.CFrame)
            end
        end
    end
    -- 家が見つからない場合のバックアップ（ランダムな広域座標）
    if #HouseCoords == 0 then
        for i = 1, 30 do table.insert(HouseCoords, CFrame.new(math.random(-1500, 1500), 10, math.random(-1500, 1500))) end
    end
end

B.MouseButton1Click:Connect(function()
    Active = not Active
    if Active then 
        ScanForSafeHouses() 
        P.CameraMaxZoomDistance = 5000 -- 三人称視点の解放
    end
    B.TextColor3 = Active and Color3.new(1, 0, 0) or Color3.new(1, 1, 1)
    B.BorderColor3 = Active and Color3.new(1, 0, 0) or Color3.new(1, 1, 1)
end)

-- 物理計算直前（Stepped）で実行してサーバーの判定を上書き
R.Stepped:Connect(function()
    if not Active then return end
    
    local Char = P.Character
    local Hum = Char and Char:FindFirstChildOfClass("Humanoid")
    if not (Char and Hum) then return end

    -- ステータスの絶対保護
    Hum.Health = 100
    Hum.RequiresNeck = false
    Hum:SetStateEnabled(Enum.HumanoidStateType.Dead, false)

    local parts = {}
    for _, p in pairs(Char:GetChildren()) do
        if p:IsA("BasePart") then table.insert(parts, p) end
    end

    for i, part in ipairs(parts) do
        -- 判定の無効化
        part.CanTouch = false
        part.CanQuery = false
        
        -- 関節を完全に破壊して個別の移動を可能にする
        for _, j in pairs(part:GetChildren()) do
            if j:IsA("Motor6D") or j:IsA("Weld") or j:IsA("ManualWeld") then
                j:Destroy()
            end
        end

        if part.Name == "HumanoidRootPart" then
            -- 【本体】 -1000の高さでマップ全域（半径10,000）を超光速移動
            local angle = tick() * 100
            local tx = math.sin(angle) * 10000
            local tz = math.cos(angle) * 10000
            
            -- アンチ対策：0.05秒周期で「奈落」と「地上」を往復し検知を回避
            if tick() % 0.1 < 0.05 then
                part.CFrame = CFrame.new(tx, -1000, tz)
            else
                part.CFrame = CFrame.new(0, 15, 0) -- 暫定的な地上位置
            end
        else
            -- 【パーツ】 各パーツをスキャンした別々の家へ分散配置
            local targetCFrame = HouseCoords[(i % #HouseCoords) + 1]
            if targetCFrame then
                part.CFrame = targetCFrame * CFrame.new(math.random(-1, 1), 0, math.random(-1, 1))
            end
        end
        
        -- 物理演算のバグ（NaNベクトル）で敵の弾丸を無効化
        part.Velocity = Vector3.new(0/0, 0/0, 0/0)
    end
end)
