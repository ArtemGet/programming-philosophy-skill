# Finding format — canonical

Every extraction agent emits findings in exactly this shape, keyed to the fixed section ids
from `BRIEF.md`. Deterministic ids make the batch mergeable and traceable.

```markdown
## <section id>. <section title>

- **<prefix>.<n> <imperative title>** -> add to §<k>: <the rule, 1-3 sentences, actionable>.
  Why: <short reason / evidence / anecdote>.
  Example:
  ```
  <minimal, correct code or config or command>
  ```
```

Rules for agents:

- **[MUST]** One idea per finding; no multi-rule paragraphs.
- **[MUST]** Every finding names its target section (`-> add to §k`) so the merge agent can slot it.
- **[MUST]** Cite evidence: `path:line`, article id/URL, spec section, or a commit. No unsupported claims.
- **[SHOULD]** Tag priority `[MUST]/[SHOULD]/[NICE]` (or rely on the section header convention).
- **[SHOULD]** Include the smallest correct example; trim long snippets.
- **[SHOULD]** If a source contradicts another, state the default and the exception.
- **[NICE]** Note the source's own reasoning (the "why"), not just the rule.

Example:

```markdown
## 7. Testing

- **Atq.21 [MUST] One assertion per test** -> add to §7: build the subject by composition,
  then a single `assertThat`; no imperative arrange block.
  Why: one behavior per test = a failing test names exactly what broke.
  Example:
  ```
  assertThat(new Cart(items).total(), equalTo(100));
  ```
```

## Batch id prefixes (suggested)

| Batch | Prefix |
|---|---|
| Primary source (book/spec) | `B` |
| Secondary docs, batch 1..N | `S1`, `S2`, … |
| Codebase, repo/module 1..M | `C1`, `C2`, … |
| Merge agent | keeps original ids, collapses duplicates, notes `(also X.y)` |
| Distillation | re-tags priorities; never invents new claims |
