# Task Tracker: Advanced Joint Support

- [x] Modify `src.ts/dynamics/impulse_joint.ts`
  - [x] Add optional `localAxis1`, `localAxis2` and `isAdvanced` to `JointData`.
  - [x] Add `genericAdvanced()` and `revoluteAdvanced()` static methods.
  - [x] Update `intoRaw()` to route calls to the new WASM methods when `isAdvanced` is true.
- [x] Modify `src/dynamics/joint.rs`
  - [x] Add `generic_advanced` function to `RawGenericJoint`.
  - [x] Add `revolute_advanced` function to `RawGenericJoint`.
- [ ] Instruct user to run build and verify.
