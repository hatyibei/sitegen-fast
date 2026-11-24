# sitegen-fast

A faster drop-in for **makefun-site 1.0.0**, make.fun's own static site generator,
entered in make.fun's *Build a static site faster, 22 November 2026* bounty
(`site-generator-speed-01-run-20261122`).

`solve.js` exports `run(input)`, which returns exactly the output tree that
makefun-site 1.0.0's `build` returns for the same input — every page's
`index.html`, the index and tag pages, `build-report.txt` and each static file,
byte for byte. `main.ts` is the harness driver: it reads the cases from
`/input/cases.json`, answers each with `run`, and writes `/output/answers.json`.

## How it is faster

The dialect is unchanged (makefun-site's own front matter, Markdown subset,
template language, slug rule and report); the machinery is rebuilt:

- **Markdown inline rendering** scans for the next special character and copies
  plain runs in bulk, escaping them in a single pass, instead of processing
  character by character.
- **Block lexing** classifies lines with hand-rolled character checks instead
  of a battery of regular expressions per line.
- **Templates are compiled** to plain JavaScript functions (text becomes
  literals, lookups call one resolver, loops become real loops), and repeated
  `{% include %}` renders whose observed lookups are unchanged are served from
  a dependency-tracked memo — headers, footers and nav partials come out
  identical for every page of a site, so they are rendered once per site.
- **Slugs** are memoised per build (titles, tags and headings repeat), with an
  ASCII fast path that skips Unicode normalisation.
- **Front matter** is scanned in place; the body is the untouched tail of the
  source text.

Verified byte-identical to makefun-site 1.0.0 on the kit's practice set and on
dozens of freshly drawn generator instances, and about 3.4× faster than the
reference on the practice set (run time only).

## Layout

- `solve.js` — the entry: a dependency-free ES module, Node 22, no I/O.
- `main.ts` — the harness driver, unchanged from the house baseline's.
- `package.json` — `{ "type": "module" }`.

## Licence

MIT (see `LICENSE`). Behaviour follows makefun-site 1.0.0's published spec and
code; see `NOTICE` for the upstream credits.
