# 05 — Skill Authoring

Turn `work/rulebook.md` (plus `CODE.SYNTHESIS.md`) into a lean, portable Agent
Skill. Follow progressive disclosure: a small always-loaded `SKILL.md`, and depth
in `references/`. Optionally ship a `knowledge/` base for RAG-heavy depth.

[MUST] Do not author before the rulebook exists and is prioritized. Rulebook is the
source of truth; the skill is a projection of it.

---

## 1. Anatomy of the produced skill

```
<skill-name>/
  SKILL.md                 # always loaded — lean, <= ~500 lines / ~5k tokens
  references/              # loaded on demand — depth
    <topic>.md
  knowledge/               # OPTIONAL — large corpus for RAG/lookup
    index.md
    <chunks...>
  PROVENANCE.md            # OPTIONAL — license + provenance note (no raw sources)
```

[MUST] `SKILL.md` contains **no acquired source** and no private data (file 06).

---

## 2. Frontmatter (name + description)

[MUST] Frontmatter must parse. `name` MUST equal the skill's folder name.

```markdown
---
name: <skill-name>
description: <capabilities>. Use when <triggers>.
---
```

[MUST] The description states **capabilities AND triggers** — that is how the agent
decides to load the skill. Be specific about both.

Good:

```
description: Enforce the Acme engineering standard for Python services — module
boundaries, immutability of domain objects, one-assertion tests, and CI gates.
Use when writing, reviewing, or reviewing PRs in Acme Python repos, or when asked
to apply the Acme style/standard.
```

Bad:

```
description: A skill about good code.
```

---

## 3. Body rules

[MUST] `SKILL.md` body includes, in this order:

1. **What + when** — one paragraph; when to load and when not to.
2. **Workflow** — the handful of steps the agent follows.
3. **Top non-negotiable rules** — the [MUST] list, ≤ 20 items.
4. **Definition of Done** — a checklist the agent must satisfy.
5. **Reference map** — a table of `references/*.md` and when to open each.

[MUST] Cap the body at **≤ ~500 lines / ~5k tokens**. Move overflow to
`references/`. A skill that exceeds the budget loses its own rules in the noise.

[MUST] Cap the **[MUST]** rules at ≤ 20. Everything else is [SHOULD]/[NICE] or lives
in references.

---

## 4. Enforceability (the acid test)

[MUST] Every `[MUST]` rule must be **enforceable** — mapped to a gate that can
fail the build. A rule that cannot fail anything will be ignored in practice.

For each [MUST], record the gate:

| Rule | Gate | Where it fails |
|------|------|----------------|
| Every public behavior has a test | test runner | `pytest` / `npm test` / `mvn test` |
| No `null`/`None` in domain API | static check | lint rule / null-checker |
| Modules respect boundaries | arch test | ArchUnit / import-linter / dependency-cruiser |
| Formatting is canonical | formatter | `gofmt -l` / `black --check` / `prettier -c` |
| CI must pass before merge | branch protection | required status check |

[MUST] If a rule has no gate, either (a) downgrade it to [SHOULD]/[NICE], or
(b) define a concrete gate in the skill (a script, a CI step, a review checklist).
State the gate in the rule text.

---

## 5. Reference files (progressive disclosure)

[MUST] Each substantial topic in the rulebook gets a `references/<topic>.md`. Keep
them focused and imperative; mirror the rulebook's section ids.

### Reference-file skeleton

```markdown
# <NN> — <Topic>

One-line purpose. Link to the rulebook section it expands.

## When to open
- <trigger>

## Rules
### [MUST] <id> — <title>
- rule: ...
- gate: <what fails>
- why: ...
- example: <short positive/negative>
- source: <source ids>
### [SHOULD] ...

## Anti-patterns
- <pattern> -> <fix>

## Checklist
- [ ] ...
```

[MUST] References may include short illustrative snippets the user owns or
link-approved public material. **Do not** paste long verbatim copyrighted passages
(file 06); paraphrase with attribution.

---

## 6. `knowledge/` (optional)

[NICE] Ship a `knowledge/` base only when there is a large body of lookup material
(API tables, catalogues, long examples) that would bloat `references/`.

- `knowledge/index.md` — a table of contents with short descriptions.
- One file per coherent chunk; stable filenames; each states its scope.
- `SKILL.md` tells the agent when to search `knowledge/`.

[MUST] `knowledge/` obeys the same redaction rules as `SKILL.md`.

---

## 7. SKILL.md skeleton

```markdown
---
name: <skill-name>
description: <capabilities>. Use when <triggers>.
---

# <Skill title>

## What this is / when to use
<2–4 lines.>

## Workflow
1. ...
2. ...
3. ...

## Non-negotiable rules ([MUST], <=20)
1. **<rule>** — gate: <gate>. _(see references/<topic>.md#...)_
2. ...

## Definition of Done
- [ ] ...
- [ ] ...

## Reference map
| File | Open when |
|------|-----------|
| references/<topic>.md | <trigger> |
```

---

## 8. Deploy to runtimes

[MUST] Place the skill in the runtime root(s) the user chose in intake. Copy the
whole directory (SKILL.md + references + optional knowledge + provenance).

```bash
# generic / cross-agent
mkdir -p .agents/skills/<skill-name> && cp -r <built-skill>/* .agents/skills/<skill-name>/

# opencode
mkdir -p .opencode/skills/<skill-name> && cp -r <built-skill>/* .opencode/skills/<skill-name>/

# claude
mkdir -p .claude/skills/<skill-name> && cp -r <built-skill>/* .claude/skills/<skill-name>/
```

[MUST] The folder name MUST equal the frontmatter `name` in every deployment.

---

## 9. Authoring validation checklist

- [ ] Frontmatter parses; `name` == folder name (every runtime).
- [ ] Description states capabilities AND triggers.
- [ ] Body ≤ ~500 lines / ~5k tokens; [MUST] count ≤ 20.
- [ ] Every [MUST] has a concrete gate that can fail the build.
- [ ] Body has: what/when, workflow, [MUST] list, Definition of Done, reference map.
- [ ] Every internal link resolves (no dead `references/…` paths).
- [ ] `knowledge/` (if present) has an index and states when to search it.
- [ ] No source folders/files, no private data, no unapproved links (file 06).
- [ ] Skill copied to every runtime root; folder name matches `name`.

Proceed to `06-legal-privacy-and-provenance.md` and run the redaction gate before
publishing.
