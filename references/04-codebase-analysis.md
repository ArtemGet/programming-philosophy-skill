# 04 — Codebase Analysis (Real-Code Mining)

Text sources say what the philosophy *claims*. Repositories show what it *does*.
This phase mines real code for enforceable, evidence-backed rules and appends them
to the same knowledge base, keyed to the same frozen section ids.

[MUST] Read `BRIEF.md` for the section ids and finding format. Reuse the finding
format from file 03 §2 with prefix `C` and evidence `path:line`.

---

## 1. Division of labour

[MUST] Assign **one agent per repository** when repos are small/medium; batch
several small repos per agent when they are tiny. Keep a single large repo to a
single agent (or one per top-level module if enormous).

[MUST] Each analysis agent:

- reads ONLY its assigned repo plus `BRIEF.md`;
- writes `work/analysis/CODE.<repo>.md` using `C.<repo>.<n>` ids;
- keys findings to the frozen section ids;
- attaches `path:line` evidence to every rule;
- never edits another agent's file.

Batch plan goes in `work/analysis/repos.md`.

---

## 2. What to inspect FIRST (in this order)

[SHOULD] Survey cheaply, then deep-read. Stop as soon as a section has solid
evidence; do not read the whole repo.

| Order | Look at | For sections |
|-------|---------|--------------|
| 1 | Build/package manifests (`pom.xml`, `build.gradle`, `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `Makefile`) | §8 Build |
| 2 | CI configs (`.github/workflows/*`, `.gitlab-ci.yml`, `Jenkinsfile`, `.circleci/`, `.drone.yml`) | §9 CI/CD, §13 Tooling |
| 3 | `README`, `CONTRIBUTING`, `CODE_OF_CONDUCT`, `CODEOWNERS`, `docs/` | §11 Repo/docs, §12 Process |
| 4 | Issue/PR templates (`.github/ISSUE_TEMPLATE/`, `PULL_REQUEST_TEMPLATE*`), commit conventions, `CHANGELOG` | §12 Process, §10 Release |
| 5 | Test directories and framework config (`test/`, `tests/`, `spec/`, `__tests__/`) | §6–§7 Testing |
| 6 | Static-analysis / lint / format config (`.editorconfig`, linter configs, pre-commit) | §13 Tooling |
| 7 | A few representative source files (the core domain, not glue) | §2–§5 Code |

[MUST] Prefer the **authors' own** files (human-written core) over generated or
vendored code. Ignore `vendor/`, `node_modules/`, generated stubs, minified assets.

---

## 3. Extract these signal classes

[MUST] For each, emit `C.*` findings keyed to the matching section:

1. **Build / package config** — how modules are structured, dependency scopes,
   version pinning, reproducibility → §8.
2. **Dependency & tool catalogue** — what is used *and why*. For each notable
   dependency/tool, note its role. This becomes the "what & why" table → §8, §13.
3. **CI/CD pipelines** — stages, gates, order, what fails the build, caching,
   release automation → §9, §10.
4. **Test strategy** — unit/integration/e2e split, fixtures, mocking policy,
   coverage gates, naming → §6, §7.
5. **Code style in practice** — actual patterns (not what docs claim): module
   boundaries, naming, state handling, error handling → §2–§5.
6. **Repository/documentation conventions** — branch model, PR shape, commit
   messages, changelog, docs layout → §11, §12.
7. **Common problems & solutions** — recurring anti-patterns as seen in the code
   and their local fix; capture as generalizable rules → §14.
8. **Generalizable rules** — anything a reader could apply to their own code,
   stated neutrally.

### Finding template

```
- **C.<repo>.<n> <title>** -> add to §<section>: <generalizable rule>. Why: <observed reason>. Evidence: <path>:<line>. [src: <repo-id>]
```

Example:

```
- **C.acme.7 Pin CI tool versions** -> add to §9 CI/CD pipelines: pin every CI action/tool to an immutable version. Why: floating versions silently break reproducible builds. Evidence: .github/workflows/build.yml:14. [src: R2]
```

---

## 4. What to explicitly EXCLUDE

[MUST] Do not add findings that are:

- **language- or vendor-tooling-specific** with no general lesson (e.g. a quirk of
  one plugin's config syntax) — unless the philosophy itself mandates it;
- **environment-specific** (a hardcoded path, a corporate mirror, a personal token);
- **secret-bearing** (never copy tokens, keys, internal hostnames, org names);
- **incidental** (typos, dead code) unless they illustrate a real anti-pattern;
- **private identifiers** — no repo names, file paths, or org names from private
  sources may survive into the shipped skill (file 06).

[SHOULD] When excluding, keep a one-line note in the analysis file:
`(excluded N items as environment-specific)` — so the count is auditable.

---

## 4a. Dependency & tool catalogue (what & why)

[MUST] Produce a table of every notable dependency/tool the repo uses and the
**reason** it is there. This is the highest-value output of the phase because it
becomes the skill's "recommended toolchain".

```markdown
| Category | Tool / dependency | Role (why) | Evidence | Section |
|----------|-------------------|------------|----------|---------|
| test runner | pytest | unit + integration tests | pyproject.toml:34 | §6 |
| linter | ruff | fast lint + import order | .github/workflows/ci.yml:22 | §13 |
| arch check | import-linter | enforce module boundaries | .importlinter:1 | §3 |
| formatter | black | canonical formatting | pre-commit-config.yaml:8 | §2 |
```

[MUST] Separate **runtime** dependencies from **dev/build** dependencies. A tool in
the catalogue only earns a skill rule if the philosophy endorses it or the whole
codebase depends on it.

---

## 4b. From tools to gates

[MUST] For each enforcement tool found, capture the **exact command** that fails
the build. These become the gates referenced by `[MUST]` rules in the skill.

```markdown
| Rule candidate | Gate command | Fails on |
|----------------|--------------|----------|
| formatting canonical | `black --check .` | unformatted file |
| imports ordered | `ruff check --select I .` | bad import order |
| boundaries respected | `lint-imports` | cross-layer import |
| types sound | `mypy --strict .` | type error |
| tests pass | `pytest -q` | failing/missing test |
```

[MUST] If a rule has no executable gate in the codebase, mark it `[no-gate]` — it
cannot be a `[MUST]` (see file 05 §4).

---

## 4c. Anti-pattern catalogue

[SHOULD] While deep-reading representative source, record recurring problems and
the local fix. These feed §14 and make the skill's "don't do this" section concrete.

```
- **C.<repo>.<n> <anti-pattern title>** -> add to §14 Common anti-patterns & fixes:
  <what to avoid>. Why: <observed harm>. Fix: <better pattern>.
  Evidence: <path>:<line> (+ N similar). [src: <repo-id>]
```

[SHOULD] Prefer anti-patterns seen **more than once**; a one-off is noise. If the
same fix appears in several repos, that is a strong `[MUST]`.

---

## 5. Cross-repo synthesis

[MUST] After per-repo analysis, one agent reads all `CODE.*.md` and emits
`work/analysis/CODE.SYNTHESIS.md`:

- rules that appear in **most/all** repos → strong [MUST] candidates;
- rules in **one** repo only → mark `[isolated]`, likely SHOULD/NICE;
- contradictions across repos → `D.*` disagreements (default vs exception), same as
  file 03 §6.

[MUST] Synthesis must NOT copy private evidence verbatim into shipped artifacts;
it carries `[src: repo-id]` tags only.

---

## 6. Survey commands

[SHOULD] Locate the high-value files quickly instead of reading blindly:

```bash
# manifests + CI + docs + tests + lint (one pass)
find . -maxdepth 3 \
  \( -name 'pom.xml' -o -name 'build.gradle*' -o -name 'package.json' \
     -o -name 'pyproject.toml' -o -name 'Cargo.toml' -o -name 'go.mod' \
     -o -name 'Makefile' -o -name '.editorconfig' \) -print

find .github .gitlab-ci.yml Jenkinsfile .circleci -maxdepth 3 -type f 2>/dev/null
find . -maxdepth 2 -iname 'readme*' -o -maxdepth 2 -iname 'contributing*' \
     -o -maxdepth 2 -iname 'codeowners' -o -maxdepth 2 -iname 'changelog*'

find . -type d \( -name test -o -name tests -o -name spec -o -name __tests__ \) \
  -not -path '*/node_modules/*' -not -path '*/vendor/*' | head -20

# dependency inventory by ecosystem
grep -nE '<artifactId>|"dependencies"|\[tool\.|^[a-zA-Z0-9_-]+ *=' \
  pyproject.toml package.json pom.xml 2>/dev/null | head -80
```

### 6.1 Choosing representative source

[MUST] Read a **small, deliberate** sample, not the whole tree:

- the core domain types (not framework glue, not generated code);
- one "happy path" flow end-to-end;
- one error/edge-case handler;
- one test file that exemplifies the testing philosophy.

[SHOULD] Cap deep-reading at roughly 5–12 files per repo. Stop when each target
section has at least one solid citation.

---

## 7. Evidence quota

[MUST] Each analysis file must provide, per section it touches:

- at least one `path:line` citation; and
- a generalizable rule (not a description of the repo).

[SHOULD] Aim for 8–20 findings per substantial repo; fewer for tiny repos. A file
with zero citations is invalid — re-run that agent.

---

## 8. Output header

[MUST] Every `CODE.<repo>.md` starts with a short, deterministic header so merge
and synthesis can trust it:

```
# CODE.<repo>   [src: <repo-id>]
public: <yes/no>   scanned: <manifest, ci, tests, src-sample>
findings: <count>  excluded: <count>
```

---

## 9. Codebase analysis checklist


- [ ] One agent per (group of) repo; `repos.md` records the assignment.
- [ ] Each agent wrote `CODE.<repo>.md` with `C.*` ids and `path:line` evidence.
- [ ] Inspected in the priority order (§2), not by random reading.
- [ ] Dependency/tool "what & why" catalogue produced.
- [ ] Exclusions noted; no environment-specific or secret-bearing content.
- [ ] Synthesis produced: cross-repo MUST candidates, isolated items, disagreements.
- [ ] No private repo names/paths/org names leak beyond the `[src]` tag.

Proceed to `05-skill-authoring.md`.
