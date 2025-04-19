type signalConnection = typeof(setmetatable) & {
	Disconnect: (self: signalConnection) -> nil,
} & {
	__index: signalConnection,
} & (...any) -> (boolean, string)

type signalClass = typeof(setmetatable) & {
	Connect: (self: signalClass, callback: (...any) -> nil) -> signalConnection,
	Send: (self: signalClass, ...any) -> nil
} & {
	Connections: { [number]: signalConnection? },
}

type visualClass = typeof(setmetatable) & {
	Create: (self: visualClass, parent: Instance, class: string, filter: { [number]: string }, highlight: boolean?) -> nil,
	Update: (self: visualClass, parent: Instance, class: string, filter: { [number]: string }, highlight: boolean?) -> nil,
	List: (self: visualClass, parent: Instance) -> { [number]: BillboardGui },
	Add: (self: visualClass, parent: Instance, child: Instance, highlight: boolean?) -> nil,
	ClearConnections: (self:visualClass) -> nil,
	Remove: (self:visualClass) -> nil,
} & {
	Objects: { [Instance]: { [BillboardGui]: Instance } },
	Connections: { [number]: RBXScriptConnection },
	PlayerSignal: signalClass,
	CameraSignal: signalClass,
	Local: Player
}

local Signal = {
	new = function()
		return setmetatable({
			Connections = {},
		}, (function(self)
			self.__index = self;
			self.__metatable = "locked";

			return self;
		end)({
			Connect = function(self: signalClass, callback: () -> nil)
				local stringId = game.HttpService:GenerateGUID(false);

				local metadata = setmetatable({}, (function(this)
					this.__index = this;

					return this;
				end)({
					Disconnect = function(this)
						rawset(self.Connections, stringId, nil)
						setmetatable(this, nil);
					end,
					__call = function(this, ...)
						return pcall(callback, ...)
					end
				}));

				rawset(self.Connections, stringId, metadata);
				return metadata;
			end,
			Send = function(self: signalClass, ...)
				for _, signal in self.Connections do
					local success: boolean, response: string = signal(...);
					if not success then warn(response) end;
				end;
			end
		}))
	end
} :: {
	new: () -> signalClass
}

type highlightClass = typeof(setmetatable) & {
	Select: (self: highlightClass, child: any) -> nil,
	Deselect: (self: highlightClass, child: any) -> nil
} & {
	Highlighting: { [number]: Highlight? },
	Instances: { [number]: any? },
	Amount: number
}

local Highlight = setmetatable({
	Instances = nil,
	Highlighting = {},
	Amount = 0
}, (function(self)
	self.__index = self;
	self.__metatable = "locked";

	self.Instances = {};

	for i = 0, 31, 1 do
		local highlight = Instance.new("Highlight")
		highlight.FillTransparency = 1
		highlight.Enabled = false;

		table.insert(self.Instances, highlight);
	end;

	if rawget(shared, "Highlight") and typeof(shared["Highlight"]) == "function" then shared["Highlight"]() end;
	shared["Highlight"] = function()
		for _, highlight in self.Instances do
			highlight:Destroy();
		end;
	end;

	return self;
end)({
	Select = function(self: highlightClass, child: any): ()
		if rawget(self.Highlighting, child) then return end

		if self.Amount >= 31 then return end

		local highlight = self.Instances[self.Amount + 1]
		if highlight then
			self.Amount = self.Amount + 1

			highlight.Parent = game.CoreGui
			highlight.Adornee = child
			highlight.Enabled = true
			rawset(self.Highlighting, child, highlight)
		end
	end,
	Deselect = function(self: highlightClass, child: any)
		local highlight = rawget(self.Highlighting, child)

		if highlight then
			self.Amount = self.Amount - 1

			highlight.Adornee = nil
			highlight.Enabled = false
			rawset(self.Highlighting, child, nil)
		end
	end,
})) :: highlightClass;

local Visual = {
	new = function()
		return setmetatable({
			Objects = {},
			Connections = {},
			PlayerSignal = Signal.new(),
			Local = game.Players.LocalPlayer
		}, (function(self)
			self.__index = self;
			self.__metatable = "locked";

			return self;
		end)({
			Start = function(self: visualClass, parent: Instance, class: string, filter: { [number]: string }, highlight: boolean?)
				table.insert(self.Connections, parent.ChildAdded:Connect(function(child: Instance)
					if filter and not table.find(filter, child.Name) then return end;
					self:Add(parent, child);
				end));

				table.insert(self.Connections, parent.ChildRemoved:Connect(function(child: Instance)
					local list = self:List(parent);
					local object: number? | Instance? = table.find(list, child);

					if typeof(object) == "BillboardGui" then
						(object :: BillboardGui):Destroy();
					end;
				end));

				table.insert(self.Connections, game:GetService("RunService").Heartbeat:Connect(function()
					local Character = self.Local.Character;
					if not Character then return end;

					self.PlayerSignal:Send(Character:GetPivot().Position);
				end));

				if rawget(shared, parent) and typeof(shared[parent]) == "function" then shared[parent]() end;
				shared[parent] = function()
					self:Remove();
				end;

				self:Update(parent, class, filter, highlight);
			end,
			Update = function(self: visualClass, parent: Instance, class: string, filter: { [number]: string }, highlight: boolean?)
				local current: { any }, list = {}, self:List(parent);

				for _, object:any in parent:GetChildren() do
					if filter and not table.find(filter, object.Name) then continue end;

					object = object:IsA("Model") and object:FindFirstChildOfClass("Part") or object

					if not object:FindFirstChildOfClass(class) then
						continue
					else
						table.insert(current, object);
						self:Add(parent, object, highlight);
					end;
				end;

				for billboard: BillboardGui, object: Instance in list do
					if table.find(current, object) then
						continue
					else
						billboard:Destroy();
					end;
				end;
			end,
			Add = function(self: visualClass, parent: Instance, child: Instance, highlight: boolean?)
				local ProximityPrompt = child:FindFirstChildWhichIsA("ProximityPrompt");
				if not ProximityPrompt then return end;

				local List = self:List(parent);

				local billboard_gui = Instance.new("BillboardGui")
				rawset(List, billboard_gui, child);

				billboard_gui.Active = true
				billboard_gui.AlwaysOnTop = true
				billboard_gui.LightInfluence = 1
				billboard_gui.Size = UDim2.new(0, 400, 0, 50)
				billboard_gui.ResetOnSpawn = true
				billboard_gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
				billboard_gui.Parent = game.CoreGui
				billboard_gui.Adornee = child
				billboard_gui.MaxDistance = 80

				local uilist_layout = Instance.new("UIListLayout")
				uilist_layout.VerticalFlex = Enum.UIFlexAlignment.Fill
				uilist_layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
				uilist_layout.SortOrder = Enum.SortOrder.LayoutOrder
				uilist_layout.VerticalAlignment = Enum.VerticalAlignment.Center
				uilist_layout.Parent = billboard_gui

				local canvas = Instance.new("CanvasGroup")
				canvas.AutomaticSize = Enum.AutomaticSize.XY
				canvas.BackgroundColor3 = Color3.new(0, 0, 0)
				canvas.BackgroundTransparency = 0.25
				canvas.BorderColor3 = Color3.new(0, 0, 0)
				canvas.BorderSizePixel = 0
				canvas.Visible = true
				canvas.Name = "Canvas"
				canvas.Parent = billboard_gui

				local uilist_layout2 = Instance.new("UIListLayout")
				uilist_layout2.VerticalFlex = Enum.UIFlexAlignment.Fill
				uilist_layout2.HorizontalAlignment = Enum.HorizontalAlignment.Center
				uilist_layout2.SortOrder = Enum.SortOrder.LayoutOrder
				uilist_layout2.Parent = canvas

				local uicorner = Instance.new("UICorner")
				uicorner.CornerRadius = UDim.new(0, 5)
				uicorner.Parent = canvas

				local uipadding = Instance.new("UIPadding")
				uipadding.PaddingBottom = UDim.new(0, 10)
				uipadding.PaddingLeft = UDim.new(0, 10)
				uipadding.PaddingRight = UDim.new(0, 10)
				uipadding.PaddingTop = UDim.new(0, 10)
				uipadding.Parent = canvas

				local context = Instance.new("TextLabel")
				context.Font = Enum.Font.Ubuntu
				context.Text = child.Name
				context.TextColor3 = Color3.new(1, 1, 1)
				context.TextSize = 18
				context.TextStrokeTransparency = 0
				context.TextWrapped = false
				context.AutomaticSize = Enum.AutomaticSize.XY
				context.BackgroundColor3 = Color3.new(1, 1, 1)
				context.BackgroundTransparency = 1
				context.BorderColor3 = Color3.new(0, 0, 0)
				context.BorderSizePixel = 0
				context.LayoutOrder = 2
				context.Visible = true
				context.Name = "Context"
				context.Parent = canvas

				local title = Instance.new("TextLabel")
				title.Font = Enum.Font.Ubuntu
				title.Text = `[{parent.Name}]`
				title.TextColor3 = Color3.new(1, 1, 1)
				title.TextSize = 13
				title.TextStrokeTransparency = 0
				title.TextTransparency = 0.25
				title.TextWrapped = false
				title.AutomaticSize = Enum.AutomaticSize.XY
				title.BackgroundColor3 = Color3.new(1, 1, 1)
				title.BackgroundTransparency = 1
				title.BorderColor3 = Color3.new(0, 0, 0)
				title.BorderSizePixel = 0
				title.LayoutOrder = 1
				title.Visible = true
				title.Name = "Title"
				title.Parent = canvas

				local uistroke = Instance.new("UIStroke")
				uistroke.Color = Color3.new(1, 1, 1)
				uistroke.Parent = canvas

				local promptShowning: boolean = false;
				table.insert(self.Connections, ProximityPrompt.PromptShown:Connect(function()
					promptShowning = true;
				end));

				table.insert(self.Connections, ProximityPrompt.PromptHidden:Connect(function()
					promptShowning = false;
				end));

				self.PlayerSignal:Connect(function(position: Vector3)
					local childPosition =
						(child:IsA("Model") and child:GetPivot().Position)
						or (string.match(child.ClassName, "Part") and child.Position)
						or (string.match(child.ClassName, "Union") and child.Position);

					local Magnitude = childPosition and (position - childPosition).Magnitude;

					local camera = workspace.CurrentCamera
					local screenWidth = camera.ViewportSize.X  -- Width of the screen
					local screenHeight = camera.ViewportSize.Y -- Height of the screen

					local viewportPosition, onScreen = camera:WorldToScreenPoint(childPosition)
					onScreen = onScreen and
						(viewportPosition.X >= 0 and viewportPosition.X <= screenWidth) and
						(viewportPosition.Y >= 0 and viewportPosition.Y <= screenHeight)

					if onScreen then
						local transparency = math.clamp(Magnitude / billboard_gui.MaxDistance, 0, 1)
						if (Magnitude and Magnitude <= ProximityPrompt.MaxActivationDistance) and promptShowning then
							billboard_gui.Enabled = false;
						else
							billboard_gui.Enabled = true;
						end;

						canvas.GroupTransparency = transparency
						uistroke.Transparency = transparency

						if highlight then Highlight:Select(child) end;
					else
						billboard_gui.Enabled = false;
						Highlight:Deselect(child)
					end;

					if ((string.match(child.ClassName, "Part") and child.Position)
						or (string.match(child.ClassName, "Union") and child.Position)) and child.Transparency == 1 then
						billboard_gui.Enabled = false;
					end;
				end);
			end,
			List = function(self: visualClass, parent: Instance)
				return rawget(self.Objects, parent) or rawset(self.Objects, parent, {}) and rawget(self.Objects, parent);
			end,
			Remove = function(self: visualClass)
				self:ClearConnections();

				for _, data in self.Objects do
					for billboard: BillboardGui, _ in data do
						billboard:Destroy();
					end;
				end;
			end,
			ClearConnections = function(self: visualClass)
				for _, Connection in self.Connections do
					Connection:Disconnect();
				end;
			end
		}));
	end
} :: {
	new: () -> visualClass
}

if _G["ESP_Enabled"] then
    _G.EndESP();
else
    _G["ESP_Enabled"] = true;

    local newClass = Visual.new();
    newClass:Start(workspace.Interactable, "ProximityPrompt", { "Roll", "Powder" }, true)
    newClass:Start(workspace, "ProximityPrompt", { "Present" });
    newClass:Start(workspace.Map.Boxes, "ProximityPrompt", { "Box" }, true);

    _G.EndESP = function()
        newClass:Remove()
        shared["Highlight"]()
        _G["ESP_Enabled"] = false;
    end
end
