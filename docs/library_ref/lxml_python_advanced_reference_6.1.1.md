# lxml 6.1.1 — advanced technical reference / feature and deployment catalog for LLM coding agents

## Version / source anchors

**Stable anchor:** `lxml==6.1.1`, released 2026-05-18.  
**Prerelease note:** 7.0 alpha releases exist; this reference intentionally stays on 6.1.1 stable.  
**Implementation:** Pythonic binding over libxml2 and libxslt.

Sources:
- https://pypi.org/project/lxml/
- https://lxml.de/6.1/
- https://lxml.de/api.html
- https://lxml.de/apidoc/lxml.etree.html
- https://lxml.de/validation.html
- https://lxml.de/xpathxslt.html

6.1.1 includes security-relevant fixes to known HTML link attributes and ships patched libxslt in Linux wheels for documented CVEs.

---

# 0. Capability map

lxml provides:

- XML and HTML parsing;
- ElementTree-compatible tree API plus parent/sibling extensions;
- namespaces;
- XPath 1.0;
- XSLT 1.0;
- DTD;
- Relax NG;
- XML Schema;
- ISO Schematron;
- canonical XML serialization;
- XInclude;
- SAX bridging;
- incremental / pull parsing;
- iterparse;
- custom parser targets;
- custom URI resolvers;
- extension XPath/XSLT functions;
- custom element classes;
- objectify data binding;
- HTML forms/links/helpers;
- builder APIs;
- optional CSS selector integration.

It is the “rich markup” library in this reference pack.

---

# 1. Installation and binary dependencies

```bash
pip install lxml==6.1.1
```

Wheels bundle appropriate native dependencies for common platforms. Source builds depend on system/libxml2/libxslt/Cython build details.

For deterministic production behavior, prefer official wheels or a deliberately controlled native build.

Inspect runtime versions:

```python
from lxml import etree
etree.LXML_VERSION
etree.LIBXML_VERSION
etree.LIBXSLT_VERSION
```

Behavior can depend on the native library version, not just the Python package version.

---

# 2. Core modules

Important modules include:

- `lxml.etree`;
- `lxml.html`;
- `lxml.objectify`;
- `lxml.builder`;
- `lxml.sax`;
- `lxml.isoschematron`;
- `lxml.cssselect` when CSSSelect dependency support is installed;
- `lxml.doctestcompare`.

Do not import legacy `lxml.html.clean` expecting a bundled sanitizer in modern environments; HTML cleaning has been split into the separate `lxml_html_clean` project.

---

# 3. XML tree construction

```python
from lxml import etree

root = etree.Element("root")
child = etree.SubElement(root, "item", id="1")
child.text = "hello"
tree = etree.ElementTree(root)
```

Elements know parents/siblings unlike stdlib ElementTree.

Useful relationships:
- `getparent()`;
- `getnext()`;
- `getprevious()`;
- `getroottree()`.

---

# 4. Element vs ElementTree semantics

An Element participates in its owning document. An `ElementTree(element)` can define an explicit operation root without physically detaching the element from its original document.

Absolute XPath and document-level serialization semantics can differ depending on whether operations are invoked on an Element or an explicit ElementTree.

For complex transformations, be explicit about the document root.

---

# 5. XML parsing

```python
parser = etree.XMLParser()
root = etree.fromstring(xml_bytes, parser)
tree = etree.parse(file_or_stream, parser)
```

Choose parser options explicitly for:
- entity resolution;
- DTD loading;
- network access;
- recovery;
- huge trees;
- whitespace;
- comments/PIs;
- namespace cleanup;
- schema validation.

---

# 6. Security: XML parser defaults

For untrusted XML, the important threat classes include:
- external entities / XXE;
- DTD retrieval;
- network access;
- entity expansion;
- huge/deep trees;
- decompression bombs when feeding compressed data elsewhere;
- XInclude/resource loading.

Modern lxml defaults are safer than historical examples, but secure design still requires explicit parser policy.

6.1 changed the default entity resolution behavior to `resolve_entities="internal"` for relevant parsers, rather than the older broad `True` behavior.

Recommended starting posture for untrusted XML:

```python
parser = etree.XMLParser(
    resolve_entities=False,
    load_dtd=False,
    no_network=True,
    huge_tree=False,
)
```

Then enable capabilities only when required.

---

# 7. HTML parsing

```python
from lxml import html

doc = html.fromstring(html_text)
```

Or use `etree.HTMLParser`.

HTML parsing repairs malformed markup. As with any HTML5-ish parser layer, parsed serialization need not equal source bytes.

For fast CSS-only HTML extraction, selectolax may be preferable. Use lxml HTML when its XPath/tree/HTML helpers add value.

---

# 8. Parser recovery

`recover=True` can parse malformed XML that strict mode would reject.

Do not enable recovery in schema/contract validation pipelines if malformed input must be rejected. Recovery trades correctness guarantees for resilience.

---

# 9. Parser targets

lxml supports target objects that receive parse events and can build custom results instead of a full Element tree.

Use parser targets when:
- extracting a compact representation;
- streaming custom processing;
- avoiding a retained DOM.

Prefer built-in pull/iterparse if they already express the use case.

---

# 10. Feed parser

Parser objects can accept incremental `.feed()` input and finalize with `.close()`.

Use when data arrives incrementally and you control chunking.

Be careful to:
- close parser on EOF;
- propagate errors;
- bound total input;
- avoid mixing chunks from separate documents.

---

# 11. iterparse

`etree.iterparse()` is a primary large-XML tool.

Process `start` / `end` events and clear completed elements to bound memory.

Canonical memory discipline:

```python
for event, elem in etree.iterparse(path, events=("end",), tag="record"):
    process(elem)
    elem.clear()
    while elem.getprevious() is not None:
        del elem.getparent()[0]
```

Tune the exact cleanup pattern to tree semantics.

---

# 12. XMLPullParser

Pull parsing provides event-driven incremental parsing under application control.

Useful for:
- network streams;
- non-blocking input;
- bounded event processing.

---

# 13. Iteration and axes

Common iteration methods:
- `.iter()`;
- `.iterchildren()`;
- `.iterdescendants()`;
- `.iterancestors()`;
- `.itersiblings()`.

When relationship is simple, these avoid compiling XPath.

---

# 14. Namespaces

Clark notation:

```python
"{urn:example}tag"
```

XPath namespace prefixes are passed separately:

```python
ns = {"x": "urn:example"}
root.xpath("//x:item", namespaces=ns)
```

The source document's prefix is not the semantic namespace identity. Code should use namespace URIs.

---

# 15. QName

`etree.QName` helps construct/decompose qualified names.

Use it instead of hand-building `{uri}local` strings repeatedly in namespace-heavy code.

---

# 16. XPath

```python
nodes = tree.xpath("//x:item[@id=$id]", namespaces=ns, id="42")
```

XPath supports:
- node sets;
- booleans;
- numbers;
- strings;
- variables;
- namespaces;
- extension functions.

XPath is a major reason to choose lxml over selectolax.

---

# 17. Compiled XPath

```python
expr = etree.XPath("//item[@type=$t]")
for tree in trees:
    result = expr(tree, t="x")
```

Compile repeated expressions instead of reparsing them for every document.

---

# 18. XPathEvaluator

Evaluator objects are useful when many XPath expressions share a context/tree and namespace configuration.

Prefer simpler compiled `XPath` objects unless evaluator state genuinely helps.

---

# 19. EXSLT and regex

lxml's XPath/XSLT layer supports useful EXSLT facilities depending on native-library support.

Treat extension availability as a version/runtime capability if cross-platform determinism matters.

---

# 20. XSLT

```python
transform = etree.XSLT(xslt_tree)
result = transform(xml_tree)
```

Compile stylesheet once and reuse.

XSLT can:
- transform XML;
- generate text/HTML/XML;
- accept parameters;
- call extension functions;
- access external resources depending on configuration.

---

# 21. XSLT security

Untrusted XSLT can be code-like.

Control:
- file read/write;
- network;
- resolvers;
- extension functions;
- resource limits.

Do not execute arbitrary user-provided stylesheets in a privileged process without isolation.

---

# 22. XSLT parameters

Use `etree.XSLT.strparam()` when a value must be passed as a literal string instead of being interpreted as an XPath expression.

This avoids subtle quoting/injection errors.

---

# 23. Validation overview

lxml supports:
- DTD;
- Relax NG;
- XML Schema;
- ISO Schematron.

Common validator pattern:

```python
schema = etree.XMLSchema(schema_tree)
if not schema.validate(doc):
    for error in schema.error_log:
        ...
schema.assertValid(doc)
```

Compile validator once and reuse across documents.

---

# 24. DTD

DTD support can occur:
- during parse;
- via `etree.DTD`.

DTD can provide validation and default attributes, but DTD loading introduces entity/resource security concerns.

---

# 25. Relax NG

```python
relaxng = etree.RelaxNG(schema_tree)
relaxng.validate(doc)
```

Use Relax NG when the document contract is naturally expressed in that schema language.

Includes/imports may resolve resources, so configure resolver/network policy.

---

# 26. XML Schema

```python
xsd = etree.XMLSchema(xsd_tree)
xsd.validate(doc)
```

Use `error_log` for diagnostics.

Schema validation is not just parsing; keep “well-formed XML” and “schema-valid XML” separate in API contracts.

---

# 27. ISO Schematron

Use `lxml.isoschematron.Schematron`.

The implementation compiles Schematron through the reference XSLT skeleton flow:
- includes;
- abstract expansion;
- XSLT compilation;
- validation/report processing.

This is richer than the legacy libxml2 pre-ISO Schematron facility.

---

# 28. Legacy etree.Schematron caution

Availability of old libxml2 Schematron support depends on the native libxml2 feature set and is disappearing with newer libxml2 lines.

For ISO Schematron, prefer `lxml.isoschematron`.

---

# 29. Error logs

Many parsers/transforms/validators expose detailed error logs.

Do not reduce them immediately to `str(exc)`. Persist structured details such as:
- domain/type;
- line/column;
- level;
- message;
- filename.

This materially improves debugging and corpus triage.

---

# 30. Serialization

```python
etree.tostring(
    node,
    encoding="utf-8",
    xml_declaration=True,
    pretty_print=False,
)
```

Options cover:
- XML/HTML/text output method;
- encoding;
- declaration;
- pretty printing;
- tails;
- standalone/doctypes depending on API.

Pretty-printing is presentation, not canonicalization.

---

# 31. Canonical XML (C14N)

lxml supports canonical XML serialization modes.

Use C14N for XML canonicalization/signature workflows rather than inventing:
- attribute sorting;
- namespace sorting;
- whitespace serialization.

Know which C14N version/options your protocol requires.

---

# 32. CDATA

lxml can construct CDATA sections where needed.

Remember that parsed XML semantics often do not distinguish CDATA from equivalent text content after parsing; only serialization/source representation does.

---

# 33. Comments and processing instructions

Comments/PIs can be preserved, iterated, created, removed, and serialized depending on parser settings.

If signatures or document preservation depend on them, do not use a parser configuration that silently removes them.

---

# 34. XInclude

lxml supports XInclude.

XInclude can load external resources. Apply the same resource-resolution security policy as DTD/XSLT/schema imports.

---

# 35. Custom resolvers

Subclass resolver APIs to control external resource resolution.

Use resolvers for:
- offline schema catalogs;
- controlled include paths;
- deterministic tests;
- blocking network access;
- remapping external identifiers.

This is superior to monkeypatching filesystem/network calls.

---

# 36. XML catalogs

libxml2 catalog facilities can support controlled external identifier resolution.

Use catalogs for standards-heavy XML systems where many schemas/imports must be resolved offline.

---

# 37. Extension XPath/XSLT functions

Python functions can be exposed to XPath/XSLT.

Use sparingly:
- they couple stylesheets/queries to Python;
- can break portability;
- may introduce security and performance concerns.

If a pure XPath/XSLT solution is adequate, keep the transformation declarative.

---

# 38. Custom element classes

lxml allows custom Python Element subclasses via class lookup schemes.

Use when element behavior is domain-specific and tightly coupled to tag/namespace identity.

Avoid building a heavy ORM-like abstraction when plain XPath/tree helpers are simpler.

---

# 39. objectify

`lxml.objectify` provides data-binding-like access where XML elements can map more naturally to Python values/attributes.

Good for:
- structured XML application data.

Be cautious when:
- preserving lexical XML distinctions;
- schemas/namespaces are complex;
- exact tree manipulation is needed.

---

# 40. lxml.html

HTML helpers include:
- document/fragment parsing;
- link iteration/rewriting;
- forms;
- element helpers;
- builders;
- diff-related functionality.

Use these when they save code over generic etree.

Do not assume HTML sanitization is safely provided by `lxml.html` itself in modern lxml; cleaning is separate.

---

# 41. Link handling

`lxml.html` knows common link-bearing attributes.

6.1.1 specifically fixed omission of `xlink:href` from known link attributes because such omissions can cause URL-bypass problems in SVG/MathML-related content.

Security lesson: link extraction/sanitization must account for namespaces and embedded vocabularies, not only `<a href>`.

---

# 42. CSS selectors

`lxml.cssselect` translates CSS selectors to XPath and depends on the separate `cssselect` package.

For HTML scraping that only needs CSS, selectolax may be faster/simpler. Use lxml CSS when the document is already an lxml tree or XPath interoperability matters.

---

# 43. Builder APIs

`lxml.builder.E` and HTML builders create trees declaratively.

Use for generating XML/HTML while preserving correct escaping and namespace construction.

Do not concatenate markup strings for structured output when builder APIs suffice.

---

# 44. SAX interoperability

lxml can bridge with SAX-style processing.

Use when integrating with APIs that consume/emit SAX events; otherwise native etree/pull parsing is usually simpler.

---

# 45. Threading

lxml releases the GIL in various native operations and has nuanced threading behavior.

Safe default:
- separate parser/tree operations by worker;
- do not mutate the same tree concurrently;
- compile/reuse read-only transforms/validators only where documented/tested for your workload.

Measure before designing a complex threaded parser pool.

---

# 46. Memory management

Large retained trees consume native + Python memory.

For high-volume XML:
- `iterparse`;
- clear completed elements;
- keep extracted primitives;
- release references promptly.

For HTML crawls:
- parse one document;
- extract;
- discard tree.

---

# 47. Performance

High-leverage optimizations:
- compile XPath;
- compile XSLT once;
- compile validators once;
- stream large XML;
- avoid repeated cross-document node moves;
- avoid Python callbacks in hot XPath/XSLT loops unless needed.

---

# 48. Node movement / copying

Moving elements between documents can incur native bookkeeping. For repeated template-like use, copy/clone deliberately rather than repeatedly reparenting a shared node.

Always be clear whether an operation moves a node or duplicates it.

---

# 49. Unicode / bytes

XML has explicit encoding declarations. Prefer passing bytes when source encoding declarations must be honored.

Passing a decoded Python `str` means decoding has already happened; an XML encoding declaration can then become inconsistent or irrelevant.

---

# 50. lxml vs stdlib ElementTree

Choose lxml for:
- parent/sibling axes;
- XPath power;
- XSLT;
- schema validation;
- C14N;
- custom resolvers;
- performance/native facilities.

Choose stdlib when:
- dependency minimization matters;
- XML needs are simple;
- security policy prefers a smaller surface.

---

# 51. lxml vs selectolax

selectolax:
- HTML-focused;
- CSS-focused;
- very fast;
- simpler extraction/mutation.

lxml:
- XML + HTML;
- XPath;
- validation;
- transformations;
- namespaces;
- canonical XML;
- richer resource resolution.

Do not use lxml solely because it is familiar if selectolax better matches a hot HTML-only path.

---

# 52. Security floor: 6.1.1

6.1.1 includes:
- correction of `xlink:href` recognition in lxml.html link attributes;
- patched libxslt in Linux wheels addressing documented CVEs.

Do not downgrade below security fixes in a system that processes untrusted markup unless a controlled build has equivalent patches.

---

# 53. 7.0 prerelease posture

7.0 alphas exist as of this reference date. Do not design production code against alpha-only behavior unless the project intentionally adopts a prerelease line.

Re-test:
- parser defaults;
- native libxml2/libxslt compatibility;
- PyPy/free-threading;
- removed/deprecated APIs.

---

# 54. Testing matrix

```text
[ ] XML bytes with declaration
[ ] Unicode XML string
[ ] malformed XML rejected
[ ] recovery mode where intentionally enabled
[ ] external entity blocked
[ ] DTD blocked / allowed cases
[ ] namespace-heavy document
[ ] XPath variable binding
[ ] compiled XPath reuse
[ ] XSLT with parameters
[ ] XSLT external-resource denial
[ ] DTD validation
[ ] Relax NG validation
[ ] XML Schema validation
[ ] ISO Schematron
[ ] C14N expected bytes
[ ] XInclude policy
[ ] iterparse large document
[ ] custom resolver
[ ] HTML malformed repair
[ ] SVG/MathML link attributes
```

---

# 55. LLM-agent execution playbook

Before implementing custom XML/HTML infrastructure, check:

```text
parse/fromstring
-> iterparse/XMLPullParser
-> XPath / compiled XPath
-> XSLT
-> validator class
-> C14N serialization
-> resolver/catalog
-> lxml.html helper
-> custom target/element class/extension function
```

The library already owns a large amount of standards logic. Reimplementation is usually lower-fidelity.

---

# 56. Anti-pattern inventory

- `resolve_entities=True` on untrusted XML without a reason;
- `no_network=False` by default in a crawler;
- regex XML parsing;
- hand-written XPath parser logic;
- recompiling XPath/XSLT/schema per record;
- reading giant XML into a DOM when iterparse suffices;
- treating pretty-print as canonicalization;
- using untrusted XSLT as data;
- assuming only `<a href>` contains links;
- using `lxml.html.clean` as though it were still bundled sanitizer behavior;
- ignoring native libxml2/libxslt versions in reproducibility investigations.
