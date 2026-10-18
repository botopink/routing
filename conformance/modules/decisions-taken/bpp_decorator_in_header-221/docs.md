# Decision 221 — A `.bpp` file's decorator is written in the header, on a line just before the separator, annotating the unfolded functio

The example turns four app files into route-table entries. `classify` gives each file its kind, `layout`, `page`, `loading` or `page`, and `entryOf` writes the kind's letter and the segment's pattern: `L|/`, `P|/blog/[slug]`, `S|/blog/[slug]` and `P|/about`. That is everything routing reads from a file's name. `ConventionFile` and `RouteEntry` have no field for a decorator, because none comes from the name: a `.bpp` page's `#[page("blog/[slug]", revalidate: hours(1))]` is written in its header, on the line just before the separator, as ordinary header code annotating the unfolded function, and a `page.bp` writes the same decorator over its function.

Departure: `conventions.classify` answers only `.bp` file names today, so the three `.bpp` files are skipped and only `app/about/page.bp` reaches the table; `.bpp` as an app file is front 157 s1, step 1.

Cells in this repository that prove it: none yet. They come with front 157's Conformance step (decision 465).

[draft/main.bp](draft/main.bp)
