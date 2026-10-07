# Shared brief — <STACK / PHILOSOPHY NAME>

> Every extraction agent receives this file. It is the vocabulary and the FIXED section
> structure that makes all outputs mergeable. Fill it from the canonical sources BEFORE
> launching any agent. Replace every `<...>`.

## Goal

Build an Agent Skill about **<what to do>** in **<language/stack>** following **<philosophy>**,
covering: <scope list — coding / testing / CI-CD / release / process / docs>.

## Sources

- Primary (canon): <book/spec/manifesto + where it lives in the staging area>
- Secondary: <articles/docs list>
- Code: <repositories/modules>
- Staging root: `<sources/>` (gitignored; NEVER referenced in the produced skill).

## Canon (the few things everyone must agree on)

- <principle 1>
- <principle 2>
- <key vocabulary / named concepts>
- <the shortest correct example of the core idea>

## FIXED section structure (use these exact ids everywhere)

1. Core concepts & design
2. <constructs/lifecycle>
3. <state/mutability/errors — adapt to the stack>
4. Naming, interfaces & contracts
5. Composition & reuse
6. Errors & edge cases
7. Testing
8. Code style & static analysis
9. Build & packaging
10. CI/CD & release
11. Repository & documentation conventions
12. Project & process
13. Tools & dependencies
14. Recurring problems & solutions

## Finding format (all agents)

`- **<prefix>.<n> <imperative title>** -> add to §<k>: <rule>. Why: <reason>. Example: <code/text>.`

Prefixes by batch (so a merge agent can trace provenance): `<book>`, `<src1>`, `<code1>`, …

## Priorities

`[MUST]` the build/lint/test must fail on it · `[SHOULD]` strong default, deviate with a written reason ·
`[NICE]` review guidance.

## Out of scope

- <things not to include>
- <language-specific trivia irrelevant to general practice>
