# FastLoop — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"FastLoop"` to your `Build.cs`.

## 2. Filter Operations

```cpp
// C++: Filter actors within distance
TArray<AActor*> CloseActors = UFastLoopBFL::FilterActorsByDistance(AllActors, Origin, 800.f);

// C++: Filter floats above threshold
TArray<float> HighScores = UFastLoopBFL::FilterFloatsAbove(Scores, 0.75f);
```

In Blueprint, use the **FastLoop | Filter** category:
- `Filter Actors By Class`
- `Filter Actors By Tag`
- `Filter Actors By Distance`
- `Filter Floats Above / Below`
- `Filter Ints Equal / Not Equal`

## 3. Sort Operations

```cpp
// Sort actors by distance (nearest first)
TArray<AActor*> Sorted = UFastLoopBFL::SortActorsByDistance(Actors, Origin);

// Sort floats descending
TArray<float> Descending = UFastLoopBFL::SortFloatsDescending(Scores);
```

## 4. Chunk Operations

```cpp
// Split 200 actors into chunks of 50
TArray<TArray<AActor*>> Chunks = UFastLoopBFL::ChunkActorArray(AllActors, 50);

// Process one chunk per frame
int32 ChunkIndex = FrameNumber % Chunks.Num();
ProcessBatch(Chunks[ChunkIndex]);
```

## 5. Map / Transform Operations

```cpp
// Extract actor names
TArray<FName> Names = UFastLoopBFL::MapActorsToNames(Actors);

// Extract actor locations
TArray<FVector> Locations = UFastLoopBFL::MapActorsToLocations(Actors);

// Scale all floats
TArray<float> Scaled = UFastLoopBFL::MultiplyFloatArray(Scores, 100.f);
```

## 6. Reduce / Aggregate Operations

```cpp
float Sum    = UFastLoopBFL::SumFloatArray(Values);
float Avg    = UFastLoopBFL::AverageFloatArray(Values);
float Max    = UFastLoopBFL::MaxFloatArray(Values);
int32 Count  = UFastLoopBFL::CountMatchingTag(Actors, FName("Enemy"));
AActor* First = UFastLoopBFL::FindFirstActorWithTag(Actors, FName("Boss"));
```

## 7. Combined Pattern

```cpp
// Typical AI: find visible enemies, score them, pick top 3
TArray<AActor*> Visible = UFastLoopBFL::FilterActorsByTag(NearbyActors, FName("Enemy"));
TArray<float> Scores    = ScoreActors(Visible);
TArray<int32> Indices   = UFastLoopBFL::SortIndicesDescending(Scores);
TArray<AActor*> Top3    = UFastLoopBFL::SelectByIndices(Visible, Indices, 3);
```
