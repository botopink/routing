# Decision 403 — `@Decl` carries no source location: a decorator's output never depends on where its file sits.

The example computes the route a page receives from where its file sits in the app tree, as the file-convention route table does at build. `classify("app", "app/(marketing)/blog/[slug]/page.bpp")` answers the file's segment, `(marketing)/blog/[slug]`, and `patternOf(parsePath(…))` drops the group and keeps the bracket spelling: `/blog/[slug]`. The route reaches `#[page]` as an argument, so the first line printed is the generated call `page("/blog/[slug]", BlogPost)`; the page itself writes no route, and nothing reads a location off the declaration. `paramNamesOf(route)` names the segment the page reads by hook (293), `slug`. The route table entry, `P|/blog/[slug]`, matches `/blog/hello` with `slug` bound to `hello`. The same page moved under another group, `(site)`, answers the same route, so the last line prints `true`: the decorator's output depends on its arguments, never on where the file sits.

Departure: `conventions.classify` answers only `.bp` file names today, so `page.bpp` is not a routing file and the example prints an empty route; `.bpp` as an app file is front 157 s1, step 1.

Cells in this repository that prove it: none yet. They come with front 157's Conformance step (decision 465).

[draft/main.bp](draft/main.bp)
