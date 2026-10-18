# Decision 306 — The type is the only schema, and `#[schema]` becomes `#[validated]`.

The example types a route's segments with a declared type. Routing matches `/produto/42` against `/produto/[id]` and hands the bound segments as text, `m.params.pairs`. `ProductParams` is the only schema: `#[validated]` on the type, `#[coerce]` and `#[minValue(1)]` on its field, and `ProductParams.bind(pairs)` — a member `#[validated]` gives the type — decodes the text into the record or answers the `ValidationReport`. `/produto/42` prints `product 42`; `/produto/abc` and `/produto/0` print `404` with the field and the violated rule; `/carrinho` matches no route. No schema value appears anywhere: there is no `Schema<T>`, no `schemas.*` combinator and no `checks.*` call, and routing takes no schema either — it answers strings, and the type says what they must be. A library that takes the type takes it as a `comptime` type parameter, refused unless `@typeInfo(T).meta(Validated)` is present.

The example uses `validation`, which the workspace manifest declares beside `routing`; the library itself imports std alone (decision 117 rule 8).

Cells in this repository that prove it: none. Routing never takes a type or a schema; the typed reading of a route is jhonstart's `use params<P>()` (293), checked by `#[page]`.

[draft/main.bp](draft/main.bp)
