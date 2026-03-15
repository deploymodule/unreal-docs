# Sleight — Universal Card Game Framework: Analysis & Architecture

## Design Philosophy

Sleight is a **data-driven, presentation-agnostic** card game framework. The core logic layer knows nothing about 3D actors, widgets, or rendering. It operates on pure data — decks, zones, cards, rules — and fires delegates. A **presentation layer** (3D board, widget-only UI, hybrid) subscribes to those delegates and visualizes the state.

This means the same Sleight core can power:
- A Slay the Spire-style deckbuilder with 3D cards on a table
- A Hearthstone-style widget-only board
- A Gwent-style row-based game
- A Balatro-style poker deckbuilder
- A Yu-Gi-Oh-style zone-heavy TCG
- A Dominion-style pure deckbuilder with a market
- A Marvel Snap-style lane game
- A simple war/matching card game

---

## Part 1: Universal Card Game Concepts (What ALL Card Games Need)

### 1.1 The Card (Data)

Every card game has cards. A card is fundamentally **an identity + a bag of properties**.

```
FSleightCardData (Data Asset / Struct)
├── CardID (FGameplayTag or FName) — unique identity
├── DisplayName (FText)
├── Description (FText)
├── CardArt (TSoftObjectPtr<UTexture2D>)
├── CardType (FGameplayTag) — e.g. "CardType.Attack", "CardType.Spell", "CardType.Creature"
├── Rarity (FGameplayTag) — e.g. "Rarity.Common", "Rarity.Legendary"
├── Cost (int32) — generic resource cost (mana, energy, gold, etc.)
├── Tags (FGameplayTagContainer) — arbitrary tags for filtering/effects
├── Properties (TMap<FGameplayTag, int32>) — e.g. Attack=5, Health=3, Duration=2
├── TextProperties (TMap<FGameplayTag, FText>) — flavor text, lore keys
├── Effects (TArray<USleightCardEffect*>) — what happens when played/discarded/etc.
└── Metadata (TMap<FName, FString>) — extensible key-value for game-specific needs
```

**Key design choice:** Cards are **templates** (Data Assets). When a card is in play, it becomes a **card instance** with mutable state (current HP, buffs, modified cost, etc.):

```
FSleightCardInstance (Runtime)
├── InstanceID (FGuid) — unique per-instance (two copies of same card have different IDs)
├── CardData (UDA_SleightCard*) — points back to template
├── OwnerID (int32) — which player owns this instance
├── CurrentZone (FGameplayTag) — where it currently lives
├── ModifiedProperties (TMap<FGameplayTag, int32>) — overrides for base properties
├── StatusEffects (TArray<FSleightStatusEffect>) — active buffs/debuffs
├── TurnPlayed (int32) — when it entered play
├── IsFaceUp (bool) — visibility state
├── IsExhausted/Tapped (bool) — has been used this turn
└── CustomState (TMap<FName, int32>) — game-specific mutable state
```

### 1.2 Zones (Where Cards Live)

Every card game has **zones** — named regions where cards can exist. The set of zones varies by game, but the concept is universal.

```
Common Zones (tagged, not hardcoded):
├── Zone.Library / Zone.Deck          — draw pile (ordered, face-down)
├── Zone.Hand                         — player's hand (ordered, visible to owner)
├── Zone.Play / Zone.Board / Zone.Field — in-play area
├── Zone.Graveyard / Zone.Discard     — used/destroyed cards
├── Zone.Exile / Zone.Banish / Zone.Removed — removed from game entirely
├── Zone.Market / Zone.Shop / Zone.Supply — cards available to buy/draft
├── Zone.Stack / Zone.Chain           — resolution stack (MTG stack, Yu-Gi-Oh chain)
├── Zone.Limbo / Zone.Pending         — cards in transit / being resolved
├── Zone.Sideboard / Zone.Reserve     — pre-game card pool
├── Zone.Equipped / Zone.Attached     — cards attached to other cards
└── Zone.Reveal                       — temporarily revealed cards
```

**Zone properties:**
```
FSleightZone
├── ZoneTag (FGameplayTag) — identity
├── OwnerID (int32) — -1 for shared zones (market, stack)
├── Cards (TArray<FSleightCardInstance*>) — ordered list
├── MaxCapacity (int32) — -1 for unlimited (hand limit, board limit)
├── Visibility (ESleightVisibility) — Public, OwnerOnly, Hidden, TopOnly
├── IsOrdered (bool) — does order matter? (deck=yes, graveyard=game-dependent)
├── AllowShuffle (bool)
├── SlotTags (TArray<FGameplayTag>) — for positional zones (lanes, rows, columns)
└── ZoneRules (TArray<USleightZoneRule*>) — constraints (e.g., "max 3 creatures in this row")
```

### 1.3 Zone Transfer (Moving Cards Between Zones)

The single most important operation. Every card game action ultimately resolves to one or more zone transfers.

```
FSleightZoneTransfer
├── CardInstance (FSleightCardInstance*)
├── SourceZone (FGameplayTag)
├── DestZone (FGameplayTag)
├── InsertPosition (ESleightInsertPosition) — Top, Bottom, Random, AtIndex, Shuffled
├── InsertIndex (int32) — for AtIndex
├── RevealOnTransfer (bool)
├── Reason (FGameplayTag) — why (Draw, Play, Discard, Destroy, Mill, Bounce, Exile, Buy, Steal)
└── Instigator (int32) — who caused this transfer
```

**Delegates fired:**
- `OnPreZoneTransfer` — can be intercepted/cancelled/redirected
- `OnPostZoneTransfer` — notification after the fact
- `OnCardEnteredZone` — zone-specific (e.g., "when a card enters the graveyard")
- `OnCardLeftZone` — zone-specific

### 1.4 The Deck (A Special Zone + Construction)

A deck is a Zone.Deck with additional operations:

```
Deck Operations:
├── Shuffle() — Fisher-Yates, fires OnDeckShuffled
├── Draw(Count) — transfer top N cards to Hand, fires OnCardDrawn per card
├── Mill(Count) — transfer top N to Graveyard
├── Peek(Count) — return top N without moving (for scry/surveil)
├── Search(Filter) — find cards matching criteria (tutoring)
├── InsertAt(Card, Position) — put card on top/bottom/at index
├── DrawFromBottom(Count)
├── RevealTop(Count) — flip face-up without moving
└── CountRemaining() — cards left
```

**Deck construction (pre-game):**
```
FSleightDeckRecipe
├── DeckName (FText)
├── Cards (TMap<UDA_SleightCard*, int32>) — card template → count
├── MinSize (int32) — e.g., 40 for Yu-Gi-Oh, 60 for MTG
├── MaxSize (int32) — e.g., 60, or -1 for no max
├── MaxCopies (int32) — e.g., 3 for Yu-Gi-Oh, 4 for MTG, 2 for Marvel Snap
├── DeckRules (TArray<USleightDeckRule*>) — validation rules
└── Validate() → bool + TArray<FText> errors
```

### 1.5 The Hand

A Zone.Hand with game logic:

```
Hand Operations:
├── PlayCard(CardInstance, TargetInfo) — attempt to play from hand
├── Discard(CardInstance) — voluntary discard
├── DiscardRandom(Count)
├── DiscardToHandSize(MaxSize) — end-of-turn hand limit
├── Mulligan(CardsToReturn) → redraw
├── RevealToOpponent(CardInstance)
├── GetPlayableCards(ResourceState) → filtered list of cards you can afford
└── SortHand(SortMode) — by cost, type, name, etc.
```

### 1.6 Resources / Economy

Almost every card game has a resource system. Mana, energy, gold, action points, etc.

```
FSleightResourcePool
├── ResourceTag (FGameplayTag) — e.g., "Resource.Mana", "Resource.Energy", "Resource.Gold"
├── Current (int32)
├── Maximum (int32) — cap
├── PerTurnGain (int32) — auto-replenish
├── Temporary (int32) — bonus that doesn't persist (e.g., "add 1 mana this turn only")
├── CanGoNegative (bool)
├── AutoRefillOnTurnStart (bool) — most games refill to max
└── RefillAmount (int32) — how much to restore (-1 = full refill)
```

**Multiple resource types supported** — some games have mana + gold + actions, etc.

### 1.7 Turn Structure

```
ESleightTurnPhase (configurable per game):
├── GameStart         — initial setup, mulligans
├── TurnStart         — begin turn triggers
├── Upkeep            — maintenance (untap, refresh, resource gain)
├── Draw              — draw phase
├── PreMain           — before main action phase
├── Main              — play cards, take actions
├── Combat/Battle     — attack/defend resolution
├── PostMain          — after combat, second main phase
├── End               — end-of-turn triggers, hand size check
├── Cleanup           — remove "until end of turn" effects
├── OpponentResponse  — priority/reaction window
└── Custom            — game-specific phases via tags
```

```
FSleightTurnStructure (Data Asset — defines the flow)
├── Phases (TArray<FGameplayTag>) — ordered phase sequence
├── PhaseRules (TMap<FGameplayTag, USleightPhaseRule*>) — what happens in each phase
├── AllowActionsInPhase (TMap<FGameplayTag, FGameplayTagContainer>) — which action types are legal
├── AutoAdvance (TMap<FGameplayTag, bool>) — does the phase auto-advance or wait for player input?
├── PrioritySystem (ESleightPriority) — ActivePlayer, Alternating, Simultaneous, Stack
└── MaxTurns (int32) — -1 for no limit
```

### 1.8 Effects / Abilities System

Cards DO things. This is the scriptable behavior layer.

```
USleightCardEffect (UObject, abstract base)
├── TriggerCondition (ESleightTrigger):
│   ├── OnPlay         — when card is played from hand
│   ├── OnEnterZone    — when card enters any zone
│   ├── OnLeaveZone    — when card leaves a zone
│   ├── OnDeath        — when card is destroyed
│   ├── OnDiscard      — when card is discarded
│   ├── OnTurnStart    — at start of owner's turn
│   ├── OnTurnEnd      — at end of owner's turn
│   ├── OnAttack       — when card attacks
│   ├── OnDefend       — when card blocks
│   ├── OnDamaged      — when card takes damage
│   ├── OnDraw         — when card is drawn
│   ├── OnReveal       — when card is revealed (Marvel Snap style)
│   ├── Passive        — always active while in play
│   ├── Activated      — player-triggered ability (tap/exhaust)
│   └── Custom         — tag-driven trigger
├── TargetingRule (USleightTargetRule*) — who/what can be targeted
├── Execute(Context) — pure virtual
└── CanExecute(Context) → bool — validation
```

**Common effect types (subclasses):**
```
USleightEffect_DealDamage          — deal N damage to target
USleightEffect_Heal                — restore N health
USleightEffect_DrawCards            — draw N cards
USleightEffect_DiscardCards         — force discard
USleightEffect_ModifyProperty       — +N/-N to a property (buff/debuff)
USleightEffect_AddStatusEffect      — apply a status
USleightEffect_MoveCard             — zone transfer (bounce, exile, etc.)
USleightEffect_SpawnCard            — create a token/copy
USleightEffect_ModifyCost           — change card costs
USleightEffect_GainResource         — add resources
USleightEffect_DestroyCard          — destroy target card
USleightEffect_TransformCard        — change a card into another
USleightEffect_ShuffleDeck          — shuffle a player's deck
USleightEffect_SearchDeck           — tutor (search deck for card)
USleightEffect_Conditional          — if/then effect wrapper
USleightEffect_Composite            — multiple effects in sequence
USleightEffect_Random               — pick from N effects randomly
USleightEffect_RepeatForEach        — for each card matching filter, do X
USleightEffect_PlayerChoice         — let the player choose between effects
```

### 1.9 Targeting System

```
USleightTargetRule (abstract)
├── RequiresTarget (bool) — some effects are untargeted
├── TargetCount (int32) — how many targets needed
├── TargetType (ESleightTargetType) — Card, Player, Zone, Slot, None
├── ValidTargets(Context) → TArray — compute valid targets
├── IsValidTarget(Context, Target) → bool
└── AutoTarget (bool) — auto-select if only one valid target
```

**Common targeting rules:**
```
USleightTarget_Any                 — any card in play
USleightTarget_Friendly            — only your cards
USleightTarget_Enemy               — only opponent's cards
USleightTarget_InZone              — cards in specific zone
USleightTarget_WithTag             — cards with specific tag
USleightTarget_WithProperty        — cards where property meets condition
USleightTarget_Self                — the card itself
USleightTarget_Adjacent            — cards in adjacent slots (positional games)
USleightTarget_Row                 — all cards in a row (Gwent)
USleightTarget_Lane                — all cards in a lane (Marvel Snap)
USleightTarget_Random              — random from valid pool
USleightTarget_All                 — all cards matching filter (board wipe)
USleightTarget_Player              — target a player (direct damage)
```

### 1.10 Combat / Conflict Resolution

Not all card games have combat, but many do. This should be opt-in.

```
USleightCombatResolver (abstract, swappable)
├── ResolveCombat(Attacker, Defender, Context)
├── CalculateDamage(Source, Target) → int32
├── OnCombatResolved delegate
└── Subclasses:
    ├── USleightCombat_Direct         — attacker deals damage to target (Hearthstone)
    ├── USleightCombat_Mutual         — both deal damage to each other (MTG blocking)
    ├── USleightCombat_RowBased       — compare row totals (Gwent)
    ├── USleightCombat_LaneBased      — per-lane resolution (Marvel Snap)
    ├── USleightCombat_PlayerTarget   — attack goes to player HP (Hearthstone face)
    └── USleightCombat_Custom         — blueprint-overridable
```

---

## Part 2: Deckbuilder-Specific Systems

### 2.1 The Market / Shop / Supply

The defining feature of deckbuilders — a shared pool of cards players can acquire.

```
USleightMarket (UObject)
├── MarketType (ESleightMarketType):
│   ├── FixedDisplay  — N cards face-up, refill when bought (Ascension, Star Realms)
│   ├── StackedPiles  — separate stacks per card type (Dominion)
│   ├── DraftPool     — cards from a shared draft (7 Wonders)
│   ├── RandomOffer   — refresh randomly (roguelike shops)
│   └── Auction       — players bid (rare, but exists)
├── DisplaySize (int32) — how many cards visible
├── RefillPolicy (ESleightRefillPolicy) — Immediate, EndOfTurn, Manual, WhenEmpty
├── CostResource (FGameplayTag) — what resource buys cards (Gold, Mana, Influence)
├── AvailableCards (TArray<FSleightMarketSlot>) — current offerings
├── SupplyDeck (deck of cards to refill from)
├── CanTrash (bool) — can you remove market cards (Ascension banish)
├── BuyCard(PlayerID, SlotIndex)
├── RefreshMarket()
└── OnCardBought delegate
```

### 2.2 Card Upgrade / Transform System

Many deckbuilders let you upgrade cards (Slay the Spire) or transform them.

```
FSleightUpgradePath
├── SourceCard (UDA_SleightCard*)
├── UpgradedCard (UDA_SleightCard*)
├── Cost (int32)
├── RequiredTag (FGameplayTag) — e.g., must be at a rest site
└── UpgradeType (ESleightUpgradeType) — Enhance, Transform, Split, Merge
```

### 2.3 Card Removal / Thinning

Deckbuilders need ways to remove cards from your deck permanently.

```
Operations:
├── TrashCard(CardInstance) — remove from game permanently
├── ExileCard(CardInstance) — remove but may return
├── SellCard(CardInstance) → gain resources
└── TransformCard(CardInstance, NewCard) — swap for different card
```

### 2.4 Relics / Artifacts / Passive Modifiers

Persistent modifiers that aren't cards but affect gameplay (Slay the Spire relics, Balatro jokers).

```
UDA_SleightRelic (Data Asset)
├── RelicID (FGameplayTag)
├── DisplayName, Description, Icon
├── PassiveEffects (TArray<USleightCardEffect*>) — always-on effects
├── TriggeredEffects — effects with conditions
├── Rarity
├── IsUnique (bool) — can only have one
└── Counters (TMap<FName, int32>) — mutable state (e.g., "charges")
```

---

## Part 3: Board / Spatial Layout System

### 3.1 Abstract Board Layout

A configurable spatial system for positional card games.

```
UDA_SleightBoardLayout (Data Asset)
├── BoardType (ESleightBoardType):
│   ├── FreeForm      — no fixed positions (Hearthstone)
│   ├── Grid          — NxM grid
│   ├── Lanes         — N vertical lanes (Marvel Snap = 3)
│   ├── Rows          — N horizontal rows per player (Gwent = 3)
│   ├── Ring          — circular arrangement
│   └── Custom        — defined by slot array
├── Slots (TArray<FSleightBoardSlot>):
│   ├── SlotTag (FGameplayTag) — e.g., "Slot.Lane1.Player1"
│   ├── Position (FVector2D) — abstract position on board
│   ├── OwnerID (int32) — -1 for neutral
│   ├── MaxCards (int32) — cards allowed in this slot
│   ├── AcceptFilter (FGameplayTagQuery) — what card types fit here
│   └── AdjacentSlots (TArray<FGameplayTag>) — neighbors for adjacency effects
├── PlayerAreas (TMap<int32, TArray<FGameplayTag>>) — which slots belong to which player
├── SharedAreas (TArray<FGameplayTag>) — market, stack, etc.
└── Symmetry (bool) — mirror layout for player 2
```

### 3.2 Board State

```
USleightBoardState (runtime)
├── Layout (UDA_SleightBoardLayout*)
├── SlotContents (TMap<FGameplayTag, TArray<FSleightCardInstance*>>)
├── GetCardsInSlot(SlotTag) → array
├── GetAdjacentCards(SlotTag) → array
├── GetAllCardsForPlayer(PlayerID) → array
├── FindEmptySlot(Filter) → SlotTag
├── PlaceCard(Card, SlotTag) — with validation
├── RemoveCard(Card)
├── MoveCard(Card, FromSlot, ToSlot)
└── OnBoardChanged delegate
```

---

## Part 4: Player System

```
FSleightPlayer
├── PlayerID (int32)
├── DisplayName (FText)
├── Health / Life (int32) — if applicable
├── MaxHealth (int32)
├── Resources (TMap<FGameplayTag, FSleightResourcePool>)
├── Zones owned (Deck, Hand, Graveyard, Play, Exile — zone refs)
├── Relics (TArray<UDA_SleightRelic*>)
├── Score (int32) — for point-based games
├── IsAI (bool)
├── TurnOrder (int32)
├── HasPriority (bool) — for stack/response systems
├── MulligansTaken (int32)
└── PlayerTags (FGameplayTagContainer) — status effects on the player
```

```
Player Actions:
├── DrawCards(Count)
├── PlayCard(CardInstance, TargetInfo)
├── ActivateAbility(CardInstance)
├── AttackWith(CardInstance, Target)
├── BuyFromMarket(SlotIndex)
├── DiscardCard(CardInstance)
├── EndPhase() / PassPriority()
├── Mulligan(CardsToReturn)
├── Concede()
└── UseResource(ResourceTag, Amount)
```

---

## Part 5: Game Session / Match Controller

The orchestrator that ties everything together.

```
USleightGameSession (UObject or ActorComponent)
├── GameRules (UDA_SleightGameRules*) — the ruleset data asset
├── Players (TArray<FSleightPlayer>)
├── TurnNumber (int32)
├── ActivePlayerIndex (int32)
├── CurrentPhase (FGameplayTag)
├── TurnStructure (UDA_SleightTurnStructure*)
├── BoardState (USleightBoardState*)
├── Market (USleightMarket*) — optional
├── CombatResolver (USleightCombatResolver*) — optional
├── EffectStack (TArray<FSleightPendingEffect>) — resolution stack
├── GameHistory (TArray<FSleightGameEvent>) — full log for replay/undo
├── RNG (FRandomStream) — seeded for determinism/replay
│
├── StartGame()
├── EndGame(WinnerID, Reason)
├── AdvancePhase()
├── AdvanceTurn()
├── ProcessAction(PlayerID, Action) → result
├── CheckWinCondition() → ESleightGameResult
├── Undo() — if enabled
├── GetGameState() → serializable snapshot
├── LoadGameState(snapshot)
│
├── Delegates:
│   ├── OnGameStarted
│   ├── OnGameEnded(WinnerID, Reason)
│   ├── OnTurnStarted(PlayerID, TurnNumber)
│   ├── OnTurnEnded(PlayerID)
│   ├── OnPhaseChanged(Phase)
│   ├── OnCardPlayed(PlayerID, CardInstance, TargetInfo)
│   ├── OnCardDrawn(PlayerID, CardInstance)
│   ├── OnCardDiscarded(PlayerID, CardInstance)
│   ├── OnCardDestroyed(CardInstance, Reason)
│   ├── OnCardMoved(CardInstance, FromZone, ToZone, Reason)
│   ├── OnDamageDealt(Source, Target, Amount)
│   ├── OnPlayerHealthChanged(PlayerID, OldHP, NewHP)
│   ├── OnResourceChanged(PlayerID, ResourceTag, OldAmount, NewAmount)
│   ├── OnMarketCardBought(PlayerID, Card)
│   ├── OnEffectResolved(Effect, Context)
│   ├── OnPlayerActionRequested(PlayerID, ValidActions) — UI prompt
│   └── OnGameStateChanged — any mutation
```

---

## Part 6: Win Conditions

```
USleightWinCondition (abstract, data-driven)
├── USleightWin_ReduceHealthToZero  — opponent HP <= 0
├── USleightWin_DeckOut             — opponent can't draw (mill win)
├── USleightWin_HighestScore        — most points when game ends
├── USleightWin_CollectSet          — collect N of something
├── USleightWin_SurviveNTurns       — last N turns
├── USleightWin_EmptyDeck           — play all your cards (Dominion-ish)
├── USleightWin_BoardControl        — control N slots/lanes
├── USleightWin_SpecialCard         — play a specific win-condition card
├── USleightWin_Composite           — multiple conditions (any/all)
└── USleightWin_Custom              — blueprint-overridable
```

---

## Part 7: AI / Opponent System

```
USleightAIBrain (abstract)
├── EvaluateGameState(State) → float — heuristic
├── ChooseAction(ValidActions) → FSleightAction
├── ChooseTargets(TargetRule, ValidTargets) → TArray
├── Difficulty (ESleightAIDifficulty) — Easy, Medium, Hard, Cheating
│
├── Subclasses:
│   ├── USleightAI_Random           — pick random valid action
│   ├── USleightAI_Greedy           — pick highest immediate value
│   ├── USleightAI_RuleBased        — priority-list heuristics
│   ├── USleightAI_MCTS             — Monte Carlo Tree Search (advanced)
│   └── USleightAI_Blueprint        — BP-overridable for custom AI
```

---

## Part 8: Presentation Layer (SEPARATE from Core)

### 8.1 3D Board Presentation

```
ASleightBoard3D (Actor)
├── Board mesh, materials, layout spawning
├── CardSlot3D actors — positions on the table
├── Subscribes to GameSession delegates
├── Animates card movement (lerp, arc, flip)
├── Card actors: ASleightCard3D
│   ├── Front/Back mesh
│   ├── Dynamic material for card art
│   ├── Highlight/selection state
│   ├── Hover preview
│   └── Physics (optional — card toss, scatter)
```

### 8.2 Widget Board Presentation

```
USleightBoardWidget (UserWidget)
├── Zone widgets (hand panel, board grid, graveyard icon, deck count)
├── Card widgets: USleightCardWidget
│   ├── Card image
│   ├── Stats overlay
│   ├── Drag & drop support
│   ├── Hover tooltip
│   └── Play animation (scale, fade, slide)
├── Subscribes to same GameSession delegates
└── Fully functional without any 3D actors
```

### 8.3 Hybrid (3D + Widget overlay)

Use 3D board for play area, widgets for hand/UI. Both presentation layers can coexist.

---

## Part 9: Serialization & Persistence

```
Save/Load:
├── FSleightSaveData — full game state snapshot
│   ├── All zone contents (card instance IDs + zone tags)
│   ├── All player states (HP, resources, relics)
│   ├── Board state
│   ├── Turn number, phase, active player
│   ├── RNG seed + call count (for deterministic replay)
│   └── Game history log
├── SerializeToJson() / DeserializeFromJson()
├── Save to slot (USaveGame)
└── Replay system — replay from RNG seed + action log
```

---

## Part 10: Game Rules Data Asset (The "Cartridge")

One data asset defines an entire card game. Swap it out to play a different game with the same engine.

```
UDA_SleightGameRules (Primary Data Asset)
├── GameName (FText)
├── PlayerCount (FIntPoint) — min/max players
├── StartingHealth (int32) — 0 = no health system
├── StartingHandSize (int32)
├── MaxHandSize (int32) — -1 for no limit
├── DrawPerTurn (int32)
├── TurnStructure (UDA_SleightTurnStructure*)
├── BoardLayout (UDA_SleightBoardLayout*)
├── ResourceDefinitions (TArray<FSleightResourceDef>)
├── DeckRules (MinSize, MaxSize, MaxCopies)
├── WinConditions (TArray<USleightWinCondition*>)
├── MarketConfig (optional)
├── CombatResolver (optional)
├── MulliganRules
├── CardPool (TArray<UDA_SleightCard*>) — all legal cards
├── StartingCards (TArray<UDA_SleightCard*>) — initial deck/hand
├── StatusEffectDefinitions
├── KeywordDefinitions (TMap<FGameplayTag, FText>) — keyword tooltips
└── CustomRules (TMap<FName, FString>) — extensible
```

---

## Part 11: Undo / History System

```
FSleightGameEvent
├── EventID (int32) — sequential
├── EventType (ESleightEventType)
├── Timestamp (FDateTime)
├── PlayerID (int32)
├── CardInstanceID (FGuid) — if card-related
├── SourceZone, DestZone
├── OldValues, NewValues (for property changes)
├── IsUndoable (bool)
└── ChildEvents (TArray<int32>) — cascaded effects
```

Full event log enables:
- Undo/Redo (for single-player / PvE)
- Replay from seed
- Match history review
- Tutorial highlighting ("here's what just happened")

---

## Part 12: Keyword / Ability System

```
UDA_SleightKeyword (Data Asset)
├── KeywordTag (FGameplayTag) — e.g., "Keyword.Taunt", "Keyword.Flying"
├── DisplayName (FText)
├── Description (FText) — tooltip text
├── ReminderText (FText) — short rules text
├── Icon (UTexture2D*)
├── Effect (USleightCardEffect*) — what the keyword actually does
├── IsStackable (bool) — can appear multiple times
└── IsNegative (bool) — for UI coloring
```

---

## Part 13: Networking Considerations

```
Replication Strategy:
├── Authoritative server model
├── GameSession lives on server
├── Clients send Actions (validated server-side)
├── Server broadcasts state deltas
├── Hidden information: server never sends face-down cards to wrong client
├── Spectator mode: full state or fog-of-war
└── RPC pattern: Client→Server (RequestAction), Server→Client (OnGameEvent multicast)
```

---

## Part 14: Module / Plugin Structure

```
Plugins/Sleight/
├── Sleight.uplugin
├── Source/
│   ├── Sleight/                          — Runtime module
│   │   ├── Sleight.Build.cs
│   │   ├── Public/
│   │   │   ├── Sleight.h                 — Module def
│   │   │   ├── Data/
│   │   │   │   ├── SleightCardData.h     — UDA_SleightCard
│   │   │   │   ├── SleightGameRules.h    — UDA_SleightGameRules
│   │   │   │   ├── SleightBoardLayout.h  — UDA_SleightBoardLayout
│   │   │   │   ├── SleightTurnStructure.h
│   │   │   │   ├── SleightKeyword.h
│   │   │   │   ├── SleightRelic.h
│   │   │   │   └── SleightTypes.h        — enums, structs, tags
│   │   │   ├── Core/
│   │   │   │   ├── SleightGameSession.h  — main orchestrator
│   │   │   │   ├── SleightZone.h         — zone logic
│   │   │   │   ├── SleightDeck.h         — deck operations
│   │   │   │   ├── SleightHand.h         — hand logic
│   │   │   │   ├── SleightPlayer.h       — player state
│   │   │   │   ├── SleightMarket.h       — shop/market
│   │   │   │   ├── SleightBoardState.h   — spatial board
│   │   │   │   ├── SleightEffectStack.h  — resolution stack
│   │   │   │   └── SleightHistory.h      — event log / undo
│   │   │   ├── Effects/
│   │   │   │   ├── SleightCardEffect.h   — base effect class
│   │   │   │   ├── SleightEffect_*.h     — effect subclasses
│   │   │   │   └── SleightTargetRule.h   — targeting rules
│   │   │   ├── Combat/
│   │   │   │   ├── SleightCombatResolver.h
│   │   │   │   └── SleightCombat_*.h     — resolver subclasses
│   │   │   ├── AI/
│   │   │   │   ├── SleightAIBrain.h
│   │   │   │   └── SleightAI_*.h         — AI strategies
│   │   │   ├── WinConditions/
│   │   │   │   └── SleightWin_*.h
│   │   │   └── SleightFunctionLibrary.h  — BFL for Blueprint/MCP access
│   │   └── Private/
│   │       └── (mirror of Public)
│   ├── SleightPresentation/               — Presentation module (optional, separate)
│   │   ├── Public/
│   │   │   ├── Board3D/
│   │   │   │   ├── SleightBoard3D.h      — 3D board actor
│   │   │   │   ├── SleightCard3D.h       — 3D card actor
│   │   │   │   └── SleightSlot3D.h       — 3D slot actor
│   │   │   └── Widgets/
│   │   │       ├── SleightBoardWidget.h   — widget-only board
│   │   │       ├── SleightCardWidget.h    — card widget
│   │   │       ├── SleightHandWidget.h    — hand panel
│   │   │       ├── SleightZoneWidget.h    — zone display
│   │   │       └── SleightMarketWidget.h  — shop widget
│   │   └── Private/
│   └── SleightEditor/                     — Editor module
│       └── (detail customizations, card editor, rule validator)
```

---

## Part 15: Example Game Configurations

### Slay the Spire Clone
- 1 player, StartingHealth=80, DrawPerTurn=5
- Zones: Deck, Hand, Discard, Exhaust(Exile), Draw
- Market: RandomOffer after each combat
- Resources: Energy (3/turn, auto-refill)
- Combat: Direct (player attacks enemies with card effects)
- WinCondition: ReduceHealthToZero (enemies)
- Relics: Yes
- Board: FreeForm (no slots)

### Dominion Clone
- 2-4 players, No health, DrawPerTurn=5
- Market: StackedPiles (10 kingdom piles + basic piles)
- Resources: Actions(1/turn), Buys(1/turn), Gold(0/turn, generated by treasure cards)
- Combat: None
- WinCondition: HighestScore (count victory point cards when 3 piles empty)
- Board: FreeForm

### Hearthstone Clone
- 2 players, StartingHealth=30, DrawPerTurn=1
- Resources: Mana (increases 1/turn, max 10, auto-refill)
- Combat: Direct (minions attack face or other minions)
- Board: FreeForm, MaxCards=7 per player
- WinCondition: ReduceHealthToZero

### Gwent Clone
- 2 players, No direct health, 3 rounds
- Board: Rows (3 rows per player: Melee, Ranged, Siege)
- Combat: RowBased (compare total power per round)
- WinCondition: Win 2 of 3 rounds (highest row total)
- No mana — play 1 card per turn or pass

### Marvel Snap Clone
- 2 players, 6 turns, DrawPerTurn=1
- Board: Lanes (3 lanes, 4 cards per lane per player)
- Resources: Energy (increases 1/turn, starts at 1)
- Combat: LaneBased (compare total power per lane, win 2/3 lanes)
- WinCondition: BoardControl (most lanes won after turn 6)
- OnReveal effects

### Balatro Clone
- 1 player, score-based
- Resources: Dollars, Discards, Hands
- No combat — poker hand evaluation
- Market: RandomOffer (shop between rounds)
- Relics: Jokers (persistent modifiers)
- WinCondition: Score target per round

---

## Part 16: Tag Structure (Suggested GameplayTag Hierarchy)

```
Sleight
├── Card
│   ├── Type (Attack, Skill, Power, Creature, Spell, Trap, Equipment, Resource, Land)
│   ├── Rarity (Common, Uncommon, Rare, Epic, Legendary)
│   ├── Element (Fire, Water, Earth, Air, Light, Dark, Neutral)
│   └── Tribe (Human, Beast, Undead, Mech, Dragon)
├── Zone (Deck, Hand, Play, Graveyard, Exile, Market, Stack, Limbo, Sideboard, Equipped, Reveal)
├── Phase (GameStart, TurnStart, Upkeep, Draw, PreMain, Main, Combat, PostMain, End, Cleanup)
├── Resource (Mana, Energy, Gold, Actions, Buys, Health)
├── Keyword (Taunt, Charge, Stealth, Flying, Trample, Lifesteal, Poison, Shield, Rush)
├── Trigger (OnPlay, OnDeath, OnDiscard, OnDraw, OnReveal, OnTurnStart, OnTurnEnd, OnAttack, OnDamaged, Passive)
├── Slot (Lane1, Lane2, Lane3, Row.Melee, Row.Ranged, Row.Siege, Grid.R0C0, ...)
├── Action (Play, Draw, Discard, Attack, Block, Buy, Sell, Trash, EndTurn, Pass, Activate, Mulligan)
├── Transfer (Draw, Play, Discard, Destroy, Mill, Bounce, Exile, Buy, Steal, Shuffle, Tutor)
└── Win (HealthZero, DeckOut, Score, BoardControl, SpecialCard, Survive, Concede)
```

---

## Summary: Core Subsystems Checklist

| # | System | Required | Deckbuilder | Notes |
|---|--------|----------|-------------|-------|
| 1 | Card Data (template + instance) | YES | YES | Foundation of everything |
| 2 | Zone System | YES | YES | All card games need this |
| 3 | Zone Transfer + Delegates | YES | YES | The core operation |
| 4 | Deck (shuffle, draw, peek, mill) | YES | YES | Special zone behavior |
| 5 | Hand (play, discard, mulligan) | YES | YES | Special zone behavior |
| 6 | Resource System | MOST | YES | Mana, energy, gold |
| 7 | Turn Structure | YES | YES | Phase sequencing |
| 8 | Effect/Ability System | YES | YES | What cards do |
| 9 | Targeting System | YES | YES | Who effects hit |
| 10 | Combat Resolver | OPTIONAL | SOME | Swappable strategies |
| 11 | Market/Shop | OPTIONAL | YES | Defining deckbuilder feature |
| 12 | Card Upgrade/Transform | OPTIONAL | YES | Slay the Spire, etc. |
| 13 | Relic/Artifact System | OPTIONAL | YES | Persistent modifiers |
| 14 | Board Layout (positional) | OPTIONAL | RARE | Lanes, rows, grids |
| 15 | Player State | YES | YES | HP, resources, ownership |
| 16 | Game Session Controller | YES | YES | Orchestrator |
| 17 | Win Conditions | YES | YES | Data-driven |
| 18 | AI System | OPTIONAL | YES (PvE) | For single-player |
| 19 | Keyword System | YES | YES | Reusable named abilities |
| 20 | History/Undo | OPTIONAL | YES | Replay, undo |
| 21 | Serialization/Save | YES | YES | Persistence |
| 22 | 3D Presentation | OPTIONAL | OPTIONAL | Board + card actors |
| 23 | Widget Presentation | OPTIONAL | OPTIONAL | UI-only board |
| 24 | Networking | OPTIONAL | OPTIONAL | Multiplayer |
| 25 | Deck Construction/Validation | YES | YES | Pre-game deck building |
| 26 | Game Rules Data Asset | YES | YES | The "cartridge" |
