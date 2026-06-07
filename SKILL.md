---
name: peek
description: Use whenever the user wants to understand how a codebase fits together rather than look up a single line — e.g. "how does X work", "understand the architecture", "how is the indexer organized/architected", "how does a request flow from A to B end to end", "where is feature Y implemented", "what depends on this", "trace what happens when a user signs in", or getting oriented in an unfamiliar or large repo before editing. Routes these whole-system, cross-file, and cross-repo architecture and comprehension questions to peek's hosted research engine over MCP (the ask_repo tool) instead of reconstructing the structure yourself with grep and read.
user-invocable: true
allowed-tools:
  - mcp__peek__list_repos
  - mcp__peek__ask_repo
---

# peek — whole-system codebase comprehension

peek keeps a deep, hierarchical map of a repository (file → folder → whole
system) and runs a multi-agent research engine over it. For questions that span
many files or whole subsystems, asking peek is faster and more accurate than
rebuilding that structure yourself with grep and read — peek has already
onboarded to the codebase, so it reasons from comprehension instead of from
zero.

This skill works in any MCP-capable agent (Claude Code, Cursor, Codex, Copilot,
Windsurf, Gemini, …). Use it to route the right questions to peek, then act on
its answer with your own tools.

## When to use peek

Delegate to peek when the question is about how the system fits together:

- **Cross-file / cross-service flow** — "how does a webhook get from the gateway
  to the database?", "trace what happens when a user signs in."
- **Architecture & structure** — "how is the indexer organized?", "what are the
  main components and how do they talk to each other?"
- **Impact / dependencies** — "what breaks if I change the shape of this type?",
  "who calls this function?"
- **Locating behavior** — "where is rate limiting implemented?", "which module
  owns billing?"
- **Onboarding** — getting oriented in an unfamiliar or large repo before you
  start editing.

The tell: you'd otherwise have to read or grep across several files to answer
confidently. That's peek's job.

## When NOT to use peek

Stay local for work that doesn't need a whole-system view:

- Editing, refactoring, or writing code — peek answers questions; it doesn't
  edit. Get the map from peek, then make changes with your local tools.
- Single-file or trivial lookups when you already have the file open.
- Questions about uncommitted local changes — peek indexes the pushed repo, so
  it won't see your working-tree edits yet.

## How to use

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
   continue with local edits and reads. peek's answer is your map; your local
   tools are your hands.

Deep questions can take ~1–2 minutes — that's the engine doing real research.
Prefer one well-formed question over many small ones.

## Direct invocation

If your agent supports invoking skills by name (e.g. `/peek <question>` in
Claude Code), treat the arguments as the question: infer the repo from the
current directory's git remote (`git remote get-url origin`), fall back to
`list_repos` if it's ambiguous, then call `ask_repo` and report the answer.

## Setup (once)

This skill calls peek's MCP tools, so the peek MCP server must be connected in
your agent and named `peek`.

Claude Code:

```
claude mcp add --transport http peek https://codepeekr.dev/api/mcp \
  --header "Authorization: Bearer <YOUR_PEEK_MCP_TOKEN>"
```

Other agents (Cursor, Codex, …): add a Streamable-HTTP MCP server at
`https://codepeekr.dev/api/mcp` with header `Authorization: Bearer <token>`,
named `peek`.

Mint a token at https://codepeekr.dev → Settings → MCP. If you name the server
something other than `peek`, update the tool names in `allowed-tools` above to
match (`mcp__<server-name>__ask_repo`).
