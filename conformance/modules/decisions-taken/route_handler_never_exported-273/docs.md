# Decision 273 — 222 holds whole: a route handler is never exported by the build.

The example puts two feeds side by side. `app/rss.xml/route.bp` is a route handler: `classify` answers kind `route`, letter `R`, pattern `/rss.xml`. `app/feed.xml/page.bp` is a page-kind file: kind `page`, letter `P`, pattern `/feed.xml`. The build's static pass prerenders the page and writes `/feed.xml|S` into the `k` blob; the handler is not a page and is never named there. `exported` answers what the build writes as a file, so it prints `false` for `/rss.xml`, which is served per request with no file written for it, and `true` for `/feed.xml`. A feed that must be a static file is written as the page-kind file.

Cells in this repository that prove it: none yet. They come with front 157's Conformance step (decision 465).

[draft/main.bp](draft/main.bp)
