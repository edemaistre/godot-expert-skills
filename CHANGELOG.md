# Changelog

## v0.1 (2026-10-02)

First release: 14 agent skills for Godot 4.7 (a lead, `godot-expert`, and 13 specialists).

- Written from 216 expert videos (about 80 h) and 74 documentation pages, each skill with references, a `gd_<domain>.py` module and GDScript kit code (the shared GDScript AgentKit lives in `godot-expert/scripts/agentkit/`).
- Verified live: every procedure was run in Godot 4.7.2.stable on macOS (Apple Silicon): headless jobs, windowed captures, GUT and gdUnit4, ENet multiplayer, macOS, Web, Android APK and iOS Xcode-project exports.
- Blind-graded written scenarios (round 1, 12 briefs, one grader each): 79 % without the skills, 92 % with them; ten improved, one tied, one worsened. Graded before a refactor round that fed every gap back into the skills (not re-graded).
- Not run: phones, store uploads, signing and notarization, Gradle AAB, xcodebuild, a C# (.NET) Godot build, Steam, Nakama, netfox, GUI-only editor steps, godot-cpp and Rust builds, GitHub runners.
- Lead toolkit fixes after domain testing: a failed script run no longer reports ok, `check_only` fails on missing paths and scans logs (Godot's own exit code is 0 on parse errors), captures follow the engine's stretch rules, each project clone gets its own `user://`.
- SKILL.md descriptions quoted so every frontmatter is strict YAML.
