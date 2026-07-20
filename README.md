--[[
    Premium Community Script Hub v1.0
    Optimizado para Xeno / Soporte UNC Completo
    Enfoque en Alto Rendimiento y Gestión Eficiente de Memoria
--]]

-- Evitar doble ejecución y limpiar hilos previos si existen
if getgenv().LAK_Hub_Loaded then
    print("[LAK HUB] Ya está ejecutándose. Limpiando instancias previas...")
    if getgenv().LAK_Cleanup then getgenv().LAK_Cleanup() end
end
getgenv().LAK_Hub_Loaded = true

-- --- ENTORNO Y VARIABLES GLOBALES (Configuración Compartida) ---
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")

local Settings = {
    Aimbot = {
        Enabled = false,
        Silent = false,
        TargetPart = "Head",
        Smoothness = 1,
        WallCheck = false,
        TeamCheck = false,
        Key = Enum.UserInputType.MouseButton2
    },
    FOV = {
        Visible = false,
        Radius = 100,
        Color = Color3.fromRGB(0, 255, 150),
        Thickness = 1
    },
    ESP = {
        Boxes = false,
        Tracers = false,
        Names = false,
        Distance = false,
        Color = Color3.fromRGB(255, 255, 255)
    },
    Specials = {
        BlockSpinTracker = false
    }
}

-- --- DIBUJO DE ELEMENTOS BÁSICOS (FOV Circle) ---
local FOVCircle = Drawing.new("Circle")
FOVCircle.Filled = false
FOVCircle.Transparency = 1

-- --- CARGA SEGURA DE LA INTERFAZ GRÁFICA ---
local Rayfield
local Success, Error = pcall(function()
    return loadstring(game:HttpGet("https://raw.githubusercontent.com/SiriusRBLX/Rayfield/main/source"))()
end)

if not Success or not Rayfield then
    warn("[LAK HUB FATAL ERROR] No se pudo cargar Rayfield Library: " .. tostring(Error))
    FOVCircle:Destroy()
    getgenv().LAK_Hub_Loaded = false
    return
end

local Window = Rayfield:CreateWindow({
    Name = "LAK Premium Community Hub",
    LoadingTitle = "Iniciando Entorno Luau...",
    LoadingSubtitle = "Optimizado para Xeno Exec",
    ConfigurationSaving = { Enabled = false },
    KeySystem = false
})

-- --- SISTEMA DE ALMACENAMIENTO DE CONEXIONES (Para limpieza de memoria) ---
local Connections = {}
local ESP_Cache = {}

local function SafeConnect(Signal, Callback)
    local Connection = Signal:Connect(Callback)
    table.insert(Connections, Connection)
    return Connection
end

-- --- FUNCIONES AUXILIARES (Aimbot & Validaciones UNC) ---
local function IsVisible(TargetPart, Character)
    if not Settings.Aimbot.WallCheck then return true end
    local RaycastParamsInstance = RaycastParams.new()
    RaycastParamsInstance.FilterType = Enum.RaycastFilterType.Exclude
    RaycastParamsInstance.FilterDescendantsInstances = {LocalPlayer.Character, Character}
    
    local Origin = Camera.CFrame.Position
    local Direction = TargetPart.Position - Origin
    local Result = workspace:Raycast(Origin, Direction, RaycastParamsInstance)
    
    return Result == nil
end

local function GetClosestPlayer()
    local Closest = nil
    local MaxDistance = Settings.FOV.Radius
    local MousePos = UserInputService:GetMouseLocation()

    for _, Player in ipairs(Players:GetPlayers()) do
        if Player ~= LocalPlayer and Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
            if Settings.Aimbot.TeamCheck and Player.Team == LocalPlayer.Team then continue end
            
            local TargetPart = Player.Character:FindFirstChild(Settings.Aimbot.TargetPart)
            local Humanoid = Player.Character:FindFirstChildOfClass("Humanoid")
            
            if TargetPart and Humanoid and Humanoid.Health > 0 then
                local ScreenPos, OnScreen = Camera:WorldToViewportPoint(TargetPart.Position)
                if OnScreen then
                    local Distance = (Vector2.new(ScreenPos.X, ScreenPos.Y) - MousePos).Magnitude
                    if Distance < MaxDistance then
                        if IsVisible(TargetPart, Player.Character) then
                            Closest = TargetPart
                            MaxDistance = Distance
                        end
                    end
                end
            end
        end
    end
    return Closest
end

-- --- PROGRAMACIÓN DE LAS PESTAÑAS (UI) ---

-- 1. PESTAÑA: AIMBOT
local TabAimbot = Window:CreateTab("Aimbot", "crosshair")

TabAimbot:CreateToggle({
    Name = "Activar Aimbot",
    CurrentValue = false,
    Callback = function(Value) Settings.Aimbot.Enabled = Value end
})

TabAimbot:CreateDropdown({
    Name = "Prioridad de Objetivo",
    Options = {"Head", "HumanoidRootPart"},
    CurrentOption = {"Head"},
    MultipleOptions = false,
    Callback = function(Option) Settings.Aimbot.TargetPart = Option[1] end
})

TabAimbot:CreateSlider({
    Name = "Suavizado (Smoothness)",
    Range = {1, 10},
    Increment = 0.5,
    Suffix = "x",
    CurrentValue = 1,
    Callback = function(Value) Settings.Aimbot.Smoothness = Value end
})

TabAimbot:CreateToggle({
    Name = "Wall Check (Verificar Paredes)",
    CurrentValue = false,
    Callback = function(Value) Settings.Aimbot.WallCheck = Value end
})

TabAimbot:CreateToggle({
    Name = "Team Check (Ignorar Aliados)",
    CurrentValue = false,
    Callback = function(Value) Settings.Aimbot.TeamCheck = Value end
})

TabAimbot:CreateKeybind({
    Name = "Tecla de Activación",
    CurrentKeybind = "MouseButton2",
    HoldToInteract = true,
    Callback = function(Keybind) Settings.Aimbot.Key = Keybind end
})

-- Subsección FOV
TabAimbot:CreateSection("Configuración del Círculo FOV")

TabAimbot:CreateToggle({
    Name = "Mostrar Círculo FOV",
    CurrentValue = false,
    Callback = function(Value) Settings.FOV.Visible = Value end
})

TabAimbot:CreateSlider({
    Name = "Radio del FOV",
    Range = {30, 500},
    Increment = 5,
    Suffix = "px",
    CurrentValue = 100,
    Callback = function(Value) Settings.FOV.Radius = Value end
})

TabAimbot:CreateColorPicker({
    Name = "Color del FOV",
    Color = Color3.fromRGB(0, 255, 150),
    Callback = function(Value) Settings.FOV.Color = Value end
})


-- 2. PESTAÑA: VISUALS & ESP
local TabVisuals = Window:CreateTab("Visuals & ESP", "eye")

TabVisuals:CreateToggle({
    Name = "Box ESP (Cajas)",
    CurrentValue = false,
    Callback = function(Value) Settings.ESP.Boxes = Value end
})

TabVisuals:CreateToggle({
    Name = "Tracers (Líneas de Origen)",
    CurrentValue = false,
    Callback = function(Value) Settings.ESP.Tracers = Value end
})

TabVisuals:CreateToggle({
    Name = "Mostrar Nombres",
    CurrentValue = false,
    Callback = function(Value) Settings.ESP.Names = Value end
})

TabVisuals:CreateToggle({
    Name = "Mostrar Distancia",
    CurrentValue = false,
    Callback = function(Value) Settings.ESP.Distance = Value end
})

TabVisuals:CreateColorPicker({
    Name = "Color General del ESP",
    Color = Color3.fromRGB(255, 255, 255),
    Callback = function(Value) Settings.ESP.Color = Value end
})


-- 3. PESTAÑA: MAIN / ESPECIALES
local TabMain = Window:CreateTab("Main / Especiales", "star")

TabMain:CreateSection("Módulos Avanzados")

-- Módulo Optimizado enfocado en la detección de elementos "Block Spin"
TabMain:CreateToggle({
    Name = "Block Spin Tracker (Inventory / Workspace)",
    CurrentValue = false,
    Callback = function(Value) 
        Settings.Specials.BlockSpinTracker = Value 
        if Value then
            Rayfield:Notify({
                Title = "Block Spin Tracker",
                Content = "Escaneo activo en Workspace e Inventario.",
                Duration = 3,
                Image = "info"
            })
        end
    end
})

TabMain:CreateSection("Interfaz")
TabMain:CreateKeybind({
    Name = "Ocultar/Mostrar Hub",
    CurrentKeybind = "RightShift",
    HoldToInteract = false,
    Callback = function()
        -- Lógica de toggle nativa para el contenedor de la interfaz
        if game:GetService("CoreGui"):FindFirstChild("Rayfield") then
            local Gui = game:GetService("CoreGui").Rayfield
            Gui.Enabled = not Gui.Enabled
        end
    end
})


-- --- ESTRUCTURA DE RENDERIZADO ALTO RENDIMIENTO (Loops Principales) ---

-- Renderizado e interacción del Aimbot y FOV
SafeConnect(RunService.RenderStepped, function()
    -- Actualización dinámica del círculo FOV
    if Settings.FOV.Visible then
        FOVCircle.Visible = true
        FOVCircle.Radius = Settings.FOV.Radius
        FOVCircle.Position = UserInputService:GetMouseLocation()
        FOVCircle.Color = Settings.FOV.Color
    else
        FOVCircle.Visible = false
    end

    -- Lógica del Aimbot de Cámara
    if Settings.Aimbot.Enabled and UserInputService:IsMouseButtonPressed(Settings.Aimbot.Key) then
        local Target = GetClosestPlayer()
        if Target then
            local TargetPos = Camera:WorldToViewportPoint(Target.Position)
            local MousePos = UserInputService:GetMouseLocation()
            -- Suavizado interpolado por Luau nativo sin caídas de frames
            Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, Target.Position), (1 / Settings.Aimbot.Smoothness))
        end
    end
end)

-- Sistema de renderizado ESP dinámico optimizado
local function CreateESP(Player)
    if ESP_Cache[Player] then return end

    local Box = Drawing.new("Square")
    Box.Thickness = 1
    Box.Filled = false
    Box.Transparency = 1

    local Tracer = Drawing.new("Line")
    Tracer.Thickness = 1
    Tracer.Transparency = 1

    local Label = Drawing.new("Text")
    Label.Size = 16
    Label.Center = true
    Label.Outline = true
    Label.Transparency = 1

    ESP_Cache[Player] = {Box = Box, Tracer = Tracer, Label = Label}

    local RenderConnection
    RenderConnection = SafeConnect(RunService.RenderStepped, function()
        if not Player or not Player.Parent or not Player.Character or not Player.Character:FindFirstChild("HumanoidRootPart") then
            Box.Visible = false
            Tracer.Visible = false
            Label.Visible = false
            if not Player.Parent then
                Box:Destroy()
                Tracer:Destroy()
                Label:Destroy()
                ESP_Cache[Player] = nil
                RenderConnection:Disconnect()
            end
            return
        end

        local RootPart = Player.Character.HumanoidRootPart
        local Head = Player.Character:FindFirstChild("Head")
        if not Head then return end

        local RootPos, OnScreen = Camera:WorldToViewportPoint(RootPart.Position)
        local HeadPos = Camera:WorldToViewportPoint(Head.Position + Vector3.new(0, 0.5, 0))
        local LegPos = Camera:WorldToViewportPoint(RootPart.Position - Vector3.new(0, 3, 0))

        if OnScreen then
            -- Configuración de Cajas (Box ESP)
            if Settings.ESP.Boxes then
                Box.Size = Vector2.new(1000 / RootPos.Z, HeadPos.Y - LegPos.Y)
                Box.Position = Vector2.new(RootPos.X - Box.Size.X / 2, RootPos.Y - Box.Size.Y / 2)
                Box.Color = Settings.ESP.Color
                Box.Visible = true
            else
                Box.Visible = false
            end

            -- Configuración de Líneas (Tracers)
            if Settings.ESP.Tracers then
                Tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                Tracer.To = Vector2.new(RootPos.X, RootPos.Y)
                Tracer.Color = Settings.ESP.Color
                Tracer.Visible = true
            else
                Tracer.Visible = false
            end

            -- Configuración de Textos (Nombres y Distancia)
            if Settings.ESP.Names or Settings.ESP.Distance then
                local TextStr = ""
                if Settings.ESP.Names then TextStr = TextStr .. Player.Name end
                if Settings.ESP.Distance then
                    local Dist = math.round((LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and (LocalPlayer.Character.HumanoidRootPart.Position - RootPart.Position).Magnitude) or 0)
                    TextStr = TextStr .. " [" .. tostring(Dist) .. "s]"
                end
                
                Label.Text = TextStr
                Label.Position = Vector2.new(RootPos.X, RootPos.Y - (Box.Size.Y / 2) - 18)
                Label.Color = Settings.ESP.Color
                Label.Visible = true
            else
                Label.Visible = false
            end
        else
            Box.Visible = false
            Tracer.Visible = false
            Label.Visible = false
        end
    end)
end

-- Inicializar ESP para jugadores actuales y futuros
for _, Player in ipairs(Players:GetPlayers()) do
    if Player ~= LocalPlayer then CreateESP(Player) end
end
SafeConnect(Players.PlayerAdded, function(Player)
    if Player ~= LocalPlayer then CreateESP(Player) end
end)

-- --- MÓDULO ESPECIAL: TRACKER DE ACCESORIOS E INVENTARIO (Block Spin) ---
task.spawn(function()
    while task.wait(1.5) do
        if not getgenv().LAK_Hub_Loaded then break end
        
        if Settings.Specials.BlockSpinTracker then
            -- 1. Escaneo en el Workspace físico
            for _, Object in ipairs(workspace:GetDescendants()) do
                if Object:IsA("Model") or Object:IsA("Tool") or Object:IsA("Accessory") then
                    if string.find(string.lower(Object.Name), "block") and string.find(string.lower(Object.Name), "spin") then
                        -- Simulación de rastreo avanzado o interacción estructurada con el objeto
                        print("[LAK TRACKER] Detectado elemento Block Spin en Workspace: " .. Object:GetFullName())
                    end
                end
            end

            -- 2. Escaneo en Inventarios (Mochila y Personajes ajenos)
            for _, Player in ipairs(Players:GetPlayers()) do
                if Player.Character then
                    for _, Item in ipairs(Player.Character:GetChildren()) do
                        if string.find(string.lower(Item.Name), "block") and string.find(string.lower(Item.Name), "spin") then
                            print("[LAK TRACKER] Elemento activo equipado en " .. Player.Name .. ": " .. Item.Name)
                        end
                    end
                end
                
                local Backpack = Player:FindFirstChild("Backpack")
                if Backpack then
                    for _, Tool in ipairs(Backpack:GetChildren()) do
                        if string.find(string.lower(Tool.Name), "block") and string.find(string.lower(Tool.Name), "spin") then
                            print("[LAK TRACKER] Elemento almacenado en inventario de " .. Player.Name .. ": " .. Tool.Name)
                        end
                    end
                end
            end
        end
    end
end)

-- --- SISTEMA DE LIMPIEZA ABSOLUTA DE MEMORIA (GC) ---
getgenv().LAK_Cleanup = function()
    print("[LAK HUB] Desconectando eventos y limpiando memoria...")
    for _, Connection in ipairs(Connections) do
        if Connection.Connected then Connection:Disconnect() end
    end
    
    if FOVCircle then FOVCircle:Destroy() end
    
    for _, Cache in pairs(ESP_Cache) do
        if Cache.Box then Cache.Box:Destroy() end
        if Cache.Tracer then Cache.Tracer:Destroy() end
        if Cache.Label then Cache.Label:Destroy() end
    end
    
    table.clear(Connections)
    table.clear(ESP_Cache)
    getgenv().LAK_Hub_Loaded = false
    print("[LAK HUB] Limpieza completada con éxito.")
end

Rayfield:Notify({
    Title = "LAK Premium Hub",
    Content = "Script cargado exitosamente. Disfruta de la optimización.",
    Duration = 5,
    Image = "check"
})

