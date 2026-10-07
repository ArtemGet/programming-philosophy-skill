# 03 — Knowledge Extraction (Multi-Agent Pipeline)

Turn raw sources into a **deduplicated, source-attributed knowledge base**, then
distill it into a prioritized rulebook. The pipeline is deliberately deterministic
so parallel agents can write without stepping on each other.

[MUST] Read `work/INTAKE.md` and `work/PROVENANCE.md` first. Read `BRIEF.md` (built
in §1) before any extraction. Use the orchestration rules in file 07.

---

## 1. Build the shared BRIEF

[MUST] One file, `work/BRIEF.md`, defines the canon every agent receives. It has
three parts:

### 1.1 Principles (the canon)

- the philosophy's **core principles**, in 5–15 crisp lines;
- the **vocabulary** (terms the rulebook must use consistently, with definitions);
- the **non-goals** (what the philosophy explicitly rejects).

### 1.2 Fixed section structure

[MUST] Freeze the section ids BEFORE extraction. Every agent keys findings to these
ids; they never change. Suggested generic skeleton (adjust to scope from intake):

```
§1  Core principles & mindset
§2  Naming & vocabulary
§3  Code structure & modules
§4  Immutability / state & data
§5  Error handling & failure
§6  Testing strategy
§7  Test writing practices
§8  Build & dependency management
§9  CI/CD pipelines
§10 Release & versioning
§11 Repository & documentation conventions
§12 Process: issues, PRs, reviews, commits
§13 Tooling & enforcement
§14 Common anti-patterns & fixes
```

### 1.3 Rules of engagement

- Output the **finding format** verbatim (§2).
- No cross-edits: only the agent that owns a file may write it.
- Bounded input: never load the whole corpus into one context.
- Attribute every finding to a source id from `PROVENANCE.md`.

---

## 2. The finding format (frozen)

[MUST] Every extracted item uses EXACTLY this one-line shape so a merge agent can
parse and insert mechanically:

```
- **<prefix>.<n> <title>** -> add to §<section>: <rule>. Why: <rationale>. Example: <short example>. [src: <source-id>]
```

- `<prefix>` identifies the producer: `P` (primary), `A` (article/secondary),
  `C` (code), `D` (disagreement).
- `<n>` is **deterministic**: assigned in input order, never renumbered.
- `<section>` must be one of the frozen ids.
- Keep each finding to 1–3 sentences. Split, don't sprawl.

Example:

```
- **A07.3 Fail fast on invalid config** -> add to §8 Build & dependency management: validate configuration at startup and abort on the first error. Why: late failure wastes build and review time. Example: a `Config.validate()` called before `main()` runs. [src: A07]
```

---

## 3. Phase A — primary source (ONE agent)

[MUST] Exactly one agent reads the primary source (book / spec / manifesto).
Multiple primary chunks are fine for the SAME agent to read sequentially, but the
output is a single file.

- Input: `sources/books/<book>.txt` (+ chunks), plus `BRIEF.md`.
- Output: `work/extraction/PRIMARY.md` — postulates + examples under the fixed
  section ids, using `P.<n>`.

Structure of `PRIMARY.md`:

```markdown
# PRIMARY — <title>   [src: P1]
## §1 Core principles & mindset
- **P.1 <title>** -> add to §1: <rule>. Why: ... Example: ...
## §2 Naming & vocabulary
- **P.2 ...**
...
## §<section with no findings>
(none)
```

[MUST] Section ids appear in order even when empty (`(none)`) so the merge is
deterministic.

---

## 4. Phase B — secondary sources (N parallel agents)

[MUST] Split secondary sources into **balanced batches**. Each batch agent:

- reads ONLY its batch + `BRIEF.md`;
- emits `work/extraction/ADD.<batch-id>.md` using `A.<n>` ids;
- keys every finding to the same frozen sections;
- **must not** read or edit other agents' outputs, nor `PRIMARY.md`.

### 4.1 Balanced batching

[SHOULD] Balance by **estimated tokens**, not file count:

- sort sources by size;
- assign greedily to the currently-smallest batch;
- target 2–6 sources per batch and keep each batch under a set token budget
  (e.g. ≤ 60k tokens of raw source);
- one batch per article if articles are long; group short docs.

Record the split in `work/extraction/batches.md`:

```
batch-01: A01, A02, A05      (~52k tok)
batch-02: A03, A04, A07      (~48k tok)
...
```

### 4.2 Bounded input rule

[MUST] No agent may dump an entire corpus. If a source exceeds the budget,
pre-summarize it (by the same agent, in a bounded first pass) and extract from the
summary. Prefer "summarize the relevant part" over "paste everything".

---

## 5. Phase C — merge (ONE agent)

[MUST] The merge agent **copies the primary output verbatim** into
`work/extraction/MERGED.md`, then inserts additions. It NEVER rewrites the base.

Algorithm:

1. Copy `PRIMARY.md` to `MERGED.md` unchanged.
2. For each `ADD.*.md`, for each section id, append the batch's findings under that
   section (after the primary findings).
3. Preserve every `P.*` and `A.*` id exactly as written.
4. Never delete, merge, or reword a primary finding.
5. If two additions are near-duplicates, keep both but tag the duplicate with
   `[dup-of: <other-id>]` — dedup happens in distillation, not here.

```bash
# deterministic ordering of additions
ls work/extraction/ADD.*.md | sort
```

[MUST] Fail loudly if any finding uses an unknown section id or a duplicated id.

---

## 6. Phase D — distill (ONE agent)

[MUST] One agent turns `MERGED.md` into `work/rulebook.md`: a prioritized,
deduplicated, enforceable rulebook.

Requirements:

- **Deduplicate** findings that say the same thing; cite all source ids.
- **Prioritize** every rule as `[MUST]`, `[SHOULD]`, or `[NICE]`.
  - [MUST] = non-negotiable and **enforceable by a gate** (build/test/lint/CI).
  - [SHOULD] = strongly recommended, may need human review.
  - [NICE] = stylistic or optional.
- **Record disagreements** explicitly: when sources conflict, state the **default**
  and the **exception** that overrides it, with the source ids on each side.
- **Keep evidence**: each rule keeps at least one source id; code rules keep a
  `path:line` (from Phase C of file 04).
- **Cap the count**: target ≤ 20 `[MUST]` rules. Everything else is SHOULD/NICE or
  moves to references. (A skill with 200 [MUST]s enforces nothing.)

Rulebook shape:

```markdown
# RULEBOOK — <philosophy>
## §6 Testing strategy
### [MUST] M1 — Every public behavior has a test
- rule: ...
- why: ...
- gate: `mvn test` / `pytest` / `npm test` fails when absent
- sources: P.4, A12.1, R1:docs/CONTRIBUTING.md
### [SHOULD] S3 — Prefer one assertion per test
- ...
### Disagreements
- D1: primary says X (§6), article A22 says Y. Default: X. Exception: Y when ...
```

---

## 7. Extraction checklist

- [ ] `BRIEF.md` frozen (principles, vocabulary, section ids, finding format).
- [ ] Phase A produced `PRIMARY.md` with stable `P.*` ids.
- [ ] Secondary sources split into balanced batches (`batches.md`).
- [ ] All `ADD.*.md` agents wrote isolated files with `A.*` ids, no cross-edits.
- [ ] Merge copied the primary verbatim, then inserted; ids intact; no unknown ids.
- [ ] Distillation deduped, prioritized ([MUST]/[SHOULD]/[NICE]), cited sources,
      recorded disagreements, and respected the ≤20 [MUST] cap.
- [ ] Every rule is attributable to at least one source id.

Proceed to `04-codebase-analysis.md`.
