# Decision 290 — No segment configuration; a page's settings are `#[page]` arguments.

The example writes the routing data a blog's tree turns into and reads it back. The route table is `L|/`, `L|/blog`, `P|/blog/[slug]`: what each file is and where it sits, with no field a segment configures. `matchPath` on `/blog/hello` answers the page and its layout chain, `L / > L /blog`, and a layout carries nothing a page under it inherits. The stage is the `k` blob's `S`, which `#[page]` wrote from the page's hooks; the program reads `prerendered`, and nothing in routing forces it. What remains of a page's settings, `revalidate: hours(1)` and `dynamicParams: true`, is written as `#[page]` arguments beside `paths:`, `head:` and `partial:`, so routing never sees them: there is no `SegmentConfig`, no `dynamic` and no `fetchCache` to carry.

Cells in this repository that prove it: none yet. `layoutChain` and the table wire are covered by `test/match_test.bp` and `test/table_test.bp`; the feature cells come with front 157's Conformance step (decision 465).

[draft/main.bp](draft/main.bp)
