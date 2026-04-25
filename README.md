-- [[ 最適解：絶対生存ライン（-450）固定版 ]]

local player = game:GetService("Players").LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hum = char:WaitForChild("Humanoid")
local root = char:WaitForChild("HumanoidRootPart")

-- 1. 物理的な「引きずられ」と「固まり」を完全に排除
task.spawn(function()
    -- RootJoint（本体と判定を繋ぐ鎖）を完全に切断
    local function disconnect()
        for _, v in ipairs(char:GetDescendants()) do
            if v:IsA("Motor6D") and (v.Name == "RootJoint" or v.Name == "Root") then
                v.Enabled = false
            end
        end
    end
    
    disconnect()
    
    -- ラグドール化（他の関節もオフ）
    for _, v in ipairs(char:GetDescendants()) do
        if v:IsA("Motor6D") then v.Enabled = false end
    end

    -- 2. メインループ：死なない限界まで判定を下げる
    while task.wait() do
        if char and root and hum then
            -- 判定パーツを即死ライン手前の「-450」に配置
            -- これ以上下げるとRobloxのシステムに消されます
            root.CanCollide = false
            root.Velocity = Vector3.new(0, 0, 0)
            root.CFrame = CFrame.new(char:GetPivot().Position.X, -450, char:GetPivot().Position.Z)

            -- 万が一のダメージを即時回復
            if hum.Health > 0 and hum.Health < 100 then
                hum.Health = 100
            end
            
            -- 掴み防止（拘束パーツの全削除）
            for _, v in ipairs(char:GetDescendants()) do
                if v:IsA("Weld") or v:IsA("ManualWeld") or v:IsA("TouchTransmitter") then
                    v:Destroy()
                end
            end
        end
    end
end)

-- 3. 死亡時の通知
hum.Died:Connect(function()
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "⚠️ 警告",
        Text = "キャラクターがリセットされました。",
        Duration = 3
    })
end)

-- 4. 指定のメインスクリプト実行
loadstring(game:HttpGet("https://raw.githubusercontent.com/monao773-wq/kill-bypass/refs/heads/main/README.md"))()

print("最適化完了：生存限界ライン -450 に固定しました")
