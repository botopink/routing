# Decision 282 — No role is declared by an export's name.

The example asks routing what six files of an app tree are, from their names alone. `classify("app", path)` answers a convention file for `page.bp` (kind `page`, letter `P`) and `layout.bp` (kind `layout`, letter `L`): route file names are the routing convention (203), and that is the one place a name gives a role. The four other files are named after what a framework once looked up by an export's name (`staticPaths.bp`, `generateMetadata.bp`, an OG image's `size.bp`, `collections.bp`), and each is `not a routing file`. Routing has no kind for them, and the route table it writes has no field that names a function, so nothing downstream can find the pages to prerender, a page's head or an image's size by a name. Those roles are said in a decorator, an argument of jhonstart's `#[page("blog/[slug]", paths: allPosts, head: postHead, partial: true)]` or a decorator of its own such as `#[ogImage(…)]`; a function left in a module under one of the old names is an error naming the decorator form, which is the framework's to report.

Cells in this repository that prove it: none yet. `conventions.classify` and `kindOf` are covered by `test/conventions_test.bp`; the feature cells come with front 157's Conformance step (decision 465).

[draft/main.bp](draft/main.bp)
