# Techs

Core can add a tech to the game's tech tree. The player buys it with tech points like any other, and a station or recipe can name it in `UnlockedBy`, so mod content remains locked until the tech is unlocked. The tech goes away again when the mod unloads.

```csharp
protected override void OnAwake()
{
    Techs.Add("timbn_infusion_rack", "Infusion Rack", tech => tech
        .Tab(TechTreeTab.Anatomy)
        .At(8, 2)
        .After(TimbnGameIds.Techs.BodyPlaces2)
        .Cost(red: 50, green: 50, blue: 20)
        .Shows("timbn_infusion_machine", "timbn_spirit_solution_red", "timbn_infusion_stand"));

    Stations.Add("timbn_infusion_machine", "Infusion Machine", "Teaches the zombies on its stands.", station => station
        .Model(table, new Vector3(0.64f, 0.5f, 0.6f))
        .UnlockedBy("timbn_infusion_rack"));
}
```

The Anatomy tab now has an Infusion Rack right after Body Places II. Buying it puts the Infusion Machine on the build desk.

Add the tech before the stations and crafts that name it, so they find it when you reload scripts with F6.

## `Techs.Add(id, name, tech)`

Adds a tech to the tree. `id` is the tech's ID, which saves use to record that it was unlocked, so keep it stable. `name` is the name shown on the tree, and leaving it empty defers resolution to your lang files using the tech ID. It returns a handle that removes the tech early, which you don't need to keep.

| Builder call | What it does |
| - | - |
| `At(column, row)` | Required. The spot the tech sits at. |
| `After(techId, ...)` | Required. The techs the player needs first. The tree joins them to this one with a line, and the tech opens once every one of them is bought. |
| `Tab(tab)` | The tab that holds it, from `TechTreeTab` (`Building`, `Metallurgy`, `Farming`, `Theology`, `Anatomy`, and `Cooking`). Defaults to the Building tab when omitted. |
| `Cost(red, green, blue)` | The tech points it costs, any of which can be left at zero. |
| `Shows(id, ...)` | The icons the tree and the tech's popup list, in this order. See Rewards. |

The names of the game's own techs are in `TimbnGameIds.Techs`.

## Where a Tech Goes

Every tab is five rows tall, rows 0 to 4, and row 0 is the bottom, so higher row numbers are positioned higher in the tree view. Columns start at 0 on the left and can go as far right as you need. The tree puts a column 210 units from the next and a row 80, and a tech is 160 units wide and 80 tall. It never moves anything out of the way, so pick a spot nothing else uses.

Half steps work too. The game's Graveyard Path and Fence I sits at row 0.5, between two rows.

If another tech on the tab sits close enough to overlap it, `Techs.Add` adds nothing and logs a warning that names both techs and the spot. A tech that wasn't added leaves every station and recipe that names it in `UnlockedBy` with nothing to unlock them, so check the log after you pick a spot.

## Rewards

A station or craft puts itself on the tech with `UnlockedBy(techId)`. See [Stations and Crafts](stations-and-crafts.md). The tech lists whatever names it as icons on its tree node and in its popup.

The popup only has room for three icons, and a node on the tree is about five icons wide. For a tech with more than that, such as one that unlocks many recipes, `Shows` picks the icons and their order. Each id can be:

- a station you added, or a build desk entry, which shows as the building,
- an item, which shows with no recipe and a line saying where it is crafted,
- a craft, which shows with its recipe.

An id the game doesn't have is skipped. What the tech unlocks doesn't change, only the icons.

## Saves

The game drops tech IDs it does not recognize when a save loads. A save that unlocked your tech still loads once the mod is gone, and the player must repurchase the tech if the mod is reinstalled.
