# Stations and Crafts

Core can add workstations with your own model, bench extensions, recipes, new items, and decorations for a grave. Each one is built, worked and dismantled like the game's own, and everything goes away again when the mod unloads.

```csharp
protected override void OnAwake()
{
    var boxModel = Models.Build("StickBox", faces => faces
        .Block("stick_box", x: 0, y: 0, z: 0, depth: 20));
    var boxSite = Models.Build("StickBoxSite", faces => faces
        .Top("stick_box_site", x: 0, y: 1, z: 0));

    var box = Stations.Add("timbn_stick_box", "Stick Box", "A red box for splitting boards into sticks.", station => station
        .Model(boxModel, new Vector3(1f, 0.6f, 1f))
        .Site(boxSite)
        .BasedOn("sawhorse")
        .Icon("timbn_stick_box")
        .Cost(("flitch", 2)));

    Crafts.Add("timbn_stick_box_sticks", "Sticks from a Board", "Split one board into 4 sticks.", craft => craft
        .In(box)
        .Needs("flitch")
        .Makes("stick", 4));
}
```

The yard build desk now offers a Stick Box for 2 boards. Placing it puts down the red outline, the player builds it with the hammer, and the box appears in its place. At the box, one board makes 4 sticks.

## Models

The `Model` calls below take a [model](models.md), which you build from PNGs with `Models.Build`. The samples on this page keep theirs in locals like `boxModel` and `pillarModel`. [Models](models.md) covers building one, along with the pixel scale and the art rules.

## Adding a Station

### `Stations.Add(id, name, description, station)`

Adds a workstation to a build desk. Core makes every definition the game expects and names them the way the game does (the station `<id>`, its construction site `<id>_place` and its build desk entry `<id>_place_p`). Keep the id the same between versions, since a save names a built station by it. Call it in `OnAwake`, before the crafts that run at it, and pass the returned [`TimbnStation`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnStation) to `In` on those crafts. The station object also holds the ids Core made, as `Id`, `SiteId` and `BuildingId`. `Stations.Add` throws when the id is empty or no model was given.

```csharp
var rack = Stations.Add("timbn_smoking_rack", "Red Workbench", "A plain red bench for smoking fish.", station => station
    .Model(benchModel, new Vector3(1.5f, 0.8f, 1f))
    .Site(benchSite)
    .BasedOn("kitchen_oven")
    .Icon("timbn_red_workbench")
    .Cost(("flitch", 4))
    .NoFuel()
    .NoExtensions());
```

This bench works like the kitchen oven, so it cooks on its own and a zombie can be put on it, but it burns no firewood and takes no extensions.

| Method | What it does |
| --- | --- |
| `Model(model)` | The finished station's look, with a footprint the size of the model. The only call you need. |
| `Model(model, size)` | The same with a footprint of your own width, height and depth in world units. |
| `Site(model)` | The construction site's look, which stands there until the hammer work is done. Without it the site is a dashed outline of the footprint with the game's warning sign at the front. |
| `NoConstruction()` | Puts the finished station down as soon as it is placed, with no site and no hammer work. |
| `BasedOn(stationId)` | The game station the new one behaves like. `kitchen_oven` runs on its own and takes a zombie, `sawhorse` is worked by hand. Defaults to the woodworking bench. |
| `Talent(talentId)` | The skill its crafts and its dismantling use, such as `talent_orange` for the woodworking bench. |
| `Tool(tool)` | The tool needed to work it, such as `ItemType.Hammer`, or `ItemType.None`. |
| `TakesZombie(takes)` | Whether a zombie can be put to work at it. |
| `Automatic(tickDuration)` | Runs its crafts by itself over time, the way the oven cooks while the player is away. `Manual()` makes it progress only while someone works it. |
| `CraftSlots(slots)` | How many ingredient slots its craft window has. |
| `Storage(slots)` | How many slots its own inventory has. |
| `WorkSound(soundId)` | The sound each work tick makes. |
| `DockDistance(distance)` | How far past the edge of the footprint the work spot is, 0.45 by default. A closer spot suits a small station. The construction site keeps at least 0.45, so the Keeper can reach it. |
| `Icon(spriteId)` | The icon over the station and in the build desk. Any sprite the game has, or one added with `Sprites.AddEmbeddedPng`. |
| `Cost((item, count), ...)` | What the build desk charges. A second call adds to the first, and no cost means free. |
| `Refund((item, count), ...)` | What dismantling the station gives back. Half the cost by default, and an unbuilt site always gives all of it back. |
| `NoRefund()` | Dismantling gives nothing back. |
| `BuiltAt(buildDeskId, areaId)` | Which build desk offers it, `builder_yard` by default. Others are `builder_graveyard` and `builder_garden` (`TimbnGameIds.BuildDesks` names every one). Pass an area id such as `yard_place` to keep it to one build area. |
| `BuildTab(tabId)` | Which tab of the build desk lists it, from `TimbnGameIds.BuildTabs`. |
| `Limit(max)` | How many the player can have at once. The build desk shows how many are built out of how many are allowed and refuses another at the cap. |
| `NoFuel()` | Takes away the fuel the template burns. |
| `Extensions(extensionId, ...)` | The bench extensions that attach to it, such as the game's `tool_rack`. The first call replaces the template's list. |
| `NoExtensions()` | No extension attaches, whatever the template took. |
| `UnlockedBy(techId)` | The station is missing from the build desk until the player researches the tech, the way `work_with_wood_1` opens the woodworking bench. The tech can be one of the game's or one you add with [`Techs.Add`](techs.md). A save that already has the tech gets it right away. |
| `Edit(edit)` | Changes anything else on the station's `WGODef` after Core has filled it in. |

## Facings

### `Left(model)`, `Up(model)` and `Right(model)`

Give the station a look for each way it can face, and the player turns it with R while placing it. R goes through the facings the station has in the order front, left, back, right. Each facing has its own footprint and work spot, and takes its own construction site as a second argument. Like `Model`, each also has a form with a size of your own, such as `Left(model, size, site)`. The station is still one building with one id, so crafts don't care which way it faces.

### `MirrorSides()`

Fills a missing left or right facing with a flipped copy of the other, so you only draw one of the two.

```csharp
station
    .Model(longStill)
    .Left(narrowStill)
    .MirrorSides();
```

## Looks That Change

### `ShowWhen(condition, model, replacesBase)`

Adds a model that shows while a condition holds and goes away when it doesn't, the way the game's autopsy table shows a cloth over a body. The game checks again whenever the station's items, values, worker or work change, so the mod does nothing more. Pass `replacesBase: true` to hide the station's own model while this one shows. Only one model per facing can replace the base, and `Stations.Add` throws when two try. A version with a [`TimbnFacing`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnFacing) first (`Down` is the front, then `Left`, `Up` and `Right`) adds the model to that facing only.

```csharp
station
    .Model(emptySlab)
    .ShowWhen(TimbnLook.HasWorker(), slabWithZombie, replacesBase: true)
    .ShowWhen(TimbnLook.Resource("glow", TimbnComparison.GreaterOrEqual, 1), glow);
```

The conditions come from [`TimbnLook`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnLook).

| Condition | True while |
| --- | --- |
| `HoldsItem(itemId, atLeast)` | The station holds at least that many of an item, 1 by default. |
| `HoldsGroup(groupId, atLeast)` | The station holds at least that many items of a group, such as `zombie`, 1 by default. |
| `Resource(resourceId, comparison, value)` | One of the station's own values compares as asked, with a [`TimbnComparison`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnComparison) (`Less`, `LessOrEqual`, `Equal`, `GreaterOrEqual` or `Greater`). A mod sets one with the game's `SetWGOPar`, and one never set counts as 0. |
| `IsWorking()` | Someone is working the station. |
| `IsCrafting()` | A craft runs on it, whoever runs it. |
| `HasWorker()` | It has a worker, the player or a zombie. |

Calling `.Not()` on any condition inverts it. A model made from pictures lines up with the station's own model, whatever their sizes.

## Zombies at a Station

### `ZombieSpot(takesOrders)`

Gives the station a spot where a zombie sits as its worker, for a station based on the game's zombie supplier (`zombie_supplier_station`). The player puts a zombie in with E and takes it out the usual way. A zombie sitting there would normally walk off to carry things for zombie crafter benches, so Core keeps it sitting unless `takesOrders` is true.

### `HoldsZombie()`

Lets the player lay a zombie on the station and take it back, the way the game's corpse pallet holds a corpse. E with a zombie overhead puts it in, and E with free hands takes it back unless a craft is running there, so a craft started on the station holds the zombie until it ends. The station can't be dismantled while a zombie lies on it. Show the zombie with `ShowWhen(TimbnLook.HoldsGroup("zombie"), ...)`.

```csharp
var stand = Stations.Add("timbn_infusion_stand", "Infusion Stand", "A zombie rests in this box.", station => station
    .Model(openBox)
    .BasedOn("resurrection_table_1")
    .HoldsZombie()
    .ShowWhen(TimbnLook.HoldsGroup("zombie"), lid));
```

[`TimbnZombies.AtStation(station)`](helpers/timbn-zombies.md) finds the zombie a station holds, for example to pay it tech points when a craft there ends.

## Slots

A station can have slots built into it, spots where another station or extension goes, the way the game's furnace has spots for its bellows. In build mode the game shows each free slot, lets the piece go down only there, and links the two. Removing the station removes what sits in its slots, refunds it, and drops anything it held on the ground.

### `Slot(slotId, position, size)`

Builds a slot into the station. `position` is where the center of what goes in it sits, measured from the station's center on the ground, and `size` is the size of what goes in it, so pass the same size you gave that one's `Model`. The height of both is ignored. While the station is being placed its slots show as blue squares. A version with a facing first adds the slot to that facing only.

```csharp
var machine = Stations.Add("timbn_infusion_machine", "Infusion Machine", "Teaches the zombies on its stands.", station => station
    .Model(table, new Vector3(0.64f, 0.5f, 0.6f))
    .Slot("timbn_infusion_slot", new Vector3(-0.8f, 0f, 0f), standSize)
    .Slot("timbn_infusion_slot", new Vector3(0.8f, 0f, 0f), standSize)
    .Extensions("timbn_infusion_stand"));
```

The game snaps buildings to a grid of cells 0.32 across and 0.3 deep (16 by 12 pixels in a model from pictures). A slot has to be whole cells, with its corner a whole number of cells from the corner of the station's footprint, and `Stations.Add` throws and names the nearest spot that fits when it isn't.

### `BuildsInSlot(slotId)`

Lets the player build the station only into a slot with that id. It goes down finished, with no construction site, like the game's own slot pieces, and it is still a normal station the player works. Give it the same footprint as the slot, and leave out its `Site`, since a site with `BuildsInSlot` throws. Bench extensions have it too.

```csharp
Stations.Add("timbn_infusion_stand", "Infusion Stand", "A zombie rests in this box.", station => station
    .Model(openBox, standSize)
    .BuildsInSlot("timbn_infusion_slot")
    .BuiltAt(TimbnGameIds.BuildDesks.Morgue));
```

> [!NOTE]
> A slot doesn't count as build ground by itself, so the ground under it still has to be a place the build desk builds on.

[`TimbnWorld.Extensions(station)`](helpers/timbn-world.md) lists what sits in a station's slots.

## Bench Extensions

### `Stations.AddExtension(id, name, description, extension)`

Adds an extension like the game's tool rack. It is placed straight from the build desk with no construction site, and there is nothing to interact with. It attaches to a station that lists it in `Extensions` when the two are built next to each other, and opens the crafts at that station that call `NeedsExtension` with its id. Like `Crafts.Add`, it returns a handle that removes the extension early.

```csharp
Stations.AddExtension("timbn_red_magic", "Red Magic Pillar", "A humming red pillar.", pillar => pillar
    .Model(pillarModel, new Vector3(0.5f, 1f, 0.5f))
    .Icon("timbn_red_magic")
    .Cost(("flitch", 2)));

var box = Stations.Add("timbn_stick_box", "Stick Box", "A red box for splitting boards into sticks.", station => station
    .Model(boxModel, new Vector3(1f, 0.6f, 1f))
    .BasedOn("sawhorse")
    .Extensions("tool_rack", "timbn_red_magic"));
```

The builder has `Model(model, size)`, `Icon`, `Cost`, `Refund`, `NoRefund`, `BuiltAt`, `BuildsInSlot`, `UnlockedBy` and `Edit` like a station's, plus two of its own. An extension always requires an explicit size, as it lacks a single-argument `Model(model)` overload.

| Method | What it does |
| --- | --- |
| `Reach(distance)` | How far past its footprint it reaches to attach to a station, 0.64 by default like the tool rack. |
| `BasedOn(extensionId)` | The game extension it is copied from, `tool_rack` by default. |

Only a station you add with `Stations.Add` has `Extensions`, so an extension can't be attached to one of the game's own stations.

## Crafts

### `Crafts.Add(id, name, description, craft)`

Adds a recipe at any station, yours or the game's. The craft starts as a copy of one of the game's, the wooden plank unless `BasedOn` names another, so it has everything the game expects and you only set what differs. Unspecified properties retain the template's values, except that a craft is available from the start unless you call `NeedsUnlock` or `UnlockedBy`. It returns a handle that removes the craft early when disposed, which you don't need to keep.

```csharp
Crafts.Add("timbn_smoke_fillet_fish_river", "Smoked Fish", "A fish fillet and a log of firewood make one smoked fish.", craft => craft
    .In(rack)
    .BasedOn("cook_baked_fish")
    .Needs(("fillet_fish_river", 1), ("firewood", 1))
    .Makes("timbn_smoked_fish")
    .AutoFinish());

Crafts.Add("timbn_stick_box_magic_plank", "Plank from Sticks", "The red pillar binds 4 sticks back into a plank.", craft => craft
    .In(box)
    .NeedsExtension("timbn_red_magic")
    .Needs("stick", 4)
    .Makes("wooden_plank"));
```

| Method | What it does |
| --- | --- |
| `In(station)` | Where it runs, a `TimbnStation` or a game station id such as `kitchen_oven`. Call it once per station. The first call replaces the template's stations. Whether the craft runs automatically depends on the station. |
| `BasedOn(craftId)` | The game craft it is copied from. Pick one from the same kind of station to get its work time, energy and tab. |
| `Needs(item, count)` | An item it takes in, 1 by default. `Needs((item, count), ...)` takes several. The first call replaces the template's. |
| `Makes(item, count)` | What it makes, 1 by default. The first call replaces the template's. |
| `NeedsExtension(extensionId)` | Runs only while that extension is attached. The craft window shows it in the extension's row. |
| `Requires(check)` | Starts only while your check passes for the station, such as a stand holding a zombie. Until then the Start button stays gray and a queued craft waits, the way they do for a missing ingredient. |
| `Mastery(level)` | The tool level the player or zombie needs. `Mastery(1)` lets any tool work it. |
| `DurationTicks(ticks)` | How long it takes in work ticks, such as a tool swing at a bench or a progress tick at an automated station. |
| `EnergyPerTick(energy)` | The energy each tick costs the player. |
| `AutoFinish()` | At a station that runs itself, drops each result in front of it and starts the next craft in the queue, instead of waiting for the player to take it. |
| `OnStarted(handler)` | Runs your code each time the craft starts at a station, which is when its ingredients are taken. A craft with a count above one starts once per unit. |
| `OnFinished(handler)` | Runs your code each time the craft finishes at a station, once per unit. A craft that fails while finishing doesn't raise it. |
| `NeedsUnlock()` | Hidden until a tech or `TimbnPlayer.UnlockCraft` unlocks it. |
| `UnlockedBy(techId)` | Hidden until the player researches the tech, which lists it in its tooltip. The tech can be one of the game's or one you add with [`Techs.Add`](techs.md). A save that already has the tech gets it right away. |
| `Icon(spriteId)` | The icon in the craft window, instead of the icon of what it makes. |
| `Edit(edit)` | Changes anything else on the `CraftDef` after Core has filled it in, such as its `tabId`. |

```csharp
Crafts.Add("timbn_lesson", "Lesson", "Teaches the zombies on the stands.", craft => craft
    .In(machine)
    .Requires(station => TimbnWorld.Extensions(station).Any(stand => TimbnZombies.AtStation(stand) != null))
    .OnFinished(done => PayZombies(done.Station)));
```

The handlers receive a [`TimbnCraftEvent`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnCraftEvent) with the `Station` the craft runs at (a `WgoData`) and the craft's `CraftId`. A handler or check that throws is logged and doesn't stop the craft. A second `Requires` adds another check that must pass too, and a check that throws counts as passing.

A count can also be one of the game's expressions, so a perk can change it. A count that doesn't parse throws when the mod loads.

```csharp
Crafts.Add("timbn_smoke_fish_batch", "Smoked Fish Batch", "A batch of fish smoked over a slow fire.", craft => craft
    .In(rack)
    .BasedOn("cook_baked_fish")
    .Needs("fillet_fish_river", 3)
    .Needs("firewood", "10-5*#perk_heat_saver")
    .Makes("timbn_smoked_fish", "3+2*#perk_cook_2"));
```

For any other expression field you set by hand in `Edit`, build it with `TimbnBalance.Expression(formula)`, which reads `#perk` and `$resource` the way the game's own balance does.

### `Crafts.Start(station, craftId)`

Starts a craft at a station (a `WgoData`, such as one from `TimbnWorld.Extensions`) from code, with no craft window, the way the game's own expressions do. It goes in the station's queue and runs like one the player started. End one early with the game's `station.CraftComponent.TryFinishCurCraft()`, which finishes it as if its time ran out. Returns false when the game has no such craft.

```csharp
foreach (var built in TimbnWorld.FindAll(machine.Id))
    foreach (var stand in TimbnWorld.Extensions(built))
        Crafts.Start(stand, "timbn_infusion_stand_infusion");
```

## Items

### `Balance.AddItem(templateItemId, id, name, description, iconId, edit)`

Adds an item copied from one of the game's, with its own name, description and icon. Everything else, such as its price and what using it does, stays the template's unless `edit` changes it.

```csharp
Sprites.AddEmbeddedPng("timbn_smoked_fish");
Balance.AddItem("cook_baked_fish", "timbn_smoked_fish", "Smoked Fish", "Golden, smoky, and good for a long night of digging.", "timbn_smoked_fish");
```

Smoked fish is now food that restores energy like baked fish. Use `edit` for anything else.

```csharp
Balance.AddItem("cook_muffin", "timbn_cheese_pie", "Cheese Pie", "Rich and filling.", "timbn_cheese_pie", pie => pie.basePrice = 40);
```

## Text From Lang Files

Pass empty text, or leave the name and description out, and the text comes from the mod's [lang files](quests.md#translations) instead.

```csharp
var box = Stations.Add("timbn_stick_box", station => station
    .Model(boxModel, new Vector3(1f, 0.6f, 1f))
    .BasedOn("sawhorse"));

Stations.AddExtension("timbn_red_magic", pillar => pillar
    .Model(pillarModel, new Vector3(0.5f, 1f, 0.5f)));

Crafts.Add("timbn_stick_box_sticks", craft => craft
    .In(box)
    .Needs("flitch")
    .Makes("stick", 4));

Balance.AddItem("cook_baked_fish", "timbn_smoked_fish", "", "", "timbn_smoked_fish");
```

```text
# lang/en.txt
timbn_stick_box = Stick Box
timbn_stick_box_place = Stick Box
timbn_stick_box_place_p = Stick Box
timbn_stick_box_place_p_d = A red box for splitting boards into sticks.
timbn_red_magic = Red Magic Pillar
timbn_red_magic_p = Red Magic Pillar
timbn_red_magic_p_d = A humming red pillar.
timbn_stick_box_sticks = Sticks from a Board
timbn_stick_box_sticks_d = Split one board into 4 sticks.
timbn_smoked_fish = Smoked Fish
timbn_smoked_fish_d = Golden, smoky, and good for a long night of digging.
```

A station's name goes on the station, its construction site and its build desk entry, and the description on the build desk entry. An extension has no site, and a craft and an item use their id for the name and `<id>_d` for the description.

## Grave Decorations

A headstone or a fence for a grave is an item plus a part drawn from a model. [Models](models.md#for-a-part-on-a-world-object) covers `Models.AddPart` and the full recipe.

## Saves

A save holding a station, craft, or item that no longer exists will not load once the mod is removed. Core strips your stations, queued crafts, items, and grave parts (and your items picked as a chest filter) from the save file during writes and restores them when the save loads. Removing the mod leaves a save that still loads, and reinstalling the mod brings everything back. There is nothing to configure.

A few rare edge cases can still leave a mod item in the save. [Removing a Mod Safely](removing-a-mod.md) lists what is handled and what may be left behind.
