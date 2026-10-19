# routing — conformance

> Path: `repository/routing/conformance/`
> Sibling (AGENTS): [`./AGENTS.md`](AGENTS.md)

The reference of `routing`, the route matcher and the routing wires the server and the browser both run (decisions 115,
116, 117). It is one implementation compiled for erlang and commonJS, neutral like std — no HTTP, no state, no host cell,
no framework name — so the server that writes a route table and the browser that reads it agree on what a segment is,
which route a URL is and what a redirect means. A library of its own since decision 326: a program that imports it
declares it in `dependencies`. Every example is a cell (decisions 457, 461, 465): the decision cells under
`modules/decisions-taken/` are the programs this reference points at, and a table, a grammar or a wire format is fenced as
`text`, never as code. The library holds no feature cells yet: they come with front 157's Conformance step.

| Section | What |
|---|---|
| *The package* | what a program declares and imports to use `routing` |
| *segment* | the file-convention segment grammar: `[x]`, `[...x]`, `[[...x]]`, `(group)`, `@slot`, `_private` |
| *table* | the route table and its line-oriented wire |
| *match* | the matcher: precedence, capture, the layout chain |
| *route_kinds*, *slot_states* | the `k` blob and the `z` blob |
| *url_rules* | `basePath`, `trailingSlash`, the redirect table |
| *navigation* | the `nav:` reasons and the `n` wire |
| *pattern* | the `:param` grammar |
| *conventions* | the app-file conventions |
| *Departures* | the spellings `src/` still writes that are retired |
| *Decisions* | the decision cells, one directory per code-bearing row of the ledger |

## The package

A program declares `routing` in `dependencies` and imports a name from the module that declares it, with `from`, as for
any package (decision 206): `import {match.matchPath, table.parseTable} from "routing";`. A module of the program's
own package is imported by its path in braces, with no `from`, and may carry the name of a library module without
clash — `import {table.banner};` is the program's, `import {table.RouteEntry} from "routing";` the library's —
[own_module_import_without_from-206](modules/decisions-taken/own_module_import_without_from-206/draft/main.bp). The
bare shorthand `import {matchPath};` is refused —
[shorthand_import_refused-337](modules/decisions-taken/shorthand_import_refused-337/draft/main.bp). The library imports std
and nothing else (`url_rules` uses std's `encoding`, `match` its `collections.Dict`), and it is pure `.bp`.

A routing wire the build writes can be read where it is needed whatever the stage: a function under `comptime` runs the
library on the wasm comptime runtime (decision 594) —
[libraries_run_at_build_on_wasm-594](modules/decisions-taken/libraries_run_at_build_on_wasm-594/draft/main.bp). An
optional the library hands back (`matchPath`, `classify`) is read with the operators of `?T`, never with methods
(decision 330) — [optional_typescript_operators-330](modules/decisions-taken/optional_typescript_operators-330/draft/main.bp).

Every refusal the library writes is ASCII text beginning `routing: `. A function that checks its input and has no
`Result` to carry the refusal halts with that text (a failed `assert`); `fillPattern` and `parsePattern` answer a
`Result` whose error is that text.

## segment

The grammar of one folder name in an app tree. Four spellings are not URL segments: `(group)` is transparent, `@slot`
renders into a named prop of the parent layout, `_private` is excluded from routing entirely, and `[dynamic]`,
`[...catchAll]` and `[[...optionalCatchAll]]` capture instead of matching.

- `SegmentKind` is the enum `Static`, `Dynamic`, `CatchAll`, `OptionalCatchAll`, `Group`, `Slot`, `Private`;
  `Segment(kind, name, raw)` is one classified segment, `name` the capture's name or the group's or slot's.
  `kindName(kind)` is the kind as a word.
- `parseSegment(raw)` is the one place the bracket and parenthesis spellings are decoded. It is total: a name that may not
  appear on a registered path is classified here and refused by `pathProblem`. An unclosed `[x` is a static segment.
- `pathProblem(seg)` is `""` when a path is registrable and the refusal text otherwise: a segment holding `|` (the route
  table's field separator) or a newline (its record separator), an empty segment, a `_`-prefixed private folder.
  `parsePath(seg)` reads a path into its `Segment`s and halts with that text on a bad one; `""` is the root and answers the
  empty list. `trimSlashes(seg)` drops the leading and trailing slashes.
- `patternOf(segments)` is the URL pattern — groups and slots removed, the bracket spelling kept, `/` for the root.
  `slotOf(segments)` is `""` for the children slot and the slot's name for an `@slot` entry; a path holds at most one, the
  innermost wins (`dashboard/@team/settings` is a `team` entry).
- `paramNamesOf(path)` is the names a path's capturing segments bind, in path order, read through `parseSegment`
  (`[a.b]` binds `a.b`; an unclosed `[x` binds nothing). The cell
  [route_read_through_hooks-293](modules/decisions-taken/route_read_through_hooks-293/draft/main.bp) reads the fields of
  the record a page's `params` hook answers off `/produto/[id]` with it.
- `fillPattern(pattern, bindings) -> Result<string, string>` is the URL a pattern names once `bindings`, a list of
  `#(name, value)`, fill its capturing segments, groups and slots dropped. `[x]` needs a binding that is neither empty nor
  holds a `/`; `[...x]` a non-empty one, which may span segments; `[[...x]]` may be unbound or bound to `""` and then
  contributes nothing. It answers an `Error` naming the problem for a name bound twice, a required name unbound, a
  binding the pattern does not have, `[x]` bound to `""` or to a value with a `/`, `[...x]` bound to `""`.
- `toColonPattern(pattern)` swaps each segment's outer `[`…`]` pair for a leading `:` (`[id]` is `:id`, `[...rest]` is
  `:...rest`, `[[...x]]` is `:[...x]`); every other segment and the separators are kept byte for byte.

The cells [bpp_page_routing_convention-203](modules/decisions-taken/bpp_page_routing_convention-203/draft/main.bp) and
[route_handler_never_exported-273](modules/decisions-taken/route_handler_never_exported-273/draft/main.bp) print the
pattern `parsePath` and `patternOf` make of an app directory.

## table

The route table crosses the boundary as a line-oriented blob, not JSON: a line split is the same operation on the BEAM and
in the browser. One record per line, in registration order:

```text
kind|pattern|slot|verb
```

`kind` is one letter: `L` layout, `T` template, `P` page, `D` default, `R` route handler, `S` loading, `E` error, `N`
not-found. `pattern` is the URL pattern with groups and slots removed and the bracket spelling kept. `slot` is `""` for the
children slot and the slot's name for an `@slot` entry. `verb` is `""` for a UI entry and the HTTP method for an `R` one.
Trailing empty fields are dropped, so no line ends with a `|` and the common record is `kind|pattern`.

- `RouteEntry(kind, pattern, slot, verb)` is one record. `writeTable(entries)` joins them with `\n` in order;
  `parseTable(wire)` reads them back, a short line as empty fields, the empty wire as no entries, so a round trip is
  exact.
- `kindLabel(letter)` is a letter's long name (`P` is `page`), `""` for a letter the table does not have.

[own_module_import_without_from-206](modules/decisions-taken/own_module_import_without_from-206/draft/main.bp) prints a
route table written with `writeTable`.

## match

One function, both targets, one precedence order, applied segment by segment, highest first: a static segment, then a
dynamic one, then a catch-all, then an optional catch-all. Groups are not in a pattern, and an entry carrying a slot is not
a candidate (a slot is matched separately against the same URL).

- `matchPath(table, pathname) -> ?RouteMatch` is the route a URL is. Only a page (`P`) or a route handler (`R`) without a
  slot claims a URL: a pattern carrying nothing but a layout answers `null`, not a match with no body. Among the
  candidates the best score wins, so `/blog/[slug]` beats `/blog/[...slug]` for `/blog/x`.
- `RouteMatch(entry, params, rest, chain)` is the entry that claimed the URL, the bound path parameters as a
  `Dict<string, string>`, the catch-all remainder as segments and the layout chain that wraps it, root first.
  `paramOf(m, name)` is a parameter by name, `""` when the route binds none of that name.
- `layoutChain(table, pattern)` is the `L` entries without a slot wrapping a pattern, root first, built from
  `ancestorPatterns(pattern)` (`/`, `/a`, `/a/b`, …).
- The pieces `matchPath` is made of are public: `splitPath(p)` (the non-empty segments), `segmentWeight(kind)` (`Static`
  3, `Dynamic` 2, `CatchAll` 1, the rest 0), `scoreBeats(a, b)` (lexicographic, a longer score winning a shared prefix),
  `attempt(pattern, pathname) -> Attempt(ok, names, values, rest, score)` (one pattern against one path) and
  `paramsOf(names, values)` (the `Dict`).

The cells [route_handler_always_server-222](modules/decisions-taken/route_handler_always_server-222/draft/main.bp) and
[page_settings_are_page_arguments-290](modules/decisions-taken/page_settings_are_page_arguments-290/draft/main.bp) match a
URL against a table and read the route it is.

## route_kinds

The `k` blob: which routes are static (`S`) and which dynamic (`D`). The build's static-generation pass writes it, one
record per line, and the browser reads it with the same code to decide whether a link target is worth prefetching
(decision 202: a page that reaches no `#[serverOnly]` hook is prerendered).

```text
pattern|K        K is S (static) or D (dynamic)
```

- `RouteKind` is `Static` or `Dynamic`. `writeKinds(kinds)` writes a list of `#(pattern, kind)`; it halts, naming the
  pattern, when a pattern holds `|` or a newline. `parseKinds(wire)` reads the list.
- `routeKindOf(wire, pattern)` is the recorded kind of a pattern, `Dynamic` when the blob is absent, does not name the
  pattern or cannot be read — the answer that is never wrong, only slower.

Reading is tolerant: a line that is not `pattern|S` or `pattern|D` is skipped. Writing is strict.
[page_stage_from_server_hooks-202](modules/decisions-taken/page_stage_from_server_hooks-202/draft/main.bp) and
[route_handler_never_exported-273](modules/decisions-taken/route_handler_never_exported-273/draft/main.bp) write the blob;
the second shows that a route handler is never named in it.

## slot_states

The `z` blob: what each parallel slot rendered. The server writes it from its slot resolutions and the browser reads it to
keep or replace each slot.

```text
slot|pattern|state      state is M (matched), D (defaulted), U (unchanged) or E (empty)
```

`SlotState` is `Matched`, `Defaulted`, `Unchanged`, `Empty`. `writeSlotStates(states)` writes a list of
`#(slot, pattern, state)` and halts, naming it, when a slot or a pattern holds `|` or a newline. `parseSlotStates(wire)`
reads the list: a line without three fields is skipped and an unknown letter reads as `Empty`, so a malformed payload from
the network cannot crash the client router.

## url_rules

The boundary half of the URL rules. `PathRules(basePath, trailingSlash)` is the pair; `""` and `"/"` both mean no base.

- `canonicalize(rules, pathname)` turns a request path into the path the route table is written in: it strips
  `basePath`, applies `trailingSlash`, then percent-decodes once (`/a%252Fb` becomes `/a%2Fb`, never `/a/b`: a second
  decode is a path-traversal primitive). The query and the fragment, from the first `?` or `#`, pass through untouched.
  A path outside `basePath` is not this application's and is answered unchanged; a path whose escapes do not decode
  keeps its raw text.
- `clientHref(rules, pathname)` is its inverse: it percent-encodes each segment, applies `trailingSlash`, prefixes
  `basePath`, and carries the query and fragment through. The cell
  [shorthand_import_refused-337](modules/decisions-taken/shorthand_import_refused-337/draft/main.bp) builds links with it.
- The redirect table is one rule per line, the rule engine that evaluates it staying on the server.

```text
source|destination|permanent        permanent is 1 or 0
```

  `RedirectRule(source, destination, permanent)`; `writeRedirectTable(rules)` halts, naming the rule, when a source or a
  destination holds `|` or a newline; `parseRedirectTable(wire)` skips any line that is not exactly three fields with
  `1` or `0` last.

## navigation

The navigation-signal vocabulary. A signal is a raised reason, and a boundary re-raises it and never renders or logs
it; the prefix is neutral, no reason carries a framework's name.

```text
nav:not-found                     404
nav:redirect:<loc>                307
nav:permanent-redirect:<loc>      308
nav:see-other:<loc>               303
```

`NavKind` is `None`, `NotFound`, `Redirect`; `NavOutcome(kind, location, status)` is what a reason names.

- `signalPrefixes()` lists the four prefixes; `isSignalReason(reason)` is true for a reason that is one of them.
- `signalReason(outcome)` is the reason raised for an outcome. It halts, naming what was asked, for a `None` outcome
  (it is not a signal) and for a redirect status outside 303, 307 and 308.
- `signalFromReason(reason)` is the outcome a reason names. It is strict, because only code that spelled a reason by
  hand can get one wrong: a `nav:` reason with an unknown verb halts, naming the verb, and so does a reason that is not a
  `nav:` one.
- The `n` wire is what an action envelope carries. `signalToWire(outcome)` writes `""` for no signal, `N` for not-found
  and `R|<status>|<location>` for a redirect; `signalFromWire(wire)` reads it, and it is tolerant — it arrives from the
  network, so anything else is `None`, and the location is everything after the second `|`, so it may itself hold a `|`.
  The cell [route_read_through_hooks-293](modules/decisions-taken/route_read_through_hooks-293/draft/main.bp) answers a
  not-found for a parameter that is no integer, with the reason `signalReason` writes.

## pattern

The `:param` path grammar, a pattern of `/`-separated segments, each a literal, `:param` (exactly one path segment) or a
trailing `:param*` (the rest of the path, including nothing). Every other form is refused by name, never given a guessed
meaning.

- `PatternSegment` is `Literal(text)`, `Param(name)` or `Rest(name)`.
- `parsePattern(pattern) -> Result<PatternSegment[], string>` reads a pattern. A glob `*`, a character class `[`, a group
  `(`, `?`, `{`, a bare `:`, an empty parameter name, or a `:param*` before the last segment is an `Error` whose text is
  `patternProblem`'s (or says the `:param*` must be last).
- `patternProblem(pattern, segment)` is the refusal text for a segment the grammar does not have.
- `matchPattern(pattern, path) -> ?#(string, string)[]` is the captures in pattern order when the path matches and `null`
  otherwise: every literal equal and the segment counts agreeing, a `:param*` absorbing the surplus. A captured value is
  the raw path text; decoding is the caller's. The empty pattern matches only `/`; a caller whose "no pattern" means
  "every path" says so itself.

[optional_typescript_operators-330](modules/decisions-taken/optional_typescript_operators-330/draft/main.bp) parses a
pattern and reads the optional that matching it answers.

## conventions

Which file names in an app tree are routing files, their kinds and wire letters, and the two conflicts a tree can hold on
its own. Pure: `classify` takes the one path it is handed (decision 173) and never reads a disk; walking the tree is the
caller's, and `rakun-ssr` consumes this module and derives no chain of its own (decision 323).

- `fileKinds()` is the eight kinds in the order they wrap a page, outermost first (decision 171): `layout`, `template`,
  `error`, `loading`, `not-found`, `page`, then `default` and `route`. `kindOf(file)` is a file name's kind, `null` when
  the name is not a convention. `kindLetter(kind)` is the kind's table letter (`page` is `P`), `""` for anything else —
  the inverse of `table.kindLabel`, so no consumer keeps a private map (decision 172).
- `ConventionFile(authoredPath, segment, kind)` is one routing file: the path as handed in (what a diagnostic names), the
  app-relative directory with groups and slots kept (`(marketing)/about`, `""` for the root) and the kind.
- `classify(appDir, path) -> ?ConventionFile` is the routing file at `path` under `appDir`; `null` outside `appDir`
  (compared part by part), for a name no kind has, and for any path under a `_folder`, which opts out with everything
  below it.
- `conventionConflicts(files) -> string[]` is the refusals a tree's classified files hold on their own, in order: a
  segment holding both a `page` and a `route` (a segment is a page or an endpoint), then a URL that two root `(group)`s
  both claim with a page (two root layouts whose subtrees match one URL). A segment `pathProblem` refuses is skipped.

[bpp_page_routing_convention-203](modules/decisions-taken/bpp_page_routing_convention-203/draft/main.bp) classifies an app
tree under one convention (`app/blog/[slug]/page.bpp`), and
[route_handler_never_exported-273](modules/decisions-taken/route_handler_never_exported-273/draft/main.bp) shows a
handler and a page that differ in kind and letter. The file names the toolchain knows are the `.bpp` roles alone
([toolchain_knows_bpp_roles_only-285](modules/decisions-taken/toolchain_knows_bpp_roles_only-285/draft/main.bp)); a
decorator in a `.bpp` header is
[bpp_decorator_in_header-221](modules/decisions-taken/bpp_decorator_in_header-221/draft/main.bp); a route handler is
always server
([route_handler_always_server-222](modules/decisions-taken/route_handler_always_server-222/draft/main.bp)); a page's
settings are `#[page]` arguments
([page_settings_are_page_arguments-290](modules/decisions-taken/page_settings_are_page_arguments-290/draft/main.bp)).

## Departures

`src/` still writes some forms that `specs/retired.md` lists; the library's own codemod is pending, and this reference
describes the API in the spelling in force.

- Its types are spelt `Array<T>` in `conventions`, `pattern`, `route_kinds`, `url_rules` and `segment.fillPattern`; the spelling in force is `array<T>`, written `T[]` (decisions 641, 617 (1)).
- `src/root.bp` declares the modules with `pub mod` and the manifest lists `files`; every `.bp` under `src/` is a module
  whose path is its name, with no `mod` / `pub mod` (decision 592 (1)).
- `conventions.kindOf` recognises `<kind>.bp` alone; decision 203 makes a `.bpp` valid where a `.bp` is
  (`app/blog/[slug]/page.bpp` is a page), and the cell above shows that reading, which `src/` does not yet give.

## Decisions

| Section | Directory | What it holds |
|---|---|---|
| Decisions | [`modules/decisions-taken/`](modules/decisions-taken/) | The ledger's index: one `<slug>-<N>/` per code-bearing row of this repository's generated [`../decisions-taken.md`](../decisions-taken.md) (front 166, decisions 646–651), its `docs.md` walking through one example program, `draft/main.bp` until front 166 s4 promotes it to a cell (649) |
