# Courier — API Reference

## UCourierSubsystem

| Function | Signature | Description |
|---|---|---|
| `RequestLoad` | `FCourierHandle (FSoftObjectPath, ECourierPriority, FOnCourierLoaded)` | Requests an async load at the given priority. Returns a handle that keeps the asset loaded. |
| `RequestBatchLoad` | `FCourierBatchHandle (TArray<FSoftObjectPath>, ECourierPriority, FOnCourierBatchLoaded)` | Loads a group of assets and fires the callback when all complete. |
| `ReleaseHandle` | `void (FCourierHandle&)` | Explicitly releases a handle and decrements the reference count. |
| `GetLoadState` | `ECourierLoadState (FSoftObjectPath)` | Returns the current load state for a given asset path. |
| `IsLoaded` | `bool (FSoftObjectPath)` | Convenience: returns true if load state is `Loaded`. |
| `GetLoadedAsset` | `UObject* (FSoftObjectPath)` | Returns the loaded `UObject*` or null if not yet loaded. |
| `PreloadByTag` | `void (FGameplayTag, ECourierPriority)` | Preloads all assets tagged with the given GameplayTag via Asset Manager. |
| `CancelLoad` | `void (FSoftObjectPath)` | Cancels a pending load if no handles are alive. Safe to call if already loaded. |

## FCourierHandle

RAII handle returned by `RequestLoad`. Keeps the asset reference-counted while alive.

| Method | Description |
|---|---|
| `Release()` | Explicitly releases this handle. Equivalent to destroying the object. |
| `IsValid() const` | Returns true if the handle currently holds a reference. |
| `GetAsset() const` | Returns the loaded `UObject*` or null if not loaded yet. |

## FCourierBatchHandle

Wraps multiple `FCourierHandle` instances for batch loads.

| Method | Description |
|---|---|
| `ReleaseAll()` | Releases all handles in the batch simultaneously. |
| `GetProgress() const` | Returns a 0.0–1.0 normalized load progress for the batch. |
| `IsComplete() const` | Returns true when all assets in the batch are loaded. |

## ECourierPriority

| Value | Int | Description |
|---|---|---|
| `Low` | 1 | Speculative/background loads. |
| `Normal` | 2 | Standard background preload. |
| `High` | 3 | Nearby, about-to-be-visible content. |
| `Critical` | 4 | Must-load immediately (player-critical). |

## ECourierLoadState

| Value | Description |
|---|---|
| `Unloaded` | Asset is not in memory. |
| `Loading` | Load request is in flight. |
| `Loaded` | Asset is fully loaded and in memory. |
| `Failed` | Load attempt resulted in an error. |

## Delegates

| Delegate | Signature | Description |
|---|---|---|
| `FOnCourierLoaded` | `void (UObject* LoadedAsset)` | Called when a single asset load completes. |
| `FOnCourierBatchLoaded` | `void ()` | Called when all assets in a batch load complete. |
| `FOnCourierLoadFailed` | `void (FSoftObjectPath FailedPath)` | Called when an asset load fails. |
