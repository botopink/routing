# Decision 285 — The toolchain knows these things about `.bpp`:

The example builds the route table the framework generates at build, from routing's file kinds. The first line is that rule, in wrap order with each kind's letter: `layout:L template:T error:E loading:S not-found:N page:P default:D route:R`. It lives in `conventions.fileKinds`, a function of the library, not in the toolchain. Each classified file becomes a call to the comptime function of its kind, on the file's unfolded default function, with the route as an argument: `layout("/", app/layout.bpp)`, `page("/blog/[slug]", app/blog/[slug]/page.bpp)`, `not-found(…)`, `route("/api", app/api/route.bp)`. The toolchain applies no decorator by file name; what a file is for is the framework's rule, and routing answers the kinds and the conflicts a tree holds on its own. `app/api/` holds both a `page.bpp` and a `route.bp`, so `conventionConflicts` prints that a segment is a page or an endpoint, never both.

Departure: `conventions.classify` answers only `.bp` file names today, so the three `.bpp` files are skipped, only `route("/api", app/api/route.bp)` is printed and no conflict is found; `.bpp` as an app file is front 157 s1, step 1.

Cells in this repository that prove it: none yet. `fileKinds`, `kindLetter` and `conventionConflicts` are covered by `test/conventions_test.bp`; the feature cells come with front 157's Conformance step (decision 465).

[draft/main.bp](draft/main.bp)
