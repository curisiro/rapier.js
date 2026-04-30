# Implementation Plan: Exporting Complete Joint Properties for Generic and Revolute Joints

## Background
Currently, Rapier JS bindings (`rapier.js`) force `GenericJoint` and `RevoluteJoint` to use a single `axis` for both `localAxis1` and `localAxis2`. Furthermore, the rotational components of the joints (`frame1` and `frame2`) are ignored entirely and only used in `FixedJoint`. This prevents developers from defining joints with complex misaligned local frames or different rotational axes.

## Proposed Changes

We will modify both the **TypeScript bindings** (`src.ts`) and the **Rust WebAssembly exports** (`src`) to fully support `localAxis1`, `localAxis2`, `frame1`, and `frame2` for `GenericJoint` and `RevoluteJoint` via new advanced factory methods.

### 1. TypeScript API Modifications (`src.ts/dynamics/impulse_joint.ts`)

- **Update `JointData` Interface**: Add `localAxis1` and `localAxis2` to the `JointData` class as optional properties (so existing code doesn't break).
- **New Methods `JointData.genericAdvanced` & `JointData.revoluteAdvanced`**:
  We will introduce new advanced factory methods that accept the complete set of parameters.
  ```typescript
  public static genericAdvanced(
      anchor1: Vector,
      anchor2: Vector,
      localAxis1: Vector,
      localAxis2: Vector,
      frame1: Rotation,
      frame2: Rotation,
      axesMask: JointAxesMask,
  ): JointData

  public static revoluteAdvanced(
      anchor1: Vector,
      anchor2: Vector,
      localAxis1: Vector,
      localAxis2: Vector,
      frame1: Rotation,
      frame2: Rotation,
  ): JointData
  ```
- **Update `JointData.intoRaw()`**: Modify the `JointType.Generic` and `JointType.Revolute` (or new internal `JointType` variants / logic) to pass these new parameters down to the WASM binary. If the `Advanced` properties are present, use the new WASM bindings; otherwise, use the legacy ones.

### 2. Rust API Modifications (`src/dynamics/joint.rs`)

- **New WASM Binding Methods**:
  Create new `#[wasm_bindgen]` functions to handle the advanced variants without breaking the existing bindings:
  ```rust
  pub fn generic_advanced(
      anchor1: &RawVector,
      anchor2: &RawVector,
      local_axis1: &RawVector,
      local_axis2: &RawVector,
      frame1: &RawRotation,
      frame2: &RawRotation,
      lockedAxes: u8,
  ) -> Option<RawGenericJoint>

  pub fn revolute_advanced(
      anchor1: &RawVector,
      anchor2: &RawVector,
      local_axis1: &RawVector,
      local_axis2: &RawVector,
      frame1: &RawRotation,
      frame2: &RawRotation,
  ) -> Option<RawGenericJoint>
  ```
- **Configure the Builders**:
  Inside the Rust bindings, we will use `Isometry::from_parts` to construct the local frames (combining `anchor` and `frame` orientations) and apply them via `.local_frame1()` and `.local_frame2()`. We will also apply `.local_axis1()` and `.local_axis2()` to ensure no rotational or axis data is lost.

## User Decisions
- **Breaking Changes**: Averted. The original `.generic()` and `.revolute()` methods will remain untouched. The new properties will be accessed via `.genericAdvanced()` and `.revoluteAdvanced()`.
- **Builder Configuration**: We will pass BOTH `frame` and `axis` down to the Rust builder to explicitly configure the joint's local geometry.
