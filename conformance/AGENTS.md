# routing — conformance/

- `modules/botopink.json` — the workspace manifest (front 166's): `"*"` members, the library's targets, `routing` by path
  and `validation` (the one decision example that types a route's segments, 306) as dependencies.
- `modules/decisions-taken/<slug>-<N>/` — front 166's decision cells (decision 651: the subtree is that front's alone): one
  per code-bearing row of `../decisions-taken.md`, a `docs.md` (first line generated) and, in phase 1, `draft/main.bp` (decision
  649; no `src/`, no manifest, so `botopink cell check` finds no cell there); kept by `scripts/decision-cells.py` of the meta
  repository and its CI check 7.
- `docs.md` — the reference (decisions 457, 461): the package and how a program imports it; one section per module of `src/` — `segment` (the folder-name grammar, `fillPattern`), `table` (the route table's wire), `match` (`matchPath`, `layoutChain`), `route_kinds` (the `k` blob), `slot_states` (the `z` blob), `url_rules` (`canonicalize`, `clientHref`, the redirect table), `navigation` (the `nav:` reasons, the `n` wire), `pattern` (the `:param` grammar) and `conventions` (the app-file kinds, `classify`, `conventionConflicts`); the departures `src/` still writes; and the row naming `modules/decisions-taken/`. No fenced `botopink` code: every example is a decision cell it links; a table, a grammar or a wire format is fenced as `text`.
