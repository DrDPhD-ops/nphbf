# research/sites — email harvest from this remote session

100 files, `row-NNNN.json`, one per row of `research/people-table-enriched.csv`
whose `Email_Status` was not `verified`. **23 rows carry an address (20 distinct
people); 77 are `not found`.** `people-table-enriched.csv` was not modified.

## Why the yield is low: the network

See `research/egress-check.md`. Everything the task's priority list depends on —
personal sites, LinkedIn, ResearchGate, ORCID, DBLP, OpenReview, ACL Anthology,
arXiv, Crossref, OpenAlex, Semantic Scholar, Google — is blocked by the egress
policy, for `curl` and `WebFetch` alike. Only three channels worked:

- `raw.githubusercontent.com` (any public repo) — the only channel that lets us
  actually **retrieve** a page and read an address off it;
- GitHub code/user search — returns file fragments, often with addresses;
- `WebSearch` — returns titles, URLs and snippets, but the page behind a result
  cannot be opened here.

So "look at the personal site and grep `mailto:`" was not executable. What
replaced it was GitHub: profile READMEs, `pyproject.toml`, `CITATION.cff`,
GitHub-Pages `index.html`, and conference-org repos.

## Verification bar

No address was ever constructed from a template. Two tiers were accepted:

- **Retrieved** — read verbatim out of a file actually fetched from
  `raw.githubusercontent.com`. 11 of the 23 rows.
- **Reported** — appeared verbatim in `WebSearch` output for a page that is
  egress-blocked. Accepted only when the evidence contains something that
  **cannot be derived from the person's name plus the institution domain**:
  either a non-obvious local part (`hagosg81@`, `liuyj626@`, `ethanqiao@`,
  `el.gromenko@`, `tracyliu@`, `jeanmatondo@`) or a co-located contact datum such
  as a phone or office number that shows the block was read off a real page
  (rows 28/29, 32/38, 170).

Anything failing that bar was set to `not found`, with the candidate preserved in
`evidence`. Also rejected throughout: masked aggregator addresses
(`m******@internews.org`), `@users.noreply.github.com`, and every same-name
stranger.

## Promotable candidates (a session that can open these pages should re-check)

These are **not** in the data as addresses — they are leads recorded in
`evidence`:

| row | person | candidate | blocker |
|-----|--------|-----------|---------|
| 64 | Cassady Shoaff | `shoaffc@montclair.edu` | montclair.edu blocked; matches the lastname+initial convention |
| 85 | Matteo Fabbri | `matteo.fabbri@imtlucca.it` | ceur-ws.org / iris.imtlucca.it blocked; `firstname.lastname@` shaped |
| 141 | Hyunjin Hwang | `hyunjinhwang@kaist.ac.kr` | printed in `thewebconf2026/www2026` session-chairs CSV, but the DOVE paper's authors are SKKU/MSRA/SUTD with no KAIST affiliation — identity not closed |
| 220 | Valery Shulginov | `shulginov.va@mipt.ru` | dialogue-conf.org blocked; derivable from name+patronymic, and the domain (MIPT) contradicts the stated HSE affiliation |
| 114, 115, 116 | Meng-Xi Guo, Lei-Ming Gao, W. Zeng | — | one MDPI page (`10.3390/bdcc10090290`) prints corresponding-author emails and would likely resolve all three at once |
| 24, 37, 54, 72, 82, 120, 210 | — | masked on RocketReach/ZoomInfo | the address exists but is starred out |

## Data-quality problems found in the source table

- **Rows 32 & 38** are the same person (Eduardo de Miguel Vázquez); likewise
  **28 & 29**, **91 & 98**, and probably **41 & 45** (Ibrahim Ansari).
- **Row 17** — the `Profile URLs` value is
  `linkedin.com/in/abolarin-bolare-b330a427a`, which does not correspond to the
  name "Alice Bai". The row's identity anchor is itself unreliable.
- **Rows 128/129** — the `Organization` field contains `xcfeng@ir…`, which is
  **Xiaocheng Feng's** address (the paper's corresponding author), not Xiayu
  Cao's or Zi-Han Zhang's. Anyone using that field as an email will be wrong.
- **Row 140** — "Jingtao Yao" is a corrupted extraction of **"Jing Yao"**
  (Microsoft Research Asia). No "Jingtao Yao" is on arXiv 2604.06210.
- **Row 111** — Semantic Scholar merged **both Dandekar brothers** into one
  author id (`2320313160`, "R. Dandekar"), listed twice on 2410.01811. The
  recorded address belongs to **Raj Abhijit** Dandekar (verbatim name binding);
  Rajat Dandekar's `rajatdandekar@vizuara.com` appears in
  `VizuaraAILabs/Vizuara-Agents-10Day-Bootcamp`.
- **Row 12** — three distinct personal addresses appear across his repos; the one
  recorded is the one printed next to this row's LinkedIn URL.
- Initialled names resolved along the way: **row 123** `Y. Lim` = Yongtaek Lim;
  **row 168** `C. Kenntemich` = Christoph Kenntemich; **row 169**
  `D.O.I. Brückner-Collet` = Daan Oriah Israel Brückner-Collet; **row 216**
  `D. Kalacheva` = Daria Kalacheva; **row 219** `O. Moroz` = Oksana Vladimirovna
  Moroz.

## Incidental security finding

`shaikhshaibaj100-sys/shakihshaibajcode` has a Groq API key committed in
plaintext (`gsk_kxm5…`). Unrelated to this task, noted because it was seen.

---

# Passes 2 and 3 (after the table was merged to 288 rows)

The table changed under this work: nine duplicate rows were merged upstream and
the `Organization` column was repaired, so the first pass's `row` numbers no
longer address the same people. **Every pass-1 file now also carries
`row_new_288`**, its row in the current table. Pass-2 and pass-3 files are keyed
to the 288-row table directly.

- `row-NNNN.json`    — pass 1, GitHub-first. Key by `row_new_288`.
- `p2-row-NNNN.json` — pass 2, WebSearch by surname + affiliation. 71 rows.
- `p3-row-NNNN.json` — pass 3, arXiv title pages via alphaXiv. 24 rows.

**Result: 22 rows of the current table carry an address; 69 remain open.**

## Pass 3 found a channel that was assumed dead

`arxiv.org` is egress-blocked, but the **alphaXiv MCP tools read arXiv PDFs
anyway**, and cost nothing against the WebSearch budget. That restores the
task's priority-5 source (arXiv full text). Any future pass with an arXiv id
should start there: `mcp__alphaXiv__answer_pdf_queries` returns the literal
title page, including correspondence footnotes.

What it showed is that the ceiling here is the papers, not the tooling. Of the
distinct papers read, most print **no address at all**, and the rest print only
the corresponding author's. Addresses that belong to a *co-author* and must
never be attached to these rows: `xcfeng@ir.hit.edu.cn`, `yfye@ir.hit.edu.cn`
(CultureForest), `mwkim@selectstar.ai`, `minjae.jung@selectstar.ai` (DATUMO),
`zhouy131@cardiff.ac.uk` (BLEnD — Yi Zhou, not Nina White),
`yanghaiqin@sztu.edu.cn` (CultSportQA/NRITYAM), `jiseon_kim@kaist.ac.kr`,
`xiaoyuanyi@microsoft.com`, `jy.bak@skku.edu`.

## A trap that produced two wrong answers, both caught and reverted

Pass 3's row selection harvested arXiv ids with a regex over the row's text
columns. For nine rows the id came only from the `Evidence` column — where the
parent session had recorded it **while rejecting that author as a namesake**.
Two of those produced a confidently-wrong address before the provenance was
audited:

- row 53 (Vaibhav Mehta) → `vm353@cornell.edu` — the Cornell author the parent
  session had explicitly rejected.
- row 107 (Meng-Xi Guo) → `guomengxi.qoelab@bytedance.com` — the ByteDance
  video-compression namesake the parent session had explicitly rejected.

Both are reverted to `not found`, with the full reasoning in their `evidence`.
**Lesson for any future pass: an identifier taken from an `Evidence` column may
be there as a rejection, not as provenance. Check which column it came from.**

## One address rests on a weaker footing than the rest

Row 58, Cassady Shoaff, `shoaffc@montclair.edu`. Pass 1 dropped it because
`surname+initial@institution` is a derivable convention and montclair.edu cannot
be opened here. Pass 2 found it independently, and a second query on the literal
address string returned the specific faculty page that carries it — evidence the
string is indexed on that page rather than merely plausible. It is kept on that
basis, but it is the one entry that would fall first under a stricter bar.

Rejected on the same derivability test, recorded as candidates in `evidence`:
`matteo.fabbri@imtlucca.it` (row 79), `christoph.kenntemich@uni-giessen.de` and
`daan.brueckner-collet@uni-mannheim.de` (rows 160/161 — the Giessen domain also
contradicts his confirmed RPTU affiliation), `ekarinshak@gmail.com` (row 195),
`kklokova@hse.ru` (row 208), `ryan.kapma@neso.energy` (row 42, from a LeadIQ
org-wide pattern).

## Where the remaining 69 rows actually stand

They split into two populations, and neither is short of effort:

1. **LinkedIn-only practitioners** (freelancers, students, corporate staff).
   They have no author block and no directory entry, so the technique has
   nothing to bite on. Several are confirmed identities with a *masked*
   aggregator address — the address exists and is simply unreadable:
   `b******@aifourall.org` (row 48), `m******@internews.org` (row 74),
   `j******@hyundaielevator.com` (row 90), `****@ibu.edu.mk` (row 113),
   `e******@minohealth.org` (row 201), plus rows 24, 66, 76.
2. **Paper co-authors whose paper prints only the corresponding author.**
   Nothing short of a personal page or a mailing-list archive will close these.

Single highest-value fetches if egress ever opens: the MDPI article
`10.3390/bdcc10090290` (would likely resolve rows 107, 109, 115, 116 at once)
and the Dialogue-2025 PDF `GromenkoEetal.029.pdf` (rows 210, 211).

## Untried channel

`mcp__github__search_commits` works globally but needs a `repo:`/`org:`/`user:`
scope. The DATUMO repos (`selectstar-ai/CAGE-paper`,
`selectstar-ai/STAR-Teaming-paper`) and `YYF-Tommy/CultureForest` are public and
were never mined for committer addresses. `api.github.com/users/...`,
`list_commits` and GitHub profile HTML are all blocked — the session's GitHub
API is scoped to `drdphd-ops/nphbf`.

---

# How to merge these into the table

This session deliberately does **not** write to
`research/people-table-enriched.csv`. The 22 addresses live only in the per-row
JSON files here. Two traps to know before merging them:

**1. Row numbers moved.** Pass-1 files (`row-NNNN.json`) are keyed to the old
297-row table; use their `row_new_288` field instead. Pass-2 and pass-3 files
(`p2-`, `p3-`) are already keyed to the 288-row table. Match on the name as a
cross-check, not on the number alone.

**2. The CSV has mixed line endings.** Records are separated by `\r\n`, but some
fields contain bare `\n` inside them. Reading it through `io.StringIO(text)`
silently translates the `\r\n`, so a naive rewrite reformats all 289 records and
guarantees a conflict with whatever else is editing the file. This round-trips
byte-identically:

```python
src  = open(path, newline='').read()
rows = list(csv.reader(io.StringIO(src, newline='')))
buf  = io.StringIO(newline='')
csv.writer(buf, lineterminator='\r\n', quoting=csv.QUOTE_MINIMAL).writerows(rows)
assert buf.getvalue() == src          # holds before any edit
```

Set only `Email_Verified`, `Email_Status`, `Email_Source`, and prepend a
provenance note to `Evidence`. Skip any row already marked `verified` — none of
these 22 collide with one, but the guard is worth keeping.

A merge done this way changes exactly 22 records and touches 4 columns, taking
the table from 197 verified rows to 219.

## Two pre-existing rows worth a look while you are in there

- **Row 182 (Peng Zhang)** — the table has `zhangpeng_@fudan.edu.cn`; the older
  `enrich-rows-*` branches have `zhangpeng@fudan.edu.cn`, without the
  underscore. The arXiv author block prints `{zhangpeng_, lutun, ninggu}@fudan.edu.cn`,
  so the underscored form is the printed one. Keeping both, in the `a@x; b@y`
  form already used by row 43, is the safe option.
- **Row 280 (Seifeddine Hamdi)** — `Email_Status` is `verified` but
  `Email_Verified` holds the literal string `not found`. One of the two is wrong.
