-- ========================================================
--                XENO MULTI-TOOL HUB (v1.0)
-- ========================================================

local OrionLib = loadstring(game:HttpGet(('https://raw.githubusercontent.com/shlexware/Orion/main/source')))()

-- Notificación de inicio
OrionLib:MakeNotification({
	Name = "Xeno Hub",
	Content = "¡Script cargado con éxito!",
	Image = "rbxassetid://4483345998",
	Time = 5
})

-- Creación de la Ventana Principal
local Window = OrionLib:MakeWindow({
    Name = "Xeno Ultimate Hub", 
    HidePremium = false, 
    SaveConfig = true, 
    ConfigFolder = "XenoHubConfig"
})

-- ========================================================
-- TAB 1: OPCIONES DE JUGADOR
-- ========================================================
local PlayerTab = Window:MakeTab({
	Name = "Jugador",
	Icon = "rbxassetid://4483345998",
	PremiumOnly = false
})

PlayerTab:AddSection({ Name = "Atributos Físicos" })

PlayerTab:AddSlider({
	Name = "Velocidad de Caminado (WalkSpeed)",
	Min = 16,
	Max = 250,
	Default = 16,
	Color = Color3.fromRGB(0, 170, 255),
	Increment = 1,
	ValueName = "Speed",
	Callback = function(Value)
		if game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("Humanoid") then
			game.Players.LocalPlayer.Character.Humanoid.WalkSpeed = Value
		end
	end    
})

PlayerTab:AddSlider({
	Name = "Fuerza de Salto (JumpPower)",
	Min = 50,
	Max = 300,
	Default = 50,
	Color = Color3.fromRGB(255, 170, 0),
	Increment = 1,
	ValueName = "Power",
	Callback = function(Value)
		if game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("Humanoid") then
			game.Players.LocalPlayer.Character.Humanoid.JumpPower = Value
		end
	end    
})

PlayerTab:AddToggle({
	Name = "Salto Infinito",
	Default = false,
	Callback = function(Value)
		_G.InfiniteJump = Value
	end
})

-- Lógica para Salto Infinito
game:GetService("UserInputService").JumpRequest:Connect(function()
    if _G.InfiniteJump and game.Players.LocalPlayer.Character then
        game.Players.LocalPlayer.Character:FindFirstChildOfClass('Humanoid'):ChangeState("Jumping")
    end
end)

-- ========================================================
-- TAB 2: UTILIDADES Y ENTORNO
-- ========================================================
local UtilityTab = Window:MakeTab({
	Name = "Utilidades",
	Icon = "rbxassetid://4483345998",
	PremiumOnly = false
})

UtilityTab:AddSection({ Name = "Ambiente y Iluminación" })

UtilityTab:AddDropdown({
	Name = "Modo de Iluminación",
	Default = "Normal",
	Options = {"Normal", "Noche Profunda", "Visión Clara (Fullbright)"},
	Callback = function(Option)
		if Option == "Visión Clara (Fullbright)" then
			game.Lighting.Brightness = 3
			game.Lighting.ClockTime = 12
			game.Lighting.GlobalShadows = false
		elseif Option == "Noche Profunda" then
			game.Lighting.ClockTime = 0
			game.Lighting.GlobalShadows = true
		else
			game.Lighting.Brightness = 1
			game.Lighting.ClockTime = 14
			game.Lighting.GlobalShadows = true
		end
	end
})

UtilityTab:AddButton({
	Name = "Reaparecer Personaje (Reset)",
	Callback = function()
		if game.Players.LocalPlayer.Character then
			game.Players.LocalPlayer.Character:BreakJoints()
		end
	end
})

-- ========================================================
-- TAB 3: CONFIGURACIÓN DEL MENÚ
-- ========================================================
local SettingsTab = Window:MakeTab({
	Name = "Ajustes",
	Icon = "rbxassetid://4483345998",
	PremiumOnly = false
})

SettingsTab:AddBind({
	Name = "Tecla para Ocultar/Mostrar Menú",
	Default = Enum.KeyCode.RightControl,
	Hold = false,
	Callback = function()
		-- Acción al presionar la tecla asignada
	end
})

-- Inicialización final
OrionLib:Init()

