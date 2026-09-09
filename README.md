-- [ Leaked by eclipwze at Exe Fpsl https://discord.gg/aP5WGpBZk ]

-- Services -------------------------------------------------------------------

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local NetworkClient = game:GetService("NetworkClient")
local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer
local environment = if getgenv then getgenv() else _G
local RUNTIME_KEY = "__VYNX_ANTI_ANTI_TP"

-- Cleanly replace an earlier copy
local previousRuntime = environment[RUNTIME_KEY]
if type(previousRuntime) == "table" and type(previousRuntime.destroy) == "function" then
pcall(previousRuntime.destroy)
end

local runtime = {
alive = true,
enabled = false,
awaitingKey = false,
boundKey = Enum.KeyCode.Delete,
character = nil,
rootPart = nil,
fakeRoot = nil,
repRootOwner = nil,
stepConnection = nil,
connections = {},
settingsRestore = {},
captureGeneration = 0,

antiBatConn = nil,
freezeConn = nil,
flingConn = nil,
lastSafeCFrame = nil,
lastCheckTime = 0,

isMinimized = false,
isLocked = false,
}

environment[RUNTIME_KEY] = runtime

local ANTI_BAT_RANGE = 5

-- General helpers ------------------------------------------------------------

local function connect(signal, callback)
local connection = signal:Connect(callback)
table.insert(runtime.connections, connection)
return connection
end

local function disconnect(connection)
if connection then
pcall(function()
connection:Disconnect()
end)
end
end

local function create(className, properties, parent)
local object = Instance.new(className)
for property, value in pairs(properties or {}) do
object[property] = value
end
if parent then
object.Parent = parent
end
return object
end

local function corner(parent, radius)
return create("UICorner", {
CornerRadius = typeof(radius) == "UDim" and radius or UDim.new(0, radius),
}, parent)
end

local function stroke(parent, color, transparency, thickness)
return create("UIStroke", {
ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
Color = color,
Transparency = transparency,
Thickness = thickness,
}, parent)
end

local function tween(object, duration, goals)
local animation = TweenService:Create(
object,
TweenInfo.new(duration, Enum.EasingStyle.Quint, Enum.EasingDirection.Out),
goals
)
animation:Play()
return animation
end

local function isBasePart(instance)
if not instance then
return false
end
local ok, result = pcall(function()
return instance:IsA("BasePart")
end)
return ok and result == true
end

local function getCurrentRoot(character)
character = character or LocalPlayer.Character
if not character then
return nil
end

local ok, root = pcall(function()
return character:FindFirstChild("HumanoidRootPart")
end)
if ok and isBasePart(root) then
return root
end
return nil
end

-- Executor compatibility -----------------------------------------------------

local function findGlobalFunction(...)
for index = 1, select("#", ...) do
local name = select(index, ...)
local value = rawget(environment, name)
if type(value) == "function" then
return value
end
end
return nil
end

local function setHidden(instance, property, value)
if not instance then
return false
end

local setter = findGlobalFunction(
"sethiddenproperty",
"set_hidden_property",
"sethiddenprop",
"set_hidden_prop"
)
if setter then
local ok = pcall(setter, instance, property, value)
if ok then
return true
end
end

return pcall(function()
instance[property] = value
end)
end

local function getHidden(instance, property)
if not instance then
return false, nil
end

local getter = findGlobalFunction(
"gethiddenproperty",
"get_hidden_property",
"gethiddenprop",
"get_hidden_prop"
)
if getter then
local ok, value = pcall(getter, instance, property)
if ok then
return true, value
end
end

local ok, value = pcall(function()
return instance[property]
end)
return ok, value
end

-- Physics/network setup ------------------------------------------------------

local function rememberSetting(instance, property)
local ok, value = pcall(function()
return instance[property]
end)
if ok then
table.insert(runtime.settingsRestore, {
instance = instance,
property = property,
value = value,
})
end
end

local function applyPublicSetting(instance, property, value)
if not instance then
return false
end
rememberSetting(instance, property)
return pcall(function()
instance[property] = value
end)
end

local function configurePhysics()
setHidden(LocalPlayer, "MaximumSimulationRadius", math.huge)
setHidden(LocalPlayer, "SimulationRadius", math.huge)

pcall(function()
local networkSettings = settings().Network
applyPublicSetting(
networkSettings,
"InterpolationThrottling",
Enum.InterpolationThrottlingMode.Disabled
)
end)

pcall(function()
local physicsSettings = settings().Physics
applyPublicSetting(
physicsSettings,
"PhysicsEnvironmentalThrottle",
Enum.EnviromentalPhysicsThrottle.Disabled
)
applyPublicSetting(physicsSettings, "AllowSleep", false)
end)

pcall(function()
NetworkClient:SetOutgoingKBPSLimit(math.huge)
end)
end

configurePhysics()

-- Replication-root runtime ---------------------------------------------------

local FAKE_ROOT_NAME = "DavidDesyncRoot"
local FAKE_ROOT_Y = -1000
local FAKE_ROOT_VELOCITY = Vector3.new(0, -1000, 0)

local function fakeRootIsUsable()
local fake = runtime.fakeRoot
if not isBasePart(fake) then
return false
end
local ok, parent = pcall(function()
return fake.Parent
end)
return ok and parent ~= nil
end

local function destroyFakeRoot()
local fake = runtime.fakeRoot
runtime.fakeRoot = nil
if fake then
pcall(function()
fake:Destroy()
end)
end
end

local function restoreReplicationRoot()
local owner = runtime.repRootOwner or runtime.rootPart
if isBasePart(owner) then
setHidden(owner, "PhysicsRepRootPart", owner)
end
runtime.repRootOwner = nil
end

local function createFakeRoot(rootPart)
destroyFakeRoot()

local fake = create("Part", {
Name = FAKE_ROOT_NAME,
Size = Vector3.new(2, 2, 1),
Anchored = true,
CanCollide = false,
CanTouch = false,
CanQuery = false,
Transparency = 1,
CFrame = CFrame.new(0, FAKE_ROOT_Y, 0),
AssemblyLinearVelocity = FAKE_ROOT_VELOCITY,
}, Workspace)

local ok, position = pcall(function()
return rootPart.Position
end)
if ok then
fake.CFrame = CFrame.new(position.X, FAKE_ROOT_Y, position.Z)
end

runtime.fakeRoot = fake
return fake
end

local function assignFakeReplicationRoot(rootPart, fake)
if not isBasePart(rootPart) or not isBasePart(fake) then
return false
end

setHidden(rootPart, "PhysicsRepRootPart", rootPart)
runtime.repRootOwner = rootPart
return setHidden(rootPart, "PhysicsRepRootPart", fake)
end

local function stepDesync()
if not runtime.alive or not runtime.enabled then
return
end

local root = runtime.rootPart
if not isBasePart(root) then
root = getCurrentRoot(runtime.character)
runtime.rootPart = root
end
if not root then
return
end

if not fakeRootIsUsable() then
local fake = createFakeRoot(root)
assignFakeReplicationRoot(root, fake)
return
end

local fake = runtime.fakeRoot

local ok, rootPosition, fakePosition = pcall(function()
return root.Position, fake.Position
end)
if ok and (
math.abs(rootPosition.X - fakePosition.X) > 0.01
or math.abs(rootPosition.Z - fakePosition.Z) > 0.01
or math.abs(fakePosition.Y - FAKE_ROOT_Y) > 0.01
) then
pcall(function()
fake.CFrame = CFrame.new(rootPosition.X, FAKE_ROOT_Y, rootPosition.Z)
end)
end

pcall(function()
fake.Anchored = true
fake.AssemblyLinearVelocity = FAKE_ROOT_VELOCITY
end)

local gotValue, current = getHidden(root, "PhysicsRepRootPart")
if not gotValue or current ~= fake then
setHidden(root, "PhysicsRepRootPart", fake)
end
end

local function stopStepConnection()
disconnect(runtime.stepConnection)
runtime.stepConnection = nil
end

local function startStepConnection()
stopStepConnection()
runtime.stepConnection = RunService.Stepped:Connect(stepDesync)
end

-- ========== ANTI-BAT / FREEZE / FLING ==========

local function stopAntiBat()
if runtime.antiBatConn then
runtime.antiBatConn:Disconnect()
runtime.antiBatConn = nil
end
runtime.lastSafeCFrame = nil
end

local function startAntiBat()
stopAntiBat()
runtime.lastSafeCFrame, runtime.lastCheckTime = nil, 0

runtime.antiBatConn = RunService.Heartbeat:Connect(function()
if not runtime.enabled or not runtime.alive then return end

local char = LocalPlayer.Character
if not char then return end
local hrp = char:FindFirstChild("HumanoidRootPart")
local hum = char:FindFirstChildOfClass("Humanoid")
if not hrp or not hum or hum.Health <= 0 then return end

local now = tick()

local velocity = hrp.AssemblyLinearVelocity
if velocity.Magnitude < 70 then
runtime.lastSafeCFrame = hrp.CFrame
runtime.lastCheckTime = now
elseif velocity.Magnitude > 110 and runtime.lastSafeCFrame and (now - runtime.lastCheckTime) < 1.5 then
hrp.CFrame = runtime.lastSafeCFrame * CFrame.new(0, 0.1, 0)
end

for _, plr in ipairs(Players:GetPlayers()) do
if plr ~= LocalPlayer and plr.Character then
local eHrp = plr.Character:FindFirstChild("HumanoidRootPart")
local tool = plr.Character:FindFirstChildWhichIsA("Tool")
if eHrp and tool and tool.Name:lower():find("bat") then
local dist = (hrp.Position - eHrp.Position).Magnitude
if dist < ANTI_BAT_RANGE then
local angle = math.rad(tick() * 500)
hrp.CFrame = hrp.CFrame * CFrame.new(math.sin(angle) * 3, 0, math.cos(angle) * 3)
end
end
end
end
end)
end

local function stopFreeze()
if runtime.freezeConn then
runtime.freezeConn:Disconnect()
runtime.freezeConn = nil
end
end

local function startFreeze()
stopFreeze()
runtime.freezeConn = RunService.Heartbeat:Connect(function()
if not runtime.enabled or not runtime.alive then return end
for _, plr in ipairs(Players:GetPlayers()) do
if plr ~= LocalPlayer and plr.Character then
local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
if hrp then
hrp.AssemblyLinearVelocity = Vector3.zero
hrp.AssemblyAngularVelocity = Vector3.zero
end
end
end
end)
end

local function stopFling()
if runtime.flingConn then
runtime.flingConn:Disconnect()
runtime.flingConn = nil
end
end

local function startFling()
stopFling()
runtime.flingConn = RunService.Heartbeat:Connect(function()
if not runtime.enabled or not runtime.alive then return end
local myChar = LocalPlayer.Character
if not myChar then return end
local myHrp = myChar:FindFirstChild("HumanoidRootPart")
if not myHrp then return end

for _, plr in ipairs(Players:GetPlayers()) do
if plr ~= LocalPlayer and plr.Character then
local eHrp = plr.Character:FindFirstChild("HumanoidRootPart")
if eHrp then
local dist = (myHrp.Position - eHrp.Position).Magnitude
if dist < ANTI_BAT_RANGE then
local dir = (eHrp.Position - myHrp.Position).Unit
eHrp.AssemblyLinearVelocity = dir * 150 + Vector3.new(0, 80, 0)
end
end
end
end
end)
end

-- Character lifecycle --------------------------------------------------------

local function bindCharacter(character)
local oldRoot = runtime.rootPart
runtime.character = character
runtime.rootPart = getCurrentRoot(character)

if runtime.enabled then
if isBasePart(oldRoot) and oldRoot ~= runtime.rootPart then
setHidden(oldRoot, "PhysicsRepRootPart", oldRoot)
end
destroyFakeRoot()

local root = runtime.rootPart
if not root and character then
local ok, waitedRoot = pcall(function()
return character:WaitForChild("HumanoidRootPart", 8)
end)
if ok and isBasePart(waitedRoot) then
root = waitedRoot
runtime.rootPart = root
end
end

if root then
local fake = createFakeRoot(root)
assignFakeReplicationRoot(root, fake)
startStepConnection()
end

startAntiBat()
startFreeze()
startFling()
end
end

bindCharacter(LocalPlayer.Character)
connect(LocalPlayer.CharacterAdded, function(character)
task.defer(bindCharacter, character)
end)

-- Interface palette (neutral, no purple) -------------------------------------

local COLORS = {
main = Color3.fromRGB(12, 12, 14),
row = Color3.fromRGB(22, 22, 26),
track = Color3.fromRGB(30, 30, 36),
button = Color3.fromRGB(18, 18, 22),
text = Color3.new(1, 1, 1),
muted = Color3.fromRGB(140, 140, 150),
accent = Color3.fromRGB(220, 220, 230),
rowStroke = Color3.fromRGB(55, 55, 65),
}

local uiParent = CoreGui
local oldGui = uiParent:FindFirstChild("VynxAntiAntiTP")
if oldGui then
oldGui:Destroy()
end

local screenGui = create("ScreenGui", {
Name = "VynxAntiAntiTP",
DisplayOrder = 999,
ResetOnSpawn = false,
IgnoreGuiInset = true,
ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
}, nil)

local parented = pcall(function()
screenGui.Parent = uiParent
end)
if not parented then
local playerGui = LocalPlayer:FindFirstChildOfClass("PlayerGui")
or LocalPlayer:WaitForChild("PlayerGui")
uiParent = playerGui
local stale = uiParent:FindFirstChild("VynxAntiAntiTP")
if stale then
stale:Destroy()
end
screenGui.Parent = uiParent
end
runtime.gui = screenGui

-- Main panel -----------------------------------------------------------------

local main = create("Frame", {
Name = "Main",
Active = true,
ClipsDescendants = true,
BackgroundTransparency = 0.15,
BackgroundColor3 = COLORS.main,
BorderSizePixel = 0,
Position = UDim2.new(0.5, -155, 0.5, -90),
Size = UDim2.new(0, 310, 0, 175),
}, screenGui)
corner(main, 16)

-- Background image (requested asset)
local bgImage = create("ImageLabel", {
Name = "Background",
BackgroundTransparency = 1,
Image = "rbxassetid://115695750561610",
ScaleType = Enum.ScaleType.Crop,
ImageTransparency = 0.35,
Size = UDim2.new(1, 0, 1, 0),
ZIndex = 0,
}, main)
corner(bgImage, 16)

local mainScale = create("UIScale", {
Scale = 1,
}, main)

-- Header ---------------------------------------------------------------------

local FULL_HEIGHT = 175
local MINI_HEIGHT = 40

local header = create("Frame", {
Name = "Header",
BackgroundTransparency = 1,
Position = UDim2.new(0, 16, 0, 6),
ZIndex = 10,
Size = UDim2.new(1, -24, 0, 40),
}, main)
corner(header, 6)

local title = create("TextLabel", {
Name = "Title",
BackgroundTransparency = 1,
Text = "VYNX ANTI ANTI TP",
TextColor3 = COLORS.text,
Font = Enum.Font.GothamBlack,
Position = UDim2.new(0, 4, 0, 0),
TextXAlignment = Enum.TextXAlignment.Left,
ZIndex = 11,
TextSize = 14,
Size = UDim2.new(1, -120, 1, 0),
}, header)

-- Lock / Unlock button
local lockBtn = create("TextButton", {
Name = "LockBtn",
AutoButtonColor = false,
BackgroundColor3 = COLORS.button,
BackgroundTransparency = 0.25,
BorderSizePixel = 0,
Position = UDim2.new(1, -92, 0.5, -12),
Size = UDim2.new(0, 58, 0, 24),
Text = "Lock",
TextColor3 = COLORS.text,
Font = Enum.Font.GothamBold,
TextSize = 11,
ZIndex = 12,
}, header)
corner(lockBtn, 6)

-- Minimize button (-)
local minimizeBtn = create("TextButton", {
Name = "MinimizeBtn",
AutoButtonColor = false,
BackgroundColor3 = COLORS.button,
BackgroundTransparency = 0.25,
BorderSizePixel = 0,
Position = UDim2.new(1, -28, 0.5, -12),
Size = UDim2.new(0, 24, 0, 24),
Text = "-",
TextColor3 = COLORS.text,
Font = Enum.Font.GothamBlack,
TextSize = 16,
ZIndex = 12,
}, header)
corner(minimizeBtn, 6)

-- Content and row helper -----------------------------------------------------

local content = create("Frame", {
Name = "Content",
BackgroundTransparency = 1,
Position = UDim2.new(0, 18, 0, 48),
ZIndex = 5,
Size = UDim2.new(1, -28, 1, -54),
}, main)
create("UIListLayout", {
Padding = UDim.new(0, 8),
SortOrder = Enum.SortOrder.LayoutOrder,
}, content)

local function makeRow(name, layoutOrder)
local row = create("Frame", {
Name = name,
BackgroundColor3 = COLORS.row,
BackgroundTransparency = 0.35,
BorderSizePixel = 0,
Size = UDim2.new(1, 0, 0, 46),
LayoutOrder = layoutOrder,
ZIndex = 5,
}, content)
corner(row, 10)
return row
end

local toggleRow = makeRow("AntiAntiRow", 1)
local toggleLabel = create("TextLabel", {
Name = "Label",
BackgroundTransparency = 1,
Text = "Enable Anti Anti",
TextColor3 = COLORS.text,
Font = Enum.Font.GothamBold,
Position = UDim2.new(0, 14, 0, 6),
TextXAlignment = Enum.TextXAlignment.Left,
ZIndex = 6,
TextSize = 13,
Size = UDim2.new(1, -74, 0, 18),
}, toggleRow)

local statusLabel = create("TextLabel", {
Name = "Status",
BackgroundTransparency = 1,
Text = "OFF",
TextColor3 = COLORS.muted,
Font = Enum.Font.GothamBold,
Position = UDim2.new(0, 14, 0, 24),
TextXAlignment = Enum.TextXAlignment.Left,
ZIndex = 6,
TextSize = 9,
Size = UDim2.new(1, -74, 0, 14),
}, toggleRow)

local toggleTrack = create("Frame", {
Name = "Toggle",
AnchorPoint = Vector2.new(1, 0.5),
BackgroundColor3 = COLORS.track,
BorderSizePixel = 0,
Position = UDim2.new(1, -12, 0.5, 0),
ZIndex = 7,
Size = UDim2.new(0, 44, 0, 22),
}, toggleRow)
corner(toggleTrack, 11)

local toggleKnob = create("Frame", {
Name = "Knob",
BackgroundColor3 = Color3.new(1, 1, 1),
BorderSizePixel = 0,
Size = UDim2.new(0, 16, 0, 16),
Position = UDim2.new(0, 3, 0, 3),
ZIndex = 8,
}, toggleTrack)
corner(toggleKnob, UDim.new(1, 0))

local toggleHit = create("TextButton", {
Name = "ToggleHit",
BackgroundTransparency = 1,
BorderSizePixel = 0,
Text = "",
AutoButtonColor = false,
ZIndex = 9,
Size = UDim2.new(1, 0, 1, 0),
}, toggleRow)

local keybindRow = makeRow("KeybindRow", 2)
local keybindLabel = create("TextLabel", {
Name = "Label",
BackgroundTransparency = 1,
Text = "Keybind",
TextColor3 = COLORS.text,
Font = Enum.Font.GothamBold,
Position = UDim2.new(0, 14, 0, 0),
TextXAlignment = Enum.TextXAlignment.Left,
ZIndex = 6,
TextSize = 13,
Size = UDim2.new(1, -84, 1, 0),
}, keybindRow)

local keybindButton = create("TextButton", {
Name = "KeybindBtn",
AutoButtonColor = false,
AnchorPoint = Vector2.new(1, 0.5),
BackgroundColor3 = COLORS.button,
BackgroundTransparency = 0.30,
BorderSizePixel = 0,
Position = UDim2.new(1, -12, 0.5, 0),
Size = UDim2.new(0, 76, 0, 26),
Text = "Delete",
TextColor3 = COLORS.accent,
Font = Enum.Font.GothamBlack,
TextSize = 11,
ZIndex = 7,
}, keybindRow)
corner(keybindButton, 7)

runtime.refs = {
screenGui = screenGui,
main = main,
mainScale = mainScale,
header = header,
title = title,
content = content,
toggleRow = toggleRow,
toggleLabel = toggleLabel,
statusLabel = statusLabel,
toggleTrack = toggleTrack,
toggleKnob = toggleKnob,
toggleHit = toggleHit,
keybindRow = keybindRow,
keybindLabel = keybindLabel,
keybindButton = keybindButton,
}

-- Interface effects ----------------------------------------------------------

local function ripple(row)
if not runtime.alive or not row or not row.Parent then
return
end

local mousePosition = UserInputService:GetMouseLocation()
local absolutePosition = row.AbsolutePosition
local absoluteSize = row.AbsoluteSize
local x = mousePosition.X - absolutePosition.X
local y = mousePosition.Y - absolutePosition.Y
local diameter = math.max(absoluteSize.X, absoluteSize.Y) * 1.35

local image = create("ImageLabel", {
Name = "Ripple",
BackgroundTransparency = 1,
Image = "rbxassetid://266543268",
ImageColor3 = Color3.fromRGB(200, 200, 210),
ImageTransparency = 0.40,
AnchorPoint = Vector2.new(0.5, 0.5),
Position = UDim2.new(0, x, 0, y),
Size = UDim2.new(0, 0, 0, 0),
ZIndex = 30,
}, row)

local animation = tween(image, 0.45, {
Size = UDim2.new(0, diameter, 0, diameter),
ImageTransparency = 1,
})
animation.Completed:Connect(function()
if image then
image:Destroy()
end
end)
end

local function applyEnabledVisual(value, instant)
statusLabel.Text = value and "ACTIVE" or "OFF"
statusLabel.TextColor3 = value and Color3.fromRGB(120, 255, 160) or COLORS.muted

local trackColor = value and Color3.fromRGB(40, 90, 55) or COLORS.track
local knobPosition = value and UDim2.new(1, -19, 0, 3)
or UDim2.new(0, 3, 0, 3)

if instant then
toggleTrack.BackgroundColor3 = trackColor
toggleKnob.Position = knobPosition
else
tween(toggleTrack, 0.18, { BackgroundColor3 = trackColor })
tween(toggleKnob, 0.18, { Position = knobPosition })
end
end

-- Enable/disable -------------------------------------------------------------

local function setEnabled(value)
if not runtime.alive then
return false
end

value = value == true
if runtime.enabled == value then
applyEnabledVisual(value, false)
return value
end

runtime.enabled = value
applyEnabledVisual(value, false)

if value then
local root = getCurrentRoot(runtime.character)
runtime.rootPart = root
if not root then
runtime.enabled = false
applyEnabledVisual(false, false)
return false
end

local fake = createFakeRoot(root)
assignFakeReplicationRoot(root, fake)
startStepConnection()

startAntiBat()
startFreeze()
startFling()
else
stopStepConnection()
restoreReplicationRoot()
destroyFakeRoot()

stopAntiBat()
stopFreeze()
stopFling()
end

return runtime.enabled
end

local function toggleEnabled()
return setEnabled(not runtime.enabled)
end

connect(toggleHit.MouseButton1Click, function()
ripple(toggleRow)
toggleEnabled()
end)

-- Key capture and bound-key toggle ------------------------------------------

connect(keybindButton.MouseButton1Click, function()
if not runtime.alive or runtime.awaitingKey then
return
end

ripple(keybindRow)
runtime.awaitingKey = true
runtime.captureGeneration += 1
local generation = runtime.captureGeneration

task.spawn(function()
for _, text in ipairs({".", "..", "..."}) do
if not runtime.alive
or not runtime.awaitingKey
or generation ~= runtime.captureGeneration
then
return
end
keybindButton.Text = text
task.wait(0.15)
end
end)
end)

local function isValidBindInput(input)
if input.KeyCode == Enum.KeyCode.Unknown then
return false
end
local t = input.UserInputType
return t == Enum.UserInputType.Keyboard
or t == Enum.UserInputType.Gamepad1
or t == Enum.UserInputType.Gamepad2
or t == Enum.UserInputType.Gamepad3
or t == Enum.UserInputType.Gamepad4
or t == Enum.UserInputType.Gamepad5
or t == Enum.UserInputType.Gamepad6
or t == Enum.UserInputType.Gamepad7
or t == Enum.UserInputType.Gamepad8
end

connect(UserInputService.InputBegan, function(input, gameProcessed)
if not runtime.alive then
return
end

if runtime.awaitingKey then
if isValidBindInput(input) then
-- Escape cancels without changing the bind
if input.KeyCode ~= Enum.KeyCode.Escape then
runtime.boundKey = input.KeyCode
end
runtime.awaitingKey = false
runtime.captureGeneration += 1
keybindButton.Text = runtime.boundKey.Name
end
return
end

if not gameProcessed and input.KeyCode == runtime.boundKey then
ripple(toggleRow)
toggleEnabled()
end
end)

-- Minimize & Lock ------------------------------------------------------------

connect(minimizeBtn.MouseButton1Click, function()
if not runtime.alive then return end
runtime.isMinimized = not runtime.isMinimized
minimizeBtn.Text = runtime.isMinimized and "+" or "-"
TweenService:Create(main, TweenInfo.new(0.22, Enum.EasingStyle.Quad), {
Size = UDim2.new(0, 310, 0, runtime.isMinimized and MINI_HEIGHT or FULL_HEIGHT)
}):Play()
end)

connect(lockBtn.MouseButton1Click, function()
if not runtime.alive then return end
runtime.isLocked = not runtime.isLocked
lockBtn.Text = runtime.isLocked and "Unlock" or "Lock"
end)

-- Draggable panel ------------------------------------------------------------

local dragging = false
local dragInput = nil
local dragStart = nil
local startPosition = nil

connect(main.InputBegan, function(input)
if runtime.isLocked then return end
if input.UserInputType == Enum.UserInputType.MouseButton1
or input.UserInputType == Enum.UserInputType.Touch
then
dragging = true
dragInput = input
dragStart = input.Position
startPosition = main.Position

local changedConnection
changedConnection = input.Changed:Connect(function()
if input.UserInputState == Enum.UserInputState.End then
dragging = false
dragInput = nil
disconnect(changedConnection)
end
end)
end
end)

connect(main.InputChanged, function(input)
if input.UserInputType == Enum.UserInputType.MouseMovement
or input.UserInputType == Enum.UserInputType.Touch
then
dragInput = input
end
end)

connect(UserInputService.InputChanged, function(input)
if runtime.isLocked or not dragging or input ~= dragInput or not dragStart or not startPosition then
return
end

local delta = input.Position - dragStart
main.Position = UDim2.new(
startPosition.X.Scale,
startPosition.X.Offset + delta.X,
startPosition.Y.Scale,
startPosition.Y.Offset + delta.Y
)
end)

-- Cleanup --------------------------------------------------------------------

local function destroy()
if not runtime.alive then
return
end

runtime.alive = false
runtime.enabled = false
runtime.awaitingKey = false
runtime.captureGeneration += 1

stopStepConnection()
restoreReplicationRoot()
destroyFakeRoot()

stopAntiBat()
stopFreeze()
stopFling()

for _, connection in ipairs(runtime.connections) do
disconnect(connection)
end
table.clear(runtime.connections)

for index = #runtime.settingsRestore, 1, -1 do
local entry = runtime.settingsRestore[index]
pcall(function()
entry.instance[entry.property] = entry.value
end)
end
table.clear(runtime.settingsRestore)

if runtime.gui then
pcall(function()
runtime.gui:Destroy()
end)
end

if environment[RUNTIME_KEY] == runtime then
environment[RUNTIME_KEY] = nil
end
end

runtime.setEnabled = setEnabled
runtime.toggle = toggleEnabled
runtime.step = stepDesync
runtime.bindCharacter = bindCharacter
runtime.setHidden = setHidden
runtime.getHidden = getHidden
runtime.destroy = destroy
runtime.getBoundKey = function()
return runtime.boundKey
end
runtime.setBoundKey = function(keyCode)
if keyCode and keyCode ~= Enum.KeyCode.Unknown then
runtime.boundKey = keyCode
keybindButton.Text = keyCode.Name
return true
end
return false
end

applyEnabledVisual(false, true)
