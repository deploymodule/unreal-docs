# Sleight — API Reference

## USleightGameStateComponent

| Function | Signature | Description |
|---|---|---|
| `InitializeGame` | `void (APlayerController* P1, APlayerController* P2, TSubclassOf<USleightGameRules>)` | Initializes the game session with two players and a rules class. |
| `DealOpeningHands` | `void ()` | Deals opening hands according to `GetStartingHandSize` in the rules. |
| `PlayCard` | `void (APlayerController*, USleightCardData*, FSleightTargetContext)` | Plays a card, pushing its effects onto the effect stack. |
| `ResolveStack` | `void ()` | Resolves all effects currently on the stack in order. |
| `DeclareAttackers` | `void (APlayerController*, TArray<FGuid> CardIds)` | Declares attackers for the current combat phase. |
| `DeclareBlockers` | `void (APlayerController*, TMap<FGuid, FGuid> Assignments)` | Assigns blockers to attackers. |
| `ResolveCombat` | `void ()` | Executes the full combat resolution sequence. |
| `DrawCard` | `void (APlayerController*, int32 Count)` | Draws `Count` cards from the player's deck to hand. |
| `DiscardCard` | `void (APlayerController*, FGuid CardId)` | Moves a card from hand to graveyard. |
| `GetZoneContents` | `TArray<FGuid> (APlayerController*, ESleightZone Zone)` | Returns the card IDs currently in a given zone. |

### Delegates

| Delegate | Parameters | Description |
|---|---|---|
| `OnCardPlayed` | `APlayerController*, USleightCardData*` | Fired when a card is successfully played. |
| `OnEffectResolved` | `FSleightEffect` | Fired after each effect resolves from the stack. |
| `OnZoneChanged` | `ESleightZone, TArray<FGuid>` | Fired when a zone's contents change. |
| `OnCombatPhaseChanged` | `ESleightCombatPhase` | Fired when the combat phase advances. |
| `OnGameOver` | `APlayerController* Winner` | Fired when win/loss conditions are met. |

## USleightCardData

| Property | Type | Description |
|---|---|---|
| `CardName` | `FText` | Display name. |
| `Cost` | `int32` | Resource cost to play. |
| `Keywords` | `FGameplayTagContainer` | Keyword tags (Haste, Trample, Deathtouch, etc.). |
| `Effects` | `TArray<FSleightEffect>` | Effects applied when the card is played. |
| `Attack` | `int32` | Combat attack value (0 if non-combat card). |
| `Defense` | `int32` | Combat defense/toughness value. |
| `ArtworkRef` | `TSoftObjectPtr<UTexture2D>` | Soft reference to card artwork texture. |

## FSleightEffect (Struct)

| Field | Type | Description |
|---|---|---|
| `EffectType` | `ESleightEffectType` | Type of effect (DealDamage, Heal, Draw, Discard, Buff, Debuff). |
| `TargetType` | `ESleightTargetType` | Who it targets (Self, Opponent, TargetCard, AllCards, etc.). |
| `Magnitude` | `int32` | Numeric value (damage amount, heal amount, etc.). |
| `ConditionTag` | `FGameplayTag` | Optional condition tag that must be active for the effect to resolve. |

## ESleightZone

| Value | Description |
|---|---|
| `Deck` | Draw pile. |
| `Hand` | Cards in hand. |
| `Battlefield` | Cards in play. |
| `Graveyard` | Discard pile. |
| `Exile` | Removed from game. |
