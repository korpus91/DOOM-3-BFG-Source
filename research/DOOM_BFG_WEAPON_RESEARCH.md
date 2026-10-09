# Doom 3 BFG weapon research for Shooter 1946

Status: **PARTIAL, first persistent checkpoint**. Task 1 only: source research and an original Godot design, with no implementation. Findings below are confirmed from the official BFG C++ source; retail shotgun numbers and script choreography remain unknown.

## Source verification and scope

| Item | Verified value |
|---|---|
| Official source | `id-Software/DOOM-3-BFG`, branch `master` |
| User fork / PR target | `korpus91/DOOM-3-BFG-Source`, base branch `master` |
| Exact source commit | `1caba1979589971b5ed44e315d9ead30b278d8b4` |
| Existing checkout | `/workspace/DOOM-3-BFG-Source`; initial local branch `work`, same source commit |
| Research branch | `research/doom-bfg-weapons-task1` |
| Required files | `neo/d3xp/Weapon.cpp` and `neo/d3xp/Weapon.h` are present and byte-for-byte equal to the official files at this commit |
| Weapon.cpp SHA256 | `a0b202d1d9890d3087d937ed88ea0c96efc4eb6dee372f1cbeec95598b6934ca` |
| Weapon.h SHA256 | `cdc74c55d866331e44c3858f2a8125837e529884b08cfdfa1b63cc0d2845a5e1` |

Native Git reads returned this exact SHA for both upstream and fork `refs/heads/master`. This is the 2012 BFG source, not the non-BFG Doom 3 repository or the original 1993 Doom. The bundled `doomclassic` directory does not change which engine is being researched. All implementation references below refer to **`neo/d3xp`**.

The source release expressly excludes retail game data: [README.txt, line 15](https://github.com/id-Software/DOOM-3-BFG/blob/1caba1979589971b5ed44e315d9ead30b278d8b4/README.txt#L15). No retail weapon `.script`, weapon `.def`, model animation or PK4 archive is present; the tracked `.def` is `neo/d3xp/Game.def`. Consequently this checkpoint can explain how the engine executes shotgun-like firing, but cannot certify the retail shotgun's pellet count, spread, damage, recoil strengths, pump/ejection frames or firing interval.

## 1. Input requests firing; a weapon script supplies the weapon's decisions

`idPlayer::FireWeapon`, `neo/d3xp/Player.cpp`, function line **3419**, checks weapon visibility/readiness and ammunition at **3434**, then calls `BeginAttack` at **3437**. It does not directly launch pellets in this function. `idWeapon::BeginAttack`, `neo/d3xp/Weapon.cpp`, function line **1792**, sets the linked script flag `WEAPON_ATTACK` at **1806**. `idWeapon::UpdateScript` begins at **2186**. The weapon's script uses these signals to decide firing/reloading/state transitions and invoke engine events. The retail shotgun script is missing, so its exact choices and event order must not be invented.

`idWeapon::GetWeaponDef`, `neo/d3xp/Weapon.cpp`, function line **952**, finds the named entity definition at **970** and reads ammunition fields at **972**, recoil fields at **985**, the first-person model at **1024**, sound keys at **1031**, and named effect joints at **1038**. Separate data and script objects give different weapons individual behavior on this shared engine layer. Further script/animation details are being traced for the next checkpoint.

## 2. Positioning, visual recoil and recovery

`idWeapon::PresentWeapon`, `neo/d3xp/Weapon.cpp`, function line **2347**, begins with the player's first-person view origin/axis at **2348**. The normal weapon branch calculates movement-based positioning through `idPlayer::CalculateViewWeaponPos` at **2374**, adds the GUI/NPC lowering offset at **2393**, then applies `MuzzleRise` at **2396**. It sets the weapon transform at **2400**, updates its script at **2405**, and animation at **2410**. Its separate flashlight branch is not the shotgun behavior.

`GetWeaponDef` reads `muzzle_kick_time` and `muzzle_kick_maxtime` as seconds and converts them to milliseconds at **985–986**. It also reads `muzzle_kick_angles` and `muzzle_kick_offset` at **987–988**. These are definition parameters, not fixed shotgun constants.

`idWeapon::Event_LaunchProjectiles`, `neo/d3xp/Weapon.cpp`, function line **3520**, accumulates visual kick duration at **3596**: start from the later of the previous end time and current real-client time; add the weapon's kick time; cap the result at current real-client time plus maximum kick time. Repeated shots can build the kick until that cap.

`idWeapon::MuzzleRise`, `neo/d3xp/Weapon.cpp`, function line **2091**, calculates remaining kick time at **2097**, rejects expired/disabled kick at **2098** and **2102**, and caps remaining time at **2106**. At **2110**, the remaining fraction is remaining milliseconds divided by maximum kick milliseconds. At **2111–2112**, that fraction scales the configured rotation and displacement. At **2114**, the transformed displacement is subtracted from the weapon origin; at **2115**, the recoil rotation is composed with the weapon axis. In plain English, the visible gun is displaced and rotated, then returns linearly as the remaining fraction falls to zero. A positive configured forward offset pulls it backward; actual directions depend on the missing definition values. The timer is extended by shots rather than integrating a physical spring.

There is a clock distinction worth keeping explicit: firing uses `gameLocal.realClientTime` at **3596**, while `MuzzleRise` reads `gameLocal.time` at **2097**. Their exact behavior under time scaling/prediction needs a separate timing trace; do not silently treat every BFG clock as interchangeable.

## 3. Camera recoil is a separate layer

`idPlayerView::WeaponFireFeedback`, `neo/d3xp/PlayerView.cpp`, function line **326**, reads `recoilTime` at **327**. It only starts the shared view kick if that value is nonzero and the prior kick has expired, at **329**. It reads `recoilAngles` with fallback `5 0 0` at **331**, stores them at **332**, and sets the end time from slow-game time plus `g_kickTime × recoilTime` at **333**. This is separate from the weapon model's `muzzle_kick_*` parameters. It also avoids replacing a damage kick already in progress; repeated fire does not keep accumulating this camera impulse through this function.

`idPlayerView::AngleOffset`, `neo/d3xp/PlayerView.cpp`, function line **373**, uses the remaining slow-game milliseconds at **377**. The angle at **379** is stored kick angles multiplied by the **square** of remaining milliseconds and `g_kickAmplitude`; each component is clamped to ±70 degrees at **381**. Thus camera recovery is quadratic in remaining time, rather than the gun model's linear envelope. Defaults are `g_kickTime=1` and `g_kickAmplitude=0.0001`: `neo/d3xp/gamesys/SysCvar.cpp`, declarations **153–154**. Neither default proves the retail shotgun's `recoilTime` or `recoilAngles`.

`idPlayer::GetViewPos`, `neo/d3xp/Player.cpp`, function line **8950**, adds this offset to player view angles and bob angles at **8962**. It affects the composed view axis, not a permanent assignment to the player's stored input `viewAngles` in this function. The visible gun follows that camera base before its additional model kick.

## 4. Shot direction and randomized pellet spread

`idWeapon::GetProjectileLaunchOriginAndAxis`, `neo/d3xp/Weapon.cpp`, function line **3498**, can select a muzzle/barrel origin if `launchFromBarrel` is set at **3502**; otherwise it uses the player view origin at **3508**. At **3512**, it **always replaces the outgoing axis with `playerViewAxis`**. Therefore cosmetic barrel rotation from model kick is not the final aiming direction in this launch helper. Camera view kick can affect the player-view basis used for subsequent firing.

`idWeapon::Event_LaunchProjectiles`, function **3520**, receives pellet/projectile count, spread, fuse offset, launch power and damage power as arguments. The count and spread are not fixed here. It validates the projectile definition at **3539**, checks/predicts ammunition at **3546**, consumes the clip amount at **3572**, and alerts nearby AI unless silent at **3577**.

The spread calculation is at `neo/d3xp/Weapon.cpp`, **3616–3621**. Convert supplied spread degrees to radians. For each pellet, draw two random values: one controls lateral radius as `sin(spreadRadians × randomValue)`, and the other controls a full-circle spin. Add that lateral displacement to the forward vector using the view's up/side axes, then normalize the direction. Each pellet receives its own random direction. This is not a fixed pellet pattern or a uniform-area cone distribution. For small spreads it weights shots toward the center. For an orthonormal basis the resulting angular deflection is `atan(lateralRadius)`, so the supplied number is not exactly the final cone half-angle. Exact retail spread and pellet count remain unavailable.

At **3659–3660**, the engine creates an `idProjectile` for each pellet; it checks a safe launch position at **3666** and **3672**, then calls `idProjectile::Launch` at **3696**. The `net_instanthit` flag is read at **3606** and affects networking at **3647**; its name alone does not establish that the retail shotgun is a standalone C++ raycast. Projectile collision/damage behavior is the next trace being completed.

After the loop, this C++ event schedules brass ejection at **3701–3702**, adds muzzle-flash light at **3708**, calls player feedback at **3711**, and resets muzzle-smoke start time at **3714**. Mechanical model animation and the retail script's ordering around this event are not present in the release.

## Firing sequence: confirmed engine path and missing script boundary

```mermaid
sequenceDiagram
    participant Input as Player input
    participant Player as idPlayer
    participant Script as Weapon script (retail data missing)
    participant Weapon as idWeapon
    participant Pellet as idProjectile
    participant Target as Hit target
    participant View as Player view / weapon presentation
    Input->>Player: Attack held/pressed
    Player->>Weapon: FireWeapon -> BeginAttack
    Weapon->>Script: WEAPON_ATTACK flag; UpdateScript
    Note over Script: Shotgun state logic, timing and animation order are unknown
    Script->>Weapon: launchProjectiles(count, spread, fuse, power, damagePower)
    Weapon->>Weapon: Validate ammo/definition, spend ammo, choose view-axis launch
    Weapon->>Weapon: Extend bounded model-kick timer
    loop Each projectile requested
        Weapon->>Weapon: Draw randomized spread direction
        Weapon->>Pellet: Create, check safe origin, Launch
        Note over Pellet,Target: Collision/damage trace is being completed
    end
    Weapon->>View: Muzzle flash, weapon fire feedback, smoke timestamp
    View->>View: Separate camera kick and model-kick recovery
```

The diagram represents the generic C++ event order. It does not assert which animation or sound occurs first in the missing shotgun script, nor that projectile impacts wait until every effect has been submitted.

## Proposed original Godot 4.7.2 design (not implemented)

Use a data-driven controller with independent responsibilities. All parameters below would be our own tuning, not claimed Doom values.

| Component | Responsibility |
|---|---|
| Input and fire controller | Interpret trigger presses/releases, semi/automatic policy, ammo, cooldown, reload/pump readiness; emit exactly one accepted-shot event per legal shot |
| Visual recoil | Apply a local transform above the weapon/hand rig; keep displacement/rotation and recovery independent of damage and input |
| Camera recoil | Apply a separate pitch/yaw feedback layer with explicit aim consequences; do not accidentally apply the visible gun transform to hit detection |
| Weapon/hand animation | Play fire, pump, reload and interruption transitions; mechanical parts use authored tracks and semantic animation events |
| Sound/muzzle effects | Subscribe to accepted-shot and mechanical-animation events; locate flashes/smoke at authored sockets; never spend ammo from an effect callback |
| Hit detection/damage | Resolve the canonical shot once, distribute independently sampled pellets, query hits or launch our own projectiles, and issue a typed hit/damage request |
| Recovery/tuning data | Store original recoil impulses, recovery curves, cooldowns, spread distribution, pellet count, damage, effects and animation references in a weapon Resource |

A fuller transform hierarchy, timing contract, recovery model and damage/reaction design will be added before the final Task 1 checkpoint. No Doom code/assets will be copied into Shooter 1946 and no Shooter files will be changed.

## Open questions

1. Actual BFG 2012 retail shotgun `.def` and `.script` parameters, including single-player/multiplayer differences.
2. Exact shotgun pump/fire/reload/shell-ejection animation clips and event timing.
3. Projectile collision, direct damage, damage scaling and target pain/death reactions: active source trace.
4. Camera/model kick clock behavior under time scaling and prediction.
5. Complete original Godot design and source-reference review: pending this task's final checkpoint.

Persistence: this file does not certify publication by itself. The handoff records the latest GitHub verification result. The GitHub API domain is currently blocked; a network addition for `api.github.com` has been saved for environment review. Git read access works. Only the user's fork may receive commits or a pull request.
