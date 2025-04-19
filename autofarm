local RunService = game:GetService("RunService")
type signalConnection = typeof(setmetatable) & {
	Disconnect: (self: signalConnection) -> nil,
} & {
	__index: signalConnection,
} & (...any) -> (boolean, string)

type signalCache = typeof(setmetatable) & {
	Clear: (self: signalCache) -> nil,
} & {
	Data: any,
} & (...any) -> any

type signalClass = typeof(setmetatable) & {
	Connect: (self: signalClass, callback: (...any) -> nil) -> signalConnection,
	Ended: (self: signalClass, callback: (...any) -> nil) -> signalConnection,
	Send: (self: signalClass, ...any) -> nil,
	Cache: (self: signalClass, ...any) -> signalCache,
} & {
	Connections: { [number]: signalConnection? },
	Caches: { [number]: signalCache },
}

local Signal = {
	new = function()
		return setmetatable(
			{
				Connections = {},
				Caches = {},
			},
			(function(self)
				self.__index = self
				self.__metatable = "locked"

				return self
			end)({
				Connect = function(self: signalClass, callback: () -> nil)
					local metadata = setmetatable(
						{},
						(function(this)
							this.__index = this

							return this
						end)({
							Disconnect = function(this)
								setmetatable(this, nil)
							end,
							__call = function(this, cache)
								task.delay(1, cache.Clear, cache) -- Not sure, best thing I could think of OKAY!
								return pcall(callback, cache())
							end,
						})
					)

					table.insert(self.Connections, metadata)

					for _, cache in self.Caches do
						local success: boolean, response: string = metadata(cache)
						if not success then
							warn(response)
						end
					end

					return metadata
				end,
				Send = function(self: signalClass, ...)
					local cache = self:Cache(...)

					for _, signal in self.Connections do
						local success: boolean, response: string = signal(cache)
						if not success then
							warn(response)
						end
					end
				end,
				Cache = function(self: signalClass, ...)
					local metadata = setmetatable(
						{ Data = { ... } },
						(function(this)
							this.__index = this

							return this
						end)({
							Clear = function(this)
								this.Data = nil --< Idk, to make EXTRA SURE.
								setmetatable(this, nil)
							end,
							__call = function(this)
								return unpack(this.Data)
							end,
						})
					)

					table.insert(self.Caches, metadata)

					return metadata
				end,
			})
		)
	end,
} :: {
	new: () -> signalClass,
} -- This could be optimized, but I don't care.

-- -- -- -- -- -- -- -- -- -- --
-- -- -- -- -- -- -- -- -- -- --

-- Let's start with the basics, tyoes. Let's make the structure before the final code.

type collectorInfo = { -- Sounds sus, but let's for real here, this will be a list that checks if the parent object has a class that's needed.
	ParentList: { [number]: string },
	ChildList: { [number]: string },
	Factor: (object: Instance) -> boolean?,
}

-- Now let's think, what is our current collector coded like? Well it has a metatable, let's tell it that.
-- First we get the metatable class. Using typeof(inst)
-- From here, we wuil remove the __call functionality from setmetatble(_, _) to setmetatable.
-- Now let's make our function class, before the constructor. There is no order to this, it's just if you want to have seperate classes in the future.
type collectorFunctions = {
	-- Function Class
	-- > Now let's make a function suitable to the thing we are making.

	-- ? To understand this, () -< is in the input and the secondary -> () is the output. In our case. I normally make it nil if it has no after functionality.
	-- ? Now to actually even utilize our metatable, we will be making this function closed? I don't remember the terminology by it's like this function:()
	-- ? What do I mean by actually utilizing the metatable, well it's simple we make it use self, like we did when we created our "Function Class/Constructor".
	-- ? Self is as it describes, it's it self, Literally self, whatever metadata it has is self, so yes, it could call it self as well.
	Collect: (self: collectorClass, child: Instance, filter: collectorInfo, callback: (...any) -> nil?) -> signalClass,
	Check: (self: collectorClass, child: Instance, filter: collectorInfo) -> boolean,
	-- Let's actually try to make a .new thingy, idk what to call it.
	-- ? Just to note, this CAN not use self, that's just how it is :) Which is why I don't like to do this.
	new: () -> collectorInfo,
} -- Also we can treat anything in here, as "Public" functions, even though all functions can be called within the metadata.

type collectorConstructor = {
	-- Constructor, just like Javascript.
}

type collectorClass = typeof(setmetatable) & collectorFunctions & collectorConstructor -- What I normally do.

-- ! Also let me explain, filter: { [number]: string }. The reason why I am defining the index as [number] is because that's just how it will for example.
-- ! { "myFilter" } would print: { [1] = "myFilter" } in type would look like { [number]: string }, simpleas that.

-- Unfortunately, since wave is special when it comes to type checking, it will count any type of "inconsistency" as an potentional "error", to fix that.
-- We will not be overwriting it's definition from it's local/const "side".
local Collector = setmetatable(
	{
		-- Can only be called by meta functions.
	},
	(function(self)
		self.__index = self
		self.__metatable = "locked"

		return self
	end)({
		Collect = function(self, child, filter, callback)
			local Signal = Signal.new()

			task.spawn(function()
				for _, object in child:GetChildren() do
					if table.find(filter.ParentList, object.Name) then
						local check = (if #filter.ChildList > 0 then self:Check(object, filter) else {})
						local factor = (if rawget(filter, "Factor") then filter.Factor(object) else true)

						if factor then
							Signal:Send(object, unpack(check))
							if callback then
								callback(object, unpack(check))
							end
						end
					else
						continue
					end
				end
			end)

			return Signal
		end,
		Check = function(self, child, filter)
			local result = {}

			for _, object in child:GetChildren() do
				if table.find(filter.ChildList, object.ClassName) then
					table.insert(result, object)
				else
					continue
				end
			end

			return result
		end,
		new = function()
			return {
				ChildList = {},
				ParentList = {},
			} -- Let's pre-define, it will be overwritten anyways. Well technically, we don't have to but for the purpose of error handling we will.
		end,
	} :: collectorFunctions)
) :: collectorClass -- Instead, we will be doing it like this, just like creating an "output".
-- ? There are other ways, but I like to code everything in "one line", per say.

-- I am thinking the easy way is to just make a callback for this, but what if someone wants to make connections, we need to make a "Signal".
-- Problem is, I already have one coded, but I am not sure if it was any good.
local NPC = workspace.NPCS["Monster Quest"]
local Character = game.Players.LocalPlayer.Character
local Previous = Character:GetPivot()
local Backpack: Folder = game.Players.LocalPlayer.Backpack

local CurrentTask = nil

local Sell, Craft, Collect

Craft = function()
	if not _G.RunAutoFarm then return end

	warn("Crafting")
	local Filter = Collector.new()
	Filter.ParentList = { "Powder", "Roll", "FireCracker" }
	Filter.Factor = function(object: Part)
		return object:IsA("Tool")
	end

	local Collection = Collector:Collect(game.Players.LocalPlayer.Backpack, Filter)

	local Items = {
		["Powder"] = 0,
		["Roll"] = 0,
		["FireCracker"] = 0,
	}

	Collection:Connect(function(object: Instance)
		local Current = rawget(Items, object.Name)
		rawset(Items, object.Name, Current + 1)

		if Items.Powder >= 1 and Items.Roll >= 1 then
			game:GetService("ReplicatedStorage").CraftMenu.Craft:FireServer("Firecracker")
			Items.FireCracker += 1
		end -- Temporary

		if Items.FireCracker >= 7 then
			game:GetService("ReplicatedStorage").CraftMenu.Craft:FireServer("Brown Monster")
		end
	end)

	delay(5, Sell)
end

Collect = function()
	if not _G.RunAutoFarm then return end
	warn("Collecting")

	local Filter = Collector.new()
	Filter.ParentList = { "Powder", "Roll" }
	Filter.ChildList = { "ProximityPrompt" }
	Filter.Factor = function(object: Part)
		return object.Transparency == 0
	end

	local Collection = Collector:Collect(workspace.Interactable, Filter)

	local Found: { [number]: { Object: Part, Prompt: ProximityPrompt } } = {}
	Collection:Connect(function(object: Part, Prompt: ProximityPrompt)
		table.insert(Found, { Object = object, Prompt = Prompt })
	end)

	local Collected = 0
	for _, data in Found do
		if not data.Prompt then
			warn(data.Object, data.Prompt)
			return
		end

		local newTask = task.spawn(function()
			while task.wait() do
				if not _G.RunAutoFarm then break end

				for i = 1, 15 do
					Character:PivotTo(CFrame.new(data.Object.Position + Vector3.new(0, 4, 0)))
					task.wait()
				end

				fireproximityprompt(data.Prompt)
			end
		end)

		data.Object:GetPropertyChangedSignal("Transparency"):Once(function()
			task.cancel(newTask)
			Collected += 1
		end)
	end

	while task.wait() do
		if Collected == #Found then
			Craft()
			break
		end
	end
end

Sell = function()
	if not _G.RunAutoFarm then return end

	warn("Selling")

	local Filter = Collector.new()
	Filter.ParentList = { "Brown Monster" }
	Filter.Factor = function(object: Part)
		return object:IsA("Tool")
	end

	local Collection = Collector:Collect(Backpack, Filter)

	local Items = {
		["Brown Monster"] = 0,
	}

	local Sold = 0
	local Connection
	Connection = Backpack.ChildRemoved:Connect(function(object: Instance)
		local Current = rawget(Items, object.Name)
		if not Current then
			return
		end

		Sold += 1
	end)

	local Found = {}
	Collection:Connect(function(object: Instance)
		local Current = rawget(Items, object.Name)
		rawset(Items, object.Name, Current + 1)

		if Items["Brown Monster"] >= 1 then
			table.insert(Found, object)
		end -- Temporary
	end)

	delay(2, function()
		CurrentTask = task.spawn(function()
			while task.wait() do
				if not _G.RunAutoFarm then
					break
				end

				Character:PivotTo(CFrame.new(NPC:GetPivot().Position - Vector3.new(0, 2, 0)))

				game:GetService("ReplicatedStorage").QuestEvents.BrownMonster:FireServer("Accept job")
				game:GetService("ReplicatedStorage").QuestEvents.BrownMonster:FireServer("Redeem job")
			end
		end)

		while task.wait() do
			if not _G.RunAutoFarm then
				break
			end

			if Sold == #Found then
				task.cancel(CurrentTask)
				Character:PivotTo(Previous)
				Connection:Disconnect()
				break
			end
		end

		if _G.RunAutoFarm then
			Collect() -- Only continue the cycle if still running
		end
	end)
end

_G.EndAutoFarm = function()
	_G.RunAutoFarm = false;
	warn("EndAutoFarm")

	if CurrentTask then
		task.cancel(CurrentTask)
		CurrentTask = nil
	end

	for i = 1, 15 do
		Character:PivotTo(Previous)
		task.wait(0.1)
	end
end

local function ToggleFarm(boolean)
	warn(_G.RunAutoFarm )
	if _G["RunAutoFarm"] then
		if _G["EndAutoFarm"]  then
			_G.EndAutoFarm();
		end

		return
	else
		_G.RunAutoFarm = true
		Collect()
	end
end

if _G["RunAutoFarm"] then
	_G.EndAutoFarm();
else
	ToggleFarm()
end

--[[
    Let's test, if our type works.
    Collector:Collect() -- It certianly does, it also tells us if self is being read correctly.
]]
