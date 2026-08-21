---
name: web-acquisition-lib-ref
description: "Reference navigator for the six-library web acquisition and HTML extraction stack. SKILL.md maps six deep-dives at `docs/library_ref/`: `httpx2_python_advanced_reference_2.9.1.md` (HTTP client §0-§36), `playwright_python_advanced_reference_1.62.0.md` (browser automation §0-§52), `lxml_python_advanced_reference_6.1.1.md` (XML/XPath/validation §0-§56), `selectolax_python_advanced_reference_0.4.11.md` (fast HTML/CSS §0-§40), `tldextract_python_advanced_reference_5.3.2.md` (PSL hostname decomposition §0-§43), `extruct_python_advanced_reference_0.18.0.md` (embedded metadata §0-§30); REFERENCE.md (same folder) holds per-doc section indexes with line numbers, the cross-document authority matrix, ten decision trees, the full 23 operating rules, and the navigation hazards. Use when Python touches `import httpx2`/`from playwright.sync_api`/`from playwright.async_api`/`from selectolax.lexbor import LexborHTMLParser`/`from lxml import etree`/`from lxml import html`/`import extruct`/`import tldextract`, or authors `AsyncClient`/`Limits`/`Timeout`/`MockTransport`/`client.stream(`/`follow_redirects`/`BrowserContext`/`page.route`/`route_from_har`/`expect_download`/`page.content()`/`LexborHTMLParser`/`css_first`/`merge_text_nodes`/`iterparse`/`XMLParser`/`etree.XPath(`/`extruct.extract`/`base_url=`/`TLDExtract(`/`top_domain_under_public_suffix`/`registry_suffix`, or edits `pyproject.toml` pins for those six. Pydantic lives in sibling skill `fastmcp-pydantic-ref`; `jsonschema_python_advanced_reference_4.26.0.md` exists but no skill routes it."
allowed-tools: Read, Grep, Glob, Bash
---

# Web Acquisition & HTML Extraction Reference Navigator

Routes six deep-dive references for the **Python** acquisition-and-extraction stack. This SKILL.md is the **core map**: version anchors and drift, the six-document topic table, the pipeline and its stage ownership, where-to-look routing, the key invariants, and the navigation entry points. The companion **`REFERENCE.md`** (same folder) carries the per-document section indexes (all 263 sections with absolute line numbers), the cross-document authority matrix, the ten decision trees (acquisition escalation · parser choice · metadata syntax · PSL policy · timeout classification · retry · streaming · test doubles · bytes-vs-str · browser isolation), the full 23 operating rules, and the navigation hazards. Reach for REFERENCE.md once you know *which* document you need and want its section map; cross-references back here are written `SKILL §...`.

These are **pure library navigators**. They index what the six references say, nothing more — no project doctrine, no design-spec anchoring, no work-packet mapping.

**Out of scope** (covered elsewhere): Pydantic 2.13.4 → sibling skill **`fastmcp-pydantic-ref`**. `jsonschema` 4.26.0 — `docs/library_ref/jsonschema_python_advanced_reference_4.26.0.md` exists and the package is a pinned direct dependency, but **no skill routes it**; read it directly. Robots/Protego, Tenacity, feedparser, `idna`, and `ipaddress` have **no reference document in this repo**. `requests` appears only transitively, as the session type tldextract accepts for PSL fetching (`tldx §16`, `§29`) — it is not an acquisition path here. Rust-side siblings: `datafusion-pyarrow-rust-ref`, `deltalake-rust-ref`, `code-facts-lib-ref`, `gix-notify-ref`, `petgraph-ref`.

---

## Version anchors

* **httpx2 2.9.1** (released 2026-07-24; Python 3.10+; BSD-3) — a **distinct package from `httpx`**, stewarded by Pydantic Services, depending on `httpcore2`/`h11`/`anyio`/`truststore`/`idna`. Sync + async clients, HTTP/2 behind an extra, native WebSockets behind the `ws` extra. Change imports deliberately; do not rely on an accidental transitive `httpx` (`httpx2 §32`).
* **Playwright 1.62.0** (released 2026-07-31) — the Python package version and the managed browser binaries are **coupled**; upgrade both together (`pw §1`, `§3`). 1.62 adds WebP screenshots, an action `scroll` option, `Locator.wait_for_function()`, and `APIResponse.timing`.
* **lxml 6.1.1** (released 2026-05-18) — binding over libxml2/libxslt; **behavior tracks the native library versions, not just the wheel** (`lxml §1`). 6.1 changed the default to `resolve_entities="internal"`; 6.1.1 is a security floor (`xlink:href` link recognition, patched libxslt in Linux wheels — `lxml §52`). 7.0 alphas exist and are not adopted (`lxml §53`).
* **selectolax 0.4.11** (released 2026-07-15; Python >=3.9,<3.15) — Cython wrapper over two engines. **Lexbor is the engine for new code**; Modest (`selectolax.parser.HTMLParser`) is legacy and its node classes are *not* interchangeable (`slax §1`, `§30`). Recent 0.4.x churn (free-threading, pretty serialization, memory fixes) is itself a reason to pin (`slax §37`).
* **tldextract 5.3.2** (released 2026-08-08; Python >=3.10) — pure Python with a **bundled PSL snapshot** plus optional remote update. 5.3.x added `registry_suffix`, `top_domain_under_public_suffix`, and `top_domain_under_registry_suffix`, and deprecated `registered_domain` (`tldx §40`).
* **extruct 0.18.0** (released 2024-11-08; Python >=3.8) — the **slowest-moving library in the pack**; PyPI classifier is still Beta. Pin it and run a metadata corpus on every dependency upgrade, watching rdflib/mf2py/lxml compatibility (`extruct §27`).

**Version drift — the docs are not all at the installed version.**

| Library | Doc anchor | Adopted-version register | `uv.lock` / installed |
|---|---|---|---|
| **httpx2** | 2.9.1 | 2.9.1 | **2.12.0** ⚠ |
| **lxml** | 6.1.1 | 6.1.1 | **6.1.2** ⚠ |
| extruct · playwright · selectolax · tldextract | 0.18.0 · 1.62.0 · 0.4.11 · 5.3.2 | identical | identical (exact) |

The httpx2 gap is **three minor releases and unaudited**. `httpx2 §32` (HTTPX→HTTPX2 migration posture) and `§33` (the 2.9.1 `alias_httpx()` fix) are the two sections most likely to have moved; verify against the installed package before relying on either. The lxml gap is one patch — treat `§52`'s security floor as a floor, not a ceiling.

---

## The six reference documents

All live at `docs/library_ref/`. All six share **one identical flat template**: a title H1, a `## Version / source anchors` block, then numbered `# N. Title` H1 sections running 5-40 lines each, ending with an **anti-pattern inventory**. There are no `## N.M` subsections except in httpx2 (11 `##` / 6 `###`) and a three-entry `##` block in `tldx §35`. That uniformity is also the pack's main hazard: **the `# N.` H1 prefix collides across all six files** — scope every grep to one filename (SKILL §Navigation).

| Doc | Path (`docs/library_ref/`) | Lines | Sections | Scope |
|-----|------|------:|---|-------|
| **httpx2** | `httpx2_python_advanced_reference_2.9.1.md` | 856 | **§0-§36** | client/transport architecture, top-level API, client lifecycle, request construction, URL model, params, headers, cookies, response model, redirects, streaming, timeouts, pooling limits, HTTP/2, TLS, env vars, proxies, auth, event hooks, transports (HTTP/WSGI/ASGI/Mock/custom), mounts, native WebSockets, request/response extensions, exception hierarchy, retries, compression, CLI, in-process testing, performance, security, HTTPX migration. |
| **pw** | `playwright_python_advanced_reference_1.62.0.md` | 881 | **§0-§52** | object model, install, sync/async, engines, `BrowserType`, `Browser`, `BrowserContext` isolation, context config, navigation, locator-first design + strategies + filtering + strictness, auto-waiting, actions, assertions, frames, shadow DOM, JS evaluation, handles, downloads/uploads/dialogs/popups, network observation, routing, HAR, WebSockets, `APIRequestContext`, storage state, permissions, emulation, `Clock`, WebAuthn, screenshots/PDF/video/tracing, ARIA snapshots, codegen, pytest plugin, parallelism, CI, security boundary, escalation policy, extraction handoff. |
| **lxml** | `lxml_python_advanced_reference_6.1.1.md` | 830 | **§0-§56** | capability map, native deps, core modules, tree construction, Element vs ElementTree, XML parsing, **parser security defaults**, HTML parsing, recovery, parser targets, feed parser, `iterparse`, pull parser, axes, namespaces, `QName`, XPath + compiled XPath + evaluator, EXSLT, XSLT + security + params, DTD/RelaxNG/XMLSchema/Schematron validation, error logs, serialization, C14N, CDATA, comments/PIs, XInclude, resolvers, catalogs, extension functions, custom element classes, objectify, `lxml.html`, link handling, cssselect, builders, SAX, threading, memory, performance, node movement, unicode/bytes, comparisons. |
| **slax** | `selectolax_python_advanced_reference_0.4.11.md` | 612 | **§0-§40** | what it is and is not, Lexbor vs Modest, install, `LexborHTMLParser` construction + fragments, root/head/body, CSS selection, selector strictness, `select()` helper, **node lifetime**, navigation, node types, tag/id, `attributes` vs `attrs`, text extraction, `merge_text_nodes`, `raw_value`, comments, script helpers, tag lookup, fragment parsing, node creation, insertion, `replace_with`, decompose, `strip_tags`, unwrap, `inner_html`, serialization, HTML5 repair semantics, encoding, Modest, free-threading, performance, memory bounds, when-to-use-lxml, when-to-use-Playwright, errors. |
| **tldx** | `tldextract_python_advanced_reference_5.3.2.md` | 680 | **§0-§43** | purpose, why dot-splitting fails, one-shot API, `TLDExtract` constructor, production extractor, `ExtractResult` fields, `suffix`, `registry_suffix`, `is_private`, `top_domain_under_public_suffix`, `registered_domain` deprecation, `top_domain_under_registry_suffix`, `fqdn`, `reverse_domain_name`, dataclass semantics, `extract_str`, `extract_urllib`, **URL-validation boundary**, unknown-suffix behavior, IPs, localhost, public vs private PSL, `include_psl_private_domains`, data sources, bundled snapshot, cache + update + timeout, custom session, `extra_suffixes`, `tlds`, **IDNA/Punycode boundary**, unicode dots, security uses and non-uses, reproducibility, concurrency, performance, CLI, data model, 5.3.x migration. |
| **extruct** | `extruct_python_advanced_reference_0.18.0.md` | 473 | **§0-§30** | purpose, install, primary API, input forms, `base_url`, syntax selection, JSON-LD + shape variability, Microdata + `return_html_node`, Open Graph + `with_og_array`, Microformats, RDFa, Dublin Core, `uniform` output, error policy, individual extractor classes, parsed-tree reuse, selectolax coexistence, Playwright coexistence, URL resolution, duplicate entities, schema.org normalization, trust boundary, resource limits, CLI, version posture. |

**Reading strategy.** Find the section in REFERENCE.md's per-doc index, then `Read(offset, limit)` with the absolute line number given there. Sections are short — **read whole sections, not offsets into them**. Every doc's final section is its anti-pattern inventory; five of six also carry a testing section (SKILL §Navigation).

---

## The pipeline

The six libraries form one linear chain, and the documents themselves already say so — `pw §47`/`§48`, `slax §34`/`§35`, `lxml §51`, `extruct §19`/`§20` are mutual cross-links. REFERENCE.md §2 formalizes the handoffs and arbitrates the overlaps.

```text
URL string
  -> tldx      host decomposition, PSL boundary, provenance fields    (tldx §0, §39)
  -> httpx2    bytes + final URL + headers, bounded and streamed      (httpx2 §9-§12)
     \-> pw    escalate ONLY when JS execution, interaction, browser
               storage, or browser-network observation is required     (pw §47)
  -> slax      HTML DOM, CSS selection, text extraction               (slax §5, §13)
  -> lxml      XML, namespaces, XPath, feeds/sitemaps, streaming      (lxml §7, §11, §16)
  -> extruct   embedded metadata, over HTML bytes or an lxml tree     (extruct §2, §18)
```

**Handoff rule.** `page.content()` is post-DOM serialization, **not** the original network bytes — page JavaScript can inject or rewrite JSON-LD, so choose server-source vs rendered metadata deliberately (`extruct §20`, `pw §48`). Do not keep a browser alive to do offline DOM traversal a parser does cheaper (`pw §48`, `slax §35`).

---

## Where do I look?

| Question | Doc |
|---|---|
| Fetch bytes over HTTP — clients, pooling, timeouts, redirects, streaming, proxies, TLS, auth | **httpx2** |
| Classify a failure: which timeout fired, which exception, is it retryable | **httpx2** §12 · §24 · §25 |
| Run JavaScript, interact with a page, observe or intercept browser network traffic | **pw** |
| Isolate an identity — cookies, storage, permissions, locale, proxy per session | **pw** §6 · §7 · §31 |
| Parse ordinary HTML fast and pull fields out by CSS | **slax** |
| Extract clean text from markup | **slax** §13 · §14 |
| Parse XML, a sitemap, an RSS/Atom feed, or anything namespaced | **lxml** §5 · §11 · §14 |
| Use XPath, XSLT, DTD/RelaxNG/XSD/Schematron validation, or C14N | **lxml** — the only doc that covers these |
| Stream a very large XML document without a full DOM | **lxml** §11 `iterparse` · §12 |
| Read embedded JSON-LD, Microdata, Open Graph, Microformats, RDFa, or Dublin Core | **extruct** |
| Split a hostname into subdomain / domain / suffix correctly | **tldx** |
| Decide public suffix vs registry suffix, or private-PSL policy | **tldx** §7 · §8 · §22 · §23 |
| Mock or replay the network in tests | **httpx2** §20/§28/§29 (transport) · **pw** §27 (HAR, browser) |
| Bound untrusted input — size, depth, entities, decompression, SSRF | **httpx2** §31 · **lxml** §6/§21 · **pw** §46 · **extruct** §24/§25 · **tldx** §34 |
| **Canonicalize a hostname (IDNA, case, Punycode)** | **httpx2** §5 — `URL.raw_host` is an IDNA 2008/UTS-46 A-label (verified). `tldx §32` disclaims it; never use stdlib `encodings.idna` (IDNA 2003). See REFERENCE §2 |

For the full arbitration — which doc is *authoritative* per topic — and the ten decision trees, see **`REFERENCE.md`** §2 and §3.

---

## Key invariants

The eight that prevent the most errors; the full set of **23 operating rules** is in `REFERENCE.md` §4.

1. **HTTPX-family clients do not follow redirects by default.** Set `follow_redirects=True` deliberately, read the chain from `response.history`, and re-apply URL policy to *every* redirect target rather than only the initial URL. (httpx2 §10, §31)
2. **One long-lived `Client`/`AsyncClient` per policy boundary — never one per request.** The client owns the pool, cookies, auth, TLS, and routing; scope it per proxy/TLS/auth/tenant boundary. Closing it releases pooled connections; context managers are the safe default. Streamed responses must be closed, and `.content` on an unread streaming response is not buffered. (httpx2 §3, §11, §36)
3. **Playwright's isolation unit is `BrowserContext`, not `Browser`** — a browser process is expensive, contexts are cheap, and one context per identity is the rule. **Locators re-resolve; `ElementHandle`s go stale.** Prefer locator APIs and web-first `expect(...)` assertions over manual polling or `time.sleep()`. (pw §0, §6, §9, §16, §20)
4. **selectolax nodes die with their parser.** Never cache a node beyond its parser's lifetime — extract Python primitives first. And `node.attributes` is a **value copy** while `node.attrs` is a **live mutating** interface; use the copy when read-only intent suffices. (slax §8, §12)
5. **For any HTML5 parser, parsed serialization ≠ source bytes.** Implied `html`/`head`/`body`, repaired nesting, normalized attributes and entities all mean the tree is not the input. If fidelity matters, retain the original bytes alongside the tree. (slax §4, §28; lxml §7)
6. **lxml needs explicit parser policy for untrusted input**, even though 6.1 improved the default to `resolve_entities="internal"` — set `resolve_entities=False`, `load_dtd=False`, `no_network=True`, `huge_tree=False`, then re-enable only what you need. Also: **`lxml.html.clean` is no longer bundled** (it is the separate `lxml_html_clean` project), and XPath/XSLT/validators must be **compiled once and reused**, not rebuilt per document. (lxml §2, §6, §17, §40, §47)
7. **extruct requires `base_url` set to the final post-redirect URL**, or relative `@id`/image/canonical values resolve wrongly or not at all. extruct *extracts*; it does not validate against schema.org and does not reconcile duplicate entities — keep normalization, validation, and entity resolution in separate downstream stages that retain source syntax and provenance. (extruct §4, §21, §22, §23)
8. **tldextract's `suffix` is not a TLD and `domain` is not a registrable domain.** Use `top_domain_under_public_suffix`; `registered_domain` is deprecated and slated for removal. `ExtractResult` is a **dataclass, not a namedtuple** — no `r[1]`, no tuple-unpacking. An empty `suffix` for an unrecognized final label is deliberate, not a failure. (tldx §7, §10, §11, §15, §19, §43)

---

## Navigation

**The H1 prefix collides across all six files.** Every doc numbers its sections `# N. Title`, so `grep '^# 13\.' docs/library_ref/*.md` returns six unrelated hits. Always scope to one filename, and always write citations with an alias (`lxml §13`, never a bare `§13`). Doc filenames embed versions and change when a doc is refreshed — glob by prefix (`extruct_python_advanced_reference_*`) rather than by exact name.

**Read the anti-pattern inventory for the library you are touching before drafting code**, and its testing section before writing tests. These twelve sections are the pack's highest-yield entry points:

| Doc | Anti-pattern inventory | Testing section |
|---|---|---|
| **httpx2** | §36 · line 844 | §35 Testing matrix · 818 |
| **pw** | §52 · 869 | §51 Testing matrix · 842 |
| **lxml** | §56 · 818 | §54 Testing matrix · 770 |
| **slax** | §40 · 601 | §38 Testing strategy · 555 |
| **tldx** | §43 · 670 | §41 Testing matrix · 625 |
| **extruct** | §30 · 462 | §28 Testing corpus · 412 |

The testing section carries **three different titles** across the pack — grep `^# [0-9]*\. Testing`, not an exact phrase. Likewise the agent-rules section is "LLM-agent decision rules" in five docs but "LLM-agent execution playbook" in lxml (§55). Full section indexes with line numbers for all 263 sections: `REFERENCE.md` §1. Further hazards: `REFERENCE.md` §5.
