# Modules

The 6 modules of **go-richdoc**. Each links to its source and to
its generated API reference.

| Module | What it is | Directions | Reference |
| --- | --- | --- | --- |
| [`richdoc`](https://github.com/go-richdoc/richdoc) | the model itself: the tree, and `Walk`, `Clone`, `PlainText` and a builder | — | [pkg.go.dev](https://pkg.go.dev/github.com/go-richdoc/richdoc) |
| [`markdown`](https://github.com/go-richdoc/markdown) | CommonMark + GFM tables and strikethrough, delegating the parse to `goldmark` | read, write | [pkg.go.dev](https://pkg.go.dev/github.com/go-richdoc/markdown) |
| [`latex`](https://github.com/go-richdoc/latex) | a practical subset of LaTeX, plus a PDF subpackage | read, write | [pkg.go.dev](https://pkg.go.dev/github.com/go-richdoc/latex) |
| [`rst`](https://github.com/go-richdoc/rst) | reStructuredText, delegating the parse to `go-docutils/docutils` | read, write | [pkg.go.dev](https://pkg.go.dev/github.com/go-richdoc/rst) |
| [`data`](https://github.com/go-richdoc/data) | CSV, JSON, XML and plain text, plus the `richdata` command | read | [pkg.go.dev](https://pkg.go.dev/github.com/go-richdoc/data) |
| [`fb2`](https://github.com/go-richdoc/fb2) | FictionBook 2 (`.fb2`): one XML file, images base64 inside it | read | [pkg.go.dev](https://pkg.go.dev/github.com/go-richdoc/fb2) |

**Which directions a module has is in the table**, because "converter" said of a
one-way reader is the kind of claim that costs somebody an afternoon. A module
that only reads is not unfinished: a `.fb2` or a `.csv` is a *source*, and
writing one back is a job nobody has asked for.

A one-line summary each, and nothing more: what a converter does with a construct
its format cannot express lives in that repository's README, beside the measurement
that decided it, so there is one place to correct when it changes.
