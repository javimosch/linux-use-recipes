# Posting a Reddit comment via linux-use

End-to-end recipe for replying to an old.reddit.com thread from a
headless Edge. Learned 2026-09-17 on a production karma-growth fleet.

## Prerequisites

- Headless Edge on Xvfb per [headless-browser.md](headless-browser.md),
  **with a WM** (keystrokes must work).
- A logged-in session for the account that will post.
- Account preference: *Settings → "Opt out of the redesign"*. Without
  it old.reddit.com redirects to `/login/?reason=lor2` **even with a
  valid session**.

## Step 1 — Find the comment textarea

The page has *two* unnamed `entry` elements:

- the post's **shortlink** field (sidebar) — first one, a trap
- the real **comment textarea** — under the comments section, inside
  `form → section → entry`

Pick the last unnamed entry, or better: the entry whose path sits under
the comments `form` node.

## Step 2 — Fill it

```bash
linux-use act "$comment_ref"          # focus
linux-use key ctrl+a; linux-use key Delete   # clear leftover text
linux-use paste "$reply_body"         # positional arg, at focus
chars=$(linux-use read "$comment_ref" | jq '.chars')
[[ "$chars" -ge 30 ]] || exit 1       # verify it landed
```

## Step 3 — The save button is offscreen (the big trap)

old.reddit renders the submit `button` **below the textarea**, and on a
1600x1000 Xvfb that puts it **below the viewport** — so it is not in the
a11y tree at all. Scroll until it appears (see
[a11y-gotchas.md](a11y-gotchas.md) #1), then:

```bash
linux-use act "$save_ref"             # press
sleep 5
```

Note: even with a **Spanish UI**, the comment submit button reports the
English name `"save"`. The post's own `guardar` (save-post) link is a
different element — do not click it.

## Step 4 — Verify on the server, not the DOM

A successful press updates the DOM optimistically; the only proof is a
reload:

```bash
linux-use key ctrl+r; sleep 7
linux-use state --app "$EDGE_APP" --all | python3 -c "
import json,sys
els=json.load(sys.stdin)['elements']
snippet='$SNIPPET'   # first ~40 chars of the reply
posted = any(snippet in e.get('name','') for e in els)
banner = any('no hay comentarios' in e.get('name','') for e in els)
print('OK' if posted and not banner else 'FAILED')
"
```

Only mark the action done when `posted && !banner`. Otherwise leave it
pending for retry — never report success unverified.

## Scraping notes (prospecting)

- Post title: read the `document web` frame name — it is
  `"<post title> : <subreddit>"`. Do not guess the title from links;
  sidebar links like "Enviar un nuevo enlace" / mod-team entries will
  win otherwise.
- The UI language follows the **account**, not `--lang=` — handle ES/FR
  strings or force English in account prefs.
- Never interpolate scraped text into double-quoted `python3 -c "..."`
  — apostrophes break the shell quoting. Pass via env vars.
- `old.reddit.com/r/<sub>/new/` and `search?q=...&restrict_sr=1` both
  work in the a11y tree once logged in.
