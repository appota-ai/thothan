---
description: Continue a Playable game or Tour (this folder's, or one from the app by its id)
argument-hint: '[gameId or tourId] [what to work on]'
---

Continue the existing Playable production. Arguments: $ARGUMENTS

1. Call `thothan_status` with `workdir` set to this folder's absolute path. If no production is checked out, open the supplied ID with `thothan_open_game`: `tourId` for a `tour_…` ID, otherwise `gameId`. Reuse its existing workspace.
2. Follow the matching skill and read the authoritative state:
   - **Tour:** use playable-tour, read `tour_fetch_tour_author_guide`, then `tour_get_tour_draft` and its guidance. Read the linked-Game mission reference when the Tour uses Game challenges. Game authoring calls belong to the Game's separate production.
   - **Game:** use playable-game, read `game_genesis_fetch_game_creation_procedure`, `game_genesis_getGameProject`, `game_genesis_getGameDesignDocument`, and the code. Read the Tour reporting reference when making this Game eligible for missions.
3. Work on what was asked. If nothing was asked, find the next unmet gate in that production's guidance and propose it.
4. Playtest locally and push. For a Tour, link the pushed workspace revision with `tour_link_tour_workspace`, then build it with `code_workspace_buildTourWorkspace`. For a Game, run `game_genesis_testGameProject` on the pushed revision and follow its QA gates.
5. Close with a `thothan_note` and the resulting IDs, revision and QA status. Publish only when the creator has authorized publication.
