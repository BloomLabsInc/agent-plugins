# Bloom plugins for coding agents

[Bloom](https://bloom.diy) is a design canvas where people and agents work together. These plugins
wake your agent's conversation when someone in Bloom tags it, or answers a question it asked, even
while the conversation is idle. Install once per computer; after that, every conversation that uses
Bloom listens, with nothing to start in each one.

| Agent | Install | Update |
| --- | --- | --- |
| Claude Code | `claude plugin install bloom --marketplace BloomLabsInc/agent-plugins` | `claude plugin marketplace update bloom && claude plugin update bloom@bloom` |
| Codex | `mkdir -p ~/.bloom && npx -y --prefix ~/.bloom @bloom/cli@latest hooks install codex` (no plugin; see below) | the same command |

`npx @bloom/cli mcp setup claude` and `npx @bloom/cli mcp setup codex` connect Bloom's MCP server
and install the wake hooks in one step. Then:

- **Claude Code** loads a newly installed plugin when a session starts: type `/reload-plugins` in a
  running one, or start a new one.
- **Codex** skips a new hook until you allow it: in Codex Desktop, open **Review hooks** above the
  message box (or **Settings > Hooks**); in the CLI, `/hooks`. Allow Bloom's hooks once. Later
  releases do not ask again. Check with `npx @bloom/cli doctor codex`.

The Codex plugin `bloom-wake` is retired: current Codex never loaded its hooks, and a plugin hook
asks for Trust again on every release. Bloom's CLI now writes the same hooks into your own
`~/.codex/hooks.json` and removes `bloom-wake`.

The plugins run on macOS and Linux (WSL included) with Node 22 or newer.

## What a plugin does

Each plugin holds the agent's hooks, a short POSIX shell launcher (`bin/bloom-hook`), Claude Code's
waiter (`bin/bloom-wait`), and `cli-version`, the `@bloom/cli` release the hooks run. The launcher
runs that release with `npx -y --prefix ~/.bloom @bloom/cli@<version>`, so it never picks up a
project's own copy. For Codex, the CLI copies the same launcher to `~/.bloom/bin/bloom-hook`, with
its pin in `~/.bloom/cli-version`, and names it in `~/.codex/hooks.json` beside your own hooks.

| Hook | When | What it does |
| --- | --- | --- |
| `PostToolUse`, MCP tools only | After the agent calls an MCP tool | When a Bloom answer carries a one-time code for this conversation, swaps it for a token that can only read this conversation's Bloom tags, and hands the conversation to the machine's one watcher. Results without a code end in the shell script, with no network request. |
| `SessionStart` and `Stop` (Claude Code, in the background) | When a session starts, and when Claude finishes a turn | Waits for the conversation's tags and answers. When one arrives it exits with code 2, which wakes the session, and Claude reads it from Bloom. A conversation Bloom never connected stops waiting after two minutes. Nothing runs in `claude -p`. |
| `SessionStart` and `UserPromptSubmit` (Codex) | When a thread opens, and when you write in it | Starts the thread's watcher again if it stopped. Codex Desktop gets each tag steered into the thread; Codex CLI gets it through `codex queue`. |

A code is kept only when the session it names is this conversation's own, and only for Bloom's own
servers. Any other MCP server's result is ignored.

## Network

- **Once, on first use:** `npx` downloads `@bloom/cli` from npm into `~/.bloom`.
- **For connected conversations only:** the CLI talks to the Bloom deployment that issued the code.
  It exchanges the code for its token, renews the token every hour, reads the conversation's tags,
  and sends a heartbeat so Bloom shows the conversation as connected. It never sends your prompts,
  the agent's replies, tool results or files.

## What it stores on your computer

Everything lives in `~/.bloom`: the CLI, the watcher's socket, each connected conversation's link
and token (owner-only files), and two logs, `watch.log` and `plugin.log`. Neither log ever contains
a token or a code.

## Turn it off or uninstall

- **Pause:** set `BLOOM_PLUGIN_DISABLED=1` in the agent's environment, or disable the plugin
  (`claude plugin disable bloom@bloom`).
- **Uninstall:** `claude plugin uninstall bloom@bloom`, or `npx @bloom/cli hooks uninstall codex`.
  Then delete `~/.bloom` if nothing else on the computer uses Bloom's CLI.

## Contributing and security

This repository is published from Bloom's main repository at each `@bloom/cli` release, so a pull
request here is applied there. Report a security problem privately through
[Bloom support](https://bloom.diy/support/chat), not in an issue.

## License

Apache License 2.0; see [LICENSE](LICENSE).
