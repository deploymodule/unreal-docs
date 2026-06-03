# Object Mixer Creator — Usage

## Installation

1. Copy the `ObjectMixerCreator` plugin folder into your project's `Plugins/` directory, or install it from the Fab marketplace.
2. Open your project in the Unreal Editor.
3. Navigate to Edit > Plugins, search for "Object Mixer Creator", and enable it.
4. Restart the editor when prompted.

Object Mixer (the built-in UE5 tool that consumes the filters) is available without any additional installation. It ships with all UE5 builds and does not require a separate enable step.

!!! note
    Both Object Mixer Creator and the built-in Object Mixer are editor-only tools. Neither appears in packaged builds, and enabling the plugin does not add any runtime modules to your project.

## Opening the Panel

Go to Main Menu > Window > "Object Mixer Filter Creator". The panel opens as a nomad dockable tab. You can dock it alongside any other editor panel or leave it floating.

The panel retains its last-used mode (Create New or Update Existing) and filter name across editor sessions.

## Creating a Filter

The following steps walk through producing a new filter from scratch using Create New mode.

1. Switch to **Create New** mode using the toggle at the top of the panel (it is the default when no other mode has been selected).
2. In the viewport, select the actors whose classes you want to include in the filter. Then click **Add from Selection** in the panel. The class list populates automatically with one entry per unique class found on the selected actors.
3. If you need a class that is not currently placed in the level, click **Add Class...** to open the UE class picker. The picker is filtered to `AActor` subclasses by default, or to a narrower base class if you have set `BaseClassRestriction` in Project Settings.
4. In the class list, click the expand arrow on any class to reveal its editable properties. Properties are grouped by their Details panel category.
5. Use the **search box** above the property tree to filter by name. Type any part of a property's display name to narrow the list.
6. **Check** the properties you want exposed as columns in Object Mixer. You can use **Check All** to select every currently-visible property, or **Uncheck All** to clear the selection.
7. Repeat steps 4–6 for each class in the list.
8. Type a name into the **Filter Name** field. This becomes the class name prefix for both generated assets (e.g., entering `SceneLights` produces `USceneLightsFilter` and `SceneLightsConfig`).
9. Click **Create**. The plugin writes the Blueprint and DataAsset to the output path, then syncs the Content Browser to both new assets.

!!! tip
    You can add multiple classes from different actor types in a single filter. Object Mixer will show all actors that match any of the listed classes, and all checked properties will be available as columns regardless of which class declares them.

!!! warning
    If a filter with the same name already exists at the output path, the Create operation will fail with a naming conflict error. Rename the filter or use Update Existing mode instead.

## Searching Properties

The search box performs a live, case-insensitive substring match against the display name of every property currently loaded in the tree. As you type, categories with no matching properties are collapsed automatically. The match runs against the human-readable display name, not the internal `FName`.

"Check All" and "Uncheck All" respect the current search filter — they operate only on the properties visible after the search is applied. To select all properties across the entire class, clear the search box first.

## Updating an Existing Filter

Use Update Existing mode when you need to change the classes or property columns in a filter that was previously created.

1. Switch to **Update Existing** mode using the toggle at the top of the panel.
2. The **Filter** dropdown auto-populates with all filters found at the configured output path. Select the filter you want to modify.
3. The panel loads the current configuration from the filter's config DataAsset. The class list and property selections reflect the saved state.
4. Add or remove classes using **Add from Selection** or **Add Class...**, and change property selections as needed.
5. Click **Update**. Only the config DataAsset is written to disk. The Blueprint is not modified and does not need to recompile.

Because the generated Blueprint reads its data entirely from the config DataAsset at runtime, the updated columns and classes take effect immediately the next time Object Mixer evaluates the filter — no editor restart required.

!!! note
    If the filter you want to update does not appear in the dropdown, verify that its assets are located at the path configured under Project Settings > Plugins > Object Mixer Creator > Output Path.

## Deleting a Filter

In Update Existing mode, select the filter you want to remove from the **Filter** dropdown, then click **Delete**. The plugin deletes both the filter Blueprint and its config DataAsset. The Content Browser reflects the deletion immediately.

!!! warning
    Deletion is permanent and not undoable through the standard editor undo stack. Confirm that the filter is no longer referenced before deleting it.

## Configuring Output Settings

Open Edit > Project Settings > Plugins > Object Mixer Creator to configure the plugin's behavior.

| Field | Description |
|-------|-------------|
| **Output Path** | Package path where generated Blueprints and DataAssets are written (e.g., `/Game/ObjectMixerFilters`). This folder is also scanned to populate the Update Existing dropdown. Change this if you want to organize filters under a different content folder. |
| **Default Filter Name** | The value pre-filled in the Filter Name field when the panel opens in Create New mode. Set this to a team convention such as your project's asset prefix. |
| **Base Class Restriction** | Restricts the "Add Class..." picker to subclasses of this class. For example, setting this to `ALight` prevents the picker from showing non-light actor classes. This setting does not affect "Add from Selection". |
| **Include Superclass Properties** | When enabled, the property tree shows properties declared on parent classes as well as the directly selected class. Useful when the properties you need are declared on a base class like `AActor` or `ALight`. |
| **Include EditConst Properties** | When enabled, read-only (`EditConst`) properties are included in the property tree. These are shown for inspection purposes in Object Mixer but cannot be edited through it. |

## Using the Filter in Object Mixer

Object Mixer Creator only creates and manages filter assets. It does not open or control Object Mixer. To use a generated filter:

1. Go to Main Menu > Window > **Object Mixer** to open the built-in Object Mixer panel.
2. In the Object Mixer toolbar, click the **Filter** dropdown (labeled "Select a Filter Class" if no filter is active).
3. Find and select your generated filter class (e.g., `SceneLightsFilter`). The class will be listed under the content path you configured.
4. Object Mixer populates the actor list with all scene actors matching the filter's classes and shows the property columns you selected.

!!! tip
    If your generated filter does not appear in the Object Mixer filter dropdown immediately after creation, try clicking the refresh button in the dropdown or reopening the Object Mixer panel. The asset registry may need a moment to register the newly created Blueprint.
