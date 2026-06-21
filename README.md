# cc-statusline

A two-line [Claude Code](https://code.claude.com) status line, written in **pure
POSIX shell + awk** — no `node`, `bun`, or `jq` required. It reads everything from
the status JSON Claude Code pipes on stdin, so it runs as a single fast process and
works on a fresh native-install machine.

```
Opus 4.8 [xhigh] ~ ⎈ prod(api) feat/x (+3)
20:45:17 | ⛁ ███░░░░░░░ 32% | 5h ◔○○○○ 6% ↻2h15m | 7d ●○○○○○○ 15% ↻5d14h │ $0.61 ⏱ 2m
```

**Line 1** — model, optional reasoning effort, working dir, optional kubectl
context, git branch + uncommitted-change count.

**Line 2** — clock (sits under the model name), ⛁ context-window bar, then the 5-hour
and 7-day rate-limit windows, then session cost and duration.

## The rate-limit dots

Each window is a row of dots — **5 dots for the 5-hour window, 7 for the 7-day**:

- **Fill = usage %** consumed in the window — the dots are a 5- or 7-segment gauge
  of how much of the limit you've spent. The in-progress dot fills by quarter
  in `◔ ◑ ◕` steps (each part = ¼ of a dot, shown once you've fully consumed it) for
  sub-dot precision — a real win on the 7-day, where each dot is ~14%; unspent are `○`.
- **Color = burn pace** — your usage vs. how far through the window you are:
  - **blue** — used ≤ elapsed: on or under pace, plenty left
  - **yellow** — used ≤ 1.5× elapsed: running tight
  - **red** — used > 1.5× elapsed: burning too fast to last the window
  - **floor** — below `CC_SL_PACE_FLOOR` % used (default 5) the color stays blue regardless of pace: a near-empty window can't be exhausted, and this keeps a just-reset window (elapsed ≈ 0) from false-flagging red
- `↻2h15m` is the time until that window resets.

So `5h ●●◑○○ 50%` red means half the 5-hour limit spent but you're well under halfway
through the window — burning too fast. `5h ●○○○○ 20%` blue: 20% spent, on pace.

## Install

```sh
claude plugin marketplace add AndrewP-GH/cc-statusline
claude plugin install cc-statusline@cc-statusline
```

Plugins can't set the main status line automatically, so enable it once:

```
/cc-statusline:setup
```

That writes a `statusLine` entry into your `~/.claude/settings.json` (backing up the
file first). The line appears on the next prompt render. To remove it, run
`/cc-statusline:setup` and ask to disable.

### Manual alternative

If you'd rather not use the skill, add this to `~/.claude/settings.json` with the
**absolute path** to the script. The `statusLine` command does not get the plugin's
`bin/` on `PATH`, so a bare `cc-statusline` would render an empty line. Resolve the
path first:

```sh
ls ~/.claude/plugins/marketplaces/*/plugins/cc-statusline/bin/cc-statusline
```

then:

```json
{
  "statusLine": { "type": "command", "command": "bash /ABSOLUTE/PATH/TO/cc-statusline", "padding": 0 }
}
```

## Requirements

- A POSIX shell, `awk`, and `date` — present on macOS and Linux out of the box.
- `git` and `kubectl` are **optional**; their segments appear only when the tool is
  installed and relevant.

## Customize

Override the dot glyphs and the pace-color floor via environment variables:

| Var | Default | Meaning |
|-----|---------|---------|
| `CC_SL_FULL`  | `●` | spent segment |
| `CC_SL_Q1`    | `◔` | in-progress dot, ~¼ |
| `CC_SL_HALF`  | `◑` | in-progress dot, ~½ |
| `CC_SL_Q3`    | `◕` | in-progress dot, ~¾ |
| `CC_SL_EMPTY` | `○` | unspent segment |
| `CC_SL_PACE_FLOOR` | `5` | used % below which a window's pace color stays blue |

## How it works

Claude Code sends a JSON blob on stdin including `model`, `workspace`,
`context_window.used_percentage`, `cost`, and `rate_limits.{five_hour,seven_day}`
with `used_percentage` and a Unix-epoch `resets_at`. `bin/cc-statusline` parses it in
one awk pass and renders both lines with integer math — no floating point, no date
parsing libraries.

## License

MIT — see [LICENSE](LICENSE).
