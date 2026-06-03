# Gaffer — API Reference

## Enums

All enums are declared in `Gaffer/Public/Types/GafferTypes.h`.

### EGafferKeyMode

Controls which frame or frames receive a key when a diamond button is pressed or auto-key fires.

```cpp
enum class EGafferKeyMode : uint8
{
    CurrentFrame,
    FirstFrameOnly,
    MatchingValue,
    AllSelected,
};
```

| Value | Description |
|-------|-------------|
| `CurrentFrame` | Creates or updates a key at the current Sequencer playhead position |
| `FirstFrameOnly` | Creates or updates a key at frame 0 of the active sequence, regardless of playhead |
| `MatchingValue` | Searches all existing keys on the track and updates any whose value matches the current property value |
| `AllSelected` | Keys all actors currently selected in the Gaffer tree at the playhead position in a single operation |

### EGafferInterpMode

Controls the tangent type assigned to newly created keys.

```cpp
enum class EGafferInterpMode : uint8
{
    Cubic,
    Linear,
    Constant,
};
```

| Value | Tangent Type | Description |
|-------|--------------|-------------|
| `Cubic` | Auto-spline | Smooth interpolation with auto-computed tangents |
| `Linear` | Linear | Straight-line segments between consecutive keys |
| `Constant` | Step / hold | Value snaps at each key with no interpolation between keys |

### EGafferTrackTarget

Controls which Sequencer track receives the key when an actor has multiple tracks for the same property.

```cpp
enum class EGafferTrackTarget : uint8
{
    First,
    Last,
    Selected,
};
```

| Value | Description |
|-------|-------------|
| `First` | Writes to the first matching track in the sequence |
| `Last` | Writes to the last matching track in the sequence |
| `Selected` | Writes only to the track that is currently selected in the Sequencer panel |

---

## Classes

### GafferKeyframeState

`GafferEditor/Public/Slate/Inspector/GafferKeyframeState.h`

Shared state object passed through the entire inspector hierarchy. All property rows and the settings bar read from and write to the same instance so that key mode, interpolation, track target, and auto-key stay synchronized.

| Member | Type | Description |
|--------|------|-------------|
| `KeyMode` | `EGafferKeyMode` | Active key mode for all subsequent key operations |
| `InterpMode` | `EGafferInterpMode` | Active interpolation mode for newly created keys |
| `TrackTarget` | `EGafferTrackTarget` | Active track targeting for key writes |
| `bAutoKeyEnabled` | `bool` | When true, every value change triggers an immediate key operation |

### SGafferKeyframePropertyRow

`GafferEditor/Public/Slate/Inspector/SGafferKeyframePropertyRow.h`

Slate widget that wraps a single keyframeable property. Constructed by `SGafferInspector::MakeKeyframePropertyRow`.

| Argument | Type | Description |
|----------|------|-------------|
| `Label` | `FText` | Display label for the property |
| `ValueWidget` | `TSharedRef<SWidget>` | Spin box, color picker, or check box for the property value |
| `KeyState` | `TSharedRef<GafferKeyframeState>` | Shared keyframe state reference |
| `OnKeyPressed` | `FSimpleDelegate` | Called when the diamond button is clicked |
| `GetIndicatorState` | `TAttribute<EGafferKeyIndicator>` | Attribute returning the left-circle indicator state |

### SGafferInspector

`GafferEditor/Public/Slate/Inspector/SGafferInspector.h`

Abstract base class for `SGafferCameraInspector` and `SGafferLightInspector`. Provides the shared infrastructure for building inspector UIs.

| Function | Signature | Description |
|----------|-----------|-------------|
| `MakeKeyframePropertyRow` | `TSharedRef<SWidget> MakeKeyframePropertyRow(FText Label, TSharedRef<SWidget> ValueWidget, FSimpleDelegate OnKey, TAttribute<EGafferKeyIndicator> Indicator)` | Constructs and returns a configured `SGafferKeyframePropertyRow` |
| `BuildKeyframeSettingsBar` | `TSharedRef<SWidget> BuildKeyframeSettingsBar()` | Builds the key mode / interpolation / track target / auto-key toolbar |
| `BuildStatusRow` | `TSharedRef<SWidget> BuildStatusRow(AActor* Actor)` | Builds the status row with class icon, actor label, and optional Bound pill |
| `AutoKeyProperty` | `void AutoKeyProperty(AActor* Actor, FName PropertyName, float Value)` | Checks `bAutoKeyEnabled` and calls `GafferSequencerBridge::KeyProperty` if true |

---

## GafferSequencerBridge

`GafferEditor/Private/Bridge/GafferSequencerBridge.cpp`

The sequencer bridge is the sole point of contact between the inspector UI and Unreal's Sequencer/MovieScene API. All key writes go through `KeyProperty`.

### KeyProperty

```cpp
static void KeyProperty(
    AActor*               Actor,
    FName                 PropertyName,
    EGafferKeyMode        KeyMode,
    EGafferInterpMode     InterpMode,
    EGafferTrackTarget    TrackTarget
);
```

| Parameter | Description |
|-----------|-------------|
| `Actor` | The scene actor whose property is being keyed |
| `PropertyName` | Sequencer property path string (e.g. `"CurrentFocalLength"`) |
| `KeyMode` | Which frame(s) to key; see `EGafferKeyMode` |
| `InterpMode` | Tangent type for the new key; see `EGafferInterpMode` |
| `TrackTarget` | Which track to write to when multiple exist; see `EGafferTrackTarget` |

`KeyProperty` performs the following steps internally:

1. Obtains the currently focused `ULevelSequence` from the Sequencer subsystem.
2. Finds the binding for `Actor` in the sequence.
3. Resolves the target track using `TrackTarget`.
4. Determines the target frame(s) using `KeyMode` and the current playhead position.
5. Calls `AddCubicKey`, `AddLinearKey`, or `AddConstantKey` depending on `InterpMode`.
6. Marks the sequence dirty so the editor knows it has unsaved changes.

!!! note
    If no sequence is open or the actor is not bound, `KeyProperty` returns silently without error. Check the Bound pill in the status row before attempting to key.

---

## GafferPropertySet

`Gaffer/Public/GafferPropertySet.h`

Data model class that describes which properties on a given actor class are available for keyframing. Subclass this and override `GetProperties` to expose custom properties.

| Function | Signature | Description |
|----------|-----------|-------------|
| `AddProperty` | `void AddProperty(FGafferPropertyDescriptor Descriptor)` | Registers a property descriptor at runtime. Calls `Modify()` internally. |
| `RemoveProperty` | `void RemoveProperty(FName PropertyName)` | Removes a previously registered descriptor by property name |
| `GetProperties` | `virtual TArray<FGafferPropertyDescriptor> GetProperties() const` | Returns all registered descriptors. Override in subclasses to declare a fixed set. |

### FGafferPropertyDescriptor

| Field | Type | Description |
|-------|------|-------------|
| `PropertyName` | `FName` | Sequencer property path string used to locate the track |
| `DisplayLabel` | `FText` | Human-readable label shown in the inspector row |

---

## GafferTagRegistry

`Gaffer/Public/GafferTagRegistry.h`

Manages the set of tags associated with scene actors. Tags are stored by actor GUID so they survive actor renames within the same session.

| Function | Signature | Description |
|----------|-----------|-------------|
| `AddTag` | `void AddTag(const FGuid& ActorGuid, FName Tag)` | Adds a tag to an actor; no-op if the tag already exists |
| `RemoveTag` | `void RemoveTag(const FGuid& ActorGuid, FName Tag)` | Removes a specific tag from an actor |
| `GetActorTags` | `TArray<FName> GetActorTags(const FGuid& ActorGuid) const` | Returns all tags currently assigned to the actor |
| `GetAllTags` | `TSet<FName> GetAllTags() const` | Returns the union of all tags across all actors (used to populate the filter dropdown) |
| `HasTag` | `bool HasTag(const FGuid& ActorGuid, FName Tag) const` | Returns true if the actor currently carries the specified tag |
| `ClearActor` | `void ClearActor(const FGuid& ActorGuid)` | Removes all tags from an actor |

---

## GafferBulkActions

`GafferEditor/Public/GafferBulkActions.h`

Provides static batch operations that operate on a supplied array of actors. All operations are undoable via the editor transaction system.

| Function | Signature | Description |
|----------|-----------|-------------|
| `KeyAllProperties` | `static void KeyAllProperties(TArrayView<AActor* const> Actors, EGafferKeyMode KeyMode, EGafferInterpMode InterpMode, EGafferTrackTarget TrackTarget)` | Keys every registered property on every actor in the array |
| `ResetAllProperties` | `static void ResetAllProperties(TArrayView<AActor* const> Actors)` | Resets all inspector properties on the selected actors to their class defaults |
| `AssignGroup` | `static void AssignGroup(TArrayView<AActor* const> Actors, FName GroupName)` | Adds all actors to the named group, creating the group if it does not exist |
| `RemoveFromGroup` | `static void RemoveFromGroup(TArrayView<AActor* const> Actors, FName GroupName)` | Removes all actors from the named group |
| `AddTag` | `static void AddTag(TArrayView<AActor* const> Actors, FName Tag)` | Adds a tag to every actor in the array |
| `RemoveTag` | `static void RemoveTag(TArrayView<AActor* const> Actors, FName Tag)` | Removes a tag from every actor in the array |
| `SelectInSequencer` | `static void SelectInSequencer(TArrayView<AActor* const> Actors)` | Selects the corresponding Sequencer bindings for all actors |

!!! tip
    `KeyAllProperties` combined with `AllSelected` key mode is the fastest way to capture a full lighting state at a specific frame — select all the lights you care about, then call this once.

---

## FGafferActorRegistry

`Gaffer/Public/GafferActorRegistry.h`

Singleton registry that tracks which actor classes appear in the Gaffer tree and which property set class each one uses.

| Function | Signature | Description |
|----------|-----------|-------------|
| `Get` | `static FGafferActorRegistry& Get()` | Returns the singleton instance |
| `RegisterClass` | `void RegisterClass(UClass* ActorClass, FText Category, FSlateIcon Icon)` | Adds an actor class to the tree under the given category |
| `UnregisterClass` | `void UnregisterClass(UClass* ActorClass)` | Removes an actor class from the tree |
| `RegisterPropertySet` | `void RegisterPropertySet(UClass* ActorClass, TSubclassOf<UGafferPropertySet> PropertySetClass)` | Associates a property set class with an actor class |
| `GetPropertySetClass` | `TSubclassOf<UGafferPropertySet> GetPropertySetClass(UClass* ActorClass) const` | Returns the registered property set class, or null if none registered |
| `GetRegisteredClasses` | `TArray<UClass*> GetRegisteredClasses() const` | Returns all currently registered actor classes |
