---
name: authoring-opencode
description: Use ONLY when writing or editing opencode config — `.opencode/SKILL.md` files, agent prompts, `AGENTS.md` instructions, or `opencode.json` — to keep guidance generic and not cite names, paths, or domain jargon from the current workspace.
---

# Authoring opencode config

Files under `.opencode/` (`SKILL.md`, agent prompts, `AGENTS.md`,
`opencode.json`) are **cross-project guidance**. Do not cite symbols, paths,
models, or business terms from the current workspace.

| Do                                                                         | Don't                                                               |
| -------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| Universal patterns and heuristics                                          | Real file paths, Prisma models, or domain jargon from one repo      |
| Invented minimal examples (`User`, `findById`)                             | Copy-paste names from the project you were in when writing the rule |
| Stack-specific rules only when the topic requires it (e.g. Prisma, NestJS) | "How we do X in this app" disguised as a global rule                |

**Examples:** use short, self-contained snippets with generic types and names.
If a rule needs a table or mapping, use placeholder labels — not import
headers or enums from a real product.

**Scope:** project-only conventions belong in that repo's rules or
`AGENTS.md`, not in shared user-level rules.

## SKILL.md frontmatter

opencode skills require frontmatter with `name` (matches folder) and
`description` (one sentence: what + when, front-loaded trigger keywords). The
description is effectively required — skills without it never surface. Write
in third person ("Use when…", "Use ONLY when…"). Keep the body focused;
always-on material belongs in `AGENTS.md`, not duplicated into a skill.