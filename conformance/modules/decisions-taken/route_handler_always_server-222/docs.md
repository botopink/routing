# Decision 222 — A route handler is always server: a `#[route]` handler (`route.bp`, answering a `Response`, every method, `GET` included

The example serves a page and a handler from one table. `/posts` is a page (`P`) that the build's static pass prerendered, so the `k` blob names it `S` and it prints `prerendered`. `/api/posts` is claimed by two handler entries (`R`), one for `GET` and one for `POST`; `matchPath` answers the handler for the URL, and the blob never names its pattern, because the static pass prerenders pages alone. It reads as `Dynamic` and prints `per request`. That holds for `GET` as for `POST` (the last line lists both verbs): a handler runs per request as if it were `#[serverOnly]`, it is never prerendered nor exported, and nothing it reaches is evaluated at the build.

Cells in this repository that prove it: none yet. `matchPath` over `R` entries is covered by `test/match_test.bp`; the feature cells come with front 157's Conformance step (decision 465).

[draft/main.bp](draft/main.bp)
