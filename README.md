# programming-philosophy-skill

A portable **Agent Skill** that teaches an agent how to systematically study a
codebase plus its documentation — in **any language, under any programming
philosophy** — and turn that study into a high-quality, enforceable
programming / CI-CD / engineering skill.

It is generic by design: it is **not** tied to Java, to Elegant Objects, or to any
single stack. The methodology was proven on a real project (a book + hundreds of
articles + a set of repositories distilled into a rulebook and then a lean skill)
and generalized here.

## What it produces

A self-contained skill directory:

```
<skill-name>/
  SKILL.md          # lean, always-loaded rules (author writes this)
  references/       # depth, loaded on demand
  knowledge/        # optional, for large lookup corpora
```

The pipeline: acquire sources → extract rules with subagents → mine real repos →
distill a prioritized rulebook → author a lean skill → redaction gate → publish.

## Install

Copy or symlink this skill directory into your agent runtime's skills root:

```bash
# generic / cross-agent
cp -r programming-philosophy-skill .agents/skills/

# opencode
cp -r programming-philosophy-skill .opencode/skills/

# claude
cp -r programming-philosophy-skill .claude/skills/
```

Then invoke it (e.g. "study this philosophy and build a skill") — the agent starts
with `references/01-intake-and-discovery.md`.

## File map

| File | Purpose |
|------|---------|
| `references/01-intake-and-discovery.md` | First-run behavior: orientation, interview, plan, confirmation |
| `references/02-source-acquisition.md` | Safe acquisition + staging of repos, docs, books (gitignored `sources/`) |
| `references/03-knowledge-extraction.md` | Multi-agent extraction: BRIEF → primary → batches → merge → distill |
| `references/04-codebase-analysis.md` | Mining real repositories for build, CI, tests, style, conventions |
| `references/05-skill-authoring.md` | Turning the rulebook into a lean, enforceable skill |
| `references/06-legal-privacy-and-provenance.md` | HARD GATE: redaction, link policy, copyright, provenance |
| `references/07-multi-agent-orchestration.md` | Parallelism, batching, deterministic ids, phase verification |

## Notes

- `SKILL.md` and `templates/` are written separately by the main author and are not
  part of this reference bundle.
- Acquired sources live in a gitignored `sources/`/`work/` and are **never** shipped
  (see `references/06-...`).
