# Troubleshooting

Common issues and solutions across all PluginDepot plugins.

## General

### Plugin doesn't appear in the Plugins menu

- Ensure the plugin folder is in `YourProject/Plugins/`, not the engine's `Plugins/` directory.
- Check the `.uplugin` file is present and valid JSON.
- Look at the Output Log on startup for module load errors.

### Compile errors after enabling a plugin

- Regenerate project files (right-click `.uproject` → **Generate Visual Studio project files**).
- Clean and rebuild the solution.
- Ensure your engine version matches the plugin's declared `EngineVersion` in the `.uplugin`.

## Rendering Plugins (Vapor, Tonal, Lucent, Aether)

### No visual effect visible

- Confirm you're running **DX12 / SM6** (`-dx12` launch argument or `r.RHI=D3D12`).
- Check the relevant CVars are non-zero (e.g., `r.Vapor.Enabled 1`).
- Make sure a **Post Process Volume** with Infinite Extent is present.

### PIE crash or GPU hang

- See the [LDR PIE fix](https://github.com/DeployModule/plugin-depot) notes — rendering plugins require the FloatRGBA UAV + blit pattern in LDR PIE.

## Gameplay Plugins

### Blueprint nodes missing

- Confirm the plugin module is listed in your `Build.cs` `PublicDependencyModuleNames`.
- Restart the editor after enabling the plugin.

### Data Asset properties not saving

- Ensure the Data Asset class matches the expected base class for the plugin (e.g., `UArsenalItemData` for Arsenal).

## Getting Help

!!! question "Still stuck?"
    Open an issue on [GitHub](https://github.com/DeployModule/plugin-depot/issues) with your UE version, plugin version, and Output Log snippet.
