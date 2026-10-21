local function createBootNotice(text, isError)
	local result = {}

	pcall(function()
		local gui = Instance.new("ScreenGui")
		gui.Name = isError and "XError" or "XBoot"
		gui.ResetOnSpawn = false
		gui.IgnoreGuiInset = true
		gui.DisplayOrder = 1001

		local parented = false
		local candidates = {
			function()
				return gethui()
			end,
			function()
				return game:GetService("CoreGui")
			end,
			function()
				return game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui")
			end,
		}

		for _, getParent in ipairs(candidates) do
			if not parented then
				local success, parent = pcall(getParent)

				if success and parent then
					parented = pcall(function()
						gui.Parent = parent
					end) and gui.Parent ~= nil
				end
			end
		end

		local label = Instance.new("TextButton")
		label.AnchorPoint = Vector2.new(0.5, 0)
		label.Position = UDim2.new(0.5, 0, 0, 12)
		label.Size = UDim2.new(0.9, 0, 0, isError and 130 or 34)
		label.BackgroundColor3 = isError and Color3.fromRGB(30, 12, 16) or Color3.fromRGB(14, 14, 14)
		label.AutoButtonColor = false
		label.Font = Enum.Font.GothamMedium
		label.Text = text
		label.TextColor3 = Color3.new(1, 1, 1)
		label.TextSize = 13
		label.TextWrapped = true
		label.Parent = gui

		local corner = Instance.new("UICorner")
		corner.CornerRadius = UDim.new(0, 10)
		corner.Parent = label

		result.gui = gui
		result.label = label
	end)

	return result
end

local boot = createBootNotice("X - iniciando... / starting...", false)

local function bootStage(text)
	print("[X] " .. text)

	if boot.label then
		pcall(function()
			boot.label.Text = "X - " .. text
		end)
	end
end

local function bootError(message)
	if boot.gui then
		pcall(function()
			boot.gui:Destroy()
		end)
	end

	local notice = createBootNotice("X - Error / Erro:\n" .. string.sub(tostring(message), 1, 400) .. "\n(tap to copy and close / toque para copiar e fechar)", true)

	if notice.label then
		notice.label.Activated:Connect(function()
			local copy = nil

			pcall(function()
				copy = getgenv().setclipboard
			end)

			if type(copy) == "function" then
				pcall(copy, tostring(message))
			end

			notice.gui:Destroy()
		end)
	end
end

local function runX()
	local function getEnvironment()
		local success, environment = pcall(getgenv)

		if success and type(environment) == "table" then
			return environment
		end

		return _G
	end

	local environment = getEnvironment()
	local shared = shared or environment

	local WINDOW_WIDTH, WINDOW_HEIGHT = 480, 300
	local MIN_SCALE, MAX_SCALE = 0.5, 2
	local TOGGLE_KEY = Enum.KeyCode.RightShift
	local TAB_HEIGHT, TAB_GAP, TAB_PADDING = 36, 6, 8
	local TOP_BAR_HEIGHT, FOOTER_HEIGHT = 42, 28
	local ROW_HEIGHT, SUB_GAP, SUB_PADDING = 38, 6, 8
	local PICKER_WIDTH, PICKER_HEIGHT = 240, 244
	local STEPS = { "step_environment", "step_executor", "step_interface", "step_aimbot", "step_esp", "step_theme" }
	local SIGNS = { -1, 1 }
	local STUDS_TO_METERS = 0.28

	local WHITE = Color3.new(1, 1, 1)
	local BLACK = Color3.new(0, 0, 0)
	local WARN_COLOR = Color3.fromRGB(255, 170, 60)
	local FAIL_COLOR = Color3.fromRGB(220, 70, 90)
	local GOOD_COLOR = Color3.fromRGB(90, 220, 120)

	local PRESET_COLORS = {
		Color3.fromRGB(255, 255, 255),
		Color3.fromRGB(255, 70, 70),
		Color3.fromRGB(255, 170, 60),
		Color3.fromRGB(90, 220, 120),
		Color3.fromRGB(80, 170, 255),
		Color3.fromRGB(190, 110, 255),
	}

	local DEFAULTS = {
		theme = "dark",
		language = "en",
		animatedBorder = true,
		includeNpcs = false,

		aimEnabled = false,
		aimMode = "always",
		aimMethod = "camera",
		aimTeamCheck = true,
		aimWallCheck = true,
		aimPart = "head",
		aimMaxDistance = 1000,
		aimSmoothness = 5,
		showFov = true,
		fovRadius = 120,
		fovThickness = 1.5,
		fovFilled = false,
		fovColor = WHITE,

		espEnabled = false,
		espMaxDistance = 1500,
		espTeamCheck = true,
		espTeamColor = true,
		espBoxes = true,
		espBoxStyle = "full",
		espBoxThickness = 1.5,
		espBoxColor = WHITE,
		espNames = true,
		espDisplayName = false,
		espNameSize = 12,
		espNameColor = WHITE,
		espDistance = true,
		espDistanceUnit = "studs",
		espDistanceColor = WHITE,
		espHealth = true,
		espHealthSide = "left",
		espHealthText = false,
		espHealthFull = GOOD_COLOR,
		espHealthEmpty = Color3.fromRGB(255, 70, 70),
		espTracers = false,
		espTracerOrigin = "bottom",
		espTracerThickness = 1,
		espTracerColor = WHITE,
	}

	local THEMES = {
		dark = {
			background = Color3.fromRGB(8, 8, 8),
			panel = Color3.fromRGB(16, 16, 16),
			element = Color3.fromRGB(26, 26, 26),
			elementHover = Color3.fromRGB(42, 42, 42),
			trackOff = Color3.fromRGB(60, 60, 60),
			accent = Color3.fromRGB(255, 255, 255),
			accentText = Color3.fromRGB(0, 0, 0),
			text = Color3.fromRGB(245, 245, 245),
			muted = Color3.fromRGB(150, 150, 150),
		},
		light = {
			background = Color3.fromRGB(250, 250, 250),
			panel = Color3.fromRGB(238, 238, 238),
			element = Color3.fromRGB(226, 226, 226),
			elementHover = Color3.fromRGB(210, 210, 210),
			trackOff = Color3.fromRGB(190, 190, 190),
			accent = Color3.fromRGB(10, 10, 10),
			accentText = Color3.fromRGB(255, 255, 255),
			text = Color3.fromRGB(15, 15, 15),
			muted = Color3.fromRGB(100, 100, 100),
		},
	}

	local LANG = {
		en = {
			tab_aim = "Aim", tab_esp = "ESP", tab_config = "Config",
			sec_aimbot = "Aimbot", sec_fov = "Field of View", sec_visuals = "Visuals", sec_interface = "Interface", sec_targets = "Targets",
			feat_aimbot = "Enable Aimbot", feat_fov = "Show FOV", feat_esp = "Enable ESP", feat_boxes = "Boxes",
			feat_names = "Names", feat_distance = "Distance", feat_health = "Health Bar", feat_tracers = "Tracers",
			set_aim_mode = "Aim Mode", set_aim_method = "Aim Method", set_aim_part = "Target Part", set_max_distance = "Max Distance",
			set_smoothness = "Smoothness", set_team_check = "Team Check", set_wall_check = "Wall Check", set_radius = "Radius",
			set_thickness = "Thickness", set_filled = "Filled", set_team_color = "Use Team Color", set_box_style = "Style",
			set_text_size = "Text Size", set_display_name = "Display Name", set_unit = "Unit", set_side = "Side",
			set_health_text = "Health Number", set_origin = "Origin",
			col_fov = "FOV Color", col_box = "Box Color", col_name = "Name Color", col_distance = "Distance Color",
			col_tracer = "Tracer Color", col_health_full = "Full Health", col_health_empty = "Low Health",
			opt_always = "Always", opt_hold = "Hold RMB", opt_camera = "Camera", opt_mouse = "Mouse",
			opt_head = "Head", opt_root = "Root", opt_torso = "Torso", opt_full = "Full", opt_corner = "Corner",
			opt_filled = "Filled", opt_studs = "Studs", opt_meters = "Meters", opt_left = "Left", opt_right = "Right",
			opt_bottom = "Bottom", opt_center = "Center", opt_top = "Top", opt_dark = "Dark", opt_light = "Light",
			opt_en = "English", opt_pt = "Portugu\u{EA}s",
			cfg_theme = "Theme", cfg_language = "Language", cfg_border = "Animated Border", cfg_npcs = "Include NPCs",
			cfg_hint = "Hold the arrow in the footer and drag to resize. PC key: RightShift. Tap the gear to configure a feature.",
			picker_rainbow = "Rainbow",
			dialog_title = "Close X?",
			dialog_message = "X will be unloaded and every feature will be turned off. Do you want to confirm?",
			dialog_cancel = "Cancel", dialog_confirm = "Confirm",
			step_environment = "Environment", step_executor = "Executor", step_interface = "Interface",
			step_aimbot = "Aimbot", step_esp = "ESP", step_theme = "Theme",
			loader_starting = "Starting...", loader_checking = "Checking %s...", loader_ready = "Ready", loader_error = "Error: %s",
			note_failed = "Failed", note_env = "Camera and player OK", note_exec_ok = "All functions found",
			note_exec_optional = "Optional: %s", note_window = "Window ready", note_camera_mouse = "Camera + Mouse",
			note_camera = "Camera mode", note_overlay = "Overlay ready",
		},
		pt = {
			tab_aim = "Mira", tab_esp = "ESP", tab_config = "Config",
			sec_aimbot = "Aimbot", sec_fov = "Campo de Vis\u{E3}o", sec_visuals = "Visuais", sec_interface = "Interface", sec_targets = "Alvos",
			feat_aimbot = "Ativar Aimbot", feat_fov = "Mostrar FOV", feat_esp = "Ativar ESP", feat_boxes = "Caixas",
			feat_names = "Nomes", feat_distance = "Dist\u{E2}ncia", feat_health = "Barra de Vida", feat_tracers = "Linhas",
			set_aim_mode = "Modo de Mira", set_aim_method = "M\u{E9}todo de Mira", set_aim_part = "Parte do Alvo", set_max_distance = "Dist\u{E2}ncia M\u{E1}xima",
			set_smoothness = "Suaviza\u{E7}\u{E3}o", set_team_check = "Ignorar Time", set_wall_check = "Checar Parede", set_radius = "Raio",
			set_thickness = "Espessura", set_filled = "Preenchido", set_team_color = "Cor do Time", set_box_style = "Estilo",
			set_text_size = "Tamanho do Texto", set_display_name = "Nome de Exibi\u{E7}\u{E3}o", set_unit = "Unidade", set_side = "Lado",
			set_health_text = "N\u{FA}mero da Vida", set_origin = "Origem",
			col_fov = "Cor do FOV", col_box = "Cor da Caixa", col_name = "Cor do Nome", col_distance = "Cor da Dist\u{E2}ncia",
			col_tracer = "Cor da Linha", col_health_full = "Vida Cheia", col_health_empty = "Vida Baixa",
			opt_always = "Sempre", opt_hold = "Bot\u{E3}o direito", opt_camera = "C\u{E2}mera", opt_mouse = "Mouse",
			opt_head = "Cabe\u{E7}a", opt_root = "Centro", opt_torso = "Tronco", opt_full = "Completa", opt_corner = "Cantos",
			opt_filled = "Preenchida", opt_studs = "Studs", opt_meters = "Metros", opt_left = "Esquerda", opt_right = "Direita",
			opt_bottom = "Baixo", opt_center = "Centro", opt_top = "Topo", opt_dark = "Escuro", opt_light = "Claro",
			opt_en = "English", opt_pt = "Portugu\u{EA}s",
			cfg_theme = "Tema", cfg_language = "Idioma", cfg_border = "Borda Animada", cfg_npcs = "Incluir NPCs",
			cfg_hint = "Segure a seta no rodap\u{E9} e arraste para redimensionar. Tecla no PC: RightShift. Toque na engrenagem para configurar a fun\u{E7}\u{E3}o.",
			picker_rainbow = "Arco-\u{ED}ris",
			dialog_title = "Fechar o X?",
			dialog_message = "O X ser\u{E1} descarregado e todas as fun\u{E7}\u{F5}es ser\u{E3}o desativadas. Deseja confirmar?",
			dialog_cancel = "Cancelar", dialog_confirm = "Confirmar",
			step_environment = "Ambiente", step_executor = "Executor", step_interface = "Interface",
			step_aimbot = "Aimbot", step_esp = "ESP", step_theme = "Tema",
			loader_starting = "Iniciando...", loader_checking = "Verificando %s...", loader_ready = "Pronto", loader_error = "Erro: %s",
			note_failed = "Falhou", note_env = "C\u{E2}mera e jogador OK", note_exec_ok = "Todas as fun\u{E7}\u{F5}es encontradas",
			note_exec_optional = "Opcional: %s", note_window = "Janela pronta", note_camera_mouse = "C\u{E2}mera + Mouse",
			note_camera = "Modo c\u{E2}mera", note_overlay = "Overlay pronto",
		},
	}

	local TWEEN_FAST = TweenInfo.new(0.15, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
	local TWEEN_MEDIUM = TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.Out)
	local TWEEN_BACK = TweenInfo.new(0.35, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
	local TWEEN_OPEN = TweenInfo.new(0.45, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
	local TWEEN_CLOSE = TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.In)

	local X = {
		connections = {},
		settings = {},
		visible = false,
		currentTab = nil,
		scale = 1,
	}

	for key, value in pairs(DEFAULTS) do
		X.settings[key] = value
	end

	if shared.X and shared.X.destroy then
		shared.X.destroy()
	end

	local playersService = game:GetService("Players")
	local tweenService = game:GetService("TweenService")
	local userInputService = game:GetService("UserInputService")
	local runService = game:GetService("RunService")
	local guiService = game:GetService("GuiService")
	local localizationService = game:GetService("LocalizationService")
	local statsService = game:GetService("Stats")
	local coreSuccess, coreGui = pcall(function()
		return game:GetService("CoreGui")
	end)

	if not coreSuccess then
		coreGui = nil
	end

	local localPlayer = playersService.LocalPlayer
	local rotatingGradients = {}
	local colorSwatches = {}
	local themeBindings = {}
	local languageBindings = {}
	local espObjects = {}
	local npcModels = {}
	local activeDrag = nil
	local espShown = false
	local screenGui = nil
	local overlayGui = nil
	local fovCircle = nil
	local fovStroke = nil
	local colorPicker = nil

	local raycastParams = RaycastParams.new()
	raycastParams.FilterType = Enum.RaycastFilterType.Exclude

	local function connect(signal, callback)
		local connection = signal:Connect(callback)
		table.insert(X.connections, connection)
		return connection
	end

	local function create(className, properties, children)
		local instance = Instance.new(className)

		for property, value in pairs(properties or {}) do
			instance[property] = value
		end

		for _, child in ipairs(children or {}) do
			child.Parent = instance
		end

		return instance
	end

	local function tween(instance, info, goal)
		local object = tweenService:Create(instance, info, goal)
		object:Play()
		return object
	end

	local function addCorner(parent, radius)
		return create("UICorner", { CornerRadius = UDim.new(0, radius), Parent = parent })
	end

	local function addStroke(parent, color, thickness, transparency)
		return create("UIStroke", {
			Color = color,
			Thickness = thickness,
			Transparency = transparency or 0,
			ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
			Parent = parent,
		})
	end

	local function isPointer(input)
		return input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch
	end

	local function isPointerMove(input)
		return input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch
	end

	local function toHex(color)
		return string.format(
			"#%02X%02X%02X",
			math.floor(color.R * 255 + 0.5),
			math.floor(color.G * 255 + 0.5),
			math.floor(color.B * 255 + 0.5)
		)
	end

	local function parseHex(text)
		local hex = (string.gsub(text, "[#%s]", ""))

		if #hex == 3 then
			hex = string.rep(string.sub(hex, 1, 1), 2) .. string.rep(string.sub(hex, 2, 2), 2) .. string.rep(string.sub(hex, 3, 3), 2)
		end

		if #hex ~= 6 or not string.match(hex, "^%x+$") then
			return nil
		end

		return Color3.fromRGB(
			tonumber(string.sub(hex, 1, 2), 16),
			tonumber(string.sub(hex, 3, 4), 16),
			tonumber(string.sub(hex, 5, 6), 16)
		)
	end

	local function getColor(flag)
		if X.settings[flag .. "Rainbow"] then
			return Color3.fromHSV((os.clock() * 0.25) % 1, 0.85, 1)
		end

		return X.settings[flag]
	end

	local function T(key, ...)
		local language = LANG[X.settings.language] or LANG.en
		local text = language[key] or LANG.en[key] or key

		if select("#", ...) > 0 then
			return string.format(text, ...)
		end

		return text
	end

	local function detectLanguage()
		local success, locale = pcall(function()
			return localizationService.RobloxLocaleId
		end)

		if not success or type(locale) ~= "string" or locale == "" then
			locale = localPlayer.LocaleId or "en-us"
		end

		return string.sub(string.lower(locale), 1, 2) == "pt" and "pt" or "en"
	end

	local function bindLanguage(instance, property, source)
		local resolver = source

		if type(source) == "string" then
			resolver = function()
				return T(source)
			end
		end

		table.insert(languageBindings, { instance = instance, property = property, resolver = resolver })
		instance[property] = resolver()
	end

	local function applyLanguage()
		for _, binding in ipairs(languageBindings) do
			pcall(function()
				binding.instance[binding.property] = binding.resolver()
			end)
		end
	end

	local function currentTheme()
		return THEMES[X.settings.theme] or THEMES.dark
	end

	local function bindTheme(instance, property, source)
		local resolver = source

		if type(source) == "string" then
			resolver = function(theme)
				return theme[source]
			end
		end

		table.insert(themeBindings, { instance = instance, property = property, resolver = resolver })
		instance[property] = resolver(currentTheme())
	end

	local function applyTheme(animated)
		local theme = currentTheme()

		for _, binding in ipairs(themeBindings) do
			pcall(function()
				local value = binding.resolver(theme)

				if animated and typeof(value) == "Color3" then
					tween(binding.instance, TWEEN_MEDIUM, { [binding.property] = value })
				else
					binding.instance[binding.property] = value
				end
			end)
		end
	end

	local function addAccentGradient(parent)
		local gradient = create("UIGradient", { Parent = parent })

		bindTheme(gradient, "Color", function(theme)
			return ColorSequence.new({
				ColorSequenceKeypoint.new(0, theme.accent),
				ColorSequenceKeypoint.new(0.5, theme.muted),
				ColorSequenceKeypoint.new(1, theme.accent),
			})
		end)

		table.insert(rotatingGradients, gradient)
		return gradient
	end

	local function createText(parent, properties, colorKey, textSource)
		properties.Parent = parent
		properties.BackgroundTransparency = 1
		properties.Font = properties.Font or Enum.Font.GothamMedium
		properties.TextSize = properties.TextSize or 14

		local label = create("TextLabel", properties)
		bindTheme(label, "TextColor3", colorKey or "text")

		if textSource then
			bindLanguage(label, "Text", textSource)
		end

		return label
	end

	local function beginDrag(onMove, onEnd)
		activeDrag = { onMove = onMove, onEnd = onEnd }
	end

	local function initializeInput()
		connect(userInputService.InputChanged, function(input)
			if activeDrag and isPointerMove(input) then
				activeDrag.onMove(input.Position)
			end
		end)

		connect(userInputService.InputEnded, function(input)
			if not activeDrag or not isPointer(input) then
				return
			end

			local drag = activeDrag
			activeDrag = nil

			if drag.onEnd then
				drag.onEnd()
			end
		end)
	end

	local function addPressEffect(target, trigger, amount)
		local pressScale = create("UIScale", { Parent = target })

		connect(trigger.InputBegan, function(input)
			if isPointer(input) then
				tween(pressScale, TWEEN_FAST, { Scale = amount or 0.97 })
			end
		end)

		connect(trigger.InputEnded, function(input)
			if isPointer(input) then
				tween(pressScale, TWEEN_BACK, { Scale = 1 })
			end
		end)

		connect(trigger.MouseLeave, function()
			tween(pressScale, TWEEN_FAST, { Scale = 1 })
		end)
	end

	local function addInteraction(target, trigger, hoverResolver, restResolver, noPress)
		hoverResolver = hoverResolver or function(theme)
			return theme.elementHover
		end

		restResolver = restResolver or function(theme)
			return theme.element
		end

		if not noPress then
			addPressEffect(target, trigger)
		end

		connect(trigger.MouseEnter, function()
			tween(target, TWEEN_FAST, { BackgroundColor3 = hoverResolver(currentTheme()) })
		end)

		connect(trigger.MouseLeave, function()
			tween(target, TWEEN_FAST, { BackgroundColor3 = restResolver(currentTheme()) })
		end)
	end

	local function makeDraggable(handle, target, getScaleCallback, state, keepOnScreen)
		connect(handle.InputBegan, function(input)
			if not isPointer(input) then
				return
			end

			local dragStart, startPosition = input.Position, target.Position
			state.moved = false

			beginDrag(function(position)
				local delta = position - dragStart
				local scale = math.max(getScaleCallback(), 0.1)

				if delta.Magnitude > 4 then
					state.moved = true
				end

				target.Position = UDim2.new(
					startPosition.X.Scale,
					startPosition.X.Offset + delta.X / scale,
					startPosition.Y.Scale,
					startPosition.Y.Offset + delta.Y / scale
				)
			end, function()

				if not keepOnScreen then
					return
				end

				local camera = workspace.CurrentCamera
				local center = target.AbsolutePosition + target.AbsoluteSize / 2

				if camera and (center.X < 0 or center.Y < 0 or center.X > camera.ViewportSize.X or center.Y > camera.ViewportSize.Y) then
					tween(target, TWEEN_MEDIUM, { Position = startPosition })
				end
			end)
		end)
	end

	local function getMaxScale()
		local camera = workspace.CurrentCamera
		local viewport = camera and camera.ViewportSize or Vector2.new(1280, 720)
		local fit = math.min(viewport.X / WINDOW_WIDTH, viewport.Y / WINDOW_HEIGHT)

		return math.clamp(fit, MIN_SCALE, MAX_SCALE)
	end

	local function parentGui(gui)
		local candidates = {
			function() return gethui() end,
			function() return coreGui end,
			function() return localPlayer:WaitForChild("PlayerGui") end,
		}

		for _, getParent in ipairs(candidates) do
			local success, parent = pcall(getParent)

			if success and typeof(parent) == "Instance" then
				local assigned = pcall(function()
					gui.Parent = parent
				end)

				if assigned and gui.Parent then
					return parent
				end
			end
		end
	end

	local function getPing()
		local success, ping = pcall(function()
			return statsService.Network.ServerStatsItem["Data Ping"]:GetValue()
		end)

		if success and type(ping) == "number" then
			return ping
		end

		local fallbackSuccess, fallback = pcall(function()
			return localPlayer:GetNetworkPing() * 1000
		end)

		return fallbackSuccess and fallback or 0
	end

	local function registerHumanoid(humanoid)
		local model = humanoid.Parent

		if not model or not model:IsA("Model") then
			return
		end

		if playersService:GetPlayerFromCharacter(model) then
			return
		end

		npcModels[humanoid] = model
	end

	local function scanNpcs()
		for _, descendant in ipairs(workspace:GetDescendants()) do
			if descendant:IsA("Humanoid") then
				registerHumanoid(descendant)
			end
		end
	end

	local function getEntities()
		local entities = {}

		for _, player in ipairs(playersService:GetPlayers()) do
			if player ~= localPlayer and player.Character then
				table.insert(entities, { name = player.Name, displayName = player.DisplayName, model = player.Character, player = player })
			end
		end

		if X.settings.includeNpcs then
			for humanoid, model in pairs(npcModels) do
				if model.Parent and model ~= localPlayer.Character then
					table.insert(entities, { name = model.Name, displayName = model.Name, model = model, humanoid = humanoid })
				else
					npcModels[humanoid] = nil
				end
			end
		end

		return entities
	end

	local function getRootPart(model)
		return model:FindFirstChild("HumanoidRootPart") or model.PrimaryPart or model:FindFirstChildWhichIsA("BasePart")
	end

	local function isAlive(entity)
		local humanoid = entity.humanoid or entity.model:FindFirstChildOfClass("Humanoid")
		return not humanoid or humanoid.Health > 0
	end

	local function isTeammate(player)
		return player.Team ~= nil and player.Team == localPlayer.Team
	end

	local function getAimPart(model)
		local choice = X.settings.aimPart
		local part

		if choice == "head" then
			part = model:FindFirstChild("Head")
		elseif choice == "torso" then
			part = model:FindFirstChild("UpperTorso") or model:FindFirstChild("Torso")
		end

		return part or getRootPart(model)
	end

	local function getAimOrigin()
		local camera = workspace.CurrentCamera

		if userInputService.TouchEnabled and not userInputService.MouseEnabled then
			return camera.ViewportSize / 2
		end

		local success, location = pcall(userInputService.GetMouseLocation, userInputService)

		if success and location then
			return location
		end

		return camera.ViewportSize / 2
	end

	local function getMouseMove()
		local callback = environment.mousemoverel
		return type(callback) == "function" and callback or nil
	end

	local function isVisible(model, part)
		local camera = workspace.CurrentCamera
		local filter = { model }

		if localPlayer.Character then
			table.insert(filter, localPlayer.Character)
		end

		local origin = camera.CFrame.Position
		local direction = part.Position - origin

		for _ = 1, 5 do
			raycastParams.FilterDescendantsInstances = filter

			local result = workspace:Raycast(origin, direction, raycastParams)

			if not result then
				return true
			end

			local hit = result.Instance

			if hit:IsA("Terrain") or (hit:IsA("BasePart") and hit.Transparency < 0.9 and hit.CanCollide) then
				return false
			end

			table.insert(filter, hit)
		end

		return false
	end

	local function evaluateCandidate(entity, currentCamera, mousePos)
		if entity.player and X.settings.aimTeamCheck and isTeammate(entity.player) then
			return nil
		end

		local hrp = getRootPart(entity.model)

		if not (hrp and isAlive(entity)) then
			return nil
		end

		if (currentCamera.CFrame.Position - hrp.Position).Magnitude > X.settings.aimMaxDistance then
			return nil
		end

		local screenPos, onScreen = currentCamera:WorldToViewportPoint(hrp.Position)

		if not onScreen then
			return nil
		end

		local dist = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(mousePos.X, mousePos.Y)).Magnitude

		if dist > X.settings.fovRadius then
			return nil
		end

		if X.settings.aimWallCheck and not isVisible(entity.model, getAimPart(entity.model) or hrp) then
			return nil
		end

		return { entity = entity, dist = dist, screenPos = screenPos }
	end

	local function getClosestPlayer()
		local currentCamera = workspace.CurrentCamera
		local closest, closestDist, candidates = nil, math.huge, {}
		local mousePos = getAimOrigin()

		for _, entity in ipairs(getEntities()) do
			local candidate = evaluateCandidate(entity, currentCamera, mousePos)

			if candidate then
				table.insert(candidates, candidate)
			end
		end

		for _, info in ipairs(candidates) do
			if info.dist < closestDist then
				closest = info.entity
				closestDist = info.dist
			end
		end

		return closest, closestDist, candidates
	end

	local function updateAim(deltaTime)
		if not X.settings.aimEnabled then
			return
		end

		if X.settings.aimMode == "hold" and not userInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
			return
		end

		local camera = workspace.CurrentCamera
		local target = getClosestPlayer()

		if not (camera and target) then
			return
		end

		local part = getAimPart(target.model)

		if not part then
			return
		end

		local smoothness = math.max(X.settings.aimSmoothness, 1)
		local alpha = smoothness <= 1 and 1 or 1 - (1 - 1 / smoothness) ^ (deltaTime * 60)
		local mouseMove = getMouseMove()

		if X.settings.aimMethod == "mouse" and mouseMove then
			local screenPos, onScreen = camera:WorldToViewportPoint(part.Position)

			if not onScreen then
				return
			end

			local delta = (Vector2.new(screenPos.X, screenPos.Y) - getAimOrigin()) * alpha
			mouseMove(delta.X, delta.Y)
			return
		end

		camera.CFrame = camera.CFrame:Lerp(CFrame.lookAt(camera.CFrame.Position, part.Position), alpha)
	end

	local function updateFov()
		if not fovCircle then
			return
		end

		local visible = X.settings.aimEnabled and X.settings.showFov
		fovCircle.Visible = visible

		if not visible then
			return
		end

		local radius = X.settings.fovRadius
		local insetSuccess, inset = pcall(guiService.GetGuiInset, guiService)
		local origin = getAimOrigin() - (insetSuccess and inset or Vector2.new(0, 36))
		local color = getColor("fovColor")

		fovCircle.Size = UDim2.fromOffset(radius * 2, radius * 2)
		fovCircle.Position = UDim2.fromOffset(origin.X, origin.Y)
		fovCircle.BackgroundColor3 = color
		fovCircle.BackgroundTransparency = X.settings.fovFilled and 0.88 or 1
		fovStroke.Color = color
		fovStroke.Thickness = X.settings.fovThickness
	end

	local function createEspObjects()
		local function createLabel(anchor, size, font)
			return create("TextLabel", {
				AnchorPoint = anchor,
				Size = UDim2.fromOffset(160, 14),
				BackgroundTransparency = 1,
				Font = font,
				TextSize = size,
				TextColor3 = WHITE,
				TextStrokeTransparency = 0.3,
				Visible = false,
				Parent = overlayGui,
			})
		end

		local box = create("Frame", { BackgroundTransparency = 1, BorderSizePixel = 0, Visible = false, Parent = overlayGui })
		local corners = {}

		for index = 1, 8 do
			corners[index] = create("Frame", { BackgroundColor3 = WHITE, BorderSizePixel = 0, Visible = false, Parent = overlayGui })
		end

		local healthBack = create("Frame", {
			BackgroundColor3 = BLACK,
			BackgroundTransparency = 0.4,
			BorderSizePixel = 0,
			Visible = false,
			Parent = overlayGui,
		})

		return {
			box = box,
			boxStroke = addStroke(box, WHITE, 1.5, 0),
			corners = corners,
			nameLabel = createLabel(Vector2.new(0.5, 1), 12, Enum.Font.GothamBold),
			distanceLabel = createLabel(Vector2.new(0.5, 0), 11, Enum.Font.GothamMedium),
			healthText = createLabel(Vector2.new(1, 0.5), 10, Enum.Font.GothamBold),
			healthBack = healthBack,
			healthFill = create("Frame", {
				AnchorPoint = Vector2.new(0, 1),
				Position = UDim2.fromScale(0, 1),
				BorderSizePixel = 0,
				Parent = healthBack,
			}),
			tracer = create("Frame", {
				AnchorPoint = Vector2.new(0.5, 0.5),
				BackgroundColor3 = WHITE,
				BorderSizePixel = 0,
				Visible = false,
				Parent = overlayGui,
			}),
		}
	end

	local function hideCorners(objects)
		for _, corner in ipairs(objects.corners) do
			corner.Visible = false
		end
	end

	local function hideEsp(objects)
		objects.box.Visible = false
		objects.nameLabel.Visible = false
		objects.distanceLabel.Visible = false
		objects.healthBack.Visible = false
		objects.healthText.Visible = false
		objects.tracer.Visible = false
		hideCorners(objects)
	end

	local function destroyEsp(objects)
		for _, value in pairs(objects) do
			if typeof(value) == "Instance" then
				value:Destroy()
			elseif type(value) == "table" then
				for _, instance in ipairs(value) do
					instance:Destroy()
				end
			end
		end
	end

	local function drawCorners(objects, minX, minY, maxX, maxY, color, thickness)
		local length = math.max(math.min(maxX - minX, maxY - minY) * 0.28, 4)
		local corners = objects.corners

		local layout = {
			{ minX, minY, length, thickness },
			{ minX, minY, thickness, length },
			{ maxX - length, minY, length, thickness },
			{ maxX - thickness, minY, thickness, length },
			{ minX, maxY - thickness, length, thickness },
			{ minX, maxY - length, thickness, length },
			{ maxX - length, maxY - thickness, length, thickness },
			{ maxX - thickness, maxY - length, thickness, length },
		}

		for index, data in ipairs(layout) do
			local corner = corners[index]
			corner.Visible = true
			corner.BackgroundColor3 = color
			corner.Position = UDim2.fromOffset(data[1], data[2])
			corner.Size = UDim2.fromOffset(data[3], data[4])
		end
	end

	local function projectModel(camera, model)
		local boundsSuccess, boxCFrame, boxSize = pcall(model.GetBoundingBox, model)

		if not boundsSuccess then
			return false, 0, 0, 0, 0
		end
		local half = boxSize / 2
		local minX, minY, maxX, maxY = math.huge, math.huge, -math.huge, -math.huge
		local visible = false

		for _, signX in ipairs(SIGNS) do
			for _, signY in ipairs(SIGNS) do
				for _, signZ in ipairs(SIGNS) do
					local corner = boxCFrame:PointToWorldSpace(Vector3.new(half.X * signX, half.Y * signY, half.Z * signZ))
					local point = camera:WorldToScreenPoint(corner)

					if point.Z > 0 then
						visible = true
						minX = math.min(minX, point.X)
						minY = math.min(minY, point.Y)
						maxX = math.max(maxX, point.X)
						maxY = math.max(maxY, point.Y)
					end
				end
			end
		end

		return visible, minX, minY, maxX, maxY
	end

	local function updateEntityEsp(entity, camera, origin, overlaySize)
		local settings = X.settings
		local model = entity.model
		local objects = espObjects[model]

		if not objects then
			objects = createEspObjects()
			espObjects[model] = objects
		end

		local root = getRootPart(model)
		local humanoid = entity.humanoid or model:FindFirstChildOfClass("Humanoid")

		if not root or (humanoid and humanoid.Health <= 0) or (entity.player and settings.espTeamCheck and isTeammate(entity.player)) then
			hideEsp(objects)
			return
		end

		local distance = (origin - root.Position).Magnitude

		if distance > settings.espMaxDistance then
			hideEsp(objects)
			return
		end

		local visible, minX, minY, maxX, maxY = projectModel(camera, model)
		local width, height = maxX - minX, maxY - minY

		if not visible or height < 4 then
			hideEsp(objects)
			return
		end

		local centerX = (minX + maxX) / 2
		local useTeamColor = settings.espTeamColor and entity.player and entity.player.Team
		local boxColor = useTeamColor and entity.player.TeamColor.Color or getColor("espBoxColor")
		local tracerColor = useTeamColor and entity.player.TeamColor.Color or getColor("espTracerColor")

		if settings.espBoxes and settings.espBoxStyle == "corner" then
			objects.box.Visible = false
			drawCorners(objects, minX, minY, maxX, maxY, boxColor, settings.espBoxThickness)
		elseif settings.espBoxes then
			hideCorners(objects)
			objects.box.Visible = true
			objects.box.Position = UDim2.fromOffset(minX, minY)
			objects.box.Size = UDim2.fromOffset(width, height)
			objects.box.BackgroundColor3 = boxColor
			objects.box.BackgroundTransparency = settings.espBoxStyle == "filled" and 0.85 or 1
			objects.boxStroke.Color = boxColor
			objects.boxStroke.Thickness = settings.espBoxThickness
		else
			objects.box.Visible = false
			hideCorners(objects)
		end

		objects.nameLabel.Visible = settings.espNames
		objects.nameLabel.Position = UDim2.fromOffset(centerX, minY - 2)
		objects.nameLabel.Text = settings.espDisplayName and entity.displayName or entity.name
		objects.nameLabel.TextSize = settings.espNameSize
		objects.nameLabel.TextColor3 = getColor("espNameColor")

		objects.distanceLabel.Visible = settings.espDistance
		objects.distanceLabel.Position = UDim2.fromOffset(centerX, maxY + 2)
		objects.distanceLabel.Text = settings.espDistanceUnit == "meters"
			and string.format("%d m", math.floor(distance * STUDS_TO_METERS))
			or string.format("%d studs", math.floor(distance))
		objects.distanceLabel.TextColor3 = getColor("espDistanceColor")

		local showHealth = settings.espHealth and humanoid ~= nil
		objects.healthBack.Visible = showHealth
		objects.healthText.Visible = showHealth and settings.espHealthText

		if showHealth then
			local ratio = math.clamp(humanoid.Health / math.max(humanoid.MaxHealth, 1), 0, 1)
			local onLeft = settings.espHealthSide == "left"
			local barX = onLeft and minX - 6 or maxX + 3

			objects.healthBack.Position = UDim2.fromOffset(barX, minY)
			objects.healthBack.Size = UDim2.fromOffset(3, height)
			objects.healthFill.Size = UDim2.fromScale(1, ratio)
			objects.healthFill.BackgroundColor3 = getColor("espHealthEmpty"):Lerp(getColor("espHealthFull"), ratio)

			objects.healthText.AnchorPoint = Vector2.new(onLeft and 1 or 0, 0.5)
			objects.healthText.Position = UDim2.fromOffset(onLeft and barX - 3 or barX + 6, minY + height * (1 - ratio))
			objects.healthText.Text = tostring(math.floor(humanoid.Health))
		end

		local originY = settings.espTracerOrigin == "top" and 0 or settings.espTracerOrigin == "center" and overlaySize.Y / 2 or overlaySize.Y
		local startPoint = Vector2.new(overlaySize.X / 2, originY)
		local endPoint = Vector2.new(centerX, maxY)
		local delta = endPoint - startPoint

		objects.tracer.Visible = settings.espTracers
		objects.tracer.Size = UDim2.fromOffset(delta.Magnitude, settings.espTracerThickness)
		objects.tracer.Position = UDim2.fromOffset((startPoint.X + endPoint.X) / 2, (startPoint.Y + endPoint.Y) / 2)
		objects.tracer.Rotation = math.deg(math.atan2(delta.Y, delta.X))
		objects.tracer.BackgroundColor3 = tracerColor
	end

	local function updateEsp()
		local camera = workspace.CurrentCamera

		if not camera then
			return
		end

		if not X.settings.espEnabled then
			if espShown then
				for _, objects in pairs(espObjects) do
					hideEsp(objects)
				end

				espShown = false
			end

			return
		end

		espShown = true

		local origin = camera.CFrame.Position
		local overlaySize = overlayGui.AbsoluteSize
		local seen = {}

		for _, entity in ipairs(getEntities()) do
			seen[entity.model] = true
			updateEntityEsp(entity, camera, origin, overlaySize)
		end

		for model, objects in pairs(espObjects) do
			if not seen[model] then
				destroyEsp(objects)
				espObjects[model] = nil
			end
		end
	end

	local function safeStep(name, callback)
		local warned = false

		return function(...)
			local success, result = pcall(callback, ...)

			if not success and not warned then
				warned = true
				warn("[X] " .. name .. " error: " .. tostring(result))
			end
		end
	end

	function X.destroy()
		pcall(runService.UnbindFromRenderStep, runService, "XAim")
		pcall(runService.UnbindFromRenderStep, runService, "XVisuals")

		for _, connection in ipairs(X.connections) do
			connection:Disconnect()
		end

		table.clear(X.connections)
		table.clear(espObjects)
		table.clear(npcModels)
		activeDrag = nil

		if screenGui then
			screenGui:Destroy()
		end

		if overlayGui then
			overlayGui:Destroy()
		end

		if shared.X == X then
			shared.X = nil
		end
	end

	local function createLogo(parent, size, backKey, frontKey, withRing)
		local logo = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.5),
			Size = UDim2.fromOffset(size, size),
			BackgroundTransparency = 1,
			Parent = parent,
		})

		local result = { logo = logo }

		if withRing then
			local ring = create("Frame", {
				AnchorPoint = Vector2.new(0.5, 0.5),
				Position = UDim2.fromScale(0.5, 0.5),
				Size = UDim2.fromOffset(size * 1.3, size * 1.3),
				BackgroundTransparency = 1,
				Parent = logo,
			})
			create("UICorner", { CornerRadius = UDim.new(1, 0), Parent = ring })

			result.ring = ring
			result.ringStroke = addStroke(ring, WHITE, 2, 0.4)
			bindTheme(result.ringStroke, "Color", "muted")
		end

		local function createBar(rotation, key, length, zIndex)
			local bar = create("Frame", {
				AnchorPoint = Vector2.new(0.5, 0.5),
				Position = UDim2.fromScale(0.5, 0.5),
				Size = UDim2.fromOffset(length, size * 0.2),
				Rotation = rotation,
				BorderSizePixel = 0,
				ZIndex = zIndex,
				Parent = logo,
			})
			create("UICorner", { CornerRadius = UDim.new(1, 0), Parent = bar })
			bindTheme(bar, "BackgroundColor3", key)
			return bar
		end

		result.back = createBar(-45, backKey, size * 0.86, 1)
		result.front = createBar(45, frontKey, size * 1.02, 2)

		return result
	end

	local function createContext(parent, scroller, bgKey)
		return {
			parent = parent,
			scroller = scroller,
			bgKey = bgKey,
			hover = function(theme)
				return bgKey == "panel" and theme.element or theme.elementHover
			end,
			rest = function(theme)
				return theme[bgKey]
			end,
		}
	end

	local function createRow(ctx, height, clip)
		local row = create("Frame", {
			Size = UDim2.new(1, 0, 0, height),
			BorderSizePixel = 0,
			ClipsDescendants = clip or false,
			Parent = ctx.parent,
		})
		bindTheme(row, "BackgroundColor3", ctx.bgKey)
		addCorner(row, 8)
		return row
	end

	local function createRowLabel(row, labelKey, widthOffset)
		return createText(row, {
			Position = UDim2.fromOffset(12, 0),
			Size = UDim2.new(1, widthOffset or -70, 0, ROW_HEIGHT),
			TextXAlignment = Enum.TextXAlignment.Left,
		}, "text", labelKey)
	end

	local function addSection(ctx, labelKey)
		createText(ctx.parent, {
			Size = UDim2.new(1, 0, 0, 18),
			Font = Enum.Font.GothamBold,
			TextSize = 11,
			TextXAlignment = Enum.TextXAlignment.Left,
		}, "muted", function()
			return string.upper(T(labelKey))
		end)
	end

	local function addInfo(ctx, labelKey)
		createText(ctx.parent, {
			Size = UDim2.new(1, 0, 0, 48),
			Font = Enum.Font.Gotham,
			TextSize = 12,
			TextWrapped = true,
			TextXAlignment = Enum.TextXAlignment.Left,
			TextYAlignment = Enum.TextYAlignment.Top,
		}, "muted", labelKey)
	end

	local function createSwitch(parent, flag, offsetX)
		local switch = {}

		local track = create("Frame", {
			AnchorPoint = Vector2.new(1, 0.5),
			Position = UDim2.new(1, offsetX, 0, ROW_HEIGHT / 2),
			Size = UDim2.fromOffset(40, 20),
			BorderSizePixel = 0,
			Parent = parent,
		})
		bindTheme(track, "BackgroundColor3", function(theme)
			return X.settings[flag] and theme.accent or theme.trackOff
		end)
		addCorner(track, 10)

		local knob = create("Frame", {
			AnchorPoint = Vector2.new(0, 0.5),
			Position = X.settings[flag] and UDim2.new(1, -18, 0.5, 0) or UDim2.new(0, 2, 0.5, 0),
			Size = UDim2.fromOffset(16, 16),
			BorderSizePixel = 0,
			Parent = track,
		})
		bindTheme(knob, "BackgroundColor3", function(theme)
			return X.settings[flag] and theme.accentText or theme.muted
		end)
		addCorner(knob, 8)

		function switch.press()
			tween(knob, TWEEN_FAST, { Size = UDim2.fromOffset(20, 16) })
		end

		function switch.release()
			tween(knob, TWEEN_FAST, { Size = UDim2.fromOffset(16, 16) })
		end

		function switch.flip()
			local state = not X.settings[flag]
			local theme = currentTheme()
			X.settings[flag] = state

			tween(track, TWEEN_FAST, { BackgroundColor3 = state and theme.accent or theme.trackOff })
			tween(knob, TWEEN_BACK, {
				Size = UDim2.fromOffset(16, 16),
				Position = state and UDim2.new(1, -18, 0.5, 0) or UDim2.new(0, 2, 0.5, 0),
				BackgroundColor3 = state and theme.accentText or theme.muted,
			})

			return state
		end

		return switch
	end

	local function addToggle(ctx, labelKey, flag, callback)
		local row = createRow(ctx, ROW_HEIGHT)
		createRowLabel(row, labelKey)

		local switch = createSwitch(row, flag, -12)
		local button = create("TextButton", { BackgroundTransparency = 1, Size = UDim2.fromScale(1, 1), Text = "", Parent = row })
		addInteraction(row, button, ctx.hover, ctx.rest)

		connect(button.InputBegan, function(input)
			if isPointer(input) then
				switch.press()
			end
		end)

		connect(button.MouseLeave, switch.release)

		connect(button.Activated, function()
			local state = switch.flip()

			if callback then
				callback(state)
			end
		end)
	end

	local function addSlider(ctx, labelKey, flag, minValue, maxValue, step, callback)
		local row = createRow(ctx, 52)
		createRowLabel(row, labelKey, -80)

		local valueLabel = createText(row, {
			AnchorPoint = Vector2.new(1, 0),
			Position = UDim2.new(1, -12, 0, 0),
			Size = UDim2.fromOffset(60, 34),
			Font = Enum.Font.GothamBold,
			Text = tostring(X.settings[flag]),
			TextSize = 13,
			TextXAlignment = Enum.TextXAlignment.Right,
		}, "text")

		local track = create("Frame", {
			Position = UDim2.new(0, 12, 0, 38),
			Size = UDim2.new(1, -24, 0, 6),
			BorderSizePixel = 0,
			Parent = row,
		})
		bindTheme(track, "BackgroundColor3", "trackOff")
		addCorner(track, 3)

		local fill = create("Frame", {
			Size = UDim2.fromScale((X.settings[flag] - minValue) / (maxValue - minValue), 1),
			BorderSizePixel = 0,
			Parent = track,
		})
		bindTheme(fill, "BackgroundColor3", "accent")
		addCorner(fill, 3)

		local knob = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(1, 0.5),
			Size = UDim2.fromOffset(14, 14),
			BorderSizePixel = 0,
			Parent = fill,
		})
		bindTheme(knob, "BackgroundColor3", "accent")
		addCorner(knob, 7)

		local hitbox = create("TextButton", {
			Position = UDim2.new(0, 6, 0, 28),
			Size = UDim2.new(1, -12, 0, 26),
			BackgroundTransparency = 1,
			Text = "",
			Parent = row,
		})

		local dragging = false

		addInteraction(row, row, ctx.hover, ctx.rest, true)

		local function setFromPosition(positionX)
			local alpha = math.clamp((positionX - track.AbsolutePosition.X) / track.AbsoluteSize.X, 0, 1)
			local value = minValue + (maxValue - minValue) * alpha
			value = math.floor(value / step + 0.5) * step
			value = math.clamp(math.floor(value * 1000 + 0.5) / 1000, minValue, maxValue)

			X.settings[flag] = value
			valueLabel.Text = tostring(value)

			tween(fill, TweenInfo.new(0.08), { Size = UDim2.fromScale((value - minValue) / (maxValue - minValue), 1) })
		end

		connect(hitbox.MouseEnter, function()
			if not dragging then
				tween(knob, TWEEN_FAST, { Size = UDim2.fromOffset(17, 17) })
			end
		end)

		connect(hitbox.MouseLeave, function()
			if not dragging then
				tween(knob, TWEEN_FAST, { Size = UDim2.fromOffset(14, 14) })
			end
		end)

		connect(hitbox.InputBegan, function(input)
			if not isPointer(input) then
				return
			end

			dragging = true
			ctx.scroller.ScrollingEnabled = false
			tween(knob, TWEEN_FAST, { Size = UDim2.fromOffset(20, 20) })
			setFromPosition(input.Position.X)

			beginDrag(function(position)
				setFromPosition(position.X)
			end, function()
				dragging = false
				ctx.scroller.ScrollingEnabled = true
				tween(knob, TWEEN_BACK, { Size = UDim2.fromOffset(14, 14) })

				if callback then
					callback(X.settings[flag])
				end
			end)
		end)
	end

	local function addCycle(ctx, labelKey, flag, options, callback)
		local index = table.find(options, X.settings[flag]) or 1
		X.settings[flag] = options[index]

		local row = createRow(ctx, ROW_HEIGHT)
		createRowLabel(row, labelKey, -150)

		local valueLabel = createText(row, {
			AnchorPoint = Vector2.new(1, 0),
			Position = UDim2.new(1, -12, 0, 0),
			Size = UDim2.fromOffset(140, ROW_HEIGHT),
			Font = Enum.Font.GothamBold,
			TextSize = 13,
			TextXAlignment = Enum.TextXAlignment.Right,
		}, "muted", function()
			return T("opt_" .. X.settings[flag])
		end)

		local button = create("TextButton", { BackgroundTransparency = 1, Size = UDim2.fromScale(1, 1), Text = "", Parent = row })
		addInteraction(row, button, ctx.hover, ctx.rest)

		connect(button.MouseEnter, function()
			tween(valueLabel, TWEEN_FAST, { TextColor3 = currentTheme().text })
		end)

		connect(button.MouseLeave, function()
			tween(valueLabel, TWEEN_FAST, { TextColor3 = currentTheme().muted })
		end)

		connect(button.Activated, function()
			index = index % #options + 1
			X.settings[flag] = options[index]
			valueLabel.Text = T("opt_" .. options[index])

			if callback then
				callback(options[index])
			end
		end)
	end

	local function addChoice(ctx, labelKey, flag, options, segmentWidth, callback)
		local row = createRow(ctx, ROW_HEIGHT)
		createRowLabel(row, labelKey, -(#options * segmentWidth + 40))

		local holder = create("Frame", {
			AnchorPoint = Vector2.new(1, 0.5),
			Position = UDim2.new(1, -8, 0.5, 0),
			Size = UDim2.fromOffset(#options * segmentWidth + (#options - 1) * 4, 26),
			BackgroundTransparency = 1,
			Parent = row,
		})
		create("UIListLayout", { FillDirection = Enum.FillDirection.Horizontal, Padding = UDim.new(0, 4), Parent = holder })

		local segments = {}

		local function refreshSegments()
			local theme = currentTheme()

			for _, entry in ipairs(segments) do
				local active = X.settings[flag] == entry.option

				tween(entry.segment, TWEEN_FAST, {
					BackgroundColor3 = active and theme.accent or theme.trackOff,
					TextColor3 = active and theme.accentText or theme.muted,
				})
			end
		end

		for _, option in ipairs(options) do
			local segment = create("TextButton", {
				Size = UDim2.fromOffset(segmentWidth, 26),
				AutoButtonColor = false,
				Font = Enum.Font.GothamBold,
				TextSize = 12,
				Parent = holder,
			})
			addCorner(segment, 6)
			table.insert(segments, { segment = segment, option = option })
			bindLanguage(segment, "Text", function()
				return T("opt_" .. option)
			end)

			bindTheme(segment, "BackgroundColor3", function(theme)
				return X.settings[flag] == option and theme.accent or theme.trackOff
			end)

			bindTheme(segment, "TextColor3", function(theme)
				return X.settings[flag] == option and theme.accentText or theme.muted
			end)

			addPressEffect(segment, segment, 0.93)

			connect(segment.MouseEnter, function()
				if X.settings[flag] ~= option then
					local theme = currentTheme()
					tween(segment, TWEEN_FAST, { BackgroundColor3 = theme.trackOff:Lerp(theme.accent, 0.25) })
				end
			end)

			connect(segment.MouseLeave, function()
				if X.settings[flag] ~= option then
					tween(segment, TWEEN_FAST, { BackgroundColor3 = currentTheme().trackOff })
				end
			end)

			connect(segment.Activated, function()
				X.settings[flag] = option
				refreshSegments()

				if callback then
					callback(option)
				end
			end)
		end
	end

	local function refreshSwatches(flag)
		for _, swatch in ipairs(colorSwatches[flag] or {}) do
			swatch.BackgroundColor3 = getColor(flag)
		end
	end

	local function addColorPicker(ctx, labelKey, flag)
		X.settings[flag .. "Rainbow"] = X.settings[flag .. "Rainbow"] or false

		local row = createRow(ctx, ROW_HEIGHT)
		createRowLabel(row, labelKey, -70)

		local swatch = create("Frame", {
			AnchorPoint = Vector2.new(1, 0.5),
			Position = UDim2.new(1, -12, 0.5, 0),
			Size = UDim2.fromOffset(34, 20),
			BackgroundColor3 = X.settings[flag],
			BorderSizePixel = 0,
			Parent = row,
		})
		addCorner(swatch, 6)
		bindTheme(addStroke(swatch, WHITE, 1, 0.2), "Color", "muted")

		colorSwatches[flag] = colorSwatches[flag] or {}
		table.insert(colorSwatches[flag], swatch)

		local swatchScale = create("UIScale", { Parent = swatch })
		local button = create("TextButton", { BackgroundTransparency = 1, Size = UDim2.fromScale(1, 1), Text = "", Parent = row })
		addInteraction(row, button, ctx.hover, ctx.rest)

		connect(button.MouseEnter, function()
			tween(swatchScale, TWEEN_BACK, { Scale = 1.12 })
		end)

		connect(button.MouseLeave, function()
			tween(swatchScale, TWEEN_BACK, { Scale = 1 })
		end)

		connect(button.Activated, function()
			colorPicker.open(flag, labelKey)
		end)
	end

	local function createGear(parent)
		local holder = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.5),
			Size = UDim2.fromOffset(20, 20),
			BackgroundTransparency = 1,
			Parent = parent,
		})

		local parts = {}

		for index = 0, 7 do
			local angle = math.rad(index * 45)
			local tooth = create("Frame", {
				AnchorPoint = Vector2.new(0.5, 0.5),
				Position = UDim2.fromOffset(10 + math.cos(angle) * 7.5, 10 + math.sin(angle) * 7.5),
				Size = UDim2.fromOffset(4, 4),
				Rotation = index * 45,
				BorderSizePixel = 0,
				Parent = holder,
			})
			addCorner(tooth, 1)
			bindTheme(tooth, "BackgroundColor3", "muted")
			table.insert(parts, tooth)
		end

		local ring = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.5),
			Size = UDim2.fromOffset(9, 9),
			BackgroundTransparency = 1,
			Parent = holder,
		})
		create("UICorner", { CornerRadius = UDim.new(1, 0), Parent = ring })

		local ringStroke = addStroke(ring, WHITE, 2.5, 0)
		bindTheme(ringStroke, "Color", "muted")

		local function tint(key)
			local color = currentTheme()[key]

			for _, part in ipairs(parts) do
				tween(part, TWEEN_FAST, { BackgroundColor3 = color })
			end

			tween(ringStroke, TWEEN_FAST, { Color = color })
		end

		return holder, tint
	end

	local function addFeature(ctx, labelKey, flag, buildSettings)
		local expanded = false
		local contentHeight = 0

		local container = create("Frame", {
			Size = UDim2.new(1, 0, 0, ROW_HEIGHT),
			BorderSizePixel = 0,
			ClipsDescendants = true,
			Parent = ctx.parent,
		})
		bindTheme(container, "BackgroundColor3", ctx.bgKey)
		addCorner(container, 8)

		createRowLabel(container, labelKey, -120)

		local header = create("TextButton", { BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, ROW_HEIGHT), Text = "", Parent = container })
		addInteraction(container, header, ctx.hover, ctx.rest, true)

		local switch = createSwitch(container, flag, -12)

		connect(header.InputBegan, function(input)
			if isPointer(input) then
				switch.press()
			end
		end)

		connect(header.MouseLeave, switch.release)

		connect(header.Activated, function()
			switch.flip()
		end)

		local gearButton = create("TextButton", {
			AnchorPoint = Vector2.new(1, 0.5),
			Position = UDim2.new(1, -62, 0, ROW_HEIGHT / 2),
			Size = UDim2.fromOffset(30, 30),
			BackgroundTransparency = 1,
			Text = "",
			Parent = container,
		})

		local gear, tintGear = createGear(gearButton)
		addPressEffect(gear, gearButton, 0.85)

		connect(gearButton.MouseEnter, function()
			tintGear("text")
		end)

		connect(gearButton.MouseLeave, function()
			tintGear("muted")
		end)

		local panel = create("Frame", {
			Position = UDim2.fromOffset(0, ROW_HEIGHT),
			Size = UDim2.new(1, 0, 0, 0),
			BackgroundTransparency = 1,
			Visible = false,
			Parent = container,
		})
		create("UIListLayout", { Padding = UDim.new(0, SUB_GAP), SortOrder = Enum.SortOrder.LayoutOrder, Parent = panel })
		create("UIPadding", {
			PaddingTop = UDim.new(0, SUB_PADDING),
			PaddingBottom = UDim.new(0, SUB_PADDING),
			PaddingLeft = UDim.new(0, SUB_PADDING),
			PaddingRight = UDim.new(0, SUB_PADDING),
			Parent = panel,
		})

		buildSettings(createContext(panel, ctx.scroller, "panel"))

		local total, count = 0, 0

		for _, child in ipairs(panel:GetChildren()) do
			if child:IsA("GuiObject") then
				total = total + child.Size.Y.Offset
				count = count + 1
			end
		end

		contentHeight = total + math.max(count - 1, 0) * SUB_GAP + SUB_PADDING * 2
		panel.Size = UDim2.new(1, 0, 0, contentHeight)

		connect(gearButton.Activated, function()
			expanded = not expanded

			if expanded then
				panel.Visible = true
			end

			tween(container, TWEEN_MEDIUM, {
				Size = UDim2.new(1, 0, 0, ROW_HEIGHT + (expanded and contentHeight or 0)),
			}).Completed:Connect(function()
				if not expanded then
					panel.Visible = false
				end
			end)

			tween(gear, TWEEN_BACK, { Rotation = expanded and 90 or 0 })
		end)
	end

	local function createColorPicker(gui, main)
		local picker = {}
		local currentFlag, hue, saturation, value = nil, 0, 0, 1

		local viewport = workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize or Vector2.new(1280, 720)
		local baseScale = math.clamp(math.min(1, viewport.X / (PICKER_WIDTH + 20), viewport.Y / (PICKER_HEIGHT + 20)), 0.6, 1)

		local dimmer = create("TextButton", {
			Name = "Dimmer",
			Size = UDim2.fromScale(1, 1),
			BackgroundColor3 = BLACK,
			BackgroundTransparency = 1,
			AutoButtonColor = false,
			Text = "",
			Visible = false,
			ZIndex = 15,
			Parent = main,
		})
		addCorner(dimmer, 12)

		local window = create("Frame", {
			Name = "ColorPicker",
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.5),
			Size = UDim2.fromOffset(PICKER_WIDTH, PICKER_HEIGHT),
			BorderSizePixel = 0,
			Visible = false,
			ZIndex = 30,
			Parent = gui,
		})
		bindTheme(window, "BackgroundColor3", "panel")
		addCorner(window, 12)
		bindTheme(addStroke(window, WHITE, 1.5, 0.2), "Color", "trackOff")

		local windowScale = create("UIScale", { Scale = baseScale * 0.85, Parent = window })

		local titleBar = create("Frame", { Size = UDim2.new(1, 0, 0, 32), BackgroundTransparency = 1, Parent = window })

		local title = createText(titleBar, {
			Position = UDim2.fromOffset(12, 0),
			Size = UDim2.new(1, -48, 1, 0),
			Font = Enum.Font.GothamBold,
			TextSize = 13,
			TextXAlignment = Enum.TextXAlignment.Left,
		}, "text")

		local closeButton = create("TextButton", {
			AnchorPoint = Vector2.new(1, 0.5),
			Position = UDim2.new(1, -8, 0.5, 0),
			Size = UDim2.fromOffset(22, 22),
			AutoButtonColor = false,
			Font = Enum.Font.GothamBold,
			Text = "\u{D7}",
			TextSize = 14,
			Parent = titleBar,
		})
		bindTheme(closeButton, "BackgroundColor3", "element")
		bindTheme(closeButton, "TextColor3", "text")
		addCorner(closeButton, 6)
		addInteraction(closeButton, closeButton, function()
			return FAIL_COLOR
		end)

		local svBox = create("Frame", {
			Position = UDim2.fromOffset(12, 40),
			Size = UDim2.fromOffset(PICKER_WIDTH - 24, 110),
			BackgroundColor3 = Color3.fromHSV(0, 1, 1),
			BorderSizePixel = 0,
			Parent = window,
		})
		addCorner(svBox, 6)

		local whiteOverlay = create("Frame", { Size = UDim2.fromScale(1, 1), BackgroundColor3 = WHITE, BorderSizePixel = 0, ZIndex = 2, Parent = svBox })
		addCorner(whiteOverlay, 6)
		create("UIGradient", {
			Transparency = NumberSequence.new({ NumberSequenceKeypoint.new(0, 0), NumberSequenceKeypoint.new(1, 1) }),
			Parent = whiteOverlay,
		})

		local blackOverlay = create("Frame", { Size = UDim2.fromScale(1, 1), BackgroundColor3 = BLACK, BorderSizePixel = 0, ZIndex = 3, Parent = svBox })
		addCorner(blackOverlay, 6)
		create("UIGradient", {
			Rotation = 90,
			Transparency = NumberSequence.new({ NumberSequenceKeypoint.new(0, 1), NumberSequenceKeypoint.new(1, 0) }),
			Parent = blackOverlay,
		})

		local svCursor = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Size = UDim2.fromOffset(14, 14),
			BackgroundTransparency = 1,
			ZIndex = 4,
			Parent = svBox,
		})
		create("UICorner", { CornerRadius = UDim.new(1, 0), Parent = svCursor })
		addStroke(svCursor, WHITE, 2, 0)

		local svHitbox = create("TextButton", { Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, Text = "", ZIndex = 5, Parent = svBox })

		local hueBar = create("Frame", {
			Position = UDim2.fromOffset(12, 158),
			Size = UDim2.fromOffset(PICKER_WIDTH - 24, 14),
			BackgroundColor3 = WHITE,
			BorderSizePixel = 0,
			Parent = window,
		})
		addCorner(hueBar, 7)

		local hueKeypoints = {}

		for step = 0, 6 do
			table.insert(hueKeypoints, ColorSequenceKeypoint.new(step / 6, Color3.fromHSV(math.min(step / 6, 0.999), 1, 1)))
		end

		create("UIGradient", { Color = ColorSequence.new(hueKeypoints), Parent = hueBar })

		local hueCursor = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0, 0.5),
			Size = UDim2.new(0, 6, 1, 4),
			BackgroundTransparency = 1,
			ZIndex = 4,
			Parent = hueBar,
		})
		addCorner(hueCursor, 3)
		addStroke(hueCursor, WHITE, 2, 0)

		local hueHitbox = create("TextButton", { Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, Text = "", ZIndex = 5, Parent = hueBar })

		local preview = create("Frame", {
			Position = UDim2.fromOffset(12, 180),
			Size = UDim2.fromOffset(26, 26),
			BorderSizePixel = 0,
			Parent = window,
		})
		addCorner(preview, 6)
		bindTheme(addStroke(preview, WHITE, 1, 0.2), "Color", "muted")

		local hexBox = create("TextBox", {
			Position = UDim2.fromOffset(46, 180),
			Size = UDim2.fromOffset(82, 26),
			Font = Enum.Font.GothamMedium,
			Text = "#FFFFFF",
			PlaceholderText = "#FFFFFF",
			TextSize = 12,
			ClearTextOnFocus = false,
			Parent = window,
		})
		bindTheme(hexBox, "BackgroundColor3", "element")
		bindTheme(hexBox, "TextColor3", "text")
		bindTheme(hexBox, "PlaceholderColor3", "muted")
		addCorner(hexBox, 6)

		local rainbowButton = create("TextButton", {
			AnchorPoint = Vector2.new(1, 0),
			Position = UDim2.new(1, -12, 0, 180),
			Size = UDim2.fromOffset(92, 26),
			AutoButtonColor = false,
			Font = Enum.Font.GothamBold,
			TextSize = 11,
			Parent = window,
		})
		addCorner(rainbowButton, 6)
		addPressEffect(rainbowButton, rainbowButton, 0.93)
		bindLanguage(rainbowButton, "Text", "picker_rainbow")
		bindTheme(rainbowButton, "BackgroundColor3", function(theme)
			return currentFlag and X.settings[currentFlag .. "Rainbow"] and theme.accent or theme.trackOff
		end)
		bindTheme(rainbowButton, "TextColor3", function(theme)
			return currentFlag and X.settings[currentFlag .. "Rainbow"] and theme.accentText or theme.muted
		end)

		local function updateRainbowButton()
			local theme = currentTheme()
			local active = currentFlag and X.settings[currentFlag .. "Rainbow"]

			tween(rainbowButton, TWEEN_FAST, {
				BackgroundColor3 = active and theme.accent or theme.trackOff,
				TextColor3 = active and theme.accentText or theme.muted,
			})
		end

		local function refresh(write)
			local color = Color3.fromHSV(hue, saturation, value)

			svBox.BackgroundColor3 = Color3.fromHSV(hue, 1, 1)
			svCursor.Position = UDim2.fromScale(saturation, 1 - value)
			hueCursor.Position = UDim2.fromScale(hue, 0.5)

			if write and currentFlag then
				X.settings[currentFlag] = color
				refreshSwatches(currentFlag)
			end

			preview.BackgroundColor3 = currentFlag and getColor(currentFlag) or color

			if not hexBox:IsFocused() then
				hexBox.Text = toHex(color)
			end
		end

		local function setColor(color)
			hue, saturation, value = color:ToHSV()
			refresh(true)
		end

		local function setFromSv(position)
			saturation = math.clamp((position.X - svBox.AbsolutePosition.X) / svBox.AbsoluteSize.X, 0, 1)
			value = 1 - math.clamp((position.Y - svBox.AbsolutePosition.Y) / svBox.AbsoluteSize.Y, 0, 1)
			refresh(true)
		end

		local function setFromHue(position)
			hue = math.clamp((position.X - hueBar.AbsolutePosition.X) / hueBar.AbsoluteSize.X, 0, 0.999)
			refresh(true)
		end

		connect(svHitbox.InputBegan, function(input)
			if isPointer(input) then
				setFromSv(input.Position)
				beginDrag(setFromSv)
			end
		end)

		connect(hueHitbox.InputBegan, function(input)
			if isPointer(input) then
				setFromHue(input.Position)
				beginDrag(setFromHue)
			end
		end)

		connect(hexBox.FocusLost, function()
			local color = parseHex(hexBox.Text)

			if color then
				setColor(color)
			else
				refresh(false)
			end
		end)

		connect(rainbowButton.Activated, function()
			if not currentFlag then
				return
			end

			X.settings[currentFlag .. "Rainbow"] = not X.settings[currentFlag .. "Rainbow"]
			updateRainbowButton()
			refreshSwatches(currentFlag)
			refresh(false)
		end)

		local presetWidth = 26
		local presetGap = (PICKER_WIDTH - 24 - #PRESET_COLORS * presetWidth) / (#PRESET_COLORS - 1)

		for index, preset in ipairs(PRESET_COLORS) do
			local presetButton = create("TextButton", {
				Position = UDim2.fromOffset(12 + (index - 1) * (presetWidth + presetGap), 214),
				Size = UDim2.fromOffset(presetWidth, 20),
				BackgroundColor3 = preset,
				AutoButtonColor = false,
				Text = "",
				Parent = window,
			})
			addCorner(presetButton, 6)
			bindTheme(addStroke(presetButton, WHITE, 1, 0.4), "Color", "trackOff")
			addPressEffect(presetButton, presetButton, 0.8)

			connect(presetButton.Activated, function()
				setColor(preset)
			end)
		end

		function picker.open(flag, titleKey)
			currentFlag = flag
			hue, saturation, value = X.settings[flag]:ToHSV()
			title.Text = T(titleKey)
			updateRainbowButton()
			refresh(false)

			dimmer.Visible = true
			window.Visible = true
			windowScale.Scale = baseScale * 0.85

			tween(dimmer, TWEEN_MEDIUM, { BackgroundTransparency = 0.45 })
			tween(windowScale, TWEEN_OPEN, { Scale = baseScale })
		end

		function picker.close()
			if not currentFlag then
				return
			end

			currentFlag = nil
			updateRainbowButton()

			tween(dimmer, TWEEN_CLOSE, { BackgroundTransparency = 1 })
			tween(windowScale, TWEEN_CLOSE, { Scale = baseScale * 0.85 }).Completed:Connect(function()
				if not currentFlag then
					window.Visible = false
					dimmer.Visible = false
				end
			end)
		end

		function picker.tick()
			if currentFlag and X.settings[currentFlag .. "Rainbow"] then
				preview.BackgroundColor3 = getColor(currentFlag)
			end
		end

		connect(closeButton.Activated, picker.close)
		connect(dimmer.Activated, picker.close)

		makeDraggable(titleBar, window, function()
			return windowScale.Scale
		end, { moved = false }, true)

		return picker
	end

	local function createWindow(gui)
		local window = {}

		local main = create("Frame", {
			Name = "Main",
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.5),
			Size = UDim2.fromOffset(WINDOW_WIDTH, WINDOW_HEIGHT),
			BorderSizePixel = 0,
			Visible = false,
			Parent = gui,
		})
		bindTheme(main, "BackgroundColor3", "background")
		addCorner(main, 12)

		window.main = main
		window.mainScale = create("UIScale", { Scale = 0, Parent = main })
		addAccentGradient(addStroke(main, WHITE, 2, 0))

		local function createBar(name, size, position, anchor, fillerPosition, linePosition)
			local bar = create("Frame", {
				Name = name,
				AnchorPoint = anchor,
				Position = position,
				Size = size,
				BorderSizePixel = 0,
				Parent = main,
			})
			bindTheme(bar, "BackgroundColor3", "panel")
			addCorner(bar, 12)

			local filler = create("Frame", { Position = fillerPosition, Size = UDim2.new(1, 0, 0, 12), BorderSizePixel = 0, Parent = bar })
			bindTheme(filler, "BackgroundColor3", "panel")

			local line = create("Frame", {
				Position = linePosition,
				Size = UDim2.new(1, 0, 0, 1),
				BorderSizePixel = 0,
				BackgroundTransparency = 0.5,
				Parent = bar,
			})
			bindTheme(line, "BackgroundColor3", "trackOff")

			return bar
		end

		local topBar = createBar(
			"TopBar",
			UDim2.new(1, 0, 0, TOP_BAR_HEIGHT),
			UDim2.fromOffset(0, 0),
			Vector2.new(0, 0),
			UDim2.new(0, 0, 1, -12),
			UDim2.new(0, 0, 1, -1)
		)

		local footer = createBar(
			"Footer",
			UDim2.new(1, 0, 0, FOOTER_HEIGHT),
			UDim2.fromScale(0, 1),
			Vector2.new(0, 1),
			UDim2.fromOffset(0, 0),
			UDim2.fromOffset(0, 0)
		)

		local topLogo = createLogo(topBar, 22, "muted", "accent", false)
		topLogo.logo.Position = UDim2.fromOffset(26, TOP_BAR_HEIGHT / 2)

		window.statusLabel = createText(footer, {
			Position = UDim2.fromOffset(14, 0),
			Size = UDim2.new(1, -60, 1, 0),
			Font = Enum.Font.GothamMedium,
			Text = "0 FPS",
			TextSize = 11,
			TextXAlignment = Enum.TextXAlignment.Left,
			RichText = true,
		}, "muted")

		local function createTopButton(text, offsetX, hoverColor)
			local button = create("TextButton", {
				AnchorPoint = Vector2.new(1, 0.5),
				Position = UDim2.new(1, offsetX, 0.5, 0),
				Size = UDim2.fromOffset(28, 28),
				AutoButtonColor = false,
				Font = Enum.Font.GothamBold,
				Text = text,
				TextSize = 16,
				Parent = topBar,
			})
			bindTheme(button, "BackgroundColor3", "element")
			bindTheme(button, "TextColor3", "text")
			addCorner(button, 8)

			addInteraction(button, button, hoverColor and function()
				return hoverColor
			end or nil)

			return button
		end

		window.closeButton = createTopButton("\u{D7}", -10, FAIL_COLOR)
		window.minimizeButton = createTopButton("\u{2013}", -44, nil)
		window.topBar = topBar
		window.footer = footer

		window.sidebar = create("Frame", {
			Name = "Sidebar",
			Position = UDim2.fromOffset(0, TOP_BAR_HEIGHT),
			Size = UDim2.new(0, 120, 1, -(TOP_BAR_HEIGHT + FOOTER_HEIGHT)),
			BorderSizePixel = 0,
			Parent = main,
		})
		bindTheme(window.sidebar, "BackgroundColor3", "panel")

		window.content = create("CanvasGroup", {
			Name = "Content",
			Position = UDim2.fromOffset(120, TOP_BAR_HEIGHT),
			Size = UDim2.new(1, -120, 1, -(TOP_BAR_HEIGHT + FOOTER_HEIGHT)),
			BackgroundTransparency = 1,
			Parent = main,
		})

		return window
	end

	local function createTabSystem(window)
		local system = {}
		local tabs = {}
		local switchToken = 0

		local function getIndicatorY(index)
			return TAB_PADDING + (index - 1) * (TAB_HEIGHT + TAB_GAP) + TAB_HEIGHT / 2
		end

		local indicator = create("Frame", {
			AnchorPoint = Vector2.new(0, 0.5),
			Size = UDim2.fromOffset(3, 20),
			Position = UDim2.fromOffset(0, getIndicatorY(1)),
			BorderSizePixel = 0,
			Parent = window.sidebar,
		})
		bindTheme(indicator, "BackgroundColor3", "accent")
		addCorner(indicator, 2)

		function system.select(tabName)
			if X.currentTab == tabName then
				return
			end

			X.currentTab = tabName
			switchToken = switchToken + 1

			local token = switchToken
			local theme = currentTheme()

			for name, data in pairs(tabs) do
				local active = name == tabName

				tween(data.button, TWEEN_FAST, {
					BackgroundTransparency = active and 0 or 1,
					TextColor3 = active and theme.text or theme.muted,
				})

				if active then
					tween(indicator, TWEEN_MEDIUM, { Position = UDim2.fromOffset(0, getIndicatorY(data.index)) })

					tween(indicator, TweenInfo.new(0.12), { Size = UDim2.fromOffset(3, 30) }).Completed:Connect(function()
						tween(indicator, TWEEN_BACK, { Size = UDim2.fromOffset(3, 20) })
					end)
				end
			end

			tween(window.content, TWEEN_FAST, { GroupTransparency = 1 })
			task.wait(0.15)

			if token ~= switchToken then
				return
			end

			for name, data in pairs(tabs) do
				data.page.Visible = name == tabName
			end

			local page = tabs[tabName].page
			page.Position = UDim2.fromOffset(0, 16)
			page.CanvasPosition = Vector2.new(0, 0)

			tween(page, TWEEN_MEDIUM, { Position = UDim2.fromOffset(0, 0) })
			tween(window.content, TWEEN_MEDIUM, { GroupTransparency = 0 })
		end

		function system.create(name, labelKey, index)
			local button = create("TextButton", {
				Position = UDim2.new(0, TAB_PADDING, 0, TAB_PADDING + (index - 1) * (TAB_HEIGHT + TAB_GAP)),
				Size = UDim2.new(1, -TAB_PADDING * 2, 0, TAB_HEIGHT),
				BackgroundTransparency = 1,
				AutoButtonColor = false,
				Font = Enum.Font.GothamMedium,
				TextSize = 14,
				Parent = window.sidebar,
			})
			bindLanguage(button, "Text", labelKey)
			bindTheme(button, "BackgroundColor3", "element")
			bindTheme(button, "TextColor3", function(theme)
				return X.currentTab == name and theme.text or theme.muted
			end)
			addCorner(button, 8)
			addPressEffect(button, button, 0.95)

			local page = create("ScrollingFrame", {
				Name = name,
				Size = UDim2.new(1, -6, 1, 0),
				BackgroundTransparency = 1,
				BorderSizePixel = 0,
				CanvasSize = UDim2.new(),
				AutomaticCanvasSize = Enum.AutomaticSize.Y,
				ScrollBarThickness = 3,
				ScrollBarImageTransparency = 0.3,
				Visible = false,
				Parent = window.content,
			})
			bindTheme(page, "ScrollBarImageColor3", "accent")

			create("UIListLayout", { Padding = UDim.new(0, 8), SortOrder = Enum.SortOrder.LayoutOrder, Parent = page })
			create("UIPadding", {
				PaddingTop = UDim.new(0, 12),
				PaddingBottom = UDim.new(0, 12),
				PaddingLeft = UDim.new(0, 12),
				PaddingRight = UDim.new(0, 10),
				Parent = page,
			})

			tabs[name] = { button = button, page = page, index = index }

			connect(button.Activated, function()
				system.select(name)
			end)

			connect(button.MouseEnter, function()
				if X.currentTab ~= name then
					tween(button, TWEEN_FAST, { BackgroundTransparency = 0.6 })
				end
			end)

			connect(button.MouseLeave, function()
				if X.currentTab ~= name then
					tween(button, TWEEN_FAST, { BackgroundTransparency = 1 })
				end
			end)

			return createContext(page, page, "element")
		end

		function system.setInitial(name)
			X.currentTab = name
			tabs[name].page.Visible = true
			tabs[name].button.BackgroundTransparency = 0
			indicator.Position = UDim2.fromOffset(0, getIndicatorY(tabs[name].index))
		end

		return system
	end

	local function createConfirmDialog(parent)
		local dialog = {}

		local overlay = create("TextButton", {
			Name = "Confirm",
			Size = UDim2.fromScale(1, 1),
			BackgroundColor3 = BLACK,
			BackgroundTransparency = 1,
			AutoButtonColor = false,
			Text = "",
			Visible = false,
			ZIndex = 40,
			Parent = parent,
		})
		addCorner(overlay, 12)

		local card = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.5),
			Size = UDim2.fromOffset(290, 150),
			BorderSizePixel = 0,
			Parent = overlay,
		})
		bindTheme(card, "BackgroundColor3", "panel")
		addCorner(card, 12)
		bindTheme(addStroke(card, WHITE, 1, 0.3), "Color", "trackOff")

		local cardScale = create("UIScale", { Scale = 0.85, Parent = card })

		createText(card, {
			Position = UDim2.fromOffset(16, 14),
			Size = UDim2.new(1, -32, 0, 24),
			Font = Enum.Font.GothamBold,
			TextSize = 17,
			TextXAlignment = Enum.TextXAlignment.Left,
		}, "text", "dialog_title")

		createText(card, {
			Position = UDim2.fromOffset(16, 42),
			Size = UDim2.new(1, -32, 0, 48),
			Font = Enum.Font.Gotham,
			TextSize = 12,
			TextWrapped = true,
			TextXAlignment = Enum.TextXAlignment.Left,
			TextYAlignment = Enum.TextYAlignment.Top,
		}, "muted", "dialog_message")

		local function createDialogButton(labelKey, positionX, filled)
			local button = create("TextButton", {
				Position = UDim2.fromOffset(positionX, 100),
				Size = UDim2.fromOffset(121, 34),
				AutoButtonColor = false,
				Font = Enum.Font.GothamBold,
				TextSize = 13,
				Parent = card,
			})
			addCorner(button, 8)
			bindLanguage(button, "Text", labelKey)

			if filled then
				bindTheme(button, "BackgroundColor3", "accent")
				bindTheme(button, "TextColor3", "accentText")
				addInteraction(button, button, function(theme)
					return theme.muted
				end, function(theme)
					return theme.accent
				end)
			else
				bindTheme(button, "BackgroundColor3", "element")
				bindTheme(button, "TextColor3", "text")
				addInteraction(button, button)
			end

			return button
		end

		dialog.cancelButton = createDialogButton("dialog_cancel", 16, false)
		dialog.confirmButton = createDialogButton("dialog_confirm", 153, true)

		function dialog.show()
			overlay.Visible = true
			cardScale.Scale = 0.85

			tween(overlay, TWEEN_MEDIUM, { BackgroundTransparency = 0.4 })
			tween(cardScale, TWEEN_OPEN, { Scale = 1 })
		end

		function dialog.hide()
			tween(overlay, TWEEN_CLOSE, { BackgroundTransparency = 1 })
			tween(cardScale, TWEEN_CLOSE, { Scale = 0.85 }).Completed:Connect(function()
				overlay.Visible = false
			end)
		end

		return dialog
	end

	local function createResizeHandle(footer, mainScale)
		local handle = create("TextButton", {
			Name = "Resize",
			AnchorPoint = Vector2.new(1, 0.5),
			Position = UDim2.new(1, -6, 0.5, 0),
			Size = UDim2.fromOffset(36, 24),
			BackgroundTransparency = 1,
			Text = "",
			Parent = footer,
		})

		local holder = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.5),
			Size = UDim2.fromOffset(18, 18),
			BackgroundTransparency = 1,
			Parent = handle,
		})

		local parts = {
			create("Frame", {
				AnchorPoint = Vector2.new(0.5, 0.5),
				Position = UDim2.fromOffset(9, 9),
				Size = UDim2.fromOffset(15, 2),
				Rotation = 45,
				BorderSizePixel = 0,
				Parent = holder,
			}),
			create("Frame", { Position = UDim2.fromOffset(9, 14), Size = UDim2.fromOffset(7, 2), BorderSizePixel = 0, Parent = holder }),
			create("Frame", { Position = UDim2.fromOffset(14, 9), Size = UDim2.fromOffset(2, 7), BorderSizePixel = 0, Parent = holder }),
		}

		for _, part in ipairs(parts) do
			bindTheme(part, "BackgroundColor3", "muted")
			addCorner(part, 1)
		end

		local holderScale = create("UIScale", { Parent = holder })

		local function tint(key, scale)
			for _, part in ipairs(parts) do
				tween(part, TWEEN_FAST, { BackgroundColor3 = currentTheme()[key] })
			end

			tween(holderScale, TWEEN_BACK, { Scale = scale })
		end

		connect(handle.MouseEnter, function()
			tint("text", 1.2)
		end)

		connect(handle.MouseLeave, function()
			tint("muted", 1)
		end)

		connect(handle.InputBegan, function(input)
			if not isPointer(input) then
				return
			end

			local startPosition, startScale = input.Position, X.scale
			tint("text", 0.9)

			beginDrag(function(position)
				local delta = position - startPosition
				X.scale = math.clamp(startScale + delta.X / WINDOW_WIDTH + delta.Y / WINDOW_HEIGHT, MIN_SCALE, getMaxScale())
				mainScale.Scale = X.scale
			end, function()
				tint("muted", 1)
			end)
		end)
	end

	local function createFloatingButton(gui, onToggle)
		local state = { moved = false }

		local button = create("TextButton", {
			Name = "Toggle",
			Position = UDim2.new(0, 16, 0.5, -22),
			Size = UDim2.fromOffset(0, 0),
			AutoButtonColor = false,
			Text = "",
			Visible = false,
			Parent = gui,
		})
		bindTheme(button, "BackgroundColor3", "accent")
		addCorner(button, 22)
		addAccentGradient(addStroke(button, WHITE, 2, 0))
		addPressEffect(button, button, 0.9)

		createLogo(button, 20, "accentText", "accentText", false)

		connect(button.MouseEnter, function()
			tween(button, TWEEN_BACK, { Size = UDim2.fromOffset(50, 50) })
		end)

		connect(button.MouseLeave, function()
			tween(button, TWEEN_BACK, { Size = UDim2.fromOffset(44, 44) })
		end)

		connect(button.Activated, function()
			if not state.moved then
				onToggle()
			end
		end)

		makeDraggable(button, button, function()
			return 1
		end, state, true)

		return button
	end

	local function populateAimTab(ctx)
		local aimMethods = getMouseMove() and { "camera", "mouse" } or { "camera" }

		addSection(ctx, "sec_aimbot")
		addFeature(ctx, "feat_aimbot", "aimEnabled", function(sub)
			addCycle(sub, "set_aim_mode", "aimMode", { "always", "hold" })
			addCycle(sub, "set_aim_method", "aimMethod", aimMethods)
			addCycle(sub, "set_aim_part", "aimPart", { "head", "root", "torso" })
			addSlider(sub, "set_max_distance", "aimMaxDistance", 50, 5000, 50)
			addSlider(sub, "set_smoothness", "aimSmoothness", 1, 20, 0.5)
			addToggle(sub, "set_team_check", "aimTeamCheck")
			addToggle(sub, "set_wall_check", "aimWallCheck")
		end)

		addSection(ctx, "sec_fov")
		addFeature(ctx, "feat_fov", "showFov", function(sub)
			addSlider(sub, "set_radius", "fovRadius", 20, 500, 1)
			addSlider(sub, "set_thickness", "fovThickness", 1, 4, 0.5)
			addToggle(sub, "set_filled", "fovFilled")
			addColorPicker(sub, "col_fov", "fovColor")
		end)
	end

	local function populateEspTab(ctx)
		addSection(ctx, "sec_visuals")

		addFeature(ctx, "feat_esp", "espEnabled", function(sub)
			addSlider(sub, "set_max_distance", "espMaxDistance", 100, 5000, 50)
			addToggle(sub, "set_team_check", "espTeamCheck")
			addToggle(sub, "set_team_color", "espTeamColor")
		end)

		addFeature(ctx, "feat_boxes", "espBoxes", function(sub)
			addCycle(sub, "set_box_style", "espBoxStyle", { "full", "corner", "filled" })
			addSlider(sub, "set_thickness", "espBoxThickness", 1, 4, 0.5)
			addColorPicker(sub, "col_box", "espBoxColor")
		end)

		addFeature(ctx, "feat_names", "espNames", function(sub)
			addToggle(sub, "set_display_name", "espDisplayName")
			addSlider(sub, "set_text_size", "espNameSize", 10, 20, 1)
			addColorPicker(sub, "col_name", "espNameColor")
		end)

		addFeature(ctx, "feat_distance", "espDistance", function(sub)
			addCycle(sub, "set_unit", "espDistanceUnit", { "studs", "meters" })
			addColorPicker(sub, "col_distance", "espDistanceColor")
		end)

		addFeature(ctx, "feat_health", "espHealth", function(sub)
			addCycle(sub, "set_side", "espHealthSide", { "left", "right" })
			addToggle(sub, "set_health_text", "espHealthText")
			addColorPicker(sub, "col_health_full", "espHealthFull")
			addColorPicker(sub, "col_health_empty", "espHealthEmpty")
		end)

		addFeature(ctx, "feat_tracers", "espTracers", function(sub)
			addCycle(sub, "set_origin", "espTracerOrigin", { "bottom", "center", "top" })
			addSlider(sub, "set_thickness", "espTracerThickness", 1, 4, 0.5)
			addColorPicker(sub, "col_tracer", "espTracerColor")
		end)
	end

	local function populateConfigTab(ctx)
		addSection(ctx, "sec_interface")
		addChoice(ctx, "cfg_theme", "theme", { "dark", "light" }, 70, function()
			applyTheme(true)
		end)
		addChoice(ctx, "cfg_language", "language", { "en", "pt" }, 86, applyLanguage)
		addToggle(ctx, "cfg_border", "animatedBorder")

		addSection(ctx, "sec_targets")
		addToggle(ctx, "cfg_npcs", "includeNpcs", function(state)
			if state then
				scanNpcs()
			end
		end)

		addInfo(ctx, "cfg_hint")
	end

	local function createLoader(parent)
		local loader = {}
		local states, dots, labels, notes = {}, {}, {}, {}
		local rowCount = #STEPS

		local viewport = workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize or Vector2.new(1280, 720)
		local loaderScale = math.clamp(math.min(viewport.X / 340, viewport.Y / 330), 0.6, 1)

		local card = create("Frame", {
			Name = "Loader",
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.5),
			Size = UDim2.fromOffset(320, 318),
			BackgroundTransparency = 1,
			Parent = parent,
		})
		addCorner(card, 14)

		local cardScale = create("UIScale", { Scale = loaderScale * 0.8, Parent = card })
		local cardStroke = addStroke(card, WHITE, 2, 1)
		addAccentGradient(cardStroke)

		local group = create("CanvasGroup", {
			Size = UDim2.fromScale(1, 1),
			BorderSizePixel = 0,
			GroupTransparency = 1,
			Parent = card,
		})
		bindTheme(group, "BackgroundColor3", "background")
		addCorner(group, 14)

		local logoParts = createLogo(group, 52, "muted", "accent", true)
		logoParts.logo.Position = UDim2.fromOffset(160, 56)

		local logoScale = create("UIScale", { Scale = 0, Parent = logoParts.logo })

		tween(logoParts.ringStroke, TweenInfo.new(0.9, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), { Transparency = 0.9 })
		tween(logoParts.ring, TweenInfo.new(0.9, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), {
			Size = UDim2.fromOffset(52 * 1.5, 52 * 1.5),
		})

		local status = createText(group, {
			Position = UDim2.fromOffset(0, 104),
			Size = UDim2.new(1, 0, 0, 18),
			Font = Enum.Font.Gotham,
			TextSize = 12,
		}, "muted", "loader_starting")

		local function resolveDot(index, theme)
			local state = states[index]

			if state == "warn" then
				return WARN_COLOR
			elseif state == "fail" then
				return FAIL_COLOR
			elseif state == "done" or state == "run" then
				return theme.accent
			end

			return theme.trackOff
		end

		local function resolveLabel(index, theme)
			return states[index] and theme.text or theme.muted
		end

		for index, stepKey in ipairs(STEPS) do
			local y = 134 + (index - 1) * 22

			local dot = create("Frame", {
				AnchorPoint = Vector2.new(0.5, 0.5),
				Position = UDim2.fromOffset(34, y + 10),
				Size = UDim2.fromOffset(8, 8),
				BackgroundTransparency = 1,
				BorderSizePixel = 0,
				Parent = group,
			})
			addCorner(dot, 6)
			bindTheme(dot, "BackgroundColor3", function(theme)
				return resolveDot(index, theme)
			end)

			local label = createText(group, {
				Position = UDim2.fromOffset(50, y),
				Size = UDim2.fromOffset(110, 20),
				TextSize = 13,
				TextTransparency = 1,
				TextXAlignment = Enum.TextXAlignment.Left,
			}, "text", stepKey)
			bindTheme(label, "TextColor3", function(theme)
				return resolveLabel(index, theme)
			end)

			local note = createText(group, {
				AnchorPoint = Vector2.new(1, 0),
				Position = UDim2.new(1, -26, 0, y),
				Size = UDim2.fromOffset(130, 20),
				Font = Enum.Font.Gotham,
				TextSize = 11,
				TextTransparency = 1,
				TextTruncate = Enum.TextTruncate.AtEnd,
				TextXAlignment = Enum.TextXAlignment.Right,
			}, "muted")

			dots[index], labels[index], notes[index] = dot, label, note
		end

		local track = create("Frame", {
			Position = UDim2.fromOffset(26, 280),
			Size = UDim2.fromOffset(268, 5),
			BorderSizePixel = 0,
			Parent = group,
		})
		bindTheme(track, "BackgroundColor3", "trackOff")
		addCorner(track, 3)

		local fill = create("Frame", { Size = UDim2.fromScale(0, 1), BorderSizePixel = 0, ClipsDescendants = true, Parent = track })
		bindTheme(fill, "BackgroundColor3", "accent")
		addCorner(fill, 3)

		local shimmer = create("UIGradient", {
			Transparency = NumberSequence.new({
				NumberSequenceKeypoint.new(0, 0.55),
				NumberSequenceKeypoint.new(0.5, 0),
				NumberSequenceKeypoint.new(1, 0.55),
			}),
			Offset = Vector2.new(-1, 0),
			Parent = fill,
		})
		tween(shimmer, TweenInfo.new(1.1, Enum.EasingStyle.Linear, Enum.EasingDirection.Out, -1), { Offset = Vector2.new(1, 0) })

		local percent = createText(group, {
			Position = UDim2.fromOffset(0, 290),
			Size = UDim2.new(1, 0, 0, 16),
			Font = Enum.Font.GothamBold,
			Text = "0%",
			TextSize = 11,
		}, "muted")

		local function setProgress(alpha)
			tween(fill, TWEEN_MEDIUM, { Size = UDim2.fromScale(alpha, 1) })
			percent.Text = string.format("%d%%", math.floor(alpha * 100 + 0.5))
		end

		local function setState(index, state, note)
			states[index] = state

			local theme = currentTheme()
			tween(dots[index], TWEEN_FAST, {
				BackgroundColor3 = resolveDot(index, theme),
				Size = state == "run" and UDim2.fromOffset(12, 12) or UDim2.fromOffset(8, 8),
			})
			tween(labels[index], TWEEN_FAST, { TextColor3 = resolveLabel(index, theme) })

			if note then
				notes[index].Text = note
			end
		end

		function loader.show()
			tween(cardScale, TWEEN_OPEN, { Scale = loaderScale })
			tween(group, TWEEN_MEDIUM, { GroupTransparency = 0 })
			tween(cardStroke, TWEEN_MEDIUM, { Transparency = 0 })

			logoParts.logo.Rotation = -180
			tween(logoScale, TweenInfo.new(0.7, Enum.EasingStyle.Back, Enum.EasingDirection.Out), { Scale = 1 })
			tween(logoParts.logo, TweenInfo.new(0.7, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), { Rotation = 0 })

			for index = 1, rowCount do
				task.delay(0.25 + (index - 1) * 0.06, function()
					pcall(function()
						tween(dots[index], TWEEN_MEDIUM, { BackgroundTransparency = 0 })
						tween(labels[index], TWEEN_MEDIUM, { TextTransparency = 0 })
						tween(notes[index], TWEEN_MEDIUM, { TextTransparency = 0 })
					end)
				end)
			end

			task.wait(0.8)
		end

		function loader.run(index, critical, callback)
			setState(index, "run")
			status.Text = T("loader_checking", string.lower(T(STEPS[index])))
			setProgress((index - 0.5) / rowCount)
			task.wait(0.2)

			local success, note, level = pcall(callback)

			if not success then
				setState(index, "fail", T("note_failed"))
				status.Text = T("loader_error", tostring(note))
				warn("[X] " .. T(STEPS[index]) .. " failed: " .. tostring(note))
				task.wait(0.6)

				if critical then
					error(tostring(note), 0)
				end

				return
			end

			setState(index, level == "warn" and "warn" or "done", note)
			setProgress(index / rowCount)
			task.wait(0.15)
		end

		function loader.hide()
			status.Text = T("loader_ready")
			task.wait(0.35)

			tween(logoScale, TWEEN_CLOSE, { Scale = 1.3 })
			tween(cardScale, TWEEN_CLOSE, { Scale = loaderScale * 0.92 })
			tween(group, TWEEN_CLOSE, { GroupTransparency = 1 })
			tween(cardStroke, TWEEN_CLOSE, { Transparency = 1 })
			task.wait(0.25)
			card:Destroy()
		end

		return loader
	end

	local function checkEnvironment()
		local waited = 0

		while not workspace.CurrentCamera and waited < 5 do
			task.wait(0.1)
			waited = waited + 0.1
		end

		assert(localPlayer, "LocalPlayer not found")
		assert(workspace.CurrentCamera, "Camera not found")

		return T("note_env")
	end

	local function checkExecutor()
		local env = environment
		local missing = {}

		if type(env.gethui) ~= "function" then
			table.insert(missing, "gethui")
		end

		if type(env.mousemoverel) ~= "function" then
			table.insert(missing, "mousemoverel")
		end

		if #missing > 0 then
			return T("note_exec_optional", table.concat(missing, ", ")), "warn"
		end

		return T("note_exec_ok")
	end

	local function bindRenderStep(name, priority, callback)
		local success = pcall(function()
			runService:BindToRenderStep(name, priority, callback)
		end)

		if not success then
			connect(runService.RenderStepped, callback)
		end
	end

	local function initializeAim()
		bindRenderStep("XAim", Enum.RenderPriority.Camera.Value + 1, safeStep("Aim", updateAim))

		return getMouseMove() and T("note_camera_mouse") or T("note_camera")
	end

	local function initializeEsp()
		bindRenderStep("XVisuals", Enum.RenderPriority.Last.Value, safeStep("Visuals", function()
			updateFov()
			updateEsp()
		end))

		connect(workspace.DescendantAdded, function(descendant)
			if X.settings.includeNpcs and descendant:IsA("Humanoid") then
				task.defer(registerHumanoid, descendant)
			end
		end)

		connect(workspace.DescendantRemoving, function(descendant)
			if descendant:IsA("Humanoid") then
				npcModels[descendant] = nil
			end
		end)

		return T("note_overlay")
	end

	local function safePopulate(name, populate, ctx)
		local success, message = pcall(populate, ctx)

		if not success then
			warn("[X] " .. name .. " tab: " .. tostring(message))
		end
	end

	local function buildInterface(gui)
		local ui = {}

		initializeInput()

		local window = createWindow(gui)
		local main, mainScale = window.main, window.mainScale
		local tabSystem = createTabSystem(window)

		local pickerSuccess, pickerResult = pcall(createColorPicker, gui, main)

		if pickerSuccess then
			colorPicker = pickerResult
		else
			warn("[X] color picker: " .. tostring(pickerResult))
			colorPicker = {
				open = function() end,
				close = function() end,
				tick = function() end,
			}
		end

		safePopulate("aim", populateAimTab, tabSystem.create("Aim", "tab_aim", 1))
		safePopulate("esp", populateEspTab, tabSystem.create("Esp", "tab_esp", 2))
		safePopulate("config", populateConfigTab, tabSystem.create("Config", "tab_config", 3))
		tabSystem.setInitial("Aim")

		createResizeHandle(window.footer, mainScale)

		local confirm = createConfirmDialog(main)

		local function setVisible(state)
			X.visible = state

			if state then
				main.Visible = true
				tween(mainScale, TWEEN_OPEN, { Scale = X.scale })
				return
			end

			colorPicker.close()

			tween(mainScale, TWEEN_CLOSE, { Scale = 0 }).Completed:Connect(function()
				if not X.visible then
					main.Visible = false
				end
			end)
		end

		local floatButton = createFloatingButton(gui, function()
			setVisible(not X.visible)
		end)

		X.scale = math.clamp(math.min(1, getMaxScale()), MIN_SCALE, MAX_SCALE)

		makeDraggable(window.topBar, main, function()
			return mainScale.Scale
		end, { moved = false }, true)

		connect(window.minimizeButton.Activated, function()
			setVisible(false)
		end)

		connect(window.closeButton.Activated, function()
			colorPicker.close()
			confirm.show()
		end)

		connect(confirm.cancelButton.Activated, confirm.hide)

		connect(confirm.confirmButton.Activated, function()
			confirm.hide()
			X.visible = false

			tween(floatButton, TWEEN_CLOSE, { Size = UDim2.fromOffset(0, 0) })
			tween(mainScale, TWEEN_CLOSE, { Scale = 0 }).Completed:Wait()
			X.destroy()
		end)

		connect(userInputService.InputBegan, function(input, processed)
			if not processed and input.KeyCode == TOGGLE_KEY then
				setVisible(not X.visible)
			end
		end)

		local camera = workspace.CurrentCamera

		if camera then
			connect(camera:GetPropertyChangedSignal("ViewportSize"), function()
				X.scale = math.clamp(X.scale, MIN_SCALE, getMaxScale())

				if X.visible then
					tween(mainScale, TWEEN_FAST, { Scale = X.scale })
				end
			end)
		end

		local fpsFrames, fpsTime = 0, 0

		connect(runService.RenderStepped, function(deltaTime)
			if X.settings.animatedBorder then
				for _, gradient in ipairs(rotatingGradients) do
					gradient.Rotation = (gradient.Rotation + deltaTime * 90) % 360
				end
			end

			for flag in pairs(colorSwatches) do
				if X.settings[flag .. "Rainbow"] then
					refreshSwatches(flag)
				end
			end

			colorPicker.tick()

			fpsFrames = fpsFrames + 1
			fpsTime = fpsTime + deltaTime

			if fpsTime >= 0.5 then
				local ping = getPing()
				local pingColor = ping < 100 and GOOD_COLOR or ping < 180 and WARN_COLOR or FAIL_COLOR

				window.statusLabel.Text = string.format(
					'%d FPS  |  <font color="rgb(%d,%d,%d)">%d ms</font>',
					math.floor(fpsFrames / fpsTime),
					math.floor(pingColor.R * 255),
					math.floor(pingColor.G * 255),
					math.floor(pingColor.B * 255),
					math.floor(ping)
				)

				fpsFrames, fpsTime = 0, 0
			end
		end)

		applyTheme(false)

		function ui.open()
			setVisible(true)
			floatButton.Visible = true
			tween(floatButton, TWEEN_OPEN, { Size = UDim2.fromOffset(44, 44) })
		end

		return ui
	end

	local function onInitializeError(error)
		warn("Failed to initialize.")
		warn(error)
		warn(debug.traceback())
		X.destroy()
		bootError(error)
	end

	local function createFallbackLoader()
		local loader = {}

		function loader.show() end

		function loader.run(_, critical, callback)
			local success, note = pcall(callback)

			if not success and critical then
				error(tostring(note), 0)
			end
		end

		function loader.hide() end

		return loader
	end

	local function closeBoot()
		if boot.gui then
			pcall(function()
				boot.gui:Destroy()
			end)
		end
	end

	local function initializeScript()
		shared.X = X

		X.settings.language = detectLanguage()

		bootStage("overlay")
		overlayGui = create("ScreenGui", {
			Name = "XOverlay",
			ResetOnSpawn = false,
			IgnoreGuiInset = false,
			ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
			DisplayOrder = 998,
		})
		parentGui(overlayGui)

		fovCircle = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			BackgroundTransparency = 1,
			BorderSizePixel = 0,
			Visible = false,
			Parent = overlayGui,
		})
		create("UICorner", { CornerRadius = UDim.new(1, 0), Parent = fovCircle })
		fovStroke = addStroke(fovCircle, WHITE, 1.5, 0)

		bootStage("gui")
		screenGui = create("ScreenGui", {
			Name = "X",
			ResetOnSpawn = false,
			IgnoreGuiInset = true,
			ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
			DisplayOrder = 999,
		})
		parentGui(screenGui)

		bootStage("loader")

		local loaderSuccess, loader = pcall(createLoader, screenGui)

		if not loaderSuccess then
			warn("[X] loader: " .. tostring(loader))
			loader = createFallbackLoader()
		end

		local ui

		loader.show()
		loader.run(1, true, checkEnvironment)
		loader.run(2, false, checkExecutor)
		loader.run(3, true, function()
			ui = buildInterface(screenGui)
			return T("note_window")
		end)
		loader.run(4, false, initializeAim)
		loader.run(5, false, initializeEsp)
		loader.run(6, false, function()
			applyTheme(false)
			return T("opt_" .. X.settings.theme)
		end)
		loader.hide()

		ui.open()
		closeBoot()
		print("[X] loaded")
	end

	bootStage("loading")

	task.spawn(function()
		xpcall(initializeScript, onInitializeError)
	end)

	return X
end

local success, result = xpcall(runX, function(message)
	warn("[X] " .. tostring(message))
	bootError(message)
	return message
end)

if success then
	return result
end
