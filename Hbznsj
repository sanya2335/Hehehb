
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- Settings -----------------------------------------------------------
local settings = {
	-- lock-on
	IgnoreTeammates = true,
	FovRadius = 150,
	MaxAngle = 40,
	MaxDistance = 300,
	Smoothing = 0.25,
	AimPart = "Head",
	AimMode = "Camera",
	WallCheck = true,
	TargetGrace = 0.25,
	-- prediction
	Prediction = false,
	PredictionTime = 0.12,
	PredictionPing = true,
	PredictionVertical = 0.5,
	PredictionMax = 15,
	ProjectileSpeed = 0,
	PredLock = true,
	-- silent aim
	SilentPart = "Head",
	SilentFov = 120,
	SilentHitChance = 100,
	SilentWallCheck = true,
	SilentShowFov = true,
	SilentPriority = "Crosshair",
	SilentPrediction = true,
	SilentProjectileSpeed = 0,
	SilentGravity = 0,
	SilentFallback = true,
	SilentHitWindow = 0.2,
	-- fov circle
	CircleThickness = 2,
	CircleOpacity = 0.6,
	-- esp
	EspDistance = 500,
	EspTeamColor = false,
	EspHideTeam = false,
	ChamsFill = 0.5,
	ChamsOutline = 0,
	BoxStyle = "Full",
	BoxThickness = 1,
	FillOpacity = 0.3,
	ShowDistance = true,
	TracerThickness = 1.5,
	TracerOrigin = "Bottom",
	TracerTarget = "Feet",
	-- snapline
	SnapThickness = 1.5,
	SnapOrigin = "Center",
	SnapAlways = false,
	-- interface
	UiScale = UserInputService.TouchEnabled and 0.8 or 1,
	ShowArray = true,
}
settings.NoShakeRoll = true
settings.ShakeScale = 0
settings.RecoilScale = 0
settings.PerfectIgnoreChance = true
settings.ZoomFov = 30
settings.ZoomSmooth = 14
settings.ZoomScroll = true
local zoomState = { on = false, base = nil }
local weaponFlags = { NoShake = false, NoRecoil = false, NoSpread = false }
local lockEnabled = false
local silentEnabled = false
local lockState, silentState = {}, {}
local circleEnabled = true
local esp = { Chams = false, Box = false, Fill = false, Health = false, Item = false, Name = false, Tracers = false }
local snapEnabled = false

-- Colors: every color is stored as three HSV values (Key.."H", Key.."S", Key.."V")
local function colorOf(key)
	return Color3.fromHSV(settings[key .. "H"], settings[key .. "S"], settings[key .. "V"])
end
local function defColor(key, c)
	local h, s, v = c:ToHSV()
	settings[key .. "H"], settings[key .. "S"], settings[key .. "V"] = h, s, v
end
local ESP_RED = Color3.fromHSV(0, 0.85, 1)
defColor("Accent", Color3.fromRGB(141, 191, 94))
defColor("ColCircle", Color3.fromRGB(236, 236, 242))
defColor("ColCircleLock", Color3.fromRGB(141, 191, 94))
defColor("ColChamsFill", ESP_RED)
defColor("ColChamsOutline", ESP_RED)
defColor("ColBox", ESP_RED)
defColor("ColBoxShadow", Color3.new(0, 0, 0))
defColor("ColFill", ESP_RED)
defColor("ColHealthFull", Color3.fromHSV(0.33, 0.9, 1))
defColor("ColHealthLow", Color3.fromHSV(0, 0.9, 1))
defColor("ColItem", Color3.new(1, 1, 1))
defColor("ColName", Color3.new(1, 1, 1))
defColor("ColTracer", ESP_RED)
defColor("ColSnap", Color3.new(1, 1, 1))
defColor("ColSilent", Color3.fromRGB(94, 170, 191))

-- Theme (GrkLeaks) -----------------------------------------------------
local FONT = Enum.Font.RobotoMono
local C = {
	Accent = colorOf("Accent"),
	Window = Color3.fromRGB(14, 14, 16),
	Side = Color3.fromRGB(11, 11, 13),
	Panel = Color3.fromRGB(20, 20, 23),
	Control = Color3.fromRGB(30, 30, 35),
	Hover = Color3.fromRGB(26, 26, 30),
	Border = Color3.fromRGB(38, 38, 44),
	Text = Color3.fromRGB(226, 226, 232),
	SubText = Color3.fromRGB(128, 128, 140),
}
local updaters = {}
local function accented(fn) fn() table.insert(updaters, fn) end
local function setAccent()
	C.Accent = colorOf("Accent")
	for _, fn in ipairs(updaters) do fn() end
end

local TWEEN_QUICK = TweenInfo.new(0.07, Enum.EasingStyle.Linear)
local TWEEN_SLIDE = TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
local TWEEN_FAST = TweenInfo.new(0.14, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
local TWEEN_OPEN = TweenInfo.new(0.38, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
local TWEEN_SLOW = TweenInfo.new(0.32, Enum.EasingStyle.Quart, Enum.EasingDirection.Out)
local TWEEN_MED = TweenInfo.new(0.25, Enum.EasingStyle.Quart, Enum.EasingDirection.Out)

-- Helpers ------------------------------------------------------------
local function make(class, props, parent)
	local inst = Instance.new(class)
	for k, v in pairs(props) do inst[k] = v end
	inst.Parent = parent
	return inst
end
local function tween(inst, info, goal) TweenService:Create(inst, info, goal):Play() end
local function round(inst, r) return make("UICorner", { CornerRadius = UDim.new(0, r) }, inst) end
local function stroke(inst, color) return make("UIStroke", { Color = color or C.Border, Thickness = 1, ApplyStrokeMode = Enum.ApplyStrokeMode.Border }, inst) end
local function snap(v) return math.floor(v + 0.5) end
local function isPointer(i)
	return i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch
end
local function isMove(i)
	return i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch
end
local function label(parent, props)
	props.BackgroundTransparency = 1
	props.Font = props.Font or FONT
	props.TextSize = props.TextSize or 12
	props.TextColor3 = props.TextColor3 or C.Text
	props.TextXAlignment = props.TextXAlignment or Enum.TextXAlignment.Left
	return make("TextLabel", props, parent)
end

-- Smooth drag helpers --------------------------------------------------
local function stepToward(obj, goal, dt)
	local cur = obj.Position
	local dx, dy = goal.X.Offset - cur.X.Offset, goal.Y.Offset - cur.Y.Offset
	if dx == 0 and dy == 0 and cur.X.Scale == goal.X.Scale and cur.Y.Scale == goal.Y.Scale then return end
	local a = 1 - math.exp(-dt * 20)
	local nx = math.abs(dx) < 2 and goal.X.Offset or cur.X.Offset + dx * a
	local ny = math.abs(dy) < 2 and goal.Y.Offset or cur.Y.Offset + dy * a
	obj.Position = UDim2.new(goal.X.Scale, nx, goal.Y.Scale, ny)
end

-- makeDraggable(obj [, handle]) -> state { Attach(handle), Reset(), Moved }
local hudStates = {}
local function makeDraggable(obj, handle)
	local state = { Moved = false }
	local home = obj.Position
	local goal = home
	local dragging, dragStart, startPos, baseAbs = false, nil, nil, nil
	function state.Attach(h)
		h.InputBegan:Connect(function(input)
			if isPointer(input) then
				dragging, dragStart, startPos, baseAbs = true, input.Position, goal, obj.AbsolutePosition
				state.Moved = false
				input.Changed:Connect(function()
					if input.UserInputState == Enum.UserInputState.End then dragging = false end
				end)
			end
		end)
	end
	function state.Reset() goal = home end
	UserInputService.InputChanged:Connect(function(input)
		if not (dragging and isMove(input)) then return end
		local d = input.Position - dragStart
		if d.Magnitude > 4 then state.Moved = true end
		if not state.Moved then return end
		local vp, sz = camera.ViewportSize, obj.AbsoluteSize
		local nx = math.clamp(baseAbs.X + d.X, -sz.X + 40, vp.X - 40)
		local ny = math.clamp(baseAbs.Y + d.Y, -sz.Y + 20, vp.Y - 20)
		goal = UDim2.new(startPos.X.Scale, snap(startPos.X.Offset + nx - baseAbs.X), startPos.Y.Scale, snap(startPos.Y.Offset + ny - baseAbs.Y))
	end)
	RunService.RenderStepped:Connect(function(dt) stepToward(obj, goal, dt) end)
	if handle then state.Attach(handle) end
	table.insert(hudStates, state)
	return state
end

-- Root GUI -----------------------------------------------------------
local gui = make("ScreenGui", {
	Name = "LockOnEspGui", ResetOnSpawn = false, IgnoreGuiInset = true,
	ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
}, player:WaitForChild("PlayerGui"))

local espLayer = make("Frame", { Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1 }, gui)

local circle = make("Frame", {
	Name = "FovCircle", AnchorPoint = Vector2.new(0.5, 0.5), Position = UDim2.fromScale(0.5, 0.5),
	BackgroundTransparency = 1,
}, gui)
round(circle, 9999)
local circleStroke = make("UIStroke", { Color = C.Text, Thickness = 2 }, circle)

local root = make("Frame", { Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1 }, gui)

-- Sounds --------------------------------------------------------------
-- Built-in Roblox UI sounds, pitched per action. Replace an id with
-- "rbxassetid://<id>" to use your own. Sounds are preloaded, rate-limited,
-- and skipped while the menu is hidden / on touch hover.
settings.UiSounds = true
settings.UiVolume = 0.5
local uiShown = true
local SOUNDS = {
	click = { "rbxasset://sounds/button.wav", 0.55, 1.0, 0.03 },
	hover = { "rbxasset://sounds/button.wav", 0.10, 1.7, 0.09 },
	on    = { "rbxasset://sounds/switch.wav", 0.65, 1.2, 0.03 },
	off   = { "rbxasset://sounds/switch.wav", 0.65, 0.85, 0.03 },
	tab   = { "rbxasset://sounds/switch.wav", 0.45, 1.0, 0.05 },
	open  = { "rbxasset://sounds/switch.wav", 0.75, 1.35, 0.1 },
	close = { "rbxasset://sounds/switch.wav", 0.75, 0.75, 0.1 },
	tick  = { "rbxasset://sounds/button.wav", 0.16, 2.0, 0.07 },
}
local soundObjs, lastSound, soundList = {}, {}, {}
for name, d in pairs(SOUNDS) do
	local snd = make("Sound", { Name = name, SoundId = d[1], Volume = d[2], PlaybackSpeed = d[3] }, gui)
	soundObjs[name] = snd
	table.insert(soundList, snd)
end
task.spawn(function() pcall(function() game:GetService("ContentProvider"):PreloadAsync(soundList) end) end)

local isTouch = UserInputService.TouchEnabled
local function playSound(name)
	if not settings.UiSounds or settings.UiVolume <= 0 then return end
	if name == "hover" and isTouch then return end
	if not uiShown and name ~= "open" and name ~= "close" then return end
	local snd, cfg = soundObjs[name], SOUNDS[name]
	if not snd then return end
	local now = os.clock()
	if lastSound[name] and now - lastSound[name] < cfg[4] then return end
	lastSound[name] = now
	snd.Volume = cfg[2] * settings.UiVolume * 2
	snd.TimePosition = 0
	snd:Play()
end

-- Loading screen (large card + pooled sparks, ~5 seconds) ---------------
-- Shows immediately; the bar is time-driven (LD_MIN) but never gets ahead of the
-- real build, which reports through setLoad(progress) (it yields one frame so the
-- card can render). Total on-screen time is about LD_MIN + 1 second.
local LD_W, LD_H, TRACK_X, TRACK_W, TRACK_Y = 480, 270, 80, 320, 172
local LD_MIN = 4.0
local loadTarget, loadReady, loadDone = 0, false, nil
local loaderFinished = false
local GRAY = Color3.fromRGB(125, 125, 135)

local loaderGui = make("ScreenGui", {
	Name = "GrkLeaksLoader", ResetOnSpawn = false, IgnoreGuiInset = true, DisplayOrder = 1000,
	ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
}, player:WaitForChild("PlayerGui"))
local loaderDim = make("Frame", {
	Size = UDim2.fromScale(1, 1), BackgroundColor3 = Color3.new(0, 0, 0), BackgroundTransparency = 1, BorderSizePixel = 0,
}, loaderGui)
local loaderHolder = make("Frame", {
	AnchorPoint = Vector2.new(0.5, 0.5), Position = UDim2.fromScale(0.5, 0.5),
	Size = UDim2.fromOffset(LD_W, LD_H), BackgroundTransparency = 1,
}, loaderGui)
local loaderScale = make("UIScale", { Scale = 0.85 }, loaderHolder)
local loader = make("CanvasGroup", {
	Size = UDim2.fromScale(1, 1), BackgroundColor3 = C.Window, BorderSizePixel = 0, GroupTransparency = 1,
}, loaderHolder)
round(loader, 5)
stroke(loader)
make("Frame", { Size = UDim2.new(1, 0, 0, 2), BackgroundColor3 = C.Accent, BorderSizePixel = 0 }, loader)
label(loader, {
	Position = UDim2.fromOffset(0, 44), Size = UDim2.new(1, 0, 0, 48), Text = "GrkLeaks", TextSize = 42,
	TextXAlignment = Enum.TextXAlignment.Center, TextColor3 = C.Accent,
})
label(loader, {
	Position = UDim2.fromOffset(0, 96), Size = UDim2.new(1, 0, 0, 16), Text = "lock-on / esp", TextSize = 13,
	TextXAlignment = Enum.TextXAlignment.Center, TextColor3 = C.SubText,
})
local loaderStatus = label(loader, {
	Position = UDim2.fromOffset(TRACK_X, TRACK_Y - 20), Size = UDim2.fromOffset(TRACK_W, 14), Text = "initializing...",
	TextSize = 11, TextColor3 = C.SubText,
})
local loaderPercent = label(loader, {
	Position = UDim2.fromOffset(TRACK_X, TRACK_Y - 20), Size = UDim2.fromOffset(TRACK_W, 14), Text = "0%",
	TextSize = 11, TextXAlignment = Enum.TextXAlignment.Right, TextColor3 = C.Accent,
})
local loaderTrack = make("Frame", {
	Position = UDim2.fromOffset(TRACK_X, TRACK_Y), Size = UDim2.fromOffset(TRACK_W, 4),
	BackgroundColor3 = C.Control, BorderSizePixel = 0,
}, loader)
round(loaderTrack, 2)
local loaderFill = make("Frame", { Size = UDim2.fromScale(0, 1), BackgroundColor3 = C.Accent, BorderSizePixel = 0 }, loaderTrack)
round(loaderFill, 2)
label(loader, {
	Position = UDim2.fromOffset(0, LD_H - 38), Size = UDim2.new(1, 0, 0, 14), Text = "RightShift hides / shows the menu",
	TextSize = 11, TextXAlignment = Enum.TextXAlignment.Center, TextColor3 = GRAY,
})

-- credits, bottom-left corner of the screen
local loaderCredits = make("TextLabel", {
	AnchorPoint = Vector2.new(0, 1), Position = UDim2.new(0, 18, 1, -14), Size = UDim2.fromOffset(0, 18),
	AutomaticSize = Enum.AutomaticSize.X, BackgroundTransparency = 1, Text = "By: Gado , RandomKnight",
	Font = FONT, TextSize = 13, TextColor3 = GRAY, TextTransparency = 1, TextXAlignment = Enum.TextXAlignment.Left,
}, loaderGui)

-- sparks: a fixed pool of frames updated by one loop (no tweens, no create/destroy)
local sparkLayer = make("Frame", { Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, ZIndex = 10 }, loaderHolder)
local sparkPool, sparkLive = {}, {}
for i = 1, 80 do
	sparkPool[i] = make("Frame", {
		AnchorPoint = Vector2.new(0.5, 0.5), BorderSizePixel = 0, Visible = false, Size = UDim2.fromOffset(3, 3),
	}, sparkLayer)
end
local function emitSpark(x, y, vx, vy, life, size, col, grav)
	local f = table.remove(sparkPool)
	if not f then return end
	f.BackgroundColor3, f.Rotation, f.Visible = col, math.random(0, 45), true
	table.insert(sparkLive, { f = f, x = x, y = y, vx = vx, vy = vy, life = life, age = 0, size = size, g = grav })
end
local function updateSparks(dt)
	local i = 1
	while i <= #sparkLive do
		local sp = sparkLive[i]
		sp.age += dt
		if sp.age >= sp.life then
			sp.f.Visible = false
			table.insert(sparkPool, sp.f)
			sparkLive[i] = sparkLive[#sparkLive]
			sparkLive[#sparkLive] = nil
		else
			sp.vy += sp.g * dt
			sp.x += sp.vx * dt
			sp.y += sp.vy * dt
			local k = sp.age / sp.life
			local sz = math.max(1, snap(sp.size * (1 - k)))
			sp.f.Position = UDim2.fromScale(sp.x / LD_W, sp.y / LD_H)
			sp.f.Size = UDim2.fromOffset(sz, sz)
			sp.f.BackgroundTransparency = k * k
			i += 1
		end
	end
end
local function sparkColor() return math.random() < 0.3 and Color3.new(1, 1, 1) or C.Accent end

local function statusFor(p)
	if p >= 0.999 then return "ready" end
	if p < 0.2 then return "initializing..." end
	if p < 0.45 then return "loading modules..." end
	if p < 0.7 then return "building interface..." end
	if p < 0.92 then return "applying theme..." end
	return "finalizing..."
end

local function setLoad(p)
	loadTarget = p
	task.wait() -- yield one frame so the loader can render
end
local function finishLoader(cb)
	loadDone, loadTarget, loadReady = cb, 1, true
end

do
	local t0, shown, finishing, lastText, lastPct = os.clock(), 0, false, nil, -1
	local headAcc, ambAcc = 0, 0
	local conn
	local function finish()
		playSound("open")
		for _ = 1, 40 do -- finishing burst
			local ang, speed = math.random() * math.pi * 2, 50 + math.random() * 110
			emitSpark(LD_W / 2, LD_H / 2, math.cos(ang) * speed, math.sin(ang) * speed, 0.5 + math.random() * 0.5, math.random(2, 4), sparkColor(), 60)
		end
		task.wait(0.2)
		tween(loader, TWEEN_SLOW, { GroupTransparency = 1 })
		tween(loaderScale, TWEEN_SLOW, { Scale = 1.06 })
		tween(loaderDim, TWEEN_SLOW, { BackgroundTransparency = 1 })
		tween(loaderCredits, TWEEN_SLOW, { TextTransparency = 1 })
		task.wait(0.7)
		conn:Disconnect()
		loaderGui:Destroy()
		loaderFinished = true
		if loadDone then loadDone() end
	end
	conn = RunService.RenderStepped:Connect(function(dt)
		local t = math.clamp((os.clock() - t0) / LD_MIN, 0, 1)
		local goal = math.min(loadTarget, t * t * (3 - 2 * t)) -- ease in/out
		shown += (goal - shown) * (1 - math.exp(-dt * 8))
		if goal - shown < 0.003 then shown = goal end
		loaderFill.Size = UDim2.fromScale(shown, 1)
		local pct = math.floor(shown * 100 + 0.5)
		if pct ~= lastPct then
			lastPct = pct
			loaderPercent.Text = pct .. "%"
		end
		local text = statusFor(shown)
		if text ~= lastText then
			lastText = text
			loaderStatus.Text = text
		end
		if not finishing then
			if shown > 0.01 and shown < 0.999 then
				headAcc += dt * 60
				local hx, hy = TRACK_X + TRACK_W * shown, TRACK_Y + 2
				while headAcc >= 1 do
					headAcc -= 1
					emitSpark(hx, hy, math.random(-12, 20), -(30 + math.random() * 70), 0.35 + math.random() * 0.45, math.random(2, 4), sparkColor(), 150)
				end
			else
				headAcc = 0
			end
			ambAcc += dt * 20
			while ambAcc >= 1 do
				ambAcc -= 1
				emitSpark(math.random(10, LD_W - 10), LD_H - 4, math.random(-14, 14), -(40 + math.random() * 50), 1.1 + math.random() * 0.7, 2, C.Accent, 0)
			end
			if loadReady and shown >= 0.999 then
				finishing = true
				task.spawn(finish)
			end
		end
		updateSparks(dt)
	end)
	tween(loaderScale, TWEEN_OPEN, { Scale = 1 })
	tween(loader, TWEEN_SLOW, { GroupTransparency = 0 })
	tween(loaderDim, TWEEN_SLOW, { BackgroundTransparency = 0.45 })
	tween(loaderCredits, TWEEN_SLOW, { TextTransparency = 0 })
end
setLoad(0.2)

-- Main window (CanvasGroup so the whole menu fades as one) --------------
local WIN_W, WIN_H, SIDE_W, TITLE_H = 580, 410, 118, 32
local window = make("CanvasGroup", {
	Name = "Window", AnchorPoint = Vector2.new(0.5, 0.5), Position = UDim2.fromScale(0.5, 0.5),
	Size = UDim2.fromOffset(WIN_W, WIN_H), BackgroundColor3 = C.Window, BorderSizePixel = 0,
	GroupTransparency = 1,
}, root)
round(window, 4)
stroke(window)
local uiScale = make("UIScale", { Scale = settings.UiScale * 0.9 }, window)

-- title bar
local titleBar = make("Frame", {
	Size = UDim2.new(1, 0, 0, TITLE_H), BackgroundColor3 = C.Side, BorderSizePixel = 0, Active = true,
}, window)
local titleText = label(titleBar, {
	Position = UDim2.fromOffset(12, 0), Size = UDim2.new(0, 100, 1, 0), Text = "GrkLeaks", TextSize = 14,
})
accented(function() titleText.TextColor3 = C.Accent end)
label(titleBar, {
	Position = UDim2.fromOffset(94, 1), Size = UDim2.new(0.6, 0, 1, 0), Text = "lock-on / esp",
	TextSize = 11, TextColor3 = C.SubText,
})
local titleLine = make("Frame", {
	AnchorPoint = Vector2.new(0, 1), Position = UDim2.fromScale(0, 1), Size = UDim2.new(1, 0, 0, 1),
	BackgroundColor3 = C.Border, BorderSizePixel = 0,
}, titleBar)

-- smooth drag: grab the title bar or the sidebar; the window eases toward the cursor
local dragAttach
local dragGoal = window.Position
local dragActive = false
do
	local dragging, dragStart, startPos = false, nil, nil
	local function attach(handle)
		handle.InputBegan:Connect(function(input)
			if isPointer(input) then
				dragging, dragStart, startPos = true, input.Position, dragGoal
				dragActive = true
				input.Changed:Connect(function()
					if input.UserInputState == Enum.UserInputState.End then dragging = false end
				end)
			end
		end)
	end
	attach(titleBar)
	UserInputService.InputChanged:Connect(function(input)
		if dragging and isMove(input) then
			local d = input.Position - dragStart
			local vp = camera.ViewportSize
			local x = math.clamp(startPos.X.Offset + d.X, -vp.X / 2 + 60, vp.X / 2 - 60)
			local y = math.clamp(startPos.Y.Offset + d.Y, -vp.Y / 2 + 20, vp.Y / 2 - 30)
			dragGoal = UDim2.new(startPos.X.Scale, snap(x), startPos.Y.Scale, snap(y))
		end
	end)
	RunService.RenderStepped:Connect(function(dt)
		stepToward(window, dragGoal, dt)
	end)
	dragAttach = attach
end

-- sidebar + content
local sidebar = make("Frame", {
	Position = UDim2.fromOffset(0, TITLE_H), Size = UDim2.new(0, SIDE_W, 1, -TITLE_H),
	BackgroundColor3 = C.Side, BorderSizePixel = 0, Active = true,
}, window)
dragAttach(sidebar)
make("UIListLayout", { SortOrder = Enum.SortOrder.LayoutOrder, Padding = UDim.new(0, 2) }, sidebar)
make("UIPadding", { PaddingTop = UDim.new(0, 8) }, sidebar)
make("Frame", {
	Position = UDim2.fromOffset(SIDE_W - 1, TITLE_H), Size = UDim2.new(0, 1, 1, -TITLE_H),
	BackgroundColor3 = C.Border, BorderSizePixel = 0,
}, window)
local content = make("CanvasGroup", {
	Position = UDim2.fromOffset(SIDE_W, TITLE_H), Size = UDim2.new(1, -SIDE_W, 1, -TITLE_H),
	BackgroundTransparency = 1,
}, window)

-- Notifications ------------------------------------------------------
local notifyHolder = make("Frame", {
	AnchorPoint = Vector2.new(0, 1), Position = UDim2.new(0, 14, 1, -42),
	Size = UDim2.fromOffset(260, 220), BackgroundTransparency = 1,
}, gui)
make("UIListLayout", {
	VerticalAlignment = Enum.VerticalAlignment.Bottom, HorizontalAlignment = Enum.HorizontalAlignment.Left,
	Padding = UDim.new(0, 4), SortOrder = Enum.SortOrder.LayoutOrder,
}, notifyHolder)

local function notify(text)
	local toast = make("TextLabel", {
		Size = UDim2.fromOffset(230, 26), BackgroundColor3 = C.Panel, BackgroundTransparency = 1,
		Text = text, Font = FONT, TextSize = 12, TextColor3 = C.Text, TextTransparency = 1,
		TextXAlignment = Enum.TextXAlignment.Left,
	}, notifyHolder)
	make("UIPadding", { PaddingLeft = UDim.new(0, 14) }, toast)
	round(toast, 3)
	local st = make("UIStroke", { Color = C.Border, Thickness = 1, Transparency = 1 }, toast)
	local bar = make("Frame", {
		Position = UDim2.fromOffset(-14, 0), Size = UDim2.new(0, 2, 1, 0),
		BackgroundColor3 = C.Accent, BackgroundTransparency = 1, BorderSizePixel = 0,
	}, toast)
	tween(toast, TWEEN_FAST, { BackgroundTransparency = 0.05, TextTransparency = 0 })
	tween(st, TWEEN_FAST, { Transparency = 0 })
	tween(bar, TWEEN_FAST, { BackgroundTransparency = 0 })
	task.delay(1.8, function()
		tween(toast, TWEEN_MED, { BackgroundTransparency = 1, TextTransparency = 1 })
		tween(st, TWEEN_MED, { Transparency = 1 })
		tween(bar, TWEEN_MED, { BackgroundTransparency = 1 })
		task.wait(0.3)
		toast:Destroy()
	end)
end

-- UI show/hide (fade + pop) -------------------------------------------
local function setUI(show)
	if show == uiShown then return end
	uiShown = show
	playSound(show and "open" or "close")
	if show then
		root.Visible = true
		uiScale.Scale = settings.UiScale * 0.92
	end
	tween(window, TWEEN_MED, { GroupTransparency = show and 0 or 1 })
	tween(uiScale, show and TWEEN_OPEN or TWEEN_MED, { Scale = show and settings.UiScale or settings.UiScale * 0.92 })
	if not show then
		task.delay(0.26, function()
			if not uiShown then root.Visible = false end
		end)
	end
end
local function toggleUI() setUI(not uiShown) end

local uiButton = make("TextButton", {
	AnchorPoint = Vector2.new(0.5, 0), Position = UDim2.new(0.5, 0, 0, 8), Size = UDim2.fromOffset(30, 24),
	BackgroundColor3 = C.Side, Text = "≡", Font = FONT, TextSize = 16, AutoButtonColor = false,
}, gui)
round(uiButton, 3)
stroke(uiButton)
accented(function() uiButton.TextColor3 = C.Accent end)
local uiButtonDrag = makeDraggable(uiButton, uiButton)
uiButton.Activated:Connect(function()
	if uiButtonDrag.Moved then return end
	toggleUI()
end)
uiButton.MouseEnter:Connect(function() tween(uiButton, TWEEN_FAST, { BackgroundColor3 = C.Hover }) end)
uiButton.MouseLeave:Connect(function() tween(uiButton, TWEEN_FAST, { BackgroundColor3 = C.Side }) end)

-- Watermark (top left) -------------------------------------------------
local watermark = make("TextLabel", {
	AnchorPoint = Vector2.new(0, 1), Position = UDim2.new(0, 10, 1, -10), Size = UDim2.fromOffset(0, 22), AutomaticSize = Enum.AutomaticSize.X,
	BackgroundColor3 = C.Side, BackgroundTransparency = 0.1, Font = FONT, TextSize = 12, TextColor3 = C.Text,
	Text = "", BorderSizePixel = 0, Active = true, Visible = false,
}, gui)
make("UIPadding", { PaddingLeft = UDim.new(0, 10), PaddingRight = UDim.new(0, 10) }, watermark)
round(watermark, 3)
stroke(watermark)
local wmLine = make("Frame", {
	Position = UDim2.new(0, -10, 0, 0), Size = UDim2.new(1, 20, 0, 2),
	BackgroundColor3 = C.Accent, BorderSizePixel = 0,
}, watermark)
accented(function() wmLine.BackgroundColor3 = C.Accent end)
do
	local frames, last = 0, os.clock()
	local Stats = game:GetService("Stats")
	local function getPing()
		local ok, v = pcall(function() return Stats.Network.ServerStatsItem["Data Ping"]:GetValue() end)
		if ok and v then return math.floor(v + 0.5) end
		return math.floor(player:GetNetworkPing() * 2000 + 0.5)
	end
	local function setText(fps, ping)
		watermark.Text = string.format("GrkLeaks  |  %s  |  %d fps  |  %d ms", player.Name, fps, ping)
	end
	setText(60, getPing())
	RunService.RenderStepped:Connect(function()
		frames += 1
		local t = os.clock()
		if t - last >= 0.5 then
			setText(snap(frames / (t - last)), getPing())
			frames, last = 0, t
		end
	end)
end

makeDraggable(watermark)

-- Arraylist (enabled modules, top right) -------------------------------
local modules = {}
local arrayEntries = {}
local arrayHolder = make("Frame", {
	AnchorPoint = Vector2.new(1, 0), Position = UDim2.new(1, -10, 0, 10),
	Size = UDim2.fromOffset(220, 0), AutomaticSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1,
}, gui)
make("UIListLayout", {
	HorizontalAlignment = Enum.HorizontalAlignment.Right, SortOrder = Enum.SortOrder.LayoutOrder,
	Padding = UDim.new(0, 1),
}, arrayHolder)
local arrayDrag = makeDraggable(arrayHolder)
accented(function()
	for _, e in pairs(arrayEntries) do e.Bar.BackgroundColor3 = C.Accent end
end)

local function refreshArray(mod)
	local entry = arrayEntries[mod]
	local want = mod.Enabled and settings.ShowArray
	if want and not entry then
		local lbl = make("TextLabel", {
			AutomaticSize = Enum.AutomaticSize.X, Size = UDim2.fromOffset(0, 18),
			BackgroundColor3 = C.Side, BackgroundTransparency = 0.2, Text = mod.Name:lower(),
			Font = FONT, TextSize = 12, TextColor3 = C.Text, LayoutOrder = -#mod.Name, BorderSizePixel = 0,
		}, arrayHolder)
		make("UIPadding", { PaddingLeft = UDim.new(0, 8), PaddingRight = UDim.new(0, 10) }, lbl)
		lbl.Active = true
		arrayDrag.Attach(lbl)
		local bar = make("Frame", {
			AnchorPoint = Vector2.new(1, 0), Position = UDim2.new(1, 10, 0, 0), Size = UDim2.new(0, 2, 1, 0),
			BackgroundColor3 = C.Accent, BorderSizePixel = 0,
		}, lbl)
		lbl.BackgroundTransparency, lbl.TextTransparency, bar.BackgroundTransparency = 1, 1, 1
		tween(lbl, TWEEN_MED, { BackgroundTransparency = 0.2, TextTransparency = 0 })
		tween(bar, TWEEN_MED, { BackgroundTransparency = 0 })
		arrayEntries[mod] = { Label = lbl, Bar = bar }
	elseif not want and entry then
		entry.Label:Destroy()
		arrayEntries[mod] = nil
	end
end

setLoad(0.5)

-- Tabs / module builders -----------------------------------------------
local layoutCounter = 0
local function nextOrder() layoutCounter += 1 return layoutCounter end
local listening = nil
local tabs, activeTab = {}, nil

local function paintTabs(animate)
	for _, t in ipairs(tabs) do
		local on = t == activeTab
		t.Page.Visible = on
		t.Page.ScrollBarImageColor3 = C.Accent
		local info = animate and TWEEN_FAST or TweenInfo.new(0)
		tween(t.Bar, info, { Size = UDim2.new(0, 2, on and 1 or 0, 0) })
		tween(t.Button, info, { BackgroundTransparency = on and 0 or 1, TextColor3 = on and C.Accent or C.SubText })
	end
end
accented(paintTabs)

local function createTab(name)
	local tab = { Name = name }
	local btn = make("TextButton", {
		Size = UDim2.new(1, 0, 0, 28), BackgroundColor3 = C.Panel, BackgroundTransparency = 1,
		Text = name:lower(), Font = FONT, TextSize = 13, TextColor3 = C.SubText,
		TextXAlignment = Enum.TextXAlignment.Left, AutoButtonColor = false, LayoutOrder = #tabs + 1,
		BorderSizePixel = 0,
	}, sidebar)
	make("UIPadding", { PaddingLeft = UDim.new(0, 16) }, btn)
	local bar = make("Frame", {
		AnchorPoint = Vector2.new(0, 0.5), Position = UDim2.new(0, -16, 0.5, 0), Size = UDim2.new(0, 2, 0, 0),
		BorderSizePixel = 0,
	}, btn)
	accented(function() bar.BackgroundColor3 = C.Accent end)

	local page = make("ScrollingFrame", {
		Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, BorderSizePixel = 0,
		ScrollBarThickness = 2, ScrollBarImageColor3 = C.Accent, CanvasSize = UDim2.new(),
		AutomaticCanvasSize = Enum.AutomaticSize.Y, ScrollingDirection = Enum.ScrollingDirection.Y,
		Visible = false,
	}, content)
	make("UIPadding", {
		PaddingTop = UDim.new(0, 10), PaddingBottom = UDim.new(0, 10),
		PaddingLeft = UDim.new(0, 10), PaddingRight = UDim.new(0, 12),
	}, page)
	tab.Cols = {}
	for i = 1, 2 do
		local col = make("Frame", {
			Position = UDim2.new(i == 1 and 0 or 0.5, i == 1 and 0 or 5, 0, 0),
			Size = UDim2.new(0.5, -5, 0, 0), AutomaticSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1,
		}, page)
		make("UIListLayout", { SortOrder = Enum.SortOrder.LayoutOrder, Padding = UDim.new(0, 10) }, col)
		tab.Cols[i] = col
	end
	tab.Button, tab.Bar, tab.Page = btn, bar, page
	btn.MouseEnter:Connect(function()
		if activeTab ~= tab then tween(btn, TWEEN_FAST, { TextColor3 = C.Text }) end
		playSound("hover")
	end)
	btn.MouseLeave:Connect(function()
		if activeTab ~= tab then tween(btn, TWEEN_FAST, { TextColor3 = C.SubText }) end
	end)
	btn.Activated:Connect(function()
		if activeTab == tab then return end
		playSound("tab")
		activeTab = tab
		paintTabs(true)
		page.Position = UDim2.fromOffset(0, 12)
		content.GroupTransparency = 1
		tween(page, TWEEN_SLOW, { Position = UDim2.fromOffset(0, 0) })
		tween(content, TWEEN_MED, { GroupTransparency = 0 })
	end)
	table.insert(tabs, tab)
	return tab
end

-- opts: Col (1/2), Bind (default KeyCode), NoToggle (header only, no checkbox), OnPress (bind action for NoToggle modules)
local function createModule(tab, name, callback, opts)
	opts = opts or {}
	local mod = { Name = name, Enabled = false, Bind = nil }

	local box = make("Frame", {
		Size = UDim2.new(1, 0, 0, 0), AutomaticSize = Enum.AutomaticSize.Y, BackgroundColor3 = C.Panel,
		BorderSizePixel = 0, LayoutOrder = nextOrder(), ClipsDescendants = true,
	}, tab.Cols[opts.Col or 1])
	round(box, 3)
	stroke(box)
	make("UIListLayout", { SortOrder = Enum.SortOrder.LayoutOrder }, box)
	local topLine = make("Frame", { Size = UDim2.new(1, 0, 0, 2), BorderSizePixel = 0, LayoutOrder = 0 }, box)
	accented(function() topLine.BackgroundColor3 = C.Accent end)

	local header = make("TextButton", {
		Size = UDim2.new(1, 0, 0, 26), BackgroundColor3 = C.Hover, BackgroundTransparency = 1, Text = "",
		AutoButtonColor = false, LayoutOrder = 1, BorderSizePixel = 0,
	}, box)
	local checkbox
	if not opts.NoToggle then
		checkbox = make("Frame", {
			AnchorPoint = Vector2.new(0, 0.5), Position = UDim2.new(0, 8, 0.5, 0), Size = UDim2.fromOffset(12, 12),
			BackgroundColor3 = C.Control, BorderSizePixel = 0,
		}, header)
		round(checkbox, 2)
		stroke(checkbox)
	end
	local title = label(header, {
		Position = UDim2.fromOffset(opts.NoToggle and 8 or 28, 0), Size = UDim2.new(1, -70, 1, 0),
		Text = name, TextSize = 12, TextColor3 = C.Text,
	})

	local bindBtn = make("TextButton", {
		AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, -6, 0.5, 0), Size = UDim2.fromOffset(0, 16),
		AutomaticSize = Enum.AutomaticSize.X, BackgroundTransparency = 1, Text = "", Font = FONT, TextSize = 11,
		TextColor3 = C.SubText, AutoButtonColor = false,
	}, header)
	make("UIPadding", { PaddingLeft = UDim.new(0, 3), PaddingRight = UDim.new(0, 3) }, bindBtn)
	local canBind = not (opts.NoToggle and not opts.OnPress)
	bindBtn.Visible = canBind

	local divider = make("Frame", {
		Size = UDim2.new(1, 0, 0, 1), BackgroundColor3 = C.Border, BorderSizePixel = 0, LayoutOrder = 2,
	}, box)
	local body = make("Frame", {
		Size = UDim2.new(1, 0, 0, 0), AutomaticSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1, LayoutOrder = 3,
	}, box)
	make("UIPadding", {
		PaddingTop = UDim.new(0, 8), PaddingBottom = UDim.new(0, 9),
		PaddingLeft = UDim.new(0, 8), PaddingRight = UDim.new(0, 8),
	}, body)
	make("UIListLayout", { Padding = UDim.new(0, 6), SortOrder = Enum.SortOrder.LayoutOrder }, body)
	mod.Body = body

	function mod:SetBind(key)
		mod.Bind = key
		bindBtn.Text = key and ("[" .. key.Name .. "]") or "[ - ]"
	end
	bindBtn.Activated:Connect(function()
		playSound("click")
		if listening and listening ~= mod then listening:SetBind(listening.Bind) end
		listening = mod
		bindBtn.Text = "[ ... ]"
	end)

	local function paint(animate)
		if checkbox then
			local goal = { BackgroundColor3 = mod.Enabled and C.Accent or C.Control }
			if animate == true then tween(checkbox, TWEEN_FAST, goal) else checkbox.BackgroundColor3 = goal.BackgroundColor3 end
			local tc = mod.Enabled and C.Text or C.SubText
			if animate == true then tween(title, TWEEN_FAST, { TextColor3 = tc }) else title.TextColor3 = tc end
		end
	end
	accented(paint)

	function mod:SetEnabled(v, silent)
		mod.Enabled = v
		paint(true)
		callback(v)
		refreshArray(mod)
		if not silent then
			playSound(v and "on" or "off")
			notify(name:lower() .. (v and " enabled" or " disabled"))
		end
	end

	header.MouseEnter:Connect(function()
		tween(header, TWEEN_FAST, { BackgroundTransparency = 0.5 })
		playSound("hover")
	end)
	header.MouseLeave:Connect(function() tween(header, TWEEN_FAST, { BackgroundTransparency = 1 }) end)
	if opts.NoToggle then
		mod.Press = opts.OnPress or function() end
	else
		mod.Press = function() mod:SetEnabled(not mod.Enabled) end
		header.Activated:Connect(function() mod:SetEnabled(not mod.Enabled) end)
	end

	mod:SetBind(opts.Bind)
	table.insert(modules, mod)
	return mod
end

local function addToggle(mod, text, key, onChange)
	local row = make("TextButton", {
		Size = UDim2.new(1, 0, 0, 18), BackgroundTransparency = 1, Text = "", AutoButtonColor = false,
		LayoutOrder = nextOrder(),
	}, mod.Body)
	local box = make("Frame", {
		AnchorPoint = Vector2.new(0, 0.5), Position = UDim2.new(0, 0, 0.5, 0), Size = UDim2.fromOffset(12, 12),
		BackgroundColor3 = C.Control, BorderSizePixel = 0,
	}, row)
	round(box, 2)
	stroke(box)
	label(row, { Position = UDim2.fromOffset(20, 0), Size = UDim2.new(1, -20, 1, 0), Text = text })
	local function refresh()
		tween(box, TWEEN_FAST, { BackgroundColor3 = settings[key] and C.Accent or C.Control })
	end
	row.Activated:Connect(function()
		settings[key] = not settings[key]
		playSound(settings[key] and "on" or "off")
		refresh()
		if onChange then onChange(settings[key]) end
	end)
	accented(refresh)
end

local function addCycle(mod, text, key, choices, onChange)
	local row = make("Frame", { Size = UDim2.new(1, 0, 0, 22), BackgroundTransparency = 1, LayoutOrder = nextOrder() }, mod.Body)
	label(row, { Size = UDim2.new(1, -92, 1, 0), Text = text })
	local btn = make("TextButton", {
		AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, 0, 0.5, 0), Size = UDim2.fromOffset(86, 20),
		BackgroundColor3 = C.Control, Text = tostring(settings[key]), Font = FONT, TextSize = 11,
		AutoButtonColor = false, BorderSizePixel = 0,
	}, row)
	round(btn, 3)
	stroke(btn)
	accented(function() btn.TextColor3 = C.Accent end)
	local function step(dir)
		playSound("click")
		local idx = table.find(choices, settings[key]) or 0
		settings[key] = choices[(idx - 1 + dir) % #choices + 1]
		btn.Text = settings[key]
		if onChange then onChange(settings[key]) end
	end
	btn.MouseEnter:Connect(function() tween(btn, TWEEN_FAST, { BackgroundColor3 = C.Hover }) end)
	btn.MouseLeave:Connect(function() tween(btn, TWEEN_FAST, { BackgroundColor3 = C.Control }) end)
	btn.Activated:Connect(function() step(1) end)
	btn.MouseButton2Click:Connect(function() step(-1) end)
end

local function addSlider(mod, name, key, min, max, step, fmt, onChange, parent)
	local holder = make("Frame", {
		Size = UDim2.new(1, 0, 0, 28), BackgroundTransparency = 1, LayoutOrder = nextOrder(),
	}, parent or mod.Body)
	label(holder, { Size = UDim2.new(0.65, 0, 0, 14), Text = name })
	local valueLabel = label(holder, {
		AnchorPoint = Vector2.new(1, 0), Position = UDim2.fromScale(1, 0), Size = UDim2.new(0.35, 0, 0, 14),
		TextXAlignment = Enum.TextXAlignment.Right, TextSize = 11,
	})
	accented(function() valueLabel.TextColor3 = C.Accent end)
	local track = make("Frame", {
		Position = UDim2.fromOffset(0, 19), Size = UDim2.new(1, 0, 0, 5),
		BackgroundColor3 = C.Control, BorderSizePixel = 0,
	}, holder)
	round(track, 2)
	stroke(track)
	local fill = make("Frame", { Size = UDim2.fromScale(0, 1), BorderSizePixel = 0 }, track)
	round(fill, 2)
	accented(function() fill.BackgroundColor3 = C.Accent end)
	local knob = make("Frame", {
		AnchorPoint = Vector2.new(0.5, 0.5), Position = UDim2.fromScale(0, 0.5), Size = UDim2.fromOffset(6, 11),
		BackgroundColor3 = C.Text, BorderSizePixel = 0, ZIndex = 2,
	}, track)
	round(knob, 2)
	local hit = make("TextButton", {
		Position = UDim2.fromOffset(0, 12), Size = UDim2.new(1, 0, 0, 16),
		BackgroundTransparency = 1, Text = "", ZIndex = 3,
	}, holder)

	local function setValue(v, instant)
		v = math.clamp(math.round(v / step) * step, min, max)
		if not instant and settings[key] ~= v then playSound("tick") end
		settings[key] = v
		local a = (v - min) / (max - min)
		if instant then
			fill.Size = UDim2.fromScale(a, 1)
			knob.Position = UDim2.fromScale(a, 0.5)
		else
			tween(fill, TWEEN_SLIDE, { Size = UDim2.fromScale(a, 1) })
			tween(knob, TWEEN_SLIDE, { Position = UDim2.fromScale(a, 0.5) })
		end
		valueLabel.Text = string.format(fmt, v)
		if onChange then onChange(v) end
	end
	local function fromX(x)
		setValue(min + math.clamp((x - track.AbsolutePosition.X) / track.AbsoluteSize.X, 0, 1) * (max - min))
	end
	local sliding = false
	hit.MouseEnter:Connect(function() tween(knob, TWEEN_FAST, { Size = UDim2.fromOffset(8, 13) }) end)
	hit.MouseLeave:Connect(function()
		if not sliding then tween(knob, TWEEN_FAST, { Size = UDim2.fromOffset(6, 11) }) end
	end)
	hit.InputBegan:Connect(function(input)
		if isPointer(input) then
			sliding = true
			fromX(input.Position.X)
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					sliding = false
					tween(knob, TWEEN_FAST, { Size = UDim2.fromOffset(6, 11) })
				end
			end)
		end
	end)
	UserInputService.InputChanged:Connect(function(input)
		if sliding and isMove(input) then fromX(input.Position.X) end
	end)
	setValue(settings[key], true)
end

local function addColor(mod, text, key, onChange)
	local container = make("Frame", {
		Size = UDim2.new(1, 0, 0, 0), AutomaticSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1,
		LayoutOrder = nextOrder(),
	}, mod.Body)
	make("UIListLayout", { Padding = UDim.new(0, 0), SortOrder = Enum.SortOrder.LayoutOrder }, container)
	local row = make("Frame", { Size = UDim2.new(1, 0, 0, 18), BackgroundTransparency = 1, LayoutOrder = 1 }, container)
	label(row, { Size = UDim2.new(1, -40, 1, 0), Text = text })
	local swatch = make("TextButton", {
		AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, 0, 0.5, 0), Size = UDim2.fromOffset(32, 14),
		BackgroundColor3 = colorOf(key), Text = "", AutoButtonColor = false, BorderSizePixel = 0,
	}, row)
	round(swatch, 2)
	stroke(swatch)

	local PICKER_H = 126
	local picker = make("Frame", {
		Size = UDim2.new(1, 0, 0, 0), BackgroundTransparency = 1, ClipsDescendants = true, LayoutOrder = 2,
	}, container)
	make("UIPadding", { PaddingTop = UDim.new(0, 6) }, picker)
	make("UIListLayout", { Padding = UDim.new(0, 2), SortOrder = Enum.SortOrder.LayoutOrder }, picker)
	local pickerOpen = false
	swatch.Activated:Connect(function()
		playSound("click")
		pickerOpen = not pickerOpen
		tween(picker, TWEEN_SLOW, { Size = UDim2.new(1, 0, 0, pickerOpen and PICKER_H or 0) })
	end)

	-- live preview strip: solid color, dark -> color -> light ramp, and the hex / rgb value
	local preview = make("Frame", {
		Size = UDim2.new(1, 0, 0, 24), BackgroundColor3 = C.Control, BorderSizePixel = 0, LayoutOrder = -1,
	}, picker)
	round(preview, 3)
	stroke(preview)
	local previewSolid = make("Frame", { Position = UDim2.fromOffset(3, 3), Size = UDim2.fromOffset(34, 18), BorderSizePixel = 0 }, preview)
	round(previewSolid, 2)
	local previewRamp = make("Frame", {
		Position = UDim2.fromOffset(41, 3), Size = UDim2.fromOffset(46, 18), BackgroundColor3 = Color3.new(1, 1, 1),
		BorderSizePixel = 0,
	}, preview)
	round(previewRamp, 2)
	local ramp = make("UIGradient", {}, previewRamp)
	local previewText = label(preview, { Position = UDim2.fromOffset(94, 0), Size = UDim2.new(1, -98, 1, 0), TextSize = 10, TextColor3 = C.SubText })

	local ready = false
	local function refreshPreview()
		local c = colorOf(key)
		swatch.BackgroundColor3 = c
		previewSolid.BackgroundColor3 = c
		ramp.Color = ColorSequence.new({
			ColorSequenceKeypoint.new(0, Color3.new(0, 0, 0)),
			ColorSequenceKeypoint.new(0.5, c),
			ColorSequenceKeypoint.new(1, Color3.new(1, 1, 1)),
		})
		local r, g, b = snap(c.R * 255), snap(c.G * 255), snap(c.B * 255)
		previewText.Text = string.format("#%02X%02X%02X  %d,%d,%d", r, g, b, r, g, b)
	end

	-- the color is stored as HSV for the rest of the script; the sliders edit RGB (0-255)
	local c0 = colorOf(key)
	settings[key .. "R"], settings[key .. "G"], settings[key .. "B"] = snap(c0.R * 255), snap(c0.G * 255), snap(c0.B * 255)
	local function changed()
		if ready then
			local c = Color3.fromRGB(settings[key .. "R"], settings[key .. "G"], settings[key .. "B"])
			local h, sat, v = c:ToHSV()
			settings[key .. "H"], settings[key .. "S"], settings[key .. "V"] = h, sat, v
		end
		refreshPreview()
		if onChange then onChange() end
	end
	addSlider(mod, "red", key .. "R", 0, 255, 1, "%d", changed, picker)
	addSlider(mod, "green", key .. "G", 0, 255, 1, "%d", changed, picker)
	addSlider(mod, "blue", key .. "B", 0, 255, 1, "%d", changed, picker)
	ready = true
	refreshPreview()
end

local function addButton(mod, text, fn)
	local btn = make("TextButton", {
		Size = UDim2.new(1, 0, 0, 22), BackgroundColor3 = C.Control, Text = text, Font = FONT, TextSize = 11,
		TextColor3 = C.Text, AutoButtonColor = false, BorderSizePixel = 0, LayoutOrder = nextOrder(),
	}, mod.Body)
	round(btn, 3)
	stroke(btn)
	btn.MouseEnter:Connect(function() tween(btn, TWEEN_FAST, { BackgroundColor3 = C.Hover }) end)
	btn.MouseLeave:Connect(function() tween(btn, TWEEN_FAST, { BackgroundColor3 = C.Control }) end)
	btn.Activated:Connect(function()
		playSound("click")
		fn()
	end)
end

local function addNote(mod, text)
	local lbl = label(mod.Body, {
		Size = UDim2.new(1, 0, 0, 0), AutomaticSize = Enum.AutomaticSize.Y, Text = text, TextSize = 11,
		TextColor3 = C.SubText, TextWrapped = true, LayoutOrder = nextOrder(),
	})
	return lbl
end

-- Keybind handling
UserInputService.InputBegan:Connect(function(input, processed)
	if input.UserInputType ~= Enum.UserInputType.Keyboard then return end
	if listening then
		local mod = listening
		listening = nil
		if input.KeyCode == Enum.KeyCode.Escape or input.KeyCode == Enum.KeyCode.Backspace then
			mod:SetBind(nil)
		else
			mod:SetBind(input.KeyCode)
		end
		return
	end
	if processed then return end
	for _, mod in ipairs(modules) do
		if mod.Bind and mod.Bind == input.KeyCode then mod.Press() end
	end
end)

setLoad(0.78)

-- Freecam (CFrame based) ------------------------------------------------------
-- WASD move, Space up, LeftCtrl down, LeftShift boost, hold right mouse to look.
settings.FreecamSpeed = 60
settings.FreecamBoost = 3
settings.FreecamSens = 1
settings.FreecamSmooth = 12
settings.FreecamFreeze = true
settings.FreecamHoldRmb = true
local ContextActionService = game:GetService("ContextActionService")
local freecamActive = false
local fc = { pos = Vector3.zero, yaw = 0, pitch = 0, tYaw = 0, tPitch = 0, vel = Vector3.zero }
local FC_KEYS = {
	Enum.KeyCode.W, Enum.KeyCode.A, Enum.KeyCode.S, Enum.KeyCode.D,
	Enum.KeyCode.Space, Enum.KeyCode.LeftControl, Enum.KeyCode.LeftShift,
}

local function freecamStep(dt)
	camera = workspace.CurrentCamera
	local typing = UserInputService:GetFocusedTextBox() ~= nil
	local rmb = UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)
	local looking = (not settings.FreecamHoldRmb) or rmb
	if settings.FreecamHoldRmb then
		UserInputService.MouseBehavior = rmb and Enum.MouseBehavior.LockCurrentPosition or Enum.MouseBehavior.Default
	else
		UserInputService.MouseBehavior = Enum.MouseBehavior.LockCenter
	end

	-- look (smoothed yaw / pitch)
	if looking and not typing then
		local d = UserInputService:GetMouseDelta()
		local k = 0.0025 * settings.FreecamSens
		fc.tYaw -= d.X * k
		fc.tPitch = math.clamp(fc.tPitch - d.Y * k, -1.55, 1.55)
	end
	local ra = 1 - math.exp(-dt * 30)
	fc.yaw += (fc.tYaw - fc.yaw) * ra
	fc.pitch += (fc.tPitch - fc.pitch) * ra
	local rot = CFrame.fromOrientation(fc.pitch, fc.yaw, 0)

	-- move (camera-relative on the ground plane axes, world up / down)
	local move = Vector3.zero
	if not typing then
		local function down(k) return UserInputService:IsKeyDown(k) and 1 or 0 end
		local local3 = Vector3.new(down(Enum.KeyCode.D) - down(Enum.KeyCode.A), 0, down(Enum.KeyCode.S) - down(Enum.KeyCode.W))
		move = rot:VectorToWorldSpace(local3) + Vector3.new(0, down(Enum.KeyCode.Space) - down(Enum.KeyCode.LeftControl), 0)
		if move.Magnitude > 1 then move = move.Unit end
		if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then move *= settings.FreecamBoost end
	end
	fc.vel = fc.vel:Lerp(move * settings.FreecamSpeed, 1 - math.exp(-dt * settings.FreecamSmooth))
	fc.pos += fc.vel * dt
	camera.CFrame = CFrame.new(fc.pos) * rot
end

local fcPrevType, fcPrevSubject
local function setFreecam(on)
	if on == freecamActive then return end
	freecamActive = on
	if on then
		camera = workspace.CurrentCamera
		local cf = camera.CFrame
		local rx, ry = cf:ToOrientation()
		fc.pos, fc.yaw, fc.pitch, fc.tYaw, fc.tPitch, fc.vel = cf.Position, ry, rx, ry, rx, Vector3.zero
		fcPrevType, fcPrevSubject = camera.CameraType, camera.CameraSubject
		camera.CameraType = Enum.CameraType.Scriptable
		if settings.FreecamFreeze then
			ContextActionService:BindActionAtPriority("GrkFreecamSink", function() return Enum.ContextActionResult.Sink end,
				false, Enum.ContextActionPriority.High.Value + 100, table.unpack(FC_KEYS))
		end
		RunService:BindToRenderStep("GrkFreecam", Enum.RenderPriority.Camera.Value + 1, freecamStep)
	else
		RunService:UnbindFromRenderStep("GrkFreecam")
		ContextActionService:UnbindAction("GrkFreecamSink")
		UserInputService.MouseBehavior = Enum.MouseBehavior.Default
		camera = workspace.CurrentCamera
		camera.CameraType = fcPrevType or Enum.CameraType.Custom
		if fcPrevSubject then camera.CameraSubject = fcPrevSubject end
	end
end

-- Character preview (3D, draggable HUD panel) ----------------------------------
-- Drag the header to move the panel, drag inside the viewport to rotate the model.
settings.PreviewAutoRotate = true
settings.PreviewSpin = 35
settings.PreviewScale = 1
local hudReady, previewEnabled = false, false
local PV_W, PV_H = 150, 205

local previewPanel = make("Frame", {
	Name = "CharacterPreview", AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, -12, 0.5, 0),
	Size = UDim2.fromOffset(PV_W, PV_H), BackgroundColor3 = C.Side, BorderSizePixel = 0, Visible = false, Active = true,
}, gui)
round(previewPanel, 4)
stroke(previewPanel)
local pvScale = make("UIScale", { Scale = settings.PreviewScale }, previewPanel)
local pvLine = make("Frame", { Size = UDim2.new(1, 0, 0, 2), BorderSizePixel = 0 }, previewPanel)
accented(function() pvLine.BackgroundColor3 = C.Accent end)
local pvHeader = label(previewPanel, {
	Position = UDim2.fromOffset(0, 2), Size = UDim2.new(1, 0, 0, 22), Text = player.DisplayName, TextSize = 11,
	TextXAlignment = Enum.TextXAlignment.Center, Active = true,
})
local pvViewport = make("ViewportFrame", {
	Position = UDim2.fromOffset(6, 26), Size = UDim2.new(1, -12, 1, -32), BackgroundColor3 = C.Window,
	BorderSizePixel = 0, Ambient = Color3.fromRGB(150, 150, 160), LightColor = Color3.new(1, 1, 1),
	LightDirection = Vector3.new(-0.6, -0.8, -0.5), Active = true,
}, previewPanel)
round(pvViewport, 3)
stroke(pvViewport)
local pvCam = make("Camera", { FieldOfView = 35 }, pvViewport)
pvViewport.CurrentCamera = pvCam
local pvEmpty = label(pvViewport, {
	Size = UDim2.fromScale(1, 1), Text = "no character", TextSize = 11, TextColor3 = C.SubText,
	TextXAlignment = Enum.TextXAlignment.Center,
})

local pvDrag = makeDraggable(previewPanel, previewPanel)
pvDrag.Attach(pvHeader)

local pvModel, pvCenter, pvDist = nil, Vector3.zero, 10
local pvYaw, pvLastYaw = math.rad(15), nil

local function updatePreviewCam()
	if not pvModel then return end
	pvCam.CFrame = CFrame.lookAt(pvCenter + Vector3.new(math.sin(pvYaw), 0.08, -math.cos(pvYaw)) * pvDist, pvCenter)
	pvLastYaw = pvYaw
end

local function buildPreview()
	if pvModel then pvModel:Destroy() pvModel = nil end
	pvEmpty.Visible = true
	local char = player.Character
	if not (char and char:FindFirstChildOfClass("Humanoid")) then return end
	local wasArchivable = char.Archivable
	char.Archivable = true
	local ok, clone = pcall(function() return char:Clone() end)
	char.Archivable = wasArchivable
	if not ok or not clone then return end
	for _, d in ipairs(clone:GetDescendants()) do
		if d:IsA("BaseScript") or d:IsA("Sound") then
			d:Destroy()
		elseif d:IsA("Humanoid") then
			d.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None
		end
	end
	clone:PivotTo(CFrame.new())
	clone.Parent = pvViewport
	pvModel = clone
	local cf, size = clone:GetBoundingBox()
	pvCenter = cf.Position
	pvDist = (size.Y / 2) / math.tan(math.rad(pvCam.FieldOfView / 2)) * 1.12 + size.Z / 2
	pvEmpty.Visible = false
	updatePreviewCam()
end

do -- drag inside the viewport to rotate
	local rotating, lastX = false, 0
	pvViewport.InputBegan:Connect(function(input)
		if isPointer(input) then
			rotating, lastX = true, input.Position.X
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then rotating = false end
			end)
		end
	end)
	UserInputService.InputChanged:Connect(function(input)
		if rotating and isMove(input) then
			pvYaw -= (input.Position.X - lastX) * 0.012
			lastX = input.Position.X
			updatePreviewCam()
		end
	end)
	RunService.Heartbeat:Connect(function(dt)
		if not previewPanel.Visible or not pvModel then return end
		if settings.PreviewAutoRotate and not rotating then
			pvYaw += math.rad(settings.PreviewSpin) * dt
		end
		if pvYaw ~= pvLastYaw then updatePreviewCam() end
	end)
end

local function applyPreview() previewPanel.Visible = previewEnabled and hudReady end
player.CharacterAppearanceLoaded:Connect(function()
	if previewEnabled then
		task.wait(0.2)
		buildPreview()
	end
end)

-- Tabs & modules ----------------------------------------------------------
local currentTarget = nil

local combatTab = createTab("Combat")
local lockModule = createModule(combatTab, "Lock-On", function(v)
	lockEnabled = v
	lockState.Char, lockState.Part = nil, nil
	currentTarget = nil
end, { Bind = Enum.KeyCode.Q, Col = 1 })
addCycle(lockModule, "Aim part", "AimPart", { "Head", "Torso", "Root", "Legs", "Closest" })
addCycle(lockModule, "Aim mode", "AimMode", { "Camera", "Snap", "Character", "Both" })
addToggle(lockModule, "Wall check", "WallCheck")
addToggle(lockModule, "Ignore teammates", "IgnoreTeammates")
local silentModule = createModule(combatTab, "Silent Aim", function(v)
	silentEnabled = v
	silentState.Char, silentState.Part = nil, nil
end, { Col = 1 })
addCycle(silentModule, "Aim part", "SilentPart", { "Head", "Torso", "Root", "Legs", "Closest" })
addSlider(silentModule, "FOV radius", "SilentFov", 20, 600, 5, "%d px")
addSlider(silentModule, "Hit chance", "SilentHitChance", 1, 100, 1, "%d%%")
addCycle(silentModule, "Priority", "SilentPriority", { "Crosshair", "Distance", "Health" })
addToggle(silentModule, "Wall check", "SilentWallCheck")
addToggle(silentModule, "Fallback to visible part", "SilentFallback")
addToggle(silentModule, "Predict movement", "SilentPrediction")
addSlider(silentModule, "Projectile speed", "SilentProjectileSpeed", 0, 1000, 10, "%d st/s")
addSlider(silentModule, "Bullet drop gravity", "SilentGravity", 0, 400, 5, "%d")
addSlider(silentModule, "Hit roll window", "SilentHitWindow", 0.05, 1, 0.05, "%.2f s")
addToggle(silentModule, "Show FOV circle", "SilentShowFov")
addColor(silentModule, "FOV color", "ColSilent")
addNote(silentModule, "The camera does not move. Your weapon code calls _G.GrkSilentAim.GetDirection(muzzleOrigin, fallbackDirection). Prediction uses the lead / ping / vertical / max values from the Prediction module. Projectile speed 0 = hitscan; gravity 0 = no drop.")
local targeting = createModule(combatTab, "Targeting", function() end, { NoToggle = true, Col = 2 })
addSlider(targeting, "FOV radius", "FovRadius", 40, 600, 5, "%d px")
addSlider(targeting, "Max angle", "MaxAngle", 5, 90, 1, "%d°")
addSlider(targeting, "Max distance", "MaxDistance", 20, 1000, 10, "%d studs")
addSlider(targeting, "Smoothness", "Smoothing", 0.02, 1, 0.01, "%.2f")
addSlider(targeting, "Target grace", "TargetGrace", 0, 1, 0.05, "%.2f s")
local predModule = createModule(combatTab, "Prediction", function(v) settings.Prediction = v end, { Col = 2 })
addSlider(predModule, "Lead time", "PredictionTime", 0, 0.5, 0.01, "%.2f s")
addToggle(predModule, "Add ping", "PredictionPing")
addSlider(predModule, "Projectile speed", "ProjectileSpeed", 0, 1000, 10, "%d st/s")
addSlider(predModule, "Vertical scale", "PredictionVertical", 0, 1, 0.05, "%.2f")
addSlider(predModule, "Max lead", "PredictionMax", 1, 40, 1, "%d studs")
addNote(predModule, "Lock-on only (silent aim is not affected). Aims ahead of moving targets: velocity x (lead + ping + distance / projectile speed). Projectile speed 0 = hitscan.")

local noShakeModule = createModule(combatTab, "No Screenshake", function(v) weaponFlags.NoShake = v end, { Col = 1 })
addToggle(noShakeModule, "Remove camera roll", "NoShakeRoll")
addSlider(noShakeModule, "Shake scale", "ShakeScale", 0, 1, 0.05, "%.2f")
addNote(noShakeModule, "Zeroes Humanoid.CameraOffset every frame, strips camera roll, and exposes _G.GrkWeapon.ShakeScale() for your own shake code (returns 0 when on).")
local noRecoilModule = createModule(combatTab, "No Recoil", function(v) weaponFlags.NoRecoil = v end, { Col = 2 })
addSlider(noRecoilModule, "Recoil scale", "RecoilScale", 0, 1, 0.05, "%.2f")
addNote(noRecoilModule, "Your weapon code calls _G.GrkWeapon.Recoil(Vector2.new(pitch, yaw)) before applying a kick; it returns the scaled value (zero when on).")
local perfectModule = createModule(combatTab, "Perfect Accuracy", function(v) weaponFlags.NoSpread = v end, { Col = 2 })
addToggle(perfectModule, "Always roll a hit", "PerfectIgnoreChance")
addNote(perfectModule, "Your weapon code calls _G.GrkWeapon.Spread(angleRadians) or _G.GrkWeapon.ApplySpread(direction, angleRadians). With this on, spread is 0 and the direction is returned unchanged.")

do
local visualsTab = createTab("Visuals")
local function espModule(name, flag, col)
	return createModule(visualsTab, name, function(v) esp[flag] = v end, { Col = col })
end
local chams = espModule("Chams", "Chams", 1)
addSlider(chams, "Fill transparency", "ChamsFill", 0, 1, 0.05, "%.2f")
addSlider(chams, "Outline transparency", "ChamsOutline", 0, 1, 0.05, "%.2f")
addColor(chams, "Fill color", "ColChamsFill")
addColor(chams, "Outline color", "ColChamsOutline")
local fillBox = espModule("Filled Box", "Fill", 1)
addSlider(fillBox, "Fill opacity", "FillOpacity", 0.05, 1, 0.05, "%.2f")
addColor(fillBox, "Color", "ColFill")
local health = espModule("Health Bar", "Health", 1)
addColor(health, "Full health", "ColHealthFull")
addColor(health, "Low health", "ColHealthLow")
local espSettings = createModule(visualsTab, "ESP Settings", function() end, { NoToggle = true, Col = 1 })
addSlider(espSettings, "Max distance", "EspDistance", 50, 2000, 50, "%d studs")
addToggle(espSettings, "Team colors", "EspTeamColor")
addToggle(espSettings, "Hide teammates", "EspHideTeam")

local box = espModule("Box", "Box", 2)
addCycle(box, "Style", "BoxStyle", { "Full", "Corners" })
addSlider(box, "Thickness", "BoxThickness", 1, 3, 1, "%d px")
addColor(box, "Color", "ColBox")
addColor(box, "Shadow color", "ColBoxShadow")
local names = espModule("Names", "Name", 2)
addToggle(names, "Show distance", "ShowDistance")
addColor(names, "Text color", "ColName")
local item = espModule("Held Item", "Item", 2)
addColor(item, "Text color", "ColItem")
local tracers = espModule("Tracers", "Tracers", 2)
addCycle(tracers, "Origin", "TracerOrigin", { "Bottom", "Center", "Top" })
addCycle(tracers, "Target", "TracerTarget", { "Feet", "Torso", "Head" })
addSlider(tracers, "Thickness", "TracerThickness", 1, 5, 0.5, "%.1f px")
addColor(tracers, "Color", "ColTracer")

local renderTab = createTab("Render")
local circleModule = createModule(renderTab, "FOV Circle", function(v) circleEnabled = v end, { Col = 1 })
addSlider(circleModule, "Thickness", "CircleThickness", 1, 6, 1, "%d px")
addSlider(circleModule, "Opacity", "CircleOpacity", 0.1, 1, 0.05, "%.2f")
addColor(circleModule, "Color", "ColCircle")
addColor(circleModule, "Locked color", "ColCircleLock")
circleModule:SetEnabled(true, true)
local snapModule = createModule(renderTab, "Snapline", function(v) snapEnabled = v end, { Col = 2 })
addCycle(snapModule, "Origin", "SnapOrigin", { "Center", "Bottom", "Top", "Mouse" })
addSlider(snapModule, "Thickness", "SnapThickness", 1, 5, 0.5, "%.1f px")
addToggle(snapModule, "Show without lock", "SnapAlways")
addColor(snapModule, "Color", "ColSnap")
local freecamModule = createModule(renderTab, "Freecam", setFreecam, { Col = 1, Bind = Enum.KeyCode.P })
addSlider(freecamModule, "Speed", "FreecamSpeed", 5, 300, 5, "%d studs/s")
addSlider(freecamModule, "Shift boost", "FreecamBoost", 1, 8, 0.5, "%.1fx")
addSlider(freecamModule, "Sensitivity", "FreecamSens", 0.2, 3, 0.05, "%.2f")
addSlider(freecamModule, "Smoothing", "FreecamSmooth", 2, 30, 1, "%d")
addToggle(freecamModule, "Freeze character", "FreecamFreeze")
addToggle(freecamModule, "Hold RMB to look", "FreecamHoldRmb")
addNote(freecamModule, "WASD move, Space / Ctrl up / down, Shift boosts. Lock-on pauses while freecam is on.")
local zoomModule = createModule(renderTab, "Zoom", function(v)
	zoomState.on = v
	if v then zoomState.base = workspace.CurrentCamera.FieldOfView end
end, { Col = 1, Bind = Enum.KeyCode.C })
addSlider(zoomModule, "Zoom FOV", "ZoomFov", 5, 70, 1, "%d°")
addSlider(zoomModule, "Smoothing", "ZoomSmooth", 2, 30, 1, "%d")
addToggle(zoomModule, "Scroll wheel adjusts", "ZoomScroll")
addNote(zoomModule, "Press the keybind (default C) to zoom in / out. While zoomed, the scroll wheel changes the FOV. Lower FOV = more zoom.")
local previewModule = createModule(renderTab, "Character Preview", function(v)
	previewEnabled = v
	applyPreview()
	if v and not pvModel then buildPreview() end
end, { Col = 2 })
addToggle(previewModule, "Auto rotate", "PreviewAutoRotate")
addSlider(previewModule, "Spin speed", "PreviewSpin", 10, 180, 5, "%d°/s")
addSlider(previewModule, "Size", "PreviewScale", 0.6, 1.8, 0.05, "%.2f", function(v) pvScale.Scale = v end)
addButton(previewModule, "Refresh model", buildPreview)
addNote(previewModule, "Drag the header to move the panel, drag the model to rotate it.")
previewModule:SetEnabled(true, true)

local settingsTab = createTab("Settings")
local theme = createModule(settingsTab, "Theme", function() end, { NoToggle = true, Col = 1 })
addColor(theme, "Accent color", "Accent", setAccent)
addSlider(theme, "UI scale", "UiScale", 0.5, 1.5, 0.05, "%.2f", function(v) uiScale.Scale = v end)
addToggle(theme, "Show arraylist", "ShowArray", function()
	for _, m in ipairs(modules) do refreshArray(m) end
end)
addToggle(theme, "UI sounds", "UiSounds")
addSlider(theme, "Sound volume", "UiVolume", 0, 1, 0.05, "%.2f")
addButton(theme, "Reset HUD positions", function()
	for _, st in ipairs(hudStates) do st.Reset() end
end)
local menu = createModule(settingsTab, "Menu", function() end, {
	NoToggle = true, OnPress = toggleUI, Bind = Enum.KeyCode.RightShift, Col = 2,
})
addNote(menu, "The keybind on this header hides / shows the menu.")
addNote(menu, "Right-click a dropdown to cycle backwards. Click a color swatch to open its picker. Drag the title bar or sidebar to move the menu. The watermark, arraylist and ≡ button can be dragged too.")

end

activeTab = tabs[1]
paintTabs()

-- intro: loading screen first, then the window pops in
finishLoader(function()
	watermark.Visible = true
	hudReady = true
	applyPreview()
	tween(window, TWEEN_SLOW, { GroupTransparency = 0 })
	tween(uiScale, TWEEN_OPEN, { Scale = settings.UiScale })
end)

local getBestTarget
do
-- Lock-on targeting --------------------------------------------------------
local rayParams = RaycastParams.new()
rayParams.FilterType = Enum.RaycastFilterType.Exclude
rayParams.IgnoreWater = true
rayParams.RespectCanCollide = true

-- A locked target is kept until it leaves FOV * STICKY (or stays hidden longer than TargetGrace).
-- This is what stops the lock from flicking between players / dropping for a single frame.
local STICKY = 1.35

-- true if nothing solid is between the camera and the part (or wall check is off)
local function isPartVisible(part, character, wallCheck)
	if not wallCheck then return true end
	rayParams.FilterDescendantsInstances = { player.Character }
	local origin = camera.CFrame.Position
	local result = workspace:Raycast(origin, part.Position - origin, rayParams)
	return result == nil or result.Instance:IsDescendantOf(character)
end

-- Velocity tracking (smoothed from position deltas, so it also works on anchored / odd rigs) ----
local trackers = setmetatable({}, { __mode = "k" })
local function updateTrackers(dt)
	for _, other in ipairs(Players:GetPlayers()) do
		local ch = other.Character
		local rp = ch and ch:FindFirstChild("HumanoidRootPart")
		if rp then
			local t = trackers[ch]
			if not t then
				trackers[ch] = { pos = rp.Position, vel = Vector3.zero }
			else
				if dt > 0 then
					local raw = (rp.Position - t.pos) / dt
					if raw.Magnitude > 300 then raw = Vector3.zero end -- ignore teleports
					t.vel = t.vel:Lerp(raw, 1 - math.exp(-dt * 14))
				end
				t.pos = rp.Position
			end
		end
	end
end

-- Where the part will be: velocity * (lead time + ping + projectile travel time)
local function predictedPosition(part, origin, apply)
	local pos = part.Position
	if not (settings.Prediction and apply) then return pos end
	local t = trackers[part.Parent]
	if not t then return pos end
	local time = settings.PredictionTime
	if settings.PredictionPing then time += player:GetNetworkPing() end
	if settings.ProjectileSpeed > 0 then time += (pos - origin).Magnitude / settings.ProjectileSpeed end
	local v = t.vel
	local offset = Vector3.new(v.X, v.Y * settings.PredictionVertical, v.Z) * time
	if offset.Magnitude > settings.PredictionMax then offset = offset.Unit * settings.PredictionMax end
	return pos + offset
end

local function scorePart(part, fov, maxAngle)
	local camCF = camera.CFrame
	local offset = part.Position - camCF.Position
	local mag = offset.Magnitude
	if mag > settings.MaxDistance or mag < 0.01 then return nil end
	local pos, onScreen = camera:WorldToViewportPoint(part.Position)
	if not onScreen or pos.Z <= 0 then return nil end
	local pixelDist = (Vector2.new(pos.X, pos.Y) - camera.ViewportSize / 2).Magnitude
	if pixelDist > fov then return nil end
	if maxAngle then
		local angle = math.deg(math.acos(math.clamp(camCF.LookVector:Dot(offset / mag), -1, 1)))
		if angle > maxAngle then return nil end
	end
	return pixelDist
end

local function isAlive(character)
	local hum = character and character:FindFirstChildOfClass("Humanoid")
	return hum ~= nil and hum.Health > 0
end

-- R15 and R6 names for each aim part
local PART_NAMES = {
	Head = { "Head" },
	Torso = { "UpperTorso", "Torso" },
	Root = { "HumanoidRootPart" },
	Legs = { "LeftUpperLeg", "Left Leg" },
}
local CLOSEST_NAMES = {
	"Head", "UpperTorso", "LowerTorso", "Torso", "HumanoidRootPart",
	"LeftUpperArm", "RightUpperArm", "Left Arm", "Right Arm",
	"LeftUpperLeg", "RightUpperLeg", "Left Leg", "Right Leg",
}

-- Returns the part to aim at on this character and its screen score, or nil
local function pickPart(character, partName, fov, maxAngle, wallCheck)
	if partName ~= "Closest" then
		for _, n in ipairs(PART_NAMES[partName] or PART_NAMES.Head) do
			local part = character:FindFirstChild(n)
			if part and part:IsA("BasePart") then
				local score = scorePart(part, fov, maxAngle)
				if score and isPartVisible(part, character, wallCheck) then return part, score end
				return nil
			end
		end
		return nil
	end
	local cands = {}
	for _, n in ipairs(CLOSEST_NAMES) do
		local part = character:FindFirstChild(n)
		if part and part:IsA("BasePart") then
			local score = scorePart(part, fov, maxAngle)
			if score then table.insert(cands, { part, score }) end
		end
	end
	table.sort(cands, function(x, y) return x[2] < y[2] end)
	for _, c in ipairs(cands) do
		if isPartVisible(c[1], character, wallCheck) then return c[1], c[2] end
	end
	return nil
end

function getBestTarget(partName, fov, maxAngle, wallCheck, priority)
	local best, bestScore = nil, math.huge
	for _, other in ipairs(Players:GetPlayers()) do
		local skip = other == player
			or (settings.IgnoreTeammates and other.Team ~= nil and other.Team == player.Team)
		if not skip then
			local character = other.Character
			if character and isAlive(character) then
				local part, score = pickPart(character, partName, fov, maxAngle, wallCheck)
				if part then
					if priority == "Distance" then
						score = (part.Position - camera.CFrame.Position).Magnitude
					elseif priority == "Health" then
						score = character:FindFirstChildOfClass("Humanoid").Health + score * 0.001
					end
					if score < bestScore then best, bestScore = part, score end
				end
			end
		end
	end
	return best
end

-- Sticky target resolver. st = { Char, Part, LostAt }
local function resolveTarget(st, partName, fov, maxAngle, wallCheck, priority)
	local now = os.clock()
	local ch = st.Char
	if ch and ch.Parent and isAlive(ch) then
		local part = pickPart(ch, partName, fov * STICKY, maxAngle and maxAngle * STICKY, wallCheck)
		if part then
			st.Part, st.LostAt = part, now
			return part
		end
		-- briefly hidden / out of range: keep the old part instead of dropping the lock
		if st.Part and st.Part.Parent and now - st.LostAt < settings.TargetGrace then return st.Part end
	end
	st.Char, st.Part = nil, nil
	local part = getBestTarget(partName, fov, maxAngle, wallCheck, priority)
	if part then st.Char, st.Part, st.LostAt = part.Parent, part, now end
	return part
end

-- Silent aim ------------------------------------------------------------------
-- The camera never moves. Your own weapon / raycast code asks for the aim point:
--   local dir = _G.GrkSilentAim.GetDirection(origin, mouseDirection)   -- Vector3 (unit)
--   local pos = _G.GrkSilentAim.GetAimPosition(origin)                 -- Vector3 or nil
--   local part = _G.GrkSilentAim.GetTarget()                           -- BasePart or nil
local silentCircle = make("Frame", {
	Name = "SilentFovCircle", AnchorPoint = Vector2.new(0.5, 0.5), Position = UDim2.fromScale(0.5, 0.5),
	BackgroundTransparency = 1, Visible = false,
}, gui)
round(silentCircle, 9999)
local silentStroke = make("UIStroke", { Thickness = 1.5, Transparency = 0.35 }, silentCircle)

local SilentAPI = {}
function SilentAPI.IsEnabled() return silentEnabled end
function SilentAPI.GetTarget()
	local p = silentState.Part
	if silentEnabled and p and p.Parent then return p end
	return nil
end
-- true if nothing solid is between origin (e.g. the muzzle) and the part
local function visibleFrom(part, origin)
	rayParams.FilterDescendantsInstances = { player.Character }
	local result = workspace:Raycast(origin, part.Position - origin, rayParams)
	return result == nil or result.Instance:IsDescendantOf(part.Parent)
end

-- Picks the part to shoot at from the weapon's origin: the selected part if it is visible,
-- otherwise (optional) the closest-to-crosshair visible part of the same character.
local function partForShot(target, origin)
	if not settings.SilentWallCheck or visibleFrom(target, origin) then return target end
	if not settings.SilentFallback then return nil end
	local ch, best, bestScore = target.Parent, nil, math.huge
	for _, n in ipairs(CLOSEST_NAMES) do
		local part = ch:FindFirstChild(n)
		if part and part:IsA("BasePart") and part ~= target and visibleFrom(part, origin) then
			local sp = camera:WorldToViewportPoint(part.Position)
			local d = (Vector2.new(sp.X, sp.Y) - camera.ViewportSize / 2).Magnitude
			if d < bestScore then best, bestScore = part, d end
		end
	end
	return best
end

-- Where to aim: velocity lead (+ ping + projectile travel time) and optional bullet drop compensation
local function solveAim(part, origin)
	local pos = part.Position
	local speed = settings.SilentProjectileSpeed
	local g = settings.SilentGravity
	local tr = settings.SilentPrediction and trackers[part.Parent] or nil
	local vel = tr and tr.vel or Vector3.zero
	local lead = 0
	if tr then
		lead = settings.PredictionTime
		if settings.PredictionPing then lead += player:GetNetworkPing() end
	end
	local aim = pos
	for _ = 1, 3 do
		local flight = speed > 0 and (aim - origin).Magnitude / speed or 0
		local offset = Vector3.new(vel.X, vel.Y * settings.PredictionVertical, vel.Z) * (lead + flight)
		if offset.Magnitude > settings.PredictionMax then offset = offset.Unit * settings.PredictionMax end
		aim = pos + offset
		if g > 0 and speed > 0 then
			aim += Vector3.new(0, 0.5 * g * flight * flight, 0)
		end
	end
	return aim
end

-- The hit chance is rolled once per time window, not once per call, so a multi-pellet or
-- multi-call shot is either fully assisted or not (no random half-aimed bursts).
local hitRoll = { at = -math.huge, ok = true }
local function rollHit()
	if weaponFlags.NoSpread and settings.PerfectIgnoreChance then return true end
	if settings.SilentHitChance >= 100 then return true end
	local now = os.clock()
	if now - hitRoll.at > settings.SilentHitWindow then
		hitRoll.at = now
		hitRoll.ok = math.random() * 100 <= settings.SilentHitChance
	end
	return hitRoll.ok
end

function SilentAPI.GetAimPosition(origin)
	local p = SilentAPI.GetTarget()
	if not p then return nil end
	origin = origin or camera.CFrame.Position
	if not rollHit() then return nil end
	local part = partForShot(p, origin)
	if not part then return nil end
	return solveAim(part, origin)
end
function SilentAPI.GetDirection(origin, fallback)
	origin = origin or camera.CFrame.Position
	local pos = SilentAPI.GetAimPosition(origin)
	if pos and (pos - origin).Magnitude > 0.01 then return (pos - origin).Unit end
	return fallback
end
_G.GrkSilentAim = SilentAPI

-- Weapon helpers (No Screenshake / No Recoil / Perfect Accuracy) ----------------
local WeaponAPI = {}
function WeaponAPI.ShakeScale()
	return weaponFlags.NoShake and settings.ShakeScale or 1
end
function WeaponAPI.Recoil(kick)
	if weaponFlags.NoRecoil then return kick * settings.RecoilScale end
	return kick
end
function WeaponAPI.Spread(angle)
	if weaponFlags.NoSpread then return 0 end
	return angle
end
function WeaponAPI.ApplySpread(direction, angle)
	if weaponFlags.NoSpread or not angle or angle <= 0 then return direction end
	local rng = Random.new()
	local cf = CFrame.lookAt(Vector3.zero, direction)
	local a, r = rng:NextNumber(0, math.pi * 2), math.sqrt(rng:NextNumber()) * angle
	return (cf * CFrame.Angles(0, 0, a) * CFrame.Angles(r, 0, 0)).LookVector
end
_G.GrkWeapon = WeaponAPI

RunService:BindToRenderStep("GrkNoShake", Enum.RenderPriority.Camera.Value + 4, function()
	if not weaponFlags.NoShake or freecamActive then return end
	local hum = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
	if hum and settings.ShakeScale < 1 then
		hum.CameraOffset = hum.CameraOffset * settings.ShakeScale
	end
	if settings.NoShakeRoll then
		local cam = workspace.CurrentCamera
		local cf = cam.CFrame
		local look, pos = cf.LookVector, cf.Position
		if math.abs(look.Y) < 0.999 then
			cam.CFrame = CFrame.lookAt(pos, pos + look)
		end
	end
end)

-- Zoom ------------------------------------------------------------------------
UserInputService.InputChanged:Connect(function(input, processed)
	if processed or not zoomState.on or not settings.ZoomScroll then return end
	if input.UserInputType == Enum.UserInputType.MouseWheel then
		settings.ZoomFov = math.clamp(settings.ZoomFov - input.Position.Z * 3, 5, 70)
	end
end)
RunService:BindToRenderStep("GrkZoom", Enum.RenderPriority.Camera.Value + 5, function(dt)
	local cam = workspace.CurrentCamera
	if freecamActive then return end
	if zoomState.on then
		zoomState.base = zoomState.base or cam.FieldOfView
		local a = 1 - math.exp(-dt * settings.ZoomSmooth)
		cam.FieldOfView += (settings.ZoomFov - cam.FieldOfView) * a
	elseif zoomState.base then
		local a = 1 - math.exp(-dt * settings.ZoomSmooth)
		cam.FieldOfView += (zoomState.base - cam.FieldOfView) * a
		if math.abs(cam.FieldOfView - zoomState.base) < 0.05 then
			cam.FieldOfView = zoomState.base
			zoomState.base = nil
		end
	end
end)

-- Character turning (restores Humanoid.AutoRotate when the lock ends)
local autoRotateHum = nil
local function restoreAutoRotate()
	if autoRotateHum then
		if autoRotateHum.Parent then autoRotateHum.AutoRotate = true end
		autoRotateHum = nil
	end
end

RunService:BindToRenderStep("LockOn", Enum.RenderPriority.Camera.Value + 1, function(dt)
	camera = workspace.CurrentCamera
	updateTrackers(dt)

	-- lock-on FOV circle
	local diameter = snap(settings.FovRadius) * 2
	circle.Visible = circleEnabled
	circle.Size = UDim2.fromOffset(diameter, diameter)
	circleStroke.Thickness = settings.CircleThickness
	circleStroke.Color = (lockEnabled and currentTarget) and colorOf("ColCircleLock") or colorOf("ColCircle")
	circleStroke.Transparency = lockEnabled and (1 - settings.CircleOpacity)
		or math.min(1, (1 - settings.CircleOpacity) + 0.3)

	-- silent aim target + circle (independent from lock-on)
	local sd = snap(settings.SilentFov) * 2
	silentCircle.Visible = silentEnabled and settings.SilentShowFov
	silentCircle.Size = UDim2.fromOffset(sd, sd)
	silentStroke.Color = colorOf("ColSilent")
	if silentEnabled and not freecamActive then
		resolveTarget(silentState, settings.SilentPart, settings.SilentFov, nil, settings.SilentWallCheck, settings.SilentPriority)
	else
		silentState.Char, silentState.Part = nil, nil
	end

	if not lockEnabled or freecamActive then
		currentTarget = nil
		lockState.Char, lockState.Part = nil, nil
		restoreAutoRotate()
		return
	end

	local part = resolveTarget(lockState, settings.AimPart, settings.FovRadius, settings.MaxAngle, settings.WallCheck)
	currentTarget = part
	if not part then
		restoreAutoRotate()
		return
	end

	local mode = settings.AimMode
	local origin = camera.CFrame.Position
	local pos = predictedPosition(part, origin, settings.PredLock)
	-- frame-rate independent smoothing
	local alpha = math.clamp(1 - (1 - settings.Smoothing) ^ (dt * 60), 0, 1)
	if (pos - origin).Magnitude > 0.01 then
		if mode == "Camera" or mode == "Both" then
			camera.CFrame = camera.CFrame:Lerp(CFrame.lookAt(origin, pos), alpha)
		elseif mode == "Snap" then
			camera.CFrame = CFrame.lookAt(origin, pos)
		end
	end
	if mode == "Character" or mode == "Both" then
		local char = player.Character
		local myRoot = char and char:FindFirstChild("HumanoidRootPart")
		local hum = char and char:FindFirstChildOfClass("Humanoid")
		if myRoot then
			if hum and autoRotateHum ~= hum then
				restoreAutoRotate()
				hum.AutoRotate = false
				autoRotateHum = hum
			end
			local flat = Vector3.new(pos.X - myRoot.Position.X, 0, pos.Z - myRoot.Position.Z)
			if flat.Magnitude > 0.5 then
				myRoot.CFrame = myRoot.CFrame:Lerp(CFrame.lookAt(myRoot.Position, myRoot.Position + flat), alpha)
			end
		end
	else
		restoreAutoRotate()
	end
end)

end

-- ESP ------------------------------------------------------------------------
-- Shared anchor for tracer beams (Beams are anti-aliased, unlike rotated GUI frames)
local tracerAnchor = make("Part", {
	Name = "TracerAnchor", Anchored = true, CanCollide = false, CanQuery = false, CanTouch = false,
	Transparency = 1, Size = Vector3.new(0.1, 0.1, 0.1), CFrame = CFrame.new(0, 0, 0), Locked = true,
}, workspace)

local espObjects = {}

-- Snapline beam (line from screen origin to the aim target)
local snapA0 = make("Attachment", {}, tracerAnchor)
local snapA1 = make("Attachment", {}, tracerAnchor)
local snapBeam = make("Beam", {
	Attachment0 = snapA0, Attachment1 = snapA1, FaceCamera = true, LightEmission = 0, LightInfluence = 0,
	Segments = 1, Enabled = false, Transparency = NumberSequence.new(0.05),
}, tracerAnchor)
local previewTarget, previewTimer = nil, 0
local BLACK = Color3.new(0, 0, 0)
local WHITE = Color3.new(1, 1, 1)

local function createEsp(plr)
	if plr == player or espObjects[plr] then return end
	local o = { Corners = {}, CornerStrokes = {} }

	-- full box: dark shadow frame behind a colored frame
	o.Shadow = make("Frame", { BackgroundTransparency = 1, BorderSizePixel = 0, Visible = false }, espLayer)
	round(o.Shadow, 2)
	o.ShadowStroke = make("UIStroke", {
		Color = BLACK, Thickness = 3, Transparency = 0.45, ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
	}, o.Shadow)
	o.Box = make("Frame", {
		BackgroundColor3 = WHITE, BackgroundTransparency = 1, BorderSizePixel = 0, Visible = false, ZIndex = 2,
	}, espLayer)
	round(o.Box, 2)
	o.Stroke = make("UIStroke", { Thickness = 1, ApplyStrokeMode = Enum.ApplyStrokeMode.Border }, o.Box)
	o.Grad = make("UIGradient", { Rotation = 90 }, o.Box)

	-- corner style: 8 crisp segments
	for i = 1, 8 do
		local seg = make("Frame", { BackgroundColor3 = WHITE, BorderSizePixel = 0, Visible = false, ZIndex = 3 }, espLayer)
		o.CornerStrokes[i] = make("UIStroke", { Color = BLACK, Thickness = 1, Transparency = 0.4, ApplyStrokeMode = Enum.ApplyStrokeMode.Border }, seg)
		o.Corners[i] = seg
	end

	-- health bar
	o.HealthBack = make("Frame", {
		BackgroundColor3 = BLACK, BackgroundTransparency = 0.35, BorderSizePixel = 0, Visible = false, ZIndex = 2,
	}, espLayer)
	round(o.HealthBack, 2)
	make("UIPadding", {
		PaddingTop = UDim.new(0, 1), PaddingBottom = UDim.new(0, 1),
		PaddingLeft = UDim.new(0, 1), PaddingRight = UDim.new(0, 1),
	}, o.HealthBack)
	o.HealthFill = make("Frame", {
		AnchorPoint = Vector2.new(0, 1), Position = UDim2.fromScale(0, 1), Size = UDim2.fromScale(1, 1), BorderSizePixel = 0,
	}, o.HealthBack)
	round(o.HealthFill, 1)

	local function label()
		return make("TextLabel", {
			Size = UDim2.fromOffset(180, 14), BackgroundTransparency = 1, Font = Enum.Font.GothamMedium,
			TextSize = 13, TextColor3 = WHITE, TextStrokeColor3 = BLACK, TextStrokeTransparency = 0.35,
			Visible = false, ZIndex = 3,
		}, espLayer)
	end
	o.Item = label()
	o.Name = label()

	-- tracer beam
	o.A0 = make("Attachment", {}, tracerAnchor)
	o.A1 = make("Attachment", {}, tracerAnchor)
	o.Beam = make("Beam", {
		Attachment0 = o.A0, Attachment1 = o.A1, FaceCamera = true, LightEmission = 0, LightInfluence = 0,
		Segments = 1, Enabled = false, Transparency = NumberSequence.new(0.05),
	}, tracerAnchor)

	espObjects[plr] = o
end

local function destroyEsp(plr)
	local o = espObjects[plr]
	if not o then return end
	if o.Highlight then o.Highlight:Destroy() end
	o.Shadow:Destroy() o.Box:Destroy() o.HealthBack:Destroy() o.Item:Destroy() o.Name:Destroy()
	for _, seg in ipairs(o.Corners) do seg:Destroy() end
	o.Beam:Destroy() o.A0:Destroy() o.A1:Destroy()
	espObjects[plr] = nil
end

for _, p in ipairs(Players:GetPlayers()) do createEsp(p) end
Players.PlayerAdded:Connect(createEsp)
Players.PlayerRemoving:Connect(destroyEsp)

local function hide2D(o)
	o.Shadow.Visible = false
	o.Box.Visible = false
	o.HealthBack.Visible = false
	o.Item.Visible = false
	o.Name.Visible = false
	o.Beam.Enabled = false
	for _, seg in ipairs(o.Corners) do seg.Visible = false end
end
local function hideAll(o)
	hide2D(o)
	if o.Highlight then o.Highlight.Enabled = false end
end

local function espColor(plr, key)
	if settings.EspTeamColor and plr.Team then return plr.TeamColor.Color end
	return colorOf(key)
end

local function placeSeg(seg, x, y, w, h, color)
	seg.Position = UDim2.fromOffset(x, y)
	seg.Size = UDim2.fromOffset(w, h)
	seg.BackgroundColor3 = color
	seg.Visible = true
end

RunService:BindToRenderStep("EspUpdate", Enum.RenderPriority.Camera.Value + 2, function(dt)
	camera = workspace.CurrentCamera
	local camPos = camera.CFrame.Position
	local viewport = camera.ViewportSize
	local focal = (viewport.Y / 2) / math.tan(math.rad(camera.FieldOfView) / 2)

	for plr, o in pairs(espObjects) do
		local character = plr.Character
		local rootPart = character and character:FindFirstChild("HumanoidRootPart")
		local hum = character and character:FindFirstChildOfClass("Humanoid")

		local show = rootPart ~= nil and hum ~= nil and hum.Health > 0
		if show and settings.EspHideTeam and plr.Team ~= nil and plr.Team == player.Team then show = false end
		local dist = show and (rootPart.Position - camPos).Magnitude or 0
		if show and dist > settings.EspDistance then show = false end

		if not show then
			hideAll(o)
			o.HealthShown = nil
		else
			local boxColor = espColor(plr, "ColBox")
			local fillColor = espColor(plr, "ColFill")
			local tracerColor = espColor(plr, "ColTracer")
			local shadowColor = colorOf("ColBoxShadow")

			-- Chams
			if esp.Chams then
				if not o.Highlight or o.Highlight.Parent ~= character then
					if o.Highlight then o.Highlight:Destroy() end
					o.Highlight = make("Highlight", { DepthMode = Enum.HighlightDepthMode.AlwaysOnTop }, character)
				end
				o.Highlight.Enabled = true
				o.Highlight.FillColor = espColor(plr, "ColChamsFill")
				o.Highlight.OutlineColor = espColor(plr, "ColChamsOutline")
				o.Highlight.FillTransparency = settings.ChamsFill
				o.Highlight.OutlineTransparency = settings.ChamsOutline
			elseif o.Highlight then
				o.Highlight.Enabled = false
			end

			local rootPos = rootPart.Position
			local top = camera:WorldToViewportPoint(rootPos + Vector3.new(0, 2.8, 0))
			local bot = camera:WorldToViewportPoint(rootPos - Vector3.new(0, 3, 0))
			if top.Z <= 0 or bot.Z <= 0 then
				hide2D(o)
			else
				-- pixel-snapped rectangle (this is what removes the blurry / jagged look)
				local h = math.max(snap(math.abs(bot.Y - top.Y)), 8)
				local w = math.max(snap(h * 0.55), 4)
				local cx = snap((top.X + bot.X) / 2)
				local x = cx - snap(w / 2)
				local y = snap(math.min(top.Y, bot.Y))
				local T = settings.BoxThickness

				local cornerStyle = esp.Box and settings.BoxStyle == "Corners"
				local fullOutline = esp.Box and not cornerStyle

				-- Full box + fill
				local showFrame = esp.Fill or fullOutline
				o.Box.Visible = showFrame
				o.Shadow.Visible = fullOutline
				if showFrame then
					o.Box.Position = UDim2.fromOffset(x, y)
					o.Box.Size = UDim2.fromOffset(w, h)
					o.Box.BackgroundColor3 = fillColor
					o.Box.BackgroundTransparency = esp.Fill and 0 or 1
					o.Stroke.Color = boxColor
					o.Stroke.Thickness = T
					o.Stroke.Transparency = fullOutline and 0 or 1
					if esp.Fill then
						local op = settings.FillOpacity
						if o.LastOpacity ~= op then
							o.LastOpacity = op
							o.Grad.Transparency = NumberSequence.new({
								NumberSequenceKeypoint.new(0, 1 - op * 0.4),
								NumberSequenceKeypoint.new(1, 1 - op),
							})
						end
					end
				end
				if fullOutline then
					o.Shadow.Position = UDim2.fromOffset(x, y)
					o.Shadow.Size = UDim2.fromOffset(w, h)
					o.ShadowStroke.Thickness = T + 2
					o.ShadowStroke.Color = shadowColor
				end

				-- Corner box
				if cornerStyle then
					local len = math.clamp(snap(math.min(w, h) * 0.28), 5, 40)
					local c = o.Corners
					for _, st in ipairs(o.CornerStrokes) do st.Color = shadowColor end
					placeSeg(c[1], x, y, len, T, boxColor)                       -- TL horizontal
					placeSeg(c[2], x, y, T, len, boxColor)                       -- TL vertical
					placeSeg(c[3], x + w - len, y, len, T, boxColor)             -- TR horizontal
					placeSeg(c[4], x + w - T, y, T, len, boxColor)               -- TR vertical
					placeSeg(c[5], x, y + h - T, len, T, boxColor)               -- BL horizontal
					placeSeg(c[6], x, y + h - len, T, len, boxColor)             -- BL vertical
					placeSeg(c[7], x + w - len, y + h - T, len, T, boxColor)     -- BR horizontal
					placeSeg(c[8], x + w - T, y + h - len, T, len, boxColor)     -- BR vertical
				else
					for _, seg in ipairs(o.Corners) do seg.Visible = false end
				end

				-- Health bar (animated)
				o.HealthBack.Visible = esp.Health
				if esp.Health then
					local pct = math.clamp(hum.Health / math.max(hum.MaxHealth, 1), 0, 1)
					o.HealthShown = (o.HealthShown or pct)
					o.HealthShown += (pct - o.HealthShown) * math.min(1, dt * 12)
					local shown = o.HealthShown
					o.HealthBack.Position = UDim2.fromOffset(x - 6 - T, y - 1)
					o.HealthBack.Size = UDim2.fromOffset(4, h + 2)
					o.HealthFill.Size = UDim2.fromScale(1, shown)
					o.HealthFill.BackgroundColor3 = colorOf("ColHealthLow"):Lerp(colorOf("ColHealthFull"), shown)
				end

				-- Held item
				local tool = esp.Item and character:FindFirstChildOfClass("Tool")
				o.Item.Visible = tool ~= nil
				if tool then
					o.Item.Text = tool.Name
					o.Item.TextColor3 = colorOf("ColItem")
					o.Item.Position = UDim2.fromOffset(cx - 90, y + h + 3)
				end

				-- Name (+ distance)
				o.Name.Visible = esp.Name
				if esp.Name then
					o.Name.Text = settings.ShowDistance
						and string.format("%s  [%d]", plr.DisplayName, snap(dist))
						or plr.DisplayName
					o.Name.Position = UDim2.fromOffset(cx - 90, y - 17)
					o.Name.TextColor3 = colorOf("ColName")
				end

				-- Tracer (3D beam, constant pixel thickness)
				o.Beam.Enabled = esp.Tracers
				if esp.Tracers then
					local oy = settings.TracerOrigin == "Top" and 0
						or (settings.TracerOrigin == "Center" and viewport.Y / 2 or viewport.Y)
					local ray = camera:ViewportPointToRay(viewport.X / 2, oy)
					local startPos = ray.Origin + ray.Direction * 2
					local targetPos
					if settings.TracerTarget == "Head" then
						targetPos = rootPos + Vector3.new(0, 2.8, 0)
					elseif settings.TracerTarget == "Torso" then
						targetPos = rootPos
					else
						targetPos = rootPos - Vector3.new(0, 3, 0)
					end
					o.A0.WorldPosition = startPos
					o.A1.WorldPosition = targetPos
					local p = settings.TracerThickness
					o.Beam.Width0 = 2 * p / focal
					o.Beam.Width1 = math.max((targetPos - camPos).Magnitude, 2) * p / focal
					o.Beam.Color = ColorSequence.new(tracerColor)
				end
			end
		end
	end
end)

RunService:BindToRenderStep("SnaplineUpdate", Enum.RenderPriority.Camera.Value + 3, function(dt)
	if not snapEnabled then
		snapBeam.Enabled = false
		return
	end
	camera = workspace.CurrentCamera

	local target = nil
	if lockEnabled and currentTarget and currentTarget.Parent then
		target = currentTarget
	elseif silentEnabled and silentState.Part and silentState.Part.Parent then
		target = silentState.Part
	elseif settings.SnapAlways then
		previewTimer += dt
		if previewTimer > 0.1 then
			previewTimer = 0
			previewTarget = getBestTarget(settings.AimPart, settings.FovRadius, settings.MaxAngle, settings.WallCheck)
		end
		if previewTarget and previewTarget.Parent and (previewTarget.Parent:FindFirstChildOfClass("Humanoid") and previewTarget.Parent:FindFirstChildOfClass("Humanoid").Health > 0) then target = previewTarget end
	end
	if not target then
		snapBeam.Enabled = false
		return
	end

	local targetPos = target.Position
	if camera:WorldToViewportPoint(targetPos).Z <= 0 then
		snapBeam.Enabled = false
		return
	end

	local viewport = camera.ViewportSize
	local focal = (viewport.Y / 2) / math.tan(math.rad(camera.FieldOfView) / 2)
	local ox, oy = viewport.X / 2, viewport.Y / 2
	local originMode = settings.SnapOrigin
	if originMode == "Top" then
		oy = 0
	elseif originMode == "Bottom" then
		oy = viewport.Y
	elseif originMode == "Mouse" then
		local m = UserInputService:GetMouseLocation()
		ox, oy = m.X, m.Y
	end
	local ray = camera:ViewportPointToRay(ox, oy)
	snapA0.WorldPosition = ray.Origin + ray.Direction * 2
	snapA1.WorldPosition = targetPos
	local p = settings.SnapThickness
	snapBeam.Width0 = 2 * p / focal
	snapBeam.Width1 = math.max((targetPos - camera.CFrame.Position).Magnitude, 2) * p / focal
	snapBeam.Color = ColorSequence.new(colorOf("ColSnap"))
	snapBeam.Enabled = true
end)

task.spawn(function()
	while not loaderFinished do task.wait(0.1) end
	task.wait(0.4)
	notify("loaded – RightShift hides the UI")
end)
