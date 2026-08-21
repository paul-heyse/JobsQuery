# extruct 0.18.0 — advanced technical reference / feature and deployment catalog for LLM coding agents

## Version / source anchors

**Release anchor:** `extruct==0.18.0`, released 2024-11-08.  
**Python floor:** >=3.8 according to released metadata.  
**Project status:** mature but comparatively slow-moving; PyPI classifier remains Beta.

Sources:
- https://pypi.org/project/extruct/0.18.0/
- https://github.com/scrapinghub/extruct
- https://github.com/scrapinghub/extruct/blob/master/HISTORY.rst

0.18.0 adds Microdata action I/O support for `valueRequired` and `valueName`.

---

# 0. Purpose

extruct extracts structured metadata embedded in HTML.

Supported families include:

- JSON-LD;
- W3C Microdata;
- Open Graph;
- Microformats;
- RDFa;
- Dublin Core HTML metadata.

It should be used after page acquisition. It is not the preferred HTTP client or browser.

---

# 1. Installation

```bash
pip install extruct
```

CLI extra:

```bash
pip install 'extruct[cli]'
```

The CLI extra pulls acquisition dependencies for the command-line fetch workflow. In application code, separate acquisition with HTTPX2/Playwright from metadata extraction.

---

# 2. Primary API

Typical use:

```python
import extruct

data = extruct.extract(
    html,
    base_url=final_url,
)
```

Return value is a dictionary keyed by syntax, each containing extracted items in that syntax's representation.

---

# 3. Input forms

Modern extruct can accept:
- HTML text/bytes as supported by its parser path;
- an already parsed lxml tree in supported flows.

Reusing a parsed tree can avoid parsing the same HTML twice when the surrounding application already uses lxml.

If the surrounding parser is selectolax, extruct does not consume a selectolax tree directly; retain the HTML bytes/text or build an lxml tree for extruct.

---

# 4. `base_url`

Always provide the actual page/base URL when relative URLs may appear in metadata.

```python
data = extruct.extract(html, base_url=response.url)
```

Use the final URL after redirects unless your metadata contract intentionally uses the pre-redirect URL.

Without a correct base URL, relative IDs/images/links can resolve incorrectly or remain unresolved.

---

# 5. Syntax selection

Select only needed syntaxes to reduce work.

Typical names include:
- `json-ld`;
- `microdata`;
- `opengraph`;
- `microformat`;
- `rdfa`;
- Dublin Core support as exposed by current API/version.

For high-volume crawling, avoid parsing every syntax when the consumer only needs JSON-LD.

---

# 6. JSON-LD

extruct locates embedded JSON-LD script data and parses it into Python structures.

Historical robustness includes:
- handling null JSON-LD;
- permissive control-character behavior;
- tolerance for certain script/comment wrappers;
- handling JS-style comments in supported cases.

Do not treat JSON-LD as trusted merely because it parsed.

---

# 7. JSON-LD shape variability

Real sites may emit:
- one object;
- an array;
- `@graph`;
- multiple script blocks;
- duplicate types/entities;
- malformed blocks next to valid blocks.

Normalize downstream into your own entity model. extruct's job is extraction, not ontology reconciliation.

---

# 8. Microdata

Microdata extraction follows HTML Microdata concepts including:
- `itemscope`;
- `itemtype`;
- `itemprop`;
- `itemid`;
- `itemref`;
- property value extraction by element type.

0.18.0 adds support for action input/output-related fields `valueRequired` and `valueName`.

Use the library implementation instead of hand-walking DOM attributes: Microdata has subtle itemref/nesting/value rules.

---

# 9. Microdata `return_html_node`

extruct historically supports returning the HTML node with Microdata results when requested.

Use only when:
- the caller genuinely needs source-node provenance;
- the node lifetime/tree ownership is controlled.

Do not serialize parser nodes into application APIs accidentally.

---

# 10. Open Graph

Open Graph extraction handles:
- core `og:*`;
- known namespaces;
- expanded type-specific metadata;
- duplicate property behavior;
- optional array behavior.

Open Graph on the public web is inconsistent. Preserve duplicates/order when the application requires them rather than assuming one property = one scalar.

---

# 11. `with_og_array`

Recent extruct lines support Open Graph array-oriented extraction behavior via `with_og_array`.

Use when repeated OG properties (for example images) must be preserved as lists.

Test the exact shape your downstream normalization expects.

---

# 12. Microformats

Microformat extraction is provided through `mf2py` integration.

Treat Microformats output as a distinct vocabulary/shape; do not force it into schema.org semantics without an explicit mapping layer.

---

# 13. RDFa

RDFa support is built around RDF tooling and is described as experimental in project metadata/docs.

Operational implications:
- more dependencies;
- more complex graph semantics;
- greater CPU/memory than simple JSON-LD extraction;
- version interactions with rdflib.

Enable only if RDFa is a real data source for the product.

---

# 14. Dublin Core

extruct supports DC-HTML-2003-style Dublin Core metadata.

Use for publishing/library/document sites where DC meta tags are meaningful.

Do not assume Dublin Core presence on ordinary commercial sites.

---

# 15. `uniform` output

extruct offers a `uniform` mode to map several syntaxes toward a common template.

Use it for convenience only if its information model matches your needs.

For high-fidelity archival/extraction:
1. retain original syntax-specific output;
2. normalize into your own model separately;
3. retain source syntax/provenance.

---

# 16. Error policy

Historical `errors` modes include:
- strict;
- log;
- ignore.

Choose deliberately.

A crawler often wants:
- page acquisition succeeds;
- one malformed syntax should not discard other valid syntaxes;
- parse errors are observable/metricized.

A compliance/validation tool may prefer strict failure.

---

# 17. Individual extractor classes

extruct's internals expose extractor classes for the supported syntax families, each generally centered on `extract_items()` over an lxml document.

Use individual extractors when:
- only one syntax is needed;
- custom orchestration is required;
- one parsed lxml document should feed multiple controlled stages.

Prefer top-level `extract()` for ordinary all-in-one use.

---

# 18. Parsed-tree reuse

Since 0.15, extruct supports receiving a parsed tree.

High-value pattern:

```text
HTML bytes
 -> lxml parse once
     -> XPath/content extraction
     -> extruct structured metadata extraction
```

This can save a parse pass and ensures both extraction stages observe the same repaired tree.

---

# 19. selectolax coexistence

If selectolax is your primary parser:

```text
HTML bytes
  -> selectolax for fast DOM/content
  -> extruct separately from original HTML
```

Do not convert a selectolax tree to serialized HTML and then to lxml unless there is a reason; that adds normalization and cost. Prefer retaining original source bytes.

---

# 20. Playwright coexistence

For dynamic metadata:

```python
html = page.content()
data = extruct.extract(html, base_url=page.url)
```

Be aware:
- page JavaScript can inject/modify JSON-LD;
- `page.content()` is post-DOM serialization, not original network HTML;
- choose explicitly whether you want server-source metadata or rendered metadata.

---

# 21. URL resolution

Metadata frequently includes:
- `@id`;
- image URLs;
- canonical URLs;
- Open Graph URLs;
- relative resources.

Resolve against the right base. Preserve:
- raw value;
- resolved value;
- source page URL
if provenance matters.

---

# 22. Duplicate entities

Multiple blocks can describe the same entity.

Do not deduplicate by naive JSON equality only. Consider:
- `@id`;
- canonical URL;
- type;
- normalized identifiers;
- source syntax;
- block order.

Entity resolution belongs outside extruct.

---

# 23. Schema.org normalization

extruct extracts JSON-LD/Microdata but does not validate against current schema.org constraints.

If downstream code depends on a type/property contract:
- validate separately;
- tolerate extensions;
- version your normalization rules.

---

# 24. Trust boundary

Embedded metadata is attacker-controlled page content.

Threats include:
- huge JSON-LD blobs;
- deep nesting;
- weird Unicode;
- URLs targeting local/private resources;
- misleading type claims;
- HTML/JS strings in metadata;
- RDF expansion complexity.

Never execute values from metadata.

---

# 25. Resource limits

Enforce limits before/during acquisition:
- HTML bytes;
- render time;
- metadata block count;
- JSON-LD block size;
- extracted object count;
- depth/normalization work.

extruct itself should not be your only anti-exhaustion boundary.

---

# 26. CLI

The `extruct` CLI can fetch a page and emit metadata.

Useful for:
- manual inspection;
- quick experiments;
- fixtures.

For production systems prefer explicit acquisition and extraction layers so network policy/timeouts/proxies/browser behavior are controlled by the architecture.

---

# 27. Version posture

0.18.0 is current but relatively old compared with fast-moving browser/HTTP libraries.

Therefore:
- pin it;
- run a representative metadata corpus on dependency upgrades;
- watch rdflib/mf2py/lxml compatibility;
- do not assume support for future metadata dialect quirks until tested.

---

# 28. Testing corpus

Include:

```text
[ ] JSON-LD object
[ ] JSON-LD array
[ ] @graph
[ ] multiple JSON-LD scripts
[ ] malformed JSON-LD next to valid block
[ ] null JSON-LD
[ ] Microdata nesting
[ ] itemref
[ ] action valueRequired/valueName
[ ] duplicate Open Graph images
[ ] Open Graph namespaces
[ ] Microformats
[ ] RDFa
[ ] Dublin Core
[ ] relative URLs + base URL
[ ] HTML with <base>
[ ] duplicate entity across syntaxes
[ ] huge-but-bounded metadata block
[ ] rendered DOM metadata from Playwright
```

---

# 29. LLM-agent decision rules

Use extruct instead of writing:
- JSON-LD `<script>` regex;
- Microdata DOM walker;
- Open Graph namespace parser;
- RDFa parser;
- Microformats parser.

Choose extraction syntax explicitly when performance matters.

Keep normalization separate:
```text
extruct extraction
 -> raw syntax-specific records
 -> validation
 -> entity resolution
 -> canonical application model
```

---

# 30. Anti-pattern inventory

- regex for `<script type="application/ld+json">`;
- omitting `base_url`;
- treating all syntax outputs as identical;
- enabling RDFa with no consumer;
- discarding valid syntaxes because one malformed block fails;
- treating metadata as authoritative truth;
- fetching URLs referenced in metadata without SSRF policy;
- reparsing HTML unnecessarily when an lxml tree already exists;
- assuming rendered and source HTML metadata are the same;
- deduplicating entities without provenance.
