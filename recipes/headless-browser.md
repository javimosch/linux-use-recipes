# Headless drivable browser (Xvfb + dedicated profile)

The foundational pattern: a real Chromium-family browser on a virtual
display, with a persistent profile, drivable by linux-use.

## Why not Playwright/Selenium

Sites with bot detection (Reddit, Google, Cloudflare-fronted apps) flag
CDP-controlled browsers. A normally-launched browser driven via AT-SPI +
XTEST is indistinguishable from a human session — because it *is* a real
session.

## Requirements

- `--force-renderer-accessibility` — without it the browser exposes one
  opaque `frame`; with it the real DOM appears as accessible elements.
  The flag **cannot** be added to a running browser.
- Dedicated `--user-data-dir` — the flag would also apply to your
  everyday browser if it shared the process/profile.
- **No** `--no-sandbox`, **no** `--disable-gpu` — bot-detection signals.
- A window manager on the Xvfb display (`xfwm4`) — see gotcha #4 in
  [a11y-gotchas.md](a11y-gotchas.md).

## Skeleton

```bash
DISPLAY_NUM=:98
PROFILE=~/.local/share/linux-use/edge-mybot
XDG_RUNTIME_DIR=~/.local/share/linux-use/xdg-mybot

Xvfb $DISPLAY_NUM -screen 0 1600x1000x24 &
DISPLAY=$DISPLAY_NUM XDG_RUNTIME_DIR=$XDG_RUNTIME_DIR xfwm4 &
DISPLAY=$DISPLAY_NUM XDG_RUNTIME_DIR=$XDG_RUNTIME_DIR \
  microsoft-edge --user-data-dir="$PROFILE" \
    --force-renderer-accessibility "$URL" &

EDGE_APP="Microsoft Edge#pid$!"
linux-use state --app "$EDGE_APP" --all
```

A generic implementation ships with linux-use as
`contrib/a11y-browser` (supports `--hidden`, `--wm`, `--xephyr`).

## One-time login pattern

Sessions live in the profile dir, so:

1. Stop the Xvfb instance.
2. Relaunch the **same** `--user-data-dir` on the real display `:0`.
3. Human logs in manually (SSO, 2FA, whatever).
4. Verify logged-in state *before* closing (check a username element in
   the a11y tree — cookies existing is not proof).
5. Kill it, restart on `:98`. The session persists.

Copying a `Cookies` file from your real profile into the bot profile
usually yields a *dead* session server-side — tokens get invalidated.
Budget for the manual login step.

## Xephyr alternative

`a11y-browser --xephyr` runs the virtual browser in a visible nested
window on the real desktop — lets the human do the login without
swapping displays.
