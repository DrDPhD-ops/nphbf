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
