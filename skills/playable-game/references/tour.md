# Playable Tours

A Playable Tour is an interactive, location-aware mini-app: a walk or a hunt through real places,
with stops, challenges and a story. You build it the way the in-app Tour author does, from a local
folder, and it shows up in the creator's Playable app like one made there.

Everything in the skill's **Safety** section applies to Tours too. What differs from a game:

- **The folder** is a checkout of the Tour's Code Workspace: a dependency-free web app
  (`index.html`, TypeScript modules under `src/`, `src/style.css`) with a capability manifest,
  `tour.manifest.json`. It has no packages: never add a `package.json` or imports from npm. The
  server bundles it when you build.
- **The services** are the Tour service (`tour_*`, Taka GraphDB: the draft, plan, private grading,
  builds and publication), Image Forge, Sound Sage, the checklist (`gtd_*`), and two workspace calls
  that only the server can make: `code_workspace_importTourImage` and
  `code_workspace_buildTourWorkspace`. Game Genesis is not part of a Tour.
- **Rooms** (live sessions with invited players) stay in the Playable app.

## Start

1. `thothan_status` with `workdir`. A Tour folder says `kind: Playable Tour`.
   - New Tour: `thothan_start_game` with `kind: "tour"`, the title, and the creator's request as
     `prompt`. It creates the Tour's workspace and checks it out. **Never create another workspace.**
   - A Tour made in the app: `thothan_open_game` with its `tourId`.
   - No Tour author: the creator makes one in the Playable app (Create → Authors).
     `thothan_list_authors` with `kind: "tour"` lists them.
2. Read `tour_fetch_tour_author_guide` once per session. It is the in-app author's standing
   rules: plan fields, challenge kinds, scoring, assets, capture, the `PlayableTour.player` API
   and the layout. Follow them, except where the table below says what changes locally.
3. On a new Tour, call `tour_create_tour_draft` once (a non-empty title and brief from the
   creator's request), then `tour_fetch_tour_guidance` with the returned `tour_id`, and follow
   that guidance. On a resumed Tour call `tour_get_tour_draft` first: the graph state is
   authoritative.

## What changes locally

| The in-app author                         | You do                                                                                                                                                                                     |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| creates one Tour workspace after the plan | It already exists: the folder. Use the `workspaceId` from `thothan_status`.                                                                                                                |
| edits files with workspace tools          | Edit the folder with your own tools, then `thothan_push` with a short `note`.                                                                                                              |
| imports each asset with `importTourImage` | Push first, then `code_workspace_importTourImage` with `expectedRevision` set to the pushed revision. The import is a new revision; the folder takes it right after (`/tour.assets.json`). |
| links and builds the latest revision      | `tour_link_tour_workspace` and `code_workspace_buildTourWorkspace` with the revision you pushed. A call with unpushed changes in the folder says so: push, then link and build again.      |
| asks questions with forms                 | Ask the creator here, one question at a time.                                                                                                                                              |

- **Never edit `/tour.assets.json`**: the server writes it on each import. Reference imported files
  only by their local `/assets/...` path.
- **Media never go in the folder.** Workspaces hold text only. Art comes from Image Forge,
  narration, music and effects from Sound Sage, and the creator's own photos, videos and voice
  notes as file ids. Each goes in through `code_workspace_importTourImage`. Never put file ids,
  `/f/` links, remote URLs, SVG or base64 in Tour source.
- **Grading stays private.** Tour source never contains correct answers or grading logic, and never
  computes points: the server grades and scores.

## Build loop

1. Plan with the creator: audience, play type, stops, challenge kinds, capabilities. Submit it with
   `tour_submit_tour_plan` (flat fields, as the guide says). Correct a validation error once; never
   resend an identical payload.
2. Write the Tour in the folder. Preview it locally with
   `npx --yes vite@6 --host 127.0.0.1 --port 5173` in the background (it serves the TypeScript
   as is) and open it at a phone viewport (390×844). `PlayableTour.player`, sensors and imported assets are only there inside
   Playable: the Tour must still load, show each stop and handle a missing bridge or a denied
   permission.
3. Generate art and sound, import each one, and wire the local paths into the Tour.
4. `thothan_push`, `tour_link_tour_workspace` with the pushed revision, then
   `code_workspace_buildTourWorkspace` with the same revision and the current `expectedTourVersion`.
   Fix what the build reports and repeat.
5. Cover: a portrait 2:3 image from Image Forge with the exact Tour title as large, readable text
   and nothing else. Bind it with `tour_set_tour_cover`.
6. Keep the checklist and the creator's chat current: `gtd_*` for the stages, `thothan_note` at
   milestones (plan approved, Tour built, cover set, ready to publish).
7. **Publish only when the creator explicitly asks**: `tour_publish_tour` with only `tour_id`.
   It refuses while the folder has changes that are not pushed and built. To take a Tour down,
   `tour_unpublish_tour`; if it names open rooms, ask the creator before closing them.

Hand off with the `tourId`, the workspace and its built revision, and the Playable link from
`thothan_status`. The creator tests it on their phone in the Playable app.
