# Installation

All PluginDepot plugins follow the same installation pattern.

## Requirements

- Unreal Engine 5.5 or later (5.7 recommended)
- Windows 10/11, Linux, or macOS
- SM6 / DirectX 12 for rendering plugins

## Steps

### 1. Copy the Plugin Folder

Copy the plugin folder (e.g., `Vapor/`) into your project's `Plugins/` directory:

```
YourProject/
└── Plugins/
    └── Vapor/          ← paste here
        ├── Vapor.uplugin
        └── Source/
```

### 2. Enable in the Editor

1. Open **Edit → Plugins**
2. Search for the plugin name
3. Enable the checkbox
4. Restart the editor when prompted

### 3. Add to Build.cs

For C++ projects, add the module to your `Build.cs`:

```csharp
PublicDependencyModuleNames.AddRange(new string[]
{
    "Vapor",   // ← add the plugin module
});
```

### 4. Regenerate Project Files

Right-click your `.uproject` → **Generate Visual Studio project files**.

---

!!! tip
    Rendering plugins (Vapor, Tonal, Lucent, Aether) require **SM6 / DX12**. Confirm `r.RHI=D3D12` in your `DefaultEngine.ini`.
