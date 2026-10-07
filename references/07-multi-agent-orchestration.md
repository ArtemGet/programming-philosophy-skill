# 07 — Multi-Agent Orchestration

Rules for running the extraction/analysis pipeline with subagents. The goal is
**mergeability by construction**: every agent produces an isolated, deterministically
named file that a merge step can combine without conflicts.

[MUST] This file governs every phase in files 03 and 04.

---

## 1. Phase diagram

```
            work/INTAKE.md + work/PROVENANCE.md
                          |
                     build BRIEF.md
                          |
        +-----------------+------------------+
        |                                    |
   [Phase A]                            [Phase B]
   1 agent                               N agents (balanced batches)
   PRIMARY.md                            ADD.01.md .. ADD.N.md
   (P.* ids, frozen sections)            (A.* ids, same sections)
        |                                    |
        +-----------------+------------------+
                          |
                     [Phase C] merge
                     1 agent: copy PRIMARY verbatim -> insert additions
                     MERGED.md
                          |
                     [Phase D] distill
                     1 agent -> work/rulebook.md ([MUST]/[SHOULD]/[NICE])

   (in parallel with Phase B, if desired)
   [Code] 1 agent per repo -> CODE.<repo>.md (C.* ids, path:line)
                          |
                     [Synthesis] 1 agent -> CODE.SYNTHESIS.md

                          v
              [Authoring] 1 agent -> SKILL.md + references/
                          v
              [Redaction gate] (file 06) -> publish
```

---

## 2. Choose the parallelism

[MUST] Pick an explicit max-concurrent-agent budget BEFORE launching, and never
exceed it. A safe default is **3–5 concurrent agents**; go lower on constrained
environments.

[SHOULD] Size the work:

| Sources | Parallelism |
|---------|-------------|
| ≤ 5 short sources | batch them into 1–2 agents |
| 6–30 sources | 3–5 batch agents (balanced) |
| 30+ sources | 5+ batches; run in waves respecting the budget |
| Repos: 1 tiny | fold into a batch agent |
| Repos: 1 large | 1 dedicated agent |
| Repos: many | 1 per repo if within budget, else batch small ones |

[MUST] Keep each agent's **source set bounded** (target ≤ 60k tokens of raw input).
Never hand an agent the whole corpus.

---

## 3. One agent vs many — guide

Use **ONE** agent when:

- you must read a single long primary source coherently (Phase A);
- you must merge or distill (Phases C, D) — merging is inherently single-writer;
- the work is small enough that coordination overhead exceeds the gain;
- the task needs a global view (synthesis).

Use **MANY** agents when:

- inputs are independent and splittable (secondary sources, separate repos);
- the work is embarrassingly parallel (per-article, per-repo);
- one context would blow the budget;
- you want throughput on I/O-bound reading.

[MUST] Never parallelize a phase that must produce a single coherent artifact.
Merging, distillation, and synthesis are single-agent.

---

## 4. Shared context: pass the BRIEF to everyone

[MUST] Every extraction/analysis agent receives, at minimum:

- `BRIEF.md` (principles, vocabulary, frozen section ids, finding format);
- its **own** bounded source set only;
- its exact output path and id prefix.

[MUST] Do NOT give agents each other's outputs during a parallel phase. Isolation
is what makes the merge deterministic.

---

## 5. Deterministic ids (mergeability)

[MUST] Every finding id is deterministic and unique:

- prefix by producer: `P` primary, `A` article/batch, `C` code, `D` disagreement;
- `<n>` assigned in **input order**, never renumbered, never reused;
- batch agents use a stable batch index (`ADD.03.md` → `A03.*`) so re-runs match.

[MUST] Output filenames are deterministic and sorted-mergeable:
`PRIMARY.md`, `ADD.<NN>.md`, `CODE.<repo>.md`.

---

## 6. Forbid cross-edits

[MUST] Each agent may write ONLY its own file. State this in every agent prompt:

```
You may read your assigned sources and BRIEF.md. Write ONLY <output-path>.
Do not read or modify any other agent's output. Do not edit PRIMARY.md.
```

[MUST] The merge agent is the ONLY writer of `MERGED.md`, and it **copies the
primary output verbatim before inserting** additions (file 03 §5). It never rewrites
the base.

---

## 7. Verify each phase before the next

[MUST] Do not start the next phase until the current one passes its check:

| Phase | Verify |
|-------|--------|
| BRIEF | section ids frozen; finding format present |
| A | `PRIMARY.md` exists; every finding has a valid §id and `P.<n>`; empty sections marked |
| B | all `ADD.*.md` exist for every batch; no unknown §ids; no duplicate ids |
| C | `MERGED.md` contains PRIMARY verbatim (diff the primary portion); all additions inserted |
| D | `rulebook.md` has [MUST] ≤ 20; every rule has ≥1 source id; disagreements recorded |
| Code | every `CODE.*.md` has `path:line` evidence; synthesis present |
| Authoring | validation checklist in file 05 §9 passes |
| Redaction | all greps in file 06 §7 clean |

[MUST] On any failure, fix the offending agent's output (re-run that agent only) —
do not paper over it downstream.

---

## 8. Reliability rules

[MUST] For each agent:

- give a **bounded, explicit input list** (no "read everything");
- require the **exact output format** with one example finding;
- require the agent to **summarize long input** rather than paste it;
- ask for a short header (`# ADD.<NN>  sources: A01,A02  findings: <count>`);
- cap output length (split if it would exceed a page).

[SHOULD] Prefer many small deterministic agents over few clever ones. Preserve
re-run reproducibility: same inputs + same ids → same merge.

[SHOULD] Log a one-line status per agent in `work/orchestration.md`:

```
2026-10-02 batch-01 ADD.01 sources=A01,A02 done findings=14
```

---

## 9. Failure modes and how to avoid them

[MUST] Watch for these; each breaks determinism or quality:

| Failure | Cause | Prevention |
|---------|-------|------------|
| Merge conflicts / lost findings | two writers on one file | one writer per file (§6) |
| Renumbered ids | agent assigned ids from its own order | ids assigned from the **input order** in `batches.md` (§5) |
| Hallucinated sections | BRIEF section ids not given | pass `BRIEF.md` to every agent (§4) |
| Context overflow | whole corpus handed to one agent | bounded sets + summarize (§8) |
| Duplicated rules | parallel agents cover overlapping sources | dedup in distill; `[dup-of]` tag in merge |
| Base corruption | merge agent "improved" the primary | copy verbatim, then insert (§6) |
| Silent gaps | a batch produced empty output | verify phase outputs (§7) |
| Unmergeable re-runs | non-deterministic filenames | deterministic `ADD.<NN>.md` names (§5) |

[MUST] On a failed run, re-run **only** the offending agent with the same inputs;
do not regenerate the whole pipeline (that re-triggers non-determinism).

---

## 10. Agent prompt template

[MUST] Use a consistent, minimal prompt for every parallel extraction agent:

```
Role: extractor (batch <NN>).
Read: BRIEF.md (frozen sections + finding format), then ONLY: <source ids>.
Write: <output-path>  (nothing else).
Ids: prefix <A03>, number by input order, never renumber.
For every finding use EXACTLY:
  - **<A03>.<n> <title>** -> add to §<section>: <rule>. Why: <why>. Example: <ex>. [src: <source-id>]
Rules: do not read/edit other agents' files; summarize long input, do not paste it;
emit a header line `# ADD.<NN>  sources: ...  findings: <count>`.
```

[MUST] Replace all placeholders; never send a template with `<...>` left in.

---

## 11. Orchestration checklist

- [ ] Max-concurrent budget chosen and respected.
- [ ] Sources split into balanced, bounded batches (`batches.md`).
- [ ] Every agent got `BRIEF.md` + only its own bounded input + its output path.
- [ ] No agent reads/edits another's output; merge is copy-then-insert.
- [ ] Ids deterministic and unique; filenames deterministic and sortable.
- [ ] Each phase verified before the next; failures fixed in-place.
- [ ] `work/orchestration.md` logs each agent's status and finding count.
- [ ] Single-writer enforced for merge, distill, and synthesis.
