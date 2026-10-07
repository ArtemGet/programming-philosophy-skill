---
name: programming-philosophy-skill
description: >-
  Turn a codebase plus its documentation into a high-quality, enforceable
  programming / testing / CI-CD / engineering skill (an Agent Skills package).
  Use when the user points you at repositories, docs, books, specs or articles
  and wants an agent skill or coding guidelines built from them. Runs an intake
  interview, acquires sources safely, extracts knowledge with a multi-agent
  pipeline, mines real code for conventions, distills a prioritized rulebook,
  and authors a lean skill — with legal/privacy redaction so the finished skill
  never references downloaded sources, only approved links.
license: MIT
---

# Programming Philosophy — studying a codebase to author a skill

A reusable methodology for turning **external knowledge** (repositories, docs,
books, specs, articles) into a **portable, enforceable Agent Skill** about how to
program in a given stack and philosophy. Language- and philosophy-agnostic: use it
for Java/EO, Rust, Go, Python, frontend, game dev, whatever the user brings.

The skill you produce is lean; the *work* of producing it is a pipeline of
subagents. This file is the map; `references/` has the depth.

## When to use

- The user gives one or more **repository URLs and/or docs** and wants an agent
  skill, coding standards, or "learn this project's style" guidance produced.
- The user wants a **CI/CD + testing + process** skill for a stack.
- The user asks you to study a philosophy/framework and write rules others (or
  agents) will follow.

**When not to use:** editing an existing skill (just edit it); one-off tasks; when
the user already knows exactly the rules and only wants them transcribed.

## What you produce

An Agent Skills package (`SKILL.md` + `references/` [+ optional `knowledge/`])
that an agent can load on demand: a lean body (workflow, non-negotiable rules,
Definition of Done, reference map) plus progressive-disclosure depth. Every
`[MUST]` rule maps to something that can fail a build, lint, test, or review gate.

## FIRST RUN — always start here

On first invocation, do NOT start cloning or writing. Do this:

1. **State intent, briefly (≤6 lines).** Tell the user what you are about to do and
   what the end artifact will be, so they can redirect early.
2. **Interview the user.** Ask for the inputs you cannot guess. Use the
   environment's question tool. Minimum set:
   - target **language(s)/stack** and the **philosophy/conventions** to capture;
   - **sources available**: repository URLs, docs, books/PDFs, specs, articles —
     and for each, is it **public or private**?
   - **scope**: coding only? + testing? + CI/CD? + release? + project process? + docs?
   - **target runtime(s)** (opencode / Claude Code / Cursor / Codex / generic) and
     desired output layout;
   - **links policy**: which links may appear in the produced skill, which must be
     omitted (see `references/06-legal-privacy-and-provenance.md`);
   - depth: lean "balanced" skill vs. a large "max" knowledge base for RAG.
3. **Suggest what is missing.** Proactively name inputs that would improve the
   result: a canonical "manifesto"/intro page, a reference implementation, style
   guides, CI configs, issue/PR conventions, books, example code, a glossary.
4. **Write a short plan and get confirmation** before acquiring anything. Save the
   intake as `analysis/INTAKE.md`.

Only after confirmation proceed to acquisition.

## The pipeline (high level)

| Phase | What | Reference |
|---|---|---|
| 0 | Intake interview + confirmed plan | `references/01-intake-and-discovery.md` |
| 1 | Acquire sources safely into a gitignored staging area | `references/02-source-acquisition.md` |
| 2 | Build a shared `BRIEF.md` (canon + fixed section structure) | `references/03-knowledge-extraction.md` |
| 3 | Extract knowledge: 1 agent for the primary source; N parallel agents for secondary sources; 1 merge agent | `references/03-knowledge-extraction.md` |
| 4 | Mine **real code**: M parallel agents per repo/module | `references/04-codebase-analysis.md` |
| 5 | Distill a prioritized rulebook ([MUST]/[SHOULD]/[NICE]) | `references/03`, `references/05` |
| 6 | Author the skill: lean `SKILL.md` + `references/` | `references/05-skill-authoring.md` |
| 7 | Validate + redact + deploy to runtime layouts | `references/05`, `references/06` |

Orchestration rules live in `references/07-multi-agent-orchestration.md`.

## Multi-agent rules (the core of the method)

- **Shared brief first.** Every agent receives the same `BRIEF.md` (principles,
  vocabulary, the FIXED section structure) so outputs are mergeable.
- **Fixed section ids.** Findings are keyed to stable ids (`§1`…`§N`); each agent
  adds findings like `- **<prefix>.<n> <title>** -> add to §<k>: <rule>. Why: … Example: …`.
- **Isolation.** Agents never edit another agent's output; each writes its own file.
- **Copy-then-insert merge.** A merge agent copies the primary output verbatim into
  a new file and appends additions; it never rewrites the base.
- **Bounded inputs.** Give each agent a bounded source set (a batch of articles, one
  repo); require it to read fully but summarize, not dump. Respect a max concurrent
  agent budget (e.g. ≤10).
- **Verify between phases** before starting the next.

## Output purity & legal (non-negotiable)

The produced skill must be **clean of the sources you downloaded**:

- **No references to staged source folders** (`sources/`, `code/`, `articles/`,
  `books/`, `work/`) and **no embedded private/proprietary code** — only external
  **links**, and only links the user explicitly approved.
- **Ask the user** which links are allowed (public/official) vs forbidden (private,
  paywalled, NDA, personal). Default: omit unless approved.
- **Paraphrase** third-party text; never paste large verbatim copyrighted passages.
- **Never leak** private repo names, absolute paths, internal URLs, org names, or secrets.
- Keep acquired sources **out of the published skill** (gitignored and/or deleted
  after distillation); record a provenance/license note.
- Run the **redaction checklist** before finishing (grep the skill for folder names,
  absolute paths, private URLs, secrets).

Full detail: `references/06-legal-privacy-and-provenance.md`.

## Definition of Done (for the produced skill)

- [ ] `SKILL.md` frontmatter parses; `name` matches the folder; `description` states
      capabilities **and** triggers.
- [ ] Body is lean (≈≤500 lines / ~5k tokens); depth is in `references/`.
- [ ] Every `[MUST]` rule is enforceable (mapped to a build/lint/test/review gate).
- [ ] Source disagreements are resolved with a stated default + exception.
- [ ] No staged-source folder names, private paths, private URLs, or secrets remain.
- [ ] Cross-links resolve; reference map is complete; sizes within budget.
- [ ] Deployed to the target runtime layout(s) and loadable (validated).
- [ ] User confirmed scope; the skill covers exactly that scope.

## Reference map

| Topic | File |
|---|---|
| First-run interview and planning | `references/01-intake-and-discovery.md` |
| Acquiring/staging sources safely | `references/02-source-acquisition.md` |
| Multi-agent knowledge extraction + merge | `references/03-knowledge-extraction.md` |
| Mining real repositories for conventions | `references/04-codebase-analysis.md` |
| Authoring + validating the final skill | `references/05-skill-authoring.md` |
| Legal, privacy, links policy, redaction | `references/06-legal-privacy-and-provenance.md` |
| Multi-agent orchestration mechanics | `references/07-multi-agent-orchestration.md` |
| Reusable templates | `templates/BRIEF-template.md`, `templates/finding-format.md`, `templates/skill-skeleton.md` |

## Guardrails (do not)

- Do not start acquiring sources before the intake is confirmed.
- Do not let any agent edit another agent's output.
- Do not embed downloaded source code, private paths, or unapproved links in the skill.
- Do not ship an unenforceable rule as `[MUST]`.
- Do not dump raw sources into context; summarize into the brief and findings.
