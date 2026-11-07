# Published Game missions

Use this flow for a required challenge such as “play Rummikub and reach 1000 points”.
The Game remains a separate published project. The Tour owns the requirement and award;
the platform owns admission, launch, evidence storage and settlement.

## Discover and pin

1. Read the current Tour guide and inspect the draft. V1 requires a solo Tour, a quiz
   stop, no question draw, and no team rules on the Game answer.
2. Discover the exact schema for `tour_get_published_game_for_mission` before calling it.
   In Codex use `thothan_find_tools`, then `thothan_call`; Claude Code can use the direct
   service tool. CLI fallback: `game call taka-playable-tour get_published_game_for_mission`.
   Its arguments are `game_project_id` and optional `game_release_id` (default `current`).
3. Use the creator's existing Game project ID. Never invent an ID, release, metric or
   claim that a particular Rummikub build supports scoring without inspecting it.
4. Require `missionReportingVersion: 1`, a matching positive `metricSchemaVersion`, and
   the requested numeric metric and bounds. A checkpoint objective also requires
   `supportsCheckpoints: true`. The release must supply the chosen evidence class.
   Current Game releases supply `client_reported`; that is not anti-cheat certification.
5. Pin the returned `projectId` and `releaseId`. Do not store `current`, a title, or a
   play URL as the mission's release identity. An older deployment without this API or
   capability must be upgraded before the link can be authored.

After confirming the metric's meaning and range, build the answer from discovered data:

```ts
const answer = {
  key: 'reach_1000',
  kind: 'game',
  prompt: 'Reach 1000 points in the game',
  points: 100,
  game: {
    projectId: release.projectId,
    releaseId: release.releaseId,
    metricSchemaVersion: release.metricSchemaVersion,
    participation: 'individual',
    verification: 'client_reported',
    history: 'mission_only',
    attemptAggregation: 'any_qualifying_attempt',
    awardScope: 'individual',
    required: true,
    objective: {
      type: 'metric',
      metric: metric.key,
      operator: 'gte',
      target: 1000,
      evaluation: 'on_checkpoint',
    },
  },
};
```

Here `release` and `metric` are the discovered descriptor and selected result metric.
The 1000 Game points qualify for a separate 100 Tour points in this example. Change
the prompt, award and target to the brief and supported metric. Use `on_finish` when
the final score must qualify or the Game only reports final results. Do not silently
change an immediate checkpoint requirement into a final-result requirement.

Put this answer in the submitted plan. Keep the objective public, while ordinary quiz
solutions remain private. Publication validates eligibility and reserves the release.

## Launch and refresh

- Read `PlayableTour.player.getState()` and use its active run. On a Play game tap call
  `startGameMission(run.id, {stopKey, answerKey, requestId})`. The host opens the pinned
  Game inside Playable; do not construct a URL, choose a session or embed another iframe.
- Keep the same request ID and payload for retrying an unacknowledged launch. A genuinely
  new gameplay attempt uses a new request ID, subject to the server's three-attempt
  limit and deadline. Do not reset the countdown or start another Tour run to gain retries.
- Read `getGameMission(run.id, {stopKey, answerKey})`. Only `result.mission.state ===
'satisfied'` means passed. Handle provisioning, playing, failed and expired states.
  Never submit the Game's score through `submitAnswer`, skip it as an ordinary answer,
  optimistically add Tour points, or treat returning to the Tour as completion.
- Listen for `playable-tour:changed` and refresh on return. Reload `getRun(run.id)` after
  qualification because settlement changes its revision. Use that revision for later
  progress or completion writes, and refresh `getScore` for the server-awarded Tour points.
  Honor `playable-tour:visibility` (`detail.suspended`)
  to pause rendering, audio and polling while the Game covers the Tour.
- Preview/capture should render the objective and button without real admission or
  gameplay. Unsupported hosts should show an update-app state. Neither may award points.

## Verification and lifetime

Verify that 999 does not pass and an accepted 1000 checkpoint passes once. For `on_finish`,
verify that a checkpoint alone does not pass. Test ordinary quiz questions both before
and after the Game, transient failures, and returning with the refreshed run revision.
Device interruption after a durable report must not require replaying gameplay; the
server reconciles it even if the player never returns to the Tour.

Removing a Game mission from the newest draft does not remove references from retained
Tour releases. Game deletion is blocked while such references exist; Tour deletion queues
their release and attempt cancellation. Do not delete/recreate either project to repair
a failed attempt. Co-op, all-member goals, team awards and player/team-versus outcomes
require a later protocol; reject those configurations rather than claiming support.
