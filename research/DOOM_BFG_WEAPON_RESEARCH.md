# Doom 3 BFG weapon research for Shooter 1946

Status: **PARTIAL, recovered research checkpoint**. All three completed specialist notes survived the interruption and are preserved below; final consolidation and Godot design are the remaining writing steps. Task 1 only: source research and an original Godot design, with no implementation. Findings below are confirmed from the official BFG C++ source; retail shotgun numbers and script choreography remain unknown.

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


## Recovered source notes: Recoil and first-person positioning

## Confirmed BFG weapon positioning and recoil findings

Source inspected: `/workspace/DOOM-3-BFG-Source`, official upstream `id-Software/DOOM-3-BFG`, commit `1caba1979589971b5ed44e315d9ead30b278d8b4` (provenance verified by root agent). These are C++ engine facts; no retail shotgun definitions, scripts, meshes or animation clips were available, so none of the following establishes the retail shotgun's exact recoil strength or timing.

### Two separate recoil systems

- `neo/d3xp/Weapon.h`, `idWeapon` member declarations, **lines 299–304**: weapon model recoil stores `kick_endtime`, `muzzle_kick_time`, `muzzle_kick_maxtime`, `muzzle_kick_angles`, and `muzzle_kick_offset`.
- `neo/d3xp/Weapon.cpp`, `idWeapon::GetWeaponDef`, **function line 952; lines 985–988**: reads these four `muzzle_kick_*` keys from the selected entity definition. The two times are definition seconds converted with `SEC2MS`; angles are pitch/yaw/roll and offset is a three-component position vector. `neo/idlib/math/Math.h:58` defines the seconds-to-ms conversion. `neo/idlib/math/Angles.h:41–43,53–55` identifies pitch, yaw and roll; `Angles.cpp`, `idAngles::ToForward`, **117,120–123**, converts angles with `DEG2RAD`, confirming degrees, and positive pitch points forward downward in the ordinary gravity frame.
- `neo/d3xp/Weapon.cpp`, `idWeapon::Event_LaunchProjectiles`, **function 3520; lines 3595–3602**: a shot sets `kick_endtime` at least to `gameLocal.realClientTime`, then adds `muzzle_kick_time`, capped at current real-client time plus `muzzle_kick_maxtime`. Rapid shots can build up the *visual model's* remaining kick until the cap. This is not an angular-velocity spring or random recoil pattern.
- `neo/d3xp/Weapon.cpp`, `idWeapon::MuzzleRise`, **function 2091; lines 2097–2115**: `remaining = kick_endtime - gameLocal.time`; no effect when remaining ≤ 0 or max-time ≤ 0. Clamp remaining to max-time; `amount = remaining / max-time`; multiply both configured angles and offset by amount; `origin -= axis * offset`; `axis = angles.ToMat3() * axis`. This gives linear return toward the un-kicked model transform as time expires. Subtraction makes positive local offsets move in the opposite direction of that rotated vector. This function alters the weapon transform, not the player's input aim or `viewAngles`.
- **Clock caveat:** ordinary launch accumulation above uses `realClientTime`, while `MuzzleRise` subtracts `gameLocal.time`. `Event_LaunchProjectilesEllipse`, **3722,3787–3794**, instead accumulates with `gameLocal.time`. These names do not mean OS wall-clock time: `neo/d3xp/Game_local.cpp`, `idGameLocal::RunFrame` time update **2299–2309**, sets fast and slow realClientTime equal to the corresponding simulation time; `SelectTimeGroup`, **4634–4641**, selects their values. Network prediction (`neo/d3xp/Game_network.cpp`, `idGameLocal::ClientRunFrame`, **1031,1056–1065**) advances realClientTime only when prediction time exceeds it. Preserve this branch/clock distinction; prediction replay details were not fully audited. A Godot controller should deliberately use a single chosen clock for each recovery system.

### Camera kick is separate, shared with damage feedback, and recovers quadratically

- `neo/d3xp/Weapon.cpp`, `idWeapon::Event_LaunchProjectiles`, **3711** calls `owner->WeaponFireFeedback` after launching, flash handling, and brass scheduling.
- `neo/d3xp/Player.cpp`, `idPlayer::WeaponFireFeedback`, **function 3376; lines 3377–3395**: resets blinking, sets `AI_WEAPON_FIRED`, calls `playerView.WeaponFireFeedback`, then reads four controller shake magnitude/duration keys and applies rumble only to a locally controlled player.
- `neo/d3xp/PlayerView.cpp`, `idPlayerView::WeaponFireFeedback`, **function 326; lines 327–335**: reads integer `recoilTime` directly (milliseconds because it is added to a millisecond clock), and `recoilAngles` (default `5 0 0`). Only accepts a new firing kick if `recoilTime != 0` and `kickFinishTime < gameLocal.slow.time`; it does not stack or reset a camera kick while one is still active. Sets `kickAngles` and finish time to `slow.time + g_kickTime * recoilTime`.
- `neo/d3xp/PlayerView.cpp`, `idPlayerView::AngleOffset`, **function 373; lines 376–389**: while active, compute `remaining_ms = kickFinishTime - slow.time`; angle offset = `kickAngles * remaining_ms * remaining_ms * g_kickAmplitude`; clamp each axis to ±70 degrees; return zero when expired. This is quadratic recovery in remaining time, **not** normalized interpolation of `recoilAngles`. Changing recoil duration also changes initial magnitude quadratically (before clamp), not just recovery duration. Engine defaults: `neo/d3xp/gamesys/SysCvar.cpp:153–154` gives `g_kickTime=1`, `g_kickAmplitude=0.0001`. Illustrative, not retail: with an accepted 100 ms kick and default amplitudes, the initial scale is 1; halfway through it is 0.25.
- `neo/d3xp/PlayerView.cpp`, `idPlayerView::DamageImpulse`, **function 241; lines 265–284**: writes the same `kickFinishTime` and `kickAngles` for incoming damage, explaining why a shot does not replace an active damage kick. The acceptance condition suppresses *any* current shared kick, despite the source comment mentioning damage.
- `neo/d3xp/Player.cpp`, `idPlayer::GetViewPos`, **function 8950; lines 8961–8971**: camera origin is eye position plus view bob; camera angles = `viewAngles + viewBobAngles + playerView.AngleOffset()`; multiply by gravity axis. Nodal-pivot offsets then adjust camera origin based on this axis (`g_viewNodalX/Z`), so this pipeline can also affect position when those settings are nonzero; engine defaults are X=3 and Z=6 at `SysCvar.cpp:280–281`. The kick is added during view construction rather than permanently editing input aim.
- `neo/d3xp/Player.cpp`, `idPlayer::CalculateFirstPersonView`, **function 8980; lines 8981–9002**: ordinary path calls `GetViewPos` (8997). A configured player-model camera path obtains animated `camera` joint (8991–8994) and also adds view bob and `AngleOffset` (8988). A sound-shake multiply at 8998–9001 is under `#if 0`; do not claim it runs.
- `neo/d3xp/Player.cpp`, `idPlayer::CalculateRenderView`, **function 9023; lines 9066–9072**: normal first-person rendering copies `firstPersonViewOrigin/Axis` into the rendered camera.

### First-person pose combines movement with the separate model kick

- `neo/d3xp/Player.cpp`, `idPlayer::CalculateViewWeaponPos`, **function 8793; lines 8799–8808**: uses already-calculated first-person camera origin/axis; adds gun-position tuning values and acceleration lag transformed by that camera axis. Global default gun position is `(3,0,0)` and model scale 1 (`neo/d3xp/gamesys/SysCvar.cpp:276–279`). These are engine defaults, not proof of final retail positioning.
- Same function, **8810–8829**: roll = signed horizontal speed × bob sine × 0.005; yaw uses 0.01; pitch = horizontal speed × bob sine × 0.005. Alternate footstep half-cycles invert roll/yaw. Adds turning-history lag; multiplayer additionally scales it with `g_mpWeaponAngleScale` (engine default 0 at SysCvar.cpp:284).
- Same function, **8831–8854**: adds landing drop (quarter of landChange), with 150 ms deflection and 300 ms return (`neo/d3xp/Player.h:57–58`), speed-sensitive sinusoidal idle angular drift `(xyspeed+40) * sin(time_seconds) * 0.01` to each angle, `independentWeaponPitchAngle`, then model axis = computed angular matrix × `g_gunScale` × camera axis.
- `neo/d3xp/Player.cpp`, `idPlayer::GunTurningOffset`, **function 8701; lines 8706–8745**: no lag for first 64 frames; average recent logged input view angles, correct yaw wraparound, subtract current view angle, multiply per-weapon scale, clamp every component to configured maximum. Stored view history has 64 slots (`Player.h:755–756`); input history is populated at `Player.cpp:6125`. `GetWeaponDef:1178–1180` defaults to 10 averaged frames, scale 0.25, max 10 degrees. These are fallback values, not established shotgun overrides.
- `neo/d3xp/Player.cpp`, `idPlayer::GunAcceleratingOffset`, **function 8756; lines 8763–8783**: traverses recent logged movement-change records younger than weaponOffsetTime; for each use `weight = (cos(2π * age/time)-1)/2`; add `weight * weaponOffsetScale * record.dir`. This weight starts at zero, becomes −1 at half the interval, and returns to zero at its end. Definition defaults are 400 ms and 0.005 (`GetWeaponDef:1182–1183`); history has 16 slots (`Player.h:757–758`). These are not necessarily measured physical acceleration: `idPlayer::Think:7557–7571` logs differences in forward/right input commands, and the movement path at **6976–6982** records a fixed vertical value of 200 when jumping.
- `neo/d3xp/Weapon.cpp`, `idWeapon::PresentWeapon`, **function 2347; lines 2348–2410**: cache owner first-person view first; normal weapons call `CalculateViewWeaponPos` (2374), add smoothly lowered GUI/NPC hide offset (2378–2393), then apply `MuzzleRise` (2396), set physics transform (2400–2401), update visuals, weapon script (2405), GUI (2407), and animation (2410). Thus skeletal animation is combined with the procedural root transform; model recoil is not the full fire/pump animation.
- Same function, **2412–2423**: only show the view model in its player's view; sets an optional weapon depth hack to avoid wall poking; separate world model/shadow handling follows at 2425–2433.
- `neo/d3xp/Weapon.cpp`, `idWeapon::GetGlobalJointTransform`, **function 1592; lines 1593–1610**: get animated joint pose at current game time; transform model-space offset/orientation by `viewWeaponOrigin/Axis`; separate world-model path exists. This lets barrel, flash, eject, particle and light joints follow skeletal animation plus procedural positioning/recoil.
- `neo/d3xp/Weapon.cpp`, `idWeapon::GetWeaponDef`, **1023–1028,1037–1049**: view model comes from `model_view`; world model initialized separately; named attachment joints are `barrel`, `flash`, `eject`, `guiLight`, `ventLight`, and optional `smoke_joint`. Actual hand placement, pump travel, fire/pump clip curves, and frame timings live in missing model/animation and script data, not these generic C++ equations.

### Shotgun-specific orientation workaround and aim separation

- `neo/d3xp/Weapon.cpp`, `idWeapon::GetMuzzlePositionWithHacks`, **function 2276; lines 2296–2301,2316–2322**: obtain barrel position; identify the ordinary shotgun by `pdaIcon == guis/assets/hud/icons/shotgun_new.tga`; take orientation from animated `trigger` joint, swap basis axes 0 and 2, negate axis 0. **It uses the trigger joint's orientation, not its position** (joint position is explicitly discarded). This is a concrete shotgun-specific source path; comment says barrel-joint axis is unsuitable.
- Same function, **2326–2335**: double shotgun is identified by `weapon_shotgun_double` or `_mp` entity-definition names and swaps axes 0/2; grabber shifts muzzle origin 4 source units forward. These prove selected per-weapon exceptions coexist with definition/script-driven behavior.
- `neo/d3xp/Weapon.cpp`, `idWeapon::GetProjectileLaunchOriginAndAxis`, **function 3498; lines 3502–3512**: if barrel joint exists and projectile definition enables `launchFromBarrel`, obtain corrected muzzle position; otherwise use player view origin/axis. **Always overwrite returned firing axis with `playerViewAxis` at 3512**, explicitly to fix initial plasma-rifle burst shot. Ordinary `launchProjectiles` therefore aims from the cached camera axis even if visible gun/bones tilt, while origin can come from the animated/recoiling barrel. Camera kick affects that cached view axis on a subsequent view update; visual MuzzleRise does not directly supply the direction in this ordinary launch path, but can affect barrel-based origin. Ellipse event has a separate muzzle path at 3778–3785 and should not be silently equated.
- `neo/d3xp/Player.cpp`, `idPlayer::UpdateLaserSight`, **function 7467; lines 7476–7480,7497–7499**: the stereo-rendered laser sight calls the muzzle workaround and uses the corrected muzzle axis to draw a beam (origin minus 2 units forward; end plus configured length). This is one actual user of the corrected shotgun orientation, distinct from ordinary projectile direction's camera override.
- `neo/d3xp/Player.cpp`, `idPlayer::Think` path, **7653–7665**: calculates first-person view, then render view, then updates weapon. Client prediction path likewise **9731–9737**. Combined with `PresentWeapon:2396,2405`, this means newly generated shot feedback/kick follows the current pose construction; do not promise same-frame camera/model recoil onset from the engine code alone.

### Effects following pose

- `GetWeaponDef:1078–1085` configures muzzle-light shader, point/projected type, color, radius and flashTime (default 0.25 seconds converted to ms); these defaults do not prove shotgun flash tuning.
- `PresentWeapon:2440–2456` emits optional muzzle smoke at explicit smoke joint, else barrel, else camera, using current animated/global pose. `PresentWeapon:2518–2534` removes flash at expiry/hidden weapon and updates active flash position every presentation, so it follows the animated/recoiling gun. Definition-specific particles/lights are also transformed from joints at 2471–2507.

### Implications for original Godot design (recommendations, not Doom facts)

- Keep aim/input, temporary camera recoil, and temporary weapon-root recoil distinct. A shot should record one authoritative aim transform and shot ID before feedback is added; decide explicitly whether temporary camera recoil influences subsequent shot aim.
- Use a weapon recoil Node3D above the animated weapon/hands and keep separate movement-sway and recoil offsets so AnimationPlayer/AnimationTree and procedural code do not write the same transform.
- Choose time-based normalized curves or a spring with explicit recoil amplitude/recovery seconds. Do not accidentally couple amplitude to recovery-duration squared as BFG camera feedback does.
- Treat pump, bolt, slide, hand movement, recoil, flash, shell ejection and firing sound as separately tunable effects synchronized to a single accepted shot/state sequence. An animated muzzle socket positions VFX; shot direction follows the chosen aim policy.
- Expose optional camera recoil strength (including zero), independent visual recoil and hand animation intensities. Frame-rate-independent recovery and per-weapon Resource tuning are appropriate for Godot 4.7.2.

### Remaining limitations

Actual 2012 retail shotgun muzzle/camera recoil values, model origin, pellet script parameters, animation names, pump sequence/timing and exactly which launch event its script uses cannot be established from the supplied source-only checkout. The `independentWeaponPitchAngle` HMD-oriented variable is initialized to zero (`Player.cpp:1278`), read at 8849, and declared `Player.h:279`; repository-wide search found no assignment to an active head-tracking value, so do not infer a complete VR behavior from its comment. Source clock differences were recorded, not resolved by executing or compiling the game.


## Recovered source notes: Firing, script and animation

## BFG firing, scripts, animation, sound and muzzle presentation findings

Research source: `/workspace/DOOM-3-BFG-Source`; verified by parent as official `id-Software/DOOM-3-BFG` commit `1caba1979589971b5ed44e315d9ead30b278d8b4`. These findings describe the BFG `neo/d3xp` engine, not `doomclassic` or original Doom 3. All references below are line numbers in that commit. Engine capabilities are confirmed; shotgun-specific retail settings and event ordering are not confirmed.

### 1. The important limitation: no retail shotgun script or assets

`README.txt:15–16` says the source release contains no game data and that game data remains covered by its EULA. Read-only file inspection finds no `.script`, `.md5anim` or `.md5mesh` files. The only tracked `.def` file observed is `neo/d3xp/Game.def` (the build export definition, not a shotgun entity/model definition). Therefore this source cannot establish shotgun pellet count, spread value, fire rate, pump duration, reload stages, recoil amounts, exact firing animation names or sound cue frames. Do not substitute original Doom 3 scripts or mod scripts.

### 2. Selection and input are separate from actual shooting

- `neo/d3xp/Player.cpp`, `idPlayer::Weapon_Combat`, function begins **4822**: switching away first calls `PutAway()` when ready (**4860–4861**); after holstering, the new weapon slot is read from `def_weaponN` (**4871–4873**), the definition is loaded with remembered clip ammo (**4874**), then `Raise()` is called (**4877**). This means a shared weapon object is configured from each weapon's definition rather than a separate C++ shotgun controller.
- `neo/d3xp/Player.cpp`, `idPlayer::Weapon_Combat`, **4899–4903**: held `BUTTON_ATTACK` calls `FireWeapon`; button release calls `EndAttack`.
- `neo/d3xp/Player.cpp`, `idPlayer::FireWeapon`, begins **3419**, checks hidden weapon/readiness and available ammo (**3434–3435**), marks attack held (**3436**), calls `BeginAttack` (**3437**).
- `neo/d3xp/Weapon.cpp`, `idWeapon::BeginAttack`, begins **1792**: records last attack time unless out of ammo (**1793–1794**), returns if script is unlinked (**1797–1798**), optionally stops hum on first press (**1801–1804**), then sets script variable `WEAPON_ATTACK = true` (**1806**). It does **not** launch a pellet directly. `EndAttack` begins **1814** and clears that flag (**1819**).
- `neo/d3xp/Weapon.cpp`, `idWeapon::IsReady`, begins **1831**: readiness includes statuses `WP_RELOAD`, `WP_READY`, and `WP_OUTOFAMMO`, provided the weapon is not hidden. Hence input acceptance is not proof that a shot occurs immediately; the script must decide what to do while reloading.
- `neo/d3xp/Weapon.h`, `weaponStatus_t`, **44–51**: ready, out of ammo, reload, holstered, rising and lowering statuses. `idWeapon` inherits `idAnimatedEntity` at **83**.

### 3. How each weapon gains individual behavior

`neo/d3xp/Weapon.cpp`, `idWeapon::GetWeaponDef`, begins **952**:

1. Clears previous weapon (**962**), loads named entity definition (**970**).
2. Reads ammo type/cost, clip size and low-ammo threshold (**972–975**); silent fire and powered ammo flags (**982–983**).
3. Loads visual kick duration, cap, angles and position (**985–988**).
4. Loads muzzle smoke (**994–1001**), view model from `model_view` (**1024–1025**) and world model (**1028**).
5. Copies definition keys prefixed `snd_` into weapon spawn arguments (**1031–1034**). Finds viewmodel joints `barrel`, `flash`, `eject`, GUI and vent (**1038–1042**) and optional smoke joint (**1044–1048**).
6. Loads `def_projectile` and confirms its spawn class derives from `idProjectile` (**1054–1065**). This enables different projectile types through data.
7. Reads flash material/color/radius/time/shape (**1078–1086**); defaults `flashTime` to 0.25 seconds (**1083**) only if omitted, **not evidence of the retail shotgun value**. Configures view-only light (**1089–1090**); separate world light copies its settings but is suppressed in owner's view (**1114–1118**).
8. Reads brass definition and integer delay (**1141–1149**).
9. Reads sway/history settings (**1178–1183**).
10. Requires the definition's `weapon_scriptobject` (**1185–1192**), links attack/reload/network/raise/lower flags (**1194–1200**), copies full definition into spawn args (**1202**), optionally starts hum (**1204–1207**), marks linked (**1210**) and executes constructor (**1213**).

`neo/d3xp/Weapon.cpp`, `idWeapon::ConstructScriptObject`, begins **2126**: ends old thread (**2129**), obtains constructor (**2132**), clears script object (**2138**), calls and executes constructor (**2139–2140**).

`neo/d3xp/Weapon.cpp`, `idWeapon::UpdateScript`, begins **2186**: only linked weapons and new simulation frames run (**2189–2195**); pending state is applied (**2198–2199**), then script thread executes and applies newly requested states (**2203–2208**) with a transition-loop guard initialized to 10. Reload request flag is cleared afterward (**2211**).

`neo/d3xp/Weapon.cpp`, `idWeapon::Event_WeaponState`, begins **2999**: validates script function (**3002–3005**), stores requested state (**3008**), marks `isFiring` true only when state name is `Fire` (**3019–3023**), stores animation blending frames (**3025**), tells thread to yield (**3026**). `SetState`, begins **1934**, invokes the named script function (**1941–1947**) and stores state/blending (**1948–1950**). Engine events expose ready/reload/holstered statuses (`Event_WeaponReady` **3034**, `Event_WeaponReloading` **3064**, `Event_WeaponHolstered` **3074**); retail script chooses when to emit these.

`neo/d3xp/Weapon.cpp`, `Reload`, begins **1667**, only sets `WEAPON_RELOAD` (**1669**); accompanying comment **1664** says auto-reload is scripted. `Raise` begins **1643** and sets raise flag (**1645**); `PutAway` begins **1654** and sets lower flag (**1657**).

### 4. Animations and mechanical movement

- `neo/d3xp/Weapon.cpp`, `idWeapon::SetModel`, begins **1563**: model is assigned to animator (**1570**) and its animated joint buffer is connected to render entity (**1573**); hidden until an animation is played (**1581–1582**).
- `neo/d3xp/Weapon.cpp`, `idWeapon::Event_PlayAnim`, begins **3247**: resolves animation name (**3250**); if present, shows weapon (**3256–3257**), plays on requested channel at current game time with `FRAME2MS(animBlendFrames)` blending (**3259**), stores animation end time (**3260**). If worldmodel has same named animation, starts it too (**3261–3264**). The blend request resets to zero (**3268**). No shotgun-specific pump movement occurs in this C++ function.
- `neo/d3xp/Weapon.cpp`, `idWeapon::Event_PlayCycle`, begins **3277**, provides the looping counterpart (**3289**) and worldmodel counterpart (**3292–3293**).
- `neo/d3xp/Weapon.cpp`, `idWeapon::Event_AnimDone`, begins **3305**: returns true when `animDoneTime - FRAME2MS(blendFrames) <= gameLocal.time` (**3306**). This lets scripts begin the next action before the old animation ends to allow blending.
- `neo/d3xp/anim/Anim.h`, `FRAME2MS`, begins **48**, computes integer milliseconds as `frames * 1000 / 24` (**49**). This fixed 24-fps helper converts blending requests; do not confuse it with the game's render or physics rate. `neo/d3xp/anim/Anim.cpp`, `idMD5Anim::ConvertTimeToFrame`, begins **555**, uses each animation's `frameRate`: `frameNum = floor(time_ms * frameRate / 1000)` (**577–578**) and a fractional weight from the remainder (**596–597**). `idMD5Anim::GetInterpolatedFrame`, begins **858**, decodes adjacent frame poses (**870–873**) and blends the joint transforms by that fractional weight (**875**). Exact shotgun clip frameRate remains unavailable with no retail animation files.
- `neo/d3xp/anim/Anim_Blend.cpp`, `idAnimBlend::BlendAnim`, begins **1807**: converts animation time to source keyframes (**1845**, **1853**) then gets interpolated animated joint positions/rotations (**1854**). `idAnimator::CreateFrame`, begins **4261**, starts from default pose (**4305**, **4315**), blends active animation channels (**4323**, **4344**), converts joint quaternions to matrices (**4385**) and adds model definition visual offset (**4429**).
- `neo/d3xp/anim/Anim_Blend.cpp`, `idAnimator::GetJointTransform`, begins **4522**, builds frame (**4527**) and returns current joint position and rotation (**4529–4530**).
- `neo/d3xp/Weapon.cpp`, `GetGlobalJointTransform`, begins **1592**: animated viewmodel joint position transforms to world by `local position * viewWeaponAxis + viewWeaponOrigin` (**1595–1596**); joint rotation multiplies weapon axis (**1597**). Thus mechanical movement can come from animated bones, and effects follow those bones. Exact pump/bolt/hand motions cannot be reconstructed without retail skeletal animation data.
- `neo/d3xp/Weapon.cpp`, `InitWorldModel`, begins **901**, reads `model_world` and `joint_attach` (**909–910**), binds world gun to owner's skeleton (**921**), disables interpolation to keep gun and player animation in sync (**925–928**), suppresses world gun in owner's first-person view (**933–935**).

### 5. Animation frame events and sound: capability versus shotgun evidence

`neo/d3xp/Weapon.cpp` event registrations **102–106** connect script-facing animation calls to weapon handlers. The weapon also inherits generic sound events from `idEntity`; `neo/d3xp/Entity.cpp` registrations **145–147** connect `startSoundShader`, `startSound`, `stopSound`.

`neo/d3xp/Entity.cpp`, `idEntity::Event_StartSound`, begins **4365**, delegates to `StartSound` (**4368**) and returns sound duration in seconds (**4369**). `idEntity::StartSound`, begins **1609**, reads `snd_` key from current entity definition (**1619–1626**), only starts on new frames (**1629–1631**), resolves sound declaration and delegates to sound system (**1634–1635**). This is an available script route for firing or pump sounds; missing retail script prevents proving which shotgun sound uses it.

`neo/d3xp/anim/Anim_Blend.cpp`, `idAnim::AddFrameCommand`, begins **282**, validates source frame number and converts definition's 1-based frame numbers to internal 0-based numbers (**292–297**). Supports global script calls (**304–309**), object calls (**313–318**), zero-argument engine events (**319–331**), sound cues, and further effect/AI events. `idDeclModelDef::ParseAnim`, begins **2462**, loads frame commands at **2610**.

`neo/d3xp/Entity.cpp`, `idAnimatedEntity::UpdateAnimation`, begins **5448**, services animation frame events over previous-to-current simulation time when visible (**5460–5462**). `neo/d3xp/anim/Anim_Blend.cpp`, `idAnimator::ServiceAnims`, begins **4164**, calls active animation blends' frame events (**4174–4176**); `idAnimBlend::CallFrameCommands`, begins **1762**, converts elapsed times into frames (**1784–1792**) and ensures first-frame events are called (**1794–1798**). `idAnim::CallFrameCommands`, begins **723**, traverses crossed frames (**731–741**) and invokes script/object/event commands (**743–753**), sound on generic channel (**758**, **763**) or weapon channel (**824**, **829**). This means a sound can be tied to an animation frame rather than a guessed wall-clock delay.

**Do not conflate AI and player weapon events:** `muzzle_flash` framecommand at `Anim_Blend.cpp:528–536` dispatches `AI_MuzzleFlash` at **912**. That event is defined at `neo/d3xp/ai/AI_events.cpp:48` and registered to `idAI::Event_MuzzleFlash` at **186**. Player `idWeapon` instead adds its flash light inside its own projectile launch (**Weapon.cpp:3708**). There is no basis to say the player's shotgun muzzle flash or pellets are launched by the generic AI framecommand. The shotgun retail script/model definition is required to establish its actual cue schedule.

### 6. Confirmed launch-event effect sequence

`neo/d3xp/Weapon.cpp`, `idWeapon::Event_LaunchProjectiles`, begins **3520**, exposed as script event `launchProjectiles` at **61** and registered at **113**:

1. Returns if hidden or no projectile definition (**3535–3542**).
2. Checks and consumes clip/inventory ammo for server/local-controlled prediction (**3546–3574**). Ammo is consumed once per launch-event call, outside the pellet loop; it is not consumed once per pellet.
3. Alerts AI unless `silent_fire` (**3577–3579**).
4. Updates random shader variation and firing time on view/world models (**3584–3589**), allowing data-driven flash/glow materials.
5. Chooses launch origin/axis (**3593**) and accumulates capped visual recoil timer (**3596–3601**).
6. Spawns/launches requested number of projectiles if relevant for server or client visualization (**3606–3618**, **3696**). [Pellet calculation researched by other agent.]
7. Queues shell/brass event once per launch call at configured integer millisecond delay if nonnegative (**3700–3702**).
8. Calls muzzle-flash light unless existing light is continuously on (**3706–3708**).
9. Calls owner's `WeaponFireFeedback` (**3711**) [camera researched by parent/other agent].
10. Sets smoke start time (**3714**).

There is no explicit firing sound or pump/bolt animation call in this launch routine; those can be scheduled by script and animation data. Retail shotgun order remains unverified.

`neo/d3xp/Weapon.cpp`, `GetProjectileLaunchOriginAndAxis`, begins **3498**, may choose an animated barrel origin when `launchFromBarrel` is set (**3502–3505**), otherwise uses player view position (**3508**); critically it **always overwrites axis with playerViewAxis at 3512**. Therefore cosmetic muzzle orientation is not automatically the firing direction. Exact shotgun launchFromBarrel setting is missing.

### 7. Flash, smoke, and ejection follow the animated gun

`neo/d3xp/Weapon.cpp`, `MuzzleFlashLight`, begins **1450**: skips disabled flashes/zero radius unless continuous light is on (**1452–1453**), skips absent flash bone (**1456–1457**), updates location (**1460**), sets material time/random diversity (**1463–1467**), schedules removal at `gameLocal.time + flashTime` (**1470**), creates or updates both view and world lights (**1472–1478**).

`UpdateFlashPosition`, begins **1410**, attaches view light to `flash` joint (**1412**), traces from 16 units behind to 8 units ahead of desired location (**1432–1435**), then moves it to 8 units behind the trace result (**1437**) to keep light away from solid walls; world light stays on world flash joint (**1442**).

`PresentWeapon`, begins **2347**, sets weapon transform, runs script (**2405**), updates animations (**2410**), renders view model (**2419–2422**); emits smoke from smoke joint or barrel or player view fallback (**2444–2454**), removes expired light (**2518–2527**) and updates light attachment as weapon moves (**2530–2534**).

`Event_EjectBrass`, begins **4125**, skips disabled brass, hidden player viewmodel, missing eject joint or definition and clients (**4126–4135**), gets animated eject joint world transform (**4142**), spawns `idDebris` and launches it (**4146–4152**); linear velocity is 40 times sum of player forward/right/up vectors (**4154**), angular components random in [-10,10] (**4155**). This is cosmetic physical debris; exact shotgun timing/model remain retail data.

### 8. A shotgun-specific BFG workaround is confirmed

`neo/d3xp/Weapon.cpp`, `GetMuzzlePositionWithHacks`, begins **2276**, comments state this is for stereo/3D TV/headset laser sight and animation-axis fixes (**2264–2271**). For the single shotgun's PDA icon string, it obtains orientation from `trigger` bone, swaps forward/up axes and negates forward (**2316–2321**). This is a muzzle orientation workaround, **not evidence that trigger movement drives camera recoil or damage**, especially because projectile launch axis is overwritten with player's view axis (**3512**). It confirms particular shotgun bone assumptions in BFG's C++ but not full gun mechanics.

### Candidate firing sequence for documentation

Confirmed engine skeleton: input -> ready/ammo gate -> attack flag -> script updates -> [missing retail shotgun script schedules launch and animation/sound] -> launch gate/ammo -> shader firing time -> launch position -> kick timer -> pellet projectiles -> delayed brass -> flash -> player feedback -> smoke. Animation branch: script-selected named clip -> interpolation/blending of bones -> crossed frame events -> mechanical sounds/effects -> rendered weapon. Do not draw a strict fire-animation-before-launch arrow as fact; actual retail ordering is unknown.


## Recovered source notes: Projectile damage and reactions

## Projectile, pellet, damage, and hit-reaction research

Source inspected: `/workspace/DOOM-3-BFG-Source`, official BFG commit `1caba1979589971b5ed44e315d9ead30b278d8b4` as verified by the parent. This is source analysis only; no game build, game execution, tool installation, repository edits, or retail data inspection.

### What shotgun-specific evidence actually exists

- `neo/d3xp/Game_local.cpp`, global `fastEntityList`, lines **95–96** names `weapon_shotgun` and `projectile_bullet_shotgun`; lines **99–100** separately name the double shotgun and its projectile. This establishes names, not the complete weapon definition or numeric tuning.
- `neo/framework/FileSystem.cpp`, `idFileSystemLocal::BuildOrderedStartupContainer` (**748**), line **831** expects `script/weapon_shotgun.script`; line **827** also expects `script/weapon_base.script`. Searching tracked `*.script` and `*.def` files found no retail shotgun script or projectile/damage definition. Therefore **the shotgun's actual caller arguments, pellet count, spread, projectile velocity/fuse, per-pellet damage, and fire/pump/reload timings remain unconfirmed**.
- Both generic `launchProjectiles` and `launchProjectilesEllipse` events are registered in `neo/d3xp/Weapon.cpp`, lines **61**, **74**, **113**, **126**. The omitted retail script is needed to establish which event a particular shotgun state uses. Do not report ordinary-event or ellipse-event usage for the retail shotgun as proven merely from these native functions.

### Confirmed generic launch sequence

`neo/d3xp/Weapon.cpp`, `idWeapon::Event_LaunchProjectiles`, starts at **3520**. Its arguments are the projectile count, spread, fuse offset, launch power, and damage power. Count and spread are supplied by the caller rather than hardcoded as shotgun constants here.

1. Hidden weapons abort (**3535–3537**); missing projectile definitions warn and abort (**3539–3543**).
2. Server or locally controlled player checks clip availability (**3545–3549**). Ammo is consumed before the projectile loop (**3564–3574**): this is once per launch event, not once per pellet. BFG power ammo has its own amount calculation (**3553–3561**).
3. Unless `silent_fire`, the event alerts nearby AI (**3577–3580**). Shader diversity and time-offset parameters update for view/world gun materials (**3582–3590**).
4. `GetProjectileLaunchOriginAndAxis` is called (**3593**). That function starts at **3498**; a valid barrel joint plus projectile `launchFromBarrel` can choose the muzzle position (**3502–3505**), otherwise origin is the player's view (**3507–3509**). Crucially, **the returned firing axis is forced to `playerViewAxis` at 3512**, even when the muzzle origin came from a gun joint. Thus this ordinary path aims from view orientation; the visible barrel rotation does not automatically determine pellet orientation.
5. The weapon's kick end time is advanced and capped (**3595–3602**).
6. Projectile spawning is gated for server, local attacker, or an instant-hit-marked definition (**3606–3610**). The pellet loop is **3617–3698**; each pellet gets its own randomized direction, entity, Create, and Launch call.
7. Brass is scheduled after the loop, if enabled (**3700–3703**). Muzzle light is triggered (**3706–3709**), owner firing feedback is called (**3711**), and muzzle smoke start time resets (**3714**). These presentation operations are separate from the damage collision that may happen inside launch/physics.

### Exact circular spread calculation

`neo/d3xp/Weapon.cpp`, `idWeapon::Event_LaunchProjectiles`, **3616–3621**:

- Convert caller spread from degrees to radians, `s`.
- Draw two random samples `u` and `v` in the unit interval.
- The lateral radius is `r = sin(s × u)`; the spin is `phi = 2π × v`.
- The candidate direction is forward plus up times `r × sin(phi)`, minus the weapon/view lateral axis times `r × cos(phi)`; normalize this vector.
- Geometric consequence: the actual angle away from forward is `atan(r)`, so for a conventional nonnegative spread below 90 degrees the outer limit is `atan(sin(s))`. The parameter is converted from degrees but is **not implemented as an exact cone half-angle**. For small spread the difference is small.
- This samples radius approximately uniformly for small angles, **not disk area uniformly** (a uniform-area disk would require a different radial sampling rule). The resulting distribution is concentrated toward the center compared with uniform area; no fixed pellet pattern or requirement that one pellet be dead center appears here.
- `neo/idlib/math/Random.h`, `idRandom::RandomFloat`, **82–83**, divides a pseudorandom integer by `MAX_RAND + 1`; values include zero and are below one. The generator updates its seed at **70–72**. Do not describe this as cryptographic randomness.

The distinct ellipse path is `neo/d3xp/Weapon.cpp`, `idWeapon::Event_LaunchProjectilesEllipse`, **3722**, spread at **3801–3810**. It uses the same random spin but independent radius samples for horizontal and vertical sine terms. Two caller spreads set different widths. Retail script evidence is required to identify a weapon using it.

### Physical projectiles versus the networking term "hitscan"

- Ordinary launch creates or reuses an `idProjectile` (**3623–3637**, **3639–3660**). The first pellet establishes a safe launch point (**3664–3674**) by checking the owner's bounds and performing a clip translation toward the muzzle, ignoring the owner. All pellets use that resulting `muzzle_pos`.
- `net_instanthit` is read as `isHitscan` at **3606**, and disables entity synchronization at **3647–3649**. This is also used for networking decisions; it is **not proof of a separate native ray-only shotgun path**.
- `neo/d3xp/Projectile.cpp`, `idProjectile::Spawn`, **110–118**, assigns a rigid-body physics object; `Create`, **220**, sets its clip-model owner at **239** and remembers its owner at **241**.
- `idProjectile::Launch`, **306**, reads a `velocity` vector at **337** and calculates speed as its length multiplied by launch power at **339**. Orientation is aligned with the supplied direction (**377–381**). Initial linear velocity is direction times this speed **plus inherited `pushVelocity`**, at **414**. Gravity, friction, bounce, mass, thrust, fuse, and actor/world detonation flags come from the projectile definition (**333–355**, **405–417**).
- Fuse offset is treated as elapsed time since firing: subtract from a positive fuse and clamp to zero before scheduling explosion/fizzle (**429–440**).
- For fuse `<= 0`, launch immediately calls `RunPhysics` and schedules removal (**424–428**). A comment says "run physics for 1 second", but the actual call has no special one-second parameter. `neo/d3xp/Entity.cpp`, `idEntity::RunPhysics`, **2645**, passes `GetPhysicsTimeStep`; `idEntity::GetPhysicsTimeStep`, **2957–2958**, returns current game time minus previous game time. Therefore do not turn that comment into a confirmed one-second simulation claim.
- `idProjectile::Think`, **476**, runs physics at **501**. `neo/d3xp/physics/Physics_RigidBody.cpp`, `idPhysics_RigidBody::Evaluate`, **830**, converts the step from milliseconds to seconds (**841**), integrates (**891**), and checks for collisions between current and next states (**898**). `CheckForCollisions`, **172**, uses swept clip-model `Motion` at **190**, not merely a point overlap at the new position. `CollisionImpulse`, **114**, invokes the projectile's virtual `Collide` at **161**.
- Without retail projectile speed, fuse, and flags, we cannot establish how many gameplay frames an actual shotgun pellet lives. Very fast short-lived physics projectiles can behave like instant bullets to a player, but **this source inspection did not prove the exact retail shotgun configuration**.

### Collision and per-pellet damage

`neo/d3xp/Projectile.cpp`, `idProjectile::Collide`, starts at **554**.

- Already exploded/fizzled projectiles stop (**562–564**); no-impact surfaces remove them (**579–584**); noclip players remove them (**593–597**).
- An optional definition-level `push` adds an impulse in normalized travel direction on the non-client side (**599–607**), separate from the rigid-body collision impulse.
- `detonate_on_actor` and `detonate_on_world` control whether the projectile detonates or continues/bounces (**617–631**).
- The projectile definition supplies the damage-definition name at **644**.
- If `impact_damage_effect` is enabled, bleeding entities receive their own effect handler; otherwise the projectile supplies material effects (**648–655**). This is independent of checking whether the entity accepts health damage.
- A damageable entity receives damage power, or 1 if that power is zero (**659–664**). Player-owned hits on actors record a hit and multiply this scale by the owner's `PROJECTILE_DAMAGE` powerup modifier (**666–673**).
- Actual virtual `Damage` receives this projectile, its owner, travel direction, damage definition, scale, and the collision clip ID converted to an animation joint at **687** or **690**, subject to multiplayer routing. **Each pellet is a separate collision and damage call**; there is no aggregate "whole shell" damage calculation in this function.
- The direct-hit entity becomes the splash ignore entity (**705**); then `Explode` runs (**709**). Optional splash is separate: `Explode`, **890**, schedules or immediately calls radius damage (**1084–1094**); `Event_RadiusDamage`, **869–872**, only does anything when `def_splash_damage` is nonempty. No retail shotgun splash definition was inspected.
- There is **no explicit shooter-to-target distance falloff or randomized damage calculation in this generic direct-collision path**. Shot spread causes fewer pellets to intersect distant targets, but do not claim specific retail falloff rules without its data and any specialized projectile class.

### Damage depends on the target class

- Monsters/actors: `neo/d3xp/Actor.cpp`, `idActor::Damage`, **2195**, retrieves the damage definition (**2230**), multiplies its integer `damage` by incoming scale and converts to an integer (**2236**), then applies a hit-location rule (**2237**). `idActor::GetDamageForLocation`, **2512–2517**, returns the unmodified amount for an invalid location, otherwise rounds **up** `damage × jointScale`. `SetupDamageGroups`, **2463**, obtains `damage_zone` mappings (**2473**) and `damage_scale` multipliers (**2493–2501**) from actor data. Actual head/limb multipliers are therefore data-dependent.
- Actors subtract damage from health (**2245**), can apply a configured health floor (**2247–2249**), call `Killed` on death (**2335**) or `Pain` when surviving (**2340**), and can gib below -20 health only when both actor and damage definition enable gibbing (**2336–2337**).
- Generic `idEntity::Damage`, `neo/d3xp/Entity.cpp`, **3243**, simply uses the damage definition's integer amount (**3265**) and subtracts health (**3271**); it does **not** use the passed damageScale in this base implementation. Therefore do not describe one universal damage formula for every target type.
- Attached actor parts: `neo/d3xp/AFEntity.cpp`, `idAFAttachment::Damage`, **374–378**, forwards damage to the owning body using the attachment joint; `AddDamageEffect`, **387–391**, similarly forwards a rewritten joint collision.
- Players have a separate path: `neo/d3xp/Player.cpp`, `idPlayer::CalcDamagePoints`, **8165**, starts with definition damage/default20 (**8170**), applies location (**8171**), single-player skill modifiers (**8174–8192**), damage scale (**8195**), self-damage rules (**8197–8204**), invulnerability (**8207–8217**), armor (**8222–8243**), and team rules (**8245–8252**). `idPlayer::Damage`, **8417**, adds knockback velocity (**8469–8472**) and a 50–200ms movement restriction (**8475**). This is an incoming-hit reaction, not the shooter's firing recoil.

### Monster hit reactions are gated and script-driven

- `neo/d3xp/Actor.cpp`, `idActor::Pain`, **2368**, debounces reactions if before the next permitted pain time (**2377–2379**) and advances that timer by `pain_delay` (**2382**). Pain delay is loaded from seconds and converted to milliseconds in `idActor::Spawn`, **539**; pain threshold loads at **540**.
- Pain sound choices use **remaining health bands**, not the size of this single hit: above75 small, above50 medium, above25 large, else huge (**2384–2391**).
- Disallowed pain, a future `painTime`, or damage below `pain_threshold` prevent an animation reaction (**2394–2401**). Since `Damage` runs per pellet, the damage argument tested here is an individual damage call, not an aggregated shell total.
- Pain-animation names are selected from prefix and damage group, with fallbacks to generic pain (**2403–2435**). This function selects a name and returns true (**2443**); it does **not** directly play that animation.
- `neo/d3xp/ai/AI.cpp`, `idAI::Pain`, **3279**, calls the actor pain function, assigns the result to `AI_PAIN`, sets `AI_DAMAGE`, and forces a blink (**3282–3286**). It records special damage (**3291**) and can make the attacker its enemy if the AI reaction flags allow (**3296–3301**). `idAI::LinkScriptVariables`, **1233–1234**, binds those AI flags to script fields. Retail AI scripts are needed to establish how a particular monster consumes them and plays animation states.
- `idAI::Killed`, **3380**, stops movement (**3438**), clears enemy and sets `AI_DEAD` (**3440–3441**), starts ragdoll when available (**3449–3450**), and enters script `state_Killed` (**3467–3469**). A specific monster's death animation/ragdoll data remains absent.

### Impact visuals follow surface and animation joints

- `neo/d3xp/Projectile.cpp`, `idProjectile::DefaultDamageEffect`, **731**, chooses a surface-specific sound, then metal/impact fallbacks (**735–753**), and a surface-specific or general decal (**757–762**).
- `neo/d3xp/Entity.cpp`, `idAnimatedEntity::AddDamageEffect`, **5571**, requires blood effects and a valid joint (**5576–5588**), converts the world impact point/normal/direction to joint-local coordinates (**5593–5598**), and calls `AddLocalDamageEffect` (**5600**).
- `AddLocalDamageEffect`, **5617**, selects material-dependent sounds (**5638–5645**), blood splats (**5648–5655**), model wound overlays (**5658–5667**), and optional bleeding particle data (**5671–5687**).
- `UpdateDamageEffects`, **5695**, asks the animator for the joint transform each update (**5719**) and transforms the saved local impact point back into world space (**5720–5723**). Consequently, blood emitters remain attached to the moving body part rather than floating at the original world collision location.

### Multiplayer boundary, if included

- `neo/d3xp/Weapon.cpp`, ordinary launch: clients mark predicted spawned entities to skip replication (**3629–3634**); instant-hit definitions disable entity sync (**3647–3649**). Non-instant projectiles use prediction keys (**3650–3656**), and server launches for remote owners queue catch-up simulation after clamping elapsed client time (**3679–3689**).
- `neo/d3xp/Player.cpp`, `idPlayer::Damage`, **8490–8495**, avoids predictive damage feedback on the local victim. Actual player health is dealt by the server (**8537–8540**); a locally controlled client attacker instead sends a reliable hit message (**8541–8560**).
- `neo/d3xp/Game_network.cpp`, `idGameLocal::ServerProcessReliableMessage` (**493**), `GAME_RELIABLE_MESSAGE_CLIENT_HITSCAN_HIT` branch **561**: host reads hit metadata (**562–568**), traces from the weapon launch origin to the reported victim joint (**593–604**), rejects if another object is hit (**606–608**), and invokes victim Damage (**610–613**). This is a host line-of-sight validation ray, distinct from the initial pellet simulation.

### Godot design implications (recommendations, not Doom facts)

- Treat one accepted shot as one ammo/cooldown event containing N pellet queries. Give each hit a clear target-side damage rule; allow pellet damage accumulation but coalesce cosmetic pain reactions so a shotgun does not restart a reaction N times.
- Separate viewmodel muzzle position and animation from the aim direction. For an original controller, decide explicitly whether recoil changes the real aim, only the rendered gun, or a recoverable camera offset; the three must not accidentally feed each other.
- Use original Resource data for pellet count, spread distribution, damage, hit-zone rules, recoil, effects, and timing. Godot may use per-pellet rays for an original shotgun and simulated projectiles for slower rounds; this would be an original implementation choice, not a claim of identical native Doom representation.
- Compute hit rays using camera aim but verify the muzzle-to-hit line if preventing firing through a near wall. Physical-projectile paths need swept collision during each physics step.
- Make sound/decal/particle selection a surface/target response system; attach moving wounds to the original character's bone or bone-attached node where needed.

### Highest-value unresolved step

Read the **user's legally owned Doom 3 BFG retail shotgun script and relevant weapon/projectile/damage definitions locally**, if they provide access in a later approved task. Record only factual values and state sequences; do not publish licensed assets or complete retail scripts. Required candidates are `script/weapon_shotgun.script`, `script/weapon_base.script`, definitions for `weapon_shotgun`, `projectile_bullet_shotgun`, its named `def_damage`, and shotgun model animation declarations. This resolves the main uncertainty left by the public C++ source: actual per-weapon configuration and mechanical animation chronology.
