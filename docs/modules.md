# Modules

The 4 modules of **go-richdoc**. Each links to its source and to
its generated API reference.

| Module | What it is | Reference |
| --- | --- | --- |
| [`richdoc`](https://github.com/go-richdoc/richdoc) | the model itself: the tree, and `Walk`, `Clone`, `PlainText` and a builder | [pkg.go.dev](https://pkg.go.dev/github.com/go-richdoc/richdoc) |
| [`markdown`](https://github.com/go-richdoc/markdown) | CommonMark + GFM tables and strikethrough, both directions | [pkg.go.dev](https://pkg.go.dev/github.com/go-richdoc/markdown) |
| [`latex`](https://github.com/go-richdoc/latex) | LaTeX, both directions, plus a PDF subpackage | [pkg.go.dev](https://pkg.go.dev/github.com/go-richdoc/latex) |
| [`rst`](https://github.com/go-richdoc/rst) | reStructuredText, both directions, delegating the parse to `go-docutils/docutils` | [pkg.go.dev](https://pkg.go.dev/github.com/go-richdoc/rst) |

A one-line summary each, and nothing more: what a converter does with a construct
its format cannot express lives in that repository's README, beside the measurement
that decided it, so there is one place to correct when it changes.
