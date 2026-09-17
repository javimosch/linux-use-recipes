# linux-use-recipes

Field-tested recipes for automating desktop and web applications with
[`linux-use`](https://github.com/javimosch/linux-use) — driving real apps
through the AT-SPI accessibility tree plus XTEST input, instead of DOM
automation (Playwright/Selenium) that bot-detection catches.

Every recipe here was learned the hard way on a production automation.
They exist so the next agent doesn't rediscover them by burning hours.

## Contents

| Recipe | Problem it solves |
|---|---|
| [recipes/headless-browser.md](recipes/headless-browser.md) | Dedicated browser profile on Xvfb that linux-use can drive, with a real session |
| [recipes/a11y-gotchas.md](recipes/a11y-gotchas.md) | The five AT-SPI traps that silently break automation (offscreen elements, paste semantics, refs, …) |
| [recipes/reddit-comment.md](recipes/reddit-comment.md) | Full old.reddit.com comment-posting flow: session, scraping, submit, verification |
| [recipes/human-verification.md](recipes/human-verification.md) | Why you must verify the *server* accepted the action, and how |

## The tool in one paragraph

`linux-use` exposes a Linux desktop's accessibility tree as JSON
(`state`), lets you search it (`find`), invoke accessible actions
(`act`), click elements or coordinates (`click`), send keystrokes via
XTEST (`key`), and paste text (`paste`, `sendtext`). Chromium-family
browsers only expose web content to AT-SPI when launched with
`--force-renderer-accessibility`, and that flag cannot be applied to an
already-running process — hence the dedicated-profile pattern in every
browser recipe.

## License

[Unlicense](LICENSE) — public domain. Use freely, no attribution needed.
