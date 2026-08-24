---
name: codegraph
description: Use the CodeGraph MCP tools (`mcp__codegraph__*` — search, callers, callees, trace, impact, node, context, explore, files, status) for structural code questions — where a symbol is defined, what calls what, blast radius of a change, signature lookup — instead of grep + read loops.
---

# CodeGraph

When **CodeGraph MCP** is enabled (see `.opencode/opencode.json`), use it for
structural questions. It is a tree-sitter knowledge graph of symbols, edges,
and files.

## When to prefer codegraph over native search

Use codegraph for **structural** questions — what calls what, what would
break, where is X defined, what is X's signature. Use native grep/read only
for **literal text** queries (string contents, comments, log messages) or
after you already have a specific file open.

| Question                                                            | Tool                              |
| ------------------------------------------------------------------- | --------------------------------- |
| "Where is X defined?" / "Find symbol named X"                       | `mcp__codegraph__search`          |
| "What calls function Y?"                                            | `mcp__codegraph__callers`         |
| "What does Y call?"                                                 | `mcp__codegraph__callees`         |
| "How does X reach/become Y? / trace the flow from X to Y"           | `mcp__codegraph__trace`           |
| "What would break if I changed Z?"                                  | `mcp__codegraph__impact`          |
| "Show me Y's signature / source / docstring"                        | `mcp__codegraph__node`            |
| "Give me focused context for a task/area"                           | `mcp__codegraph__context`         |
| "See several related symbols' source at once"                       | `mcp__codegraph__explore`         |
| "What files exist under path/"                                      | `mcp__codegraph__files`           |
| "Is the index healthy?"                                             | `mcp__codegraph__status`          |

## Rules of thumb

- **Answer directly — don't delegate exploration.** For "how does X work" /
  architecture questions, answer with 2-3 codegraph calls:
  `mcp__codegraph__context` first, then ONE `mcp__codegraph__explore` for the
  source of the symbols it surfaces. For a specific **flow** ("how does X
  reach Y") start with `mcp__codegraph__trace` from→to — one call returns the
  whole path with dynamic hops bridged — then ONE `mcp__codegraph__explore`
  for the bodies; don't rebuild the path with `mcp__codegraph__search` +
  `mcp__codegraph__callers`. Codegraph IS the pre-built index, so spawning a
  separate file-reading sub-task/agent — or running a grep + read loop —
  repeats work codegraph already did and costs more for the same answer.
- **Trust codegraph results.** They come from a full AST parse. Do NOT
  re-verify them with grep — that's slower, less accurate, and wastes context.
- **Don't grep first** when looking up a symbol by name.
  `mcp__codegraph__search` is faster and returns kind + location + signature
  in one call.
- **Don't chain `mcp__codegraph__search` + `mcp__codegraph__node`** when you
  just want context — `mcp__codegraph__context` is one call.
- **Don't loop `mcp__codegraph__node` over many symbols** — one
  `mcp__codegraph__explore` call returns several symbols' source grouped in a
  single capped call, while each separate node/Read call re-reads the whole
  context and costs far more.
- **Index lag — check the staleness banner, don't guess a wait.** When a
  codegraph response starts with "⚠️ Some files referenced below were edited
  since the last index sync…", the listed files are pending re-index — Read
  those specific files for accurate content. Files NOT in that banner are
  fresh and codegraph is authoritative for them. `mcp__codegraph__status` also
  lists pending files under "Pending sync".

## If `.codegraph/` doesn't exist

The MCP server returns "not initialized." Ask the user: *"I notice this
project doesn't have CodeGraph initialized. Want me to run `codegraph init -i`
to build the index?"*