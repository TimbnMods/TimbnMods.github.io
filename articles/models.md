# Models

A model is the look of something Core adds to the world, such as a station, a bench extension or a decoration on a grave. Core draws it with a copy of the game's own prop material, so it takes the game's lighting and fog like any prop. A model does nothing on its own. You make one, then hand it to `Stations.Add`, `Stations.AddExtension` or `Models.AddPart`, and that puts it in the world.

A model is built from pictures with `Models.Build`, which gives flat pixel art in the game's own look. All you need is PNGs, with no 3D modeling and no Unity. It lives on the `Models` property of your mod class ([`TimbnPluginModels`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnPluginModels)) and hands back a [`TimbnModel`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnModel).

Here is a small mod that builds a splitting block from three PNGs and turns boards into firewood on it.

```csharp
protected override void OnAwake()
{
    var block = Models.Build("SplittingBlock", faces => faces
        .Top("block_shadow", x: 0, y: 1, z: 0)
        .Block("block_stump", x: 2, y: 0, z: 3, depth: 20)
        .Front("block_axe", x: 14, y: 18, z: 10));

    var station = Stations.Add("timbn_splitting_block", "Splitting Block", "A stump for splitting boards into firewood.", s => s
        .Model(block)
        .BasedOn("sawhorse")
        .Cost(("flitch", 2)));

    Crafts.Add("timbn_splitting_block_firewood", "Firewood from a Board", "Split one board into 2 logs of firewood.", craft => craft
        .In(station)
        .Needs("flitch")
        .Makes("firewood", 2));
}
```

The block_shadow picture is 44 by 26 pixels and lies on the ground. The block_stump picture is 40 wide and 38 tall, so its top takes the upper 20 rows and its front the lower 18. The block_axe picture is 6 by 16 and stands on the stump. The rest of this page covers each piece.

> [!NOTE]
> `Models.Build` only makes a model. Nothing shows in the world until a station, an extension or a part uses it.

## Models From Pictures

### `Models.Build(name, faces)`

Makes a model from PNGs. The game's camera looks forward and down and never turns sideways, so only two kinds of face are ever seen. One stands up toward the camera (a front) and one lies face up (a top). A solid box is both at once (a block). Sides, backs and undersides are never built, because nobody can see them.

Each PNG is a face exactly as the player sees it, so the PNG's own size is the face's size and you only say where it stands. `name` is the model's name, for example `"SplittingBlock"`, and shows up in error messages and logs. The faces are added in the callback, and the model is ready when `Build` returns. It is freed when the mod unloads.

`Build` throws an `InvalidOperationException` when the model has no faces, a PNG is missing or can't be read, a block's depth leaves no front, or two faces on the same plane overlap. It throws an `ArgumentOutOfRangeException` when a position is below zero or a block's depth is below 1. The message names the face.

### Faces

| Method | What it does |
| --- | --- |
| `Front(png, x, y, z)` | Stands a picture up facing the camera, such as a screen or a sign. |
| `Top(png, x, y, z)` | Lays a picture flat, face up, such as a shadow or a keyboard on a desk. Its bottom row is the front edge, so it reaches from `z` to `z` plus the PNG's height. |
| `Block(png, x, y, z, depth)` | A solid box from one picture, with its top stacked above its front. The upper `depth` rows are the top and the rest are the front, so the front is the PNG's height minus `depth`. Both share the block's `x` and width. |

Each of the three has a second form that takes the PNG's bytes instead of a name, for a picture made in code, such as `Front(bytes, x, y, z, name)`. The optional `name` is what error messages call the face.

`png` is the name of a PNG embedded in the mod's DLL, written without ".png". Embed the pictures with the file name as the resource name, or Core won't find them.

```xml
<ItemGroup>
  <EmbeddedResource Include="Art\*.png" LogicalName="%(Filename)%(Extension)" />
</ItemGroup>
```

Positions are in pixels from the model's front, bottom, left corner as the player sees it, standing in front of it. The x axis runs right, y runs up and z runs away from the player. A position can't be negative, so the model's left, floor and front edges are always at 0 or more.

| Position | Counts | Meaning |
| --- | --- | --- |
| `x` | Pixels from the model's left edge | Where the face's left edge is. |
| `y` | Pixels up from the floor | Where the face's bottom (or, for a top, the face itself) is. |
| `z` | Pixels back from the model's front | Where the face stands, or where a top's front edge is. |

Two faces on the same plane can't cover the same spot, since they would flicker. Two fronts at one `z` or two tops at one `y` count as the same plane, and `Build` throws if they overlap. Faces on one plane may sit side by side, and faces on different planes may touch.

The same PNG used for more than one face is loaded once, so repeating a picture costs nothing extra. That holds for the same name and for the same byte array.

## Size and the Pixel Scale

A model made with `Build` keeps its faces in pixels and turns them into world units when the mesh is made. The game's camera looks down at about 53 degrees, so a pixel is a different length in each direction.

| Direction | Pixels per world unit |
| --- | --- |
| Across (x) | 50 |
| Up (y) | 30 |
| Deep (z) | 40 |

A point further back therefore sits higher on screen. A round thing lying on the ground is about 5 pixels wide to every 4 deep, so draw a round basin as a `Top` that is wider than it is tall.

`model.Size` is the model's width, height and depth in world units, and `model.Name` is its name. The splitting block above is 44 pixels across (0.88), 34 up (about 1.13) and 26 deep (0.65).

```csharp
var size = block.Size;
```

A model is centered across and deep on the object it stands on, with its floor at the ground. That is why a station can take it without a size, and why a part sits on its world object's origin. The game also snaps buildings to a grid of cells 0.32 across and 0.3 deep, which is 16 by 12 pixels. A slot has to line up with those cells (see [Slots](stations-and-crafts.md#slots)).

## Pixel Art Rules

Core only enforces the overlap rule and warns about semi-transparent pixels. The rest is advice from making the art for Timbn's own mods.

- Draw each picture as the player sees it. There is nothing to rotate or project, and the camera never shows a side or a back.
- Keep every pixel fully solid or fully clear. Semi-transparent pixels can cause rendering artifacts, and `Build` logs a warning with the count found in each picture.
- Offset coplanar faces by one pixel to prevent flickering (Z-fighting). For example, place a glow face one pixel closer to the camera than the front face, and lines drawn on a lid one pixel above it. A shadow on the ground goes at `y` 1, under whatever stands on it.
- Keep a rounded shape one pixel in from every edge of its picture, since a shape that touches the edge can lose its outline there.
- Check colors in-game, because the game's lighting can shift them.
- Avoid a lone light pixel on a dark area, which tends to read as a speck.
- Draw the pieces of a set from the same front, bottom, left corner, so a station and its looks line up.

## Using a Model

### For a station

`Stations.Add` takes a model for the finished station, its construction site and each facing. Pass a model to `Model` with no size and the footprint is the model's own, or pass a size of your own to make the footprint different from the drawing.

```csharp
var oven = Models.Build("ClayOven", faces => faces
    .Block("oven_body", x: 0, y: 0, z: 0, depth: 24));
var ovenSite = Models.Build("ClayOvenSite", faces => faces
    .Top("oven_site", x: 0, y: 1, z: 0));

Stations.Add("timbn_clay_oven", "Clay Oven", "A squat oven for baking bread.", station => station
    .Model(oven)
    .Site(ovenSite)
    .BasedOn("kitchen_oven"));
```

Without a `Site`, a station gets a plain outline of its footprint with the game's warning sign. Extra models that show while a condition holds, and a model for each facing the player can turn the station to, are covered in [Facings](stations-and-crafts.md#facings) and [Looks That Change](stations-and-crafts.md#looks-that-change). Each extra model lines up with the station's own model, whatever their sizes. The full list of station calls is in [Adding a Station](stations-and-crafts.md#adding-a-station).

### For a bench extension

`Stations.AddExtension` always needs a size, since it has no `Model(model)` on its own. Pass the model's own size to give it the footprint of the drawing.

```csharp
var rack = Models.Build("ChoppingRack", faces => faces
    .Front("rack_tools", x: 0, y: 0, z: 0));

Stations.AddExtension("timbn_chopping_rack", "Chopping Rack", "A rack of sharp tools.", extension => extension
    .Model(rack, rack.Size)
    .Cost(("flitch", 2)));
```

See [Bench Extensions](stations-and-crafts.md#bench-extensions) for the rest.

### For a part on a world object

#### `Models.AddPart(partId, model, solid)`

Makes a model an extra part that the game hangs on a world object with its `AddWgoPart` expression, the way a headstone or a fence goes on a grave. `partId` is the id that `AddWgoPart` and `RemoveWgoPart` use, so prefix it with your mod, for example `"timbn_grave_bot_1"`. Registering an id twice replaces the first.

The part sits at the world object's origin, and `AddPart` has no offset of its own. A model is centered on the origin, so place its faces in the picture to move it, for example a headstone toward the back of the grave.

A part is walked through unless it gets a `solid` box, which is optional. The box is in the world object's space, with a center and a size. A fence needs one, since a grave hides its own mound and collision while a fence is on it and expects the fence to bring both.

```csharp
var border = Models.Build("StoneBorder", faces => faces
    .Top("border_stones", x: 0, y: 1, z: 0));

Models.AddPart("timbn_grave_bot_1", border,
    new Bounds(new Vector3(0f, 0.3f, 0.075f), new Vector3(0.6f, 0.6f, 1f)));
```

That box covers a whole grave. `AddPart` returns an `IDisposable` that removes the part's look early. You don't need to keep it, since the part goes with the mod anyway.

A stone with no solid box is the simpler case.

```csharp
var cross = Models.Build("StoneCross", faces => faces
    .Front("cross", x: 0, y: 0, z: 0));

Models.AddPart("timbn_grave_top_1", cross);
```

> [!NOTE]
> A part hung on a world object is kept out of the game's save while the game writes it and put back when the save loads, so the save still loads once the mod is gone. See [Removing a Mod Safely](removing-a-mod.md).

#### A headstone

A decoration is an item that sits in the grave plus a part with the same id, so everything except the model is copied from a game stone or fence. This mod adds one headstone, copied from the game's `grave_top_stn_1`, and one model for it.

```csharp
protected override void OnAwake()
{
    const string id = "timbn_grave_top_slate";
    const string set = "set_" + id;
    const string remove = "rem_" + id;
    const string scaffolding = "grave_prebuild_top";

    var marker = Models.Build("SlateMarker", faces => faces
        .Front("slate_marker", x: 0, y: 0, z: 0));
    Models.AddPart(id, marker);

    Sprites.AddEmbeddedPng(id);
    Text.Add(new Dictionary<string, string>
    {
        [set] = "Slate Marker",
        [remove] = "Remove",
    });

    // The make craft below supplies the item's name and description, since both use the item's id.
    Balance.AddItem("grave_top_stn_1", id, "", "", id, item => item.quality = 4);
    Balance.AddCopy<WGODef>("grave_top_stn_1", id);

    Crafts.Add(id, "Slate Marker", "A plain slate that stands at the head of a grave.", craft => craft
        .BasedOn("grave_top_stn_1")
        .Makes(id)
        .Icon(id));

    Balance.AddCopy<CraftDef>("set_grave_top_stn_1", set, craft =>
    {
        craft.needItems = [new NeedItemData(id, 1)];
        craft.iconId = id;
        craft.customCraftResultIcon = TimbnBalance.TextExpression(id);
        craft.onCraftStartExpressions = [TimbnBalance.Expression($"AddWgoPart(\"{scaffolding}\")")];
        craft.onCraftEndExpressions =
        [
            TimbnBalance.Expression($"AddWgoPart(\"{id}\")"),
            TimbnBalance.Expression($"RemoveWgoPart(\"{scaffolding}\")"),
        ];
    });

    Balance.AddCopy<CraftDef>("rem_grave_top_stn_1", remove, craft =>
    {
        craft.dropFromWgoItemsEnd = [new NeedItemData(id, 1)];
        craft.iconId = id;
        craft.customCraftResultIcon = TimbnBalance.TextExpression(id);
        craft.onCraftEndExpressions = [TimbnBalance.Expression($"RemoveWgoPart(\"{id}\")")];
    });
}
```

The item, the `WGODef` and the make craft all use the stone's id. The `set_` craft puts the item in the grave, shows the game's scaffolding while it runs and hangs the part when it ends. The `rem_` craft drops the item and takes the part off. The game finds both by the item's id, so the names have to be `set_<id>` and `rem_<id>`. A fence is the same with `grave_bot_stn_1` as the template, `grave_prebuild_bot` as the scaffolding and a `solid` box on its part.

Core's [`TimbnBalance`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnBalance) builds the expressions (see [Changing the Game](changing-the-game.md)), and [Stations and Crafts](stations-and-crafts.md) covers `Crafts.Add` and `Balance.AddItem`.

The `AddWgoPart` expression is the game's, so a part can be hung on any world object whose craft or code calls it, but the grave is the place the game does it for you.

## When the Mod Unloads

Everything you make through `Models` belongs to your mod and goes away when it unloads.

- A model is destroyed, with its textures and meshes.
- A part is unregistered, and the world objects drawing it let go of it.

When the same part comes back after a reload (F6 with the mod in `BepInEx/scripts`), the objects already in the world are drawn again with the new model, so a placed station stays visible. Models do not survive the unload, so keep them in locals or on your mod instance and make them again in `OnAwake`. Don't keep one in a static field, since a reload makes a fresh model.

## Gotchas

- A missing picture throws when `Build` runs. An unhandled exception in `OnAwake` aborts mod loading, so nothing after that line is added. The resource is looked up by the exact name plus ".png", and the name is case sensitive. If the file is embedded without `LogicalName="%(Filename)%(Extension)"` its resource name carries the project's namespace and won't be found.
- A model needs at least one face, and every position is zero or more. Start from the model's front, bottom, left corner at 0 and place everything else from it.
- Two fronts at one depth or two tops at one height can't overlap. Move one a pixel, or put it on its own `z` or `y`.
- A `Block` is both faces, so its PNG has to be taller than its `depth`. The top takes the upper rows and the front needs at least one more.
- Semi-transparent pixels can cause rendering artifacts and log a warning. Keep every pixel solid or clear.
- A part with no `solid` box can be walked through. A fence has to bring its own box and its own mound in the model, since the grave hides both.
- A model is not placed by itself. If nothing appears in the game, check that a station, extension or part actually uses it.
