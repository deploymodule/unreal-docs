# Kinetic — Changelog

## v1.0.0 — Initial Release

### Added
- `FAnimNode_KineticAdditiveApply` — layered additive blending with bone mask and global weight.
- `FAnimNode_KineticBodyDynamics` — thread-safe spring-physics secondary motion for bone chains.
- `FAnimNode_KineticStateProxy` — single-node locomotion proxy replacing multi-node blend space setups.
- `UKineticBoneMaskAsset` Data Asset with per-bone weight entries and `bIncludeChildren` propagation.
- Blueprint-accessible properties on all animation nodes via pin exposure.
- Full compatibility with Linked Anim Layers and Control Rig.
- Editor thumbnail and category registration under **Kinetic** in the Anim Graph node picker.
