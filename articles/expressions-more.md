# More Game Expressions

These are the game's expression functions that have no example in the game's own balance data. They are split from [Game Expressions](expressions.md) so that page stays short. That page also explains how expressions are written, the `$`, `@`, `%` and `#` shortcuts, and how to read the columns below.

Everything here was written from reading the game's code and has not been tested in-game. Parameter names and types come from how the game reads them, so a name such as `value` can mean a count or an amount. A parameter marked `?` is optional. Most functions return `true` when they run, and the descriptions mention a return value only when it matters.

The Needs column lists what a function reads from the place it runs. One that needs a world object or a combat target has nothing to work on when a potion runs it, so expect those to do nothing there. A function that takes the id or custom tag of its target finds it by itself, so it needs nothing from the place it runs. A blank Needs means the code reads nothing from the place it runs.

The last column is a real expression from the game's own data. A blank means no expression in the game's balance data calls it, though the game's story scripts may.

## Parameters

Each of these is what an `=`, `*=` or `/=` shortcut turns into. For example `%name*=2` becomes `MultiplyWorldPar("name", 2)`. The player has no multiply or divide function, so `$name*=2` is not possible. The divide functions multiply by one over the value, and division by zero is not guarded against.

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `SetWorldPar` | id: string, value: float | Sets a world parameter and returns the new value. This is `%name=value`. | | |
| `MultiplyWorldPar` | id: string, value: float | Multiplies a world parameter by the value and returns the new value. This is `%name*=value`. | | |
| `DivideWorldPar` | id: string, num: float | Divides a world parameter by the value and returns the new value. This is `%name/=value`. | | |
| `AddWGOPar` | id: string, value: float | Adds to a parameter of the world object. Returns `false` when there is no world object. This is `@name+=value`. | a world object | |
| `DecWGOPar` | id: string, num: float | Subtracts from a parameter of the world object. Returns `false` when there is no world object. This is `@name-=value`. | a world object | |
| `MultiplyWGOPar` | id: string, value: float | Multiplies a parameter of the world object. Returns `false` when there is no world object. This is `@name*=value`. | a world object | |
| `DivideWGOPar` | id: string, num: float | Divides a parameter of the world object. Returns `false` when there is no world object. This is `@name/=value`. | a world object | |
| `AddWorkerPar` | type: string, value: float | Adds to a parameter of the worker at the world object. The worker is whoever is working there, or the player when nobody is. Returns `false` when there is no world object. This is `#name+=value`. | a world object | |
| `DecWorkerPar` | type: string, num: float | Subtracts from a parameter of the worker. Same worker as `AddWorkerPar`. This is `#name-=value`. | a world object | |
| `MultiplyWorkerPar` | type: string, value: float | Multiplies a parameter of the worker. Same worker as `AddWorkerPar`. This is `#name*=value`. | a world object | |
| `DivideWorkerPar` | type: string, num: float | Divides a parameter of the worker. Same worker as `AddWorkerPar`. This is `#name/=value`. | a world object | |
| `SetWorkerPar` | type: string, value: float | Sets a parameter of the worker. Same worker as `AddWorkerPar`. This is `#name=value`. | a world object | |

## Player and Items

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `UseItem` | itemId: string | Uses one of the player's items as if the player used it from the inventory. Its on-use effects, sound, and on-use expressions run, and one is used up unless the item stays on use. Does nothing for an item that cannot be used. The player is expected to be carrying the item. | | |
| `GetEquippedShovelIcon` | | Returns the icon id of the shovel on the player's tool belt, or `i_shovel_0` when there is none. | | |
| `IncreaseOverheadStackLimit` | num: int | Raises how many things the player can carry overhead at once by that amount. The value must be above 0, otherwise it logs an error and returns `false`. | | |
| `SetOverheadStackLimit` | num: int | Sets how many things the player can carry overhead at once. The value must be 1 or more, otherwise it logs an error and returns `false`. | | |
| `PlayerOvrhdItemIsZombie()` | | True when an item the player carries overhead is a zombie. The game registers this under a name that already includes the parentheses. Written as `PlayerOvrhdItemIsZombie()` it matches, but going by the expression parser's code, whatever follows it is read as its argument and fails to parse, so use it alone or as the last part of an expression. | | |
| `AddItemToCrowNest` | itemId: string, count: int | Puts items in the crow nest's inventory. Returns `false` when the id is unknown, the count is not above 0, the nest is not in the world, or the items do not fit. | | |

## World Object Inventory and Crafting

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `CanAddItemToWgo` | itemId: string, count: int | True when the world object's inventory has room for that many of the item. | a world object | |
| `GetItemByGrp` | item: string | Returns the first item in the world object's inventory that belongs to that item group, or `false` when none does. | a world object | |
| `GetItemById` | text: string | Returns the first item in the world object's inventory with that id, or `false` when there is none. | a world object | |
| `IsNotCraftingNow` | | True when the world object has no craft running. False when there is no world object. | a world object | |
| `CrIn_SummQualGrd` | item: string | Adds up the quality of the specific items the last craft queued at the world object requires, counting only those that belong to the given item group. A requirement given as a whole item group is not counted. Returns 0 when there is no world object. | a world object | |
| `CrIn_SummQualGrd10` | item: string | The same total as `CrIn_SummQualGrd`, divided by 10. | a world object | |
| `WorkerIsPlayer` | | Returns 1 when the world object's worker is the player and 0 otherwise. The worker is whoever is working there, or the player when nobody is. Returns 0 when there is no world object. | a world object | |

## Unlocks and Customization

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `UnlockCustomizationPart` | id: string, type: int | Unlocks a look for the player. The type is 0 for hair, 1 for beard, 2 for body, 3 for arms and 4 for head. Unlocking a body also unlocks the arms with the matching id (`bdy_` becomes `arm_`). Returns `false` when the id is empty or the part was already unlocked. | | |
| `UnlockCustomizationColor` | type: int, paletteName: string, partSkinId: int | Unlocks a color palette for a part of the player. The type is 0 for the head (`Hed`) and 1 to 3 for the three body color slots (`Bdy1`, `Bdy2`, `Bdy3`). Returns `false` when the palette name is not valid for that part or was already unlocked. | | |
| `RemoveTech` | id: string | Takes a tech back out of the player's unlocked techs. | | |
| `UnlockTownBuilding` | buildingId: string | Unlocks a town building. Town buildings marked as needing an unlock only count once they are unlocked, and `LockTownBuilding` can still block them. | | |
| `IsBuildPlaceUnlocked` | text: string | True when that build place is in the list of unlocked buildings, the one `UnlockBuilding` adds to. An empty id counts as unlocked. | | |
| `AddToKnownMapZones` | text: string | Marks a map zone as known to the player. Returns `false` when the id is empty or the zone was already known. | | |

## Quests

These match `AwaitQuest`, `CompleteQuest` and `ChangeQuestHiddenState` on the main page.

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `AwaitQuestDemoDependent` | id: string, arg2: string, delayTime?: float | Does the same as `AwaitQuest`. The second value is read and then ignored. | | |
| `CompleteQuestDemoDependent` | id: string, arg2: string, delayTime?: float | Does the same as `CompleteQuest`. The second value is read and then ignored. | | |
| `ChangeQuestUnknownState` | id: string, flag: bool | Sets whether the quest log shows a quest as unknown. Passing `true` clears the unknown flag and `false` sets it. | | |

## Town, Vendors, and Reputation

The three `WZQual` examples are from the display text of the game's world zones, where they sit inside `LE{...}`. Those places run them for a zone, so they work there without anything extra.

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `AddRepWGO` | id: string, value: int | Changes the player's reputation with a character, like `AddRep`, but takes the character's world object id. The game looks up the reputation name in that world object's definition. | | |
| `SetRepWGO` | id: string, value: int | Sets the player's reputation with a character, like `SetRep`, but takes the character's world object id. | | |
| `AddOrderSlot` | | Adds an empty vendor order slot. Returns `false` once there are already 5. | | |
| `RepairHousesById` | text: string, stageIdx: int | Repairs the repairable town buildings with that world static object id. The stage is the repair stage to fix, or -1 for every stage. The code reads the second value whenever any value is given, so always pass both. Returns `false` when the id is empty or no building has it. | | |
| `RepairHousesByTag` | text: string, stageIdx: int | The same as `RepairHousesById`, but the first value is a custom tag. | | |
| `WZQual` | text: string | Returns the total quality of the world zone with that id. Logs an error and returns 0 when the id is empty or unknown. | | `WZQual("morgue")` |
| `WZQualPowerSource` | | Returns the zone's total quality counted from conveyor power sources. Returns 0 when there is no zone. | a zone | `WZQualPowerSource()` |
| `WZQualConveyorCells` | | Returns the zone's total quality counted from conveyor cells. Returns 0 when there is no zone. | a zone | `WZQualConveyorCells()` |

## Windows and Tutorials

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `OpenAutopsyWindow` | | Opens the autopsy window for the world object. Returns `false` when there is no world object. | a world object | |
| `OpenAutopsyWindow_CB` | | Opens the autopsy window for the world object with the id `autopsy_table_1`, which it finds by itself. When the expression was started with a close callback, the callback runs when the window closes. Returns nothing when the table is not in the world. | | |
| `OpenResurrectionWindow` | | Opens the resurrection window for the world object. Returns `false` when there is no world object. | a world object | |
| `OpenGardenWindow` | | Opens the garden bed window for the world object. Returns `false` when there is no world object. | a world object | |
| `OpenDemoEndWindow` | | Opens the demo end window. | | |
| `ShowTechUnlock` | text: string | Opens the tech window with the "new tech unlocked" title and an OK button. The tech must already be unlocked and be a common tech, otherwise it logs an error and returns `false`. | | |
| `IsShownUIWindow` | arg1: string | True when the window with that name is open. It only looks at windows the game can operate with custom input, so other windows come back false. | | |
| `AttachArrowTutorial` | wgoCustomTag: string | Points the tutorial arrow at the world object with that custom tag. | | |
| `UnAttachArrowTutorial` | | Removes the tutorial arrow. | | |
| `EnableTutorialMode` | text...: string | Turns tutorial mode on. While it is on, the player can only interact with world objects on an allowed list. Each value is the custom tag of a world object to put on that list, and any number can be given. | | |
| `DisableTutorialMode` | | Turns tutorial mode off and empties the allowed list. | | |
| `TutorialModeAddToExclude` | text...: string | Adds world objects (by custom tag, any number) to the tutorial mode allowed list. Does nothing while tutorial mode is off. | | |
| `TutorialModeRemoveFromExclude` | text...: string | Takes world objects (by custom tag, any number) off the tutorial mode allowed list. Does nothing while tutorial mode is off. | | |

## World Objects

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `SpawnWgo` | wgoId: string, x: float, y: float, z: float, gameSceneId: string, customTag: string | Creates a world object at that position in that scene and gives it the custom tag. | | |
| `SpawnConveyorWgo` | wgoId: string, x: float, y: float, z: float, gameSceneId: string, customTag: string | The same as `SpawnWgo`, and when the new object is a conveyor it also connects it to the conveyors around it. | | |
| `AddPerkToWgoByTag` | text: string, id: string | Gives a perk to the world object with that custom tag. Logs an error and returns `false` when there is no such object. | | |
| `RemovePerkFromWgoByTag` | text: string, id: string | Takes a perk off the world object with that custom tag. Logs an error and returns `false` when there is no such object. | | |
| `RemovePerkWgoSelf` | id: string | Takes a perk off the world object the expression runs on. | a world object | |
| `RemoveAllPerksWgoSelf` | | Takes every perk off the world object the expression runs on. | a world object | |
| `SetWgoGameResInt` | text: string, id: string, value: int | Sets a parameter on the world object with that custom tag, like `SetWGOParByTag` but with a whole number. Logs an error and returns `false` when there is no such object. | | |
| `EnableGDPointByTag` | customTag: string | Turns on every point with that custom tag, where `EnableGDPointById` goes by id. | | |
| `DisableGDPointByTag` | customTag: string | Turns off every point with that custom tag. | | |
| `TeleportToGD` | gdPointId: string, environmentPreset?: string, soundOnTeleport?: string | Teleports the player to the point with that id. The optional values name the environment preset to switch to and a sound to play, like `TeleportTo`. | | |

## Fights

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `KillerIsPlayer` | | Returns 1 when the attacker is the player and 0 otherwise. | a combat target (the attacker) | |

## NPC Simulation

These work on the game's NPC life simulation, which sends groups of townsfolk to points of interest and back home. `NPCSim_RollActivityForGroup` and `NPCSim_SendGroupHome` are on the main page.

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `NPCSim_AddWgoToGroupFromBalance` | wgoId: string | Puts the world object with that id into the group named in its own definition. It leaves any group it was in first. | | |
| `NPCSim_AddWgoToGroup` | wgoId: string, groupId: string | Puts the world object with that id into that group. It leaves any group it was in first. | | |
| `NPCSim_RemoveWgoFromGroup` | wgoId: string | Takes the world object with that id out of its group. | | |
| `NPCSim_UnlockPointOfInterest` | pointId: string | Turns a point of interest on, so the simulation can send characters there. | | |
| `NPCSim_LockPointOfInterest` | pointId: string | Turns a point of interest off. | | |
| `NPCSim_SendAllGroupsHome` | | Does `NPCSim_SendGroupHome` for every group. | | |
| `NPCSim_RollActivityForAllGroups` | | Does `NPCSim_RollActivityForGroup` for every group. | | |
| `NPCSim_ForceAllGroupsTeleportHome` | | Stops every group member that has a home point and moves them straight to it, with no walking. | | |
| `NPCSim_ForceGroupTeleportHome` | groupId: string | Stops a group's members and moves them straight to their home point, with no walking. As the game's code is written, it stops after it moves the first member that has a home point, so the rest stay where they are. | | |

## Weather, Light, and Effects

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `SetWeatherState` | stateName: string | Switches the weather to the state with that name and marks it as forced. Logs a warning and does nothing when the name is unknown. | | |
| `SetWeatherComponent` | componentName: string, isActive: bool | Fades a weather component in or out and remembers it in the saved weather. Logs a warning and does nothing when the name is unknown. | | |
| `SetPostProcessProfile` | profileName: string | Applies the camera look with that name. An empty name goes back to the default look. Does nothing when the platform has post-processing turned off. | | |
| `ResetPostProcessProfile` | | Goes back to the camera's default look. | | |
| `SetLightEnvironmentOverride` | presetName: string | Applies a light environment preset on top of the normal lighting. An empty name clears the override. Logs an error when the name is unknown. | | |
| `ResetLightEnvironmentOverride` | | Clears the light environment override. | | |

## Navigation

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `ScanGraph` | arg1: string | Rescans a walking graph by name. Returns `false` when the name is not a graph or the graph is not a recast graph (the kind the game can rescan). The names are PortArea, DriedGatewayArea, GdPointGraph, PlayerGraph, DevPlayground, Church, ZombieFighters, WZYard, TestZombieZone, VillageForestArea, RuinedTemple, Fighting_Recast, Carrier, Sawmill, Mine, SandClay, Garden, Vineyard, Conveyors, VineyardBasement, BasementWriting, AlchemyLab, and PlayerHouse. | | |
| `Debug_ScanGraphWithDelay` | arg1: string | The same as `ScanGraph`, but it waits one second first. | | |
