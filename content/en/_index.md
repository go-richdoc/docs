---
title: "Overview"
weight: 10
---

# go-richdoc {#go-richdoc}

One rich-document model, and the converters that read and write it — Markdown,
LaTeX, reStructuredText, HTML, EPUB, Kindle MOBI, FictionBook, email, CSV,
JSON, XML and plain text.

Part of the **Documents, typesetting & fonts** family of the
[pure-Go ecosystem](https://go-desktop.github.io/) — 10 modules,
all `CGO_ENABLED=0`.

## What is here {#what-is-here}

This site is the organisation's reference index: what each module is, and where its
API documentation lives. The API itself is generated from the source and served by
[pkg.go.dev](https://pkg.go.dev/), which is always current with the released tags —
duplicating it here would only create a second copy to go stale.

- **[Modules]({{< relref "modules.md" >}})** — every module in go-richdoc, with its source and its reference.

## One thing to know before converting untrusted input {#one-thing-to-know-before-converting-untrusted-input}

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

### The other question: how much work can a small document ask for? {#how-much-work-can-a-small-document-ask-for}

Every reader here takes a ceiling and refuses past it — a `Limits` everywhere except
`data`, whose four readers carry theirs on their `Options` — because the answer turned
out to be "a great deal", and each bound's default is picked from a measurement over
real documents rather than from taste:

| reader | what it bounds | the measurement behind the default |
|---|---|---|
| `markdown` | block nesting (v0.11.0) | 14 651 real `.md` files: the deepest has **5** `>` markers, the widest indent **73** columns |
| `latex` | environment nesting (v0.8.0) | 97 689 real `.tex` files: the deepest nests **64**, the next **26** |
| `epub` | entries, entry bytes, total bytes, depth, nodes | a ZIP entry declaring 8 638 bytes and opening to 8 MiB |
| `email` | parts, part bytes, total bytes, depth, nodes | a `message/rfc822` forwarded a hundred times |
| `fb2` | depth, and one binary's size | depth is checked in a token pass of its own, because the stack a deep file overflows is `encoding/xml`'s |
| `html` | depth, nodes, input bytes, source elements (v0.2.0) | 123 369 real `.html` files, **all** accepted: the biggest 19.87 MB, the widest **98 961** elements — and a refusal that cost 96 MB now costs **7 096 bytes** |
| `mobi` | records, text bytes, image bytes, total image bytes, depth, nodes | a PalmDOC record declaring more than it holds, and the ceiling handed on to the HTML reader underneath |
| `data` | input bytes, rows, columns, cells, depth, nodes, source values | 60× to 176× the source allocated per byte, by format — and the ceiling that stopped the answer and not the work |
| `rst` | line length | docutils' own `line_length_limit` |

Three shapes of defect came out of asking, and all three are worth knowing if you
build something similar:

- **a definition that references itself is an unrecoverable crash.** A substitution
  (`.. |a| replace:: |a|`, 21 bytes) and a footnote (`[^1]: the note[^1]`, 55 bytes)
  were each inlined at their reference, so each recursed forever and Go answered
  `fatal error: stack overflow` — which no `recover()` catches. Fixed in `rst` v0.13.0
  and `markdown` v0.11.0. The second was found **after** an audit for the first had
  swept the family and missed it, because the audit looked for the shape of the code it
  had already seen rather than for the behaviour.
- **nesting costs more than bytes.** Parsing is quadratic in depth in both Markdown and
  LaTeX, and the cheap shapes are asymmetric: a block quote costs two bytes a level, a
  one-line nested list four. 400 KB of block quotes occupied a core for 22 seconds
  before the bound. Each claim is a committed benchmark with an **equal-size flat
  control**, because without the control the numbers only say that a big document is
  slow.
- **a ceiling can stop the answer and not the work.** Two flat JSON arrays, the second
  eight times the source of the first, both refused by the same ceiling at the same
  node — so the reader should have stopped at the same byte in both. They cost
  **243 MB** and **1.275 GB**, a ratio of 5.2, because `encoding/json` decodes the
  whole value into an `any` *before* the converter counts its first node. A pass over
  the source bytes now refuses a hostile 16 MiB array in **40 bytes**. The same test on
  the XML reader, which counts inside its own token walk, gives **1.0** — and without
  that row the numbers would only say that a bigger file allocates more. Fixed in
  `data` v0.2.0, and the same control then found it in `html`: `x/net/html` builds the
  whole tree before our first node is counted, and has no ceiling of its own, so a
  refusal cost **96 MB** where it now costs **7 096 bytes** (v0.2.0). The sibling that
  a fix suggests is worth checking **by running its control**, not by reading its code.
- **and a measurement's proxy can be wrong about its own subject.** The HTML corpus
  says the deepest of 123 369 real files nests **3 140** levels against a `MaxDepth`
  of 256 — so 0.1% of real documents ought to be refused, and the number actually
  refused is **zero**: unclosed tags make a *tokenizer's* nesting climb where the
  HTML5 algorithm closes them. A default "corrected" from that percentile would have
  loosened a ceiling that was never tight. The probe now has a mode that asks the
  reader for its verdict on every file instead, which is the only column a default
  should be chosen from.
- **and a guard can refuse what the thing it protects would accept**, which is worse
  than no guard. The fuzz target checking the new pass against the converter broke it
  in **700 milliseconds** with `0 0 0[0[0[0`: nine values to a scan of the file, *one*
  value to `Decoder.Decode`, which reads one and leaves the stream alone. The same
  minimised input showed `Parse` had been returning **half a file in silence** — a
  second JSON value dropped with no error — so it now says so. A guard has to stop
  where the thing it guards stops, and the way to find out that it does not is to
  hunt for a counterexample rather than argue the derivation.

Every reader in the family is fuzzed (`go test -fuzz`), with the failing inputs
committed: **thirteen** targets over nine repositories. Each asserts a property of the
**output** rather than the absence of a panic — a document XOR an error, nothing built
deeper than the reader's own ceiling allows, and the same bytes twice giving the same
document.

## The model, in one paragraph {#the-model-in-one-paragraph}

`richdoc` is a typed tree: a `Document` of `Block`s and `Inline`s, both CLOSED
interface sets, so a converter can type-switch over them exhaustively. Each
converter maps one format onto that tree in both directions. Where a format has
something the model does not, the converter says so in its own README with the
population it measured — that is how the model grows: `Classes` on the nodes that
carry them (v0.4.0), `Cell.Blocks` and `Table.Caption` (v0.5.0), each added because
a corpus showed what was being dropped, and each additive so no existing consumer
breaks.

## What every module here is held to {#what-every-module-here-is-held-to}

- `CGO_ENABLED=0`: no cgo, and no shelling out to a command-line tool in place of a
  library.
- **Built** for ten targets — `linux/{amd64,arm64,riscv64,loong64,ppc64le,s390x}`,
  `darwin/{amd64,arm64}`, `windows/amd64` and `js/wasm` — and **tested** on
  `amd64`, with the race detector.

{{< callout type="info" >}}
**⛔ Built is not tested, and this page used to say both at once**

It read "built and tested on amd64, arm64, riscv64, loong64, ppc64le and
s390x". Cross-compiling proves a program compiles; it says nothing about
what it does when a byte order or a word size differs. These modules are
a tree and a great deal of string handling, which is a different risk
from a format whose integers come off a disk — and where that *is* the
risk, the lane exists: [`go-ooxml`](https://go-ooxml.github.io/) runs its
suite under qemu on five more architectures, big-endian s390x included,
because an OPC package is a ZIP full of little-endian integers.
{{< /callout >}}
- 100% statement coverage as a CI gate, error branches included — with one stated
  exception: `rst` enforces a **94% floor** instead. It walks a doctree whose exact
  shape it does not control (`go-docutils/docutils` is a separate, independently
  evolving engine), so its `ok`-checked type assertions and its `default:` case for
  an unknown doctree tag are a safety net for a FUTURE tag rather than dead code.
  The reason is in that repository's own workflow, beside the number.
- BSD-3-Clause.

The standard is described in full on the
[ecosystem map](https://go-desktop.github.io/docs/latest/standards/).
