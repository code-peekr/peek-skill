# peek — agent skill

[![skills.sh](https://www.skills.sh/b/code-peekr/peek-skill)](https://www.skills.sh/code-peekr/peek-skill)

**Whole-system codebase comprehension for your AI coding agent.**

[peek](https://codepeekr.dev) keeps a deep, hierarchical map of your
repositories. This skill teaches your agent *when and how* to read that map —
cross-file flows, architecture, "how does X work end to end", "where does Y
live" — instead of grepping around to rebuild the structure itself.

It reads the map **local-first**: if you've run `peek sync` in a repo, the map
lives on disk under `.peek/` and the agent reads it instantly and for free;
otherwise (or for cross-cutting / cross-repo questions) it escalates to peek's
hosted research engine over MCP (`ask_repo`).

Works across **Claude Code, Cursor, Codex, GitHub Copilot, Windsurf, Gemini**,
and other skill-aware agents. One `SKILL.md`, every platform.

## Install

```bash
npx skills add code-peekr/peek-skill
```

Or install manually by cloning into your agent's skills directory:

| Agent | Global | Project |
|---|---|---|
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| Cursor | `~/.cursor/skills/` | `.cursor/skills/` |
| Codex | `~/.agents/skills/` | `.agents/skills/` |
| GitHub Copilot | — | `.github/copilot/skills/` |
| Windsurf | `~/.windsurf/skills/` | `.windsurf/skills/` |

```bash
git clone https://github.com/code-peekr/peek-skill ~/.claude/skills/peek
```

## Setup

### Local overlay (recommended — instant, free)

Install the [peek CLI](https://codepeekr.dev/docs), then sync an indexed repo so
its map lands on disk:

```bash
peek login          # authorize in the browser
peek sync           # writes .peek/ into the current repo (git-invisible)
```

Re-run `peek sync` to refresh after a reindex. The overlay is added to
`.git/info/exclude`, so it never appears in `git status`.

### Hosted engine (`ask_repo` / `list_repos` over MCP)

For repos without a local overlay, and for cross-cutting / cross-repo questions,
the skill calls peek's MCP tools. Connect the server once, named `peek`:

```bash
claude mcp add --transport http peek https://codepeekr.dev/api/mcp \
  --header "Authorization: Bearer <YOUR_PEEK_TOKEN>"
```

Mint a token at **https://codepeekr.dev → Settings → API tokens** (the same
token the CLI uses). Other agents: add the same Streamable-HTTP MCP endpoint
with the bearer header.

## What peek is good at

Cross-service flows, architecture overviews, impact/dependency questions, "where
does X live", and onboarding to an unfamiliar or large repo — anything where
you'd otherwise read or grep across many files. peek answers from a precomputed
map plus live reads; it doesn't edit code, and it indexes the pushed repo (not
your uncommitted working tree).

## Links

- **Web app:** https://codepeekr.dev
- **Docs:** https://codepeekr.dev/docs/agent-skill
