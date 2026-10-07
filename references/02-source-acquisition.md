# 02 — Source Acquisition

Goal: gather every source into a **local, gitignored staging area** with recorded
provenance, without contaminating the final skill and without violating licenses.

[MUST] Nothing acquired here is ever committed or shipped. `sources/` and `work/`
are disposable staging; the produced skill must be self-contained without them
(enforced in file 06).

---

## 1. Layout

[MUST] Use this staging layout (or an equivalent that is equally isolated):

```
<project-root>/
  sources/                 # gitignored — NEVER shipped
    repos/                 # shallow clones
    articles/              # downloaded + converted articles
    docs/                  # specs, wikis, style guides
    books/                 # PDFs + OCR output
  work/                    # gitignored — pipeline working files
    INTAKE.md
    BRIEF.md
    extraction/
    analysis/
    rulebook.md
  <final-skill-dir>/       # what actually ships
    SKILL.md
    references/
```

### 1.1 `.gitignore`

[MUST] Before any download, write:

```gitignore
sources/
work/
*.tmp
```

[SHOULD] If the eventual skill lives in a fresh repo, add a short `PROVENANCE.md`
(see §5) that ships; the raw sources never do.

---

## 2. Shallow-clone repositories

[MUST] Clone **shallow** — we want code to read, not history:

```bash
git clone --depth 1 https://<host>/<org>/<repo>.git sources/repos/<repo>
```

[SHOULD] For a monorepo or huge repo, sparse-checkout to limit context:

```bash
git clone --depth 1 --filter=blob:none --sparse <url> sources/repos/<repo>
cd sources/repos/<repo>
git sparse-checkout set src test .github
```

[NICE] Record the exact commit for provenance:

```bash
git -C sources/repos/<repo> rev-parse HEAD
```

[MUST] Never clone over SSH with personal keys the user has not authorized; prefer
HTTPS for public repos, and for private repos ask the user how to authenticate.

---

## 3. Download documents and articles

[MUST] Fetch only sources the user listed in `INTAKE.md`. Do not crawl the web.

```bash
# one file
curl -L --fail --retry 3 -o sources/articles/<slug>.html "<url>"

# rate-limited polite batch (bash)
while read -r url; do
  slug=$(echo "$url" | sed 's|https\?://||; s|[/?]|_|g')
  curl -L --fail --retry 3 --compressed \
       -A "skill-research/1.0 (contact: <user>)" \
       -o "sources/articles/${slug}.html" "$url"
  sleep 1
done < sources/article-urls.txt
```

[SHOULD] Respect `robots.txt`, `Retry-After`, and site Terms of Service. If a site
forbids automated fetching, ask the user to download it manually.

---

## 4. Convert to a reading format

[MUST] Normalize everything to Markdown so extraction agents read uniformly and
cheaply. Convert HTML → Markdown and OCR scanned books.

### HTML → Markdown

```bash
# pip install html2text  (or use pandoc if already present)
python -m html2text --ignore-links sources/articles/<slug>.html \
  > sources/articles/<slug>.md
```

```bash
# alternative if pandoc is available
pandoc -f html -t gfm -o sources/articles/<slug>.md sources/articles/<slug>.html
```

### PDFs

[MUST] Try a text layer first; OCR only if the PDF is a scan.

```bash
# text-layer extraction
pdftotext -layout sources/books/<book>.pdf sources/books/<book>.txt

# scanned PDF -> OCR (tesseract)
pdftoppm -r 300 -png sources/books/<book>.pdf /tmp/pg
for f in /tmp/pg-*.png; do
  tesseract "$f" "${f%.png}" -l eng
done
cat /tmp/pg-*.txt > sources/books/<book>.txt
```

[SHOULD] Clean OCR noise (headers/footers, hyphenation, page numbers) before
extraction; a messy primary source produces a messy rulebook.

### Chunk long sources

[MUST] Split very long texts into stable, numbered chunks so each extraction agent
has a **bounded** input:

```bash
# ~1500 lines per chunk
split -l 1500 sources/books/<book>.txt sources/books/<book>.part-
```

---

## 5. Record provenance

[MUST] For every acquired item, write a row in `work/PROVENANCE.md`:

```markdown
| id | title | type | location (url/path) | retrieved (YYYY-MM-DD) | commit/sha | license | public? |
|----|-------|------|---------------------|------------------------|------------|---------|---------|
| P1 | <book/spec title> | book | <url> | 2026-10-02 | - | CC-BY-4.0 | yes |
| A07| <article> | article | <url> | 2026-10-02 | - | © author | yes |
| R2 | <repo> | repo | <url> | 2026-10-02 | <sha> | Apache-2.0 | yes |
```

[MUST] Capture the license (or "unknown / © author"). If a license is missing or
restrictive, flag it to the user — you may still *read* it locally, but you may not
ship its text (file 06).

---

## 6. When acquisition fails

[MUST] Handle failures explicitly; never leave a silently truncated source.

| Symptom | Action |
|---------|--------|
| `curl` 403/404 | record it in `PROVENANCE.md` as `failed`; ask the user for a manual copy |
| site forbids scraping (`robots.txt`, ToS) | stop; ask the user to download manually |
| private repo, no auth | ask the user how to authenticate; do NOT attempt credentials |
| PDF has no text layer | OCR it (§4); if OCR is poor, flag to the user |
| source is an SPA / JS-rendered | ask the user for a static export or PDF |
| huge download | prefer the shallow/sparse options in §2 |

[SHOULD] Verify downloads are non-empty and plausible before extraction:

```bash
# find suspiciously small files (< 1 KB) that likely failed
find sources -type f -size -1k -print
```

---

## 7. Chunk metadata

[MUST] When chunking long sources, keep an index so extraction agents can cite
positions deterministically. Write `sources/books/<book>.parts.md`:

```
<book>.part-00 : lines 1-1500      (ch1..ch3)
<book>.part-01 : lines 1501-3000   (ch4..ch6)
```

[SHOULD] Prefer chunking on natural boundaries (chapters/sections) over fixed line
counts when the format allows — it keeps findings coherent.

---

## 8. Pre-extraction checklist

- [ ] `sources/` and `work/` exist and are gitignored.
- [ ] Every repo is a shallow clone; provenance has the commit SHA.
- [ ] Articles/docs downloaded only from user-approved URLs.
- [ ] Everything converted to Markdown/text; long sources chunked.
- [ ] `work/PROVENANCE.md` has a row per source with license + public flag.
- [ ] No secrets, tokens, or personal data were written into `sources/` or `work/`.
- [ ] User confirms acquisition is complete (or pauses for missing items).

Proceed to `03-knowledge-extraction.md`.
