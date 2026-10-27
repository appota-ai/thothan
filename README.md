# Thổ Thần for Claude Code and Codex

Build Playable games on your own machine with Claude Code or Codex. The game ends up in the
Playable app just like one made there:

- it is listed under the author;
- the production chat shows each step;
- progress, assets, tests, the phone preview and the revisions are all there;
- you can keep chatting with the author in the app.

## Install

**Claude Code**

```bash
claude plugin marketplace add appota-ai/thothan
claude plugin install thothan@appota-ai
```

Restart Claude Code, open an empty folder and ask for a game, or use
`/thothan:new <idea>`. To keep working on a game, use `/thothan:continue [gameId]`.

**Codex**

```bash
codex plugin marketplace add appota-ai/thothan
codex plugin add thothan@appota-ai
```

Restart Codex, open an empty folder and ask for a game.

- Codex lists the folder tools (`thothan_*`) and reaches the service APIs through
  `thothan_find_tools` and `thothan_call`. With every API listed separately, Codex would
  hide most of them.
- Codex doesn't tell the plugin which folder you're in, so the agent passes it as `workdir`.
  The skill tells it to.
- If you installed with `git clone` and `game install codex` before, run
  `node ~/.thothan/bin/thothan.mjs game install codex` once more. It switches you to the
  plugin and removes the old copied skill and `config.toml` entry, which would otherwise
  win over the plugin.

You need Node.js 18 or newer. The games are Vite projects, so `npm` is needed too.

## Cloud sessions

**Only a phone?** Start from [appota-ai/thothan-starter](https://github.com/appota-ai/thothan-starter):
tap **Use this template**, pick the new repository in Claude Code or Codex, turn on the
environment's internet access, and ask for a game.

Claude Code on the web and Codex cloud work like Claude Code on your machine, with two
differences:

1. **Push the game folder to GitHub.** Cloud sessions install no plugins; they use what the
   repository contains. The plugin makes every game folder ready for this when it starts or
   opens a game: it adds `.thothan/` (the plugin's CLI), `.mcp.json`, the skill and `AGENTS.md`.
   Commit them with the game. None of them are pushed into the game itself.
2. **Allow Thổ Thần in the cloud environment, once.** Set its network access to **Custom** and
   add `thothan.appota.ai` (keep the default list ticked). In Codex, allow agent internet
   access to `thothan.appota.ai`.

Then ask for your game. The first time in a session the agent shows a sign-in link and code;
approve it on any device, as on your machine. A session that sits idle long enough to lose its
machine asks again.

A game folder inside a larger repository is left alone; run `thothan game install repo` in that
repository to set it up.

<details>
<summary>Advanced: sessions nobody is watching (scheduled tasks)</summary>

Without someone to approve a link, sign in with an API key from
[Settings → API Keys](https://thothan.appota.ai/settings/apikey):

- **Claude Code:** add a network secret to the environment for host `thothan.appota.ai`
  (header `X-API-Key`, no prefix, the key as the value) and the variable
  `THOTHAN_AUTH=proxy`. The key never enters the session.
- **Codex:** set the variable `LOBEHUB_CLI_API_KEY` (Codex removes secrets before the agent
  runs). Anyone who can open the environment can read it.

A key from the environment is only ever sent to `thothan.appota.ai`.

</details>

## What you need

- **A Thổ Thần account**: the same account as the Playable app and thothan.appota.ai.
- **A game author**: create one in the Playable app (Create → Authors). Each game belongs to
  one, and it's the author you keep chatting with in the app.
- **A plan with the creation tools**: starting a game needs Image Forge, as it does in the
  app. Music and sound effects need Sound Sage.

## Sign in

Signing in uses Thổ Thần's OAuth device flow; no password or key goes through the agent.

1. The first time, the agent calls `thothan_login`.
2. It gives you a link and a code. Approve "Thổ Thần CLI" in your browser.
3. The agent picks up the sign-in on its own.

You can also run `node bin/thothan.mjs login` in a terminal.

The sign-in is stored in `~/.thothan/cli/`, readable only by your user. Claude Code, Codex and
the CLI on this machine share it, so you sign in once. It renews itself; after about 30
days without use, you sign in again. `node bin/thothan.mjs logout` signs out.

## Security

- **What the agent can do.** Signed in, the agent acts as you in Thổ Thần, but only through
  the game tools.
  - It can create and edit your games and their chats, and generate art and sound on your plan.
  - It can publish a game. The skill tells it to publish only when you ask, but that is an
    instruction, not a technical block, so review publishing steps as you would any other.
  - Your sign-in token stays in the MCP server; no tool hands it to the agent, and the skill
    forbids reading `~/.thothan/cli/`. The file there is encrypted with a key derived from this
    machine, which stops casual reading but not a program running as you. Treat it like an
    SSH key: anything you let run as your user can use it.
  - The token only ever goes to the server that issued it. If `LOBEHUB_SERVER` or
    `~/.thothan/cli/settings.json` points somewhere else, the plugin acts signed out, and
    `thothan_login` names the other server before you approve anything.
- **Approve only codes you started.** Approve a sign-in code only when you just asked the
  agent to sign in. A code someone else sends you would give them your account.
- **Credentials never sync.** `.env*`, `.npmrc`, `.mcp.json` and key files are never pushed
  from your folder. Game builds leave them out too, since published game files are served
  to players.
- **What a shared game can and can't plant.** Collaborators on a shared author can edit the
  game's code, and that code runs on your machine when you play it locally.
  - A pull never writes agent or editor config: `CLAUDE.md`, `AGENTS.md`, `.mcp.json`,
    `.claude/`, `.codex/`, `.agents/`, `.vscode/`, `.npmrc`, `.git/`, in any letter case.
  - A push only goes to the game's own workspace, as the hub reports it. A folder whose
    checkout file names another workspace is refused.
  - The skill has the agent show you changes to `package.json` or `vite.config.ts` before it
    installs or runs them.
- **Signing out.** `node bin/thothan.mjs logout` signs out on this machine. To end every
  session, sign out of Thổ Thần on the web.

Report a vulnerability as described in [SECURITY.md](SECURITY.md).

## How it works

- **The game folder** is a checkout of the game's Code Workspace (Vite + PixiJS). The agent
  edits it with its own tools, runs the dev server, and playtests it in a real browser.
- **The `thothan` MCP server** (`bin/thothan.mjs game mcp`) gives the agent:
  - every Game Genesis, Image Forge, Sound Sage and checklist API that the in-app game
    author has, with live schemas;
  - `thothan_*` tools for the folder: sign in, start or open a game, push, pull, post a
    note, restore scaffold files, list revisions, and revert;
  - `thothan_view_images`, which shows the agent Thổ Thần images (Spine close-ups, motion
    sheets, generated art) so it can review its own characters. It only fetches the
    server's own `/f/` file links.
- **Every call runs on Thổ Thần, against the game's production chat**, and is recorded
  there as the author's step. Pushes from the folder become workspace revisions, which the
  app's preview and revisions follow.
- **The `playable-game` skill** teaches the agent the procedure. It reads the canonical
  procedure from Game Genesis, then applies what changes when working locally.

`.appota-game.json` in the game folder says which production the folder belongs to. It is
never pushed into the game.

## Without an agent

`node bin/thothan.mjs game --help`:

| Command                              | What it does                              |
| ------------------------------------ | ----------------------------------------- |
| `game new "<title>"`                 | Start a game                              |
| `game open --game <gameId>`          | Check out a game made in the app          |
| `game status`                        | Show where the game stands                |
| `game push` / `game pull`            | Sync the folder with the game's workspace |
| `game call <service> <api> '<json>'` | Run one service call                      |

---

`bin/thothan.mjs` is the Thổ Thần CLI's plugin entry, bundled unminified so it can be
read and audited. It is built from the CLI source with `bun run plugin:build`; do not edit
it by hand.
