# Rules

This file defines how AI agents should work with project memory, source material, and durable notes across all projects.

The goal is not to build a giant autonomous wiki. The goal is to preserve useful context with low overhead so future sessions in Cursor, Codex, Claude, and similar tools can start from maintained project memory instead of rediscovering everything from scratch.

## Core Principle

Treat the project memory layer as a maintained derivative of the real project.

- The repository, code, docs, tickets, and source files are the source of truth.
- The memory layer summarizes, organizes, and cross-links durable knowledge from those sources.
- The agent may freely update the memory layer.
- The agent must not silently treat the memory layer as more authoritative than the underlying sources.

## Project Structure

Every project should contain this folder structure:

```text
.ai/
  raw/
  wiki/
    index.md
    log.md
    architecture.md
    decisions.md
    current-state.md
    commands.md
    glossary.md
    open-questions.md
```

### Folder meanings

- `.ai/raw/`: immutable source material gathered for the project. Examples: clipped articles, external docs, meeting notes, specs, screenshots, exported tickets, transcripts.
- `.ai/wiki/`: maintained markdown memory for the project. This is the durable working context for agents.
- `.ai/wiki/index.md`: the content index of the wiki with one-line descriptions of pages.
- `.ai/wiki/log.md`: append-only chronological record of ingests, analyses, decisions, and maintenance passes.

If the project is small, keep the wiki small. Do not create pages just to satisfy the structure. Only keep pages that are useful.

## Agent Startup Behavior

At the start of any non-trivial task, the agent should:

1. Read `.ai/wiki/index.md`.
2. Read the 2 to 5 most relevant linked wiki pages.
3. Read the underlying source files or code before making claims.
4. Prefer updating existing canonical pages instead of creating duplicate notes.

If `.ai/wiki/` does not exist yet, the agent should create the minimal structure and start small.

## What Belongs In The Wiki

Store durable, reusable knowledge:

- architecture explanations
- domain concepts and glossary terms
- setup and recurring commands
- decision records and tradeoffs
- current project state
- debugging discoveries worth reusing
- recurring failure modes
- integration quirks
- high-value summaries of external sources
- answers that required real synthesis and may be useful again

Do not store transient noise:

- raw stream-of-consciousness chat dumps
- large copied code blocks unless essential
- speculative claims without source grounding
- repetitive task-by-task chatter
- trivial one-off answers with no future value

## Canonical Pages

Use these pages as the default home for durable knowledge:

- `architecture.md`: components, boundaries, data flow, important abstractions
- `decisions.md`: decision log with date, rationale, alternatives, consequences
- `current-state.md`: active priorities, current constraints, notable ongoing work
- `commands.md`: setup, test, build, deploy, troubleshooting commands
- `glossary.md`: project-specific terms, entities, acronyms, shorthand
- `open-questions.md`: unresolved issues, data gaps, follow-up investigations

Create additional pages only when a topic becomes substantial enough to deserve its own canonical document.

## Ingest Workflow

When a new external source is added to `.ai/raw/`, the agent should:

1. Read the source.
2. Create or update a concise summary in `.ai/wiki/`.
3. Update any relevant canonical pages.
4. Add or update the source entry in `index.md`.
5. Append an entry to `log.md`.
6. Flag contradictions or uncertainty explicitly.

The agent should prefer one source at a time unless asked to batch process.

## Query Workflow

When answering a project question, the agent should:

1. Search the wiki first.
2. Read the relevant pages.
3. Verify important claims against the codebase or raw sources.
4. Answer with references to source files, wiki pages, or dates when appropriate.
5. If the answer creates durable knowledge, file it back into the wiki.

The agent should not rely only on prior wiki summaries when the underlying code or source material is available.

## End-Of-Task Distillation

At the end of any meaningful task, the agent should distill what matters.

Distill only if the task produced durable knowledge such as:

- a new architectural understanding
- a non-obvious fix
- a repeated debugging pattern
- a new decision or tradeoff
- a new command or setup step
- an answer worth reusing later

When distilling, the agent should:

1. Update the relevant canonical page or create a narrowly scoped new page.
2. Update `index.md` if pages changed or were added.
3. Append a dated summary entry to `log.md`.

## Lint And Maintenance

Periodically, the agent should run a wiki maintenance pass and look for:

- stale claims superseded by newer facts
- contradictions between pages
- duplicate pages covering the same topic
- orphan pages with no inbound references
- important concepts mentioned but undocumented
- commands that no longer work
- open questions that were actually resolved

The agent should suggest cleanup, but should not rewrite history silently. When a claim becomes outdated, mark it as superseded or update it with the new date and source.

## Trust And Safety Rules

The agent must follow these rules:

- Never invent facts about code, people, tickets, or sources.
- Prefer explicit uncertainty over confident guessing.
- Keep citations or references close to non-obvious claims.
- Preserve dates when a fact may change over time.
- Treat summaries as derivatives, not ground truth.
- Do not silently delete important disagreement or contradiction.
- If the wiki and the code disagree, trust the code and update the wiki.
- If two sources disagree, record the disagreement and cite both.

## Writing Style For Wiki Pages

Wiki pages should be easy for humans and agents to scan.

- Use short sections.
- Prefer bullets over long prose when possible.
- Keep pages concise and topical.
- Link related pages with relative markdown links.
- Include dates on time-sensitive statements.
- Add a short summary near the top of each substantial page.

Recommended source section format:

```md
## Sources

- [Source name](../raw/source-file.md) - imported 2026-04-17
- `src/module/file.ts`
- Issue #123
```

## Index Format

`index.md` should be a compact catalog of wiki pages.

Recommended format:

```md
# Index

## Canonical

- [architecture.md](./architecture.md): System shape, major components, and boundaries.
- [decisions.md](./decisions.md): Key decisions, rationale, and consequences.
- [current-state.md](./current-state.md): Active work, risks, and immediate priorities.

## Topic Pages

- [auth-flow.md](./auth-flow.md): Login flow, token lifecycle, and edge cases.
```

## Log Format

`log.md` should be append-only and easy to parse.

Recommended format:

```md
# Log

## [2026-04-17] ingest | OAuth provider docs
- Added source summary.
- Updated `architecture.md` and `auth-flow.md`.
- Noted mismatch between provider docs and current implementation.

## [2026-04-17] task-distill | Fix flaky signup test
- Documented root cause in `current-state.md`.
- Added retry guidance to `commands.md`.
```

## Cross-Tool Usage

This file is vendor-neutral. Tool-specific config files should point to it and follow it.

Recommended mapping:

- Codex: reference this file from `AGENTS.md`.
- Claude Code: reference this file from `CLAUDE.md`.
- Cursor: reference this file from project rules or workspace instructions.

Recommended instruction to include in each tool-specific file:

```md
Follow the project memory workflow defined in `rules.md`.
Before non-trivial work, read `.ai/wiki/index.md` and the most relevant linked pages.
Treat the repository and `.ai/raw/` as source of truth.
Update `.ai/wiki/` when durable knowledge is created.
Prefer updating canonical pages over creating duplicate notes.
```

## Global And Local Memory

Use this project file for project-specific memory only.

If you also maintain a personal global knowledge base outside the repo, use it for:

- personal preferences
- reusable workflows
- tool-specific habits
- writing style
- life admin and personal systems
- cross-project lessons

Project memory should stay focused on the project.

## Default Operating Rule

If unsure whether something belongs in the wiki, ask:

Will future me or a future agent benefit from seeing this without having to rediscover it from chat, code archaeology, or memory?

If yes, store it in the wiki.
If no, leave it in the conversation and move on.