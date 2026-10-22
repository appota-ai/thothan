# Local playtest

Playtest the game locally before every gate. This catches what the server's type-check
cannot: a blank stage, runtime exceptions, dead controls, a HUD off-screen, a restart that
leaks.

## Dev server

```bash
npm run dev -- --host 127.0.0.1 --port 5173
```

Run it in the background and leave it running. Vite hot-reloads as you edit.

## With your browser tool

If you have a browser tool (Claude in Chrome, a built-in browser pane, Playwright MCP):

1. Open `http://127.0.0.1:5173/`.
2. Resize the viewport to the game's orientation:
   - portrait: 390×844
   - landscape: 844×390
3. Read the console for errors.
4. Play a full round with taps/clicks and keys. Then pause, lose or win, and restart.
5. Take a screenshot at each state and look at it.

## With the Playwright script

Use this when you have no browser tool, or for a repeatable check after each change.
Install once, into `.playtest/` (the checkout never syncs that folder):

```bash
mkdir -p .playtest && npm --prefix .playtest init -y > /dev/null && npm --prefix .playtest i --ignore-scripts playwright@1.49.1 && npx --prefix .playtest playwright install chromium
```

Save as `.playtest/playtest.mjs`:

```js
import { chromium } from 'playwright';

const url = process.argv[2] ?? 'http://127.0.0.1:5173/';
const landscape = process.argv.includes('--landscape');
const viewport = landscape ? { width: 844, height: 390 } : { width: 390, height: 844 };

const browser = await chromium.launch();
const page = await browser.newPage({
  viewport,
  hasTouch: true,
  isMobile: true,
  deviceScaleFactor: 2,
});
const errors = [];
page.on('console', (m) => m.type() === 'error' && errors.push(m.text()));
page.on('pageerror', (e) => errors.push(String(e?.stack ?? e)));

await page.goto(url, { waitUntil: 'networkidle' });
await page.waitForFunction(() => globalThis.__PIXI_APP__, null, { timeout: 15000 });
await page.waitForTimeout(2500); // the loader shows for at least 900 ms
await page.screenshot({ path: '.playtest/01-start.png' });

// Into play: tap the middle (title / start button), then let it run.
await page.mouse.click(viewport.width / 2, viewport.height * 0.6);
await page.waitForTimeout(3000);
await page.screenshot({ path: '.playtest/02-play.png' });

const scene = await page.evaluate(() => {
  const app = globalThis.__PIXI_APP__;
  const screen = app.renderer.screen;
  let visible = 0;
  const hud = [];
  const walk = (node) => {
    if (!node || node.visible === false || node.renderable === false) return;
    visible += 1;
    if (typeof node.label === 'string' && node.label.startsWith('hud:')) {
      const b = node.getBounds();
      hud.push({
        label: node.label,
        x: Math.round(b.x),
        y: Math.round(b.y),
        w: Math.round(b.width),
        h: Math.round(b.height),
        offscreen:
          b.x < 0 || b.y < 0 || b.x + b.width > screen.width || b.y + b.height > screen.height,
      });
    }
    for (const child of node.children ?? []) walk(child);
  };
  walk(app.stage);
  return {
    hud,
    layout: globalThis.__GAME_LAYOUT__ ?? null,
    screen: { w: screen.width, h: screen.height },
    visible,
  };
});

console.log(JSON.stringify({ errors, scene }, null, 2));
await browser.close();
process.exit(errors.length ? 1 : 0);
```

Run it with `node .playtest/playtest.mjs` (add `--landscape` for landscape games). Then:

- **Exit code 1:** fix every listed error first.
- **A tiny `visible` count, or a blank screenshot:** nothing is drawn. Check `Assets.load`
  paths and screen registration.
- **A `hud` entry with `offscreen: true`:** it is placed with raw coordinates. Place it with
  `placeHudItem` from the HUD kit.
- **Controls:** each one needs `w` and `h` of at least 44 (CSS px).
- **Screenshots:** always look at them. The numbers do not tell you whether the game looks
  right.

Extend the script for the game's own controls: `page.keyboard.press('Space')`,
`page.mouse.move`/`down`/`up` for drags, `page.touchscreen.tap`. Also add a second round
to check that restart starts clean.

## What a local run cannot show

The game runs without the Playable host here. Missing are player data and leaderboards,
cloud saves, multiplayer rooms, and native camera/location. The seeded shell catches the
unavailable host, so the game must still boot and play. Those features are verified on the
creator's phone. Ask them to open the game in Playable and use Test on this phone, then call
`game_genesis_checkGamePreview`.
