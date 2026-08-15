# dsh-composer-expand

Composer expand toggle for [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) (`dsh`). A ⬆/⬇ button in the composer tool row toggles the input between the default capped height and a tall 70vh writing view, so long drafts no longer scroll inside a tiny window.

## Why

The harness caps the composer textarea at a small height by default (the seat sets `--dsh-composer-text-max-height: 336px`, which caps the textarea scrollport). When the draft grows long, you end up scrolling inside a few lines instead of seeing the whole draft. This plugin keeps the default behavior for short drafts and offers a one-click toggle for the times when you are writing a long prompt.

## Features

- **⬆ button in the composer tool row** — sits in `conversation.input.right`, the official seat for clickable controls beside the send button.
- **Grows composer to 70vh on expand** — flips a CSS class on the conversation scrollport; the composer card + textarea both relax their `max-height`. Click again to restore the default capped height.
- **Persists per browser** — the toggle writes `localStorage[dsh-composer-expand:expanded]` so the choice survives reloads and workspace switches.
- **i18n zh/en** — button label and tooltip follow the harness `locale` service.
- **No custom wire protocol** — pure client-side CSS toggle; no host command, no LLM call, no session log entry.
- **Localize-safe** — anchors the CSS on DSH's stable `data-conversation-scroll` / `data-conversation-composer-overlay` data attributes, not on hashed classnames that change per build.

## Install

### Prerequisites

The `dsh` CLI must be on your `PATH`. If you only ever ran the harness through `npx`, `dsh` is not installed and you will get `zsh: command not found: dsh` — install it globally first:

```sh
npm install -g @deepseek-ai/dsh
```

### Add the bundle

For local development (this repo):

```sh
# 1. build the bundles (tsdown produces lib/index.js + lib/client.js)
pnpm install
pnpm build

# 2. add the bundle to your web profile
dsh plugin --profile web add ./

# 3. restart the harness with that profile
dsh --profile web
```

After the restart the ⬆ button appears in the composer tool row, next to the send button. Verify under Settings → Plugins that `dsh-composer-expand` is listed.

## How it works

| Piece | Mechanism |
| --- | --- |
| Button seat | `conversation.input.right` list slot (session scope); `ctx.slots.inject` registers a single React component |
| Toggle state | local React `useState` seeded from `localStorage[dsh-composer-expand:expanded]`; effect mirrors it back to localStorage on change |
| DOM hook | `document.querySelector('[data-conversation-scroll]').classList.toggle('dsh-composer-expand-on', on)` — the official stable attribute on the conversation scrollport |
| CSS contract | the `.dsh-composer-expand-on` class overrides `--dsh-composer-text-max-height` on `[data-composer-seat]` (the real 336px cap) to `var(--dsh-composer-expand-height)` (default `70vh`) and gives `[data-input-mirror]` a `min-height: 120px` so the box visibly grows even with little content |
| Hard deps | `export const inject = ['slots', 'locale']` — both are official harness client services |
| No host commands | `lib/index.js` is a no-op `apply()`; the whole plugin is client-side |

## Build

```sh
pnpm install
pnpm build   # tsdown: lib/index.js (ESM host) + lib/client.js (ModuleLoader client bundle)
```

## Known limitations

- The toggle is per browser, not per session. DSH renders only one composer at a time, so the mismatch is invisible today; if a future release ever renders more than one, narrow the DOM selector.
- The expanded height is fixed at `70vh`. A future version could expose a height slider in a settings card.
- The 336px cap (`--dsh-composer-text-max-height`) is a DSH design choice. This plugin only relaxes the composer textarea container, not the disclosure rows themselves.

## License

MIT