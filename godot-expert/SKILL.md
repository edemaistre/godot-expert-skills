---
name: godot-expert
description: "Use when an agent drives Godot 4.7 on a Mac for any task: headless --script jobs, --import, --check-only, command-line exports, GUT tests, windowed screenshots, the headless editor (-e) or an MCP server into a running editor; when a run hangs or fails silently, a capture is black, Godot 3 code breaks in 4.x, 'Identifier not found', export templates are missing; or when a Godot brief (2D, 3D, shaders, VFX, UI, multiplayer, export) must go to the right godot-* specialist."
metadata:
  version: "0.1"
  date: "2026-10-02"
---

# Godot expert (technical director and agent protocol)

**Version 0.1** (2026-10-02), part of Godot Expert Skills v0.1. Target: Godot 4.7.2.stable (standard build, no C#), macOS Apple Silicon, Metal by default.

Expert Godot work is a loop run against evidence: set the budget, build one stage, measure it, look at it, fix it, advance. An agent without a mouse gets there by picking the right channel, passing only on a machine-readable result, and never trusting one signal alone. This skill is that protocol, the shared toolkit every godot-* skill calls (`scripts/`), and the map of the team.

**Status (2026-10-02):** every toolkit function below ran in Godot 4.7.2 here (23 live tests, 17 offline, all pass; `references/procedures.md`). The pre-release triage of the 13 domain skills' findings is in `tests/OPEN_ISSUES.md` (Lead triage).

## Stance (the expert delta)

1. **Headless output is the truth; the editor's Output panel is not.** MCP servers that scrape it "report errors for lines no longer on disk", and it does not list every project error (Fennara, 2vSYP7GyA5U [00:02:51, 00:06:20]). PixelLab's agent runs the Godot binary itself and reads the console (THwZYWuOdZI [00:13:45]). Pass on the `AGENT_RESULT` envelope plus a log scan, never on "it exited": a script with a parse error exits 0, with or without `--check-only`, and so does a script path Godot cannot find (version deltas, verified; L6).
2. **One writer per project.** A GUI editor holding a project keeps its own copy of the open scenes, so its next save can overwrite a headless edit [added]; merged `.tscn` files can fail to parse (Lilith Duncan, CAJ_iIedx_I [00:01:36]). The toolkit refuses a project a GUI Godot has open; work through that editor (MCP) or ask the user to close it. Never close a Godot or Blender process you did not start [added].
3. **Milliseconds against a budget, on the target.** Measure before optimising (Dan Does Dev, s2C2RO_WMh0 [00:07:12]); profile on the real device class, thermals included (Ian Bolton, WrjaUNAXYqk [00:09:35]). Headless frame time is loop time, not game cost; windowed frame time on this Mac is pinned to the display refresh; judge `gpu_ms` and `cpu_ms` (observed).
4. **Look AND check.** Every frame goes through `gd_review.image_checks` and a contact sheet you open. Headless draws nothing, so captures need a windowed run (observed). After AI asset import, check facing, animation direction, tile seams and camera jitter by eye (PixelLab, THwZYWuOdZI).
5. **Tests are the contract.** Assert on signals and public results, not internals (Godotneers, CreugthdgJ0 [01:08:36]; Butch Wesley, ImqhHLlPfZg [00:16:59]; Mike Schulze, x6GdTgIAFiU [00:24:19]); bound every await (2 s Godotneers, 5 s Schulze); prove the test bites by emptying the function and watching it go red (Wesley [00:13:40]).
6. **Design for a mouseless driver.** Split body and brain so a test brain can steer a character (Godotneers [00:18:14]). Headless has no mouse device (Wesley [00:48:33]), yet GUI input still tests headless: `get_viewport().push_input(event, true)` reaches Controls, while `Input.parse_input_event` never reaches a MOUSE_FILTER_STOP Control (godot-gameplay, godot-ui, observed); pass `true` so positions are logical, not window pixels. Levels hold a spawn point, not the player (Godotneers [00:56:56]).
7. **Edits persist only with `owner` and a save.** `add_child` alone is invisible in the Scene dock and lost on save (Godot docs, Running code in the editor); script edits bypass Undo unless routed through `EditorUndoRedoManager` (Queble, nW7YtSSJzbQ [00:04:41]).
8. **Be honest about visual editors.** VisualShader, AnimationTree, the TileSet and theme editors, and the particle inspector are visual; the agent writes `.gdshader`, `.tres`/`.tscn` text, or builds the resource in code, and names the GUI path for a human. Pick an MCP by feedback fidelity, not tool count (Fennara [00:09:22]).

## Execution channels

| Channel | Use for | Cost (measured here) | Rules |
|---|---|---|---|
| **Headless job** `gd_run.run_script` | logic, data, audits, scene building, imports, tests, exports | 0.2 s for a small job; `--import` about 2 s | default channel; watchdog ends a hung job (exit 124) |
| **Windowed job** `capture_scene`, `capture_sequence`, `profile_scene` | anything that must render: screenshots, sequences, GPU time | about 1 s for 4 views | 160x90 window, SubViewport at the requested size, focus handed back (2 restores per run observed); `GD_MAX_WINDOWED` 2 |
| **Headless editor** `run_script(editor=True)` | EditorInterface: open, edit and save scenes; EditorFileSystem | 1.1 s | wait 2 frames before editor calls; without `-e` EditorInterface calls fail (observed) |
| **GUI editor + MCP** (hybridindie `godot-editor-mcp`) | the user's open editor: live tree, selection, undoable edits | stdio probe 0.9 s, 23 tools exposed by default | only when the user has the editor open; project-local `.mcp.json` only (`gd_live.write_mcp_config`) |
| **Running game + MCP** (tugcantopaloglu) | drive a running game (input, eval) | not run here | clone, `npm run build`, a game autoload on port 9090 |

`gd_live.channels(P)` reports what is possible now. Prefer the headless channels; add an MCP for live scene inspection.

## Establish first

Ask once, then run: target platforms and lowest device; frame budget (default 60 fps desktop, 30 fps mobile); 2D or 3D; renderer (Forward+ desktop 3D, Mobile for demanding phones, Compatibility for web and widest reach, after Ray Hayes, KWhVVMpihsc [00:06:03]); art style and pixel art or not; whether the user's editor has the project open; the deliverable. Then `gd_live.channels(P)` and `gd_run.audit(P, "project")`, and write the renderer, physics engine (Jolt for 3D), stretch mode, main scene and autoloads into the plan.

## Workflow (the expert loop)

1. **Brief in numbers.** Budgets per platform, resolution, scale, style. GATE: every later check has a number to compare with.
2. **Project.** `gd_env.base_project("3d"|"2d", dest)` (APFS clone, 0.01 s, its own `user://` folder) or `new_project(...)`, which writes the project-manager defaults a hand-written `project.godot` lacks. GATE: `audit(P, "project")` has no flags.
3. **Baseline.** `check_all(P)`, `audit(P, "scene", scene=...)`, captures from fixed views, a profile if performance matters. GATE: zero parse and script errors; flags fixed or listed; sheet opened.
4. **Build one stage** with small idempotent jobs (get-or-create, `owner` set, explicit save). Route domain work to its owner (below).
5. **Measure.** Audit, tests (`run_tests`), `profile_scene` plus `gd_stat.budget_check`, or the export result. GATE: numbers against the brief.
6. **Look AND check.** Same views, `image_checks`, contact sheet opened, judged with the domain skill's critique. GATE: no flag that sets `ok` false, the change visible (`compare` changed_fraction above 0).
7. **Fix before advancing**, one change at a time; `compare` proves parity or gain.
8. **Deliver with evidence.** Each step with its toolkit call, then **Verified** (what ran, result file, number or frame) and **Assumed** (not run, defaults). Never call a scene, shader, level or build done without numbers and a frame you looked at.

## Shared toolkit (`scripts/`, the contract for every godot-* skill)

`import sys; sys.path.insert(0, "<godot-expert>/scripts")`. System python3, standard library (Pillow optional).

```python
gd_env.find_godot(); gd_env.version(); gd_env.templates_ok(tag=None, platforms=("macos",)); gd_env.install_templates()
gd_env.new_project(path, template="2d", renderer="forward_plus", jolt=True, name=None, stretch_mode="canvas_items",
                   stretch_aspect="expand", pixel_art=False, viewport=(1152, 648), main_scene=True, do_import=True,
                   isolate_user=False)
gd_env.base_project(kind, dest=None, isolate_user=True); gd_env.install_agentkit(project); gd_env.install_gut(project, version="9.7.1")
gd_env.isolate_user_dir(project, label=None); gd_env.user_data_dir(project); gd_env.userdata_report()
gd_env.read_project(project); gd_env.set_project_setting(project, key, value_literal)
gd_run.run_script(project, script_path_or_method, args=None, timeout=600, headless=True, editor=False,
                  out_dir=None, window=(160, 90), quit_after=None, extra_args=None, env=None) -> dict
gd_run.import_project(project); gd_run.check_only(script, project=None); gd_run.check_all(project, root="res://")
gd_run.run_tests(project, framework="gut"|"gdunit4", filter=None, dirs=("res://test",))
gd_run.ensure_preset(project, name, platform, export_path="", options=None)
gd_run.export(project, preset, out, release=True, pack=False); gd_run.verify_pack(pck, expect=(), load=())
gd_run.capture_scene(project, scene=None, out="captures/shot.png", size=(1280, 720), camera=None, frames=8, views=None,
                     extra_args=None)   # e.g. ["--debug-collisions"]
gd_run.capture_sequence(project, scene, out_dir="captures/seq", count=8, every=6, size=(640, 360))
gd_run.profile_scene(project, scene=None, seconds=5.0, warmup=30, size=None, headless=False, driver=None)
gd_run.audit(project, what="project"|"scene"|"resources"|"scripts"|"imports"|"classdb", **kwargs)
gd_review.image_checks(path); gd_review.compare(a, b); gd_review.contact_sheet(paths, out, cols=4); gd_review.godot3_flags(root)
gd_stat.parse_godot_log(text); gd_stat.frame_stats(frames_ms); gd_stat.budget_check(frames_ms, target_ms)
gd_stat.read_monitor_csv(path); gd_stat.parse_junit(path); gd_live.channels(project); gd_live.mcp_probe()
```

Every `gd_run` run returns `ok`, `error`, `result` (the job's dictionary), `exit_code`, `timed_out`, `parse_errors`, `script_errors`, `shader_errors`, `engine_errors`, `warnings`, `leaks`, `log_path`, `result_path`. Output lands in `<project>/.agent_out/` (carries `.gdignore`). Concurrency: `GD_MAX` (6) Godot processes, `GD_MAX_WINDOWED` (2), one per project, all through `gd_run.godot_slot`; `GodotBusy` when a GUI editor holds the project.

A domain job is a GDScript file in the project (AgentKit is copied to `res://addons/agentkit/`):

```gdscript
extends "res://addons/agentkit/agent_job.gd"
func run() -> Dictionary:
	var scene := await load_scene("res://main.tscn", 2)   # in the tree, 2 frames waited
	note("children", scene.get_child_count())
	return {"ok": true, "speed": arg("speed", 4.0)}       # arg() typed by its default
```

Or a method on any RefCounted script, `run_script(P, "res://tools/x.gd:build", args)` (it receives the job). Errors logged during the job flip `ok` to false unless `expect_errors` is set. A script error aborts a typed `run() -> Dictionary`, which then returns `{}`: an empty result always fails (`aborted: true`), even with `expect_errors`, so return at least `{"ok": true}`. `export` reports `bytes` for every file it wrote (`main_bytes` for the named one) and treats an iOS `export_project_only` Xcode project as the `artifact`. Captures emulate the project stretch (keep, keep_width, keep_height, expand, ignore, integer scale, scale factor, viewport mode) with black bars, matching the engine on 15 cases (L20); in your own job read a `make_viewport` SubViewport with `agent_capture.grab(vp)`. Helpers: `arg`, `out_path`, `load_scene`, `wait_frames`, `wait_physics_frames`, `wait_seconds`, `note`, `is_headless`, `captured_errors`; libraries `agent_build.gd` (`save_scene`, presets), `agent_profile.gd` (`record`). Domain code goes in `scripts/gd_<domain>.py` and `scripts/agentkit/<domain>/`.

## Numbers

| Value | Relative to | Source |
|---|---|---|
| 16.67 / 33.33 ms | frame budget at 60 / 30 fps | arithmetic |
| 8.33 ms | windowed `frame_ms` floor on this 120 Hz display, even with `--disable-vsync` | observed |
| 6.9 ms | headless frame pacing: the low-processor sleep (`low_processor_usage_mode_sleep_usec` 6900), whatever the work; benchmark with `--fixed-fps 60` and time the work | observed; godot-gameplay, godot-ui |
| 1.4 to 1.8 ms / 0 | GPU p95 of the fx test scene at 1080p on Vulkan (four runs) / on Metal | observed |
| 0.2 s, 10.2 s | small job; a hung job stopped by the watchdog (timeout 20 s) | observed |
| 60.8 / 75.0 / 41.4 MB | macOS universal zip / Linux x86_64 (binary plus .pck) / Web (9 files) export of Base3D | observed, L9 |
| 430 MB | iOS Xcode project (`export_project_only`), mostly the engine xcframework | observed, L9 |

## Quality gates

- **Measurable:** `ok` true with zero parse, script and shader errors; `check_all` `failed_files == []`; JUnit `failures == 0` and `tests > 0`; audit flags fixed or justified; `budget_check` ok (p95 under target, hitch ratio at most 1%); export exit 0, artifact size recorded, `verify_pack` lists the expected files.
- **Visual:** every capture passes `image_checks` (`ok` false on all_black, all_white, uniform, transparent) and was opened in a contact sheet; before and after from the same views.

## Common mistakes

| Mistake | What it looks like | Fix |
|---|---|---|
| trusting exit codes | parse error, exit 0 | `run_script`/`check_all` read the log |
| `--check-only` on code using autoloads | "Identifier not found: GameState" | `check_all` (in-process load); `check_only` files it under `autoload_false_positives` |
| hand-written `project.godot` | GodotPhysics3D, stretch disabled, UI does not scale | `new_project`, or set the keys |
| `class_name` in a fresh project | "Identifier not found" | `import_project` first |
| capture or `--write-movie` headless | 0 frames drawn; `--write-movie` crashes | windowed `capture_scene` |
| GPU timing on Metal | `gpu_ms` 0 | `profile_scene(driver="vulkan")` |
| export output inside the project | build imported and packed next time | outside, or `.gdignore` (auto) |
| macOS export | "ETC2 ASTC" or "Invalid bundle identifier" | `import_etc2_astc=true`; `ensure_preset` sets a bundle id |
| clones sharing `user://` | saves, input maps and the shader cache of other agents' clones show up (every clone was named Base3D) | `base_project` isolates by default; `gd_env.isolate_user_dir(P)` for other scratch projects |
| per-frame `TIME_PROCESS` | the same value on every CSV row (the monitor refreshes about once per second; 0 under `--fixed-fps`) | `process_wall_ms` column of `profile_scene`, or sentinel nodes |

## Team routing and handoffs

| Skill | Route here when the brief mentions |
|---|---|
| godot-architecture | project structure, scenes vs nodes, signals, Resources, autoloads, composition, typing, save data |
| godot-2d | sprites, TileMapLayer, TileSet, platformer or top-down feel, Camera2D, pixel-perfect, 2D lights, parallax |
| godot-3d-world | CharacterBody3D, blockout, CSG, GridMap, terrain, GLB import, Jolt, procedural meshes |
| godot-rendering-lighting | renderer choice, SDFGI, VoxelGI, LightmapGI, environment, fog, shadows, post, Compositor |
| godot-shaders | `.gdshader`, VisualShader substitutes, global uniforms, screen reading, compute, stencil |
| godot-vfx | GPUParticles, trails, sub-emitters, particle collision, impacts, juice |
| godot-animation | AnimationPlayer, AnimationTree, tweens, skeletons, IK, retargeting, root motion, cutscenes |
| godot-ui | Control layout, containers, themes, menus, HUD, focus and gamepad, localization, scaling on phones |
| godot-gameplay | state machines, inventory, dialogue, save and load, navigation, AI, procedural generation, input |
| godot-audio | buses, effects, spatial audio, adaptive music, SFX variation |
| godot-multiplayer | high-level multiplayer, RPCs, authority, spawner and synchronizer, lobbies, dedicated servers |
| godot-pipeline-automation | CLI and CI, editor plugins, GUT and gdUnit4, import plugins, GDExtension, MCP workflows |
| godot-performance-export | profiler, monitors, stutter, mobile, web, desktop and Steam export, signing |

A cross-cutting item has one owner: shader stutter and web threading go to godot-performance-export, naming the second skill in the brief. Every handoff packet carries: project path and channel, the brief in numbers, what was done with its toolkit calls, evidence (result JSON path, contact sheet and flags, CSV with `budget_check`, JUnit totals, export sizes), Verified, Assumed, open issues, and a fence (what the receiver may touch). The receiver reproduces one piece of evidence before building on it. Sister teams: blender-expert (asset fixes before GLB export), unreal-expert, unity-expert, maya-expert, zbrush-expert.

## Scenario assets into Godot

| Asset | Generated with | Lands in |
|---|---|---|
| sprites, props, icons, tilesets, UI pieces | scenario-game-assets | godot-2d, godot-ui |
| animation frames, sprite sheets, VFX loops | scenario-sprite-animation, scenario-sprite-pipeline | godot-2d (SpriteFrames), godot-vfx (flipbooks) |
| parallax planes | scenario-parallax-stage | godot-2d (Parallax2D) |
| GLB meshes | scenario-3d | godot-3d-world (import), godot-animation (rigs) |
| PBR textures, tileable materials | scenario-textures | godot-rendering-lighting, godot-shaders |
| skyboxes, 360 panoramas | scenario-skyboxes | godot-rendering-lighting (sky) |
| music, SFX, ambience, voice | scenario-audio, scenario-elevenlabs | godot-audio |

Check the asset before import (scenario-quality-gate) and again after it, in a capture from the game camera judged by the receiving skill, which also audits GLB scale, pivot and triangle count.

## Godot 4.7 traps (full list: version deltas section 21; observed ones in `references/expert-notes.md`)

- Godot 3 names parse silently on typed locals: `ps.instance()`, `tween.interpolate_property()` pass `--check-only`; run the code. `godot3_flags` scans for 24 old names; `audit(P, "classdb", checks=[...])` confirms an API exists.
- `--script` needs a SceneTree script ending in `quit()`; nodes are not in the tree during `_initialize` (await a frame); a runtime error before `quit()` hangs the process.
- `--upwards` does not exist and unknown flags are ignored silently: check `godot --help`.
- `-d` waits on stdin at the first error: never unattended.
- Editor singletons in `_init` of an `-e` run crashed (signal 11): wait 2 frames, open a scene first.
- 4.7 changes: typed-return overrides without `return` are parse errors; keyboard and mouse device IDs are 16 and 32; packed-array element writes skip setters; `AudioStreamPlayer2D/3D.area_mask` defaults to 0, so area bus overrides stop applying.
- Headless is not a renderer: MultiMesh `set_instance_*` writes are dropped (assign `MultiMesh.buffer` or build windowed), GPUParticles do not simulate (`finished` never fires), `create_local_rendering_device()` is null. Headless DOES lay out at any `root.size` with the project stretch, and the Dummy audio driver still mixes (godot-performance-export, godot-3d-world, godot-vfx, godot-shaders, godot-ui, godot-audio).
- `ProjectSettings.save()` from a job rewrites `project.godot`: the header comment and every key equal to its default go (`rendering_method="forward_plus"`, the viewport size). Prefer `gd_env.set_project_setting` and re-read the file (godot-architecture, godot-shaders).
- Warn-level GDScript warnings are not printed in headless runs: a headless typing gate needs the warning at error level (godot-architecture).
- The first `--import` after adding a GDExtension or a big addon can crash at shutdown (signal 11); `import_project` retries once and reports `retried`. A `.import` edited in the same second as the last import can be skipped: push its mtime forward (godot-2d, godot-audio, godot-pipeline-automation).
- Loading the running script with `CACHE_MODE_IGNORE` corrupts the VM ("Internal script error! Opcode", "Bad address index"): parse checks skip the running files (godot-vfx, godot-multiplayer; `check_all` skips `addons/agentkit/`).
- Deprecated but present, which hides the problem: TileMap, ParallaxBackground, SkeletonIK3D, `add_control_to_dock`. Do not copy 4.8-dev APIs.

## macOS notes

- Binary `/opt/homebrew/bin/godot` (link to `/Applications/Godot.app`); templates in `~/Library/Application Support/Godot/export_templates/4.7.2.stable/`; logs in `user://logs/godot.log`.
- `user://` is `~/Library/Application Support/Godot/app_userdata/<config/name>/` by default; isolated clones use `~/Library/Application Support/godot-agentkit/userdata/<name>-<hash>/`. Both are left behind after runs (logs, shader caches, about 100 MB in app_userdata on 2026-10-02): `gd_env.userdata_report()` lists them with sizes for a cleanup the user approves; app_userdata also holds real projects' data, never touch those.
- Windowed Godot activates itself; `open -g` and `--embedded` did not stop it (`--embedded` crashed). The toolkit hands focus back to the previous app (`GD_RESTORE_FOCUS=0` turns that off).
- Metal reports no GPU time: profile on Vulkan. Signing and notarization stay a human step (Artium Nihamkin, _n9l15CANag [00:04:05]). Quote every path.

## References

- `references/expert-notes.md`: expert claims by source with timestamps, and traps observed while building the toolkit.
- `references/procedures.md`: the toolkit API in full, job-writing rules, and each procedure with its live test and result.
- `references/critique.md`: the rubric for judging an agent's Godot output and a handoff.
- `references/gui-paths.md`: editor menus and settings for the same tasks.
- `references/sources.md`: every source with credentials, URLs and best timestamps.
