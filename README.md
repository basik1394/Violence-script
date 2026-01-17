-- ROUND-SAFE AUTO REFRESHING GIFT ESP
local ESP_ENABLED = true
local ESP_CACHE = {}

local function addESP(part)
	if not part:IsA("BasePart") then return end
	if ESP_CACHE[part] then return end

	local box = Instance.new("BoxHandleAdornment")
	box.Name = "GiftESP"
	box.Adornee = part
	box.Size = part.Size
	box.AlwaysOnTop = true
	box.ZIndex = 10
	box.Color3 = Color3.fromRGB(0, 255, 0) -- green highlight
	box.Transparency = 0.4
	box.Parent = part

	ESP_CACHE[part] = box
end

local function removeESP(part)
	if ESP_CACHE[part] then
		ESP_CACHE[part]:Destroy()
		ESP_CACHE[part] = nil
	end
end

local giftList = {}
local function refreshESP()
	local currentGifts = {}
	giftList = {}

	for _, obj in pairs(workspace:GetDescendants()) do
		if obj.Name == "GiftHandle" and obj:IsA("BasePart") then
			currentGifts[obj] = true
			addESP(obj)
			table.insert(giftList, obj) -- sync giftList for tween
		end
	end

	-- Remove highlights for gifts that no longer exist
	for part, _ in pairs(ESP_CACHE) do
		if not currentGifts[part] then
			removeESP(part)
		end
	end
end

-- Initial ESP
refreshESP()

-- Auto refresh every 15 seconds
task.spawn(function()
	while true do
		task.wait(15)
		if ESP_ENABLED then
			refreshESP()
		end
	end
end)

-- ================== TWEEN / BUTTON SCRIPT ==================
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local player = Players.LocalPlayer

-- Fixed position (X)
local savedCFrame = nil
local giftIndex = 1
local giftOffset = Vector3.new(0, 3, 0)

-- Tween helper
local function tweenTo(cf)
	local char = player.Character
	local hrp = char and char:FindFirstChild("HumanoidRootPart")
	if not hrp then return end

	local dist = (hrp.Position - cf.Position).Magnitude
	local time = math.clamp(dist / 80, 0.15, 0.9)

	TweenService:Create(
		hrp,
		TweenInfo.new(time, Enum.EasingStyle.Linear),
		{CFrame = cf}
	):Play()
end

-- Button logic
local function setX()
	local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
	if hrp then
		savedCFrame = hrp.CFrame
	end
end

local function toX()
	if savedCFrame then
		tweenTo(savedCFrame)
	end
end

local function toGift()
	if #giftList == 0 then refreshESP() end
	if #giftList == 0 then return end

	if giftIndex > #giftList then
		giftIndex = 1
	end

	local gift = giftList[giftIndex]
	if gift and gift.Parent then
		tweenTo(CFrame.new(gift.Position + giftOffset))
		giftIndex += 1
	else
		refreshESP()
	end
end

-- ===== GUI =====
local gui = Instance.new("ScreenGui")
gui.Name = "UltraMiniTopButtons"
gui.ResetOnSpawn = false
gui.Parent = player.PlayerGui

local function makeBtn(text, xScale)
	local b = Instance.new("TextButton")
	b.Size = UDim2.new(0, 62, 0, 22) -- ULTRA SMALL
	b.Position = UDim2.new(xScale, 0, 0, 2) -- slightly up
	b.BackgroundColor3 = Color3.fromRGB(45, 45, 45) -- GREY
	b.TextColor3 = Color3.fromRGB(200, 200, 200) -- GREY TEXT
	b.TextScaled = true
	b.Text = text
	b.BorderSizePixel = 0
	b.Parent = gui
	Instance.new("UICorner", b).CornerRadius = UDim.new(0, 5)
	return b
end

-- Line-wise top buttons
local btnSetX = makeBtn("SET X", 0.02)
local btnToX  = makeBtn("TO X", 0.15)
local btnGift = makeBtn("GIFT",  0.28)

btnSetX.MouseButton1Click:Connect(setX)
btnToX.MouseButton1Click:Connect(toX)
btnGift.MouseButton1Click:Connect(toGift)
