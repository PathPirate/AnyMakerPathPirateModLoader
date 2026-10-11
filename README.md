# AnyMaker Mini Mod Loader

One `dinput8.dll` loads optional Catalog, Toolbox, and Lighting/Weather plugins. Each can run alone or with the others. The loader forwards DirectInput calls to Windows and provides shared hook installation and inventory state.

## Install or update

1. Close AnyMaker and back up existing mod DLLs.
2. Replace the previous root `dinput8.dll` with this loader. Remove the old Lighting `xinput9_1_0.dll` if it is our Lighting version; do not remove unrelated third-party DLLs. Running old and new copies together duplicates hooks.
3. Install each desired mod from its own release package by extracting it into the game's Mods folder. The loader package contains no mods.
4. Enable the plugins you want and restart the game.

```text
Anymaker/
  game.exe
  dinput8.dll
  Mods/
    Catalog/Catalog.dll + mod.ini
    Toolbox/Toolbox.dll + mod.ini
    LightingWeather/LightingWeather.dll + mod.ini
```

Manifests use `mod.ini` with a `[Mod]` section;
Each plugin's manifest contains:

```ini
[Mod]
Enabled=1
File=Toolbox.dll
Author=Justin
Version=1.0.0
```

Author and Version are optional descriptive fields, recorded in the existing startup log line. Version is the mod release version, separate from the loader API version; it does not enforce dependencies or compatibility. The initial loader editions use 1.0.0; Catalog 1.0.1 corrects pickup forwarding to Toolbox. Fill Author with the mod creator name.

Set `Enabled=0` or remove that plugin folder to disable it. Restart after changing plugins. Each immediate Mods subfolder is discovered independently; missing manifests, disabled plugins, directory links, and paths in File are skipped. Discovery order is unspecified. No plugin depends on Catalog being enabled.


The build does not modify the game installation. To revert, close the game, remove this loader and its plugin folders, and restore your backed-up DLLs. Preserve unrelated files.


## Adding future plugins

`Common/mod_plugin.h` defines API 2. Export `int WINAPI AnyMakerModInit(const AnyMakerModHost *host, HMODULE module)`. Validate host size, API version, and required services; copy the host before returning. Return zero for accepted startup and use a worker thread for initialization. Do not perform game discovery inside DllMain.

Use `install_hooks` to register a batch of target/replacement/original triples. It stores the previous handler in each original pointer before enabling hooks. Forward through that pointer with the game's exact calling convention and arguments. Later registrations run first; do not assume registration order. Never patch the same routine separately with your own hook library. A handler that intentionally consumes an operation must coordinate that behavior with other plugins.

Use `code_matches` to validate up to 32 original instruction bytes at an exact hook target. `get_inventory_state` returns -1 before fresh state is available, otherwise 0 or 1. `subscribe_frontend` registers a process-lifetime callback receiving frontend and inventory state, with at most eight distinct subscribers. Callbacks execute on the game thread before the original frontend routine; keep them short and do not subscribe from inside one. Hook registration supports up to 128 distinct targets. There is no hook removal or hot reload.

Use `log` for readiness and actionable failures. Plugins with initialization exports remain loaded until process exit even if initialization fails, to avoid dangling worker or hook code. Dependencies load from the plugin folder and Windows system directories.





