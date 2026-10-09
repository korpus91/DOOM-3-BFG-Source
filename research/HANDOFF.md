# Task 1 research handoff

Status: **PARTIAL, first checkpoint**. Continue only Task 1, within the user's approximately 20–30 minute total budget. Do not start Task 2 without approval.

## Objective and repository identity

Research real Doom 3 BFG Edition (2012) weapon mechanics, beginning with the shotgun, for an original Godot 4.7.2 controller for Shooter 1946. Explain findings in plain English and cite source path, function and exact lines. Do not implement the controller.

- Official repository: `https://github.com/id-Software/DOOM-3-BFG`, `master`.
- User fork and exclusive PR target: `https://github.com/korpus91/DOOM-3-BFG-Source`, base `master`.
- Exact source base SHA: `1caba1979589971b5ed44e315d9ead30b278d8b4`.
- Working directory: `/workspace/DOOM-3-BFG-Source`.
- Dedicated research branch: `research/doom-bfg-weapons-task1`.
- Both origins' `master` resolve to the exact source SHA. `neo/d3xp/Weapon.cpp` and `.h` were independently compared with official raw files at this commit, byte-for-byte equal. This is BFG, not another Doom engine.
- Research start: 2026-10-08 22:27 America/New_York (02:27 UTC October 9). Prioritize final checkpoint/push by roughly 22:52–22:57 local time. Make an intermediate checkpoint about every ten minutes.

## Completed findings

The companion report records input -> script attack flag; definition-driven ammo/model/sound/recoil data; PresentWeapon transform/script/animation order; bounded accumulating visual weapon-kick duration with linear return; separate quadratic camera recoil gated by prior shared kick; camera view influencing the shot axis; independent randomized pellet directions; safe projectile spawning; flash/light/feedback/smoke C++ ordering. Every current finding has a source reference. The Godot design is currently a component outline.

The retail game data is absent (README.txt:15). No shotgun script, weapon definition, retail model/animation or PK4 archive is available. Do not fill in retail numbers from memory or the non-BFG game.

## Next steps for this task

1. Complete `idProjectile::Launch` / `Collide` / damage dispatch and target pain/death reaction tracing in `neo/d3xp/Projectile.cpp`, `Entity.cpp`, `Actor.cpp`, `Player.cpp` and `ai/AI.cpp`, with exact lines.
2. Complete script construction/state and animation-event tracing in Weapon.cpp and animator files; separate generic engine capability from unknown retail shotgun choreography.
3. Add first-person movement bob/lag positioning and exact camera feedback/aim relationship, including clock caveats.
4. Finish the original Godot architecture: transform hierarchy, accepted-shot timing, independent recoil recovery, animation/effects events, hit/damage contract and tunable Resource data. Do not implement it.
5. Review every important claim against the pinned source; update the Mermaid sequence and list unresolved questions explicitly.
6. Commit only `research/DOOM_BFG_WEAPON_RESEARCH.md` and `research/HANDOFF.md`; push only to the fork's research branch. Verify the exact remote commit is reachable. Create a PR with `--repo korpus91/DOOM-3-BFG-Source --base master --head research/doom-bfg-weapons-task1`, never upstream. Verify the PR and changed-file list.

## Persistence and access status

At creation of this handoff, the initial checkpoint is not yet claimed saved on GitHub. After committing/pushing, verify `git ls-remote origin refs/heads/research/doom-bfg-weapons-task1` and the commit through native Git. Update this section in the next checkpoint with actual results rather than guessed IDs. A checkpoint's own commit cannot be embedded in itself; use the branch tip and the previous verified checkpoint SHA.

GitHub API access to `api.github.com` fails at the proxy CONNECT step with HTTP 403, including an escalated read attempt. This is a domain-policy failure, not evidence that a new token is needed. Native Git reads work. A configuration draft now adds `api.github.com` while preserving known existing destinations; the user can apply it in environment settings. Do not print tokens or invent a PR URL/number. Saving the draft does not establish runtime connectivity or publication. If API access remains blocked, still push and verify the checkpoint; mark PR creation outstanding honestly.

## Constraints for any continuing AI

- Use existing checkouts; do not change Shooter 1946 or studio files.
- Do not install tools, compile/run the game, execute unknown game files, upload licensed assets, buy services or enable autonomous follow-up tasks.
- Do not copy Doom source or assets into the original Godot project.
- Do not fetch substitute non-BFG weapon scripts to imply retail BFG evidence.
- Do not commit source changes, staging directories or scratch notes. Stage the two named Markdown files explicitly.
- Keep confirmed C++ facts, missing retail data and proposed Godot choices visibly separate.
- Protect existing user changes; inspect Git status before/after writes.
- The root agent owns the research files/branch/commits. Parallel source notes are temporary research only.
- When time is short, stop expanding the trace and commit/push the useful findings; report PARTIAL. No Task 2 without user approval.
