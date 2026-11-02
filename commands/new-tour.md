---
description: Start a new Playable Tour from an idea and build its first version
argument-hint: <tour idea>
---

Use the playable-tour skill to make a new Playable Tour from this idea: $ARGUMENTS

1. Call `thothan_status` first, with `workdir` set to this folder's absolute path. If this folder already belongs to a game or Tour, ask whether to continue it instead.
2. Read the Tour author's rules (`tour_fetch_tour_author_guide`).
3. Propose the Tour in a few lines and wait for the creator's go-ahead: title, audience, play type, the stops and their challenges, and what each stop needs from the phone (location, camera, voice).
4. Call `thothan_start_game` with `kind: "tour"` (the idea above as `prompt`), then `tour_create_tour_draft`, then `tour_fetch_tour_guidance` with the returned `tour_id`.
5. Submit the plan, build the first version, then push, link and build it.
6. Stop there with a `thothan_note` and a short summary here.
