-- =============================================================================
--    BLOCK SPIN - NAME PROTECTOR V3 (MOBILE & TABLET COMPATIBLE)
--    Optimizado para Ejecutores de Celular (Xeno Mobile)
--    Instrucciones: Escribe tu nombre y presiona el botón "APLICAR" en pantalla
-- =============================================================================

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local CoreGui = game:GetService("CoreGui")
local TweenService = game:GetService("TweenService")

-- ⚙️ VARIABLE DINÁMICA
local FakeName = "Dubin_Oculto" 

local RealName = LocalPlayer.Name
local RealDisplayName = LocalPlayer.DisplayName

-- ==================== MOTOR DE PROTECCIÓN DINÁMICO ====================

-- 1. HOOK DE METATABLA
local hookmetamethod = getgenv().hookmetamethod
if type(hookmetamethod) == "function" then
    local oldIndex
    oldIndex = hookmetamethod(game, "__index", function(self, key)
        if not checkcaller() then
            if self == LocalPlayer then
                if key == "Name" or key == "DisplayName" then
                    return FakeName
                end
            end
        end
        return oldIndex(self, key)
    end)
end

-- 2. PROTECCIÓN DIRECTA AL PERSONAJE
local function protegerPersonaje(char)
    if not char then return end
    
    local hum = char:WaitForChild("Humanoid", 5)
    if hum then
        hum.DisplayName = FakeName
    end
    
    local function limpiarTagCustom(inst)
        if inst:IsA("TextLabel") or inst:IsA("TextButton") then
            if inst.Text:find(RealName) or inst.Text:find(RealDisplayName) then
                inst.Text = inst.Text:gsub(RealName, FakeName):gsub(RealDisplayName, FakeName)
            end
        end
    end
    
    char.DescendantAdded:Connect(limpiarTagCustom)
    for _, child in ipairs(char:GetDescendants()) do
        limpiarTagCustom(child)
    end
end

if LocalPlayer.Character then protegerPersonaje(LocalPlayer.Character) end
LocalPlayer.CharacterAdded:Connect(protegerPersonaje)

-- 3. ESCANEO Y FILTRADO DE UI
local function corregirTextoUI(inst)
    if inst:IsA("TextLabel") or inst:IsA("TextButton") or inst:IsA("TextBox") then
        local txt = inst.Text
        if txt:find(RealName) or txt:find(RealDisplayName) then
            inst.Text = txt:gsub(RealName, FakeName):gsub(RealDisplayName, FakeName)
        end
    end
end

LocalPlayer:WaitForChild("PlayerGui").DescendantAdded:Connect(corregirTextoUI)
CoreGui.DescendantAdded:Connect(corregirTextoUI)

local function forzarBarridoCompleto()
    for _, v in ipairs(LocalPlayer.PlayerGui:GetDescendants()) do corregirTextoUI(v) end
    for _, v in ipairs(CoreGui:GetDescendants()) do corregirTextoUI(v) end
end
forzarBarridoCompleto()

-- ==================== INTERFAZ ADAPTADA A MÓVILES ====================

local function getSafeGuiParent()
    local success, target = pcall(function() return CoreGui end)
    if success and target then return target end
    return LocalPlayer:WaitForChild("PlayerGui")
end

local gui = Instance.new("ScreenGui")
gui.Name = "BlockSpin_MobileMenu"
gui.ResetOnSpawn = false
gui.Parent = getSafeGuiParent()

-- Frame principal un poco más alto para albergar el botón táctil
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 240, 0, 115)
MainFrame.Position = UDim2.new(0.5, -120, 0.2, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 22)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true -- Lo puedes arrastrar libremente con el dedo
MainFrame.Parent = gui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8)
corner.Parent = MainFrame

local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(0, 210, 255)
stroke.Thickness = 1.2
stroke.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 25)
Title.BackgroundTransparency = 1
Title.Font = Enum.Font.GothamBold
Title.TextSize = 10
Title.TextColor3 = Color3.fromRGB(180, 190, 210)
Title.Text = "👤 BLOCK SPIN - CAMBIAR NOMBRE"
Title.Parent = MainFrame

-- Cuadro de texto
local InputBox = Instance.new("TextBox")
InputBox.Size = UDim2.new(1, -20, 0, 32)
InputBox.Position = UDim2.new(0, 10, 0, 30)
InputBox.BackgroundColor3 = Color3.fromRGB(24, 24, 36)
InputBox.Font = Enum.Font.GothamBold
InputBox.TextSize = 11
InputBox.TextColor3 = Color3.fromRGB(255, 255, 255)
InputBox.PlaceholderText = "Toca aquí para escribir..."
InputBox.Text = FakeName
InputBox.Parent = MainFrame

local inputCorner = Instance.new("UICorner")
inputCorner.CornerRadius = UDim.new(0, 5)
inputCorner.Parent = InputBox

local inputStroke = Instance.new("UIStroke")
inputStroke.Color = Color3.fromRGB(45, 45, 60)
inputStroke.Thickness = 1
inputStroke.Parent = InputBox

-- Botón táctil "APLICAR"
local ApplyBtn = Instance.new("TextButton")
ApplyBtn.Size = UDim2.new(1, -20, 0, 32)
ApplyBtn.Position = UDim2.new(0, 10, 0, 72)
ApplyBtn.BackgroundColor3 = Color3.fromRGB(0, 210, 255)
ApplyBtn.Font = Enum.Font.GothamBold
ApplyBtn.TextSize = 11
ApplyBtn.TextColor3 = Color3.fromRGB(10, 10, 18)
ApplyBtn.Text = "APLICAR CAMBIO"
ApplyBtn.Parent = MainFrame

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 5)
btnCorner.Parent = ApplyBtn

-- Función central que ejecuta el cambio de identidad
local function ejecutarCambioDeNombre()
    local textoFormateado = InputBox.Text:gsub("%s+", "") -- Evita espacios en blanco
    if textoFormateado ~= "" then
        FakeName = InputBox.Text
        
        -- Cambiar tag en el Humanoid de forma manual e inmediata
        if LocalPlayer.Character then
            local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if hum then hum.DisplayName = FakeName end
        end
        
        -- Barrido total de menús e interfaces
        forzarBarridoCompleto()
        
        -- Animación de éxito (Destello verde en el botón)
        TweenService:Create(ApplyBtn, TweenInfo.new(0.15), {BackgroundColor3 = Color3.fromRGB(0, 255, 140)}):Play()
        TweenService:Create(stroke, TweenInfo.new(0.15), {Color = Color3.fromRGB(0, 255, 140)}):Play()
        
        task.delay(0.4, function()
            TweenService:Create(ApplyBtn, TweenInfo.new(0.25), {BackgroundColor3 = Color3.fromRGB(0, 210, 255)}):Play()
            TweenService:Create(stroke, TweenInfo.new(0.25), {Color = Color3.fromRGB(0, 210, 255)}):Play()
        end)
    else
        InputBox.Text = FakeName
    end
end

-- Ejecutar tanto al tocar el botón como por si acaso pierde el foco
ApplyBtn.MouseButton1Click:Connect(ejecutarCambioDeNombre)
InputBox.FocusLost:Connect(function(enterPressed)
    -- En móvil se ejecuta sin importar si fue enter o si solo cerró el teclado
    ejecutarCambioDeNombre()
end)

print("[Block Spin] Versión móvil cargada correctamente. Usa el botón 'APLICAR CAMBIO'.")
