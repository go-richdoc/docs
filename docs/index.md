# go-richdoc

One rich-document model, and the converters that read and write it — Markdown, LaTeX and reStructuredText.

Part of the **Documents, typesetting & fonts** family of the
[pure-Go ecosystem](https://go-desktop.github.io/) — 4 modules,
all `CGO_ENABLED=0`.

## What is here

This site is the organisation's reference index: what each module is, and where its
API documentation lives. The API itself is generated from the source and served by
[pkg.go.dev](https://pkg.go.dev/), which is always current with the released tags —
duplicating it here would only create a second copy to go stale.

- **[Modules](modules.md)** — every module in go-richdoc, with its source and its reference.

## One thing to know before converting untrusted input

A converter turns one format into another, so the question is whether CONTENT CAN
BECOME MARKUP on the way out. It could, once: a Markdown heading of
`# <b>.. include:: /etc/passwd</b>` became a reST **directive**, because reST tries
explicit markup before it tries a title — and a docutils parse with a source path then
reads that file. Fixed in `rst` v0.3.0, with a probe over every position a document can
hold text to say it was the only one (`injectprobe`: 14 pairs where text became markup,
0 after; nothing at all in `markdown` or `latex`).

What no converter here protects you from, because the reference does not either: a URI
scheme. `javascript:alert(1)` in a link survives every hop, and docutils writes the
same `href`. An allow-list belongs in whatever renders the model. Each repository's
README has a **Security** section with its own audit.

## The model, in one paragraph

`richdoc` is a typed tree: a `Document` of `Block`s and `Inline`s, both CLOSED
interface sets, so a converter can type-switch over them exhaustively. Each
converter maps one format onto that tree in both directions. Where a format has
something the model does not, the converter says so in its own README with the
population it measured — that is how the model grows: `Classes` on the nodes that
carry them (v0.4.0), `Cell.Blocks` and `Table.Caption` (v0.5.0), each added because
a corpus showed what was being dropped, and each additive so no existing consumer
breaks.

## What every module here is held to

- `CGO_ENABLED=0`: no cgo, and no shelling out to a command-line tool in place of a
  library.
- Built and tested on amd64, arm64, riscv64, loong64, ppc64le and s390x — the last
  being big-endian, which keeps every on-disk and on-wire encoding honest.
- 100% statement coverage as a CI gate, error branches included — with one stated
  exception: `rst` enforces a **94% floor** instead. It walks a doctree whose exact
  shape it does not control (`go-docutils/docutils` is a separate, independently
  evolving engine), so its `ok`-checked type assertions and its `default:` case for
  an unknown doctree tag are a safety net for a FUTURE tag rather than dead code.
  The reason is in that repository's own workflow, beside the number.
- BSD-3-Clause.

The standard is described in full on the
[ecosystem map](https://go-desktop.github.io/docs/latest/standards/).
