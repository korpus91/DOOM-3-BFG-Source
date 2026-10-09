# Doom 3 BFG weapon research for Shooter 1946

**Task 1 source investigation and original controller design completed; exact retail shotgun reconstruction remains PARTIAL. Task 2 is now authorized, with retail-data inspection blocked on locating the files.** The public C++ source confirms the execution machinery, but omits the scripts, definitions and animation assets that specify the retail shotgun's numbers and choreography. No game was compiled or run, no tools were installed, and no Shooter 1946 files or assets were changed.

In plain English: pressing fire sets a signal for a weapon script. That script decides when a shot actually happens. The engine moves the visible gun and kicks the camera using separate calculations. Its ordinary multi-projectile event spends ammunition once, creates independently scattered pellets, and lets each collision damage its target separately. Pumping, hand movement and firing cadence cannot be reconstructed fully without the missing retail data.

## Source identity and reference convention

| Item | Verified value |
|---|---|
| Official repository and source branch | `id-Software/DOOM-3-BFG`, `master` |
| User fork and exclusive pull-request target | `korpus91/DOOM-3-BFG-Source`, base `master` |
| Exact source commit | `1caba1979589971b5ed44e315d9ead30b278d8b4` |
| Checkout | `/workspace/DOOM-3-BFG-Source`; initial local branch `work` at the source commit |
| Dedicated research branch | `research/doom-bfg-weapons-task1` |
| Required source files | `neo/d3xp/Weapon.cpp` and `neo/d3xp/Weapon.h`, both present |
| Weapon.cpp SHA256 | `a0b202d1d9890d3087d937ed88ea0c96efc4eb6dee372f1cbeec95598b6934ca` |
| Weapon.h SHA256 | `cdc74c55d866331e44c3858f2a8125837e529884b08cfdfa1b63cc0d2845a5e1` |

Both repositories' `refs/heads/master` resolved to that SHA during verification. The two Weapon files were independently compared byte-for-byte with official raw files at that commit. GitHub's repository API also confirms this fork's parent is `id-Software/DOOM-3-BFG`. This is the 2012 BFG source. The bundled `doomclassic` directory was not substituted for the BFG engine.

Every source line below refers to the pinned commit, not the moving research branch. For readability, a filename such as `Weapon.cpp:985` means **`neo/d3xp/Weapon.cpp`, exact line 985**; the same `neo/d3xp/` prefix applies to Player.cpp, Player.h, Weapon.h, PlayerView.cpp, Actor.cpp, Entity.cpp, AFEntity.cpp, Projectile.cpp, Projectile.h, Game_local.cpp, Game_network.cpp, gamesys/, anim/, ai/ and physics/. Other paths are written in full. Function names identify the surrounding implementation; ranges identify a calculation spanning several lines.

Immutable entry points: [Weapon.cpp](https://github.com/id-Software/DOOM-3-BFG/blob/1caba1979589971b5ed44e315d9ead30b278d8b4/neo/d3xp/Weapon.cpp#L952), [Weapon.h](https://github.com/id-Software/DOOM-3-BFG/blob/1caba1979589971b5ed44e315d9ead30b278d8b4/neo/d3xp/Weapon.h#L83), [Player.cpp](https://github.com/id-Software/DOOM-3-BFG/blob/1caba1979589971b5ed44e315d9ead30b278d8b4/neo/d3xp/Player.cpp#L8793), [PlayerView.cpp](https://github.com/id-Software/DOOM-3-BFG/blob/1caba1979589971b5ed44e315d9ead30b278d8b4/neo/d3xp/PlayerView.cpp#L326), [Projectile.cpp](https://github.com/id-Software/DOOM-3-BFG/blob/1caba1979589971b5ed44e315d9ead30b278d8b4/neo/d3xp/Projectile.cpp#L306).

## 1. Start with the shotgun: what is actually present

`Game_local.cpp`, global `fastEntityList`, lines **95–96**, names `weapon_shotgun` and `projectile_bullet_shotgun`; lines **99–100** name the double-shotgun equivalents. `neo/framework/FileSystem.cpp`, `idFileSystemLocal::BuildOrderedStartupContainer` (function **748**), expects `script/weapon_shotgun.script` at **831** and `script/weapon_base.script` at **827**. These establish the intended data names, not their contents.

[README.txt:15](https://github.com/id-Software/DOOM-3-BFG/blob/1caba1979589971b5ed44e315d9ead30b278d8b4/README.txt#L15) explicitly excludes game data. The checkout has no retail `.script`, `.md5anim`, `.md5mesh` or PK4 files. Its tracked `.def` is `neo/d3xp/Game.def`, a build export definition. Therefore actual pellet count, spread, per-pellet damage, projectile speed/fuse, recoil strengths, pump duration, reload stages and sound frames are **unconfirmed**. Original Doom 3 scripts, mods and gameplay videos were not used as substitutes.

There is a concrete shotgun-specific exception in C++. `idWeapon::GetMuzzlePositionWithHacks`, `Weapon.cpp` function **2276**, obtains barrel position at **2296–2301**. At **2316–2322**, it identifies the single shotgun by its PDA icon string, obtains the animated **trigger joint orientation**, discards that joint's position, swaps basis axes 0 and 2, and negates axis 0. The comment explains the barrel axis is unsuitable. Double-shotgun definition names receive another axis correction at **2326–2335**. This is an orientation workaround, not evidence that trigger movement causes recoil or damage. `idPlayer::UpdateLaserSight`, `Player.cpp` **7467**, uses this corrected muzzle basis at **7476–7480, 7497–7499** for the stereo laser sight. Ordinary projectile direction is separately overridden with the camera basis, as explained below.

Both `launchProjectiles` and `launchProjectilesEllipse` are exposed and registered in `Weapon.cpp:61, 74, 113, 126`. The missing retail script is needed to prove which event a particular shotgun state calls. The following sections distinguish confirmed shared engine behavior from those missing decisions.

## 2. Input, scripts and individual weapon behavior

### Input requests an attack; it does not itself launch pellets

`idPlayer::Weapon_Combat`, `Player.cpp` function **4822**, handles switching: lower the old weapon (**4860–4861**), read the next `def_weaponN` (**4873**), call `GetWeaponDef` with remembered clip ammunition (**4874**), then raise it (**4877**). Held attack calls `FireWeapon`; release calls `EndAttack` (**4899–4903**).

`idPlayer::FireWeapon` starts at **3419**, checks visibility/readiness/ammunition at **3434–3435**, then calls `BeginAttack` at **3437**. `idWeapon::BeginAttack`, `Weapon.cpp` **1792**, checks script linkage (**1797–1798**), optionally stops a hum on the initial press (**1801–1804**), and sets linked `WEAPON_ATTACK` at **1806**. `EndAttack` (**1814**) clears it at **1819**. Neither is the pellet-launch function. `IsReady` (**1831**) includes reload, ready and out-of-ammo statuses when visible, so this input gate does not promise an immediate shot during reload. The script decides the response. Weapon statuses are declared in `Weapon.h:44–51`; `idWeapon` inherits `idAnimatedEntity` at **83**.

### `idWeapon::GetWeaponDef` is the main configuration junction

All lines in this table are in `Weapon.cpp`, `idWeapon::GetWeaponDef`, function **952**.

| Lines | Confirmed operation | Why it matters |
|---|---|---|
| 962, 970 | Clear previous weapon and load named entity definition | One shared weapon object can acquire different behavior |
| 972–975, 982–983 | Read ammo type/cost, clip size, low-ammo, silent-fire and powered-ammo fields | Firing rules are configurable |
| 985–988 | Read model-kick times, angles and displacement | Visible recoil has per-definition settings |
| 994–1001, 1024–1028 | Load smoke, first-person model, separate world model | Presentation is selected by data |
| 1031–1048 | Copy sound keys; find barrel, flash, eject, GUI, vent and smoke joints | Effects can follow animated sockets |
| 1054–1065 | Load `def_projectile`; require a class derived from `idProjectile` | Different projectile implementations can be selected |
| 1078–1090, 1114–1118 | Configure muzzle-light material/color/radius/time and separate view/world lights | Flash is more than a gun animation |
| 1141–1149 | Read brass definition and delay | Ejection is independently scheduled |
| 1178–1183 | Read turning-lag and movement-lag settings | Movement response varies by definition |
| 1185–1200 | Require `weapon_scriptobject` and link attack/reload/network/raise/lower flags | The script supplies the weapon state machine |
| 1202–1213 | Copy definition into spawn arguments, optionally start hum, link and construct script | Definition and script jointly finish setup |

`idWeapon::ConstructScriptObject` (**2126**) obtains and executes the constructor (**2132, 2138–2140**). `UpdateScript` (**2186**) runs only when linked and on new simulation frames (**2189–2195**), applies pending states (**2198–2199**), and executes the thread/state-transition loop with a guard initialized to 10 (**2203–2208**). It clears the reload request at **2211**.

`Event_WeaponState` (**2999**) validates the requested script function (**3002–3005**), stores the next state (**3008**), sets `isFiring` according to the literal state name `Fire` (**3019–3023**), saves blend frames and yields (**3025–3026**). `SetState` (**1934**) calls that script function and stores state/blending (**1941–1950**). `Reload` (**1667, 1669**), `Raise` (**1643, 1645**) and `PutAway` (**1654, 1657**) set script flags. The comment at **1664** explicitly describes auto-reload as scripted.

Thus individual weapons obtain behavior from entity definitions, their script objects, selected projectile classes, animation/model definitions, sound/effect data, and some explicit C++ exceptions. A single universal hardcoded shotgun routine does not contain all retail mechanics.

## 3. First-person positioning, hands and mechanical animation

### Procedural positioning before skeletal animation

`idPlayer::CalculateViewWeaponPos`, `Player.cpp` **8793**, begins with the already calculated camera origin/axis, adds configured gun position plus movement lag in that camera basis (**8799–8808**). Engine defaults in `gamesys/SysCvar.cpp:276–279` are gun offset `(3,0,0)` and scale 1; these are source units and fallback settings, not verified retail shotgun placement.

The same function adds movement bob (**8810–8829**): roll and pitch use horizontal speed × bob sine × 0.005; yaw uses 0.01; alternate footstep half-cycles reverse roll/yaw. It adds turning history, with an extra multiplayer scale whose engine default is zero (`gamesys/SysCvar.cpp:284`). Landing displacement uses one quarter of `landChange` (**8831–8838**), with 150 ms deflection and 300 ms return (`Player.h:57–58`). It adds small idle angular drift `(horizontal speed + 40) × sin(time in seconds) × 0.01` (**8841–8846**), independent weapon pitch (**8849**), then composes the angular matrix, gun scale and camera basis (**8851–8854**).

Two distinct lag calculations explain why the gun trails player motion:

- `idPlayer::GunTurningOffset`, `Player.cpp` **8701**, returns zero during the first 64 frames (**8706**); then averages logged input angles with yaw-wrap correction, subtracts current angles, scales and clamps each component (**8716–8745**). History is logged at **6125**, with 64 slots in `Player.h:755–756`. Definition fallbacks are 10 averaged frames, scale 0.25 and maximum 10 degrees (`GetWeaponDef`, `Weapon.cpp:1178–1180`). They are not proven shotgun overrides.
- `idPlayer::GunAcceleratingOffset`, `Player.cpp` **8756**, processes recent movement-change records (**8763–8783**). Each contributes its direction times a scale and `(cos(2π × age/window) − 1) / 2`. The weight starts at zero, reaches −1 halfway through, then returns to zero. Fallback window/scale are 400 ms and 0.005 (`Weapon.cpp:1182–1183`); history has 16 slots (`Player.h:757–758`). These records include input-command changes in `Think` (**7557–7571**) and a fixed vertical jump value of 200 (**6976–6982**), so this is not simply measured physical acceleration.

`idWeapon::PresentWeapon`, `Weapon.cpp` **2347**, caches first-person view (**2348–2349**). Its ordinary weapon branch calls `CalculateViewWeaponPos` (**2374**), applies the smoothly lowered hide offset (**2378–2393**), then calls `MuzzleRise` (**2396**). It sets the transform (**2400–2401**), updates script (**2405**), GUI (**2407**) and animation (**2410**), then renders (**2419–2422**). The flashlight has a separate branch (**2351–2371**). Viewmodel visibility is owner-only (**2413**), with an optional depth hack (**2416**) and separate world-model/shadow handling (**2425–2433**).

### Animation supplies the actual hand, pump and bolt motion

`idWeapon::SetModel`, `Weapon.cpp` **1563**, loads the animator model (**1570**) and render joint buffer (**1573**); it is initially hidden (**1581–1582**). `Event_PlayAnim` (**3247**) resolves a named animation (**3250**), shows the model (**3256**), starts the selected channel with a blend (**3259**) and records its end time (**3260**). A matching world-model clip is started too (**3261–3264**). `Event_PlayCycle` (**3277, 3289**) is the looping version. `Event_AnimDone` (**3305–3306**) compares current time against animation end minus requested blend time, allowing an early transition for blending.

The blend helper `FRAME2MS`, `anim/Anim.h:48–49`, converts frames using `frames × 1000 / 24`. This does **not** establish the game's physics/render rate or every clip's frame rate. `idMD5Anim::ConvertTimeToFrame`, `anim/Anim.cpp` **555**, uses the clip's own frame rate (**577–578**) and a fractional remainder (**596–597**). `GetInterpolatedFrame` (**858**) decodes adjacent poses and blends joint transforms (**870–875**). `idAnimBlend::BlendAnim`, `anim/Anim_Blend.cpp` **1807, 1845–1854**, obtains that pose. `idAnimator::CreateFrame` (**4261**) starts from the default pose (**4305, 4315**), blends active channels (**4323, 4344**), converts quaternions to matrices (**4385**) and adds model visual offset (**4429**).

`idAnimator::GetJointTransform`, `anim/Anim_Blend.cpp` **4522**, builds the frame and returns a joint pose (**4527–4530**). `idWeapon::GetGlobalJointTransform`, `Weapon.cpp` **1592**, transforms joint position by the procedural weapon axis and origin and composes its orientation (**1595–1597**). Consequently a barrel, eject socket or hand combines skeletal movement with movement sway and model recoil. `InitWorldModel` (**901**) separately reads world model/attachment (**909–910**), binds it to the player's skeleton (**921**), disables interpolation to maintain animation synchronization (**925–928**) and hides it from its owner's view (**933–935**).

The engine confirms this mechanism. It does not provide the missing shotgun clips' hand placement, pump travel, bolt curves or exact timing. There is no basis to invent those motions from `MuzzleRise` alone.

## 4. Visual recoil and camera recoil use different calculations

### Visible gun: capped accumulated time, linear recovery

`Weapon.h:299–304` declares the model-kick state. `idWeapon::GetWeaponDef`, `Weapon.cpp:985–988`, reads `muzzle_kick_time` and `muzzle_kick_maxtime` in seconds and converts them to milliseconds, plus `muzzle_kick_angles` and `muzzle_kick_offset`. `neo/idlib/math/Math.h:58` defines the conversion; angle components are pitch/yaw/roll (`neo/idlib/math/Angles.h:41–43, 53–55`) in degrees, as used by `idAngles::ToForward`, `neo/idlib/math/Angles.cpp:117, 120–123`. Position remains in engine units; no metre conversion is assumed.

`idWeapon::Event_LaunchProjectiles`, `Weapon.cpp:3596–3601`, advances the kick end time as follows: take the later of the previous end and current real-client time, add the configured kick duration, then cap it at current time plus maximum duration. Rapid shots can accumulate remaining kick time up to that limit.

`idWeapon::MuzzleRise`, function **2091**, calculates remaining time at **2097**, exits for expired/disabled kick (**2098, 2102**), clamps remaining time (**2106**), then divides by maximum duration (**2110**). This fraction scales both configured angles and displacement (**2111–2112**). It subtracts the rotated displacement from the weapon origin (**2114**) and composes the recoil rotation with the weapon axis (**2115**). In plain English: the visible gun shifts and tilts, then returns linearly as its timer expires. Positive local displacement is subtracted; exact direction depends on the missing definition. This is an envelope driven by time, not a physical spring.

The ordinary event accumulates with `realClientTime`, whereas `MuzzleRise` subtracts `gameLocal.time`. The ellipse event accumulates with `gameLocal.time` instead (`Weapon.cpp:3787–3794`). These are engine clocks, not OS wall time: `idGameLocal::RunFrame`, `Game_local.cpp:2299–2309`, updates fast/slow times; `SelectTimeGroup:4634–4641` selects them. `idGameLocal::ClientRunFrame`, `Game_network.cpp` **1031, 1056–1065**, preserves the furthest predicted real-client time. Those distinctions are recorded; exhaustive slow-motion/prediction replay behavior was outside this timebox.

### Camera: a shared, gated kick with quadratic recovery

After projectile launch, `Weapon.cpp:3711` calls `idPlayer::WeaponFireFeedback`, `Player.cpp` **3376**. That function resets blinking, sets `AI_WEAPON_FIRED`, calls player-view feedback and applies configured rumble for a locally controlled player (**3377–3395**).

`idPlayerView::WeaponFireFeedback`, `PlayerView.cpp` **326**, reads integer `recoilTime` in milliseconds (**327**). It accepts a new firing kick only if the duration is nonzero and the old `kickFinishTime` is strictly earlier than slow-game time (**329**). It reads `recoilAngles` with fallback `5 0 0` (**331**), stores them (**332**), and sets the end to slow time plus `g_kickTime × recoilTime` (**333**). It does not extend or replace an active camera kick. Incoming damage writes the same state in `DamageImpulse` (**241, 265–284**), so this gate applies to an active damage kick as well as an active firing kick.

`idPlayerView::AngleOffset`, `PlayerView.cpp` **373**, uses remaining slow-game milliseconds (**377**). The returned angle is stored angles × remaining milliseconds squared × `g_kickAmplitude` (**379**), clamped to ±70 degrees on each axis (**381–386**), or zero when expired. Defaults are `g_kickTime=1` and `g_kickAmplitude=0.0001` (`gamesys/SysCvar.cpp:153–154`). This is **not** normalized interpolation: increasing duration also increases initial magnitude quadratically before clamping. Illustrative only: a 100 ms kick with default amplitude initially scales the stored angle by 1; halfway through it scales by 0.25. That is not a claim about the shotgun's retail duration.

`idPlayer::GetViewPos`, `Player.cpp` **8950**, starts at eye position plus bob (**8961**) and combines input view angles, bob angles and `AngleOffset` (**8962**), then gravity orientation (**8964**). It does not permanently assign recoil back into input `viewAngles` here. Nodal-pivot offsets can also alter camera position (**8966–8971**); engine defaults are X=3, Z=6 (`gamesys/SysCvar.cpp:280–281`). `CalculateFirstPersonView` (**8980**) normally calls this at **8997**; its optional animated player-camera path also includes kick (**8981–8994**). Sound-shake code at **8998–9001** is disabled by `#if 0`. `CalculateRenderView` (**9023**) copies the ordinary first-person origin/axis at **9066–9072**.

The visible gun inherits this camera base and then receives its separate model kick. `idPlayer::Think`, `Player.cpp:7653–7665`, and the client path at **9731–9737** calculate view before updating the weapon; `PresentWeapon` also applies model kick before running the script. Therefore this static trace does not promise newly generated feedback appears in the same frame. `independentWeaponPitchAngle` is initialized to zero (**1278**) and read at **8849**, with no active head-tracking assignment found; its HMD-related comment is not evidence of a complete VR implementation.

## 5. Firing direction, pellet spread and projectile simulation

### One launch event, multiple independent pellets

`idWeapon::Event_LaunchProjectiles`, `Weapon.cpp` **3520**, receives count, spread, fuse offset, launch power and damage power from its caller. It rejects hidden weapons or absent projectile definitions (**3535–3543**). On the server or locally controlled prediction path it checks ammunition (**3546–3549**) and consumes it (**3564–3574**) **once per event**, outside the pellet loop. Powered ammo has a separate amount calculation (**3553–3561**). Unless silent, firing alerts AI (**3577–3579**); view/world shader randomness and time update at **3584–3589**.

`GetProjectileLaunchOriginAndAxis`, `Weapon.cpp` **3498**, chooses the animated barrel origin when a barrel exists and projectile data enables `launchFromBarrel` (**3502–3505**); otherwise it uses the player-view origin (**3508–3509**). It **always sets the outgoing axis to `playerViewAxis` at 3512**, even after the shotgun orientation workaround. Therefore cosmetic model recoil can affect a barrel-based origin but does not directly set ordinary pellet direction. Camera kick can affect the cached view basis on a subsequent view update.

The ordinary event chooses that origin/axis at **3593**, advances model kick (**3596–3601**), then enters a projectile loop (**3617–3698**) under server/client spawning rules (**3606–3610**). It creates or reuses a projectile (**3623–3637**), checks its class (**3639–3642**), calls `Create` (**3660**) and `Launch` (**3696**). The first pellet establishes a safe muzzle position using owner bounds and a clip translation ignoring the owner (**3664–3674**); subsequent pellets share the adjusted position.

### Exact circular spread calculation

In `Event_LaunchProjectiles`, `Weapon.cpp:3616–3621`, convert the supplied spread from degrees to radians, called `s`. For each pellet, draw independent random values `u` and `v` in `[0,1)`. Set lateral radius `r = sin(s × u)` and spin `phi = 2π × v`. Add `up × r × sin(phi)` and subtract `side × r × cos(phi)` from forward, then normalize the result. `idRandom::RandomFloat`, `neo/idlib/math/Random.h:82–83`, establishes that interval; seed update is at **70–72**.

For an orthonormal basis the angular deflection is `atan(r)`. For conventional nonnegative spreads below 90 degrees, its outer limiting angle is `atan(sin(s))`, not exactly the supplied spread angle. Small-angle sampling is approximately uniform in radius, which puts more density near the center than uniform disk-area sampling. No fixed pattern or guaranteed central pellet appears in this function. Actual shotgun count/spread remain unknown.

`Event_LaunchProjectilesEllipse`, `Weapon.cpp` **3722**, is a separate available event. At **3801–3810** it uses one random spin with separate sine-scaled random radii for its two caller-supplied spreads. It obtains muzzle origin directly from the joint or view (**3778–3785**), but **also uses `playerViewAxis` for direction at 3809**. Its power argument is passed as launch power at **3837**; damage power uses the default 1 from `idProjectile::Launch`, `Projectile.h:55`. Do not silently treat the two event signatures or clock paths as identical, or claim retail shotgun usage without its script.

### Physical simulation versus the networking label “instant hit”

`net_instanthit` is read in ordinary launch (`Weapon.cpp:3606`) and disables projectile entity synchronization (**3647–3649**). It does not by itself prove a standalone ray-only native shotgun implementation. Non-instant projectiles receive prediction keys (**3650–3656**); server launches for remote owners can queue time-clamped catch-up simulation (**3679–3689**).

`idProjectile::Spawn`, `Projectile.cpp:110–118`, installs rigid-body physics. `Create` (**220**) sets the collision owner and projectile owner (**239, 241**). `Launch` (**306**) reads definition velocity (**337**) and uses its length × launch power as speed (**339**). It aligns orientation with the chosen direction (**377–381**) and sets initial velocity to direction × speed plus inherited `pushVelocity` (**414**). Mass, gravity, friction, bounce, thrust, fuse and detonation flags come from projectile data (**333–355, 405–417**).

For a positive fuse, elapsed firing time is subtracted and clamped before explosion/fizzle is scheduled (`Launch:429–440`). For fuse ≤ 0, the server/non-client or locally predicted path immediately runs physics and schedules removal (**424–428**). Although a nearby comment says “run physics for 1 second,” `idEntity::RunPhysics`, `Entity.cpp` function **2603**, actually passes `GetPhysicsTimeStep` into evaluation (**2645**); that getter returns current minus previous game time (**2957–2958**). The comment does not prove a special one-second integration step.

`idProjectile::Think`, `Projectile.cpp` **476**, runs physics at **501**. `idPhysics_RigidBody::Evaluate`, `physics/Physics_RigidBody.cpp` **830**, converts milliseconds to seconds (**841**), integrates (**891**) and tests collision between old/new states (**898**). `CheckForCollisions` (**172**) uses swept clip-model `Motion` (**190**), not merely a point overlap at the destination. `CollisionImpulse` (**114**) calls the projectile's virtual `Collide` (**161**).

Without retail speed, fuse, flags and class selection, actual shotgun pellet lifetime and the exact retail configuration cannot be certified. Very fast projectiles may feel instantaneous; that is not evidence of a different code path.

## 6. Damage and hit reactions

### Each pellet collides and deals damage independently

`idProjectile::Collide`, `Projectile.cpp` **554**, rejects already finished projectiles (**562–564**), removes them on no-impact surfaces (**579–584**) or noclip players (**593–597**), and optionally applies definition-controlled travel-direction push on the non-client side (**599–607**). Actor/world detonation flags determine whether it continues/bounces or detonates (**617–631**).

The damage-definition name comes from `def_damage` (**644**). Impact effects can be sent to bleeding entities or generic surface effects (**648–655**), independently of whether health damage is accepted. For damageable targets, the scale starts from damage power, using 1 when that power is zero (**659–664**). Player-owned hits on actors apply the owner's `PROJECTILE_DAMAGE` modifier (**666–673**). The virtual target `Damage` receives projectile, owner, travel direction, definition, scale and collision joint (**687, 690**, subject to multiplayer routing). Each pellet produces a separate call; this function does not aggregate an entire shell into one damage amount.

The direct target becomes the splash-ignore entity (**705**) before `Explode` (**709**). Optional splash is separate: `idProjectile::Explode` (**890, 1084–1094**) schedules or applies radius damage; `Event_RadiusDamage` (**869–872**) requires a nonempty `def_splash_damage`. No retail shotgun splash definition was inspected. This generic direct-collision path has no explicit shooter-to-target distance falloff or random damage roll; that does not establish universal retail rules or exclude a specialized projectile class.

### Target classes apply different rules

| Target path and exact source | Confirmed calculation or sequence |
|---|---|
| `idActor::Damage`, `Actor.cpp` **2195**, definition **2230**, scale **2236**, location **2237** | Convert definition damage × incoming scale to an integer, then apply location rules. `GetDamageForLocation` (**2512–2517**) leaves invalid locations unchanged; otherwise rounds up damage × joint scale. `SetupDamageGroups` (**2463, 2473, 2493–2501**) loads zone mappings/scales from actor data. |
| Actor feedback/health, same function **2241, 2245–2249, 2335–2340** | Attacker `DamageFeedback` may modify the amount before health subtraction. Apply configured health floor; call `Killed` or `Pain`. Gibbing below −20 also requires actor and damage-definition permission. |
| `idEntity::Damage`, `Entity.cpp` **3243, 3265–3271** | Base implementation starts from the definition integer and does not apply its incoming scale argument itself; attacker feedback can still modify damage before subtraction. It is not the actor formula. |
| `idPlayer::DamageFeedback`, `Player.cpp` **8131, 8134–8138** | Client returns early; otherwise player attacker feedback multiplies damage by the `BERSERK` modifier. Therefore definition × scale × hit zone is not a universally final formula. |
| `idAFAttachment::Damage`, `AFEntity.cpp` **374–378** | Forward damage to the owning body with its attachment joint. `AddDamageEffect` (**387–391**) also forwards a rewritten joint collision. |
| `idPlayer::CalcDamagePoints`, `Player.cpp` **8165** | Definition damage/default20 (**8170**), location (**8171**), single-player skill (**8174–8192**), incoming scale (**8195**), self-damage (**8197–8204**), invulnerability (**8207–8217**), armor (**8222–8243**) and team rules (**8245–8252**) form a separate player path. These defaults are not shotgun damage. |
| `idPlayer::Damage`, `Player.cpp` **8417, 8469–8475** | Applies knockback velocity and a 50–200 ms movement restriction. This is the victim's reaction, separate from shooter recoil. |

### Pain is filtered; damage does not guarantee an immediate flinch

`idActor::Pain`, `Actor.cpp` **2368**, first debounces reactions (**2377–2379**) and advances its timer using `pain_delay` (**2382**). `idActor::Spawn` loads delay in seconds converted to milliseconds and a threshold (**539–540**). Sound bands depend on **remaining health**, not the size of the shot: above75, above50, above25, otherwise the lowest-health band (**2384–2391**).

Disallowed pain, a future allowed-pain time or damage below `pain_threshold` can suppress animation reaction (**2394–2401**). Since damage arrives per pellet, that argument is one damage call, not the shell's total. `Pain` selects a zone/prefix animation name with fallbacks (**2403–2435**) and returns true (**2443**); it does not directly play the clip.

`idAI::Pain`, `ai/AI.cpp` **3279**, calls the actor function, sets `AI_PAIN` from its result, sets `AI_DAMAGE` and forces blinking (**3282–3286**). It records special damage and may select the attacker as enemy (**3291–3301**). The flags are linked to script in `idAI::LinkScriptVariables` (**1233–1234**). Retail AI scripts determine how monsters consume them. `idAI::Killed` (**3380**) stops movement (**3438**), clears enemy/sets death (**3440–3441**), starts ragdoll when available (**3449–3450**) and enters script `state_Killed` (**3467–3469**). Specific monster choreography remains missing.

### Impact sound, wounds and blood are separate responses

`idProjectile::DefaultDamageEffect`, `Projectile.cpp` **731**, selects surface sound/fallbacks (**735–753**) and surface/general decals (**757–762**). `idAnimatedEntity::AddDamageEffect`, `Entity.cpp` **5571**, requires blood effects and a valid joint (**5576–5588**), converts world hit data to joint-local coordinates (**5593–5598**) and calls `AddLocalDamageEffect` (**5600**). That function (**5617**) chooses material sounds (**5638–5645**), splats (**5648–5655**), wound overlays (**5658–5667**) and optional bleeding particles (**5671–5687**). `UpdateDamageEffects` (**5695**) transforms the saved point through the current animated joint (**5719–5723**), keeping effects attached to moving body parts.

### Multiplayer boundary recorded, not fully audited

Predicted projectiles can skip replication (`Event_LaunchProjectiles`, `Weapon.cpp:3629–3634`). `idPlayer::Damage`, `Player.cpp:8490–8495`, avoids predictive damage feedback on the local victim; server health processing is at **8537–8540**, while a local client attacker can send a reliable hit message (**8541–8560**). `idGameLocal::ServerProcessReliableMessage`, `Game_network.cpp` **493**, handles `GAME_RELIABLE_MESSAGE_CLIENT_HITSCAN_HIT` at **561**: reads metadata (**562–568**), traces to the reported victim joint (**593–604**), rejects an intervening object (**606–608**) and invokes damage (**610–613**). This validation ray is distinct from initial pellet simulation. Full networking/prediction behavior was not a goal of this first task.

## 7. Sounds, muzzle effects and ejection timing

The ordinary `Event_LaunchProjectiles` orders the following after its pellet loop: queue brass once if delay is nonnegative (`Weapon.cpp:3700–3702`), add flash unless the light is continuously on (**3706–3708**), call player feedback (**3711**) and reset smoke start time (**3714**). There is no explicit firing sound or pump-animation call in this routine. Those can be scheduled by script/animation data, whose shotgun sequence is missing.

`idWeapon::MuzzleFlashLight`, `Weapon.cpp` **1450**, checks flash enable/radius/joint (**1452–1457**), updates its position (**1460**), sets shader time/diversity (**1463–1467**), schedules expiry (**1470**) and creates/updates view/world lights (**1472–1478**). `GetWeaponDef:1083` has a fallback flash time of 0.25 seconds, not a proven shotgun setting. `UpdateFlashPosition` (**1410**) takes the flash joint (**1412**), traces from 16 source units behind to 8 ahead (**1432–1435**), then places the light 8 behind the hit (**1437**) to avoid solid walls. World flash uses its own joint (**1442**).

`PresentWeapon`, `Weapon.cpp:2444–2454`, emits optional smoke from the smoke joint, barrel or camera fallback. It removes expired/hidden flash lights (**2518–2527**) and updates the active attachment (**2530–2534**). `Event_EjectBrass` (**4125**) checks enablement, viewmodel visibility, joint/definition and client status (**4126–4135**), gets the animated eject transform (**4142**), then spawns/launches `idDebris` (**4146–4152**). Its linear velocity is 40 times the sum of player forward/right/up (**4154**) and angular components are random from −10 to 10 (**4155**). These are generic cosmetic debris settings; the shotgun's brass definition and delay remain absent.

Generic sound events are registered in `Entity.cpp:145–147`. `idEntity::Event_StartSound` (**4365–4369**) delegates to sound playback and returns duration in seconds. `StartSound` (**1609**) resolves a `snd_` key from current, potentially mutable spawn arguments (**1619–1626**), runs only on new frames (**1629–1631**) and resolves the sound declaration (**1634–1635**). Those arguments originate from definition data but need not remain immutable.

Animation frame commands provide another scheduling route. `idAnim::AddFrameCommand`, `anim/Anim_Blend.cpp` **282**, validates frames and converts 1-based declarations to 0-based indices (**292–297**); it supports script/global/object/engine events (**304–331**) and sound cues. `idDeclModelDef::ParseAnim` (**2462**) loads commands at **2610**. `idAnimatedEntity::UpdateAnimation`, `Entity.cpp` **5448, 5460–5462**, services crossed animation times when visible. `idAnimator::ServiceAnims`, `anim/Anim_Blend.cpp` **4164, 4174–4176**, calls active blends; `idAnimBlend::CallFrameCommands` (**1762, 1784–1798**) maps times to frames and handles initial-frame events. `idAnim::CallFrameCommands` (**723, 731–753**) dispatches commands, including generic and weapon sound channels (**758, 763, 824, 829**). Mechanical sounds can therefore follow authored frames, but the actual shotgun cues are unknown.

**Important distinction:** the `muzzle_flash` animation command parsed at `anim/Anim_Blend.cpp:528–536` dispatches `AI_MuzzleFlash` at **912**. It is defined in `ai/AI_events.cpp:48` and registered to `idAI` at **186**. That is not evidence that the player's shotgun flash or pellets use this AI command; the confirmed player flash call is `Weapon.cpp:3708`.

## 8. Firing sequence: confirmed engine path, missing retail decisions

```mermaid
sequenceDiagram
    participant Input as Player input
    participant Player as idPlayer
    participant Weapon as idWeapon
    participant Script as Weapon script
    participant Anim as Animator and sound
    participant Pellet as idProjectile / physics
    participant Target as Target Damage / Pain
    participant View as Weapon and camera effects
    Input->>Player: Attack held
    Player->>Weapon: FireWeapon -> BeginAttack
    Weapon->>Script: Linked attack flag; UpdateScript
    Note over Script: Retail shotgun script is missing
    opt Script or model data schedules animation and sound
        Script->>Anim: Named clip / sound request
        Anim->>Anim: Blend bones; service crossed frame commands
    end
    Note over Script,Anim: Ordering relative to launch is not established
    opt Script calls ordinary launchProjectiles
        Script->>Weapon: Count, spread, fuse offset, launch power, damage power
        Weapon->>Weapon: Validate; spend ammo once; alert AI and update shaders
        Weapon->>Weapon: Choose origin and view-axis aim; extend bounded model kick
        loop Each requested pellet
            Weapon->>Weapon: Sample independent spread direction
            Weapon->>Pellet: Create; safe muzzle placement; Launch
            opt Collision during launch or later physics
                Pellet->>Target: Per-pellet Damage and impact effects
                Target->>Target: Target rules and attacker feedback; health; pain/death
            end
        end
        Weapon->>View: Schedule brass; flash; player feedback; smoke timestamp
    end
    View->>View: Model linear recovery; separate gated quadratic camera kick
    Note over Pellet,View: Impacts can occur before or after effects; no universal fixed order
```

This diagram documents the ordinary native event when invoked. It does not certify that the retail shotgun uses it instead of the ellipse event, or prescribe the missing script's fire/pump/reload ordering. Animation and impact branches are conditional, not proof of a single retail chronology.

## 9. Proposed original Godot 4.7.2 weapon controller

Everything in this section is a **new design recommendation**, not a claim about Doom implementation or retail tuning. It targets the requested Godot version; no Godot project was edited and no API/runtime behavior was tested. The design uses original data, animations, sounds and assets. The source research informs the separation of responsibilities; it is not a source-code port.

### Ownership and one accepted-shot contract

| Component | Owns | Produces or consumes |
|---|---|---|
| Input adapter | Trigger edges/held state, reload and switch requests | Requests only; no damage or effects |
| Weapon controller | Equipped weapon, ammunition/chamber, legal state transitions, cadence and shot IDs | Emits one accepted-shot record for each legal shot |
| Shot resolver | Canonical aim snapshot, spread sampling, collision queries or projectile creation | Hit records; never edits a visual recoil transform |
| Weapon recoil presenter | Local position/rotation kick and recovery | Presentation transform above animated hands/weapon |
| Camera recoil controller | Temporary recoverable aim offset and its limits | Camera/aim basis; independent settings from gun recoil |
| Animation coordinator | Fire, pump, reload, raise/lower and interruption transitions | AnimationTree/AnimationPlayer requests and mechanical presentation cues |
| Effects presenter | Firing/mechanical audio, muzzle flash, smoke and ejected cases | Consumes shot IDs/state events; uses authored sockets |
| Target damage receiver | Health, armor, hit zones, modifiers and death | Validated damage result |
| Target response presenter | Pain sound, flinch cooldown, impact material, decals and physical reaction | Consumes damage/impact results separately from health calculation |
| WeaponDefinition Resource | Read-only original tuning and asset references | Shared configuration; runtime ammo/state live elsewhere |

The controller should process gameplay decisions in `_physics_process`. Input can be collected earlier, but each request is consumed once at a physics boundary. Use trigger-edge policy for a semi-automatic weapon and held-trigger cadence for an automatic weapon. A configurable, bounded input buffer can preserve a press just before a pump finishes; it must not create extra shots when the trigger remains held in semi-automatic mode.

For each legal shot, the controller checks its chamber/ammo and state, reserves exactly one ammunition/cadence change, assigns a monotonically increasing shot ID, then snapshots aim, launch position, weapon identity and a spread seed. The resolver and all presentation consumers receive this same record. Apply the new shot's recoil **after** that aim snapshot so it affects subsequent shots, not its own direction retroactively. A failed readiness check emits no accepted-shot record. Define catch-up policy explicitly: cap the number of shots processed after a stalled frame instead of silently emitting an unlimited burst.

No sound callback, particle callback or animation blend should independently consume ammo, generate another shot or apply damage. Duplicate delivery of a shot ID must not repeat gameplay. A future multiplayer implementation could validate these records on an authority; networking is not part of this proposed first controller.

### Transform hierarchy and aim policy

Use a hierarchy conceptually like this; this is a scene design, not an implemented scene file:

```text
PlayerBody
└── AimRoot                         input yaw/pitch and eye position
    └── CameraRecoilPivot            recoverable camera recoil, applied once
        ├── Camera3D
        ├── ShotOrigin3D             stable gameplay origin near the gun
        └── WeaponBase              authored first-person placement
            └── MovementSway        movement bob/lag only
                └── WeaponRecoil    cosmetic translation/rotation only
                    └── AnimatedRig hands, weapon and mechanical bones
                        └── Sockets muzzle, eject, grip, etc.
```

**Chosen aim policy:** input aim plus the recoverable camera offset defines subsequent shot direction. Cosmetic gun kick, pump motion and movement sway do not rotate that aim. They move only the presentation branch. The gun naturally follows camera recoil once through the hierarchy; do not add the same camera angle again in its local recoil calculation. Optional decorative camera shake should remain a separately defined presentation effect, excluded from the gameplay aim snapshot.

The stable gameplay launch marker is intentionally independent of animated muzzle motion; it prevents a pump/fire pose from unpredictably changing collision origin. Use the animated muzzle socket for visual effects, and a close-wall check for gameplay obstruction. Tune the stable marker to the original weapon's dimensions. If a later design chooses a truly animated launch origin, make that an explicit weapon option and validate every animated pose against cover.

Use `Skeleton3D` with authored mechanical/hand tracks and `BoneAttachment3D`/`Marker3D` sockets where appropriate. Animation owns rig bones; procedural sway and recoil own the parent nodes. Any grip IK owns only its designated bones after animation evaluation. Each transform has one final writer, avoiding animation and recoil scripts overwriting each other. A third-person/world gun should have its own attachment and presentation, while sharing accepted-shot and mechanism events.

### Recoil recovery with independent amplitude and duration

For an initial implementation, use **time-evaluated normalized response curves**, independently for camera and weapon. On a shot, store the shot time and original position/angle impulse. At presentation time, evaluate each active contribution by elapsed time divided by its configured recovery duration. A curve goes from 1 at onset to 0 at completion; multiply it by the shot's amplitude. Sum active contributions with separate translation and angle limits, then remove expired contributions. Evaluate from elapsed seconds, rather than repeatedly interpolating by a fixed per-frame percentage.

This provides a controllable snap followed by recovery, permits repeated-shot accumulation within limits, and lets recovery duration change without implicitly squaring the starting amplitude. Optional rise time can produce a short authored kick-in before recovery; the same elapsed-time rule applies. Camera and gun use different curves, amplitudes and caps. A spring could be substituted later if playtesting justifies it, but is not required for the initial design.

Advance the gameplay camera-recoil state on the fixed gameplay clock because it affects future aim. Interpolate the rendered state for smooth display. Pause/time scaling should use the same chosen gameplay clock; pausing should freeze recovery. Cosmetic weapon curves can be evaluated for the interpolated presentation time from the same shot timestamps. Keep input look angles separate so temporary recoil recovery does not erase the player's mouse movement.

Expose camera intensity, visual gun intensity and hand-animation intensity independently. Zero camera recoil must leave weapon kick and gameplay functional; if camera intensity also changes aim consequences under the selected policy, document that as a gameplay setting rather than silently labeling it cosmetic.

### Firing, pump and reload state timing

For an original pump shotgun, use explicit states such as Lowered, Raising, Ready, Firing, Cycling, ReloadStart, ReloadInsert, ReloadEnd and Lowering. A chamber flag and ammo count determine readiness. After an accepted shot the chamber becomes empty and the controller enters the cycle sequence; only its defined completion can restore readiness. The exact cycle duration, shell handling and reload interruption rules are our design choices, not established Doom facts.

Store semantic timing markers in the weapon's own action data: shot accepted, eject case, insert shell, chamber ready and action complete. The controller evaluates gameplay markers once as its state time crosses them. Animation is synchronized to that timeline, including playback speed where required. Cosmetic animation callbacks may request a sound/effect, but must carry the action ID and be deduplicated. This prevents blended, skipped or replayed animation frames from duplicating ammunition or damage.

Define interruption rules explicitly. For example, a per-shell reload can accept a fire request after a shell has been inserted, finish the required closing/chambering stage, then fire when Ready. Cancelling before insertion must not award a shell. Switching weapons must resolve/cancel pending action markers according to the same rules. Avoid an animation-only “ready” flag that changes merely because a visual clip was interrupted.

### Sound and muzzle effects

An accepted shot starts its firing sound, visual recoil, fire animation request and muzzle-effect timeline. A separate cycle/eject/insert event starts pump, case or reload sounds. Effects use named animated sockets but never decide whether the shot was legal. Pool short-lived particles, lights and case objects as appropriate; set expiry in seconds so a flash does not last a different duration at different frame rates.

Use original `AudioStreamPlayer3D` cues or an appropriate first-person audio mix, `GPUParticles3D` smoke/flash where useful and an optional short light pulse. Keep light intensity, duration, particle count, sound variation and mechanical timing in the definition. Let surface-impact effects select material sounds/decals separately. Near-wall presentation can fade/lower the gun or reposition a flash, while the shot resolver's obstruction result remains authoritative.

### Hit detection and damage

For the original shotgun, a practical initial choice is one primary aim ray per pellet plus obstruction checks through `PhysicsDirectSpaceState3D.intersect_ray`, with explicit collision masks, owner exclusions and maximum range. That is our representation choice; the traced BFG path creates projectile entities and runs physics. Slow projectiles can use a separate controller with swept collision per physics step.

Choose the original spread deliberately. For example, uniform disk-area spread uses radial sample `sqrt(u)` and random azimuth, scaled by `tan(half_angle)` in the aim plane before direction normalization. This produces an explicitly defined cone and differs from BFG's sine-of-uniform-radius sampling. Save the seed in the shot record so a repeated simulation/debug replay reproduces the same pellet directions. Do not reuse a random value implicitly across sound, recoil and spread; those presentation choices should not change hits.

First validate clearance from the player's eye/body to the stable gameplay muzzle. If that marker is inside or beyond nearby cover, clamp it to a safe position before the obstruction or reject the blocked launch; a muzzle-to-target query alone would miss a wall already behind that marker. Then aim each pellet from the camera basis to find its intended impact/range endpoint and check the line from the validated muzzle to that point. A closer obstruction wins. Exclude the owner and define treatment of transparent/penetrable materials explicitly. Penetration, damage falloff and ricochet should be absent unless intentionally specified and tuned; do not infer them from a BFG networking flag.

A hit record should contain shot ID, pellet index, attacker/target identity, hit position and normal, direction, surface/zone or bone, proposed damage and physical impulse. The target damage receiver applies its own armor, zone scales, modifiers and health rules once, rejects duplicate hit records using shot ID + pellet index + target, and returns the actual result. Cosmetic responses can instead group by shot ID + target. If distance falloff is desired, specify its curve and range units in our data.

Pellets may all contribute damage, but group response presentation by shot/target so one shell does not restart a flinch or play the same pain sound many times. Select a representative hit or strongest response; apply death exactly once. Keep health reduction, pain animation cooldown, push impulse, blood/decal placement and death/ragdoll as separate decisions. A wound that must follow a moving limb should be stored in that limb's local coordinates or attached through a bone socket.

### Tuning data and units

| Group | Original fields to expose | Units / rule |
|---|---|---|
| Input/cadence | Trigger mode, interval or RPM, input-buffer duration | Convert RPM to seconds once; use one cadence authority |
| Ammo/mechanism | Capacity, ammo cost, chamber policy, reload/cycle markers | Counts and seconds; runtime state is not in a shared Resource |
| Visual recoil | Position impulse, pitch/yaw/roll impulse, rise/recovery curve and duration, accumulation caps | Metres, degrees for authoring, seconds; convert angular API values at a clear boundary |
| Camera recoil | Pitch/yaw impulse, optional bounded variation, recovery and limits | Independent of visual recoil; aim policy documented |
| Movement presentation | Base pose, sway/bob amplitudes and recovery | Metres/degrees/seconds; no effect on pellet direction |
| Animation | Original clips, transitions, speed and semantic markers | Timeline in seconds, not rendering-frame counts |
| Pellet query | Count, cone half-angle, distribution, range and masks | Integer, degrees, metres; explicit seed |
| Damage/response | Per-pellet base, optional range curve, target zone rules, push and reaction cooldown | Original gameplay units; target owns final health calculation |
| Effects/audio | Socket names, own resources, duration, light strength, variation | Presentation only; deduplicate by shot/action ID |

Future validation criteria, **not tests claimed to have run**: equivalent recovery at different rendering frame rates; identical pellet directions for a given seed; one ammo cost and one accepted-shot event; no shots while an uninterruptible mechanism stage is active; no free shell on cancelled insertion; no damage from replayed animation callbacks; close-wall obstruction; camera intensity zero with functioning gun presentation; and one death event despite multiple pellet hits. Runtime tests should be written when an implementation exists and can actually exercise these behaviors.

## 10. Remaining evidence gaps and the next useful task

The missing retail data prevents an exact retail shotgun specification: launch-event choice, pellet count and spread, ammo/cadence rules, projectile class/speed/fuse, damage definition, recoil parameters, sound cues, pump/reload/ejection markers, animation curves and single-player/multiplayer overrides. Specific monster reactions also depend on absent AI scripts and model data. Clock/prediction edge cases and actual runtime behavior were recorded as limits, not claimed verified by execution.

**The user has approved Task 2: local read-only inspection of their legally owned Doom 3 BFG retail shotgun script plus its referenced weapon, projectile and damage definitions.** Start with `script/weapon_shotgun.script`, `script/weapon_base.script`, `weapon_shotgun`, `projectile_bullet_shotgun` and the referenced `def_damage`; then record relevant animation declarations/timing markers. Record factual values and state sequences with local provenance. Do not upload licensed assets or full retail scripts, fetch non-BFG substitutes, or implement Shooter 1946 as part of that inspection.

## 11. Task 2 availability checkpoint — 2026-10-09

**PARTIAL / BLOCKED: authorization is present; retail files have not been located.** Task 2 began by inspecting the saved handoff, Git history and available directories. The existing Task 1 checkpoint `2002d9b73746a01654cb7068ffdaf560566d4165` was still the head of the clean research branch and GitHub PR #1. The other two repositories were clean. No completed weapon-mechanics research was repeated.

A filename inventory including hidden files, excluding Git internals, searched `/workspace`, `/mnt` and `/media` for `.resources`, `.pk4`, `.script`, `.def`, `.md5anim`, `.md5mesh`, common archive files and shotgun names. It found no BFG retail candidates. The `.def` result was the already-known build export file, not game data. `/mnt`, `/media`, `/workspace/library-files`, `/workspace/scratch` and `/workspace/shared/downloads` were empty. Conventional Steam/game directories checked under `/home/agent` were absent. This is a bounded search result, not a claim that every possible filesystem location was examined. The user has been asked for the installation or mounted-data path; no replacement data was downloaded.

### Container and effective-value checks needed before reading retail numbers

These additional findings are from the same pinned public source, not from retail files:

- **Include `.resources` containers in the inventory.** `idFileSystemLocal::AddGameDirectory`, `neo/framework/FileSystem.cpp` **2433**, enumerates that extension (**2454**), sorts the results (**2455**) and loads/appends containers (**2461–2462**). A search for PK4 files alone would be insufficient. `idResourceContainer::Init`, `neo/framework/File_Resource.cpp` **55**, specifically handles `_ordered.resources` (**57–60**) and reads a custom indexed header/table (**68–91**). A ZIP-only inspection is not a valid test for contained scripts.
- **Resolve which copy of a virtual file wins.** `idFileSystemLocal::GetResourceCacheEntry`, `neo/framework/FileSystem.cpp` **2688**, normalizes names (**2698–2699**) and searches loaded containers backward (**2700–2713**). `fs_resourceLoadPriority` defaults to 1 (**290**); `OpenFileReadFlags` (**2780**) then checks resource files before loose paths (**2813–2817, 2823–2832**). Setting it to zero moves resource lookup after loose paths (**2926–2929**). A loose script by itself need not be the active version.
- **Follow inherited entity definitions.** `idDeclEntityDef::Parse`, `neo/framework/DeclEntityDef.cpp` **56**, gathers `inherit*` references (**100–115**) and applies them with `SetDefaults` (**118–120**). `idDict::SetDefaults`, `neo/idlib/Dict.cpp` **179–191**, fills only missing keys. Explicit child values therefore win; among inherited defaults, the first applied value fills a missing key before later parents. Record the source of each effective value.
- **Do not merge every similarly named declaration.** `idDeclFile::LoadAndParse`, `neo/framework/DeclManager.cpp` **613**, warns/skips a declaration already defined in another file (**735–738**). Declaration duplication differs from replacing the same virtual file in another container. Actual installed files and load order are required to resolve the outcome.

### Resume at the missing-data boundary

The next input needed is the user's BFG installation or extracted-data location accessible to this environment. Once available, inventory only that location, establish edition/build and mod/override provenance, and inspect the shotgun dependency chain read-only. Record each fact with virtual path, containing file, local hash/build provenance, declaration or function and exact lines. Separate literal values, inherited values and calculations. Record state transitions and timing units, then reconcile them with the confirmed native functions above.

No retail pellet count, spread, damage, recoil amount or pump/reload timing was obtained in this checkpoint. Approval to perform Task 2 already exists; a continuing AI should ask only for missing access/location information, not repeat an approval request. No licensed content should be committed or uploaded, and no implementation is authorized.

## Checkpoint and validation record

The initial source findings were pushed as `266cc83679677427ec442f8d3972a091bd378c19`. After interruption, the working tree was clean and three completed scratch notes survived; those were preserved in pushed checkpoint `a40eb56954cb2cffe152de1ef0f11053e14b5d7a` before consolidation. That commit retains the original recovered notes in history; this report incorporates their findings with reviewed corrections, including mutable damage feedback, ellipse view-axis aiming, the physics function reference, client guards and sound lookup.

A read-only second review checked the principal recoil equations, damage/reaction paths, animation frame-command distinction and cited source locations against the pinned source. No source files changed. Validation is static inspection, reference checking and documentation/Git checks, not a game build or runtime test. Publication and exact continuation instructions are recorded in [HANDOFF.md](HANDOFF.md). The fork-only pull request is [korpus91/DOOM-3-BFG-Source#1](https://github.com/korpus91/DOOM-3-BFG-Source/pull/1); inspect its current head for the final document commit.
