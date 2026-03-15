# Sleight — Final Architecture Plan

## Design Pillars

1. **Core is presentation-agnostic.** Logic never touches actors, widgets, or rendering.
2. **Data Assets are the API for designers.** Any game, any rules, configured without code.
3. **Delegates are the integration surface.** Presentation and external systems subscribe; they never reach into core state.
4. **C++ does the math. Blueprint does the game.** Heavy lifting (scoring, shuffling, effect resolution, combat math) in C++. Game-specific rules, card effects, AI behavior, and presentation — Blueprint.
5. **Tags over enums everywhere.** Zones, phases, resources, keywords, card types — all FGameplayTags. Games extend the tag tree without touching plugin code.

---

## Module Layout

```
Plugins/Sleight/
├── Sleight.uplugin
├── Source/
│   ├── Sleight/            ← Runtime core. No UMG, no actors, no rendering.
│   ├── SleightBoard/       ← 3D presentation. Opt-in. Depends on Sleight.
│   └── SleightEditor/      ← Editor tooling. Card editor, rule validator, DA factories.
├── Content/
│   ├── Core/               ← Default DA templates (GameRules, TurnStructure, BoardLayout)
│   ├── Icons/              ← Keyword icons, rarity frames
│   └── Examples/           ← Example game configs (Deckbuilder, Hearthstone-like, etc.)
└── Config/
    └── DefaultGameplayTags.ini  ← All Sleight.* tag definitions
```

### Module Dependencies

```
SleightEditor  →  Sleight, SleightBoard, UnrealEd, Slate
SleightBoard   →  Sleight, Engine
Sleight        →  Core, CoreUObject, Engine, GameplayTags
```

---

## Layer Diagram

```
┌─────────────────────────────────────────────────────┐
│                  YOUR GAME / PROJECT                │
│  BP card effects │ custom AI │ custom win conds     │
└───────────────┬──────────────────────────┬──────────┘
                │ configures               │ subscribes
                ▼                          ▼
┌─────────────────────────┐   ┌───────────────────────┐
│   SLEIGHT CORE          │   │  PRESENTATION LAYER   │
│  GameSession            │──►│  SleightBoard (3D)    │
│  ZoneManager            │   │  Widget Board (UMG)   │
│  EffectStack            │   │  Hybrid (both)        │
│  TurnController         │   └───────────────────────┘
│  CombatResolver         │
│  MarketManager          │◄── Delegates only. No reach-in.
│  AIController           │
│  HistoryLog             │
└─────────────────────────┘
                ▲
                │ defines rules
┌───────────────┴─────────────────────────────────────┐
│              DATA ASSET LAYER                       │
│  DA_GameRules │ DA_Card │ DA_AIConfig │ DA_BoardLayout│
└─────────────────────────────────────────────────────┘
```

---

## Module 1: Sleight (Runtime Core)

### 1.1 Types & Tags — `SleightTypes.h`

All enums, base structs, and tag constants in one header.

```cpp
// Zone transfer reason tags — games extend these
namespace SleightTags
{
    // Zones
    extern const FGameplayTag Zone_Deck;
    extern const FGameplayTag Zone_Hand;
    extern const FGameplayTag Zone_Play;
    extern const FGameplayTag Zone_Discard;
    extern const FGameplayTag Zone_Exile;
    extern const FGameplayTag Zone_Market;
    extern const FGameplayTag Zone_Stack;
    extern const FGameplayTag Zone_Limbo;

    // Transfer reasons
    extern const FGameplayTag Transfer_Draw;
    extern const FGameplayTag Transfer_Play;
    extern const FGameplayTag Transfer_Discard;
    extern const FGameplayTag Transfer_Destroy;
    extern const FGameplayTag Transfer_Mill;
    extern const FGameplayTag Transfer_Exile;
    extern const FGameplayTag Transfer_Bounce;
    extern const FGameplayTag Transfer_Buy;
    extern const FGameplayTag Transfer_Tutor;

    // Phases
    extern const FGameplayTag Phase_TurnStart;
    extern const FGameplayTag Phase_Draw;
    extern const FGameplayTag Phase_Main;
    extern const FGameplayTag Phase_Combat;
    extern const FGameplayTag Phase_End;
    extern const FGameplayTag Phase_Cleanup;

    // Resources
    extern const FGameplayTag Resource_Mana;
    extern const FGameplayTag Resource_Energy;
    extern const FGameplayTag Resource_Gold;
    extern const FGameplayTag Resource_Actions;
}

UENUM(BlueprintType)
enum class ESleightInsertPosition : uint8
{
    Top, Bottom, Random, AtIndex, Shuffled
};

UENUM(BlueprintType)
enum class ESleightVisibility : uint8
{
    Public,         // Everyone sees it
    OwnerOnly,      // Only the owning player
    Hidden,         // No one (face-down deck)
    TopOnly         // Only the top card visible
};

UENUM(BlueprintType)
enum class ESleightGameResult : uint8
{
    Ongoing, Player1Wins, Player2Wins, Draw, Abandoned
};

UENUM(BlueprintType)
enum class ESleightActionResult : uint8
{
    Success,
    InvalidAction,      // Not a legal action right now
    InvalidTarget,      // Target doesn't pass targeting rules
    CannotAfford,       // Not enough resources
    ZoneFull,           // Destination zone at capacity
    NotYourTurn,        // Wrong player
    Cancelled           // Intercepted by an effect
};
```

---

### 1.2 Card Data — `SleightCardData.h/.cpp`

**`UDA_SleightCard`** — The card template. One per unique card in your game.

```cpp
UCLASS(BlueprintType)
class UDA_SleightCard : public UPrimaryDataAsset
{
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Identity")
    FGameplayTag CardID;                        // Sleight.Card.Fireball

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Identity")
    FText DisplayName;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Identity")
    FText RulesText;                            // Rendered from template string

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Identity")
    TSoftObjectPtr<UTexture2D> CardArt;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Identity")
    TSoftObjectPtr<UTexture2D> CardArtLandscape; // For 3D board — wider format

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Classification")
    FGameplayTag CardType;                       // Sleight.CardType.Spell

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Classification")
    FGameplayTag Rarity;                         // Sleight.Rarity.Rare

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Classification")
    FGameplayTagContainer CardTags;              // Keyword tags: Taunt, Flying, etc.

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Stats")
    int32 Cost = 0;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Stats")
    TMap<FGameplayTag, int32> Properties;        // Attack, Health, Duration, etc.

    // Effects — instanced objects, configured inline in the DA
    UPROPERTY(EditAnywhere, Instanced, BlueprintReadOnly, Category="Effects")
    TArray<TObjectPtr<USleightCardEffect>> OnPlayEffects;

    UPROPERTY(EditAnywhere, Instanced, BlueprintReadOnly, Category="Effects")
    TArray<TObjectPtr<USleightCardEffect>> OnDeathEffects;

    UPROPERTY(EditAnywhere, Instanced, BlueprintReadOnly, Category="Effects")
    TArray<TObjectPtr<USleightCardEffect>> OnDrawEffects;

    UPROPERTY(EditAnywhere, Instanced, BlueprintReadOnly, Category="Effects")
    TArray<TObjectPtr<USleightCardEffect>> OnDiscardEffects;

    UPROPERTY(EditAnywhere, Instanced, BlueprintReadOnly, Category="Effects")
    TArray<TObjectPtr<USleightCardEffect>> OnTurnStartEffects;

    UPROPERTY(EditAnywhere, Instanced, BlueprintReadOnly, Category="Effects")
    TArray<TObjectPtr<USleightCardEffect>> OnTurnEndEffects;

    UPROPERTY(EditAnywhere, Instanced, BlueprintReadOnly, Category="Effects")
    TArray<TObjectPtr<USleightCardEffect>> PassiveEffects;    // Always active while in play

    UPROPERTY(EditAnywhere, Instanced, BlueprintReadOnly, Category="Effects")
    TObjectPtr<USleightCardEffect> ActivatedAbility;          // Tap/exhaust ability (optional)

    // Upgrade path (optional — for Slay the Spire style)
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Upgrade")
    TObjectPtr<UDA_SleightCard> UpgradedVersion;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Upgrade")
    int32 UpgradeCost = 0;

    // Targeting rule for playing from hand
    UPROPERTY(EditAnywhere, Instanced, BlueprintReadOnly, Category="Targeting")
    TObjectPtr<USleightTargetRule> PlayTargeting;             // null = no target required

    // Presentation hints (used by board layer, ignored by core)
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Presentation")
    bool bIsToken = false;
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Presentation")
    FLinearColor FrameColor = FLinearColor::White;

    // Editor utility
    UFUNCTION(BlueprintPure) int32 GetProperty(FGameplayTag Key, int32 Default = 0) const;
    UFUNCTION(BlueprintPure) bool HasTag(FGameplayTag Tag) const;
};
```

**`FSleightCardInstance`** — Runtime copy. Lives inside zones. Mutable.

```cpp
USTRUCT(BlueprintType)
struct FSleightCardInstance
{
    UPROPERTY(BlueprintReadOnly) FGuid InstanceID;
    UPROPERTY(BlueprintReadOnly) TObjectPtr<UDA_SleightCard> CardData;
    UPROPERTY(BlueprintReadOnly) int32 OwnerPlayerID = -1;
    UPROPERTY(BlueprintReadOnly) FGameplayTag CurrentZone;
    UPROPERTY(BlueprintReadOnly) bool bIsFaceUp = false;
    UPROPERTY(BlueprintReadOnly) bool bIsExhausted = false;
    UPROPERTY(BlueprintReadOnly) int32 TurnEnteredCurrentZone = -1;
    UPROPERTY(BlueprintReadOnly) FGameplayTag CurrentSlot;     // empty if no positional board

    // Mutable runtime overrides (from buffs, debuffs, cost reduction, etc.)
    UPROPERTY(BlueprintReadWrite) TMap<FGameplayTag, int32> ModifiedProperties;
    UPROPERTY(BlueprintReadWrite) int32 CostModifier = 0;
    UPROPERTY(BlueprintReadWrite) FGameplayTagContainer GrantedTags;    // runtime-added tags
    UPROPERTY(BlueprintReadWrite) FGameplayTagContainer RemovedTags;    // runtime-removed tags

    // Status effects active on this card
    UPROPERTY(BlueprintReadWrite) TArray<FSleightStatusEffect> StatusEffects;

    // Game-specific mutable state (counters, charges, etc.)
    UPROPERTY(BlueprintReadWrite) TMap<FName, int32> Counters;

    // Queries — work on combined base + modified data
    UFUNCTION(BlueprintPure) int32 GetEffectiveCost() const;
    UFUNCTION(BlueprintPure) int32 GetEffectiveProperty(FGameplayTag Key) const;
    UFUNCTION(BlueprintPure) bool HasEffectiveTag(FGameplayTag Tag) const;
    UFUNCTION(BlueprintPure) FGameplayTagContainer GetEffectiveTags() const;
    UFUNCTION(BlueprintPure) bool IsValid() const;
};
```

---

### 1.3 Zone System — `SleightZone.h/.cpp`

**`USleightZone`** — Runtime zone. Owned by `USleightZoneManager`.

```cpp
UCLASS(BlueprintType)
class USleightZone : public UObject
{
    UPROPERTY(BlueprintReadOnly) FGameplayTag ZoneTag;
    UPROPERTY(BlueprintReadOnly) int32 OwnerPlayerID = -1;   // -1 = shared
    UPROPERTY(BlueprintReadOnly) ESleightVisibility Visibility;
    UPROPERTY(BlueprintReadOnly) int32 MaxCapacity = -1;     // -1 = unlimited
    UPROPERTY(BlueprintReadOnly) bool bIsOrdered = true;
    UPROPERTY(BlueprintReadOnly) TArray<FSleightCardInstance> Cards;

    // Slot-aware (for positional boards)
    UPROPERTY(BlueprintReadOnly) TMap<FGameplayTag, TArray<FGuid>> SlotContents;

    // Queries
    UFUNCTION(BlueprintPure) int32 Count() const;
    UFUNCTION(BlueprintPure) bool IsEmpty() const;
    UFUNCTION(BlueprintPure) bool IsFull() const;
    UFUNCTION(BlueprintPure) const FSleightCardInstance* FindByID(FGuid InstanceID) const;
    UFUNCTION(BlueprintPure) TArray<FSleightCardInstance> GetCardsForPlayer(int32 PlayerID) const;
    UFUNCTION(BlueprintPure) TArray<FSleightCardInstance> Filter(FGameplayTagQuery Query) const;
    UFUNCTION(BlueprintPure) FSleightCardInstance GetTop() const;
    UFUNCTION(BlueprintPure) FSleightCardInstance GetBottom() const;
    UFUNCTION(BlueprintPure) TArray<FSleightCardInstance> Peek(int32 Count) const;
    UFUNCTION(BlueprintPure) bool CanAcceptCard(const FSleightCardInstance& Card) const;

    // Mutations — called only by ZoneManager, not directly
    void InsertCard(const FSleightCardInstance& Card, ESleightInsertPosition Pos, int32 Index = 0);
    void RemoveCard(FGuid InstanceID);
    void Shuffle(FRandomStream& RNG);
    void Sort(TFunction<bool(const FSleightCardInstance&, const FSleightCardInstance&)> Predicate);
};
```

**`USleightZoneManager`** — The sole authority on card positions. All zone transfers go through here.

```cpp
UCLASS()
class USleightZoneManager : public UObject
{
public:
    // Zone lifecycle
    USleightZone* CreateZone(FGameplayTag Tag, int32 OwnerID, ESleightVisibility Vis, int32 MaxCap = -1);
    USleightZone* GetZone(FGameplayTag Tag, int32 OwnerID = -1) const;
    TArray<USleightZone*> GetAllZonesForPlayer(int32 PlayerID) const;

    // Card lifecycle
    FSleightCardInstance& CreateCardInstance(UDA_SleightCard* Template, int32 OwnerID);
    void DestroyCardInstance(FGuid InstanceID);
    const FSleightCardInstance* FindCardAnywhere(FGuid InstanceID) const;
    FGameplayTag GetCardZone(FGuid InstanceID) const;

    // THE core operation
    ESleightActionResult TransferCard(
        FGuid InstanceID,
        FGameplayTag DestZoneTag,
        ESleightInsertPosition Position,
        FGameplayTag Reason,
        int32 InstigatorID,
        int32 InsertIndex = 0,
        bool bRevealOnTransfer = false
    );

    // Batch transfers (e.g., board wipe — all at once, one delegate broadcast)
    void TransferAllMatching(
        FGameplayTag SourceZone,
        FGameplayTag DestZone,
        FGameplayTagQuery Filter,
        FGameplayTag Reason,
        int32 InstigatorID
    );

    // Deck-specific helpers
    void ShuffleZone(FGameplayTag ZoneTag, int32 OwnerID);
    TArray<FSleightCardInstance> PeekZone(FGameplayTag ZoneTag, int32 OwnerID, int32 Count) const;

    // Delegates
    DECLARE_MULTICAST_DELEGATE_TwoParams(FOnPreTransfer, FSleightZoneTransfer&, bool& /*bCancelled*/);
    DECLARE_MULTICAST_DELEGATE_OneParam(FOnPostTransfer, const FSleightZoneTransfer&);
    DECLARE_MULTICAST_DELEGATE_TwoParams(FOnCardEnteredZone, const FSleightCardInstance&, FGameplayTag /*Zone*/);
    DECLARE_MULTICAST_DELEGATE_TwoParams(FOnCardLeftZone, const FSleightCardInstance&, FGameplayTag /*Zone*/);
    DECLARE_MULTICAST_DELEGATE_OneParam(FOnZoneShuffled, FGameplayTag /*Zone*/);

    FOnPreTransfer   OnPreTransfer;
    FOnPostTransfer  OnPostTransfer;
    FOnCardEnteredZone OnCardEnteredZone;
    FOnCardLeftZone    OnCardLeftZone;
    FOnZoneShuffled    OnZoneShuffled;

private:
    TMap<TTuple<FGameplayTag, int32>, TObjectPtr<USleightZone>> Zones;
    TMap<FGuid, FSleightCardInstance> AllInstances;
    TMap<FGuid, TTuple<FGameplayTag, int32>> InstanceToZone; // fast lookup
};
```

---

### 1.4 Player — `SleightPlayer.h/.cpp`

```cpp
USTRUCT(BlueprintType)
struct FSleightResourcePool
{
    UPROPERTY(EditAnywhere, BlueprintReadWrite) int32 Current = 0;
    UPROPERTY(EditAnywhere, BlueprintReadWrite) int32 Maximum = 0;
    UPROPERTY(EditAnywhere, BlueprintReadWrite) int32 PerTurnGain = 0;     // added each turn
    UPROPERTY(EditAnywhere, BlueprintReadWrite) int32 PerTurnRefill = -1;  // -1 = full refill
    UPROPERTY(EditAnywhere, BlueprintReadWrite) bool bCanGoNegative = false;
    UPROPERTY(EditAnywhere, BlueprintReadWrite) bool bAutoRefillOnTurnStart = true;

    bool CanSpend(int32 Amount) const;
    bool TrySpend(int32 Amount);
    void Gain(int32 Amount);
    void RefillForTurn();
};

UCLASS(BlueprintType)
class USleightPlayer : public UObject
{
    UPROPERTY(BlueprintReadOnly) int32 PlayerID;
    UPROPERTY(BlueprintReadOnly) FText DisplayName;
    UPROPERTY(BlueprintReadOnly) int32 Health = 30;
    UPROPERTY(BlueprintReadOnly) int32 MaxHealth = 30;
    UPROPERTY(BlueprintReadOnly) bool bIsAI = false;
    UPROPERTY(BlueprintReadOnly) bool bHasPriority = false;
    UPROPERTY(BlueprintReadOnly) int32 TurnOrder = 0;

    // Resources (mana, energy, gold, actions, buys — whatever the game uses)
    UPROPERTY(BlueprintReadOnly) TMap<FGameplayTag, FSleightResourcePool> Resources;

    // Status effects on the player themselves
    UPROPERTY(BlueprintReadOnly) TArray<FSleightStatusEffect> StatusEffects;

    // Persistent passive modifiers (relics, jokers, etc.)
    UPROPERTY(BlueprintReadOnly) TArray<TObjectPtr<UDA_SleightRelic>> Relics;

    // Score (for point-based games)
    UPROPERTY(BlueprintReadOnly) int32 Score = 0;

    // Mulligan tracking
    UPROPERTY(BlueprintReadOnly) int32 MulligansTaken = 0;

    // Zone shortcuts (convenience — source of truth is ZoneManager)
    UFUNCTION(BlueprintPure) int32 GetHandSize() const;
    UFUNCTION(BlueprintPure) int32 GetDeckSize() const;
    UFUNCTION(BlueprintPure) bool CanAfford(FGameplayTag Resource, int32 Amount) const;
    UFUNCTION(BlueprintPure) bool CanPlayCard(const FSleightCardInstance& Card) const;

    // Resource API
    UFUNCTION(BlueprintCallable) bool TrySpendResource(FGameplayTag ResourceTag, int32 Amount);
    UFUNCTION(BlueprintCallable) void GainResource(FGameplayTag ResourceTag, int32 Amount);
    UFUNCTION(BlueprintCallable) void SetResource(FGameplayTag ResourceTag, int32 Amount);
    UFUNCTION(BlueprintCallable) int32 GetResource(FGameplayTag ResourceTag) const;
    UFUNCTION(BlueprintCallable) void RefillResourcesForTurn();

    // Health
    UFUNCTION(BlueprintCallable) bool ApplyDamage(int32 Amount, int32 SourcePlayerID);
    UFUNCTION(BlueprintCallable) bool Heal(int32 Amount);
    UFUNCTION(BlueprintCallable) bool IsAlive() const { return Health > 0; }

    // Delegates
    FSimpleMulticastDelegate OnHealthChanged;
    FSimpleMulticastDelegate OnResourceChanged;
    FSimpleMulticastDelegate OnRelicAdded;
};
```

---

### 1.5 Effects — `SleightCardEffect.h/.cpp`

**`FSleightEffectContext`** — Everything an effect needs to know when it fires.

```cpp
USTRUCT(BlueprintType)
struct FSleightEffectContext
{
    UPROPERTY(BlueprintReadOnly) FSleightCardInstance SourceCard;
    UPROPERTY(BlueprintReadOnly) int32 InstigatorPlayerID = -1;
    UPROPERTY(BlueprintReadOnly) FGameplayTag TriggerReason;     // why this effect fired
    UPROPERTY(BlueprintReadOnly) FGameplayTag CurrentPhase;
    UPROPERTY(BlueprintReadOnly) int32 TurnNumber = 0;

    // Resolved targets (populated by TargetRule before Execute is called)
    UPROPERTY(BlueprintReadOnly) TArray<FSleightCardInstance> TargetCards;
    UPROPERTY(BlueprintReadOnly) TArray<int32> TargetPlayerIDs;
    UPROPERTY(BlueprintReadOnly) TArray<FGameplayTag> TargetSlots;

    // Back-reference (effects need to call back into the session)
    UPROPERTY(BlueprintReadOnly) TObjectPtr<USleightGameSession> Session;
};
```

**`USleightCardEffect`** — The abstract base. Blueprint-implementable.

```cpp
UCLASS(Abstract, Blueprintable, EditInlineNew, DefaultToInstanced, CollapseCategories)
class USleightCardEffect : public UObject
{
    // Override this in C++ subclasses or Blueprint
    UFUNCTION(BlueprintNativeEvent, BlueprintCallable)
    void Execute(const FSleightEffectContext& Context);

    UFUNCTION(BlueprintNativeEvent, BlueprintPure)
    bool CanExecute(const FSleightEffectContext& Context) const;

    // Optional targeting rule (null = no targets needed)
    UPROPERTY(EditAnywhere, Instanced, BlueprintReadOnly, Category="Targeting")
    TObjectPtr<USleightTargetRule> TargetRule;

    // For display in tooltips ("Deal 3 damage" auto-generates from this)
    UFUNCTION(BlueprintNativeEvent, BlueprintPure)
    FText GetDescription(const FSleightCardInstance& Card) const;
};
```

**Built-in C++ effect subclasses** (designer configures values in the DA, no code):

```
USleightEffect_DealDamage           Damage, bSplitAmongTargets
USleightEffect_Heal                 Amount, bFullHeal
USleightEffect_DrawCards            Count, bFromBottom
USleightEffect_DiscardCards         Count, bRandom, bOpponent
USleightEffect_GainResource         ResourceTag, Amount, bTemporary
USleightEffect_SpendResource        ResourceTag, Amount (for costs on abilities)
USleightEffect_ModifyProperty       PropertyTag, Delta, bUntilEndOfTurn
USleightEffect_AddTag               TagToAdd, bUntilEndOfTurn
USleightEffect_RemoveTag            TagToRemove
USleightEffect_AddCounter           CounterName, Amount
USleightEffect_MoveCard             DestZone, InsertPos, Reason (bounce, exile, etc.)
USleightEffect_DestroyCard          (moves to graveyard with Destroy reason)
USleightEffect_CreateToken          TokenTemplate, DestZone, Count
USleightEffect_CopyCard             SourceCard, DestZone (create a copy)
USleightEffect_TransformCard        NewCardTemplate (change into)
USleightEffect_ModifyCost           Delta, bUntilEndOfTurn, Filter
USleightEffect_ShuffleDeck          PlayerID (-1 = self)
USleightEffect_SearchDeck           Filter, DestZone (tutor)
USleightEffect_AddStatus            StatusEffect, Duration
USleightEffect_AddRelic             RelicDA (for deckbuilders)
USleightEffect_GainScore            Amount (for point games)
USleightEffect_Conditional          Condition, IfTrue[], IfFalse[] (if/then)
USleightEffect_Composite            Effects[] (execute all in order)
USleightEffect_Random               Effects[], Count (pick N randomly)
USleightEffect_RepeatForEach        Filter, Effect (for each matching card, do X)
USleightEffect_PlayerChoice         Options[], bChooseFromHand, bChooseFromBoard
USleightEffect_Blueprint            (empty shell — override Execute_Implementation in BP)
```

---

### 1.6 Targeting — `SleightTargetRule.h/.cpp`

```cpp
UCLASS(Abstract, Blueprintable, EditInlineNew, DefaultToInstanced)
class USleightTargetRule : public UObject
{
    UPROPERTY(EditAnywhere, BlueprintReadOnly) int32 RequiredCount = 1;    // 0 = optional
    UPROPERTY(EditAnywhere, BlueprintReadOnly) int32 MaxCount = 1;
    UPROPERTY(EditAnywhere, BlueprintReadOnly) bool bAutoSelectIfOne = true;
    UPROPERTY(EditAnywhere, BlueprintReadOnly) bool bTargetsPlayers = false;
    UPROPERTY(EditAnywhere, BlueprintReadOnly) bool bTargetsSlots = false;

    UFUNCTION(BlueprintNativeEvent) TArray<FSleightCardInstance> GetValidCardTargets(const FSleightEffectContext& Ctx) const;
    UFUNCTION(BlueprintNativeEvent) TArray<int32> GetValidPlayerTargets(const FSleightEffectContext& Ctx) const;
    UFUNCTION(BlueprintNativeEvent) bool IsValidTarget(const FSleightEffectContext& Ctx, const FSleightCardInstance& Card) const;
};

// Built-in targeting rules:
USleightTarget_Any              // Any card in play (both sides)
USleightTarget_Friendly         // Your cards only
USleightTarget_Enemy            // Opponent's cards only
USleightTarget_InZone           // ZoneTag filter
USleightTarget_WithTag          // Must have specific tag (e.g., "Taunt")
USleightTarget_WithoutTag       // Must NOT have tag
USleightTarget_PropertyMin      // Property >= threshold (e.g., Attack >= 4)
USleightTarget_PropertyMax      // Property <= threshold
USleightTarget_Self             // The card itself
USleightTarget_Adjacent         // Cards in adjacent slots
USleightTarget_SameRow          // All cards in same row
USleightTarget_SameLane         // All cards in same lane
USleightTarget_Random           // Random valid card
USleightTarget_AllMatching      // All cards matching filter (no player choice)
USleightTarget_Player           // Target a player directly
USleightTarget_FaceDown         // Face-down cards only
USleightTarget_Blueprint        // Override GetValidCardTargets in BP
```

---

### 1.7 Turn Controller — `SleightTurnController.h/.cpp`

Drives the phase sequence. Reads `UDA_SleightTurnStructure`.

```cpp
UCLASS()
class USleightTurnController : public UObject
{
    UPROPERTY(BlueprintReadOnly) int32 TurnNumber = 0;
    UPROPERTY(BlueprintReadOnly) int32 ActivePlayerID = 0;
    UPROPERTY(BlueprintReadOnly) FGameplayTag CurrentPhase;
    UPROPERTY(BlueprintReadOnly) bool bWaitingForPlayerInput = false;

    // Phase progression
    UFUNCTION(BlueprintCallable) void AdvancePhase();
    UFUNCTION(BlueprintCallable) void AdvanceTurn();
    UFUNCTION(BlueprintCallable) void ForcePhase(FGameplayTag Phase);    // effects can jump phases
    UFUNCTION(BlueprintCallable) void GivePriority(int32 PlayerID);

    // Phase queries
    UFUNCTION(BlueprintPure) bool IsPhase(FGameplayTag Phase) const;
    UFUNCTION(BlueprintPure) bool IsActivePlayer(int32 PlayerID) const;
    UFUNCTION(BlueprintPure) bool IsActionLegalNow(FGameplayTag ActionTag) const;
    UFUNCTION(BlueprintPure) int32 GetNextPlayerID() const;

    // Delegates
    DECLARE_MULTICAST_DELEGATE_TwoParams(FOnTurnStarted, int32 /*PlayerID*/, int32 /*TurnNumber*/);
    DECLARE_MULTICAST_DELEGATE_OneParam(FOnTurnEnded, int32 /*PlayerID*/);
    DECLARE_MULTICAST_DELEGATE_TwoParams(FOnPhaseChanged, FGameplayTag /*Old*/, FGameplayTag /*New*/);
    DECLARE_MULTICAST_DELEGATE_OneParam(FOnPriorityChanged, int32 /*PlayerID*/);

    FOnTurnStarted  OnTurnStarted;
    FOnTurnEnded    OnTurnEnded;
    FOnPhaseChanged OnPhaseChanged;
    FOnPriorityChanged OnPriorityChanged;
};
```

**`UDA_SleightTurnStructure`** — Defines the phase flow.

```cpp
UCLASS(BlueprintType)
class UDA_SleightTurnStructure : public UPrimaryDataAsset
{
    // Ordered phase sequence
    UPROPERTY(EditAnywhere, BlueprintReadOnly) TArray<FGameplayTag> Phases;

    // Which action types are legal per phase
    UPROPERTY(EditAnywhere, BlueprintReadOnly) TMap<FGameplayTag, FGameplayTagContainer> LegalActionsPerPhase;

    // Does this phase auto-advance, or wait for EndPhase() call?
    UPROPERTY(EditAnywhere, BlueprintReadOnly) TMap<FGameplayTag, bool> AutoAdvancePhase;

    // Per-phase auto-actions (e.g., "in Draw phase, draw 1 card automatically")
    UPROPERTY(EditAnywhere, Instanced, BlueprintReadOnly) TMap<FGameplayTag, TObjectPtr<USleightCardEffect>> AutoPhaseEffects;

    // Priority model
    UPROPERTY(EditAnywhere, BlueprintReadOnly) bool bAlternatingPriority = true;  // false = active player only
    UPROPERTY(EditAnywhere, BlueprintReadOnly) bool bUseEffectStack = false;       // MTG-style stack

    // Simultaneous turns (some games have all players act at once)
    UPROPERTY(EditAnywhere, BlueprintReadOnly) bool bSimultaneousTurns = false;
};
```

---

### 1.8 Effect Stack — `SleightEffectStack.h/.cpp`

Handles resolution order. Simple queue for most games, LIFO stack for MTG-style.

```cpp
UCLASS()
class USleightEffectStack : public UObject
{
    // Push an effect to be resolved
    void Push(const FSleightEffectContext& Context, USleightCardEffect* Effect);

    // Resolve top of stack (called after priority passes)
    void ResolveTop();

    // Resolve ALL (for games without response windows)
    void ResolveAll();

    // Insert a response BEFORE current resolution
    void Respond(const FSleightEffectContext& Context, USleightCardEffect* Effect);

    UFUNCTION(BlueprintPure) bool IsEmpty() const;
    UFUNCTION(BlueprintPure) int32 Count() const;

    // Delegates
    DECLARE_MULTICAST_DELEGATE_OneParam(FOnEffectPushed, const FSleightEffectContext&);
    DECLARE_MULTICAST_DELEGATE_OneParam(FOnEffectResolved, const FSleightEffectContext&);
    DECLARE_MULTICAST_DELEGATE(FOnStackEmptied);

    FOnEffectPushed   OnEffectPushed;
    FOnEffectResolved OnEffectResolved;
    FOnStackEmptied   OnStackEmptied;

private:
    TArray<TTuple<FSleightEffectContext, USleightCardEffect*>> Stack;
};
```

---

### 1.9 Combat Resolver — `SleightCombatResolver.h/.cpp`

```cpp
UCLASS(Abstract, Blueprintable)
class USleightCombatResolver : public UObject
{
    UFUNCTION(BlueprintNativeEvent)
    void ResolveCombat(
        const FSleightCardInstance& Attacker,
        const FSleightCardInstance& Defender,
        USleightGameSession* Session
    );

    UFUNCTION(BlueprintNativeEvent, BlueprintPure)
    bool CanAttack(const FSleightCardInstance& Card, int32 PlayerID) const;

    UFUNCTION(BlueprintNativeEvent, BlueprintPure)
    bool CanBlock(const FSleightCardInstance& Blocker, const FSleightCardInstance& Attacker) const;

    DECLARE_MULTICAST_DELEGATE_FourParams(FOnCombatResolved,
        const FSleightCardInstance& /*Attacker*/,
        const FSleightCardInstance& /*Defender*/,
        int32 /*AttackerDamage*/,
        int32 /*DefenderDamage*/
    );
    FOnCombatResolved OnCombatResolved;
};

// Built-in resolvers:
USleightCombat_Direct       // Attacker deals damage to defender, defender deals back
USleightCombat_Mutual       // MTG-style (attackers/blockers, both deal simultaneously)
USleightCombat_RowBased     // Gwent-style (sum rows, no individual attacks)
USleightCombat_LaneBased    // Marvel Snap-style (compare lane totals)
USleightCombat_PlayerFace   // Unblocked attackers deal damage to player
USleightCombat_Blueprint    // Full BP override
```

---

### 1.10 Market — `SleightMarket.h/.cpp`

```cpp
UCLASS(BlueprintType)
class USleightMarket : public UObject
{
    UPROPERTY(BlueprintReadOnly) TObjectPtr<UDA_SleightMarketConfig> Config;

    // Current display slots
    UPROPERTY(BlueprintReadOnly) TArray<FSleightMarketSlot> DisplaySlots;

    // Supply (what the display refills from)
    UPROPERTY(BlueprintReadOnly) TArray<TObjectPtr<UDA_SleightCard>> SupplyPile;

    UFUNCTION(BlueprintCallable) ESleightActionResult TryBuy(int32 PlayerID, int32 SlotIndex);
    UFUNCTION(BlueprintCallable) void RefreshDisplay();
    UFUNCTION(BlueprintCallable) void RefreshSlot(int32 SlotIndex);
    UFUNCTION(BlueprintPure)    TArray<FSleightMarketSlot> GetAffordableSlots(int32 PlayerID) const;
    UFUNCTION(BlueprintPure)    bool CanBuy(int32 PlayerID, int32 SlotIndex) const;

    DECLARE_MULTICAST_DELEGATE_ThreeParams(FOnCardBought, int32 /*PlayerID*/, const FSleightMarketSlot&, int32 /*SlotIndex*/);
    DECLARE_MULTICAST_DELEGATE(FOnMarketRefreshed);

    FOnCardBought    OnCardBought;
    FOnMarketRefreshed OnMarketRefreshed;
};

USTRUCT(BlueprintType)
struct FSleightMarketSlot
{
    UPROPERTY(BlueprintReadOnly) TObjectPtr<UDA_SleightCard> Card;
    UPROPERTY(BlueprintReadOnly) int32 CurrentCost = 0;      // may differ from base cost (sales, etc.)
    UPROPERTY(BlueprintReadOnly) int32 SupplyCount = -1;     // -1 = unlimited (fixed display games)
    UPROPERTY(BlueprintReadOnly) bool bIsAvailable = true;
    UPROPERTY(BlueprintReadOnly) FGameplayTag SlotTag;       // for layout/visual binding
};
```

---

### 1.11 AI System — `SleightAI*.h/.cpp`

*(Full detail in separate section above — summary here)*

```
USleightAIBrain (abstract)     — orchestrator, calls scoring loop or BP override
UDA_SleightAIConfig            — DA: array of ScoringRules + weights + difficulty noise
USleightAIScoringRule (abstract, BlueprintNativeEvent) — ScoreAction(Context, Action) → float
FSleightAIContext (struct)      — full game state snapshot + query helpers

Built-in scoring rules (C++, configure values in DA):
  SleightScore_Lethal, DealDamage, ManaEfficiency, BoardClear,
  Survival, CardAdvantage, MarketValue, Random, Blueprint

Built-in brains:
  USleightAI_Random             — random legal action
  USleightAI_Greedy             — scoring loop, no lookahead
  USleightAI_PriorityList       — ordered conditional rules
  USleightAI_Blueprint          — full BP override of ChooseAction
```

---

### 1.12 History Log — `SleightHistoryLog.h/.cpp`

```cpp
UENUM(BlueprintType)
enum class ESleightEventType : uint8
{
    CardMoved, CardCreated, CardDestroyed, DamageDealt, HealApplied,
    ResourceChanged, HealthChanged, EffectExecuted, PhaseChanged,
    TurnStarted, TurnEnded, GameStarted, GameEnded, ActionTaken,
    MarketCardBought, RelicAdded, CounterChanged, StatusApplied, Custom
};

USTRUCT(BlueprintType)
struct FSleightHistoryEvent
{
    UPROPERTY(BlueprintReadOnly) int32 EventIndex;
    UPROPERTY(BlueprintReadOnly) ESleightEventType EventType;
    UPROPERTY(BlueprintReadOnly) int32 TurnNumber;
    UPROPERTY(BlueprintReadOnly) FGameplayTag Phase;
    UPROPERTY(BlueprintReadOnly) int32 InstigatorPlayerID;
    UPROPERTY(BlueprintReadOnly) FGuid CardInstanceID;
    UPROPERTY(BlueprintReadOnly) FGameplayTag SourceZone;
    UPROPERTY(BlueprintReadOnly) FGameplayTag DestZone;
    UPROPERTY(BlueprintReadOnly) FGameplayTag Reason;
    UPROPERTY(BlueprintReadOnly) TMap<FName, int32> IntPayload;   // damage amount, resource delta, etc.
    UPROPERTY(BlueprintReadOnly) TMap<FName, FString> StrPayload;
    UPROPERTY(BlueprintReadOnly) TArray<int32> ChildEventIndices; // cascaded effects
    UPROPERTY(BlueprintReadOnly) bool bIsUndoable;
};

UCLASS()
class USleightHistoryLog : public UObject
{
    void Record(FSleightHistoryEvent Event);
    const FSleightHistoryEvent& GetEvent(int32 Index) const;
    TArray<FSleightHistoryEvent> GetEventsForTurn(int32 TurnNumber) const;
    TArray<FSleightHistoryEvent> GetEventsForCard(FGuid InstanceID) const;

    bool CanUndo() const;
    bool Undo(USleightGameSession* Session);    // rolls back last undoable event chain

    // Serialization (for replay)
    FString SerializeToJson() const;
    bool DeserializeFromJson(const FString& Json);
};
```

---

### 1.13 Win Conditions — `SleightWinCondition.h/.cpp`

```cpp
UCLASS(Abstract, Blueprintable, EditInlineNew, DefaultToInstanced)
class USleightWinCondition : public UObject
{
    // Called by GameSession after every state-changing action
    UFUNCTION(BlueprintNativeEvent)
    ESleightGameResult Evaluate(USleightGameSession* Session) const;
};

// Built-in:
USleightWin_ReduceHealthToZero    // PlayerID with 0 HP loses
USleightWin_DeckOut               // Player who can't draw loses
USleightWin_HighestScore          // After N turns, highest score wins
USleightWin_EmptyDeck             // First to exhaust deck wins
USleightWin_BoardControl          // Control >= N slots/lanes
USleightWin_SpecialCard           // Having a specific card in play wins
USleightWin_SurviveNTurns         // Last player standing after N turns
USleightWin_CollectSet            // Have N cards with a specific tag wins
USleightWin_Composite             // TArray of conditions (Any / All modes)
USleightWin_Blueprint             // BP override
```

---

### 1.14 Game Session — `SleightGameSession.h/.cpp`

**The orchestrator.** Everything else is owned here.

```cpp
UCLASS(BlueprintType)
class USleightGameSession : public UObject
{
public:
    //--- Owned subsystems ---
    UPROPERTY(BlueprintReadOnly) TObjectPtr<USleightZoneManager>    ZoneManager;
    UPROPERTY(BlueprintReadOnly) TObjectPtr<USleightTurnController>  TurnController;
    UPROPERTY(BlueprintReadOnly) TObjectPtr<USleightEffectStack>     EffectStack;
    UPROPERTY(BlueprintReadOnly) TObjectPtr<USleightMarket>          Market;      // null if no market
    UPROPERTY(BlueprintReadOnly) TObjectPtr<USleightCombatResolver>  Combat;      // null if no combat
    UPROPERTY(BlueprintReadOnly) TObjectPtr<USleightHistoryLog>      History;
    UPROPERTY(BlueprintReadOnly) TObjectPtr<USleightBoardState>      Board;       // null if no positional board
    UPROPERTY(BlueprintReadOnly) TArray<TObjectPtr<USleightPlayer>>  Players;

    // The "cartridge"
    UPROPERTY(BlueprintReadOnly) TObjectPtr<UDA_SleightGameRules> Rules;

    // RNG — seeded for determinism
    UPROPERTY(BlueprintReadOnly) FRandomStream RNG;
    UPROPERTY(BlueprintReadOnly) int32 RNGSeed;

    //--- Lifecycle ---
    UFUNCTION(BlueprintCallable) void Initialize(UDA_SleightGameRules* InRules, int32 InSeed = -1);
    UFUNCTION(BlueprintCallable) void StartGame();
    UFUNCTION(BlueprintCallable) void EndGame(int32 WinnerID, FGameplayTag Reason);
    UFUNCTION(BlueprintCallable) void Forfeit(int32 PlayerID);

    //--- Player actions (validated entry points) ---
    UFUNCTION(BlueprintCallable) ESleightActionResult PlayCard(int32 PlayerID, FGuid CardInstanceID, FSleightTargetInfo TargetInfo);
    UFUNCTION(BlueprintCallable) ESleightActionResult ActivateAbility(int32 PlayerID, FGuid CardInstanceID, FSleightTargetInfo TargetInfo);
    UFUNCTION(BlueprintCallable) ESleightActionResult DeclareAttack(int32 PlayerID, FGuid AttackerID, FGuid DefenderID);
    UFUNCTION(BlueprintCallable) ESleightActionResult BuyFromMarket(int32 PlayerID, int32 SlotIndex);
    UFUNCTION(BlueprintCallable) ESleightActionResult DiscardCard(int32 PlayerID, FGuid CardInstanceID);
    UFUNCTION(BlueprintCallable) ESleightActionResult EndPhase(int32 PlayerID);
    UFUNCTION(BlueprintCallable) ESleightActionResult Mulligan(int32 PlayerID, TArray<FGuid> CardsToReturn);
    UFUNCTION(BlueprintCallable) ESleightActionResult UpgradeCard(int32 PlayerID, FGuid CardInstanceID);
    UFUNCTION(BlueprintCallable) ESleightActionResult TrashCard(int32 PlayerID, FGuid CardInstanceID);
    UFUNCTION(BlueprintCallable) ESleightActionResult PassPriority(int32 PlayerID);

    //--- Queries (safe to call from anywhere) ---
    UFUNCTION(BlueprintPure) USleightPlayer* GetPlayer(int32 PlayerID) const;
    UFUNCTION(BlueprintPure) TArray<ESleightActionResult> GetValidActionTypes(int32 PlayerID) const;
    UFUNCTION(BlueprintPure) TArray<FSleightCardInstance> GetPlayableCards(int32 PlayerID) const;
    UFUNCTION(BlueprintPure) bool IsGameOver() const;
    UFUNCTION(BlueprintPure) int32 GetActivePlayerID() const;
    UFUNCTION(BlueprintPure) FGameplayTag GetCurrentPhase() const;

    //--- Internal helpers (C++, not exposed to BP) ---
    void ExecuteEffect(USleightCardEffect* Effect, const FSleightEffectContext& Context);
    void TriggerEffectsForEvent(FGameplayTag TriggerReason, const FSleightCardInstance& Card);
    void ProcessPassiveEffects();
    void CheckWinConditions();
    void DoAutoPhaseActions();
    void HandleTurnStart(int32 PlayerID);
    void HandleTurnEnd(int32 PlayerID);
    void HandlePhaseEntered(FGameplayTag Phase);

    //--- ALL delegates (presentation and external systems subscribe here) ---
    DECLARE_MULTICAST_DELEGATE(FOnGameStarted);
    DECLARE_MULTICAST_DELEGATE_TwoParams(FOnGameEnded, int32 /*WinnerID*/, FGameplayTag /*Reason*/);
    DECLARE_MULTICAST_DELEGATE_TwoParams(FOnTurnStarted, int32 /*PlayerID*/, int32 /*TurnNumber*/);
    DECLARE_MULTICAST_DELEGATE_OneParam(FOnTurnEnded, int32 /*PlayerID*/);
    DECLARE_MULTICAST_DELEGATE_TwoParams(FOnPhaseChanged, FGameplayTag /*Old*/, FGameplayTag /*New*/);
    DECLARE_MULTICAST_DELEGATE_ThreeParams(FOnCardPlayed, int32 /*PlayerID*/, const FSleightCardInstance&, const FSleightTargetInfo&);
    DECLARE_MULTICAST_DELEGATE_TwoParams(FOnCardDrawn, int32 /*PlayerID*/, const FSleightCardInstance&);
    DECLARE_MULTICAST_DELEGATE_TwoParams(FOnCardDiscarded, int32 /*PlayerID*/, const FSleightCardInstance&);
    DECLARE_MULTICAST_DELEGATE_TwoParams(FOnCardDestroyed, const FSleightCardInstance&, FGameplayTag /*Reason*/);
    DECLARE_MULTICAST_DELEGATE_ThreeParams(FOnCardMoved, const FSleightCardInstance&, FGameplayTag /*From*/, FGameplayTag /*To*/);
    DECLARE_MULTICAST_DELEGATE_ThreeParams(FOnDamageDealt, FGuid /*Source*/, FGuid /*Target*/, int32 /*Amount*/);
    DECLARE_MULTICAST_DELEGATE_ThreeParams(FOnPlayerDamaged, int32 /*PlayerID*/, int32 /*Amount*/, FGuid /*SourceCard*/);
    DECLARE_MULTICAST_DELEGATE_ThreeParams(FOnHealthChanged, int32 /*PlayerID*/, int32 /*Old*/, int32 /*New*/);
    DECLARE_MULTICAST_DELEGATE_FourParams(FOnResourceChanged, int32 /*PlayerID*/, FGameplayTag /*Res*/, int32 /*Old*/, int32 /*New*/);
    DECLARE_MULTICAST_DELEGATE_TwoParams(FOnMarketCardBought, int32 /*PlayerID*/, const FSleightMarketSlot&);
    DECLARE_MULTICAST_DELEGATE_TwoParams(FOnRelicAdded, int32 /*PlayerID*/, UDA_SleightRelic* /*Relic*/);
    DECLARE_MULTICAST_DELEGATE_OneParam(FOnEffectResolved, const FSleightEffectContext&);
    DECLARE_MULTICAST_DELEGATE_ThreeParams(FOnActionRequested, int32 /*PlayerID*/, FGameplayTagContainer /*ValidActions*/, const TArray<FSleightCardInstance>& /*Playable*/);
    DECLARE_MULTICAST_DELEGATE_OneParam(FOnActionResult, ESleightActionResult);

    FOnGameStarted      OnGameStarted;
    FOnGameEnded        OnGameEnded;
    FOnTurnStarted      OnTurnStarted;
    FOnTurnEnded        OnTurnEnded;
    FOnPhaseChanged     OnPhaseChanged;
    FOnCardPlayed       OnCardPlayed;
    FOnCardDrawn        OnCardDrawn;
    FOnCardDiscarded    OnCardDiscarded;
    FOnCardDestroyed    OnCardDestroyed;
    FOnCardMoved        OnCardMoved;
    FOnDamageDealt      OnDamageDealt;
    FOnPlayerDamaged    OnPlayerDamaged;
    FOnHealthChanged    OnHealthChanged;
    FOnResourceChanged  OnResourceChanged;
    FOnMarketCardBought OnMarketCardBought;
    FOnRelicAdded       OnRelicAdded;
    FOnEffectResolved   OnEffectResolved;
    FOnActionRequested  OnActionRequested;
    FOnActionResult     OnActionResult;
};
```

---

### 1.15 Game Rules Data Asset — `SleightGameRules.h/.cpp`

The "cartridge." Swap this to play a different game with the same engine.

```cpp
UCLASS(BlueprintType)
class UDA_SleightGameRules : public UPrimaryDataAsset
{
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Game")
    FText GameName;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Players")
    FIntPoint PlayerCount = {2, 2};

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Players")
    int32 StartingHealth = 0;          // 0 = no health system

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Players")
    int32 StartingHandSize = 5;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Players")
    int32 MaxHandSize = -1;             // -1 = no limit

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Players")
    int32 DrawPerTurn = 1;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Players")
    TArray<FSleightResourceDef> ResourceDefinitions;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Players")
    TArray<TObjectPtr<UDA_SleightCard>> StartingDeckCards;   // what players begin with

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Turn")
    TObjectPtr<UDA_SleightTurnStructure> TurnStructure;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Board")
    TObjectPtr<UDA_SleightBoardLayout> BoardLayout;           // null = no positional board

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Market")
    TObjectPtr<UDA_SleightMarketConfig> MarketConfig;         // null = no market

    UPROPERTY(EditAnywhere, Instanced, BlueprintReadOnly, Category="Combat")
    TObjectPtr<USleightCombatResolver> CombatResolver;        // null = no combat

    UPROPERTY(EditAnywhere, Instanced, BlueprintReadOnly, Category="Win")
    TArray<TObjectPtr<USleightWinCondition>> WinConditions;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Deck")
    FSleightDeckRules DeckConstructionRules;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Deck")
    bool bAllowMulligans = true;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Deck")
    int32 MulliganHandSize = -1;        // -1 = same as StartingHandSize

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="AI")
    TObjectPtr<UDA_SleightAIConfig> DefaultAIConfig;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Keywords")
    TArray<TObjectPtr<UDA_SleightKeyword>> KeywordDefinitions;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Cards")
    TArray<TObjectPtr<UDA_SleightCard>> LegalCardPool;

    // Game-specific extensible config (no code changes needed)
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Custom")
    TMap<FName, int32> IntConfig;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Custom")
    TMap<FName, FString> StrConfig;
};
```

---

### 1.16 Blueprint Function Library — `SleightFunctionLibrary.h/.cpp`

All BFL helpers. Makes core accessible from BP without holding references.

```cpp
UCLASS()
class USleightFunctionLibrary : public UBlueprintFunctionLibrary
{
    // Session access
    UFUNCTION(BlueprintPure, meta=(WorldContext="WorldContext"))
    static USleightGameSession* GetSession(const UObject* WorldContext);

    // Card instance helpers
    UFUNCTION(BlueprintPure) static bool IsValidInstance(const FSleightCardInstance& Inst);
    UFUNCTION(BlueprintPure) static int32 GetEffectiveCost(const FSleightCardInstance& Inst);
    UFUNCTION(BlueprintPure) static bool HasKeyword(const FSleightCardInstance& Inst, FGameplayTag Keyword);

    // Zone helpers
    UFUNCTION(BlueprintPure) static int32 GetZoneCount(USleightGameSession* Session, FGameplayTag Zone, int32 PlayerID = -1);
    UFUNCTION(BlueprintPure) static TArray<FSleightCardInstance> GetZoneCards(USleightGameSession* Session, FGameplayTag Zone, int32 PlayerID = -1);
    UFUNCTION(BlueprintPure) static TArray<FSleightCardInstance> FilterCards(const TArray<FSleightCardInstance>& Cards, FGameplayTagQuery Query);

    // Player helpers
    UFUNCTION(BlueprintPure) static int32 GetResource(USleightGameSession* Session, int32 PlayerID, FGameplayTag ResourceTag);
    UFUNCTION(BlueprintPure) static TArray<FSleightCardInstance> GetPlayableCards(USleightGameSession* Session, int32 PlayerID);

    // Effect helpers (for use inside BP card effects)
    UFUNCTION(BlueprintCallable) static void DealDamage(const FSleightEffectContext& Context, FGuid TargetInstanceID, int32 Amount);
    UFUNCTION(BlueprintCallable) static void DealDamageToPlayer(const FSleightEffectContext& Context, int32 TargetPlayerID, int32 Amount);
    UFUNCTION(BlueprintCallable) static void DrawCards(const FSleightEffectContext& Context, int32 PlayerID, int32 Count);
    UFUNCTION(BlueprintCallable) static void MoveCard(const FSleightEffectContext& Context, FGuid InstanceID, FGameplayTag DestZone, FGameplayTag Reason);
    UFUNCTION(BlueprintCallable) static void GainResource(const FSleightEffectContext& Context, int32 PlayerID, FGameplayTag ResourceTag, int32 Amount);

    // Math / utility
    UFUNCTION(BlueprintPure) static int32 SumProperty(const TArray<FSleightCardInstance>& Cards, FGameplayTag PropertyTag);
    UFUNCTION(BlueprintPure) static int32 CountWithTag(const TArray<FSleightCardInstance>& Cards, FGameplayTag Tag);
    UFUNCTION(BlueprintPure) static FSleightCardInstance GetHighestProperty(const TArray<FSleightCardInstance>& Cards, FGameplayTag PropertyTag);
};
```

---

## Module 2: SleightBoard (3D Presentation)

Completely optional. Depends on Module 1 but Module 1 does not depend on it.

### 2.1 Board Actor — `SleightBoard3D.h/.cpp`

```
ASleightBoard3D
├── Subscribes to ALL GameSession delegates at BeginPlay
├── Owns all card and slot actors
├── CardActorPool — pooled ASleightCard3D for performance
├── SlotActors (TMap<FGameplayTag, ASleightSlot3D*>) — built from BoardLayout
├── AnimQueue — serialized card movement animations (never two cards moving through same slot at once)
├── BP Events fired for each delegate (so BP subclass can respond)
│   ├── OnCardMoved_BP(CardActor, FromSlot, ToSlot, Reason)
│   ├── OnCardPlayed_BP(CardActor, PlayerID, TargetInfo)
│   ├── OnCardDrawn_BP(CardActor, PlayerID)
│   ├── OnDamageDealt_BP(SourceActor, TargetActor, Amount)
│   └── ... (one BP event per GameSession delegate)
└── Configuration
    ├── CardMoveArcHeight (float)
    ├── CardMoveDuration (float)
    ├── HandFanRadius (float)
    ├── HandFanAngle (float)
    └── HandCardLiftOnHover (float)
```

### 2.2 Card Actor — `SleightCard3D.h/.cpp`

```
ASleightCard3D
├── CardInstanceID (FGuid) — identity
├── StaticMeshComponent — the card geometry
├── FrontMID (UMaterialInstanceDynamic*) — card art via texture param
├── BackMID (UMaterialInstanceDynamic*) — card back
├── SelectionHighlight — niagara or post-process
├── States: Normal, Hovered, Selected, Dragged, Tapped, Exhausted, Dead
├── bIsFaceUp — drives which side renders
│
├── UFUNCTION(BlueprintCallable) void FlipToFace()
├── UFUNCTION(BlueprintCallable) void FlipToBack()
├── UFUNCTION(BlueprintCallable) void SetExhausted(bool bExhausted)
├── UFUNCTION(BlueprintCallable) void PlayEnterAnimation()
├── UFUNCTION(BlueprintCallable) void PlayDestroyAnimation()
│
├── BP Overridable:
│   ├── OnHovered_BP()
│   ├── OnUnhovered_BP()
│   ├── OnSelected_BP()
│   ├── OnDeselected_BP()
│   └── OnBecomesPlayable_BP() / OnBecomesUnplayable_BP()
│
└── Input handling (click, hover) → fires to Board3D for game logic routing
```

### 2.3 Slot Actor — `SleightSlot3D.h/.cpp`

```
ASleightSlot3D
├── SlotTag (FGameplayTag)
├── SlotMesh — highlight ring, zone indicator
├── bIsHighlighted — valid drop target
├── bIsOccupied
├── CurrentCards (TArray<FGuid>)
└── GetWorldTransformForCard(int32 CardIndex) → FTransform
    (stacking/fanning behavior per zone type)
```

---

## Module 3: SleightEditor

```
Card Editor
├── Custom AssetFactory for UDA_SleightCard
├── Detail customization — shows card preview, renders rules text from template
├── Card validation (missing art, undefined properties, etc.)

Game Rules Validator
├── Button in DA_SleightGameRules details panel: "Validate Rules"
├── Checks: turn structure has required phases, win conditions defined,
│          legal card pool populated, resource definitions complete

Board Layout Preview
├── 2D grid preview in DA_SleightBoardLayout details panel
├── Shows slot positions, adjacency links, player areas colored

DA Factories
├── Right-click → Create Sleight → Card, GameRules, AIConfig, BoardLayout,
│   TurnStructure, MarketConfig, Relic, Keyword
└── All use factory objects that pre-populate sensible defaults
```

---

## Data Asset Map (What Designers Author)

| Asset | Purpose | One per |
|-------|---------|---------|
| `UDA_SleightCard` | One card definition | Unique card |
| `UDA_SleightGameRules` | Complete game ruleset | Game variant |
| `UDA_SleightTurnStructure` | Phase sequence + legal actions | Turn flow variant |
| `UDA_SleightBoardLayout` | Spatial slot definitions | Board configuration |
| `UDA_SleightMarketConfig` | Shop behavior | Market configuration |
| `UDA_SleightAIConfig` | AI personality (rules + weights) | AI difficulty/style |
| `UDA_SleightRelic` | Persistent passive modifier | Relic/joker/artifact |
| `UDA_SleightKeyword` | Named ability definition | Keyword |

---

## Key Decisions Summary

| Decision | Choice | Why |
|----------|--------|-----|
| Zone identity | FGameplayTag | Games extend without touching plugin |
| Card identity | FGameplayTag | Same |
| Effect authoring | C++ subclasses + BP shell | Both audiences served |
| Combat | Optional swappable resolver | Not every game needs it |
| Market | Optional separate system | Not every game has one |
| Board | Optional separate module | Widget-only games skip entirely |
| AI | DA-configured scoring rules | Designer can tune without code |
| History | Optional event log | Adds cost, not always needed |
| Presentation | Separate module, delegate-driven | Core stays clean |
| Networking | Not in v1 | Add via GameSession action RPCs later |
| Undo | History log + optional rollback | Off by default |
| Serialization | JSON snapshot of ZoneManager + Players + RNG state | Replay from seed + action log |

---

## Build.cs Dependencies

```csharp
// Sleight.Build.cs
PublicDependencyModuleNames.AddRange(new string[] {
    "Core", "CoreUObject", "Engine", "GameplayTags"
});

// SleightBoard.Build.cs
PublicDependencyModuleNames.AddRange(new string[] {
    "Core", "CoreUObject", "Engine", "GameplayTags", "Sleight", "Niagara"
});

// SleightEditor.Build.cs
PublicDependencyModuleNames.AddRange(new string[] {
    "Core", "CoreUObject", "Engine", "GameplayTags",
    "Sleight", "SleightBoard",
    "UnrealEd", "Slate", "SlateCore", "PropertyEditor",
    "AssetTools", "ContentBrowser"
});
```
