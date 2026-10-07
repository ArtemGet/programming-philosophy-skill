# 01 — Intake and Discovery (First Run)

You are about to build a **programming-philosophy skill**: an enforceable, portable
agent skill that captures how a codebase is *meant* to be written, tested, shipped,
and reviewed — in any language, under any philosophy. The methodology is generic.
Elegant Objects, "Clean Code", PEP-8-as-a-philosophy, or an internal corporate
standard are all equally valid targets; never assume the subject.

This file defines what you do on the **first invocation**, before you acquire a
single byte.

---

## 0. First-run opening message

[MUST] On first run, before asking anything, print a short orientation — **6 lines
maximum**. It must state:

1. what you are going to do (systematically study sources → build a knowledge base →
   distill → author a lean skill), and
2. what the end artifact will be (a self-contained skill directory you can drop into
   a runtime).

Template:

```
I will build a portable programming skill from your sources.
Plan: acquire sources -> extract rules (multi-agent) -> mine repos -> distill a
prioritized rulebook -> author a lean SKILL.md + reference depth.
End artifact: a self-contained skill folder (SKILL.md + references/ + optional
knowledge/) you can install into a runtime's skills directory.
I will confirm the plan before downloading anything. Next: a short intake interview.
```

[MUST] Do not download, clone, or fetch anything during this message.

---

## 1. Interview the user

[MUST] Use the environment's **question tool** (e.g. an interactive question /
elicitation tool) for the interview — not free-form prose dumps. Ask in small
batches. If the environment has no question tool, fall back to a numbered
questionnaire and wait for written answers.

[SHOULD] Ask only what you cannot infer. Do not ask a question whose answer is
already in the user's message.

### 1.1 Required questions

| # | Question | Why it matters |
|---|----------|----------------|
| Q1 | Which **language(s) / stack / framework(s)** does this skill target? | Drives tooling, CI, examples. Or "language-agnostic". |
| Q2 | Which **philosophy or conventions** are we capturing? Give its name, if any. | Sets the canon; may be unnamed internal rules. |
| Q3 | Which **repositories** (URLs or local paths) are available? | Real-code evidence. |
| Q4 | Which **documents** exist — books/PDFs, specs, wikis, style guides, blog articles, mailing-list threads? | Primary + secondary sources. |
| Q5 | What **scope** should the skill cover? (coding only / +testing / +CI-CD / +release / +process / +documentation) | Bounds the rulebook. |
| Q6 | Which **agent runtime(s)** must the skill install into? | Output layout (see Q7). |
| Q7 | What **output layout / directory** do you want? | Portable vs runtime-specific. |
| Q8 | Any **license / copyright constraints** on the sources? | Legal gate (file 06). |
| Q9 | Which **links** may appear in the produced skill vs must be omitted? | Redaction gate (file 06). |
| Q10 | Are the sources **public or private**? | Redaction gate (file 06). |

### 1.2 Scope checkboxes (Q5)

[MUST] Present scope as checkboxes and record the answer verbatim:

- [ ] Coding style / structure
- [ ] Testing strategy
- [ ] CI/CD pipelines
- [ ] Release / versioning
- [ ] Development process (PRs, reviews, commits)
- [ ] Documentation

### 1.3 Runtime checkboxes (Q6)

[SHOULD] Offer common skill roots (adapt to the environment):

- [ ] `.agents/skills/<name>/`   (generic / cross-agent)
- [ ] `.opencode/skills/<name>/`
- [ ] `.claude/skills/<name>/`
- [ ] other (ask the user to specify)

---

## 2. Ask for MISSING inputs (be proactive)

[MUST] After the first pass, explicitly name what is **absent** and suggest
concrete additions. Never silently proceed with thin sources.

Suggestions to offer when missing:

- a **canonical manifesto / philosophy page** (the "why" behind the rules);
- a **book or long-form spec** (the primary source — Phase A);
- **reference implementations** the authors themselves consider exemplary;
- **style guides** (official or de-facto);
- **CI configuration files** (`.github/workflows`, `Jenkinsfile`, `.gitlab-ci.yml`, …);
- **issue / PR / commit conventions** (templates, CONTRIBUTING, CODEOWNERS);
- **example code** (a "golden" repo vs an anti-pattern repo);
- **review checklists** used internally;
- **changelog / release notes** showing how rules evolved.

[MUST] If the user cannot supply a primary source, say so plainly: the rulebook
will be weaker and less authoritative. Ask whether to proceed anyway.

---

## 3. Produce a short written plan (get confirmation)

[MUST] Before acquiring anything, write a plan and ask for explicit confirmation.

Plan template:

```
INTAKE PLAN
- Subject philosophy : <name / "unnamed internal standard">
- Target stack      : <languages/frameworks or "language-agnostic">
- Scope             : <checked scopes>
- Runtimes          : <checked runtimes>
- Sources
    Primary   : <book/spec/none>
    Secondary : <N articles/docs>
    Repos     : <N URLs>
- Acquisition      : shallow clones + downloads into sources/ (gitignored)
- Extraction       : 1 primary agent -> N batch agents -> 1 merge -> 1 distill
- Analysis         : 1 agent per repo (or batched by size)
- Output           : SKILL.md (<=500 lines) + references/ + knowledge/? 
- Legal            : links approved=<list>; private=<yes/no>; copy=no
- Exclusions       : sources/ is NEVER shipped; no verbatim copyrighted passages
Proceed? [yes / adjust]
```

[MUST] Do not proceed to file 02 until the user confirms. If they adjust, update
the plan and re-confirm.

---

## 4. Record the intake

[MUST] Persist the answers to `work/INTAKE.md` (create `work/` if absent; it is
gitignored). This is the single source of truth the whole pipeline reads.

### Intake record template

```markdown
# INTAKE — <project/skill name>
date: <YYYY-MM-DD>
interviewer: agent

## Subject
- philosophy name:
- description (1-2 lines):
- canonical manifesto URL (if any, and only if link-approved):

## Targets
- languages/stack:
- philosophy/conventions:
- scope: [coding, testing, ci-cd, release, process, docs]
- runtimes: [.agents/skills, .opencode/skills, .claude/skills, ...]
- output layout / dir:

## Sources
### Primary (long-form, authoritative)
- <title> | type: book|spec|wiki | location: <url|path> | license:
### Secondary (articles, docs, guides)
- <title> | type: article|doc | location: <url|path> | license:
### Repositories
- <name> | url: <url> | public/private | license:
### Missing / requested
- <what is absent and what would help>

## Legal & privacy
- sources public/private:
- link policy: allowed=[...] forbidden=[...]
- copyright constraints:
- NDA / personal-data constraints:

## Notes
- user preferences, hard constraints, ambiguous points
```

[MUST] Keep secrets out of `INTAKE.md`. If a source is under NDA, record only
"private source, link status: forbidden" — never the URL.

---

## 5. Intake completion checklist

Before moving on, verify all of these are true:

- [ ] Orientation message was <=6 lines and promised no premature downloads.
- [ ] Q1–Q10 asked (or explicitly answered already).
- [ ] Missing inputs named and suggestions offered.
- [ ] Written plan produced and **explicitly confirmed** by the user.
- [ ] `work/INTAKE.md` written with provenance + legal fields.
- [ ] `sources/` (or `work/`) will be gitignored — noted for file 02.
- [ ] User acknowledged that the final skill ships **no** source folders and only
      link-approved URLs.

Only then proceed to `02-source-acquisition.md`.
