# Decision 337 — The shorthand import goes; `mod m;` binds the namespace `m` in the module that declares it.

The example builds a docs link and reads it back. `draft/links.bp` is a module of the package: it declares `href` as `pub` and builds the link with the library's `clientHref` under the base path `/docs`. `main` imports that function by the module's path, `import {links.href};`, the module first and then the item, with no `from`. Every other import names its source: `url_rules.PathRules` and `url_rules.canonicalize` come `from "routing"`, and `io.terminal` `from "std"`. The program prints `/docs/guides/getting%20started`, then `canonicalize` strips the base path and decodes it back to `/guides/getting started`.

The refused spelling is the shorthand. `import {href};` has no `from`, and its first segment names no module of the package, so it is refused with `shorthand-import` at the item. Only `links` declares `href` as `pub`, so the error gives the fix: `write import {links.href};`, the line the example uses. A bare `import {clientHref};` does not reach the library either, even though only `routing` declares that name; it has to be written `from "routing"`.

The decision's parts on `mod` / `pub mod` namespaces and re-export by value are overtaken by decision 592 (ctr-x): a file is a module, and there is no `mod`. The library's own `src/root.bp` still declares `pub mod` per module, which 592's codemod over the library removes.

Cells in this repository that prove it: none yet. The library's modules already import each other by path (`import {segment.parseSegment, …};` in `src/conventions.bp`); its feature cells come with front 157's Conformance step (decision 465).

[draft/main.bp](draft/main.bp)
[draft/links.bp](draft/links.bp)
