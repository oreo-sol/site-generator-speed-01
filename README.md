[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/oreo-sol/site-generator-speed-01/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/oreo-sol/site-generator-speed-01/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

Entry for [Build a static site faster, 22 November 2026](https://make.fun/bounty/site-generator-speed-01-run-20261122) on make.fun.

# site-generator-speed-01 — a faster drop-in for makefun-site 1.0.0

`run(input)` returns exactly makefun-site 1.0.0's answer for a generated site
`{ site, templates, files }`: the full output tree — every page's `index.html`,
the index and tag pages, `build-report.txt`, and each static file — byte for
byte. It is just much faster about it.

## How it is fast

The reference walks every source through several passes (lex, then render,
then join) and re-checks the same things per page. This solver does one
left-to-right pass per source instead:

- Markdown is rendered in a single pass over the source's lines — no
  lex-token array, no body re-join; paragraphs, lists and blockquotes
  accumulate into one string, and word counts are taken by a hand scan in
  the same pass.
- Inline markup is rendered with native `indexOf` jumps between the next
  interesting character instead of a per-character loop; plain runs are
  escaped in bulk with a module-level regex.
- Slugify has an ASCII fast path plus a per-case cache; cards and word
  counts are cached per build; link resolutions are cached per build
  (broken links are still reported once, like the reference).
- Templates are parsed once per distinct source per case, and rendered
  with reused scope objects and a base-scope chain instead of a fresh
  spread per render. A case is answered alone: both caches are cleared
  at the start of every `run()`, so nothing is carried from one case to
  the next.

## Verified

- Practice set: 3/3 cases byte-identical to the reference's
  `practice/reference/expected.json` (canonical JSON compare, as the
  scorer does it).
- Differential fuzzing against the reference libraries (inline, Markdown,
  front matter, templates, full builds on random sites, three seeds):
  no mismatches.
- Warm `run()` on this machine: ~270 ms for the 3 practice cases vs
  ~1,080 ms for the house baseline — about 4× faster.
