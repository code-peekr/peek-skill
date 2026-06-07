# peek — agent skill

[![skills.sh](https://www.skills.sh/b/code-peekr/peek-skill)](https://www.skills.sh/code-peekr/peek-skill)

**Whole-system codebase comprehension for your AI coding agent.**

[peek](https://codepeekr.dev) keeps a deep, hierarchical map of your
repositories and answers whole-system questions over MCP. This skill teaches
your agent *when* to ask peek — cross-file flows, architecture, "how does X work
end to end" — instead of grepping around to rebuild the structure itself.

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

## Prerequisite: connect the peek MCP server

The skill calls peek's two MCP tools (`list_repos`, `ask_repo`). Connect the
server once, named `peek`:

```bash
claude mcp add --transport http peek https://codepeekr.dev/api/mcp \
  --header "Authorization: Bearer <YOUR_PEEK_MCP_TOKEN>"
```

Mint a token at **https://codepeekr.dev → Settings → MCP**. Other agents: add
the same Streamable-HTTP MCP endpoint with the bearer header.

## What peek is good at

Cross-service flows, architecture overviews, impact/dependency questions, "where
does X live", and onboarding to an unfamiliar or large repo — anything where
you'd otherwise read or grep across many files. peek answers from a precomputed
map plus live reads; it doesn't edit code, and it indexes the pushed repo (not
your uncommitted working tree).

## Links

- **Web app:** https://codepeekr.dev
- **Docs:** https://codepeekr.dev/docs/agent-skill
