# Decision 202 — No page declares or forces its stage: jhonstart's `#[page]` reads through 186's capability whether the page reaches a `#

The example writes the `k` blob, routing's record of which routes are static, from what `#[page]` read of each page: whether it reaches a `#[serverOnly]` hook by `use`. `/` and `/blog/[slug]` reach none, so they are `S`, prerendered at build; `/account` reaches one, so it is `D`, rendered per request. `Page` holds nothing else that could decide the stage: no `prerender` value, no output mode, no flag that forces a page that reaches no server hook to render per request. `routeKindOf` reads the blob the way the browser does when it decides whether a link is worth prefetching. A pattern the blob does not name, `/unknown`, reads as `Dynamic`, the answer that is never wrong, only slower; that is the reader's tolerance, not a setting.

Cells in this repository that prove it: none yet. `route_kinds` is covered by `test/route_kinds_test.bp`; the feature cells come with front 157's Conformance step (decision 465).

[draft/main.bp](draft/main.bp)
