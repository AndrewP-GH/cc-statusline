---
name: setup
description: Configure (or remove) the cc-statusline status line in Claude Code by writing statusLine into the user's settings.json. Use when the user asks to enable, install, set up, or disable cc-statusline.
---

# cc-statusline setup

Claude Code plugins cannot set the main `statusLine` automatically, so this skill
writes it into the user's `~/.claude/settings.json`. The plugin's `bin/` directory
is added to `PATH` while the plugin is enabled, so the status line is invoked by
name as `cc-statusline` (no version-pinned absolute path — survives plugin updates).

## Enable

1. **Verify the binary resolves.** Run `command -v cc-statusline`.
   - If it prints a path → good, continue.
   - If it prints nothing → the plugin is not enabled on PATH. Tell the user to run
     `/plugin` and enable **cc-statusline**, then re-run this skill. Stop here.

2. **Locate settings.** Target `~/.claude/settings.json` (create `{}` if missing).

3. **Back it up.** Copy to `~/.claude/.settings.json.<timestamp>.bak` before editing.

4. **Write the statusLine block**, preserving every other key. Set:
   ```json
   {
     "statusLine": {
       "type": "command",
       "command": "cc-statusline",
       "padding": 0
     }
   }
   ```
   Use the Read + Edit tools to merge this key in. Do not rewrite unrelated settings.
   (`padding: 0` lets the line use the full terminal width; drop it to keep default
   padding.)

5. **Confirm.** Tell the user it's set and that the status line appears on the next
   prompt render (a new session guarantees a clean reload).

## Disable

1. Back up `~/.claude/settings.json` as above.
2. Remove the `statusLine` key (or restore a prior backup).
3. Confirm removal; the default status line returns on next render.

## Customize

The renderer reads optional env vars (set them in your shell profile or via a
wrapper command):

- `CC_SL_FULL`  — glyph for an elapsed unit (default `●`)
- `CC_SL_HALF`  — glyph for the in-progress unit (default `◐`)
- `CC_SL_EMPTY` — glyph for a remaining unit (default `○`)
