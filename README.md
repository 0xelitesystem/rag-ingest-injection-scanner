# RAG Ingest Injection Scanner

Drop an HTML page or a .docx you are about to index and see every character a text extractor hands the model that a reader never sees: hidden runs, clipped text, off-canvas payloads.

## Live demo

https://0xelitesystem.github.io/rag-ingest-injection-scanner/

## Features

- **Two texts, one delta.** The document is rendered in a visible, sandboxed, network-blocked frame. The tool computes what a reader actually sees and what a text extractor with no CSS engine hands the model, and reports the gap character by character.
- **Only decidable findings.** `display:none`, `visibility:hidden`, zero-area client rects, `clip-path: inset(100%)`, `text-indent: -9999px`, `opacity: 0`, off-canvas absolute positioning, the legacy screen-reader-only `clip` rect, `template` content, `noscript` subtrees, `style`/`script` source and document-head text. Every rule is decided from a computed style value, a geometry measurement or a tag name. Characters are attributed to one rule only, and the structural rules win over `display:none`, because a stylesheet computing to `display:none` is not why a reader cannot see it.
- **Two labelled extractor profiles.** The default drops `script`, `style` and `template` text, approximating BeautifulSoup 4.9+ `get_text()`. The raw profile keeps every text node, approximating lxml.html `text_content()`, which is XPath `string()`. Switching profiles shows exactly how much of a naive "just take textContent" delta is padding made of CSS and JavaScript source.
- **Colour against background is a suspect, never a finding.** It never moves the numbers, and it refuses to return a verdict at all for any node whose ancestor chain contains a background image, a filter, a backdrop-filter, a blend mode or an opacity below 1, because in those cases the painted colour is not knowable from `getComputedStyle` alone.
- **DOCX with no library.** The package is unzipped in the tab with `DecompressionStream("deflate-raw")` and the WordprocessingML is scanned directly for `w:vanish` hidden runs, `w:del` tracked deletions and comment bodies in `word/comments.xml`.
- **Other channels, kept separate.** HTML comments, `alt`, `title`, `aria-label`, `data-*` and `aria-hidden` subtrees are inventoried but deliberately excluded from the delta, because neither shipped profile emits them. The page says which readers do.
- **Invisible characters** inside the text a reader believes they read: zero-width spaces, joiners, bidi marks and Unicode tag characters.
- Dark by default with a persisted light theme, keyboard accessible, responsive to 360px, and a built-in sample for both the HTML and DOCX paths.

## How it works

1. Your document is loaded into an `iframe` with `sandbox="allow-same-origin"` and **without** `allow-scripts`, so nothing in it executes while the parent can still read its DOM. A `Content-Security-Policy` meta of `default-src 'none'` is prepended inside the frame, so no image, font, stylesheet or script is ever fetched.
2. The frame is **rendered, never hidden**. MDN's `HTMLElement.innerText` page states that when an element is not being rendered the returned value is the same as `Node.textContent`. Measure inside a hidden frame and the visible text collapses onto the extractor text, the delta goes to zero, and every document is reported clean.
3. Rendering alone is not enough either. In a rendered frame, `innerText` omits `display:none` text but still returns text hidden by `clip-path` or `opacity: 0`. So every text node is classified individually from its ancestors' computed styles and its own client rects.
4. The classified text nodes are fed to pure functions that return a report object: visible characters, gap characters, per-rule totals, the gap text itself, and the suspects. Visible plus gap equals the extractor total exactly, because all three are counted with the same per-node whitespace normalization.
5. For a `.docx`, there is no layout to measure. The zip central directory is parsed per the PKWARE `APPNOTE.TXT` field layout, deflated entries are inflated with `DecompressionStream`, and `word/document.xml`, any header and footer parts and `word/comments.xml` are scanned for the hidden channels.

Two things the page says out loud, first, before any result:

- **A zero delta is not a safe document.** Injections in plain visible text work just as well and this tool cannot see them.
- **Subresource blocking is a caveat on the measurement itself.** Blocking the network is the right call for an untrusted document, and it also means layout that depends on a background image or a remote font is not the layout your reader gets. Every check that a blocked subresource could invert refuses to return a verdict instead of guessing.

Library behaviour (which reader emits which channel) is kept in one dated block, visually separated from the tool's own computation, and every row is an inference from reading that library's source on the stamped date rather than a run.

## Privacy

Everything happens in your browser. Your document is never uploaded, there is no backend, no analytics, no API key and no external dependencies. The page makes zero network requests after it loads, and the frame it renders your document in is forbidden from making any at all.

## License

MIT. See [LICENSE](LICENSE).

## More

- Companion, the render side of the same problem: [llm-output-sink-scanner](https://0xelitesystem.github.io/llm-output-sink-scanner/)
- What your splitter does to the text once it is extracted: [rag-chunk-visualizer](https://0xelitesystem.github.io/rag-chunk-visualizer/)
- Mitigations: [prompt-injection-defense-reference](https://github.com/0xelitesystem/prompt-injection-defense-reference)
- All tools: https://0xelitesystem.github.io/
- https://elitesystem.ai
