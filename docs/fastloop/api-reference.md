# FastLoop — API Reference

## UFastLoopBFL

`#include "FastLoopBFL.h"`

All functions are static Blueprint-callable nodes.

### Filter Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `FilterActorsByClass` | `(Actors, Class) → TArray<AActor*>` | Keep actors of given class |
| `FilterActorsByTag` | `(Actors, Tag) → TArray<AActor*>` | Keep actors with tag |
| `FilterActorsByDistance` | `(Actors, Origin, MaxDist) → TArray<AActor*>` | Keep actors within distance |
| `FilterFloatsAbove` | `(Values, Threshold) → TArray<float>` | Keep floats above threshold |
| `FilterFloatsBelow` | `(Values, Threshold) → TArray<float>` | Keep floats below threshold |
| `FilterFloatsInRange` | `(Values, Min, Max) → TArray<float>` | Keep floats in range |
| `FilterIntsEqual` | `(Values, Target) → TArray<int32>` | Keep matching integers |
| `FilterNamesContaining` | `(Names, Substring) → TArray<FName>` | Keep names containing substring |

### Sort Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `SortActorsByDistance` | `(Actors, Origin) → TArray<AActor*>` | Sort nearest first |
| `SortFloatsAscending` | `(Values) → TArray<float>` | Sort ascending |
| `SortFloatsDescending` | `(Values) → TArray<float>` | Sort descending |
| `SortIndicesDescending` | `(Values) → TArray<int32>` | Returns sorted indices |
| `SelectByIndices` | `(Actors, Indices, Count) → TArray<AActor*>` | Pick actors by index list |

### Chunk Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `ChunkActorArray` | `(Actors, ChunkSize) → TArray<TArray<AActor*>>` | Split into equal chunks |
| `ChunkFloatArray` | `(Values, ChunkSize) → TArray<TArray<float>>` | Split floats into chunks |
| `GetChunk` | `(Actors, ChunkSize, Index) → TArray<AActor*>` | Get one chunk by index |

### Map / Transform Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `MapActorsToNames` | `(Actors) → TArray<FName>` | Extract actor names |
| `MapActorsToLocations` | `(Actors) → TArray<FVector>` | Extract actor locations |
| `MapActorsTags` | `(Actors, TagIndex) → TArray<FName>` | Extract a tag by index |
| `MultiplyFloatArray` | `(Values, Scale) → TArray<float>` | Scale all floats |
| `AddFloatArrays` | `(A, B) → TArray<float>` | Element-wise addition |

### Reduce / Aggregate Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `SumFloatArray` | `(Values) → float` | Sum all values |
| `AverageFloatArray` | `(Values) → float` | Arithmetic mean |
| `MaxFloatArray` | `(Values) → float` | Maximum value |
| `MinFloatArray` | `(Values) → float` | Minimum value |
| `CountMatchingTag` | `(Actors, Tag) → int32` | Count actors with tag |
| `FindFirstActorWithTag` | `(Actors, Tag) → AActor*` | First match or null |
| `AnyActorWithTag` | `(Actors, Tag) → bool` | True if any match |
| `AllActorsWithTag` | `(Actors, Tag) → bool` | True if all match |
