---
description: Start a new Playable game from an idea and build its first playable version
argument-hint: <game idea>
---

Use the playable-game skill to make a new Playable game from this idea: $ARGUMENTS

1. Call `thothan_status` first, with `workdir` set to this folder's absolute path. If this folder already belongs to a game, ask whether to continue it instead.
2. Read the procedure (`game_genesis_fetch_game_creation_procedure`).
3. Propose the concept in a few lines and wait for the creator's go-ahead: title, genre key, core loop, controls, camera, screen flow and orientation.
4. Call `thothan_start_game` (the idea above as `prompt`), then `game_genesis_startGameProject`.
5. Build to the first playable slice and playtest it locally.
6. Stop at that gate with a `thothan_note` and a short summary here.
