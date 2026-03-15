# Sleight — Usage

## Step 1: Enable the Plugin

**Edit > Plugins > Sleight** — enable and restart.

## Step 2: Create Card Data Assets

1. Content Browser > right-click > **Data Asset > SleightCardData**.
2. Fill in **Card Name**, **Cost**, **Keywords**, and **Effects**.
3. For each effect, pick an `ESleightEffectType` and fill in parameters (e.g., DealDamage: amount = 3, target = opponent).

## Step 3: Define Game Rules

1. Create a Blueprint subclass of `USleightRules_Deckbuilder` or `USleightRules_TCG`.
2. Override **GetStartingHandSize**, **GetMaxHandSize**, **GetResourcesPerTurn**, and **IsLegalPlay** as needed.
3. Assign the rules class in your Game Mode defaults.

## Step 4: Initialize a Game Session

=== "C++"
    ```cpp
    #include "Sleight/SleightGameStateComponent.h"

    USleightGameStateComponent* Sleight = GameState->GetComponentByClass<USleightGameStateComponent>();

    // Start a game for two players
    Sleight->InitializeGame(Player1Controller, Player2Controller, GameRulesClass);

    // Deal opening hands
    Sleight->DealOpeningHands();
    ```

=== "Blueprint"
    Call **Sleight > Initialize Game** on the Game State Component in **BeginPlay** or your match initialization event.

## Step 5: Play a Card

```cpp
// On the server (or single-player)
Sleight->PlayCard(PlayerController, CardDataAsset, TargetContext);
// This pushes the card's effects onto the SleightEffectStack
// Effects resolve at end-of-priority (call ResolveStack to trigger)
Sleight->ResolveStack();
```

## Step 6: Combat Phase

```cpp
// Declare attackers
Sleight->DeclareAttackers(AttackerController, AttackingCardIds);

// Declare blockers (opponent)
Sleight->DeclareBlockers(BlockerController, BlockAssignments);

// Resolve combat
Sleight->ResolveCombat();
```

## Step 7: Subscribe to Game Events

```cpp
Sleight->OnCardPlayed.AddDynamic(this, &AMyGameHUD::HandleCardPlayed);
Sleight->OnEffectResolved.AddDynamic(this, &AMyGameHUD::HandleEffectResolved);
Sleight->OnGameOver.AddDynamic(this, &AMyGameMode::HandleGameOver);
```

## Step 8: Custom Keywords (C++)

```cpp
// Register a custom keyword handler
Sleight->GetKeywordRegistry().RegisterKeyword(
    FGameplayTag::RequestGameplayTag("Keyword.Lifesteal"),
    FOnSleightKeywordProc::CreateUObject(this, &UMyRules::HandleLifesteal)
);
```
