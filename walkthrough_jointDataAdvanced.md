# Walkthrough: Advanced Generic and Revolute Joints

I have successfully updated the Rapier source code to expose the complete rotational properties (`frame1`, `frame2`) and `localAxis1`, `localAxis2` parameters for `GenericJoint` and `RevoluteJoint`.

## Changes Implemented

### 1. TypeScript API (JS Bindings)
Added two new methods to the `JointData` class so that your existing code won't be broken:
- `JointData.genericAdvanced(...)`
- `JointData.revoluteAdvanced(...)`

Both of these now accept the full set of orientation parameters instead of a single shared `axis`.

### 2. Rust API (WASM Engine)
Created `generic_advanced` and `revolute_advanced` bindings in `RawGenericJoint`. These bindings explicitly construct full 3D Isometries using `Isometry::from_parts` and apply `.local_frame1()`, `.local_frame2()`, `.local_axis1()`, and `.local_axis2()` down to Rapier's core joint builders. This guarantees absolute zero data loss when passing quaternions down to C++/Rust.

## How to Test and Use It

> [!IMPORTANT]
> Because we modified both TypeScript and Rust code, you **MUST** rebuild the WASM binaries and the compat wrappers.

**Bước 1: Build lại toàn bộ dự án**
Mở terminal WSL của bạn (hoặc bash nếu bạn chạy trên Windows) tại thư mục `D:\10.github\rapier.js`:
```bash
./builds/prepare_builds/prepare_all_projects.sh
cd rapier-compat
npm run build
```

**Bước 2: Cài đặt lại thư viện ở App của bạn**
Hãy chắc chắn chạy lại `yarn install` trong App của bạn nếu bạn đang dùng cách trỏ `file:` để App lấy file vừa được build mới nhất.

**Bước 3: Sử dụng API mới trong code JS/TS của bạn**
Trong app của bạn, thay vì gọi `JointData.generic()`, bây giờ bạn có thể gọi:

```typescript
let myJointData = RAPIER.JointData.genericAdvanced(
    { x: 0, y: 1, z: 0 },         // anchor1
    { x: 0, y: -1, z: 0 },        // anchor2
    { x: 1, y: 0, z: 0 },         // localAxis1
    { x: 0, y: 0, z: 1 },         // localAxis2
    { x: 0, y: 0, z: 0, w: 1 },   // frame1 (Quaternion của vật 1)
    { x: 0, y: 0.7, z: 0, w: 0.7 },// frame2 (Quaternion của vật 2)
    RAPIER.JointAxesMask.AngX     // axesMask
);

world.createImpulseJoint(myJointData, body1, body2, true);
```
