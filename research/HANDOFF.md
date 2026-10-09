# Task 1 research handoff

**Status: source investigation and original Godot design complete; exact retail shotgun reconstruction PARTIAL because retail data is absent.** Task 2 is not authorized. Do not start implementation or further retail-data research without the user's approval.

## Objective and exact source identity

Research Doom 3 BFG Edition (2012) weapon mechanics beginning with the shotgun, for an original Godot 4.7.2 controller for Shooter 1946. Explain the actual C++ execution, cite source paths/functions/exact lines, separate missing retail data, and propose an original design without implementing it.

- Official repository: `https://github.com/id-Software/DOOM-3-BFG`, source branch `master`.
- User fork and **exclusive** pull-request target: `https://github.com/korpus91/DOOM-3-BFG-Source`, base `master`.
- Exact source base: `1caba1979589971b5ed44e315d9ead30b278d8b4`.
- Checkout: `/workspace/DOOM-3-BFG-Source`.
- Dedicated research branch: `research/doom-bfg-weapons-task1`.
- Required `neo/d3xp/Weapon.cpp` and `Weapon.h` exist and were compared byte-for-byte to official raw files at that SHA. Both master refs matched it. GitHub API confirms the fork parent. This is BFG; `doomclassic` and non-BFG Doom 3 were not substituted.
- Task began 2026-10-09 02:27 UTC, paused after a usage interruption, and resumed at approximately 15:45 UTC. The user's 20–30 minute timebox applies to active work; do not mistake the long interruption for active research time. Do not expand this task to use additional budget.

## Last verified checkpoint and recovery

1. Before interruption, commit `266cc83679677427ec442f8d3972a091bd378c19` was pushed to the research branch. On resume, native Git again verified that remote SHA and the working tree was clean. This was the last completed, verified action from the interrupted research pass.
2. Three completed scratch notes survived at `/tmp/bfg-recoil-findings.md`, `/tmp/bfg-firing-findings.md` and `/tmp/bfg-projectile-findings.md`. No uncommitted repository changes were lost. Their findings had not yet been incorporated into the first GitHub checkpoint.
3. The recovered notes were preserved in the report and pushed as `a40eb56954cb2cffe152de1ef0f11053e14b5d7a`; the remote branch SHA was verified. The original raw notes therefore remain available in Git history even after consolidation.
4. GitHub API access now works. The previous CONNECT 403 issue is resolved; do not repeat network onboarding or ask for another token. The fork-only PR was successfully created: **https://github.com/korpus91/DOOM-3-BFG-Source/pull/1**.
5. The complete report/design was committed and pushed as **`4ae927c72f31c06741db812bc213974774a733ac`**. Native Git and the GitHub commit API both verified this exact SHA. The API confirmed that its changes are only the two research Markdown files.
6. PR #1 is **open and ready for review**, not merged. GitHub verified base repository `korpus91/DOOM-3-BFG-Source`, base branch `master`, head repository the same fork, head branch `research/doom-bfg-weapons-task1`, and exactly the two intended changed files. Its description records the completed scope and retail-data limitations. This final handoff update records those successful checks; use the branch/PR tip for any subsequent bookkeeping commit rather than assuming the full-report SHA is still the newest commit.

## Completed research and deliverables

`research/DOOM_BFG_WEAPON_RESEARCH.md` contains:

- Official-source provenance, hashes, immutable source links and an exact-line reference convention.
- Shotgun-specific names, expected retail script paths and the trigger-joint muzzle-orientation workaround.
- Weapon selection, input flags, `GetWeaponDef`, script construction/state transitions and per-weapon customization.
- `PresentWeapon`, movement bob/turning/acceleration lag, skeleton interpolation, hand/mechanical animation boundaries and world-model attachment.
- `MuzzleRise`: capped accumulated visual kick time and linear model return; independent camera feedback with a shared active-kick gate and quadratic time-based decay.
- Camera/aim/weapon transform relationships and recorded clock/prediction limitations.
- Ordinary/ellipse spread math, camera-based launch direction, safe launch origin, ammo once per event and a projectile physics/collision trace.
- Per-pellet damage, target-specific scaling/armor/location rules, mutable attacker feedback, pain gates, AI script flags, death and impact effects.
- Script/animation sound routes, flash/light positioning, smoke, brass scheduling and the distinction between AI frame commands and player muzzle flash.
- A Mermaid firing sequence that labels the missing retail-script boundary and allows impacts during launch or later physics.
- An original Godot 4.7.2 design: component ownership, accepted-shot contract, hierarchy and aim policy, separate recoil recovery, animation/action timelines, effects, ray/projectile choices, damage/response contracts and tuning units. No implementation.

A second read-only review checked the core source claims. Corrections incorporated: attacker DamageFeedback can modify damage before subtraction; ellipse direction also uses playerViewAxis; idEntity::RunPhysics begins at Entity.cpp:2603, with its evaluation call at2645; immediate fuse-zero physics is server/local-prediction guarded; sound lookup uses mutable spawnArgs. These details supersede imprecise wording in the raw recovered notes. No source file was edited.

## Continuation instructions: publication already verified

Do not repeat source verification, completed research, PR creation or the ready-for-review action. Source review, 54 source path/line-bound checks, two balanced diagram fences and `git diff --check` passed. Comparison with source base and GitHub's PR-file API found only the two intended documents. Shooter 1946 and gamedev-studio working trees remained clean. No game or Godot runtime tests were run.

1. If this session was interrupted again before final delivery, inspect local status/log and PR head. The full report is already saved at the verified commit above. Commit/push only a still-unpublished final handoff update if one exists; otherwise make no duplicate checkpoint.
2. Verify the current remote SHA if making any new persistence claim, then deliver the actual commit/PR link and limitations. Keep this PR unmerged unless separately instructed.
3. After delivery, stop. Await explicit approval for the single next research task described below; no continuation task is enabled or scheduled.

Tool note for a future necessary PR edit: the preinstalled `gh pr edit` encountered a deprecated Projects-classic GraphQL query. Updating the body through `gh api --method PATCH repos/korpus91/DOOM-3-BFG-Source/pulls/1 --field body=@/tmp/bfg-research-pr-body.md` succeeded. `gh pr ready` also succeeded. No tool installation or replacement was needed.

## Unresolved questions and single next research task

The repository expressly excludes game data (README.txt:15). The only tracked `.def` is a build export definition; retail scripts, shotgun model/animation files and PK4 archives are absent. Consequently exact retail count/spread/damage, launch event/class, recoil magnitudes, fire/pump/reload/ejection timing, sounds and multiplayer overrides remain unknown. Specific monster script reactions and exhaustive slow-motion/prediction edge cases also remain outside the completed evidence.

**After new approval**, the most useful next task is read-only inspection of locally available, legally owned BFG retail `script/weapon_shotgun.script` and `script/weapon_base.script`, plus `weapon_shotgun`, `projectile_bullet_shotgun`, their referenced damage definitions and relevant animation declarations. Record factual values/state transitions with exact local provenance. Do not publish full licensed scripts or assets. Do not use non-BFG data to fill gaps. This next task has not begun and is not scheduled autonomously.

## Constraints for every continuing AI

- Keep all existing work. No reset, reinstall, repeated completed research or environment restart.
- Do not change Shooter 1946 or gamedev-studio. Only the two research documents belong in this PR.
- Do not install tools, compile/run the game, execute unknown game files, upload licensed assets, purchase services or enable autonomous follow-up tasks.
- Do not copy Doom source or game assets into the original Godot project.
- Keep confirmed code, missing retail data and proposed original behavior distinct. Godot recommendations have not been runtime-tested.
- Do not claim a specific retail shotgun launch event, pellet count, damage or timing without the absent data.
- Never push to or open a pull request against upstream `id-Software/DOOM-3-BFG`.
- Save useful changes frequently, preferably about every ten active minutes. If interrupted/short on budget, checkpoint first and report PARTIAL honestly.
- No Task 2 or implementation without approval.
