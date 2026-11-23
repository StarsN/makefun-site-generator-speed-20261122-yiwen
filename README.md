[![site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/StarsN/makefun-site-generator-speed-20261122-yiwen/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/StarsN/makefun-site-generator-speed-20261122-yiwen/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)

# Fast site-generator

A faster drop-in for make.fun's own makefun-site 1.0.0, entered in
[Build a static site faster](https://make.fun/bounty/site-generator-speed-01-run-20261122).

`solve.js` exports `run(input)`, which returns exactly the answer makefun-site
1.0.0 returns for every case's input `{ site, templates, files }` — the output
tree (each page's `index.html`, the index and tag pages, `build-report.txt`
and each static file), byte for byte. `main.ts` is the program the harness
runs: it reads every case from `/input/cases.json`, answers each one with
`run`, and writes `/output/answers.json`.

How it is faster, without changing any answer:

- Markdown is lexed and rendered in one pass, dispatching on each line's
  first character instead of running several regexes per line.
- Inline rendering escapes whole runs at once (one scan, fast path when a
  run holds no escapable character) instead of escaping one character at a
  time.
- `slugify` takes an ASCII fast path (no NFKD, no combining-mark regex)
  and caches results within a case; tag slugs repeat heavily across posts.
- Each template is compiled once per case to a small JS function (plain JS,
  no dependency, no WASM): loop variables become locals, paths resolve
  through the same scope chain the reference walks, includes call the
  compiled template directly.
- Each page's card is built once and reused for its layout, the index, the
  tag pages and the recent list; word counting uses a fast ASCII path.

On the kit's practice set (3 cases, ~1,070 sources) it answers every case
identically to the reference and runs about 1.6x faster than the house
baseline's CPU time, measured locally with `resource.getrusage` on the
children; the same holds on two extra instances drawn from other drand
rounds with the kit's own generator (9 cases checked in total, every
output file identical).

## Layout

- `solve.js` — the fast generator, dependency-free ES module, no state kept
  from one call to the next.
- `main.ts` — the driver the harness runs (`node /bot/main.ts`).

## License

MIT. See [LICENSE](LICENSE).
