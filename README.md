# @gicm/context

Skill-aware context injection: score SKILL.md files against each message and inject only the best matches within a token budget.

Agents that load every skill into every prompt spend tokens on material the current task never touches. This package indexes your skills once, scores them against each incoming message and builds a prompt from the top few that fit the budget. It is for developers who keep a folder of SKILL.md files and want a small, dependency-light selector in front of them. Scoring is lexical: no embeddings and no network.

Part of [gICM](https://github.com/0motionguy/gICM), a family of OpenClaw agent packages. API reference and examples: [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md).

## What it does

- **Index:** `indexSkills([{ content, path }])` parses SKILL.md frontmatter (`name`, `version`, `description`, `author`, `tags`) into entries with a token estimate (characters divided by 4). Files without frontmatter are skipped.
- **Score:** each query word of three letters or more adds 10 for a name match, 5 for a tag match, 3 for a description match and 1 per body occurrence (capped at 3 per word). The boost values are configurable.
- **Inject:** `inject(message)` ranks the scored skills and takes them in order until `maxSkills` (5 by default) or `maxTokens` (8,000 by default) is reached. A skill that would overflow the budget is skipped in favour of a smaller one.
- **Result:** the chosen skills with their match reasons, tokens used, the fraction of budget spent and a formatted `## Loaded Context` prompt.
- **Manage:** add, get, remove and clear helpers, hit counts per skill through `getStats()`, and events `context:indexed`, `context:injected` and `context:budget-exceeded`.

## Example

```ts
import { ContextEngine } from "@gicm/context";

const engine = new ContextEngine({ maxTokens: 8000, maxSkills: 5 });
engine.indexSkills([{ content: reactSkillMd, path: "skills/react/SKILL.md" }]);

const { skills, totalTokens, prompt } = engine.inject("Build a React dashboard");
```

## How it's built

TypeScript (ESM) bundled with tsup and tested with vitest; zod validates the config. `parser.ts` is a small line-based frontmatter reader (single-line fields, tags written as `[a, b]`, LF line endings), `scorer.ts` does the keyword scoring and `context-engine.ts` handles the budget and prompt.

## Repository layout

- `src/`: `context-engine`, `scorer`, `parser`, `types`
- `src/__tests__/`: vitest suite (30 test cases)
- `skill/SKILL.md`: the OpenClaw skill definition

## Run it locally

```bash
pnpm install
pnpm build   # tsup -> dist/
pnpm test    # vitest (add --run for a single pass)
```

## Status

Beta. Version 1.0.0, a single extracted release (February 2026) with a test suite. The `excludePatterns` option is accepted but not used yet.
