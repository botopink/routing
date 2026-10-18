# Decision 203 — One routing convention: `app/blog/[slug]/page.bpp` — a `.bpp` is valid where a `page.bp` is, beside `layout`, `loading`

The example classifies an app tree under one convention. `app/blog/[slug]/page.bpp` is a page of `/blog/[slug]`, exactly where a `page.bp` would be, and `layout.bpp`, `loading.bp` and `error.bp` beside it are that segment's layout, loading and error files; `app/about/page.bp` is the page of `/about`. A file named after its route, `app/blog.bp`, is not a routing file, and neither is anything under a `pages/` tree, `pages/blog/[slug].bpp`: there is no second convention where a file is a route. The last line counts the conflicts the classified files hold, `0`.

Departure: `conventions.classify` answers only `.bp` file names today, so `page.bpp` and `layout.bpp` print `not a routing file`; `.bpp` (with `.md` and `.html`) as an app file is front 157 s1, step 1.

Cells in this repository that prove it: none yet. `conventions.classify` is covered for `.bp` files by `test/conventions_test.bp`; the feature cells come with front 157's Conformance step (decision 465).

[draft/main.bp](draft/main.bp)
