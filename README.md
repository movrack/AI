# AI

English | [Français](README.fr.md)

Agents and skills for AI coding assistants, written once and reused in any project.

Each agent is a plain Markdown file: a small header (name, description, tools) and instructions. An agent
never names a project: it reads the rules of the project it works in (`AGENTS.md`, `CLAUDE.md`, `README`,
glossary) and applies them. The instructions are in English; reports are written in English unless your configuration says otherwise.

## Agents

| Agent | Role | Writes | Spoken call |
|---|---|---|---|
| [`john-117`](agents/john-117.md) | Orchestrator (Master Chief of Halo): analyses, splits the work, picks the agent and the model, delegates, checks the evidence, reports | nothing | « run John-117 » |
| [`nemesis`](agents/nemesis.md) | Devil's advocate of specs and plans: contradictions, edge cases, omissions, risks, feasibility; ranked list of flaws | nothing | « run Nemesis » |
| [`mamy`](agents/mamy.md) | Judges whether each screen can be understood without help by an elderly, non-technical user | nothing (the session saves its report) | « run mamy » |
| [`ally`](agents/ally.md) | Accessibility audit, WCAG 2.2 AA: keyboard, screen reader, measured contrast, touch targets, zoom, motion | nothing (the session saves its report) | « run Ally » |
| [`stryker`](agents/stryker.md) | Mutation testing (mutmut for Python, StrykerJS for JS/TS); on request, adds tests to kill surviving mutants | its report; tests in kill mode only | « run Stryker » |
| [`heimdall`](agents/heimdall.md) | Delivery gatekeeper: quality level, tests, build artifacts, versions, translations, glossary, leaks to a public repository, hygiene, docs | its report only | « run the gatekeeper » |
| [`iris`](agents/iris.md) | CSS and visual consistency: duplicated patterns styled differently, tokens, class-name collisions, light and dark themes, contrast; on request, harmonises styles into a shared stylesheet | its report; styles only in harmonise mode | « run Iris » |
| [`c3po`](agents/c3po.md) | Translation checker: every string exists in every language (keys, placeholders, plurals, files) and means the same thing everywhere (meaning, numbers, negations, glossary terms); on request, fixes translation files only | its report; translation files only in fix mode | « run C-3PO » |
| [`parker`](agents/parker.md) | Photographer: starts the dev instance with stable test data and shoots every screen (small and large, light and dark) | screenshots and an index | « run the photographer » |

All of them work **on request only**, never touch a real environment, never commit or push, and never change
production code (`stryker` adds tests only when asked to kill mutants).

## Reports

Every agent that produces a result writes it to `reports/ai/<agent>/YYYY-MM-DD-<subject>.md` in the project
being reviewed (`reports/` is git-ignored). `parker` writes its screenshots and an `INDEX.md` to
`reports/ai/parker/YYYY-MM-DD/`. `john-117` writes no file by default: it consolidates the reports of the agents
it launches. The reports follow the same shape (scope and context, ranked findings, top 5, what is clear and must
not be touched) and the same severity scale: **Blocker / Major / Minor**. `mamy` and `ally` give a checklist
that the session can turn into a tracking issue; `heimdall` gives a verdict (CAN PASS / CAN PASS WITH RESERVES /
DOES NOT PASS) and a checklist.

## Shared guidelines

All agents follow the same "Ground rules" block: on request only, read the project rules first, never touch a real
environment (no commit, no push, no deployment), evidence over promises ("not verified" when unproven), no secrets
or personal data, never mention an assistant or an AI in anything copied into a repository, and the same severity
scale. Reports are in English unless the user's configuration or request says otherwise. Each file starts with a `# Name:` comment that explains the name.

## Agent file format

```markdown
---
name: my-agent                  # required, same as the file name, lowercase and dashes
description: When to use it.    # required, one line
tools: read, search, execute, edit, mcp:chrome-devtools/take_screenshot   # or * for all tools
model: inherit                  # optional
---
Instructions of the agent (Markdown).
```

Generic tools: `read`, `search`, `edit`, `execute`, `web`, `agent`, `mcp:<server>/<tool>`. Map them to your
assistant when you install an agent (for example `read` → `Read`, `search` → `Grep`/`Glob`, `execute` →
`Bash`, `edit` → `Edit`/`Write`, `mcp:chrome-devtools/take_screenshot` → `mcp__chrome-devtools__take_screenshot`).

## Install

**A plain symlink is not enough.** The `tools` line of these files uses neutral names (`read`, `search`,
`execute`, `edit`, `mcp:<server>/<tool>`), while each assistant wants its own (for example `Read`, `Grep`/`Glob`,
`Bash`, `Edit`/`Write`, `mcp__<server>__<tool>` for Claude Code), and some use another file format altogether (TOML
for Codex). An agent symlinked as it is would end up with tool names the assistant does not know.

A tested script is bundled in the `install-agents` skill. Three ways to install:

1. **Run the skill or the script (recommended).** Clone this repository (for example to `~/Dev/AI`), then run
   `python3 skills/install-agents/scripts/sync_agents.py --dry-run` to see the plan, and the same command without
   `--dry-run` to apply it (or ask your assistant to run the `install-agents` skill). Restart the assistant.
   Re-run after each `git pull`; the generated files are never edited by hand.
2. **Write your own script** for an assistant the script does not cover (next section).
3. **By hand, one agent.** Copy the file into your assistant's agents folder (for Claude Code:
   `~/.claude/agents/<name>.md` for all your projects, or `.claude/agents/<name>.md` inside one project) and
   translate the `tools` line yourself using the table below.

Browser-driven agents (`mamy`, `ally`, `parker`, and `iris` for the visual checks) need the Chrome DevTools MCP server.

## Sync script

### Use the bundled script

```
python3 skills/install-agents/scripts/sync_agents.py --dry-run      # show the plan, write nothing
python3 skills/install-agents/scripts/sync_agents.py                # install into the detected assistants
```

Options: `--assistants claude,codex,gemini,copilot` (default: those whose config folder exists), `--scope user|project`
with `--project PATH` for project-level files, `--only NAME[,NAME]`, `--source PATH`. It is idempotent, never
deletes anything and backs up any foreign file under `~/.config/ai-agents/_backup/`. Tests:
`python3 -m unittest discover skills/install-agents/tests -v`. The file formats of Codex, Gemini CLI and Copilot CLI
follow the table below; check them against the current documentation of each assistant.

### Write your own sync script (for an assistant not covered)

What the script has to do:
- **Read** every `agents/*.md`: a header between two `---` lines (`name`, `description`, `tools`, optional `model`;
  ignore the `# ...` comment lines) and the body (the instructions). Check that `name` equals the file name.
- **Translate the `tools` line** with this table (neutral name on the left):

| Neutral | Claude Code | Gemini CLI | Copilot CLI |
|---|---|---|---|
| `read` | `Read` | `read_file`, `read_many_files` | `read` |
| `search` | `Grep`, `Glob` | `grep_search`, `glob`, `list_directory` | `search` |
| `edit` | `Edit`, `Write` | `replace`, `write_file` | `edit` |
| `execute` | `Bash` | `run_shell_command` | `execute` |
| `web` | `WebFetch`, `WebSearch` | `web_fetch`, `google_web_search` | `web` |
| `agent` | `Agent` | none | `agent` |
| `mcp:<server>/<tool>` | `mcp__<server>__<tool>` | `mcp_<server>_<tool>` | `<server>/<tool>` |
| `*` (or no line) | no `tools` field = all tools | same | same |

- **Write one file per assistant**, in the format and the place it expects (check each assistant's documentation; this
  is what the author's script writes):

| Assistant | File | Header fields | Body |
|---|---|---|---|
| Claude Code | `~/.claude/agents/<name>.md` (or `<project>/.claude/agents/`) | `name`, `description`, `tools` (comma-separated), `model` | the instructions |
| Codex | `~/.codex/agents/<name_with_underscores>.toml` | `name`, `description` | `developer_instructions = """..."""` |
| Gemini CLI | `~/.gemini/agents/<name>.md` | `name`, `description`, `kind: local`, `tools` (list) | the instructions |
| Copilot CLI | `~/.copilot/agents/<name>.agent.md` (or `<project>/.github/agents/`) | `name`, `description`, `tools` (array), `user-invocable: true` | the instructions |

- **Be safe**: idempotent (do not rewrite an identical file), a `--dry-run` option, a "generated file, do not edit"
  line at the top of each output, a backup of any existing file it did not generate before replacing it, and never
  delete anything. Parse the header in a real language (Python standard library), not with `sed`.

#### A prompt to give to your AI assistant

> Write a Python 3 script (standard library only) named `sync_agents.py` that generates the agent files of my AI
> assistants from the Markdown sources in `~/Dev/AI/agents/*.md`.
>
> Source format: a header between two `---` lines with `name` (must equal the file name without `.md`),
> `description`, `tools` (comma-separated neutral names: `read`, `search`, `edit`, `execute`, `web`, `agent`,
> `mcp:<server>/<tool>`, or `*` for all tools; absent means all) and optional `model`; lines starting with `#` in the
> header are comments to ignore; then the body with the instructions.
>
> Generate, for each agent, the files of the assistants I use: [LIST: Claude Code, Codex, Gemini CLI, Copilot CLI].
> Translate the `tools` line with this mapping: [PASTE THE TWO TABLES OF THE README SECTION "What the script has to
> do"]. Follow the current documentation of each assistant for the file format and location. Each generated file
> starts with a comment saying it is generated by this script and must not be edited.
>
> Requirements: `--dry-run` shows what would be created or updated without writing; idempotent (an identical file is
> left alone, reported as unchanged); if a target file exists, differs and was not generated by this script, copy it
> to `~/.config/ai-agents/_backup/<timestamp>/` before replacing it; never delete anything; fail with a clear
> message on a missing or invalid header; print one line per file written. Also accept `--project <path>` to write the
> project-level copies (`.claude/agents/`, `.github/agents/`) in a repository.
>
> Then show me how to run it and how to test it on one agent.

Run the script with `--dry-run` first, read what it would write, then run it for real and restart the assistant.

## Skills

| Skill | Use |
|---|---|
| [`install-agents`](skills/install-agents/SKILL.md) | Install or update the agents of this repository into Claude Code, Codex, Gemini CLI or Copilot CLI, with a dry run, backups and tests |

## Contributing

The owner is the only one who pushes directly to `main`. Requests go through issues and changes through pull requests: see [CONTRIBUTING.md](CONTRIBUTING.md) and [SECURITY.md](SECURITY.md).

## License

[MIT](LICENSE)
