# routing

> Repository: `botopink/routing` (`git@github.com:botopink/routing.git`) · in the meta checkout: `repository/routing/`

The `routing` library (decisions 115, 116, 117): the route matcher and the
routing wires that the server and the browser must both run, **one** implementation
compiled for erlang and commonJS (`contracts.md § 1`). Neutral like std — no framework
imports another to get the matcher; both import it by name,
`import {match.matchPath, table.parseTable} from "routing";`.

**A library of its own** (decision 326). It was bundled with the compiler until
`03-bundled-libs/138` moved it here with its history; the compiler now embeds std alone.
A program that imports `from "routing"` declares it in `dependencies` (decision 242) —
`{ "routing": { "git": "https://github.com/botopink/routing.git", "branch": "feat" } }`; inside the meta checkout that entry
resolves by name through the `repository/` root (`repository/routing`), elsewhere
through the install store. Without the entry, `from "routing"` is
`unresolved import source "routing" — declare it in botopink.json "dependencies"`.

Pure `.bp` only (decision 117 rule 8): no `#[@External]` cell, no `declare fn`, no
`.erl` / `.mjs` sidecar, and no framework name anywhere under `src/`. It imports `std`
and nothing else (`url_rules` uses `encoding.percentDecode` / `percentEncode`, `match`
uses `collections.Dict`).

## Tree

```text
routing/
├── AGENTS.md
├── botopink.json      ← "name": "routing", "target": "erlang", "targets": ["erlang", "commonJS"], `files` = every src module
├── src/
│   ├── root.bp        ← `pub mod` per module, a module before the siblings that import it
│   ├── segment.bp     ← SegmentKind, Segment, parseSegment, kindName, pathProblem, trimSlashes, parsePath, patternOf, slotOf, paramNamesOf, fillPattern, toColonPattern
│   ├── table.bp       ← RouteEntry, writeTable, parseTable, kindLabel            (the route table wire, `kind|pattern|slot|verb`)
│   ├── match.bp       ← RouteMatch, matchPath, layoutChain, paramOf, paramsOf, splitPath, attempt, Attempt, segmentWeight, scoreBeats, ancestorPatterns
│   ├── route_kinds.bp ← RouteKind { Static, Dynamic }, writeKinds, parseKinds, routeKindOf   (the `k` blob, `pattern|S|D`)
│   ├── slot_states.bp ← SlotState { Matched, Defaulted, Unchanged, Empty }, writeSlotStates, parseSlotStates   (the `z` blob, `slot|pattern|M|D|U|E`)
│   ├── url_rules.bp   ← PathRules, canonicalize, clientHref, RedirectRule, writeRedirectTable, parseRedirectTable   (`source|destination|1|0`)
│   ├── navigation.bp  ← NavKind { None, NotFound, Redirect }, NavOutcome, signalReason, signalFromReason, isSignalReason, signalPrefixes, signalToWire, signalFromWire
│   ├── pattern.bp     ← PatternSegment { Literal, Param, Rest }, parsePattern, matchPattern, patternProblem   (the `:param` grammar)
│   └── conventions.bp ← ConventionFile, fileKinds, kindOf, kindLetter, classify, conventionConflicts   (the app-file conventions)
└── test/              ← one `<module>_test.bp` per module, suite `routing:`
```

## Wires and rules

| Module | Wire / rule | Reading | Writing |
|---|---|---|---|
| `table` | `kind\|pattern\|slot\|verb`, trailing empty fields dropped; kinds `L T P D R S E N` | a short line reads as empty fields | — |
| `route_kinds` | `pattern\|K`, `K` = `S` or `D` | tolerant: a malformed line is skipped; an unnamed pattern is `Dynamic` | strict: `\|`/newline in a pattern halts |
| `slot_states` | `slot\|pattern\|state`, state `M D U E` | tolerant: a line without 3 fields is skipped; an unknown letter is `Empty` | strict: `\|`/newline in a slot or pattern halts |
| `url_rules` | `source\|destination\|1\|0` | tolerant | strict, naming the rule |
| `navigation` | reasons `nav:not-found`, `nav:redirect:<loc>` (307), `nav:permanent-redirect:<loc>` (308), `nav:see-other:<loc>` (303); wire `""` / `N` / `R\|<status>\|<loc>` | the wire is tolerant (anything else is `None`; the location keeps its `\|`); a `nav:` reason with an unknown verb halts, naming the verb | `signalReason(None)` and a redirect status outside 303/307/308 halt |
| `pattern` | literal, `:param`, trailing `:param*` | anything else (`*`, `[`, `(`, `?`, `{`, a bare `:`) is an `Error` whose text is `patternProblem`'s; `:param*` not last is an `Error` | — |

| `conventions` | eight kinds in wrap order (decision 171): `layout template error loading not-found page default route`; a kind's file is `<kind>.bp`; `kindLetter` is `table.kindLabel`'s inverse (172) | `classify(appDir, path)` takes one path (173), never a disk: `null` outside `appDir` (compared part by part, = `startsWith(appDir + "/")`), for a name no kind has, under a `_folder` | `conventionConflicts(files)`: page beside route in one segment, then one URL claimed by two root `(group)`s with a page; a segment `pathProblem` refuses is skipped; texts `routing: …`, ASCII |
| `segment` helpers | `paramNamesOf(path)` reads through `parseSegment` (`[a.b]` binds `a.b`, unclosed `[x` binds nothing); `toColonPattern` swaps a segment's outer `[`…`]` pair for `:` (`[...r]` → `:...r`) | — | `fillPattern(pattern, bindings)`: `Error` for a name bound twice, a required name unbound, `[x]` bound to `""` or a `/`-holding value, `[...x]` bound to `""`, a binding the pattern lacks; `[[...x]]` unbound or `""` contributes nothing |

`url_rules.canonicalize` strips `basePath`, applies `trailingSlash`, then percent-decodes
**once**; the query / fragment (from the first `?` or `#`) pass through untouched; a path
outside `basePath` is answered unchanged; an undecodable escape keeps its raw text.
`clientHref` percent-encodes each segment, applies `trailingSlash`, prefixes `basePath`.
The empty `:param` pattern matches only `/` — a caller whose "no pattern" means "every
path" says so itself.

## Testing

```sh
../botopink-lang/zig-out/bin/botopink test --target erlang
../botopink-lang/zig-out/bin/botopink test --target commonJS
../botopink-lang/zig-out/bin/botopink format --check src test
```

`zig build test-libs` discovers the package as the cells `routing · erlang` and
`routing · commonJS`. Every expected wire string in the tests is a literal, never a second
call's output. Enum values are compared through a local `case` helper, arrays element by
element (`==` on arrays is reference equality).

## Language notes (measured on the compiler this library landed on)

- The erlang backend refuses a `try` statement inside a `while` body that also reassigns
  a `var` (`variable 'I@2' unsafe in 'case'`) — loops in tests use `assert`, not `try`.
- A `throw` inside a `while` that mutates `var`s, and a `case` over
  `xs.at(i).unwrapOr(Enum.Variant(…))` inside a loop, do not compile on erlang — the
  parser and the matcher compute a problem string first and `throw` after the loop, and
  read enum payloads through a private `case`-returning helper.
- On commonJS a `case (r) { .Ok(_) -> return a; .Error(e) -> return e; }` statement as a
  function's last statement fails with `JumpInValuePosition`; bind a `case r { Ok(_) ->
  a; Error(e) -> e; }` expression instead.
- `asserts.throwsWith` cannot match a needle through a non-ASCII refusal text on erlang
  (the harness reads the `~p` rendering), so every refusal text is ASCII.
- `String.slice` counts bytes on erlang and UTF-16 units on commonJS; every substring goes
  through a private `charOf` / `sub` pair.

## Local gate

`scripts/git-hooks/pre-commit` is the tracked pre-commit gate, self-contained:
it sources `scripts/git-hooks/lib/runner-standalone.sh` from this repository and
reaches nothing outside it, so a standalone clone, a checkout inside the botopink
meta workspace and a worktree run the same gate. Install it once per clone:

```sh
git config core.hooksPath scripts/git-hooks
```

The repository is one plain package, so the gate's test stage runs
`botopink test --target <t>` at the root on each target `botopink.json` declares
(`erlang`, `commonJS`). Never commit with `--no-verify`; fix the red instead.
`scripts/git-hooks/pre-commit` and `scripts/git-hooks/lib/runner-standalone.sh`
are one text across every library repository: the meta repository's
`hook-integrity` workflow compares the bytes (its check 4), so a change to either
lands in all of them together. CI: `.github/workflows/test.yml` runs the same
package on linux and macos, on each declared target, with the compiler built from
`botopink/botopink-lang` `feat`.
