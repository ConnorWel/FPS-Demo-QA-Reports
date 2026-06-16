# FPS-Demo-QA-Reports
QA testing documentation, bug tracking, and test cases for a custom 3D FPS project.

#  FPS Project: QA Testing & Bug Tracking Documentation

This repository contains the Quality Assurance documentation, bug tracking logs, and test cases for a custom-built 3D First-Person Shooter developed in Unity.

## 🐛 Comprehensive Bug Tracking Log (Active & Resolved)

| Bug ID | Title / Summary | Module / System | Severity | Status | Steps to Reproduce | Expected vs Actual Result |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **BUG-001** | [Animato r] Rapid right-click for ADS causes transition lock and weapon clipping | Animation / Weapon | High | 🟢 Closed | 1. Rapidly click right-mouse button within 0.2s.<br>2. Observe animation state. | **Exp:** Smooth blend back to Hip-fire.<br>**Act:** Stucks in between, weapon clips. |
| **BUG-002** | [Object Pool] Bullets failing to return to pool on hitting geometry with custom shader | VFX / Combat | Medium | 🟢 Closed | 1. Fire weapon at walls with 'Dissolve' shader.<br>2. Check Hierarchy. | **Exp:** Deactivated and returned to pool.<br>**Act:** Leaks memory, instances grow. |
| **BUG-003** | [Controller] Jump physics conflicts with rotation interpolation on slope angles > 45° | Physics / Movement | High | 🟢 Closed | 1. Walk up a 45 degree slope.<br>2. Press Jump while turning 180 degrees. | **Exp:** Normal jump arc.<br>**Act:** Character snaps rotation and falls through mesh. |
| **BUG-004** | [UI] Ammo counter text string format throws deprecation warning | UI / System | Low | 🟢 Closed | 1. Launch game.<br>2. View Console logs during reloading. | **Exp:** Clean log output.<br>**Act:** Deprecation warning for API. |
| **BUG-005** | [Weapon] Magazine clips/floats during reload; backend logic executes but animation fails | Weapon / Animator | High | 🟢 Closed | 1. Fire weapon.<br>2. Press 'R'.<br>3. Observe magazine transforms. | **Exp:** Full reload animation plays.<br>**Act:** Logic executes, animation fails, mag floats. |
| **BUG-006** | [AI/NavMesh] Spider-bot gets stuck on stair edges and spams anti-stuck jump | AI / Navigation | Medium |  🟢 Closed | 1. Stand above stairs.<br>2. Lure Spider-bot.<br>3. Observe behavior at edge. | **Exp:** Smooth NavMesh traversal.<br>**Act:** Gets stuck, spams jump function indefinitely. |
| **BUG-007** | [State Machine] Rapidly entering/exiting aggro range causes AI Animator to jitter | AI / Animation | Medium | 🔴 New | 1. Stand on edge of AI detection radius.<br>2. Strafe left and right repeatedly. | **Exp:** Cooldown or smooth transition.<br>**Act:** AI twitches rapidly between Patrol/Chase. |
| **BUG-008** | [AI/Combat] Spider-bot fails to abort Attack State when target teleports away | AI / Combat | High | 🟡 In Progress | 1. Trigger AI attack.<br>2. Player immediately dashes behind wall. | **Exp:** Attack sequence aborts.<br>**Act:** AI finishes the entire combo hitting thin air. |
| **BUG-009** | [Weapon/Script] Recoil calculation throws NullReferenceException when switching weapons mid-fire | Weapon / Logic | High |  🟢 Closed  | 1. Hold LMB to fire.<br>2. Scroll mouse wheel to switch weapons mid-burst. | **Exp:** Fire stops, weapon switches cleanly.<br>**Act:** Script error in Recoil.cs, camera locks up. |
| **BUG-010** | [Performance] Severe FPS drop when >20 instantiated bullet casings trigger physics collisions | Performance | Medium | 🟡 In Progress | 1. Spawn 5 enemies.<br>2. Fire LMG continuously in a tight corner.<br>3. Monitor Profiler. | **Exp:** Stable framerate, old casings sleep.<br>**Act:** Physics step spikes, framerate drops to 40. |
| **BUG-011** | [Weapon/Transform] Pistol mesh detaches and floats outside character's hand during reload animation | Weapon / Visual | High | 🟢 Closed | 1. Equip Handgun.<br>2. Fire 1 round.<br>3. Press 'R' to reload. | **Exp:** Pistol remains firmly in hand during reload.<br>**Act:** Pistol floats mid-air in front of the hand, clipping severely. |
| **BUG-012** | [Performance/Pool] Severe FPS drop and mid-air floating casings during sustained automatic fire | System / Performance | High | 🟢 Closed | 1. Equip AR.<br>2. Hold LMB to fire 30+ rounds.<br>3. Observe casings and Profiler. | **Exp:** Casings fall, disappear, FPS remains stable.<br>**Act:** Casings freeze mid-air, meshes accumulate, causing severe FPS drop. |
| **BUG-013** | [AI/Animator] Spider-bot fails to execute 'Unstuck Jump'; throws Animator parameter error | AI / Animator | Medium | 🟢 Closed | 1. Lure Spider-bot to a complex terrain edge until it gets stuck.<br>2. Check Console logs. | **Exp:** AI triggers Jump animation to cross the gap.<br>**Act:** Console throws `Parameter 'Jump' does not exist`. AI remains stuck. |
| **BUG-014** | [Weapon/Physics] Instantiated bullets freeze instantly at muzzle and fail to despawn | Weapon / Logic | High | 🟢 Closed | 1. Fire weapon.<br>2. Observe bullet trajectory and hierarchy. | **Exp:** Bullet travels at speed 100, despawns after 5s.<br>**Act:** Bullets hang stationary in mid-air permanently. |


---
# BUG-05: [Weapon/Animator] Magazine clips/floats during reload; backend logic executes but animation fails to play

**Reporter:** [Connor Wei]
**Date:** 2026-06-05
**Environment/Version:** Unity 2026 / Windows 11
**Module:** Weapon System / Animator

## 1. Severity & Priority
- **Severity:** High - Severely impacts core combat visual feedback, causing a disconnect between game logic and visual representation.
- **Priority:** P2

## 2. Pre-conditions
- Player is in Play Mode.
- Player has the primary weapon equipped.
- Current magazine is not full, and reserve ammo is greater than 0.

## 3. Steps to Reproduce
1. Left-click to fire the weapon and consume at least one round of ammunition.
2. Press the `R` key to initiate the reload sequence.
3. Observe the weapon model, magazine position, and Console log outputs.

## 4. Actual Result
Upon pressing the reload key, the backend C# logic executes successfully (e.g., ammo count updates in the UI/Console). However, the character's Reload Animation fails to trigger. As a result, the magazine model is left floating or clipping incorrectly on the weapon mesh, without playing the detach and attach sequence.

## 5. Expected Result
The Animator should immediately respond to the reload input alongside the C# logic, playing the complete reload animation. The magazine's Transform should properly follow the animation keyframes to visually complete the detach and insert sequence.

## 6. Investigation Notes
*Initial QA Triage:*
- **Animator State Machine:** The C# script might be successfully calling `animator.SetTrigger("Reload")`, but the transition in the Animator Controller (e.g., from `Any State` or `Idle` to `Reload`) is not properly configured or the transition conditions are not met.
- **Layer Weights:** If the reload animation is on a separate Animation Layer, its Weight might be unintentionally set to 0.
- **Missing References:** The AnimationClip itself might have lost the binding path/reference to the magazine model's Transform.

- # BUG-006: [AI/NavMesh] Spider-bot gets stuck on stair edges and repeatedly triggers anti-stuck jump mechanism

**Reporter:** [Connor Wei]
**Date:** 2026-06-06
**Environment/Version:** Unity 2026 / Windows 11
**Module:** AI / Navigation System

## 1. Severity & Priority
- **Severity:** Medium - The game does not crash, but the AI combat loop is broken, rendering the enemy completely ineffective in specific terrain.
- **Priority:** P2

## 2. Pre-conditions
- Player is in Play Mode on `Test_Scene_01`.
- The Spider-bot enemy is spawned and active.
- Player is positioned on a raised platform or at the top of a staircase.
- NavMesh is baked, but the stair step height/slope is discontinuous.

## 3. Steps to Reproduce
1. Fire a weapon to alert the Spider-bot and trigger its `Chase` state.
2. Stand at the top of the stairs and wait for the Spider-bot to navigate towards the player.
3. Observe the Spider-bot's behavior as it reaches the bottom edge of the staircase.

## 4. Actual Result
The Spider-bot reaches the edge of the stairs and stops moving forward. Because its forward velocity drops to near zero but the destination is not reached, the custom `EnemyControl.cs` anti-stuck logic is triggered. The bot executes a vertical jump, but lands back in the exact same invalid NavMesh position, causing it to spam the jump animation infinitely while trapped at the stair base.

## 5. Expected Result
The Spider-bot should smoothly traverse the stairs using the NavMesh. If a jump is intentionally required to clear an obstacle, the anti-stuck jump logic should apply a forward physical impulse to clear the gap, or properly utilize an Off-Mesh Link, allowing the bot to resume its pathfinding after landing.

## 6. Investigation Notes & Stack Trace
*QA Investigation:*
- **NavMesh Bake Settings:** Checked the Navigation window. The `Max Slope` and `Step Height` parameters might be set too low for the specific stair geometry, causing the NavMesh surface to break at the steps.
- **Code Logic:** Reviewed `EnemyControl.cs`. The anti-stuck mechanism only adds `linearVelocity` on the Y-axis. 
- **Recommendation:** Re-bake the NavMesh with adjusted Step Height, or add an Off-Mesh Link for the stairs. Additionally, add a short cooldown (e.g., 2 seconds) to the jump trigger in the script to prevent animation spamming.

# BUG-009: [Weapon/Script] Recoil calculation throws NullReferenceException when switching weapons mid-fire

**Reporter:** [Connor Wei]
**Date:** 2026-06-06
**Environment/Version:** Unity 2026 / Windows 11
**Module:** Weapon System / Core Logic

## 1. Severity & Priority
- **Severity:** High - Causes a script execution halt. The camera recoil logic freezes permanently, and subsequent weapon firing mechanics break until the scene is reloaded.
- **Priority:** P1

## 2. Pre-conditions
- Player is in Play Mode.
- Player has at least two weapons equipped (e.g., Assault Rifle and Pistol).
- Assault Rifle is currently active in hands.

## 3. Steps to Reproduce
1. Hold down the Left Mouse Button (LMB) to fire the Assault Rifle continuously.
2. While still holding LMB (mid-burst), scroll the mouse wheel to instantly switch to the secondary weapon.
3. Release LMB and attempt to fire the new weapon.
4. Open the Unity Console.

## 4. Actual Result
The game stutters for a frame. The recoil camera shake locks up at an offset position. Attempting to fire the newly equipped weapon fails. The Unity Console outputs a continuous stream of `NullReferenceException` errors.

## 5. Expected Result
Switching weapons should cleanly interrupt the firing state of the current weapon. The recoil calculation Coroutine or Update loop should terminate gracefully before the active weapon object is disabled or destroyed, allowing the new weapon to function normally.

## 6. Investigation Notes & Stack Trace
*QA Investigation & Console Output:*
```text
NullReferenceException: Object reference not set to an instance of an object
at RecoilSystem.ApplyRecoil() in Assets/Scripts/Weapons/RecoilSystem.cs:line 45
at WeaponController.Update() in Assets/Scripts/Weapons/WeaponController.cs:line 112




```

# **BUG-011: [Weapon/Transform] Pistol mesh detaches and floats outside character's hand during reload animation**

**Reporter:** [Connor Wei]
**Date:** 2026-06-15
**Environment/Version:** Unity 2026 / Windows 11
**Module:** Weapon System / Animation & Transform

## 1. Severity & Priority
- **Severity:** High - Critically breaks visual immersion and core combat feedback.
- **Priority:** P2

## 2. Pre-conditions
- Player is in Play Mode.
- Handgun weapon is equipped.
- Handgun GameObject is a child of the `WeaponHolder` bone.

## 3. Steps to Reproduce
1. Enter aiming state (Hold RMB).
2. Fire the weapon to consume ammo.
3. Press `R` to execute the reload animation sequence.
4. Observe the weapon's spatial relationship with the character's right hand.

## 4. Actual Result
When the reload animation triggers, the pistol model separates from the character's hand, floating in mid-air in front of the player. The character's hand animation plays normally (grabbing the magazine), but the weapon is entirely detached from the grip position.

## 5. Expected Result
The pistol should remain firmly attached to the character's hand bone (relative position `(0,0,0)`) throughout the entire reload animation sequence without any clipping or floating.

## 6. Investigation Notes
*Root Cause Analysis:*
- **Transform Offset Conflict:** The `Transform.LocalPosition` of the Handgun was manually altered in the Inspector to align the iron sights with the center of the screen. 
- Because the Animator controls the arm/hand bones expecting the weapon to be at a `(0,0,0)` local offset, the manual coordinate shift created a permanent displacement. When the animation played, it applied this displacement, causing the weapon to float.
- **Resolution:** Reset the Handgun's Local Transform to `(0,0,0)`. Shifted the aiming logic to move an external `AimOffsetContainer` (parent of the camera/arms) instead of directly moving the weapon mesh.

---

# **BUG-012: [Performance/Pool] Severe FPS drop and mid-air floating casings during sustained automatic fire**

**Reporter:** [Connor Wei]
**Date:** 2026-06-15
**Environment/Version:** Unity 2026 / Windows 11
**Module:** Object Pooling / Physics

## 1. Severity & Priority
- **Severity:** High - Massive memory/rendering leak leading to severe game stuttering and eventual crashing if unchecked.
- **Priority:** P1

## 2. Pre-conditions
- Player has an automatic weapon (AR) equipped.
- `ObjectPoolManager` is active for bullet casings.

## 3. Steps to Reproduce
1. Hold LMB to fire the AR continuously.
2. Observe the ejected casings.
3. Monitor the Unity Profiler and active GameObjects in the Hierarchy.

## 4. Actual Result
Ejected casings fail to reach the ground. After approximately 1 second, they freeze perfectly still in mid-air. Furthermore, the meshes are never destroyed or disabled. Firing 100 rounds results in 100 active casing meshes rendering on-screen, dropping the frame rate drastically.

## 5. Expected Result
Casings should complete their physics trajectory (hit the ground), remain for a designated lifetime (e.g., 3 seconds), and then cleanly deactivate (return to the Object Pool) to maintain a stable framerate.

## 6. Investigation Notes
*Root Cause Analysis:*
- **Object Pool Lifecycle Mismatch:** The `CasingSleep.cs` script used the `Start()` method for the despawn timer. `Start()` only executes once per GameObject lifetime. Re-used casings from the Object Pool never triggered the timer again.
- **Improper Despawn Logic:** The script attempted to "sleep" the casing using `Rigidbody.isKinematic = true` instead of `gameObject.SetActive(false)`. This halted physics calculations instantly (causing the mid-air freeze) but left the geometry active, consuming GPU rendering resources.
- **Resolution:** Migrated initialization and timer logic to `OnEnable()`. Replaced kinematic sleep with `SetActive(false)` for proper Object Pool recycling. Added `OnDisable()` with `CancelInvoke()` to prevent race conditions.

---

# **BUG-013: [AI/Animator] Spider-bot fails to execute 'Unstuck Jump'; throws Animator parameter error**

**Reporter:** [Connor Wei]
**Date:** 2026-06-15
**Environment/Version:** Unity 2026 / Windows 11
**Module:** AI / Animator

## 1. Severity & Priority
- **Severity:** Medium - AI pathfinding gets permanently locked, degrading PvE combat experience.
- **Priority:** P3

## 2. Pre-conditions
- Spider-bot enemy is spawned and actively chasing the player.
- Terrain contains complex geometry or stair edges.

## 3. Steps to Reproduce
1. Lure the Spider-bot to a terrain edge or obstacle until its NavMeshAgent gets stuck (velocity drops while remaining distance is high).
2. Wait for the `stuckTimer` to exceed the `stuckTimeThreshold` (1.5s).
3. Check the Unity Console logs.

## 4. Actual Result
The C# script successfully detects the stuck state and attempts to fire the jump logic, but the AI remains stuck. The Unity Console outputs the warning: `Parameter 'Jump' does not exist`. No jump animation is played.

## 5. Expected Result
The script triggers the `Jump` parameter in the Animator, playing the assigned `wallJump` animation, and the NavMeshAgent safely warps across the obstacle to resume chasing.

## 6. Investigation Notes
*Root Cause Analysis:*
- **Animator Parameter Missing/Misnamed:** The `EnermyControl.cs` script called `anim.SetTrigger("Jump")`, but the Animator Controller's Parameters list did not contain a Trigger named exactly "Jump" (case-sensitive).
- **Missing State Transition:** Even after creating the parameter, there was no Transition line connecting the `Any State` node to the jump animation node in the Animator grid.
- **Resolution:** Created a Trigger parameter named `Jump`. Created a transition from `Any State` to the `wallJump` state with the condition `Jump`.

---

# **BUG-014: [Weapon/Physics] Instantiated bullets freeze instantly at muzzle and fail to despawn**

**Reporter:** [Connor Wei]
**Date:** 2026-06-15
**Environment/Version:** Unity 2026 / Windows 11
**Module:** Weapon Backend / Physics

## 1. Severity & Priority
- **Severity:** High - Projectile logic completely broken; enemies cannot be damaged via physical projectiles.
- **Priority:** P1

## 2. Pre-conditions
- Weapon equipped.
- `BulletControl.cs` is attached to the bullet prefab in the Object Pool.

## 3. Steps to Reproduce
1. Left-click to fire the weapon.
2. Observe the muzzle point and bullet trajectory.

## 4. Actual Result
Bullets spawn at the `FirePoint` but do not travel forward. They remain permanently suspended in mid-air at the muzzle location. The 5-second `Invoke` despawn timer fails to trigger, and bullets pile up on top of each other.

## 5. Expected Result
Bullets should launch forward with the assigned `Impulse` force, collide with objects (applying damage/forces), or cleanly despawn after 5 seconds if no collision occurs.

## 6. Investigation Notes
*Root Cause Analysis:*
- **Missing Component / NullReferenceException:** The Bullet Prefab was missing a `Rigidbody` component. In `BulletControl.cs`, `OnEnable()` calls `rb = GetComponent<Rigidbody>();` and immediately attempts to access `rb.linearVelocity`. Because `rb` was null, a `NullReferenceException` occurred.
- **Silent Failure:** This exception halted the execution of the entire `OnEnable()` block instantly. Consequently, the `AddForce` command and the `Invoke` timer were never reached, leaving the mesh stranded permanently.
- **Object Pool Sleep Trap:** Furthermore, pooled objects were not receiving a fresh lifecycle state upon retrieval.
- **Resolution:** Added `Rigidbody` to the prefab (Gravity enabled, Kinematic disabled). Added a forced `bullet.SetActive(false); bullet.SetActive(true);` cycle in the Weapon script's firing logic to ensure `OnEnable()` is consistently triggered upon instantiation.
