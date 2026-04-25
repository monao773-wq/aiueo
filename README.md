-- [[ XENO ULTIMATE BYPASS : MULTILOCATIONAL-SPEEDSTER ]] --
local P = game.Players.LocalPlayer
local R = game:GetService("RunService")
local C = game:GetService("CoreGui")

if C:FindFirstChild("UltraBypass") then C.UltraBypass:Destroy() end

local G = Instance.new("ScreenGui", C); G.Name = "UltraBypass"
local B = Instance.new("TextButton", G)
B.Size = UDim2.new(0, 220, 0, 50); B.Position = UDim2.new(0, 10, 0.8, 0)
B.Text = "ULTRA BYPASS: OFF"; B.BackgroundColor3 = Color3.fromRGB(50, 0, 0); B.TextColor3 = Color3.new(1,1,1)

local Active = false
local Houses = {}

-- マップ内の「家」や「建物」をスキャンして座標をリスト化
local function ScanHouses()
    Houses = {}
    for _, v in pairs(workspace:GetDescendants()) do
        if v:IsA("Model") and (v.Name:lower():find("house") or v.Name:lower():find("building") or v.Name:lower():find("home")) then
            local primary = v.PrimaryPart or v:FindFirstChildWhichIsA("BasePart")
            if primary then table.insert(Houses, primary.CFrame) end
        end
    end
    -- 家が見つからない場合のバックアップ座標
    if #Houses == 0 then
        for i = 1, 10 do table.insert(Houses, CFrame.new(math.random(-500, 500), 10, math.random(-500, 500))) end
    end
end

B.MouseButton1Click:Connect(function()
    Active = not Active
    if Active then ScanHouses() end
    B.Text = Active and "ULTRA ACTIVE: PHANTOM" or "ULTRA BYPASS: OFF"
    B.BackgroundColor3 = Active and Color3.fromRGB(0, 150, 255) or Color3.fromRGB(50, 0, 0)
end)

-- 超光速・分散ロジック
R.Heartbeat:Connect(function()
    if not Active then return end
    
    local Char = P.Character
    if not Char then return end
    local Root = Char:FindFirstChild("HumanoidRootPart")
    local Hum = Char:FindFirstChildOfClass("Humanoid")
    
    if Hum and Root then
        Hum.Health = 100
        Hum:SetStateEnabled(Enum.HumanoidStateType.Dead, false)

        local parts = {}
        for _, p in pairs(Char:GetChildren()) do
            if p:IsA("BasePart") then table.insert(parts, p) end
        end

        for i, part in ipairs(parts) do
            part.CanTouch = false
            part.CanQuery = false
            
            if part.Name == "HumanoidRootPart" then
                -- 1. 本体（ルート）だけを -1000 で「ウルトラ超光速移動」
                -- マップ全域を1フレームごとに入れ替わり立ち替わりテレポート
                local speedX = math.sin(os.clock() * 50) * 10000
                local speedZ = math.cos(os.clock() * 50) * 10000
                part.CFrame = CFrame.new(speedX, -1000, speedZ)
            else
                -- 2. 他のパーツを「別々の家」に一個ずつ配置
                -- i 番目のパーツを Houses のリストから順に割り当て
                local housePos = Houses[(i % #Houses) + 1]
                if housePos then
                    part.CFrame = housePos
                end
            end
            
            -- 3. 速度ベクトルを極大化して判定を破壊
            part.AssemblyLinearVelocity = Vector3.new(1e7, 1e7, 1e7)
        end
        
        -- 関節を破壊して個別の動きを許可
        for _, j in pairs(Char:GetDescendants()) do
            if j:IsA("Motor6D") or j:IsA("Weld") then j.Enabled = false end
        end
    end
end)
