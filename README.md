-- Carregar Fluent UI local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

-- Criar Janela local Window = Fluent:CreateWindow({ Title = "Mat Hub", SubTitle = "Made by @venaszr9", TabWidth = 160, Size = UDim2.fromOffset(580, 360), Acrylic = true, Theme = "Dark", MinimizeKey = Enum.KeyCode.RightControl })

-- Criar Aba Principal local MainTab = Window:AddTab({ Title = "Main", Icon = "Target" })

-- Serviços e Variáveis local Players = game:GetService("Players") local RunService = game:GetService("RunService") local UserInputService = game:GetService("UserInputService") local LocalPlayer = Players.LocalPlayer local Camera = workspace.CurrentCamera

local AimbotEnabled, ESPEnabled, SpeedEnabled, AutoFireEnabled, FlyEnabled, NoclipEnabled = false, false, false, false, false, false local currentColor = 1 local colorOptions = { Color3.fromRGB(0, 255, 0), Color3.fromRGB(255, 0, 0), Color3.fromRGB(0, 0, 255), Color3.fromRGB(255, 255, 0), } local FOVRadius = 100

-- FOV Círculo local FOVCircle = Drawing.new("Circle") FOVCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2) FOVCircle.Radius = FOVRadius FOVCircle.Thickness = 1.5 FOVCircle.Color = colorOptions[currentColor] FOVCircle.Filled = false FOVCircle.Visible = false

-- Jogador mais próximo com checagens local function GetClosestPlayer() local closest, distance = nil, math.huge for _, player in pairs(Players:GetPlayers()) do if player ~= LocalPlayer and player.Team ~= LocalPlayer.Team and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Head") then local screenPos, onScreen = Camera:WorldToViewportPoint(player.Character.Head.Position) if onScreen and (player.Character.Head.Position - Camera.CFrame.Position).Magnitude < 300 then local diff = (Vector2.new(screenPos.X, screenPos.Y) - FOVCircle.Position).Magnitude if diff < FOVRadius and diff < distance then distance = diff closest = player.Character.Head end end end end return closest end

-- Aimbot Loop RunService.RenderStepped:Connect(function() if AimbotEnabled then local target = GetClosestPlayer() if target then local aim = CFrame.new(Camera.CFrame.Position, target.Position) Camera.CFrame = aim end end end)

-- ESP Verde RunService.RenderStepped:Connect(function() if ESPEnabled then for _, player in pairs(Players:GetPlayers()) do if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then if not player.Character.Head:FindFirstChild("ESPBox") then local box = Instance.new("SelectionBox") box.Name = "ESPBox" box.Adornee = player.Character.Head box.LineThickness = 0.05 box.Color3 = Color3.new(0, 1, 0) box.Parent = player.Character.Head end end end else for _, player in pairs(Players:GetPlayers()) do if player.Character and player.Character:FindFirstChild("Head") and player.Character.Head:FindFirstChild("ESPBox") then player.Character.Head.ESPBox:Destroy() end end end end)

-- Opções UI MainTab:AddToggle("QueueTeleportToggle", { Title = "Queue on Teleport", Default = getgenv().queue_teleport or false, Callback = function(state) getgenv().queue_teleport = state writefile("queue_config.txt", tostring(state)) end })

MainTab:AddToggle("AimbotToggle", { Title = "Aimbot (FOV/Wall/Team)", Default = false, Callback = function(v) AimbotEnabled = v end })

MainTab:AddToggle("MostrarFOV", { Title = "Mostrar FOV", Default = false, Callback = function(state) FOVCircle.Visible = state end })

MainTab:AddToggle("ESP", { Title = "ESP Verde", Default = false, Callback = function(v) ESPEnabled = v end })

MainTab:AddToggle("SpeedHack", { Title = "Speed Hack", Default = false, Callback = function(v) SpeedEnabled = v if v then if not _G.SpeedConnection then _G.SpeedConnection = RunService.Stepped:Connect(function() local char = LocalPlayer.Character if char and char:FindFirstChild("Humanoid") then char.Humanoid.WalkSpeed = 60 end end) end else if _G.SpeedConnection then _G.SpeedConnection:Disconnect() end _G.SpeedConnection = nil local char = LocalPlayer.Character if char and char:FindFirstChild("Humanoid") then char.Humanoid.WalkSpeed = 16 end end end })

MainTab:AddToggle("AutoFire", { Title = "Auto Fire", Default = false, Callback = function(v) AutoFireEnabled = v end })

MainTab:AddButton({ Title = "Trocar Cor do FOV", Callback = function() currentColor = (currentColor % #colorOptions) + 1 FOVCircle.Color = colorOptions[currentColor] end })

MainTab:AddButton({ Title = "Teleport para inimigo mais próximo", Callback = function() local target = GetClosestPlayer() if target then LocalPlayer.Character:PivotTo(target.CFrame + Vector3.new(0, 5, 0)) end end })

MainTab:AddToggle("Fly", { Title = "Fly", Default = false, Callback = function(v) FlyEnabled = v local char = LocalPlayer.Character if not char or not char:FindFirstChild("HumanoidRootPart") then return end

if v then
        local bg = Instance.new("BodyGyro", char.HumanoidRootPart)
        bg.Name = "FlyGyro"
        bg.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
        bg.P = 10000
        bg.CFrame = char.HumanoidRootPart.CFrame

        local bv = Instance.new("BodyVelocity", char.HumanoidRootPart)
        bv.Name = "FlyVelocity"
        bv.MaxForce = Vector3.new(9e9, 9e9, 9e9)
        bv.Velocity = Vector3.new(0, 0, 0)

        RunService.RenderStepped:Connect(function()
            if FlyEnabled then
                bv.Velocity = Vector3.new(0, UserInputService:IsKeyDown(Enum.KeyCode.Space) and 20 or 0, 0)
                bg.CFrame = Camera.CFrame
            end
        end)
    else
        local root = char:FindFirstChild("HumanoidRootPart")
        if root:FindFirstChild("FlyGyro") then root.FlyGyro:Destroy() end
        if root:FindFirstChild("FlyVelocity") then root.FlyVelocity:Destroy() end
    end
end

})

MainTab:AddToggle("Noclip", { Title = "Noclip", Default = false, Callback = function(v) NoclipEnabled = v RunService.Stepped:Connect(function() if NoclipEnabled then for _, part in pairs(LocalPlayer.Character:GetDescendants()) do if part:IsA("BasePart") then part.CanCollide = false end end end end) end })

