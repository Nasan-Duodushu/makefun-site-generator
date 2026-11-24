# site-generator-speed-01 — build notes ($100, closes 2026-11-25 04:00 CST)

Write solve.js, a faster drop-in for makefun-site 1.0.0: byte-identical
answers for every case AND faster than the house baseline at 95%
confidence (one-sided, over the final heat's repeated runs). 1st-10th
place $10 each; a house bot holding a paid place keeps its prize.

## What is known (verified 2026-11-24)
- Baseline = vendor/makefun-site (metalsmith-derived, MIT): index.js
  (11 KB) + lib/frontmatter.js, markdown.js, template.js, text.js.
  solve.js is an 807-byte wrapper calling lib.build(input).
- Practice set: 3 cases, 257/348/486 Markdown files, 21-27 templates.
- Baseline timing (this VM, node 24): ~2.8 s for all 3 cases (~0.9 s/case).
- Baseline passes the scorer 3/3 (verified with the kit scorer, no
  Docker needed — see "Local check" below).
- Repo cap: 4 MiB total. Scored set is drawn AFTER the deadline from a
  fresh drand round — optimisations must generalise, never overfit the
  practice cases.

## Local check (no Docker in this VM)
- Run: `MAKEFUN_INPUT_DIR=/tmp/sg-in MAKEFUN_OUTPUT_DIR=/tmp/sg-out
  node main.ts` from baselines/makefun-site/ (input = practice/input/cases.json).
- Score: `MAKEFUN_OUTPUT_DIR=/tmp/sg-out
  MAKEFUN_REFERENCE_DIR=<kit>/practice/reference
  MAKEFUN_SCORES_DIR=/tmp/sg-scores node <kit>/program/scorer/main.ts`
  → scores.json must read {"passed":1,"cases_passed":3,...}.
- The scorer compares canonical(answers[i]) to expected.json cases[i].output.

## Plan
1. ~~Set up repos/site-generator/~~ DONE 2026-11-24 ~17:15 run: main.ts + solve.js (baseline wrapper) + vendor/makefun-site/ + SOURCE.txt copied from the kit's baselines/makefun-site/ (84K total). NOTE: do NOT run profiling/timing while the tiny-chess v2 trial is still running (it owns the CPU; a loaded VM risks the engine's 10s/move wall clock). Profile only after the trial finishes.
2. Profile the baseline (node --cpu-prof) on the practice set. Prime
   suspects: template re-parsed per page (21-27 templates vs 257-486
   pages — compile once, cache), markdown re-parsed per file, slug
   normalisation, repeated regex in frontmatter/text.
3. Optimise with semantics preserved: cache compiled templates and
   parsed frontmatter shapes; avoid re-doing per-page work that only
   depends on the template; keep every output byte identical.
4. Re-run scorer after every change (3/3 or stop). Time 5+ runs of
   baseline vs candidate for a 95% read; target ≥ 2x faster.
5. Submit only if 3/3 AND clearly faster. Bundle + check + submit via
   the kit CLI (see ~/workspace/makefun/skill.md).

## Trial results — 2026-11-24 ~18:00 CST (research run, NOT submitted)

solve.js is now a fast re-implementation (original code, written in-session
from the baseline's spec; vendor/ stays as the credited reference and is no
longer imported). What changed vs the baseline wrapper:
- markdown inliner batches plain runs through one escapeHtml call (the
  profile's top cost: escapeHtml was ~20% of samples, called per character);
- lexer dispatches on the first character before touching a regex, fast
  trimLine / blockquote strip, exact CONTINUED regex kept;
- front matter parsed by index scanning, body taken by slice (no split+join);
- slugify memoised + ASCII fast path (skips normalize/mark-strip);
- wordsOf counts in one pass (falls back to match(/\S+/g) if non-ASCII);
- pageOf reuses the card already built for posts (wordsOf ran twice before);
- templates compiled once per build into straight-line functions
  (new Function, literals via JSON.stringify) with identical lookup /
  textOf / truthy / include-depth-8 semantics; render joins an array.

Verification (this VM, practice set, local scorer, no Docker):
- answers.json byte-identical to the baseline wrapper's output
  (md5 a6d0c9e69266278a32804ebf7e124292, both files);
- official scorer: {"passed":1,"cases_passed":3,"cases_total":3};
- interleaved A/B, 5 rounds each, back to back: baseline 1025/1018/897/
  1014/964 ms (mean 984, median 1014); candidate 531/617/562/525/541 ms
  (mean 555, median 541). Ranges do not overlap: ~1.8x faster, clearly
  faster at any reasonable confidence — passes the competition's 95%
  one-sided rule. The internal 2x target was not quite reached (1.8-1.9x);
  remaining cost is diffuse (harness JSON parse/stringify ~15%, escapeHtml,
  GC), further gains would be small and risk semantic drift.
- Baseline solve.js kept at /tmp/sg-solve-baseline.js; baseline answers at
  /tmp/sg-baseline-answers.json (same md5 as above).

Submit path (for the hourly cron / main chat, NOT done here — trial first):
repo makefun-site-generator does not exist yet; push main.ts + solve.js +
vendor/makefun-site/ + SOURCE.txt (+ LICENSE), bundle + check + submit via
the kit CLI (see ~/workspace/makefun/skill.md). Watch the GitHub secondary
rate limit (~7-9 repos/hour) and the one-liner queue ahead of it. Deadline
2026-11-25 04:00 CST.

## Standing rules (user)
- Original implementation in-session; study the best comparable work
  first (here: the baseline itself + Hugo/Eleventy/Zola approaches the
  rules name), then implement originally. Do not paste the baseline's
  code as "your" solve.js and call it done — the entry must be a
  genuinely faster re-implementation; vendored MIT code stays credited.
- Free only; this one is $0 fee. Deadline 2026-11-25 04:00 CST — after
  the GTA submission (23:00 CST) this is the next code task.
