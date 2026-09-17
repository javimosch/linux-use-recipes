# Verify server-side, never optimistically

The single most important rule for automation that has real-world
effects (posts, comments, messages, purchases):

> A DOM change proves the client *tried*. Only a reload proves the
> server *accepted*.

## The pattern

1. Perform the action (click submit, press save).
2. Wait for navigation/settle (`sleep 5`+).
3. **Reload** (`ctrl+r`) or re-fetch the affected resource.
4. Assert the post-condition in fresh state:
   - the created object renders in the listing/thread, **and**
   - the empty-state banner is gone, **and/or**
   - a known confirmation string appears.
5. Only then mark the action complete. On failure keep it pending and
   let the retry policy handle it.

## Anti-patterns that caused real bugs

- "Typed text appeared in the box" → treated as posted. The box had
  stale clipboard junk and submit never happened.
- `success=true  # optimistic — may have posted` — a comment in
  production code. It would have reported success on every failure.
- Checking cookies exist → treated as "logged in". The session was dead
  server-side; the site showed a login wall.

## Human approval gates

For anything externally visible, keep a human in the loop upstream
(queue → approve → dispatch → execute) — but still verify execution
server-side. Approval authorizes the action; it does not prove it
happened.
