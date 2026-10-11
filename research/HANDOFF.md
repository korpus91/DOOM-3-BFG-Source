# Latest verified state — 2026-10-10

Data04 extraction is complete: 267/267 files passed original CRC32/size validation. Data01–03 payloads remain pending; all four indexes are decoded. The user expanded scope to all guns. Read the self-contained Claude handoff at the end of this file. No extraction process is running.

# Doom BFG weapon research handoff

**Historical status before package extraction, 2026-10-10: Task 1 and Task 2 source-only research complete; retail shotgun reconstruction PARTIAL / BLOCKED on unpacked BFG data. PANZERV local Windows access is verified. The user-identified D:\Games location contains BFG installer packages, not an accessible installed base folder; existing 7-Zip cannot list them.** The user explicitly authorized continuation, committing/pushing progress and updating existing PR #1, while preserving all work and leaving Shooter 1946 untouched. No approval is missing. The earlier “Go ahead with task 2” instruction and checkpoints below are historical context, not a pending approval or instruction to restart.

## Latest local checkpoint — 2026-10-10

- Local shell verified `PANZERV` / DNS `PanzerV`, starting in `C:\Users\Korpus\Workbench`; Windows filesystem is readable. Historical cloud-access restrictions below no longer describe this session.
- Existing research branch cloned with authorization into `C:\Users\Korpus\Workbench\Projects\Code\2026-10-10-DOOM-3-BFG-Source`, starting from live Git/PR head `133061c97717523a9b055f7916cda3a187182ee6`. No reset, new research branch, history rewrite or new PR. Only the two research documents changed.
- Report section 13 records the actual local search and packaging evidence. Steam's three registered libraries contain no `appmanifest_208200.acf`. User clarified that files are in `D:\Games`; the BFG candidate there contains four `.dxn` files and `Setup.exe`, with no subdirectories/base folder. The separate DOOM 3 Collection candidate contains two ISO images of unverified BFG relevance.
- Existing 7-Zip 26.01 cannot statically list either `Data01.dxn` or `Setup.exe` (exit 2). No setup/game execution, installation, mounting or whole-installation extraction was performed. No raw licensed payload was uploaded or committed.
- No retail weapon value was resolved. Active files, declaration inheritance, multiplayer alternatives, animation variants, timings, recoil, sounds and effects remain unknown. Installed build and override conditions are also unverified.
- **Next step:** identify an accessible unpacked BFG `base` folder or an already available compatible read-only package reader. Keep the no-install/no-execution constraints. Resume at section 12's dependency table; do not restart source research or re-run the same directory searches. This is a data-format/access blocker, not missing research approval.
- Publish this factual checkpoint to the same branch and existing fork-only PR #1; leave it open/unmerged. Verify live branch/PR SHA and both remote document blobs before claiming upload completion. Use the live tip rather than historical SHA notes below.

## Objective and exact source identity

Research Doom 3 BFG Edition (2012) weapon mechanics beginning with the shotgun, for an original Godot 4.7.2 controller for Shooter 1946. Explain the actual C++ execution, cite source paths/functions/exact lines, separate missing retail data, and propose an original design without implementing it.

- Official repository: `https://github.com/id-Software/DOOM-3-BFG`, source branch `master`.
- User fork and **exclusive** pull-request target: `https://github.com/korpus91/DOOM-3-BFG-Source`, base `master`.
- Exact source base: `1caba1979589971b5ed44e315d9ead30b278d8b4`.
- Checkout: `/workspace/DOOM-3-BFG-Source`.
- Dedicated research branch: `research/doom-bfg-weapons-task1`.
- Required `neo/d3xp/Weapon.cpp` and `Weapon.h` exist and were compared byte-for-byte to official raw files at that SHA. Both master refs matched it. GitHub API confirms the fork parent. This is BFG; `doomclassic` and non-BFG Doom 3 were not substituted.
- Task began 2026-10-09 02:27 UTC, paused after a usage interruption, and resumed at approximately 15:45 UTC. The user's 20–30 minute timebox applies to active work; do not mistake the long interruption for active research time. Do not expand this task to use additional budget.

## Current recovery and continuation checkpoint — 2026-10-10

- Exact resume head: **`7ae6c7dc0e22ebe66c1da394c21d723f023894a9`**, verified in PR #1 and fetched from the existing fork. Task 2 had stopped after the no-retail-files availability check and section 11's resource precedence/entity-inheritance findings. No retail script, definition or animation had been inspected.
- The current local checkout initially contained only clean branch `work` at `1caba1979589971b5ed44e315d9ead30b278d8b4`; no untracked/ignored repository files or stash existed. Its reflog showed environment setup returning to the source base. Restored the existing remote research branch in the same checkout using fetch and switch, preserving all five research ancestors. No reset, new repository, reinstall or environment creation was performed by this continuation.
- The historical `/tmp/bfg-recoil-findings.md`, `/tmp/bfg-firing-findings.md` and `/tmp/bfg-projectile-findings.md` were absent in this session. Their previously preserved text remains in commit `a40eb56954cb2cffe152de1ef0f11053e14b5d7a`; the consolidated report remains intact. This cannot certify the contents of any unsaved memory from a previous session.
- User confirmed retail location: **Windows PC PANZERV**. This cloud's network snapshot has `vpn_configured: false` and no TCP destinations configured; no filesystem connection to PANZERV was supplied. Do not pretend a Windows path is readable here. Do not repeat the earlier archive/directory search.
- New verified findings are in report **section 12**: script wait/getTime clock differences; manual-thread wait semantics; shared weapon animation-end/blend fields; `_mp` declaration selection; reload inventory transfer; animation aliases/inheritance/variants; text clip length versus blend frames; binary `.bMD5anim` and generated mesh loading; frame-command timing interpretation; and exact local-file/provenance requirements. Task 1's investigation/design was preserved rather than repeated.
- **Next action for the user:** make the installed BFG `base` folder on PANZERV accessible read-only to a local research session or explicitly connected filesystem. Example only: `<Steam library>\steamapps\common\DOOM 3 BFG Edition\base`. It must include containers/subdirectories; do not request that licensed assets be uploaded to GitHub or this chat. Read report section 12's dependency table before narrowing the request. Additional access is limited to identified active overrides and build metadata.
- **Exact next research step after access:** inspect only that installation's resource indexes and active loose overrides; resolve the two weapon scripts and referenced declarations/models/animation binaries; record the actual state/launch/timing and recoil facts with provenance. No research remains blocked on permission, installation of tools or another C++ overview.
- **Still unknown:** retail launch event/count/spread, effective damage/projectile tuning, recoil values, shotgun state transitions, fire/pump/reload/ejection timings, clip variants/curves, sound cues and installed multiplayer overrides. Source rules are verified; retail behavior and runtime observations are not.
- Publication is to the same branch and **fork-only PR #1**, unmerged. Use the live Git/PR tip for the latest commit rather than any historical full-report SHA below. Only the two research documents may change.

### Verified publication of the continuation

The completed source-only Task 2 continuation was committed and pushed as **`43564530ff53435666855dbb30b6a0d5dbd2eec3`**. Native Git `ls-remote` and GitHub's PR metadata independently confirmed that head. PR #1's title/body were updated to cover the final Task 1/Task 2 scope and the explicit PANZERV blocker; GitHub confirmed it is open, ready for review, unmerged, based on this fork's `master`, with only the two research documents changed. The working tree was clean after that push.

Validation passed: 31 new source citation path/range groups, manual review of new timing/data-loading claims, balanced document fences, `git diff --check`, preservation of every prior research ancestor, and byte-for-byte preservation of the original source-identity section and report sections 1–11. No game or Godot runtime tests were run. This publication note is a subsequent bookkeeping change; the latest saved SHA is the current research branch/PR tip, not necessarily `4356453`. Before a final response, verify that note is also pushed and that both remote document blobs match the local committed files. If interrupted, continue with that verification only; do not repeat the completed source research.

## Historical Task 1 checkpoint and recovery

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

## Historical Task 1 publication instructions

Do not repeat source verification, completed research, PR creation or the ready-for-review action. Source review, 54 source path/line-bound checks, two balanced diagram fences and `git diff --check` passed. Comparison with source base and GitHub's PR-file API found only the two intended documents. Shooter 1946 and gamedev-studio working trees remained clean. No game or Godot runtime tests were run.

1. If this session was interrupted again before final delivery, inspect local status/log and PR head. The full report is already saved at the verified commit above. Commit/push only a still-unpublished final handoff update if one exists; otherwise make no duplicate checkpoint.
2. Verify the current remote SHA if making any new persistence claim, then deliver the actual commit/PR link and limitations. Keep this PR unmerged unless separately instructed.
3. The user subsequently approved Task 2. Continue at the missing-data boundary described below; no autonomous follow-up task is enabled or scheduled.

Tool note for a future necessary PR edit: the preinstalled `gh pr edit` encountered a deprecated Projects-classic GraphQL query. Updating the body through `gh api --method PATCH repos/korpus91/DOOM-3-BFG-Source/pulls/1 --field body=@/tmp/bfg-research-pr-body.md` succeeded. `gh pr ready` also succeeded. No tool installation or replacement was needed.

## Unresolved questions and single next research task

The repository expressly excludes game data (README.txt:15). The only tracked `.def` is a build export definition; retail scripts, shotgun model/animation files and PK4 archives are absent. Consequently exact retail count/spread/damage, launch event/class, recoil magnitudes, fire/pump/reload/ejection timing, sounds and multiplayer overrides remain unknown. Specific monster script reactions and exhaustive slow-motion/prediction edge cases also remain outside the completed evidence.

**Task 2 is now approved:** read-only inspection of locally available, legally owned BFG retail `script/weapon_shotgun.script` and `script/weapon_base.script`, plus `weapon_shotgun`, `projectile_bullet_shotgun`, their referenced damage definitions and relevant animation declarations. Record factual values/state transitions with exact local provenance. Do not publish full licensed scripts or assets. Do not use non-BFG data to fill gaps.

## Historical Task 2 checkpoint — 2026-10-09

Task 2 started on 2026-10-09 by inspecting the existing handoff, clean working trees and GitHub PR. The last verified Task 1 head was `2002d9b73746a01654cb7068ffdaf560566d4165`. Preserve it and all prior findings; continue on the existing research branch and fork-only PR, changing only the two research documents.

Completed availability check: no retail candidates were found by a filename search covering `/workspace`, `/mnt` and `/media`, including `.resources`, `.pk4`, scripts, definitions, model/animation files and archive/shotgun names. The checked upload/shared directories were empty, and conventional Steam/game paths under `/home/agent` were absent. Do not repeat that same search unless the user identifies a new location or makes data available. This was a bounded inventory, not a full-machine assertion.

The report's new section 11 records source-confirmed BFG `.resources` handling, file precedence and entity-definition inheritance. Those checks help avoid reading an inactive override or mistaking a missing child key for an absent value. No retail weapon values have been recovered. A text question is pending for the installation/mount path; the blocker is missing files, not missing approval or a network policy rejection.

1. Read any new user response for the BFG installation/data path. If it exists only on the user's own computer, explain that this cloud workspace cannot read that filesystem; obtain platform/path information to determine a local read-only workflow. Do not request licensed-asset uploads or install tools.
2. Once data is accessible within the authorized scope, identify edition/build, actual containers or loose files, overrides and provenance. Inspect the shotgun script, its base script and only the referenced weapon/projectile/damage/model declarations. Do not execute scripts or game binaries.
3. Resolve container/file precedence and entity inheritance before reporting effective values. Record factual values and state/timing sequences with exact local paths, hashes and lines. Keep raw licensed content out of Git and do not copy it into Shooter 1946.
4. Reconcile those facts with Task 1's native functions, update these two documents, then commit/push and verify the GitHub SHA and fork-only PR. If files remain unavailable, report PARTIAL/BLOCKED without inventing retail values.
5. Do not implement the Godot controller or start another task without an instruction authorizing it.

## Constraints for every continuing AI

- Keep all existing work. No reset, reinstall, repeated completed research or environment restart.
- Do not change Shooter 1946 or gamedev-studio. Only the two research documents belong in this PR.
- Do not install tools, compile/run the game, execute unknown game files, upload licensed assets, purchase services or enable autonomous follow-up tasks.
- Do not copy Doom source or game assets into the original Godot project.
- Keep confirmed code, missing retail data and proposed original behavior distinct. Godot recommendations have not been runtime-tested.
- Do not claim a specific retail shotgun launch event, pellet count, damage or timing without the absent data.
- Never push to or open a pull request against upstream `id-Software/DOOM-3-BFG`.
- Save useful changes frequently, preferably about every ten active minutes. If interrupted/short on budget, checkpoint first and report PARTIAL honestly.
- Task 2 retail research is approved. Implementation and other tasks remain outside the authorized scope.


## Active extraction checkpoint — 2026-10-10

The user explicitly authorized unpacking every Doom/BFG candidate under `D:\Games` and set an active completion goal. Section 14 now supersedes the previous assumption that no archive inspection route exists: all four BFG FreeArc indexes were decoded with CRC checks (399 files), and the BFG installer metadata plus all 22 embedded files passed SHA1 verification. Bundled archiver components were recovered without running setup. Local extraction is proceeding via a dedicated .NET helper with a larger worker-stack reservation. The original package files are unchanged. Both collection ISOs' Doom-content directories have been extracted locally; nested installers remain separate, and no non-BFG weapon values are used.

Scratch/helper/index/output directory: `C:\Users\Korpus\Workbench\Inbox\2026-10-10-bfg-package-inspection`. Resume the running extraction, validate output CRCs against the package indexes, then inspect the resource indexes and exact shotgun dependencies per sections 11–12. Never execute game or installer binaries/scripts or upload licensed data. Only the two research documents are committed. This is an intermediate factual checkpoint, not completion of retail reconstruction.


---

# Claude handoff — Doom 3 BFG — 2026-10-10

## Start here: confirmed state

You are continuing an existing research task, not starting over. Verify your shell is on **PANZERV** (DNS `PanzerV`) before accessing these paths. A cloud-only Claude session cannot read them. The user now wants all Doom/BFG packages under `D:\Games` unpacked and research expanded from the shotgun to **all guns**. Do extraction and verification yourself; no more approval is required within that scope.

**Data04 is successfully extracted:** 267/267 files, 16,457,224 bytes, every original file CRC32 and size matched; zero missing/bad files. **Data01–03 are indexed but their payloads are not extracted.** No decoder process was running at handoff. No retail weapon values have yet been recovered. Earlier claims that no extraction route exists are superseded.

## Exact locations

**Repository:** `C:\Users\Korpus\Workbench\Projects\Code\2026-10-10-DOOM-3-BFG-Source`

Read `research\HANDOFF.md` and `research\DOOM_BFG_WEAPON_RESEARCH.md`, especially sections **11–12** (precedence, inheritance, multiplayer selection, timing/provenance) and **13–14** (local package investigation). Source code is under this checkout's `neo\`; official source base is `1caba1979589971b5ed44e315d9ead30b278d8b4`. Completed source research must not be repeated.

**GitHub:** https://github.com/korpus91/DOOM-3-BFG-Source — branch `research/doom-bfg-weapons-task1` — existing PR https://github.com/korpus91/DOOM-3-BFG-Source/pull/1. Last independently verified head before saving this handoff: `6cd17351318de4a34a2401f19f08725185ea950c`; both remote documents matched. This handoff is a later checkpoint: fetch the live tip, never roll back to that SHA. Preserve history/work; no new branch/PR, reset, clean, force push, merge, or upstream push.

**Original BFG packages:** `D:\Games\[dixen18] DOOM 3 BFG Edition\Data01.dxn`, `Data02.dxn`, `Data03.dxn`, `Data04.dxn`, `Setup.exe`. Use PowerShell `-LiteralPath` for brackets. Original SHA256 hashes are in report section 13.

**Other original Doom packages:** `D:\Games\DOOM 3 Collection\DVD1.iso` and `DVD2.iso`. These are older Doom III/expansion/mod collections; never substitute their values for BFG.

**All local working copies, helpers and indexes:**
`C:\Users\Korpus\Workbench\Inbox\2026-10-10-bfg-package-inspection`

Within that directory:

| Relative path | Contents / status |
|---|---|
| `bfg-Data04\` | Correct, CRC-verified Data04 extraction. Includes `Doom3BFG.exe`, `goggame-1135892318.info`, `base\default.cfg`, classic music, libraries. No weapon scripts/resource containers in this package. |
| `Data01.dxn.index.json` through `Data04.dxn.index.json` | CRC-validated FreeArc directory indexes: paths, sizes, original file CRCs, compressed block offsets/methods. 399 files total. |
| `Data04-solid.lolz` | Original compressed block, 7,031,713 bytes. |
| `Data04-stage.srep` | Successfully decoded LOLZ output, 15,052,192 bytes. |
| `Data04-srep.arc` | Synthetic scratch wrapper around that SREP stream, 15,059,019 bytes; retains original file sizes/CRCs. Used for successful extraction. |
| `bundled-decoders\` | Six archive-decoder components recovered from Setup.exe, plus local `CLS.ini`. Components' stored SHA1 checks passed. These are unpacking tools, not game executables. |
| `dvd1\` | Extracted Doom folders: `Doom_III_Nightmare`, `Doom_III_Phobos_Anomaly`, `Doom_III_Resurrection_of_Evil`. 20 files / 4,252,992,367 bytes. |
| `dvd2\` | Extracted Doom folders: `Doom_III`, `Doom_III_New_Star_Station`, `Doom_III_Padshiy_Angel`. 12 files / 4,342,431,456 bytes. |
| `unpacked-Data04.dxn\base\default.cfg` | **Failed attempt: zero-byte file. Do not use it.** No need to delete it. |
| `2026-10-10-claude-file-manifest.json` | 337-file snapshot with exact paths/sizes; SHA256 for helpers, decoder components and all Data04 outputs. Created before the last two helpers below, so they are not listed in that snapshot. |

## Working code and extraction route

All filenames below are inside the scratch directory above:

- `2026-10-10-inspect-arc.py`: reads the original four package indexes; checks descriptor/directory CRC32; outputs JSON. It prints long inventories, so redirect stdout if rerunning.
- `2026-10-10-unpack-inno-tools.py`: recovers six decoder components and verifies all 22 embedded files using saved `bfg-inno-payload.bin` and `bfg-inno-entries.json`.
- `2026-10-10-archive-host.cs` and `.exe`: **working x86 .NET extraction host** using bundled `unarc.dll`; executable's default stack was patched to 32 MiB. Run from `bundled-decoders`. It refuses overwrites.
- `2026-10-10-stage-arc.py`: wraps `<DataNN>-stage.srep` as `<DataNN>-srep.arc`, preserving original file CRCs/sizes. Validated end-to-end on Data04. Synthetic timestamps are placeholders, not original provenance.
- `2026-10-10-copy-solid.py`: new convenience helper to copy the original compressed solid block for Data01/02/04 using its index. Source is read-only; destination is exclusive-create. Not yet run on Data01/02.
- `2026-10-10-validate-extracted.py`: streams extracted files against original index sizes/CRC32. Passed for Data04; use it for subsequent extractions.
- `2026-10-10-decode-bfg.ps1`: **failed earlier host, do not use.** PowerShell-hosted unarc hit stack overflow. Direct combined `srep_old+lolz` extraction via the compiled host then stalled in piped LOLZ; those task-owned processes were stopped. The two-stage route below solved Data04.

Example continuation for **Data01**, then repeat for **Data02**. This route is proven for Data04, not yet these larger packages. Do not overwrite an existing intermediate; inspect/reuse it or use a fresh destination.

```powershell
$s = 'C:\Users\Korpus\Workbench\Inbox\2026-10-10-bfg-package-inspection'
Set-Location -LiteralPath "$s\bundled-decoders"
python "$s\2026-10-10-copy-solid.py" Data01
& '.\cls-lolz_x64.exe' d "$s\Data01-solid.lolz" "$s\Data01-stage.srep"
# Check the decoder exit code and output before proceeding.
python "$s\2026-10-10-stage-arc.py" Data01
New-Item -ItemType Directory -Path "$s\bfg-Data01" -Force | Out-Null
& "$s\2026-10-10-archive-host.exe" "$s\bundled-decoders" "$s\bfg-Data01" "$s\Data01-srep.arc"
python "$s\2026-10-10-validate-extracted.py" Data01 bfg-Data01
```

Data01: 18 files / 1,553,970,890 unpacked bytes, method `srep_old+lolz`; contains separate **ENGTEXT/RUSTEXT** `_common.resources` and `_ordered.resources`, plus English/Russian voice groups. **This is the next useful BFG research target.** Keep language variants separate.

Data02: 63 files / 6,389,005,568 bytes, same method; map resource containers. Inspect relevant duplicates and map-loading precedence.

Data03: 51 files / 277,060,544 bytes, method **`bpk+srep_old`**. Do not use the LOLZ helper for it. Bundled `cls-bpk.dll` and `cls-srep_old.dll` exist. Try the compiled archive host on the original Data03 package into a fresh `bfg-Data03` directory, then validate. This is **untested**, and the Bink group may not affect weapon research.

Other collection folders contain nested Inno installers (5.0.4, 5.1.2, 5.3.9) and a Wise installer in Nightmare. Existing 7-Zip lists Nightmare's setup as ZIP with errors; nested payloads are not fully unpacked. Existing tool: `C:\Program Files\7-Zip\7z.exe`. Do not execute setup. Keep non-BFG results separate.

## Saved parser data / reproducibility

`bfg-inno-header-0.bin` (187,364 bytes), `bfg-inno-header-1.bin` (1,628 bytes), `bfg-inno-payload.bin` (5,799,743 bytes), and `bfg-inno-entries.json` were created by inline Python, not a saved general-purpose Inno extractor. All header/subblock CRCs and 22 embedded-file SHA1s passed. Setup.exe's Inno signature starts at byte 2,399,494; header streams at 2,399,558 and 2,426,249; `zlb\x1a` payload marker at 279,552. Exact layouts are documented in report section 14.

Downloaded **text references only**, also in scratch: `ArhiveStructure.hs`, `ArhiveDirectory.hs`, `ByteStream.hs`, `ArcCommand.h`, `unarcdll.cpp`, `UnarcDllExample.cpp` (mirror/freearc); `inno-stream-block.cpp`, `inno-stream-chunk.cpp`, `inno-stream-lzma.cpp`, `inno-loader-offsets.cpp`, `inno-setup-data.cpp`, `inno-exefilter.hpp` (dscharrer/innoextract). Their source URLs are in the research report. No innoextract/FreeArc installation occurred. The only compilation was our small inspection host using existing `C:\Windows\Microsoft.NET\Framework\v4.0.30319\csc.exe`.

## Research after extraction

Resolve BFG resource indexes, active virtual-file precedence, inherited entity/model declarations, and multiplayer selection separately before reporting values. Start with `script/weapon_shotgun.script`, `script/weapon_base.script`, their includes, weapon/projectile/damage/brass declarations, every referenced animation variant, generated `.bMD5anim`, skeletons and sound/effect markers. Then expand the dependency inventory to all guns, as the user requested. Do not assume extraction proves which language/mod/mode was active on this PC.

Track pellet/projectile count/spread/damage; visual and camera recoil; successful-launch-to-successful-launch cadence; pump/reload/ammo-transfer/ejection; sound/effect boundaries. Distinguish script clocks, animation lengths, 24-fps blend units, clip frame rates, nominal markers and runtime timing. Record physical path/hash, virtual path, exact lines or binary offsets, inheritance provenance and calculations. No runtime measurements have been made.

Recovered GOG metadata identifies game `1135892318`, build `50332792385232087`, English. Executable version resources read **FileVersion 1.0.0.1**, **ProductVersion 1.0.34.6456**. These are package identity facts, not a verified installed/active build or a guarantee of exact agreement with the public C++ source.

## Local / Dropbox / cloud and constraints

All original packages and extracted licensed data are **local on PANZERV only**. Dropbox's configured local root is **`D:\Dropbox`**, but this task placed no copies there and performed no Dropbox upload; remote Dropbox contents were not checked. **GitHub contains factual research documents only**, not extracted assets, decoder binaries, helpers or the local manifest. Historical `/workspace/DOOM-3-BFG-Source` refers to a prior cloud session; no current cloud asset copy is established. A cloud-only Claude must obtain local access, not pretend these Windows paths are mounted.

Keep original packages read-only. Do not install tools, run game/setup executables or extracted installer/game scripts, modify unrelated Workbench/Shooter 1946/gamedev-studio files, implement Godot, or upload licensed assets. The user authorized extraction and use of the bundled archive decoders. Only `research/HANDOFF.md` and `research/DOOM_BFG_WEAPON_RESEARCH.md` may change inside the repository. Commit/push factual progress to the existing branch, update PR #1, leave it unmerged, and verify remote head plus both document contents before claiming publication. No background extraction or scheduled automation is active. Codex's goal bookkeeping last reported `usageLimited`, not complete; the research remains unfinished.
