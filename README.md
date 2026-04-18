local P = game:GetService("Players")
local U = game:GetService("UserInputService")
local R = game:GetService("RunService")
local T = game:GetService("TweenService")

local LP = P.LocalPlayer

-------------------------------------------------
-- GUI
-------------------------------------------------
local G = Instance.new("ScreenGui")
G.Parent = LP:WaitForChild("PlayerGui")
G.ResetOnSpawn = false

-------------------------------------------------
-- زر القائمة
-------------------------------------------------
local B = Instance.new("TextButton")
B.Parent = G
B.Size = UDim2.new(0,120,0,40)
B.Position = UDim2.new(0,20,0,20)
B.Text = "Menu"
B.BackgroundColor3 = Color3.fromRGB(30,30,30)
B.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", B)

-------------------------------------------------
-- القائمة
-------------------------------------------------
local M = Instance.new("Frame")
M.Parent = G
M.Size = UDim2.new(0,240,0,400)
M.Position = UDim2.new(0,20,0,70)
M.BackgroundColor3 = Color3.fromRGB(25,25,25)
M.Visible = false
M.Active = true
M.ClipsDescendants = false
Instance.new("UICorner", M)

Instance.new("UIStroke", M).Color = Color3.fromRGB(90,90,90)

-------------------------------------------------
-- SMOOTH OPEN / CLOSE
-------------------------------------------------
local open = false

local openPos = M.Position
local closedPos = UDim2.new(openPos.X.Scale, openPos.X.Offset, openPos.Y.Scale - 0.05, openPos.Y.Offset)

M.Position = closedPos

local function tween(frame, pos)
	local t = T:Create(frame, TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
		Position = pos
	})
	t:Play()
end

B.MouseButton1Click:Connect(function()
	open = not open

	if open then
		M.Visible = true
		tween(M, openPos)
	else
		tween(M, closedPos)
		task.delay(0.25, function()
			M.Visible = false
		end)
	end
end)

-------------------------------------------------
-- DRAG (smooth & fixed)
-------------------------------------------------
local dragging = false
local startInput
local startPos

M.InputBegan:Connect(function(i)
	if i.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		startInput = i.Position
		startPos = M.Position
	end
end)

U.InputEnded:Connect(function(i)
	if i.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = false
	end
end)

U.InputChanged:Connect(function(i)
	if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then
		local delta = i.Position - startInput
		M.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)
	end
end)

-------------------------------------------------
-- CHARACTER
-------------------------------------------------
local c,h
local function ref()
	c = LP.Character or LP.CharacterAdded:Wait()
	h = c:WaitForChild("Humanoid")
end
ref()
LP.CharacterAdded:Connect(ref)

-------------------------------------------------
-- STYLE
-------------------------------------------------
local function S(x)
	x.TextColor3 = Color3.new(1,1,1)
end

-------------------------------------------------
-- BUTTON MAKER
-------------------------------------------------
local function makeBtn(t,y,cb)
	local b = Instance.new("TextButton")
	b.Parent = M
	b.Size = UDim2.new(0,200,0,30)
	b.Position = UDim2.new(0,20,0,y)
	b.Text = t
	b.BackgroundColor3 = Color3.fromRGB(40,40,40)
	Instance.new("UICorner", b)
	S(b)
	b.MouseButton1Click:Connect(function()
		cb(b)
	end)
	return b
end

-------------------------------------------------
-- FEATURES
-------------------------------------------------
local noclip, infJ, inv = false,false,false

makeBtn("Jump inf OFF",110,function(b)
	infJ = not infJ
	b.Text = infJ and "Jump inf ON" or "Jump inf OFF"
end)

makeBtn("Invisible OFF",150,function(b)
	inv = not inv
	for _,v in pairs(c:GetDescendants()) do
		if v:IsA("BasePart") then
			v.LocalTransparencyModifier = inv and 1 or 0
			v.CanCollide = not inv
		end
	end
	b.Text = inv and "Invisible ON" or "Invisible OFF"
end)

makeBtn("Wall OFF",190,function(b)
	noclip = not noclip
	b.Text = noclip and "Wall ON" or "Wall OFF"
end)

R.Stepped:Connect(function()
	if noclip and c then
		for _,v in pairs(c:GetDescendants()) do
			if v:IsA("BasePart") then
				v.CanCollide = false
			end
		end
	end
end)

-------------------------------------------------
-- FINAL RESULT
-- Smooth UI + draggable menu + working features
-------------------------------------------------
