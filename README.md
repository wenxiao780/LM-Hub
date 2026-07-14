# LM-Hub
--[[
    LM Hub v1.0
    作者: LM Team
    启动方式: loadstring(game:HttpGet("你的GitHub链接"))()
--]]

-- 等待游戏加载
if not game:IsLoaded() then
    game.Loaded:Wait()
end

-- 加载UI库
local Fluent = loadstring(game:HttpGet("https://raw.githubusercontent.com/Xingtaiduan/Script/refs/heads/main/Library/Fluent.lua"))()

-- 创建文件夹
if not isfolder("LM-Hub") then
    makefolder("LM-Hub")
end
if not isfolder("LM-Hub/Configs") then
    makefolder("LM-Hub/Configs")
end

-- 服务变量
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

-- 创建窗口
local Window = Fluent:CreateWindow({
    Title = "LM Hub v1.0",
    SubTitle = "Universal Script",
    TabWidth = 160,
    Size = UDim2.fromOffset(600, 480),
    Acrylic = true,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.RightControl
})

-- 创建标签页
local Tabs = {
    Main = Window:AddTab({ Title = "🏠 主页", Icon = "home" }),
    Player = Window:AddTab({ Title = "👤 玩家", Icon = "user" }),
    Combat = Window:AddTab({ Title = "⚔️ 战斗", Icon = "swords" }),
    Visuals = Window:AddTab({ Title = "👁️ 视觉", Icon = "eye" }),
    Movement = Window:AddTab({ Title = "🏃 移动", Icon = "run" }),
    World = Window:AddTab({ Title = "🌍 世界", Icon = "globe" }),
    Teleports = Window:AddTab({ Title = "📍 传送", Icon = "map-pin" }),
    Settings = Window:AddTab({ Title = "⚙️ 设置", Icon = "settings" })
}

-- 通知函数
local function Notify(title, text, dur)
    Fluent:Notify({
        Title = title or "LM Hub",
        Content = text or "",
        Duration = dur or 3
    })
end

-- 通用函数
local function FindPlayer(name)
    name = name:lower()
    for _, plr in pairs(Players:GetPlayers()) do
        if plr.Name:lower():find(name) or plr.DisplayName:lower():find(name) then
            return plr
        end
    end
end

local function GetNearestPlayer(dist)
    local nearest, shortest = nil, dist or math.huge
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local d = (LocalPlayer.Character.HumanoidRootPart.Position - plr.Character.HumanoidRootPart.Position).Magnitude
            if d < shortest then
                shortest = d
                nearest = plr
            end
        end
    end
    return nearest, shortest
end

-- ============ 主页 ============
Tabs.Main:AddParagraph({
    Title = "🎮 欢迎使用 LM Hub",
    Content = "通用游戏辅助脚本\n版本: v1.0\n作者: LM Team\n\n按 RightControl 隐藏/显示界面"
})

Tabs.Main:AddButton({
    Title = "🔄 重新加载",
    Description = "重新加载脚本",
    Callback = function()
        loadstring(game:HttpGet("你的GitHub链接"))()
    end
})

Tabs.Main:AddButton({
    Title = "💾 保存配置",
    Description = "保存当前设置",
    Callback = function()
        local cfg = {}
        for k, v in pairs(Fluent.Options) do
            if v.Value ~= nil then cfg[k] = v.Value end
        end
        writefile("LM-Hub/Configs/Default.json", game:GetService("HttpService"):JSONEncode(cfg))
        Notify("成功", "配置已保存")
    end
})

Tabs.Main:AddButton({
    Title = "📂 加载配置",
    Description = "加载保存的设置",
    Callback = function()
        local s, d = pcall(function() return readfile("LM-Hub/Configs/Default.json") end)
        if s then
            local cfg = game:GetService("HttpService"):JSONDecode(d)
            for k, v in pairs(cfg) do
                if Fluent.Options[k] then Fluent.Options[k]:SetValue(v) end
            end
            Notify("成功", "配置已加载")
        end
    end
})

-- ============ 玩家 ============
Tabs.Player:AddParagraph({
    Title = "📋 玩家信息",
    Content = "名称: " .. LocalPlayer.DisplayName .. "\n用户名: " .. LocalPlayer.Name .. "\nID: " .. LocalPlayer.UserId
})

local PlayerDropdown = Tabs.Player:AddDropdown({
    Title = "🎯 选择玩家",
    Values = {},
    Multi = false,
    Default = nil
})

local function UpdatePlayerList()
    local names = {}
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then table.insert(names, plr.Name) end
    end
    PlayerDropdown:SetValues(names)
end
UpdatePlayerList()
Players.PlayerAdded:Connect(UpdatePlayerList)
Players.PlayerRemoving:Connect(UpdatePlayerList)

Tabs.Player:AddButton({
    Title = "👁️ 观看玩家",
    Description = "切换到玩家视角",
    Callback = function()
        local t = FindPlayer(PlayerDropdown.Value)
        if t and t.Character then
            Camera.CameraSubject = t.Character
            Notify("成功", "正在观看: " .. t.Name)
        end
    end
})

Tabs.Player:AddButton({
    Title = "📍 传送到玩家",
    Description = "传送到玩家位置",
    Callback = function()
        local t = FindPlayer(PlayerDropdown.Value)
        if t and t.Character and t.Character:FindFirstChild("HumanoidRootPart") then
            LocalPlayer.Character:MoveTo(t.Character.HumanoidRootPart.Position)
            Notify("成功", "已传送")
        end
    end
})

-- ============ 战斗 ============
Tabs.Combat:AddParagraph({ Title = "⚔️ 战斗功能", Content = "自动战斗辅助" })

Tabs.Combat:AddToggle("KillAura", {
    Title = "💀 杀戮光环",
    Description = "自动攻击附近玩家",
    Default = false
})

Tabs.Combat:AddSlider("KillAuraRange", {
    Title = "📏 攻击范围",
    Description = "攻击距离",
    Default = 15, Min = 5, Max = 50, Rounding = 0
})

Tabs.Combat:AddToggle("Aimbot", {
    Title = "🎯 自动瞄准",
    Description = "自动瞄准敌人",
    Default = false
})

-- ============ 视觉 ============
Tabs.Visuals:AddParagraph({ Title = "👁️ 视觉效果", Content = "ESP和视觉增强" })

Tabs.Visuals:AddToggle("PlayerESP", {
    Title = "👤 玩家ESP",
    Description = "高亮显示玩家",
    Default = false
})

Tabs.Visuals:AddColorpicker("ESPColor", {
    Title = "🎨 ESP颜色",
    Default = Color3.fromRGB(255, 0, 0)
})

Tabs.Visuals:AddSlider("FOV", {
    Title = "🔭 视野角度",
    Description = "调整相机FOV",
    Default = 70, Min = 30, Max = 120, Rounding = 0
})

-- ============ 移动 ============
Tabs.Movement:AddParagraph({ Title = "🏃 移动功能", Content = "移动辅助" })

Tabs.Movement:AddSlider("WalkSpeed", {
    Title = "👟 行走速度",
    Description = "修改行走速度",
    Default = 16, Min = 16, Max = 200, Rounding = 0
})

Tabs.Movement:AddSlider("JumpPower", {
    Title = "🦘 跳跃高度",
    Description = "修改跳跃高度",
    Default = 50, Min = 50, Max = 500, Rounding = 0
})

Tabs.Movement:AddToggle("Fly", {
    Title = "🕊️ 飞行模式",
    Description = "自由飞行",
    Default = false
})

local FlySpeed = Tabs.Movement:AddSlider("FlySpeed", {
    Title = "飞行速度",
    Description = "飞行移动速度",
    Default = 50, Min = 10, Max = 200, Rounding = 0
})

Tabs.Movement:AddToggle("NoClip", {
    Title = "🚫 穿墙模式",
    Description = "穿过墙壁",
    Default = false
})

-- ============ 世界 ============
Tabs.World:AddParagraph({ Title = "🌍 世界修改", Content = "修改游戏环境" })

Tabs.World:AddSlider("Time", {
    Title = "🕐 游戏时间",
    Description = "修改时间",
    Default = 12, Min = 0, Max = 24, Rounding = 1
})

Tabs.World:AddSlider("Brightness", {
    Title = "💡 亮度",
    Description = "调整亮度",
    Default = 3, Min = 0, Max = 10, Rounding = 1
})

Tabs.World:AddToggle("NoFog", {
    Title = "🌫️ 移除雾效",
    Description = "禁用雾气",
    Default = false
})

-- ============ 传送 ============
Tabs.Teleports:AddParagraph({ Title = "📍 传送点", Content = "快速传送" })

Tabs.Teleports:AddButton({
    Title = "💾 保存位置",
    Description = "保存当前位置",
    Callback = function()
        local pos = LocalPlayer.Character.HumanoidRootPart.Position
        local data = readfile("LM-Hub/Configs/Teleports.json") or "{}"
        local tps = game:GetService("HttpService"):JSONDecode(data)
        table.insert(tps, {Name = "位置" .. #tps + 1, Pos = {pos.X, pos.Y, pos.Z}})
        writefile("LM-Hub/Configs/Teleports.json", game:GetService("HttpService"):JSONEncode(tps))
        Notify("成功", "位置已保存")
    end
})

Tabs.Teleports:AddButton({
    Title = "📍 传送到保存位置",
    Description = "传送到最近保存的位置",
    Callback = function()
        local s, d = pcall(function() return readfile("LM-Hub/Configs/Teleports.json") end)
        if s then
            local tps = game:GetService("HttpService"):JSONDecode(d)
            if #tps > 0 then
                local p = tps[#tps].Pos
                LocalPlayer.Character:MoveTo(Vector3.new(p[1], p[2], p[3]))
                Notify("成功", "已传送")
            end
        end
    end
})

-- ============ 设置 ============
Tabs.Settings:AddParagraph({ Title = "⚙️ 脚本设置", Content = "配置选项" })

Tabs.Settings:AddToggle("Watermark", {
    Title = "💧 显示水印",
    Description = "显示LM Hub水印",
    Default = true
})

Tabs.Settings:AddButton({
    Title = "🗑️ 清除所有配置",
    Description = "删除配置文件",
    Callback = function()
        local files = listfiles("LM-Hub/Configs")
        for _, f in pairs(files) do delfile(f) end
        Notify("成功", "配置已清除")
    end
})

-- ============ 主循环 ============
RunService.RenderStepped:Connect(function()
    local char = LocalPlayer.Character
    if not char then return end
    
    -- 行走速度
    if Fluent.Options.WalkSpeed then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum.WalkSpeed = Fluent.Options.WalkSpeed.Value end
    end
    
    -- 跳跃高度
    if Fluent.Options.JumpPower then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum.JumpPower = Fluent.Options.JumpPower.Value end
    end
    
    -- 视野
    if Fluent.Options.FOV then
        Camera.FieldOfView = Fluent.Options.FOV.Value
    end
    
    -- 时间
    if Fluent.Options.Time then
        Lighting.ClockTime = Fluent.Options.Time.Value
    end
    
    -- 亮度
    if Fluent.Options.Brightness then
        Lighting.Brightness = Fluent.Options.Brightness.Value
    end
    
    -- 雾效
    if Fluent.Options.NoFog then
        Lighting.FogEnd = Fluent.Options.NoFog.Value and 100000 or 1000
    end
    
    -- 穿墙
    if Fluent.Options.NoClip then
        for _, part in pairs(char:GetDescendants()) do
 
