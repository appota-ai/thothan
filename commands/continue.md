---
description: Continue a Playable game or Tour (this folder's, or one from the app by its id)
argument-hint: [gameId] [what to work on]
---

Use the playable-game skill to continue a Playable game. Arguments: $ARGUMENTS

1. Call `thothan_status` with `workdir` set to this folder's absolute path. If this folder has no game and a game id was given, call `thothan_open_game` with it.
   If it is a Playable Tour (`kind: Playable Tour`, or a `tour_…` id was given), continue with the playable-tour skill instead: `thothan_open_game` with the `tourId`, then `tour_get_tour_draft`.
2. Read the procedure (`game_genesis_fetch_game_creation_procedure`).
3. Read the game's state: `game_genesis_getGameProject`, `game_genesis_getGameDesignDocument`, and the code.
4. Work on what was asked. If nothing was asked, find the next unmet gate in the procedure and propose it.
5. Playtest locally, push, run `game_genesis_testGameProject`, and close with a `thothan_note`.
