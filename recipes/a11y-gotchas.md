# AT-SPI / linux-use gotchas

Five traps that each cost real debugging time. Check these first when an
automation "almost works".

## 1. Elements below the viewport don't exist

AT-SPI only exposes what is inside the visible viewport. On a headless
Xvfb screen (e.g. 1600x1000) with a browser window sized to fit, DOM
controls rendered below the fold are **absent** from `linux-use state` —
not disabled, not hidden, *not there*.

Symptom: you know a `<button>` exists in the DOM but the tree doesn't
show it.

Fix: click a neutral page area to move focus out of inputs, scroll with
`linux-use key Next` (X11 PageDown), rescan, loop:

```bash
for i in 1 2 3 4; do
  ref=$(linux-use state --app "$APP" --all | jq -r '
    .elements[] | select(.role=="push button" and (.name|ascii_downcase)=="save") | .ref' | head -1)
  [[ -n "$ref" ]] && break
  linux-use click --x 900 --y 400   # neutral spot, out of the textarea
  linux-use key Next                # PageDown
  sleep 2
done
```

## 2. `paste` takes text *positionally*

```bash
linux-use paste "$MY_TEXT"        # correct
linux-use paste --text "$MY_TEXT" # WRONG: --text is ignored, stale clipboard pasted
```

`paste` types at the **current focus** — there is no ref argument.
Focus the field first (`act <ref>` or `click <ref>`), then paste.
Always verify afterwards:

```bash
linux-use read "$field_ref" | jq '.chars'   # 0 means nothing landed
```

## 3. Refs need the `#fingerprint`

Refs from `state` look like `App#pid123:/0/1/2#533926811`. The
`#<fingerprint>` suffix is mandatory for `act`/`read`/`click <ref>` —
a bare path is rejected with `bad_ref`. Take refs verbatim; never
truncate.

## 4. No window manager → keystrokes silently die

On a bare Xvfb display there is no WM, so X input focus is unset and
XTEST keystrokes (`key`, `type`, `paste`) go nowhere, while accessible
actions (`act`, `click`) still work. If text entry does nothing, you are
probably missing a WM — start `xfwm4` (or any WM) on that display.

## 5. The window is not the document

Browser chrome (tab bar, alerts like "Restore pages") shares the app
tree with the web document. When scanning for page elements, restrict to
the `document web` subtree (path under the web area), or you'll "find"
the wrong link/button — e.g. clicking a post's `save` action instead of
a comment form's.
