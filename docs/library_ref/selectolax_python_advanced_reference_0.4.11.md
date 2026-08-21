# selectolax 0.4.11 — advanced technical reference / feature and deployment catalog for LLM coding agents

## Version / source anchors

**Release anchor:** `selectolax==0.4.11`, released 2026-07-15.  
**Python:** >=3.9,<3.15.  
**Primary engine for new code:** Lexbor.  
**License:** selectolax MIT; bundled parser-engine licensing differs (Lexbor Apache-2.0, Modest LGPL-2.1).

Sources:
- https://pypi.org/project/selectolax/0.4.11/
- https://selectolax.readthedocs.io/en/latest/
- https://selectolax.readthedocs.io/en/latest/lexbor.html
- https://github.com/rushter/selectolax
- https://github.com/rushter/selectolax/blob/master/CHANGES.md

0.4.11 adds crash-prevention checks in CSS selector handling and updates Lexbor; 0.4.8+ added free-threading support and richer pretty serialization.

---

# 0. What selectolax is

selectolax is a Cython wrapper around high-performance HTML parsing engines with:

- HTML5 parsing;
- CSS selectors;
- fast text extraction;
- node navigation;
- DOM mutation;
- fragment parsing;
- serialization.

It is not:
- an XML schema system;
- an XPath engine;
- a JavaScript runtime;
- a browser.

Use it for offline HTML once bytes/HTML have already been acquired.

---

# 1. Backend choice: Lexbor vs Modest

Two historical backends exist:

- `selectolax.lexbor.LexborHTMLParser`;
- `selectolax.parser.HTMLParser` (Modest).

New applications should generally prefer Lexbor because it receives the most active feature/performance work.

Do not assume node classes from the two backends are interchangeable.

---

# 2. Installation

```bash
pip install selectolax
```

If building an older/non-wheel environment requires Cython:

```bash
pip install 'selectolax[cython]'
```

0.4.11 publishes broad binary wheel coverage including modern CPython and free-threaded variants.

---

# 3. LexborHTMLParser construction

```python
from selectolax.lexbor import LexborHTMLParser

tree = LexborHTMLParser(html_bytes_or_str)
```

Constructor supports fragment parsing options:

```python
LexborHTMLParser(
    html,
    is_fragment=False,
    fragment_tag="div",
    fragment_namespace="html",
)
```

Use explicit fragment mode for HTML fragments rather than pretending every fragment is a full document.

---

# 4. Root/head/body

Common properties:

```python
tree.root
tree.head
tree.body
tree.html
tree.inner_html
tree.raw_html
```

`html` returns serialization of the document/tree. `raw_html` exposes source bytes where supported by the parser object.

Do not assume parsed serialization is byte-identical to input HTML; HTML5 parsers repair/normalize malformed markup.

---

# 5. CSS selection

Core:

```python
tree.css("article.card")
tree.css_first("main", default=None)
tree.css_matches("body.home")
tree.any_css_matches(("article", "main"))
```

`css_first(..., strict=True)` can enforce that exactly one match exists; without strictness it can stop at the first match for speed.

Lexbor supports modern CSS selector forms and project-specific helpers such as `:lexbor-contains(...)`.

---

# 6. Selector strictness

Use strict matching when the extraction contract expects uniqueness:

```python
title = tree.css_first("h1", strict=True)
```

Use non-strict first-match when:
- any first match is acceptable;
- performance matters;
- the document is known to contain duplicates.

Do not silently use first-match semantics for identity-critical fields unless documented.

---

# 7. Advanced selector helper

`select()` returns a selector object supporting chained filtering / additional selector workflows beyond simple `.css()`.

Use it when the selector helper directly models the operation; avoid building custom loops over all nodes if the library can filter natively.

---

# 8. LexborNode lifetime

Nodes belong to a parser/document. Clones are still tied to parser-owned memory semantics.

**Agent rule:** never cache node objects beyond the lifetime of their parser.

If data must outlive the tree, extract Python primitives/strings/bytes.

---

# 9. Node navigation

Important relationships include:

- parent;
- first child / child alias;
- last child where exposed;
- next sibling;
- previous sibling;
- traversal/iteration APIs;
- parser reference.

Node walking is cheaper and clearer than re-querying CSS when the relative structure is already known.

---

# 10. Node identity/type

Lexbor exposes useful type indicators such as:
- element node;
- text node;
- comment node;
- document node.

Use these rather than inferring type from serialization.

---

# 11. Tag and ID properties

Common properties include:
- tag name;
- `id`;
- attributes.

Avoid relying on HTML attribute case/serialization peculiarities without tests on target documents.

---

# 12. Attributes: copy vs live mapping

`node.attributes` returns a normal dictionary-style snapshot of attributes.

`node.attrs` is a dict-like live interface that mutates the node.

```python
attrs = node.attributes       # safe value copy for inspection
node.attrs["data-x"] = "1"   # modifies DOM
del node.attrs["id"]
```

**Agent rule:** prefer `attributes` when read-only intent is sufficient. Use `attrs` when mutation is intentional.

Empty HTML attributes may have `None` values.

---

# 13. Text extraction

```python
node.text(
    deep=True,
    separator=" ",
    strip=True,
    skip_empty=True,
)
```

Key semantics:
- `deep=True` includes descendants;
- `separator` controls concatenation;
- `strip` trims pieces;
- `skip_empty` removes whitespace-only text nodes.

Text extraction is a policy decision. Different combinations can change token boundaries.

---

# 14. merge_text_nodes

After unwrapping markup, formerly separated text nodes may create artificial separators.

`merge_text_nodes()` merges adjacent text nodes and is useful before clean text extraction.

Typical pipeline:

```python
node.unwrap_tags(["strong", "em"])
node.merge_text_nodes()
text = node.text(deep=True, separator=" ", strip=True)
```

---

# 15. Raw text-node value

`raw_value` can expose original raw bytes for text nodes in Lexbor.

Use it when entity spelling matters. Parsed/serialized `.html` may normalize entities and escaping.

---

# 16. Comment content

`comment_content` extracts comment text for comment nodes.

Use node-type checks before assuming every node has meaningful comment content.

---

# 17. Script helpers

Parser/node helpers include:
- `scripts_contain(query)`;
- `script_srcs_contain(queries)`.

These cache script data on first use for repeated checks.

Useful for fingerprinting page technology, but do not use substring checks as a security verdict.

---

# 18. Tag lookup

`tags(name)` returns matching tag nodes efficiently.

Use direct tag lookup when selector expressiveness is unnecessary.

---

# 19. Fragment parsing

Lexbor supports HTML fragment parsing and recent releases improved:
- empty fragments;
- fragment tag;
- fragment namespace;
- multiple root-level fragment nodes.

Use fragment parsing for:
- snippets;
- HTML stored in API fields;
- template fragments;
- replacement/mutation payloads.

---

# 20. Node creation

The Lexbor parser can create element nodes.

```python
new_node = tree.create_node("span")
```

Use parser-owned node creation for structural mutation instead of concatenating raw HTML strings.

---

# 21. Insertion

Mutation methods include inserting:
- before;
- after;
- as child.

They can accept text/bytes or nodes depending on the API.

Important: string input is treated as text and escaped. Pass a Node when actual markup insertion is intended.

This is a valuable built-in safety property—do not defeat it with hand-built concatenation.

---

# 22. replace_with

Replace a node with text or another node.

When passing a node, understand ownership/movement semantics and whether later mutation of the passed node affects the tree.

Clone when independent structural identity is required.

---

# 23. Decompose / remove

`decompose()` removes a node, optionally recursively. `remove()` is an alias in Lexbor.

Double-removal bugs historically caused crashes in older releases; current versions include fixes, but application code should still maintain coherent mutation state.

---

# 24. strip_tags

`strip_tags(tags, recursive=False)` removes specified tags.

Clarify whether children should survive:
- remove wrapper but preserve descendants;
- or recursively remove content.

Do not use broad strip lists on data where script/style content or embedded metadata must still be inspected later.

---

# 25. unwrap / unwrap_tags

Unwrapping removes element wrappers while preserving descendants.

Use for text normalization and markup simplification.

After unwrapping, consider `merge_text_nodes()` before separator-based text extraction.

---

# 26. inner_html

`inner_html` can be read and set.

This maps conceptually to browser `innerHTML` but happens in the offline parser.

Setting HTML reparses/replaces children; do not treat it as a zero-copy textual splice.

---

# 27. Serialization

Core serialization surfaces:
- `.html`;
- `.inner_html`;
- `html_pretty()`;
- `inner_html_pretty()`.

Pretty options can control:
- indentation;
- whitespace-only nodes;
- comments;
- escaping/raw output;
- closing tags;
- namespace tags;
- text indentation;
- doctype detail;
- `html5test` mode.

Use regular serialization for application output; pretty modes are primarily diagnostics/fixtures unless exact formatting is part of your contract.

---

# 28. HTML5 repair semantics

HTML5 parsers:
- create implied html/head/body structure;
- repair malformed nesting;
- normalize certain empty attributes/tags;
- may reorder/imply nodes according to parsing algorithm.

Therefore:

```text
input HTML bytes != parsed tree serialization
```

If source fidelity matters, keep original bytes alongside the parsed tree.

---

# 29. Encoding

Prefer acquiring bytes plus HTTP charset metadata. selectolax can parse str or bytes, but encoding decisions made before passing `str` cannot be undone.

For crawlers:
1. retain response bytes;
2. decide encoding policy;
3. parse;
4. retain source metadata for provenance.

---

# 30. Modest backend

`selectolax.parser.HTMLParser` remains available.

Do not select it by habit from old examples. Verify whether the required method exists on both backends and whether behavior differs.

For a new design, standardize on one backend—usually Lexbor—to avoid dual semantics.

---

# 31. Free-threading and concurrency

Recent selectolax releases support free-threaded CPython builds.

Still treat parser/node instances as job-local unless documentation explicitly guarantees concurrent mutation safety.

Best parallel model:
- parse independent documents in independent tasks/threads;
- do not mutate one tree from multiple workers.

---

# 32. Performance

selectolax is designed for fast HTML parsing and CSS selection.

Performance rules:
- parse once;
- query the existing tree;
- use `css_first(..., strict=False)` when truly only first match matters;
- use tag/script caches where appropriate;
- extract primitives then release tree memory;
- avoid repeated parse/serialize/reparse cycles.

---

# 33. Memory and large documents

HTML can be attacker-controlled.

Bound:
- downloaded bytes before parsing;
- number/size of pages in flight;
- retained node trees;
- extracted text.

A fast parser can still become a resource-exhaustion vector if fed unlimited input.

---

# 34. When to use lxml instead

Use lxml when you need:
- XPath;
- XML namespaces;
- XML DTD/XSD/RelaxNG/Schematron validation;
- XSLT;
- C14N;
- streaming XML parsers;
- custom resolvers;
- rich XML document semantics.

Use selectolax when HTML speed and CSS-based extraction dominate.

---

# 35. When to use Playwright instead

Use Playwright when you need:
- JavaScript execution;
- hydrated DOM;
- browser storage/cookies;
- interaction;
- browser network/WebSocket behavior.

A common efficient pattern is:

```text
Playwright acquisition -> page.content() -> selectolax offline extraction
```

---

# 36. Error handling

`SelectolaxError` is used for parser/selector operations that fail.

Do not rely on segfault-like historical failure behavior as an exception contract. 0.4.x has repeatedly hardened NULL checks and memory handling; stay current.

Validate selectors in tests before applying them across large crawls.

---

# 37. 0.4.11-specific changes

0.4.11:
- adds NULL checks in CSS selector modules to prevent crashes;
- fixes HTML5-test serialization-mode checks;
- updates Lexbor.

Relevant recent 0.4.x changes also include:
- free-threading support;
- pretty HTML serialization;
- fragment namespace/tag controls;
- improved text handling;
- comment content;
- node creation;
- many memory/segfault fixes.

This recent churn is a reason to pin the parser version in reproducible extraction systems.

---

# 38. Testing strategy

Fixture categories:

```text
[ ] valid HTML5
[ ] severely malformed HTML
[ ] fragment / empty fragment
[ ] duplicate IDs/classes
[ ] Unicode selector
[ ] empty attributes
[ ] comments
[ ] entities / raw_value
[ ] script/style content
[ ] unwrapping + merge_text_nodes
[ ] decompose twice / mutation edge cases
[ ] inner_html set
[ ] multi-root fragment
[ ] very large HTML within configured bound
[ ] CSS selector syntax errors
[ ] serialization snapshot where needed
```

---

# 39. LLM-agent decision rules

Before writing Python loops, check whether selectolax already provides:

- CSS query;
- first-match query;
- selector matching;
- advanced selector helper;
- tag lookup;
- text extraction;
- script-content helper;
- unwrap/strip;
- text-node merge;
- insert/replace;
- fragment parser;
- pretty serialization.

Prefer library traversal/mutation primitives over string surgery on HTML.

---

# 40. Anti-pattern inventory

- using regex to parse HTML;
- reparsing the same document for every field;
- defaulting to Modest because an old snippet imports it;
- assuming serialization equals original bytes;
- keeping Nodes after parser lifetime;
- using live `.attrs` when read-only `.attributes` suffices;
- blindly stripping tags before metadata extraction;
- using `separator=" "` without considering adjacent text-node semantics;
- accepting unlimited HTML sizes;
- using selectolax where XML validation/XPath is actually required.
