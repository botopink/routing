# Decision 293 — A page reads its route through hooks, never a parameter.

The example is what the `params` hook answers a page under `/produto/[id]`. The page reads `use params<ProductParams>()` and takes no parameter; `params` here plays that hook for one request. `paramNamesOf("/produto/[id]")` names the pattern's segments, `id`, which are the record's fields `#[page]` checks. The matched segment is parsed to the field's type: `/produto/42` answers `ProductParams(id: 42)` and prints `product 42`. `abc` is no integer, so the request answers a not-found before the page renders, and `signalReason` prints `nav:not-found`. There is no `route: PageContext` parameter, no `StaticPath.data: Json` and no `pageData(route, schema)`.

Cells in this repository that prove it: none yet. `matchPath` and `paramNamesOf` are covered by `test/match_test.bp` and `test/segment_test.bp`; the feature cells come with front 157's Conformance step (decision 465).

[draft/main.bp](draft/main.bp)
