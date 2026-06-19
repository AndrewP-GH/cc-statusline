---
name: setup
description: Configure (or remove) the cc-statusline status line in Claude Code by writing statusLine into the user's settings.json. Use when the user asks to enable, install, set up, or disable cc-statusline.
---

# cc-statusline setup

Claude Code plugins cannot set the main `statusLine` automatically, so this skill
writes it into the user's `~/.claude/settings.json`.

> **Why an absolute path (not the bare command name):** the `statusLine` command runs
> in a subprocess that does **not** get the plugin's `bin/` on `PATH` (only hooks do).
> Pointing `statusLine.command` at the bare name `cc-statusline` therefore renders an
> **empty** status line. We resolve the script's absolute path and write that instead.

## Enable

1. **Resolve the script's absolute path.** Prefer the unversioned marketplace clone
   (it survives plugin version bumps; the `cache/.../<version>/` path does not):
   ```sh
   BIN=$(ls ~/.claude/plugins/marketplaces/*/plugins/cc-statusline/bin/cc-statusline 2>/dev/null | head -1)
   [ -n "$BIN" ] || BIN=$(command -v cc-statusline)
   echo "$BIN"
   ```
   - Prints a path → continue, using it below.
   - Prints nothing → the plugin is not installed/enabled. Tell the user to run
     `/plugin` and enable **cc-statusline**, then re-run this skill. Stop here.

2. **Locate settings.** Target `~/.claude/settings.json` (create `{}` if missing).

3. **Back it up.** Copy to `~/.claude/.settings.json.<timestamp>.bak` before editing.

4. **Write the statusLine block**, preserving every other key, substituting the `$BIN`
   path resolved in step 1:
   ```json
   {
     "statusLine": {
       "type": "command",
       "command": "bash <ABSOLUTE_PATH_FROM_STEP_1>",
       "padding": 0
     }
   }
   ```
   Use the Read + Edit tools to merge this key in. Do not rewrite unrelated settings.
   (`padding: 0` lets the line use the full terminal width; drop it to keep default
   padding.)

5. **Confirm.** Tell the user it's set and that the status line appears on the next
   prompt render (a new session guarantees a clean reload).

> After a plugin **update**, if you used `command -v` (versioned cache path) instead of
> the marketplace clone, re-run this skill so the path points at the new version.

## Disable

1. Back up `~/.claude/settings.json` as above.
2. Remove the `statusLine` key (or restore a prior backup).
3. Confirm removal; the default status line returns on next render.

## Customize

The renderer reads optional env vars (set them in your shell profile):

- `CC_SL_FULL`  — glyph for an elapsed unit (default `●`)
- `CC_SL_HALF`  — glyph for the in-progress unit (default `◐`)
- `CC_SL_EMPTY` — glyph for a remaining unit (default `○`)
