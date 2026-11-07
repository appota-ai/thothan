# Report gameplay to a Tour mission

Use this when the creator wants this Game's published build to qualify for Tour goals.
Work in the Game's existing production; authoring the Tour is a separate production.

## Eligibility

Inspect the existing result metrics and gameplay first. Confirm what “1000 points” means,
its units, bounds, and when scores change. Do not invent Rummikub rules or assume that a
published Game already reports a compatible metric.

Read the live `game_genesis_configureGamePlayerData` schema with `thothan_find_tools` in
Codex, or the direct service tool in Claude Code. Preserve existing field definitions,
leaderboards and player-card settings when updating the configuration. Add:

```json
"missionReporting": { "enabled": true, "checkpoints": true }
```

The metric version is server-owned; do not send `config.version` back as a
configuration field. Pass the revision and configuration fields the live schema accepts.

Set `checkpoints` to false for a final-result-only Game. Only solo Games currently qualify
for this protocol; enabling ordinary multiplayer does not enable Tour co-op or PvP.
Game-reported metrics remain `client_reported`, even though settlement runs on the server.

If the API schema lacks `missionReporting`, the service needs an upgrade. If the local
player-data shell lacks `getMissionContext` or `reportMissionProgress`, push local edits
and restore `/src/shell/playerData.ts` through `thothan_restore_scaffold`, then inspect the
returned shell. Do not fabricate a capability or write service credentials into the Game.

## Runtime

Use the exports from the seeded `src/shell/playerData.ts`:

- `getMissionContext()` tells whether the host has bound this play to a mission. Supported
  ordinary play returns `available: false`. Keep ordinary gameplay usable when mission
  support is unavailable; never assume every launch is a Tour attempt.
- When the context is available, call `reportMissionProgress(metrics, requestId)` after
  meaningful score changes if checkpoint support is enabled. Create a new request ID
  for each logical checkpoint. Retry an unacknowledged checkpoint with the same ID and
  identical metrics; reusing that ID for a changed score is rejected. Do not send per-frame
  reports or unconditionally call the mission API during ordinary play.
- Keep the normal `startRun` / `submitResult` final-result flow. In a mission the native
  host binds it to the authorized session. The Game never chooses a Tour run, attempt,
  beneficiary, verification level or Tour award. Handle a refused/expired report without
  claiming success or falling back to a personal result as mission evidence.
- Mission saves are isolated by the host. Do not import an ordinary personal save or
  historical best to satisfy the objective. Persistent online/economy APIs are unavailable
  in a mission; a Game using them needs a compatible mission gameplay path before opting in.

The Game can continue after a checkpoint satisfies the Tour objective. A checkpoint does
not finish its match. The Tour reads its own server state; no custom callback, direct
Tour HTTP request, boolean answer or client-side Tour scoring is needed.

## Publish and verify

Run normal Game QA and a device playtest of the reporting path. Check below-target,
threshold, duplicate retry, final result and interrupted-app behavior. Local preview and
ordinary personal scores do not qualify for a real Tour mission. An acknowledgment means
the source stored evidence, not that the Game itself may award Tour points.

When the creator has authorized publication, publish a new workspace revision with the
opt-in configuration and reporting code together. Published builds are immutable; do not
overwrite a retained build or guess its release ID. In the Tour production, discover the
published descriptor with `tour_get_published_game_for_mission` and pin its returned release
and metric version. Until the service and supported Playable host are deployed, describe
the integration as pending rather than promising that an ordinary Game launch will count.
