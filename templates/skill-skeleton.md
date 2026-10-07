# SKILL.md skeleton (copy and fill for the produced skill)

```markdown
---
name: <skill-name-matching-folder>
description: >-
  <What it does — capabilities>. Use when <triggers: writing/reviewing/setting up <stack>>.
  <Optional: what it includes — tests, CI/CD, repo conventions>.
license: <MIT>
---

# <Skill title>

<One or two sentences: the domain and the promise.>

## When to use
- <trigger 1>
- <trigger 2>
**When not to use:** <exclusions>.

## The 30-second version
1. <core rule 1>
2. <core rule 2>
…  (5-8 lines max)

## Canon (the shape to memorize)
<terse table/list of the philosophy's core principles>

## Non-negotiable rules (the build must enforce these)
1. **[MUST]** <rule> — <enforced by: <tool/gate>>
…
(≤20 rules)

## How to work on any task
```
1. <step>
2. <step>
…
```

## Definition of Done
- [ ] <item>
- [ ] <item>

## Reference map (read only what the task needs)
| Topic | File |
|---|---|
| <topic> | `references/01-....md` |
…

## Anti-patterns at a glance
<comma list of things never to do>

## Pragmatic deviations (allowed, but written down)
- <deviation> — <when/why>
```

Guidelines:
- Body ≤ ~500 lines / ~5k tokens; depth goes to `references/`.
- `description` states capabilities **and** triggers; ≤ ~1024 chars.
- Every `[MUST]` maps to something that can fail (build/lint/test/review).
- No staged-source folder names, private paths, private URLs, or secrets.
