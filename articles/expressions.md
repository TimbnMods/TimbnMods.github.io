# Game Expressions

`StartExpressions` and `EndExpressions` on a potion's buff, and the `OnStartExpression` and `OnDoneExpression` calls on a quest stage, take strings written in the game's own expression language. So do `OfferWhenExpression` on a quest and the expression forms of `RunExpression` and `When` in a talk. Each string calls one of the game's functions, such as `DropItem("stick", 5)`. Ids are the game's balance ids, so check the spelling against the game's data.

An expression is written inside a C# string, so the quotes around its arguments must be escaped with a backslash.

```csharp
StartExpressions = ["DropItem(\"stick\", 5)"],
```

Only the functions with a known description or a real example from the game's own data are listed here. The rest are on [More Game Expressions](expressions-more.md). Parameter names and types come from how the game reads them, so a name such as `value` can mean a count or an amount. A parameter marked `?` is optional. Descriptions are based on the game's code.

The Needs column lists what a function reads from the place it runs. One that needs a world object, an item, a combat target, or a zone has nothing to work on when a potion runs it, so expect those to do nothing there. In a talk's `RunExpression` and `When`, the world object is the NPC being talked to, while `OfferWhenExpression` runs with nothing. A function that takes the id or custom tag of its target finds it by itself, so it needs nothing from the place it runs. A few of them (such as `WGOFireEvent` and `SetWgoInteractionState`) also make that object the current world object for the rest of the expression. None of these have been run from a potion.

The last column is a real expression from the game's own data, the shortest one that uses the function. A blank means no expression in the game's balance data calls it, though the game's story scripts may.

## Logic

The game provides an if statement for use inside expressions.

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `If2` | arg1: bool, arg2: any | Runs the second part only when the first is true. Does nothing otherwise. | | `If2($perk_scientist==1, AddPPar("tech_red", 2))` |
| `True` | | Always true. | | |
| `False` | | Always false. | | `False()` |

The expression library the game uses also brings its usual math and string functions, such as `Abs`, `Min`, `Max`, `Round`, `Floor`, `Ceiling`, `Pow`, `Sqrt`, `Random`, `Contains` and `Length`, and `If(condition, ifTrue, ifFalse)`, which picks one of two values. The game's own data uses `If` and `Pow`. Function names are case sensitive.

## Parameters and Shortcuts

Instead of calling a parameter function, an expression can write a one-character prefix followed by the parameter's name. The game rewrites it into the matching function before running it.

### `$name`

Reads a player resource, the same as `PPar("name")`. The player's money, tech points, and many flags are player resources. Some perks set one when they are added, so `$perk_scientist==1` is true for a player who has that perk. That depends on the perk's data, so check that the perk sets a resource with that name.

### `@name`

Reads a parameter of the world object the expression runs on, the same as `WGOPar("name")`. It needs a world object.

### `%name`

Reads a world parameter, the same as `WorldPar("name")`. These are values saved with the world.

### `#name`

Reads a parameter of a worker, the same as `WorkerPar("name")`. It needs a world object.

Each prefix can also change the value. These four are real expressions from the game's data.

```text
$cur_bodies_count+=1
$g_garden_lvl=1
%graveyard_module_alley_1+=1
If2(@reached_silver == 0, AddInspiration("insp_surgical_mystake", 1))
```

`=` sets the value, and `+=` and `-=` add to it and subtract from it. `*=` and `/=` also exist for `@`, `%` and `#`, but not for `$`, since the game has no player multiply or divide function, so `$name*=2` does not work. Use `==` to compare, as in `$perk_scientist==1`.

## Player

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `AddInspiration` | inspirationId: string, counterToAdd: int | | | `AddInspiration("mason", 1)` |
| `AddPerk` | id: string | Gives the player a perk. | | `AddPerk("buff_beer")` |
| `AddPerkWgoSelf` | id: string | | a world object | `AddPerkWgoSelf("perk_fertilize_peat")` |
| `AddPlayerHP` | num: int | Heals the player when the amount is positive and damages them when it is negative. | | `AddPlayerHP(20)` |
| `AddTalentValue` | talentId: string, value: int | Adds talent progress. It's the most common expression in the game's own perks. | | `AddTalentValue("talent_red",2)` |
| `IsPlayerControlsEnabled` | | | | `IsPlayerControlsEnabled()` |
| `RemovePerk` | id: string, silent: bool | Takes a perk away. | | `RemovePerk("buff_damage_1", 0)` |
| `SetPlayerDamageImmunity` | isImmuneToDamage: bool | | | `SetPlayerDamageImmunity(true)` |
| `SetTimeWithoutSleep` | timeWithoutSleep: float | Sets how long the player has gone without sleep. | | |
| `TrySetOrRemoveResurrectionDebuff` | | | | `TrySetOrRemoveResurrectionDebuff()` |

## Items and Inventory

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `AddItemToWgoInv` | id: string, value: int | | a world object | `AddItemToWgoInv("water",1)` |
| `CanPlayerFertilize` | | | a world object | `CanPlayerFertilize()` |
| `DropBurialRewards` | | | a world object | `DropBurialRewards()` |
| `DropHappiness` | h: int | | | `DropHappiness(5)` |
| `DropItem` | id: string, value: int | Drops items on the ground in front of the player. | | `DropItem("gem", 8)` |
| `DropItemByGroup` | item: string | | a world object | `DropItemByGroup("body")` |
| `HasItemInInv` | itemId: string, count: int | | a world object | `NoOvhdItem() && HasItemInInv("wood", 1)` |
| `HasItemInInvByGroup` | item: string | | a world object | `NoOvhdItem() && HasItemInInvByGroup("corpse")` |
| `HasOverheadWithSkulls` | skullsName: string, skullsCount: int | True when the items inside the overhead item the player carries add up to at least that many skulls. The name is `red` or `white`. | | `PlayerHasOvrhdAndCanInsert() && HasPlayerOvrhdItemByGrp("zombie") && HasOverheadWithSkulls("red","8")` |
| `HasPlayerItemEquip` | itemId: string, count: int | | | `HasPlayerItemEquip("pickaxe_0", 1)` |
| `HasPlayerItemInInv` | itemId: string, count: int | | | `HasPlayerItemInInv("note_lab_tower", 1)` |
| `HasPlayerOvrhdItem` | overheadItemId: string | | | `HasPlayerOvrhdItem("body_corpse")` |
| `HasPlayerOvrhdItemByGrp` | overheadItemGroup: string | | | `PlayerHasOvrhdAndCanInsert() && HasPlayerOvrhdItemByGrp("zombie") && HasOverheadWithSkulls("red","8")` |
| `IncreasePlayerInventory` | num: int | Adds inventory slots. The value must be above 0. | | `IncreasePlayerInventory(1)` |
| `InsertOvrhdItem` | | | a world object | `InsertOvrhdItem()` |
| `InsertOvrhdItemWithSkulls` | | | a world object | `InsertOvrhdItemWithSkulls()` |
| `NoOvhdItem` | | | | `NoOvhdItem() && HasItemInInv("wood", 1)` |
| `PlayerHasIntrctSeed` | | | | `PlayerHasIntrctSeed()` |
| `PlayerHasOvrhdAndCanInsert` | | | a world object | `PlayerHasOvrhdAndCanInsert()` |
| `ReducePlayerInventory` | num: int | | | `ReducePlayerInventory(1)` |
| `RemoveItem` | itemId: string | | | `RemoveItem("intro_prison_key")` |
| `RemoveItemByGroup` | groupId: string, num: int | | a world object | `RemoveItemByGroup("corpse",1)` |
| `TakeOvrhdItem` | itemId: string | | a world object | `TakeOvrhdItem("wood")` |
| `TakeOvrhdItemByGroup` | groupId: string | | a world object | `TakeOvrhdItemByGroup("corpse")` |

## Unlocks, Tech, and Crafts

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `AddTech` | id: string | | | `AddTech("lacquer")` |
| `BlackListPhrase` | text: string | | | `BlackListPhrase("fight_B1_1")` |
| `ChangeCharTabLockedState` | page: int, arg2: bool | | | `ChangeCharTabLockedState(3, true)` |
| `CollectedCount` | | | an item | `AddInspiration("insp_keys_collect", CollectedCount())` |
| `HasCraft` | | | a world object | `HasCraft()` |
| `IsBuildingLocked` | text: string | | | `If2(IsBuildingLocked("garden_compost_pile_s"), UnlockBuilding("garden_compost_pile_upgrade_s"))` |
| `IsTechUnlocked` | text: string | | | `If2(IsTechUnlocked("fertilizer_3"), UnlockBuilding("garden_compost_pile_upgrade_s"))` |
| `LockBuilding` | text: string | | | `LockBuilding("millstone_s")` |
| `LockTechTab` | techTreeTab: int | | | `LockTechTab(0)` |
| `RemovePhraseFromBlackList` | text: string | | | `RemovePhraseFromBlackList("19_base_ceremony_ready_1")` |
| `RevealTalentLevelUp` | talentLevelUpId: string | | | `RevealTalentLevelUp("talent_dark_lust_envy")` |
| `RevealTech` | techId: string | | | `RevealTech("dynamite")` |
| `StartCraft` | craftId: string | | a world object | `StartCraft("flower_night")` |
| `StartCraftOnWgo` | wgoId: string, craftId: string | Adds a craft to the queue of the world object with that id. | | `StartCraftOnWgo("well_garden_2","well_garden_water_respawn")` |
| `SuperUnlockBodyCustomization` | id: string | | an item | `SuperUnlockBodyCustomization("bdy_9008")` |
| `UnlockBuilding` | text: string | | | `UnlockBuilding("home_attic_s")` |
| `UnlockCraft` | craftId: string | Unlocks a craft. | | `UnlockCraft("jewelry")` |
| `UnlockHudDaySprite` | text: string | | | `UnlockHudDaySprite("day_envy")` |
| `UnlockOrgan` | itemType: int | | | `UnlockOrgan(35)` |
| `UnlockPhrase` | text: string | Unlocks a phrase in the player's knowledge. | | `UnlockPhrase("fight_B3_1")` |
| `UnlockTechTab` | techTreeTab: int | | | `UnlockTechTab(2)` |

## Quests

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `AwaitQuest` | id: string, delayTime?: float | | | `AwaitQuest("4_intro_base_home")` |
| `CancelQuest` | id: string | | | `CancelQuest("127_palace_battle_lose")` |
| `ChangeQuestHiddenState` | id: string, flag: bool | Sets whether the quest log shows a quest as hidden. Passing `true` clears the hidden flag and `false` sets it. | | `ChangeQuestHiddenState("130_town_wishes_portal", 1)` |
| `CompleteQuest` | id: string, delayTime?: float | | | `CompleteQuest("insp_bridges")` |
| `IsQuestCompleted` | id: string | | | `If2(IsQuestCompleted("21_base_donkey_nun"), StartQuest("33_base_complain_go"))` |
| `StartQuest` | id: string, delayTime?: float | Starts a quest. The second value is an optional delay. | | `StartQuest("repair_1201")` |
| `StartQuestDemoDependent` | id: string, arg2: string, delayTime?: float | | | `StartQuestDemoDependent("15_guards_fightback_win","15_guards_fightback_win_demo")` |
| `TryOpenQuestTree` | | | | `ShowTutorialWindowWithCallback("tutorial_hdr_questslog", "TryOpenQuestTree()")` |

## Town, Vendors, and Reputation

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `AddCustomRectToWZ` | id: string, num: int, num2: int | | a world object | `AddCustomRectToWZ("garden", 9, 15)` |
| `AddMoneyToVendor` | vendorId: string, money: int | Gives a vendor money. | | |
| `AddRep` | repRes: string, value: int | Changes the player's reputation with an NPC. | | `AddRep("nun_REP", 10)` |
| `AddTownQuality` | num: int | | | `AddTownQuality(5)` |
| `ClearTownPalettes` | | | | `ClearTownPalettes()` |
| `CreateTownBuilding` | | | a world object | `CreateTownBuilding()` |
| `DestroySignboard` | | | a world object | `DestroySignboard()` |
| `GetChurchObjectQuality` | | | a world object | `GetChurchObjectQuality()` |
| `LockTownBuilding` | buildingId: string | | | `LockTownBuilding("Fish_t1")` |
| `LvlUpVendor` | vendorId: string | | | `LvlUpVendor("npc_herm")` |
| `RemoveCustomRectFromWZ` | id: string, num: int, num2: int | | a world object | `RemoveCustomRectFromWZ("graveyard", 12, 12)` |
| `RepairTownCluster` | | | a world object | `RepairTownCluster()` |
| `ResetVendors` | | | | `ResetVendors()` |
| `SetRep` | repRes: string, value: int | Sets the player's reputation with an NPC, where AddRep changes it. | | |
| `TryFinishPanicReductionMachineCraft` | | | | `If2($panic_reduction_duration <= 0, TryFinishPanicReductionMachineCraft())` |
| `WZAddAdditionalQuality` | text: string, num: int | | | `WZAddAdditionalQuality("graveyard",10)` |

## Windows, Tutorials, and Messages

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `AchTriggerCountable` | trigger: string, countProgress: int | | | `AchTriggerCountable("pot_done", 1)` |
| `AchUnlock` | achievementId: string | | | `AchUnlock("ach_gunter")` |
| `CloseUIWindow` | arg1: string | | | `CloseUIWindow("UICraftWindow")` |
| `OpenCustomizationWindow` | | | | `OpenCustomizationWindow()` |
| `OpenNotesWindow` | id: string | | | `OpenNotesWindow("inq_note_1")` |
| `OpenSurveyResultWindow` | text: string | | a world object | `OpenSurveyResultWindow("hop")` |
| `ShowTextNotification` | locale: string | Shows a game notification with the text for that locale key. | | |
| `ShowTutorialWindow` | arg1: string | | | `ShowTutorialWindow("tut_town_hdr")` |
| `ShowTutorialWindowWithCallback` | pageId: string, exprOnClose: string | | | `ShowTutorialWindowWithCallback("tut_tech_hdr", "TryOpenTechTree()")` |
| `TryOpenTechTree` | | | | `ShowTutorialWindowWithCallback("tut_tech_hdr", "TryOpenTechTree()")` |

## Parameters

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `AddPPar` | type: string, value: float | Adds to a player resource and returns the new total. | | `AddPPar("church_best",1)` |
| `AddWorldPar` | id: string, value: float | | | `AddWorldPar("graveyard_module_mixed_2", 1)` |
| `AttackerPar` | resId: string | | a combat target | `100+50*AttackerPar("zperk_undead_resistance")` |
| `DecPPar` | type: string, num: float | | | `DecPPar("cur_bodies_count", 1)` |
| `DecWorldPar` | id: string, num: float | | | `DecWorldPar("graveyard_module_mixed_3", 1)` |
| `PPar` | type: string | | | `PPar("wz_town")>=50` |
| `SetPPar` | type: string, value: float | | | `SetPPar("clearing_A", 0)` |
| `SetWGOPar` | id: string, value: float | | a world object | `SetWGOPar("is_opened", 0)` |
| `WGOPar` | id: string | | a world object | `WGOPar("seed_b")` |
| `WorkerPar` | type: string | | a world object | `2+1*WorkerPar("perk_cook")` |
| `WorldPar` | id: string | | | `WorldPar("graveyard_module_grave_1")` |

## World Objects

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `AddInteractionEvent` | text: string, id: string, isFake?: bool | Adds an interaction event to the world object with that custom tag. | | `AddInteractionEvent("bed_home", "work", true)` |
| `AddWgoPart` | wgoPartId: string | | a world object | `AddWgoPart("grave_bot_wd_2")` |
| `ChangeWgo` | newWgoId: string | | a world object | `ChangeWgo("grave_body")` |
| `DestroyWgoData` | text: string | Removes the world object with that custom tag from the world. | | `DestroyWgoData("repair_sign_1304")` |
| `DisableGDPointById` | gdPointId: string | | | `DisableGDPointById("gd_potato_farm_t1")` |
| `EnableGDPointById` | gdPointId: string | | | `EnableGDPointById("gd_potato_farm_t1")` |
| `FireEventOnWgo` | eventName: string | | a world object | `FireEventOnWgo("OnCreate")` |
| `GSFireEvent` | scriptName: string, eventName: string | | | `GSFireEvent("NPCSimTown", "1402")` |
| `GSRun` | arg1: string | | | `GSRun("Building_LiftCrane")` |
| `LazyExpressionOnWgo` | text: string, text2: string | Runs the second expression on the world object with that custom tag. | | `LazyExpressionOnWgo("repair_sign_1304", "RepairTownCluster()")` |
| `RemoveAllWgoParts` | | | a world object | `RemoveAllWgoParts()` |
| `RemoveInteractionEvent` | text: string, id: string | Removes an interaction event from the world object with that custom tag. | | `RemoveInteractionEvent("npc_herm", "talk")` |
| `RemoveWgoPart` | wgoPartId: string | | a world object | `RemoveWgoPart("grave_bot_wd_1")` |
| `ReplaceWgoByIds` | id: string, newWgoId: string | | | `ReplaceWgoByIds("garden_t0","garden_t1")` |
| `SetRandomSpawnWgoStages` | configIds...: string | Picks one of the given spawn configuration names at random and gives the world object that configuration's spawn stages. Any number of names can be given. | a world object | `SetRandomSpawnWgoStages("tree_apple_clr1_v1","tree_apple_clr1_v2","tree_apple_clr1_v3")` |
| `SetRandomVariation` | | | a world object | `SetRandomVariation()` |
| `SetResWgo` | text: string, id: string, value: float | | | `SetResWgo("npc_goddess_statue","has_tears",0)` |
| `SetWGOParByTag` | text: string, id: string, value: float | Sets a parameter on the world object with that custom tag. | | `SetWGOParByTag("bed_home","bed_upgrade",1)` |
| `SetWgoHiddenState` | text: string, flag: bool | | | `SetWgoHiddenState("looters_chest", false)` |
| `SetWgoHiddenStateByTag` | text: string, flag: bool | Hides or shows the world object with that custom tag. | | `SetWgoHiddenStateByTag("chest_graveyard", "true")` |
| `SetWgoInteractionState` | text: string, flag: bool | Turns interaction on or off for the world object with that custom tag. | | `SetWgoInteractionState("npc_nun", false)` |
| `SetWgoPartStateAll` | arg1: string, arg2?: int | | a world object | `SetWgoPartStateAll("open")` |
| `SetWgoVariation` | text: string, arg2: string, arg3?: int | | | `SetWgoVariation("church_doors", "opened")` |
| `SpawnConveyorWgoOnGDPoint` | wgoId: string, text: string, gameSceneId: string, customTag: string | | | `SpawnConveyorWgoOnGDPoint("conveyor_wine_beer_pallet","conveyor_wine_beer_pallet_point","RuinedTemple","")` |
| `SpawnConveyorWgoOnThis` | text: string, arg2?: bool | | a world object | `SpawnConveyorWgoOnThis("zombie_carousel")` |
| `SpawnWgoOnGDPoint` | wgoId: string, text: string, gameSceneId: string, customTag: string | | | `SpawnWgoOnGDPoint("millstone","millstone_point","RuinedTemple","")` |
| `Spawning` | | | a world object | `Spawning()` |
| `TeleportTo` | wgoIdTeleportTo: string, environmentPreset?: string, soundOnTeleport?: string | Teleports the player to a world object's teleport spot. The optional values name the environment preset to switch to and a sound to play. | | `TeleportTo("tp_cave_top")` |
| `WGOFireEvent` | text: string, eventName: string | Fires an event on the world object with that custom tag. | | `WGOFireEvent("npc_herm", "tp_to_spot")` |

## Fights

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `Despawn_FightingLvl` | id: string, gameSceneId?: string | | | `Despawn_FightingLvl("fight_C0_1")` |
| `GetArmor` | | | a world object | `Item_Armor(GetArmor())` |
| `GetWeapon` | | | a world object | `Item_Dmg(GetWeapon())` |
| `IsFightActive` | | | | `IsFightActive()` |
| `Item_Armor` | item: any | | an item | `Item_Armor(GetArmor())` |
| `Item_AtkPause` | item: any | | an item | `Item_AtkPause(GetWeapon())` |
| `Item_DPTag` | item: any | | an item | `Item_DPTag(GetWeapon())` |
| `Item_Dmg` | item: any | | a combat target, an item | `Item_Dmg(GetWeapon())` |
| `Item_KnForce` | item: any | | an item | `Item_KnForce(GetWeapon())` |
| `Item_Range` | item: any | | an item | `Item_Range(GetWeapon())` |
| `LockMapFightIcon` | icon: string | | | `LockMapFightIcon("icon_A3_1")` |
| `Replace_To_Broken_Barricade` | | | a world object | `Replace_To_Broken_Barricade()` |
| `SetMercenariesPaymentId` | mercenariesPaymentId: string | | | `SetMercenariesPaymentId("item_price_1")` |
| `SetStage_FightingLvl` | id: string, stageId: int, gameSceneId?: string | | | `SetStage_FightingLvl("fight_A3_1", 5)` |
| `Spawn_FightingLvl` | id: string, gameSceneId?: string | | | `Spawn_FightingLvl("fight_B3_1")` |
| `UnlockMapFightIcon` | icon: string | | | `UnlockMapFightIcon("icon_B2_1")` |

## NPC Simulation

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `NPCSim_RollActivityForGroup` | groupId: string | | | `NPCSim_RollActivityForGroup("town_A4_guards")` |
| `NPCSim_SendGroupHome` | groupId: string | | | `NPCSim_SendGroupHome("town_A1_day_all")` |

## Weather, Light, and Effects

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `ResetWeatherState` | | | | `ResetWeatherState()` |
| `SpawnFxOnGDPoint` | text: string, name: string | | | `SpawnFxOnGDPoint("millstone_point", "puff")` |

## Other

| Function | Parameters | Description | Needs | Example from the game |
| --- | --- | --- | --- | --- |
| `GetDeltaValue` | | | a change amount | `AddInspiration("insp_church_quality", GetDeltaValue())` |
| `TryPlaceGatherOrder` | | | a world object | `TryPlaceGatherOrder()` |
| `TryPlacePlantOrder` | | | a world object | `TryPlacePlantOrder()` |
