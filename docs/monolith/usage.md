# Monolith — Usage

## Installation

1. Open your project in the Unreal Editor.
2. Go to **Edit > Plugins**.
3. Search for **Monolith** under the **Virtual Production** category.
4. Enable the checkbox.

!!! note
    Monolith does not require an editor restart when you first enable it. The plugin's menus register through `UToolMenus` on module load, which happens immediately when the plugin is activated.

Because Monolith is editor-only, you do not need to add it as a dependency in your project's `Build.cs` unless you are writing C++ code that calls into `MonolithEditor` APIs directly. For most production workflows, enabling the plugin is sufficient.

---

## Accessing Utilities

### From the World Outliner

1. Select one or more actors in the World Outliner or the viewport.
2. Right-click the selection to open the context menu.
3. Hover over **Monolith** to expand the category list.
4. Hover over the category you need to see its operations.
5. Click the operation to execute it immediately.

### From the Content Browser

1. Select one or more assets in the Content Browser.
2. Right-click the selection to open the context menu.
3. Hover over **Monolith** to access the asset-level categories: **Materials**, **Dependencies**, and **Placement**.

!!! tip
    You can select actors using the standard Unreal selection tools — Ctrl+click for additive selection, box-select in the viewport, or filter by class in the Outliner — before opening the Monolith menu. All operations act on whatever is selected at the moment you invoke them.

---

## Enabling and Disabling Categories

If your workflow only uses a subset of Monolith's categories, disable the rest to keep the context menu compact.

1. Go to **Edit > Project Settings**.
2. Under **Plugins**, select **Monolith Utilities**.
3. Toggle the boolean checkboxes for each category group.

Changes take effect immediately. Disabled categories are removed from the context menu without an editor restart. The settings are saved to your project's `Config/DefaultGame.ini` and are checked into source control along with the rest of your project config.

---

## Creating a Config DataAsset

The default numeric values built into Monolith (grid spacing, randomization ranges, sequence number formats, etc.) are conservative defaults. For most productions you will want to create a project-specific config.

1. In the Content Browser, navigate to a suitable folder (for example, `Content/Settings`).
2. Right-click and choose **Miscellaneous > Data Asset**.
3. In the class picker, search for and select **MonolithConfig**.
4. Name the asset (for example, `DA_MonolithConfig`) and save it.
5. Open the asset and fill in the properties relevant to your scene scale.
6. Go to **Edit > Project Settings > Plugins > Monolith Utilities**.
7. Assign your new DataAsset to the **Config** field.

From this point on, all operations use the values from your DataAsset instead of the built-in defaults.

!!! tip
    You can create multiple `UMonolithConfig` DataAssets for different stages of production — for example, one with tight grid spacing for a prop-dressing pass and another with wide randomization ranges for scattering foliage — and swap between them in Project Settings as needed.

---

## Organizing the Scene with Folders

Folders in the World Outliner help you group actors by department, shot, or function. Monolith's Folder category lets you create and populate folders without manually dragging actors one by one.

**Example: organize all lights into a dedicated folder**

1. In the World Outliner, use the **Filter** dropdown to show only light actors, or Ctrl+click each light you want to group.
2. Right-click the selection and go to **Monolith > Folders > Create Folder from Selection**.
3. Enter a folder name in the dialog (for example, `Lighting/KeyLights`).
4. All selected actors are moved into the new folder immediately.

**Organize an entire section by class automatically:**

1. Select a broad set of actors — for example, all actors on a set piece.
2. Right-click and choose **Monolith > Folders > Organize by Class**.
3. Monolith creates subfolders named after each actor class (StaticMeshActors, PointLights, SpotLights, etc.) and distributes the selection into them.

If you have a `UMonolithOrganizationPreset` assigned in Project Settings, the "Organize by Class" operation uses the class-to-folder mappings in that preset instead of generating names automatically.

---

## Batch Renaming

Consistent naming keeps the Outliner readable and makes automated pipelines (export scripts, Sequencer bindings, EDL matching) more reliable.

**Example: add a prefix to all static mesh actors in a set piece**

1. Select the mesh actors you want to rename.
2. Right-click and choose **Monolith > Naming > Add Prefix**.
3. Enter the prefix string in the dialog (for example, `SM_SetPiece_`).
4. All selected actor labels are updated immediately.

**Add sequential numbers to a group of props:**

1. Select the props.
2. Choose **Monolith > Naming > Add Sequence Numbers**.
3. Monolith appends zero-padded numbers (format configurable via `UMonolithConfig`) in the order actors appear in the selection.

**Find and replace across many actors:**

1. Select the actors whose labels need updating.
2. Choose **Monolith > Naming > Find and Replace**.
3. Enter the search string and the replacement string.
4. Monolith updates all labels that contain the search string.

!!! note
    All naming operations preserve the actor's Object name (the internal identifier used by Sequencer and other systems). Only the display label shown in the Outliner is changed.

---

## Arranging Actors

The Layout category is useful when placing sets, dressing crowds, or testing different spatial configurations quickly.

**Example: arrange a row of columns in a grid**

1. Select the column actors in the Outliner.
2. Right-click and choose **Monolith > Layout > Arrange in Grid**.
3. Enter the number of rows and columns and the X/Y spacing in the dialog.
4. The actors are repositioned into the grid, centered on the midpoint of the original selection's bounding box.

**Arrange actors in a circle:**

1. Select the actors.
2. Choose **Monolith > Layout > Arrange in Circle**.
3. Enter the radius. Monolith distributes the actors at equal angular intervals around the circle.

**Distribute actors evenly along an axis:**

If you have actors that are roughly lined up but unevenly spaced, use **Transform > Distribute Evenly** to equalize the spacing along a chosen axis without moving the first or last actor.

---

## Running an Audit

The Audit category helps catch common scene problems before they reach a build or review.

**Example: find actors with zero scale**

1. You do not need to pre-select anything.
2. Right-click any actor (or an empty area of the Outliner) and choose **Monolith > Audit > Find Zero Scale Actors**.
3. Monolith scans the entire level and selects all actors whose world scale is zero on any axis.
4. The results are also printed to the Output Log with actor names and locations.

**Find actors with broken material references:**

Choose **Monolith > Audit > Find Broken Material References**. Monolith selects all mesh actors that have a null or otherwise invalid material slot and logs each one.

**Find actors outside world bounds:**

Choose **Monolith > Audit > Find Actors Outside World Bounds**. This is useful for catching actors that have been accidentally translated far from the scene origin.

!!! warning
    Audit operations scan all actors in the persistent level by default. In levels with thousands of actors this can take a moment. The results are shown via Outliner selection and the Output Log — check both.

---

## Managing Visibility Groups

Visibility Groups let you toggle the visibility of a named set of actors with a single menu click. Unlike standard Outliner folders, visibility groups survive folder reorganization because they use actor tags as the backing store.

**Create a visibility group:**

1. Select the actors you want to group.
2. Right-click and choose **Monolith > Visibility Groups > Create Group from Selection**.
3. Enter a group name. Monolith assigns a tag in the format `MonolithGroup_[name]` to each selected actor.

**Toggle a group:**

Right-click any actor and choose **Monolith > Visibility Groups > Toggle Group > [group name]**. All actors carrying that group tag are hidden or shown.

**Show all groups:**

Choose **Monolith > Visibility Groups > Show All Groups** to make all visibility group members visible regardless of their current state.

**Select all members of a group:**

Choose **Monolith > Visibility Groups > Select Group > [group name]** to select all actors in the group in the Outliner and viewport.

---

## Batch Sequencer Binding

When setting up a cinematic sequence, you often need to bind a large number of actors at once and ensure they have transform tracks.

**Add actors to a sequence:**

1. Open the Level Sequence you want to bind actors to in Sequencer.
2. Select the actors in the Outliner.
3. Right-click and choose **Monolith > Level Sequence > Add to Sequence**.
4. Monolith creates Sequencer bindings for each selected actor and adds a transform track initialized to the actor's current world transform.

**Bake sequencer-evaluated transforms back to world:**

If you have actors driven by a Sequencer sequence and you want to "freeze" their evaluated positions as actual world transforms:

1. Scrub the Sequencer playhead to the frame you want to bake from.
2. Select the actors.
3. Choose **Monolith > Level Sequence > Bake to World Transform**.

**Set sequence default values from world transforms:**

If your actors have been repositioned in the world outside of Sequencer and you want their Sequencer default (section-zero) values to match:

1. Select the actors.
2. Choose **Monolith > Level Sequence > Set Defaults from World**.

---

## Exporting Dependencies

The Dependency utilities in the Content Browser help you understand what a level or asset folder depends on before packaging, archiving, or handing off to another team.

**Show level dependencies:**

1. Right-click the Level asset in the Content Browser.
2. Choose **Monolith > Dependencies > Show Level Dependencies**.
3. A report is printed to the Output Log with assets grouped by type (StaticMesh, Material, Texture, Sound, etc.).

**Find unused assets in a folder:**

1. Right-click a Content Browser folder.
2. Choose **Monolith > Dependencies > Find Unused Assets**.
3. Monolith compares assets in the folder against all level references and selects assets that are not referenced by any loaded level.

**Export dependency list to CSV:**

1. Right-click a Level or folder asset.
2. Choose **Monolith > Dependencies > Export to CSV**.
3. A save dialog appears. The exported CSV contains columns for Asset Name, Asset Type, Path, and Reference Count.

!!! tip
    Export the dependency CSV before major handoffs to give your pipeline team a complete manifest of what a scene requires.
