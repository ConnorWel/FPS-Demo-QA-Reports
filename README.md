# FPS-Demo-QA-Reports
QA testing documentation, bug tracking, and test cases for a custom 3D FPS project.

#  FPS Project: QA Testing & Bug Tracking Documentation

This repository contains the Quality Assurance documentation, bug tracking logs, and test cases for a custom-built 3D First-Person Shooter developed in Unity.

## 🐛 Comprehensive Bug Tracking Log (Active & Resolved)

| Bug ID | Title / Summary | Module / System | Severity | Status | Steps to Reproduce | Expected vs Actual Result |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **BUG-001** | [Animator] Rapid right-click for ADS causes transition lock and weapon clipping | Animation / Weapon | High | 🟢 Closed | 1. Rapidly click right-mouse button within 0.2s.<br>2. Observe animation state. | **Exp:** Smooth blend back to Hip-fire.<br>**Act:** Stucks in between, weapon clips. |
| **BUG-002** | [Object Pool] Bullets failing to return to pool on hitting geometry with custom shader | VFX / Combat | Medium | 🟢 Closed | 1. Fire weapon at walls with 'Dissolve' shader.<br>2. Check Hierarchy. | **Exp:** Deactivated and returned to pool.<br>**Act:** Leaks memory, instances grow. |
| **BUG-003** | [Controller] Jump physics conflicts with rotation interpolation on slope angles > 45° | Physics / Movement | High | 🟢 Closed | 1. Walk up a 45 degree slope.<br>2. Press Jump while turning 180 degrees. | **Exp:** Normal jump arc.<br>**Act:** Character snaps rotation and falls through mesh. |
| **BUG-004** | [UI] Ammo counter text string format throws deprecation warning | UI / System | Low | 🟢 Closed | 1. Launch game.<br>2. View Console logs during reloading. | **Exp:** Clean log output.<br>**Act:** Deprecation warning for API. |
| **BUG-005** | [Weapon] Magazine clips/floats during reload; backend logic executes but animation fails | Weapon / Animator | High | 🟢 Closed | 1. Fire weapon.<br>2. Press 'R'.<br>3. Observe magazine transforms. | **Exp:** Full reload animation plays.<br>**Act:** Logic executes, animation fails, mag floats. |
| **BUG-006** | [AI/NavMesh] Spider-bot gets stuck on stair edges and spams anti-stuck jump | AI / Navigation | Medium | 🔴 New | 1. Stand above stairs.<br>2. Lure Spider-bot.<br>3. Observe behavior at edge. | **Exp:** Smooth NavMesh traversal.<br>**Act:** Gets stuck, spams jump function indefinitely. |
| **BUG-007** | [State Machine] Rapidly entering/exiting aggro range causes AI Animator to jitter | AI / Animation | Medium | 🔴 New | 1. Stand on edge of AI detection radius.<br>2. Strafe left and right repeatedly. | **Exp:** Cooldown or smooth transition.<br>**Act:** AI twitches rapidly between Patrol/Chase. |
| **BUG-008** | [AI/Combat] Spider-bot fails to abort Attack State when target teleports away | AI / Combat | High | 🟡 In Progress | 1. Trigger AI attack.<br>2. Player immediately dashes behind wall. | **Exp:** Attack sequence aborts.<br>**Act:** AI finishes the entire combo hitting thin air. |
| **BUG-009** | [Weapon/Script] Recoil calculation throws NullReferenceException when switching weapons mid-fire | Weapon / Logic | High | 🔴 New | 1. Hold LMB to fire.<br>2. Scroll mouse wheel to switch weapons mid-burst. | **Exp:** Fire stops, weapon switches cleanly.<br>**Act:** Script error in Recoil.cs, camera locks up. |
| **BUG-010** | [Performance] Severe FPS drop when >20 instantiated bullet casings trigger physics collisions | Performance | Medium | 🟡 In Progress | 1. Spawn 5 enemies.<br>2. Fire LMG continuously in a tight corner.<br>3. Monitor Profiler. | **Exp:** Stable framerate, old casings sleep.<br>**Act:** Physics step spikes, framerate drops to 40. |

---
*Note: Detailed reports and test cases for specific systems (e.g., AI Pathfinding, Weapon Mechanics) can be found in the attached PDF portfolios.*

# BUG-015: [Weapon/Animator] Magazine clips/floats during reload; backend logic executes but animation fails to play

**Reporter:** [Your Name]
**Date:** 2026-06-05
**Environment/Version:** Unity 2023.x / Windows 11
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
