# Your First Mod

This guide builds a small mod that shows a message at the start of every day, with a setting to turn it off. It uses a mod class, a setting, an event, and some text—representing the core components of most Timbn mods. It takes about ten minutes.

You need the .NET SDK, Graveyard Keeper 2 with [BepInEx for Graveyard Keeper 2](https://www.nexusmods.com/graveyardkeeper2/mods/48) installed, and Timbn Core in the game's `BepInEx/plugins` folder.

The project below is written out by hand so you can see every part. To skip it, run `dotnet new gk2mod -n MyFirstMod` (see [Getting Started](getting-started.md)) and carry on from The Mod Class.

## The Project

Make a folder and add a `MyFirstMod.csproj`. Change `GameDir` if the game is installed somewhere else.

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netstandard2.1</TargetFramework>
    <AssemblyName>MyFirstMod</AssemblyName>
    <Version>1.0.0</Version>
    <LangVersion>latest</LangVersion>
    <Nullable>enable</Nullable>
    <DebugType>embedded</DebugType>
    <ImplicitUsings>enable</ImplicitUsings>
    <GameDir>C:\Program Files (x86)\Steam\steamapps\common\Graveyard Keeper 2</GameDir>
    <RestoreAdditionalProjectSources>https://nuget.bepinex.dev/v3/index.json</RestoreAdditionalProjectSources>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="BepInEx.Core" Version="5.*" />
    <PackageReference Include="BepInEx.PluginInfoProps" Version="2.*" PrivateAssets="all" />
    <PackageReference Include="UnityEngine.Modules" Version="6000.3.9" IncludeAssets="compile" />
  </ItemGroup>

  <ItemGroup>
    <GameAssembly Include="Assembly-CSharp;Assembly-CSharp-firstpass;LazyBearTechnology;Sirenix.Serialization;Unity.Addressables;Unity.ResourceManager;Unity.TextMeshPro;UnityEngine.UI" />
    <Reference Include="@(GameAssembly)">
      <HintPath>$(GameDir)\GraveyardKeeper2_Data\Managed\%(Identity).dll</HintPath>
      <Private>false</Private>
    </Reference>
    <Reference Include="Timbn.GraveyardKeeperTwo.Core">
      <HintPath>$(GameDir)\BepInEx\plugins\Timbn.GraveyardKeeperTwo.Core\Timbn.GraveyardKeeperTwo.Core.dll</HintPath>
      <Private>false</Private>
    </Reference>
  </ItemGroup>

</Project>
```

`DebugType` embeds the debug symbols in the DLL, which a mod loaded from `BepInEx/scripts` needs. The game assemblies are listed because Core's calls use types from them, and the compiler asks for them as soon as your code touches one. `Private` is false so the game's and Core's DLLs are not copied into your output. `BepInEx.PluginInfoProps` writes a `MyPluginInfo` class from your project's name and version, which the mod class below reads.

## The Mod Class

Add a `Plugin.cs`.

```csharp
using BepInEx;
using BepInEx.Configuration;
using Timbn.GraveyardKeeperTwo.Core;
using Timbn.GraveyardKeeperTwo.Core.Framework;

namespace MyFirstMod;

[BepInPlugin(MyPluginInfo.PLUGIN_GUID, MyPluginInfo.PLUGIN_NAME, MyPluginInfo.PLUGIN_VERSION)]
[BepInDependency(TimbnCorePlugin.Guid, "2.0.0")]
public class Plugin : TimbnFrameworkPlugin<Plugin>
{
    internal static ConfigEntry<bool> Greet { get; private set; } = null!;

    protected override void BindConfig(ConfigFile config) =>
        Greet = config.Bind("General", "Greet", true, "Show a message at the start of every day.");

    protected override void OnAwake()
    {
        Text.Add("my_first_mod_hello", "Good morning, keeper!");

        Settings.While(Greet, () => Events.NewDayStarted(_ => TimbnUI.Notify("my_first_mod_hello")));
    }
}
```

Here is what each part does.

- `TimbnFrameworkPlugin<Plugin>` is the base class every Timbn mod uses. It gives you `Events`, `Settings`, `Text` and the other properties used below. See [Getting Started](getting-started.md).
- `BindConfig` runs before `OnAwake` and makes the setting players see in their config file or a config manager.
- `Text.Add` registers the message under a key. The game shows text by key, so a notification takes the key, not the words.
- `Events.NewDayStarted` runs your code at the start of every day. See [Game Events](events.md).
- `Settings.While` keeps the event hooked only while the setting is on, so switching it off in a config manager stops the message right away. See [Settings](settings.md).

You never undo any of this. Everything registered through those properties is owned by your mod and removed when it unloads.

## Build and Run

```bash
dotnet build -c Release
```

Copy `MyFirstMod.dll` from the output folder into `BepInEx/plugins/MyFirstMod/` in the game folder, then start the game and load a save. Sleep through a night and the message shows when the new day starts.

If you use BepInEx's ScriptEngine, put the DLL in `BepInEx/scripts` instead. Then you can rebuild and press F6 in-game to reload it without restarting the game.

## Where Next

- [Changing the Game](changing-the-game.md) for balance values, player control and game speed.
- [Save Data](saves.md) to keep your own data with a save.
- [Quests, Dialog, and Text](quests.md), [NPCs](npcs.md) and [Stations and Crafts](stations-and-crafts.md) for adding content.
- [Removing a Mod Safely](removing-a-mod.md) before you add items, buffs or stations.
