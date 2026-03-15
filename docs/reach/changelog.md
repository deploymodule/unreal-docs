# Reach — Changelog

## v1.0.0 — Initial Release

### Added
- `UReachDetectorComponent` with periodic sphere sweep detection, priority sorting, and Enhanced Input binding.
- `UReachInteractableComponent` with four interaction modes: Instant, Hold, Smash, Rhythm.
- `CanInteract` virtual for custom condition logic in C++ subclasses or Blueprint.
- Replication via server-authoritative RPC with distance validation.
- `OnInteractableFound`, `OnInteractableLost`, `OnFocusChanged` delegates on the detector.
- `OnInteracted`, `OnHoldProgress`, `OnHoldCancelled`, `OnSmashProgress` delegates on the interactable.
- `RhythmCurve` support for timed-press interaction patterns.
- Full Blueprint exposure for all component properties and functions.
- Zero forced base classes — works with any `APawn`/`AActor` subclass.
