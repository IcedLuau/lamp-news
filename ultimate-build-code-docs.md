# The Environment
This is in a Roblox game, called Ultimate Build ('UB'). Code runs through blocks called 'Code Blocks', which run client-sided full Luau via the FiU VM.

# What is a 'brick'?
A brick is a part, either primitive or logical. Primitive parts don't have any logic on their own; logical parts can be hooked up to the wiring system. These logical parts have inputs and outputs, letting you interact with them through APIs. All bricks have attributes, which are configuration settings.

It's also known as a 'sandboxed part', as it is, well, sandboxed.

## self/script
The code block that is running the current script is exposed as `self` or `script`. This value is a *part*, not a script. Meaning you can do stuff like this
```luau
local newPart = Instance.new("Part")
newPart.Position = script.Position -- or self.Position!
```

## The Brickset
These are all parts. Some have an adornee under them, but they are *all* parts. They can be created with the clone tool (player only), or via `Instance.new("blocktype")`. For example, you can write this simple code:

```luau
local displayPart = Instance.new("Display")

displayPart:SetAttribute("Value", "Hello! This is a display.")
```

### More on attributes
Attributes modify how parts behave and also tell our script how the environment works with parts. Some are read-only, like `Name` (the UUID you can use to index it). All bricks have the following attributes:
* `Name` - The UUID identifier (Read-only)
* `DisplayName` - A human-readable (or unreadable) name that can be written to. You can index parts in workspace with this too, but `Name`-based indexing takes priority.

### Indexing
As mentioned previously, you can index parts in the workspace by their `Name` or `DisplayName`:

```luau
local myPart = workspace["c59d6c2a-695c-4bf8-b993-46b58331a854"]
local display = myPart:GetAttribute("DisplayName")

print(display)

local myPartAgain = workspace[display] -- Less stable, as multiple parts can have the same DisplayName
```

### Wiring
These 'logical' bricks mostly have wiring inputs and outputs. These can be wired together to create a full system of logic, computing or control. Types are not strictly enforced, but booleans specifically are based off of whether Luau treats them as truthy. Outputs can be wired to inputs, obviously, and the code block API for this is:
```luau
local myPart = workspace.my.part -- hypothetical and doesn't exist

myPart.inputs.a:SetState(true) -- activate my part
myPart.inputs.a:GetState() -- 'true'

myPart.outputs.a:SetState("yay!") -- (nil unless called on self code block)
myPart.outputs.a:GetState() -- 'yay!' (or anything else)

-- EXAMPLE / DEMO
local myButton = workspace.button -- placeholder paths
local myLight = workspace.pointLight

while task.wait() do
    if myButton.outputs["Output Left"]:GetState() then
        script.BrickColor = BrickColor.random()
        myLight.inputs.a:SetState(true) -- Ideally, you'd use wiring, but here we directly set the input for demo.
    else
        script.BrickColor = BrickColor.Black()
        myLight.inputs.a:SetState(false)
    end
end

```

Below is the list of available bricks and their API:

---

`Basic` OR `Part` (Alias)
* Description: The most simple part. It is unanchored by default with weld surfaces.

---

`Code`
* Description: Code blocks that execute full Luau scripts.
* Attributes:
    * `Run` (Bool) - Determines whether it is running. Disabling ends the main thread; enabling starts the script.
    * `Code` (String) - The source code string executed by the block.

---

`Display`
* Description: A block used to display text.
* Attributes:
    * `Value` (Any) - The text (any type that can be converted via `tostring`) displayed on the screen. Not filtered directly, but the SurfaceGui will reflect filtered output.
* Wiring:
    * IN `Data` (Any) - Sets `Value` through wiring.
    * OUT `Filtered` (String) - The filtered output visible on the display.

---

`Image`
* Description: A block used to display images.
* Attributes:
    * `Image`(String) - A string determining the asset URL/ID (e.g., `'rbxassetid://14844791672'`).
* Wiring:
    * IN `Image` (string) - Sets `Image` through wiring.

---

`Variable`
* Description: Simple block used to store wiring values.
* Attributes:
    * `Value` (Any) - The stored value.
* Wiring:
    * IN `Overwrite` (Any) - Overwrites the stored value through wiring. Accepts all inputs (including `false`).
    * OUT `Read` (Any) - Outputs the currently stored value.

---

`Button`
* Description: A block that outputs signals based on mouse click interactions.
* Attributes:
    * `Clickability` (StringEnum) - Undocumented.
    * `Momentary` (Bool) - If `true`, outputs as long as held. If `false`, outputs a one-shot signal based on `Time`.
    * `Time`  (Number) - Duration of the output signal when `Momentary` is false.
    * `Output Left` (Bool) - Mirrors the `Left Click` wiring output; used for linking.
    * `Output Right` (Bool) - Mirrors the `Right Click` wiring output; used for linking.
    * `Last Clicked` (String) - Username of the last player to click the button.
* Wiring:
    * OUT `Left Click` (Bool) - Outputs when a click is triggered/held via Left Mouse Button (LMB).
    * OUT `Right Click` (Bool) - Outputs when a click is triggered/held via Right Mouse Button (RMB).

---

`Switch`
* Description: A block that toggles state when clicked or triggered via wiring.
* Attributes:
    * `Clickability`(StringEnum) - Undocumented.
    * `Toggle` (Bool) - Forceful state override (`true`/`false`).
* Wiring:
    * IN `a` (Bool) - Toggles state when set to `true`.
    * OUT `a` (Bool) - Outputs the current toggle state of the switch.

---

`PointLight`, `SurfaceLight`, `SpotLight` (Seperate Types)
* Description: These are bricks with their corresponding [light](https://create.roblox.com/docs/en-us/reference/engine/classes/Light) under them. Color is determined by the Color property, not an attribute
* Attributes:
    * `Angle` (Number) - Only for surface and spot lights. RBLX Light.Angle
    * `Face` (StringEnum NormalId) - Which face it appears on. Only surface and spot lights. RBLX Light.Face
    * `Shadows` (Bool) - Determines whether shadows are enabled. RBLX Light.Shadows
    * `Range` (Bool) - The light range in studs. Capped to 100 even though engine supports 120.
* Wiring:
    * IN `a` (Bool) - Determines whether the light is enabled.

---

`Clock`
* Description: A block that outputs equivalent to [os.clock()](https://create.roblox.com/docs/reference/engine/libraries/os#clock)
* Attributes:
    * `Continous` (Bool) - If false, will only fire when input is true. If true, it updates every frame.
    * `Output` (Number) - Same as wiring output.
* Wiring:
    * OUT `Output`(Number) - Output of os.clock() when it fires
    * IN `Input` (Bool) - If continous is false, it will update when set to true once. Otherwise does nothing

---

`PID`
* Description: A block that functions as a [PID controller](https://en.wikipedia.org/wiki/PID_controller)
* Attributes AND Inputs (Matching):
    * `P` (Number) - Proportional
    * `I` (Number) - Integral
    * `D` (Number) - Derivative
    * `Process` (Number) - The current state
    * `Goal` (Number) - The target state
    * `Output` (Number) - The value used to get to that state

---

`Math`
* Description: A block that can do various math operations
* Attributes:
    * `Input 1` (Mathable) - Same as wiring input.
    * `Input 2` (Mathable) - Same as wiring input.
    * `Math` (StringEnum Math) - The operation type, see extra.
* Wiring:
    * `Input 1` (Mathable) - Left operand
    * `Input 2` (Mathable) - Right operand (used conditionally)
* Extra:
    * Options for math input: + - ÷ * % ^ √ log abs sign
    * Mathable: Vector3, CFrame, Number


---

`VectorMath`
* Description: A block for doing vector specific math/dealing with vectors
* Wiring:
    * IN `X` - X Component (used conditionally)
    * IN `Y` - Y Component (used conditionally)
    * IN `Z` - Z Component (used conditionally)
    * IN `Input 1` - Left operand/Main vector to use (used conditionally)
    * IN `Input 2` - Right operand (used conditionally)
    * OUT `Output` (Number/Vector3)
* Attributes:
    * X, Y, Z, Input 1, Input 2 (Wiring Mirrors)
    * Vector (StringEnum Vector)
* Extra
    * Options for Vector input:
        * dot
        * cross
        * unit
        * magnitude
        * angle
        * fuzz
        * vector3 - New Vector
        * x - get x component of input 1
        * y
        * z

---

`Trigonometry`
Description: Block that does [trigonometry](https://en.wikipedia.org/wiki/Trigonometry) functions
* Attributes:
    * `Output` - Output Mirror
    * Trigonometry - Type of trig function to use
    * Radians, X, Y - Input mirrors
* Wiring:
    * IN `Radians` - Radians, used for functions that use angles, such as sin
    * IN `X` - First input to operations that use 2 values, like atan
    * IN `Y` - Same, but for the second input
    * OUT `Output` 

---

`Functions`

* Description: A block used to execute standard scalar math functions (such as rounding, clamping, random number generation, or noise) on numbers and convert types.
* Attributes:
    * `Function` (StringEnum) - The mathematical function to run (`deg`, `rad`, `clamp`, `floor`, `ceil`, `round`, `noise`, `random`, `tonumber`).
    * `Radians`, `Degrees`, `Number`, `Min`, `Max`, `X`, `Y`, `Apply` (Wiring Mirrors) - Corresponding input attribute values for the active function.


* Wiring:
    * IN `Radians` (Number) - Radians value for `deg` conversion.
    * IN `Degrees` (Number) - Degrees value for `rad` conversion.
    * IN `Number` (Any / Number) - Input number for standard functions (`floor`, `ceil`, `round`, `clamp`, `tonumber`).
    * IN `Min` (Number) - Lower boundary for `clamp` or `random`.
    * IN `Max` (Number) - Upper boundary for `clamp` or `random`.
    * IN `X` (Number) - X coordinate for `noise`.
    * IN `Y` (Number) - Y coordinate for `noise`.
    * IN `Apply` (Bool / Any) - Trigger/pulse input to recalculate or apply certain functions (e.g., `random`).
    * OUT `Output` (Number) - Outputs the calculated numerical result.

# More Script Globals
Our environment provides some more script globals. Some being bulk changes, as all changes have to go through a RemoteEvent, these functions make that more efficient.

This is not an exhaustive list, as these are extremely gatekept

## ChangeProperties
ChangeProperties is a global function to change the part properties of a brick in bulk. Currently, you can only make 512 changes in one call.
It takes in a array (<512) of arrays (3 values), and uses this syntax for the changes list.
```luau
local PropertyChanges = { -- An array of arrays
    {Brick, Property, Value}, -- The sandbox-brick, the Property name (string), and the Value (depends)
}

ChangeProperties(PropertyChanges)
```

## ChangeAttributes
Similar to ChangeProperties, ChangeAttributes allows you to mass edit the attributes of up to 512 parts in one call.
```luau
local AttributeChanges = {
    {Brick, AttributeIdentifier, Value} -- The sandbox-brick, the Attribute id (string), and the Value (depends)
}

ChangeAttributes(AttributeChanges)
```

## Wire
The wire function lets you wire two opposite kind wiring references.
```luau
local Variable0, Variable1 = workspace.Variable0, workspace.Variable1
local wireColorSequence = ColorSequence.new(Color3.new(.5,.5,.5)) -- Colors, optional param.

Wire(Variable0.inputs.Overwrite, Variable1.outputs.Read, wireColorSequence) -- works in any order for i/o
-- InputRef, OutputRef, ColorSequence (optional)
```

## SnipWire
SnipWire allows you to snap the wire between two wiring references **if the wire exists**
```luau
-- Destroy a wire that exists between Variable0 and Variable1.
local Variable0, Variable1 = workspace.Variable0, workspace.Variable1

SnipWire(Variable0.inputs.Overwrite, Variable1.outputs.Read)
-- InputRef, OutputRef
```

## Instance.custom
This one lets you create regular, non-sandbox instances instead of the bricks. But this will result in them only existing client sided
```luau
local pointBrick = Instance.new("PointLight") -- Sandboxed part with real PointLight under
local pointUnsandbox = Instance.custom("PointLight") -- A single, real (client sided) PointLight
```
