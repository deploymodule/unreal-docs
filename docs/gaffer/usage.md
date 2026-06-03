# Gaffer — Usage

## Installation

### Enable the Plugin

1. Open your project in the Unreal Editor.
2. Go to **Edit > Plugins**.
3. Search for **Gaffer** under the **Virtual Production** category.
4. Enable the checkbox and restart the editor when prompted.

### Add the Module Dependency (C++ Projects)

If your game module needs to call Gaffer's registration API at startup, add the module name to your `Build.cs`:

```cpp
// MyProject.Build.cs
PublicDependencyModuleNames.AddRange(new string[]
{
    "Gaffer"   // data model and registration API
});
```

You do not need to depend on `GafferEditor` from game code. If you are writing an editor utility that extends Gaffer's UI, add `GafferEditor` instead and gate the dependency behind `bBuildEditor`:

```cpp
if (Target.bBuildEditor)
{
    PrivateDependencyModuleNames.Add("GafferEditor");
}
```

!!! warning
    `GafferEditor` contains Slate headers. Do not add it as a public dependency of a runtime module.

---

## Opening Gaffer

With the plugin enabled, open the panel from the menu bar:

**Window > Gaffer**

The panel is dockable. Drag its tab to any editor window or leave it floating. The layout is saved with your editor layout profile and restored on next launch.

---

## Navigating the Actor Tree

The actor tree on the left side of the panel lists every registered actor in the current level, organized by category (Cameras, Lights, Environment, and any custom categories you have registered).

### Search

Type in the search box at the top of the tree to filter by actor label. The filter is case-insensitive and matches substrings.

### Filter by Group

Use the Group dropdown (next to the search box) to limit the tree to actors that belong to a specific named group. Select **All Groups** to clear the filter.

### Filter by Tag

Use the Tag dropdown to limit the tree to actors carrying a specific tag. Select **All Tags** to clear the filter.

### Selection

Click any actor row to select it and load its inspector on the right. Hold **Ctrl** and click additional rows to build a multi-selection. Multi-selection activates the Property Matrix view instead of the single-actor inspector.

!!! tip
    Right-click any actor row to open the context menu. From there you can add or remove the actor from groups, assign tags, add the actor to the current Working Set, or run bulk actions.

---

## Inspecting a Camera

1. Select a `CineCameraActor` (or any registered camera class) in the actor tree. The Camera Inspector loads on the right.
2. The **status row** at the top shows the camera's class icon, its actor label, and — if it is bound in the currently open Sequence — a green **Bound** pill.
3. Review the eight keyframeable properties: Focal Length, Aperture, ISO, Shutter Speed, Exposure Compensation, Focus Distance, Sensor Width, and Sensor Height.
4. Expand the **Advanced** section at the bottom of the inspector to access Blade Count.
5. Change a value by clicking into a spin box and typing, or by dragging left/right on the label. The value updates on the actor immediately.

!!! note
    Changes made through the inspector are undoable via **Ctrl+Z** like any other editor operation.

---

## Keyframing Properties

### Prerequisites

- A Level Sequence must be open in Sequencer.
- The actor must be bound in that sequence (the green Bound pill is visible in the status row). If the actor is not yet bound, add it to the sequence through the Sequencer **+ Track** button first.

### Setting the Key Mode

The keyframe settings bar sits above the inspector. Set the three dropdowns before pressing any key buttons:

| Control | Options | Effect |
|---------|---------|--------|
| Key Mode | CurrentFrame, FirstFrameOnly, MatchingValue, AllSelected | Determines which frame or frames receive the key |
| Interpolation | Cubic, Linear, Constant | Sets the tangent type of newly created keys |
| Track Target | First, Last, Selected | Chooses which Sequencer track to write to when multiple tracks exist for the same property |

### Pressing the Diamond Button

Click the diamond button on the right side of any property row to key that property with the current settings bar configuration. The left circle indicator on the row changes appearance to confirm a key was written.

### Using Auto-Key

Toggle **Auto Key** in the settings bar. While active, every value change you make — drag, type, or click — immediately creates a key at the Sequencer playhead using the current Key Mode, Interpolation, and Track Target settings. No button press is needed.

!!! warning
    Auto-key creates keys on every change, including intermediate drag values. If you are scrubbing a value to find the right number, disable auto-key first, set the final value, then re-enable it and press the diamond button once.

---

## Working with Groups

### Create a Group

1. Right-click any actor row in the tree.
2. Choose **Groups > New Group**.
3. Enter a name in the dialog that appears.

### Add Actors to a Group

1. Select one or more actors in the tree.
2. Right-click and choose **Groups > Add to Group > [group name]**.

### Edit Group Overrides

Click the group row (the colored bar at the top of the group in the tree) to select it. The inline group editor appears in the inspector panel:

| Field | Description |
|-------|-------------|
| Name | Editable text field for the group name |
| Color | Color swatch that opens a picker |
| Intensity Scale | Float multiplier applied to all lights in the group |
| Temperature Offset | Kelvin offset added to all lights in the group |
| Color Tint | Color multiplied into all lights in the group |

Changes to overrides take effect immediately on all actors in the group. The overrides are non-destructive: the original property values on each actor are unchanged; the group scaling is applied on top.

### Remove an Actor from a Group

Right-click the actor row and choose **Groups > Remove from Group > [group name]**.

---

## Managing Tags

### Add a Tag to an Actor

1. Right-click the actor row and choose **Tags > Add Tag**.
2. The tag input dialog opens — a floating window with a single text field.
3. Type the tag name and press **Enter** or click **Add**.

An actor can have any number of tags.

### Filter the Tree by Tag

Use the Tag dropdown above the actor tree. Select a tag name to show only actors carrying that tag. Select **All Tags** to remove the filter.

### Remove a Tag

Right-click the actor row and choose **Tags > Remove Tag > [tag name]**.

---

## Using Working Sets

A Working Set is a named snapshot of the current actor selection. Use Working Sets to switch quickly between groups of actors you work with frequently — for example, the key lights for scene A versus the practicals for scene B.

### Save a Working Set

1. Select the actors you want to include in the tree.
2. In the Working Sets bar (above the tree), type a name and click **Save**.

### Recall a Working Set

Click a saved Working Set name in the bar. Gaffer selects exactly those actors in the tree and loads the multi-actor Property Matrix if more than one actor is recalled.

### Delete a Working Set

Right-click a Working Set name in the bar and choose **Delete**.

---

## Registering a Custom Actor Class

Call `FGafferActorRegistry::Get().RegisterClass` from your module's `StartupModule`. The call must happen each editor session because the registry is in-memory.

```cpp
// MyEditorModule.cpp
#include "GafferActorRegistry.h"

void FMyEditorModule::StartupModule()
{
    FGafferActorRegistry::Get().RegisterClass(
        AMyVolumetricLight::StaticClass(),
        FText::FromString(TEXT("Volumetric Lights")),  // tree category label
        FSlateIcon(FMyStyle::StyleSetName, "Icons.VolumetricLight")
    );
}
```

To expose custom properties for keyframing from the inspector, create a `UGafferPropertySet` subclass, override `GetProperties`, and register it:

```cpp
// MyLightPropertySet.h
#pragma once
#include "GafferPropertySet.h"
#include "MyLightPropertySet.generated.h"

UCLASS()
class MYEDITORMODULE_API UMyLightPropertySet : public UGafferPropertySet
{
    GENERATED_BODY()
public:
    virtual TArray<FGafferPropertyDescriptor> GetProperties() const override;
};

// MyLightPropertySet.cpp
TArray<FGafferPropertyDescriptor> UMyLightPropertySet::GetProperties() const
{
    return {
        { TEXT("VolumetricScatteringIntensity"), FText::FromString(TEXT("Scatter Intensity")) },
        { TEXT("IndirectLightingIntensity"),     FText::FromString(TEXT("Indirect Intensity")) },
    };
}

// Registration (in StartupModule, after RegisterClass):
FGafferActorRegistry::Get().RegisterPropertySet(
    AMyVolumetricLight::StaticClass(),
    UMyLightPropertySet::StaticClass()
);
```

!!! note
    Property descriptor names must match the Sequencer property path strings exactly — the same strings you would see in a manually created Sequencer track for that property.
