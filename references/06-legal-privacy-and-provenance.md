# 06 — Legal, Privacy, and Provenance

**THIS FILE IS A HARD GATE.** A skill that leaks a private repo name, a paywalled
link, or a copied passage can cause real legal and professional harm. When in
doubt, **omit**.

The produced skill is a **synthesis** — rules, paraphrases, and approved links. It
is **not** a redistribution of the sources.

---

## 1. The non-negotiables

[MUST] The produced skill MUST contain:

- **NO references to downloaded source folders or files** — never mention
  `sources/`, `work/`, `code/`, `repos/`, `articles/`, `books/`, or any local path
  used during extraction.
- **NO embedded private/proprietary source code** — not even "just one function".
- **NO path from a private repo** — no `src/internal/foo.py:42`.
- **Only external links the user explicitly approved** (see §2).
- **NO secrets** — no tokens, keys, internal hostnames, emails, org names.

[MUST] Any finding that needed private evidence is expressed as a **general rule**,
attributed only by a neutral source id — never by the private path itself.

---

## 2. Link policy — ask before publishing

[MUST] Before publishing, ask the user which links are allowed. Do not guess.

Ask:

1. Which links are **allowed**? (public official docs, canonical manifests,
   open-source repo homepages the authors endorse)
2. Which links are **forbidden**? (private/internal wikis, paywalled docs, NDA
   material, personal blogs, anything copyright-sensitive)
3. For each source in `PROVENANCE.md`, is its URL allowed in the skill?

Record the decision in `work/LINK_POLICY.md`:

```markdown
# LINK POLICY
## Allowed (may appear in shipped skill)
- https://<public official docs>
## Forbidden (must NOT appear)
- <private/paywalled/NDA/personal>
## Default
- omit unless explicitly allowed
```

[MUST] **Default is omit.** A link is included ONLY if the user approved it. If the
user is unsure, omit and say why.

---

## 3. Copyright handling

[MUST] Never paste large verbatim passages from copyrighted sources. 

- **Paraphrase** the rule in your own words, with attribution ("per <source title>").
- Short, necessary quotes (a phrase, a term) are acceptable only for publicly
  licensed or fair-use material, and MUST be attributed.
- Do not reproduce chapter/section text, tables, or figures from a book.
- Do not reproduce article bodies; state the rule they imply.
- Respect source licenses from `PROVENANCE.md` (CC-BY needs attribution; "© author"
  needs care; unknown means omit).

[NICE] If a work is CC-BY, attribute it in `PROVENANCE.md` with license + URL (if
allowed).

### 3.1 License handling matrix

[MUST] Behave according to the source's license:

| License / status | May read locally | May ship its text | Attribution required | Link (if approved) |
|------------------|:---:|:---:|:---:|:---:|
| Public domain / CC0 | yes | paraphrase only | no | yes |
| CC-BY | yes | no (paraphrase + attribute) | yes | yes |
| CC-BY-NC / ND | yes | no | yes | ask |
| MIT / Apache-2.0 (docs) | yes | no (code stays out) | yes | yes |
| © author, no license | yes | no | no reuse | omit by default |
| Paywalled | yes | no | no reuse | forbid |
| NDA / private | yes | no | n/a | forbid |
| Unknown | yes | no | n/a | omit |

[MUST] "May read locally" never implies "may ship". The shipped skill carries
**paraphrases and approved links**, never source text.

### 3.2 Good vs bad

```
BAD  (copied)  "As Chapter 4 states: 'A class shall never expose its mutable
                internals, for the exposure of state is the root of coupling...'"
GOOD (paraphrase + attribute)
               "Keep internals encapsulated; do not expose mutable state
                (per <source title>)."
```

---

## 4. Private-source handling

[MUST] For any source flagged private/NDA/personal in `INTAKE.md`:

- never include its name, URL, repo path, or code in the skill;
- never include org names, team names, or identifiers;
- keep its contribution as an unattributed general rule (or drop it if the rule
  itself is identifying).

[SHOULD] Keep private proofs in `work/` only, which is never shipped.

---

## 5. Keep acquired sources out of the published skill

[MUST] Choose one and state it in `PROVENANCE.md`:

- (a) `sources/` and `work/` are separate and **gitignored** — recommended; or
- (b) **delete** them after distillation.

[MUST] The published skill directory must build and read with `sources/` and
`work/` **absent**. Test this: move them away and re-read the skill; if anything
breaks, it was leaking.

---

## 6. Provenance note (ships, sanitized)

[NICE] Ship a short `PROVENANCE.md` next to `SKILL.md`:

```markdown
# Provenance
This skill synthesizes rules from the following source TYPES (names/links per the
user's link policy). No source text is reproduced; all rules are paraphrased.
- Primary: a long-form source on <topic>. License: <license or omitted>.
- Secondary: <N> articles/docs. Links: <approved / omitted>.
- Code: analysis of <N> repositories (identifiers omitted / per policy).
Generated: <YYYY-MM-DD>. Review link policy before redistributing.
```

[MUST] This note must not contain forbidden links, private names, or paths.

---

## 7. Pre-publication redaction checklist

[MUST] Run these greps over the **shipped skill directory** and confirm ZERO
unexpected hits. Replace `<skill>` with the skill dir.

```bash
# source folders / staging
grep -rInE 'sources/|work/|/repos/|/articles/|/books/|extraction/|analysis/' <skill>

# absolute local paths
grep -rInE '([A-Za-z]:\\\\|/home/|/Users/|/root/)' <skill>

# secrets / credentials
grep -rInE 'api[_-]?key|secret|token|password|BEGIN [A-Z ]*PRIVATE KEY|AKIA[0-9A-Z]{16}' <skill>

# private URLs / internal hosts
grep -rInE 'https?://[^ ]*(internal|corp|local|\.lan|\.internal)|git@' <skill>

# org / repo / person identifiers (fill in from a blocklist you built from PROVENANCE)
grep -rInE '<private-org>|<private-repo>|<private-person>' <skill>

# every URL in the skill must be in the allowed list
grep -rInoE 'https?://[^ )"'"'"']+' <skill> | sort -u
```

[MUST] Cross-check every URL found against `LINK_POLICY.md` "Allowed". Any URL not
on the allowed list → remove it.

[MUST] If a pattern legitimately appears (e.g. the word "token" in normal prose),
record the justification in `work/LINK_POLICY.md` under a "known false positives"
section — do not silently ignore a hit.

### 7.1 What a clean result looks like

```
$ grep -rInE 'sources/|work/|/repos/|/articles/|/books/' <skill>
(no output)

$ grep -rInE '([A-Za-z]:\\\\|/home/|/Users/|/root/)' <skill>
(no output)

$ grep -rInoE 'https?://[^ )"]+' <skill> | sort -u
https://<approved-public-url-1>
https://<approved-public-url-2>
```

[MUST] Every URL listed by the last command must be present in `LINK_POLICY.md`
"Allowed". An empty URL list is perfectly acceptable — links are optional.

### 7.2 Common leaks to double-check

- an example that still contains a real repo name or contributor handle;
- a "how to run" snippet with an absolute developer path;
- a copied CI YAML with a corporate runner label or internal registry;
- an attribution line naming a private author or team;
- a `PROVENANCE.md` row that crept in with a forbidden link.

---

## 8. Sign-off

[MUST] Publishing is blocked until ALL are true:

- [ ] All source folder/file references removed from the shipped skill.
- [ ] No absolute local paths in the shipped skill.
- [ ] No secrets, keys, or credentials anywhere in the shipped skill.
- [ ] No private repo/org/person identifiers.
- [ ] Every URL in the skill is explicitly allowed by the user.
- [ ] No verbatim copyrighted passages; rules are paraphrased + attributed.
- [ ] `PROVENANCE.md` present (if used) and sanitized.
- [ ] `sources/` + `work/` gitignored or deleted; skill reads without them.
- [ ] User reviewed and approved the link policy + final artifact.

Only then is the skill safe to publish or install.
