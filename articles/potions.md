# Potions

Core lets you add potions that provide buffs or debuffs, appear in the folio, and can be crafted.

## Adding a Potion

Potions are added with the `Potions` service and a new [`TimbnPotion`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnPotion) object. Adding a potion with an existing ID replaces the previous registration.

```csharp
Potions.Add(new TimbnPotion
{
    Id = "timbn_swift_and_sticky",
    Name = "Swift and Sticky",
    Description = "Makes you walk faster for a while, and drops a few sticks.",
    //Red Green Blue
    Runes = new Vector3Int(2, 0, 1),
    Buff = new TimbnPotionBuff
    {
        Name = "Swift",
        Description = "You walk faster.",
        GlowColor = () => "yellow",
        DurationSeconds = () => 120f,
        StartExpressions = ["DropItem(\"stick\", 5)"],
        OnStart = () => _swift = Player.MultiplyWalkSpeed(1.5f),
        OnEnd = () => _swift?.Dispose(),
    },
});
```

Rune counts must be unique. The game gives each mix to the first formula whose runes match, so a potion sharing runes with an existing one would never work. Core refuses to add such a potion and logs why. It also skips a potion whose id is already an item in the game.

Core makes the potion an item, a buff, and a formula that brews at the alchemy mixer, and calculates every ingredient combination that matches its runes. Drinking it also counts toward the game's potion drinker inspiration, like any potion. The buff's id is the potion's id with `_buff` on the end (`BuffId`). The names and descriptions are added for you, so there is no lang file to write. `Potions.Add` returns a handle that removes the potion early, which you don't need to keep.

| Property | What it does |
| --- | --- |
| `Id` | The potion's item id. Keep it the same between versions, since saves keep the item under it. |
| `Name`, `Description` | What the player reads on the item. |
| `Runes` | The red, green and blue rune counts of the formula, as a `Vector3Int`. |
| `Buff` | What drinking it does, described below. |

## The Buff

The `Buff` holds the name and description the HUD shows, and the hooks that run while it lasts.

| Property | What it does |
| --- | --- |
| `Name`, `Description` | The buff's name in the HUD and its tooltip. |
| `DurationSeconds` | How long it lasts in real seconds, 60 by default. It is a function so it can follow a setting, and it is read each time the potion is drunk. |
| `GlowColor` | The color of the effects that play when the buff starts, `ltyellow` by default. It is a function like the duration. One effect plays on the player's character and another on the buff's icon in the HUD. The game has both effects for `blue`, `gray`, `green`, `ltblue`, `ltyellow`, `pink`, `purple`, `red`, and `yellow`. It is a plain lowercase name, so any other name has no effects. The buff still works, but the game logs an error each time it starts. |
| `IconId` | The buff's icon, `b_speed` by default. |

## Status Effects

These are properties of the `Buff`. They control how the effect is applied and removed.

### `StartExpressions`

Uses the game's own [expressions](expressions.md). Write each one as a C# string, with the quotes around its arguments escaped, such as `"DropItem(\"stick\", 5)"`. Use them for changes the save keeps, such as items or unlocks. A potion with an expression that doesn't work is skipped with a warning.

> [!NOTE]
> Runs only when the potion is first applied.

### `EndExpressions`

Works the same way as `StartExpressions`, but runs when the buff ends. If the mod is removed while the buff is active, Core ends the buff the next time the save loads and runs these expressions then.

### `OnStart`

Runs custom logic when the potion is applied.

> [!NOTE]
> Runs when the potion is applied and when a save loads with the potion already active.

### `OnEnd`

Runs custom logic when the potion expires, the player leaves the game, or the potion is removed.

### `WhileActive`

Runs every frame while the potion buff is active.

### `ApplyChange` and `RevertChange`

Used to keep C# changes out of the save. `ApplyChange` runs when the buff starts, `RevertChange` reverts the modifications immediately before every save write and when the buff ends, and `ApplyChange` restores them immediately after the save. The save never holds the change, so removing the mod while a buff is active leaves nothing behind. Both run on every save, so keep them to setting state, with no sounds, toasts, or drops.

## Customization

The potion's `IconId` and the buff's `IconId` can each be any sprite the game has (potions use `i_pot_*`, buffs `b_*`), or one added with `Sprites.AddPng(name, pngBytes)`. The potion keeps its template's icon when `IconId` is empty. PNGs smaller than the game's 48 by 48 icon canvas are centered on one, and a larger PNG is squashed to fit with a warning in the log.

`TemplateItemId` is the usable item the new potion is copied from, `third_eye_salt` by default. `Price` is what the potion sells for, 100 by default. `Tab` is the folio tab its formula shows under (`Simple`, `Medium`, `Hard`, `Epic`, `Heroic`, or one of the rune tabs), `Epic` by default, and `HiddenAtStart` keeps the formula hidden until the player finds it.
