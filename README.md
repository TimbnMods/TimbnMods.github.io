# timbn-core-docs

Documentation site for Timbn Core, built with [DocFX](https://dotnet.github.io/docfx/). The API reference is generated from Core's source and XML doc comments, and the guides are markdown files in `articles/`.

## Build

```bash
dotnet tool install -g docfx
```

```bash
docfx docfx.json --serve
```

Guides use a `###` heading for each member, with a short example under it. `[API reference](xref:<uid>)` links point at the generated pages, and `> [!NOTE]` or `> [!WARNING]` makes a callout. The look of the pages is tweaked in `template/public/main.css`, which `docfx.json` layers over the modern template.

The API reference pages are generated into `api/` and committed, since the GitHub Action on push to main can only run `docfx build` (it has no game DLLs). Run the full `docfx docfx.json` after Core changes and commit the new `api/` files. The prose says mod, not plugin. The site is written to `_site/`. It reads Core from `../Timbn.GraveyardKeeperTwo` (keep that checkout on `main`) and needs the game DLLs in its `lib/`, so it builds locally only. `_site/` contains plain static files and can be hosted anywhere.
