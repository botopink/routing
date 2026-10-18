# Decision 206 — `import {…} from "<name>"` resolves only to a package — std, a bundled package or a declared dependency.

The example prints a route table through a banner. The package has a module of its own named `table` (`draft/table.bp`, which holds `banner`), and the `routing` library it depends on has a `table` module too, so one name means two things and each import line says which. `import {table.RouteEntry, table.writeTable} from "routing";` names a package, so it reaches the library's `table` module and never the module beside `main`, even though that module has the same name. `import {table.banner};` is the brace form with no `from`: the module's path, then the item, so it reaches this package's own `table.bp`. Every `.bp` file of the package is one of its modules, named by its path, so nothing declares it. `io.terminal` comes `from "std"`, a package like `routing`.

`main` writes a layout, a page and a `GET` handler with the library's `writeTable` and hands the wire to the package's `banner`, which prints `table L|/ ; P|/blog/[slug] ; R|/rss.xml||GET`. Had the example written `import {table.banner} from "routing";`, the import would look for `banner` in the library, which does not declare it. Writing `from` with a module of this package as the source (`import {banner} from "table";`) is refused with `module-import-with-from` at the source string, and the fix is the brace form the example uses.

Cells in this repository that prove it: none yet. The library's own modules already import each other in the brace form (`import {segment.parseSegment, …};` in `src/conventions.bp`); its feature cells come with front 157's Conformance step (decision 465).

[draft/main.bp](draft/main.bp)
[draft/table.bp](draft/table.bp)
