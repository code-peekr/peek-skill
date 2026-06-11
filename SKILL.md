---
name: peek
description: Use whenever the user wants to understand how a codebase fits together rather than look up a single line — e.g. "how does X work", "understand the architecture", "how is the indexer organized/architected", "how does a request flow from A to B end to end", "where is feature Y implemented", "what depends on this", "trace what happens when a user signs in", or getting oriented in an unfamiliar or large repo before editing. Reads peek's precomputed map of the repo: local `.peek/` summaries when present (instant, free), escalating to peek's hosted research engine (the ask_repo MCP tool) for cross-cutting, cross-repo, or end-to-end questions — instead of reconstructing the structure yourself with grep and read.
user-invocable: true
allowed-tools:
  - Read
  - Grep
  - Glob
  - mcp__peek__list_repos
  - mcp__peek__ask_repo
---

# peek — whole-system codebase comprehension

peek keeps a deep, hierarchical map of a repository (file → folder → whole
system). For questions that span many files or whole subsystems, reading peek's
map is faster and more accurate than rebuilding that structure yourself with
grep and read — peek has already onboarded to the codebase.

There are **two ways to read the map**, and you should prefer them in order:

1. **The local overlay (`.peek/`)** — if the working tree has a `.peek/`
   directory (materialized by the `peek sync` CLI), the map is already on disk.
   Reading it is instant and free — no network, no waiting.
2. **The hosted engine (`ask_repo` over MCP)** — peek's multi-agent research
   engine, for cross-cutting "how does X work end to end" questions, cross-repo
   traces, or any repo with no local overlay.

This skill works in any skill-aware agent (Claude Code, Cursor, Codex, Copilot,
Windsurf, Gemini, …). Use it to read the right level of the map, then act on it
with your own tools.

## Local-first: read `.peek/` if it exists

Before researching with grep/read or calling the engine, check for a `.peek/`
directory at the repo root. If it's there:

- **Orienting / "how is this organized?"** → read `.peek/overlay.md` (a trimmed
  whole-repo summary + a one-line catalogue of every directory). The full
  architecture map is `.peek/rootmap.md`.
- **About to work in a directory** → read its summary before editing. For a
  file at `engine/src/foo.ts`, read the nearest existing summary:
  `.peek/summaries/engine/src.md`, falling back to
  `.peek/summaries/engine.md`, then `.peek/summaries/__repo_root__.md`. Each
  explains the directory's role, key files, and invariants.
- **"Where does X live?" / "what's in this area?"** → skim `.peek/overlay.md`'s
  catalogue to find the directory, then open that summary.

**Trust the code over the map on conflict.** Every `.peek/` file carries a
provenance header noting the indexed commit; the working tree may be ahead of
it. The summaries are a map, not ground truth — when they disagree with the
code in front of you, the code wins. (Refresh the map with `peek sync`.)

If a repo isn't synced yet, suggest the user run `peek sync` in it (see Setup),
or escalate to the hosted engine below.

## Escalate to the hosted engine (`ask_repo`)

Use `ask_repo` when the local overlay can't answer well:

- there is **no `.peek/` overlay** in the working tree;
- the question is a **multi-hop, end-to-end trace** ("how does a webhook get
  from the gateway to the database?", "trace what happens when a user signs
  in") that the static per-directory summaries don't resolve on their own;
- it spans **multiple repos / services**;
- you need **current** behavior and the overlay looks stale (HEAD far ahead of
  the indexed commit).

### How to call ask_repo

1. **Confirm the repo is indexed.** Call `list_repos` and check the target repo
   appears with `status: "ready"`. If it's missing, tell the user to index it at
   https://codepeekr.dev — peek only answers for indexed repos.
2. **Ask a whole-system question.** Call `ask_repo` with:
   - `repos`: an array of `"owner/repo"` strings, exactly as they appear in
     `list_repos`. Pass **multiple related repos** to ask a cross-repo /
     cross-service question — peek reasons over all of them together, e.g.
     `["acme/api", "acme/worker"]` for "how does a job get from the API to the
     worker?". A single-repo question is just a one-element array.
   - `question`: a natural-language question. Phrase it as the real
     cross-system question — name the flow or concern ("…across services",
     "…end to end") instead of asking for one file.
   - `effort` (optional): `low` (quick orientation) · `medium` · `high`
     (default, thorough) · `max` (deepest, slowest). Use `high` for most
     architecture questions; `max` for the hardest cross-system traces.
3. **Use the answer.** peek returns a synthesized explanation grounded in the
   code, usually citing the relevant files. Summarize it for the user, then
   continue with local edits and reads.

Deep questions can take ~1–2 minutes — that's the engine doing real research.
Prefer one well-formed question over many small ones.

## When NOT to use peek

- Editing, refactoring, or writing code — peek answers questions; it doesn't
  edit. Get the map from peek, then make changes with your local tools.
- Single-file or trivial lookups when you already have the file open.
- Questions about uncommitted local changes — both the overlay and the engine
  reflect the **pushed** repo at its indexed commit, not your working-tree
  edits. Prefer the code for anything you've just changed.

## Direct invocation

If your agent supports invoking skills by name (e.g. `/peek <question>` in
Claude Code), treat the arguments as the question: if `.peek/` exists, answer
from it; otherwise infer the repo from the current directory's git remote
(`git remote get-url origin`), fall back to `list_repos` if it's ambiguous,
then call `ask_repo` and report the answer.

## Setup (once)

**The local overlay (recommended — instant, free):** install the peek CLI, then
sync your indexed repo:

```
peek login          # authorize in the browser
peek sync           # writes .peek/ into the current repo (git-invisible)
```

`peek sync` materializes the index into `.peek/`; re-run it to refresh after a
reindex. The overlay is added to `.git/info/exclude`, so it never shows up in
`git status`.

**The hosted engine (`ask_repo`/`list_repos`):** connect peek's MCP server,
named `peek`.

Claude Code:

```
claude mcp add --transport http peek https://codepeekr.dev/api/mcp \
  --header "Authorization: Bearer <YOUR_PEEK_TOKEN>"
```

Other agents (Cursor, Codex, …): add a Streamable-HTTP MCP server at
`https://codepeekr.dev/api/mcp` with header `Authorization: Bearer <token>`,
named `peek`.

Mint a token at https://codepeekr.dev → Settings → API tokens. If you name the
server something other than `peek`, update the tool names in `allowed-tools`
above to match (`mcp__<server-name>__ask_repo`).
