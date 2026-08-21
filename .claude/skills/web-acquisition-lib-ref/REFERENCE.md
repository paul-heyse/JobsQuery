# Web Acquisition & HTML Extraction — Detailed Reference

The deep-dive companion to `SKILL.md`. SKILL.md is the core map — version anchors and drift, the six-document topic table, the pipeline, where-to-look routing, the eight key invariants, and the entry-point table. This file is the mechanical layer: every section of every document with its absolute line number, the arbitration between documents that claim the same topic, the decision trees, the full operating rules, and the hazards of navigating this particular pack.

Cross-references back to the core map are written `SKILL §...`. Come here when you already know *which* document you need and want its section map, or when two documents appear to cover the same ground and you need to know which one is authoritative.

All six documents share one shape: a title H1, a `## Version / source anchors` block, then numbered `# N. Title` H1 sections, ending with an anti-pattern inventory. Sections run **5-40 lines**; read whole sections rather than paging into them. Only httpx2 has meaningful nesting (11 `##` / 6 `###`); `tldx §35` has a three-entry `##` block. Everything else is flat.

**Document aliases** (used throughout, defined in SKILL §The six reference documents):

| Alias | File (`docs/library_ref/`) | Lines | Sections |
|---|---|---:|---|
| `httpx2` | `httpx2_python_advanced_reference_2.9.1.md` | 856 | §0-§36 (37) |
| `pw` | `playwright_python_advanced_reference_1.62.0.md` | 881 | §0-§52 (53) |
| `lxml` | `lxml_python_advanced_reference_6.1.1.md` | 830 | §0-§56 (57) |
| `slax` | `selectolax_python_advanced_reference_0.4.11.md` | 612 | §0-§40 (41) |
| `tldx` | `tldextract_python_advanced_reference_5.3.2.md` | 680 | §0-§43 (44) |
| `extruct` | `extruct_python_advanced_reference_0.18.0.md` | 473 | §0-§30 (31) |

263 numbered sections across 4,332 lines.

## Table of Contents

- **§1 — Per-document section indexes** (every section, with absolute line numbers, for `Read(offset, limit)`)
- **§2 — Cross-document authority matrix** (which document wins when several cover a topic)
- **§3 — Decision trees** (ten, covering escalation, parser choice, metadata, PSL, timeouts, retries, streaming, test doubles, encoding, browser isolation)
- **§4 — Operating rules** (the full 23)
- **§5 — Navigation hazards** (what breaks when you grep this pack naively)

---

## §1 — Per-document section indexes

### §1.1 `httpx2_python_advanced_reference_2.9.1.md` — section index (§0-§36)

Front matter: title `1`, `## Version / source anchors` `3`, `### Source-of-truth hierarchy` `10`, `## Capability inventory` `27`. Numbered sections start at line `52`. **This is the only doc with real nesting** — subsections are listed inline below.

| § | Line | Title | Key content |
|---|------|-------|-------------|
| **0** | 52 | Mental model and architecture | the four-layer stack `application → Client → Request/Response → Transport → httpcore2`; what the client owns; do not bypass the client layer |
| **1** | 70 | Installation and extras | extras table (`http2`/`socks`/`ws`/`cli`/`brotli`/`zstd`) · `### 94` production dependency rule — pin `httpcore2` too if you use trace-extension event names |
| **2** | 100 | Top-level request API | `get`/`post`/`request` module functions; no connection reuse; anti-pattern in a crawler loop |
| **3** | 120 | Client lifecycle | context-manager usage; one client per policy boundary; `### 147` concurrency — asyncio/Trio via AnyIO, never mix sync and async transports |
| **4** | 153 | Request construction | `build_request`/`send`; `content`/`json`/`data`/`files`; bodies on GET/DELETE need generic `request()` · `### 177` chunked bodies from iterators |
| **5** | 183 | URL model | `httpx2.URL` is structured, not a string; `str(url)` at serialization boundaries; IDNA handling; parse success ≠ safety |
| **6** | 203 | Query parameters | `QueryParams`; duplicate-key and ordering semantics; do not collapse to a dict |
| **7** | 216 | Headers | case-insensitive multi-dict; repeated headers matter; never log credentials by default |
| **8** | 229 | Cookies | cookies belong on the Client; stricter than Requests; `CookieConflict` on ambiguous name-only lookup |
| **9** | 243 | Response model | `status_code`/`headers`/`url`/`history`/`content`/`text`/`encoding`/`extensions`; `json()`; `raise_for_status()` · `### 269` **encoding** — prefer bytes when fidelity matters |
| **10** | 275 | Redirects | **not followed by default**; `follow_redirects=True`; `history`; `next_request`; re-apply SSRF policy to every redirect target |
| **11** | 289 | Streaming downloads | `client.stream()` sync/async; `iter_bytes`/`aiter_bytes`; must be closed; `.content` on unread stream is not buffered; `StreamConsumed`/`StreamClosed`/`ResponseNotRead` |
| **12** | 318 | Timeouts | four components — connect/read/write/pool; `httpx2.Timeout(...)`; `ConnectTimeout`/`ReadTimeout`/`WriteTimeout`/`PoolTimeout`; differentiated budgets |
| **13** | 350 | Resource limits and pooling | `httpx2.Limits`; max connections, max keep-alive, expiry; `PoolTimeout` as a saturation signal, not a retry trigger |
| **14** | 366 | HTTP/2 | `http2=True` + extra; multiplexing is not automatically faster; `response.http_version`; proxies alter negotiation |
| **15** | 387 | TLS / SSL | verification on by default; `truststore`; deliberate SSL context for custom CA/mTLS; separate clients per TLS policy |
| **16** | 402 | Environment variables | proxy and cert env vars honored by default; `trust_env=False` for hermetic services |
| **17** | 416 | Proxies | `proxy=`; `mounts` for per-scheme routing; SOCKS extra; **HTTPS destinations still use an `http://` proxy URL** via CONNECT |
| **18** | 440 | Authentication | Basic/Digest/custom auth flow classes; body replay for challenges; streamed bodies complicate retry; scope tokens per host |
| **19** | 455 | Event hooks | request/response hooks for observability, request IDs, metrics, header injection; do not build retry orchestration inside hooks |
| **20** | 473 | Transports | `## 475` HTTPTransport/Async (connect retries, local address, UDS, proxy placement) · `## 488` WSGITransport · `## 498` ASGITransport (lifespan is separate) · `## 504` MockTransport · `## 515` custom `BaseTransport` |
| **21** | 523 | Mount-based routing | route by scheme/host/wildcard/port; routing belongs in client construction |
| **22** | 539 | Native WebSockets in HTTPX2 | `ws` extra; `client.websocket(...)` sync and async; text/binary/JSON; subprotocols, keepalive pings, receive timeout; `httpx2.websockets` exceptions · `### 585` **ASGIWebSocketTransport** — the major difference from the `httpx-ws` era |
| **23** | 593 | Request / response extensions | low-level escape hatch: request `trace`/`sni_hostname`/`timeout`/`target`; response `http_version`/`reason_phrase`/`stream_id`/`network_stream` · `## 611` trace (version-coupled to httpcore2) · `## 615` SNI override (security-sensitive) · `## 621` target override · `## 625` network_stream |
| **24** | 631 | Exceptions | the `HTTPError → RequestError → TransportError` tree; `HTTPStatusError`; `InvalidURL`/`CookieConflict`/stream-state errors; catch narrowly |
| **25** | 670 | Retries | **no universal retry switch**; transport retries cover connection establishment only; the eight things an application retry layer must define |
| **26** | 689 | Compression and decoding | automatic content-encoding; Brotli/Zstd extras; keep raw-wire vs decoded-body requirements separate for signatures/hashes |
| **27** | 702 | CLI | `cli` extra; diagnostics only; pin if output is parsed |
| **28** | 714 | In-process application testing | WSGI→`WSGITransport`, ASGI→`ASGITransport`, ASGI WS→`ASGIWebSocketTransport`, external→`MockTransport` |
| **29** | 727 | Mocking ecosystem | native `MockTransport` is enough for deterministic request→response mapping |
| **30** | 735 | Performance playbook | eight rules: reuse clients, stream, bound pools, async only with an async stack, no unbounded fan-out, HTTP/2 after validation, stable policy per client, measure phases separately |
| **31** | 748 | Security playbook | the untrusted-URL checklist — schemes, credentials, SSRF, redirect revalidation, DNS rebinding, forbidden ranges, size/time bounds, TLS, decompression bounds, `Content-Type`, log redaction |
| **32** | 768 | HTTPX → HTTPX2 migration posture | distinct package; change imports deliberately; verify third-party integrations; plugins using private internals may break ⚠ *doc predates the installed 2.12.0* |
| **33** | 785 | 2.9.1-specific release note | `alias_httpx()` redirects `httpcore` → `httpcore2`; aliasing is migration infrastructure, not steady state ⚠ *version-specific* |
| **34** | 793 | LLM-agent decision rules | when to use httpx2 vs Playwright; the built-in escalation ladder `Client option → request option → event hook → transport/mount → extension → custom transport` |
| **35** | 818 | Testing matrix | 21-item checklist: protocols, redirects, four timeout classes, proxy paths, TLS failure, streaming abort, size limits, decompression, duplicate headers, auth, retries, WSGI/ASGI, WebSocket close/disconnect/timeout, SSRF-denied targets |
| **36** | 844 | Anti-pattern inventory | 11 items — client-per-request, global `verify=False`, flat exception handling, blind retries, eager large bodies, unbounded fan-out, accidental proxy env, extensions over options, unpinned trace names, unbounded WS messages, parse-success-equals-safety |

### §1.2 `playwright_python_advanced_reference_1.62.0.md` — section index (§0-§52)

Front matter: title `1`, `## Version / source anchors` `3`. Numbered sections start at line `20`. Flat.

| § | Line | Title | Key content |
|---|------|-------|-------------|
| **0** | 20 | Mental model | the object chain `Playwright → BrowserType → Browser → BrowserContext → Page → Frame → Locator`; secondary families; **contexts are the isolation unit** |
| **1** | 50 | Installation | `python -m playwright install`, `--with-deps` on Linux/CI, per-engine install; package and browser binaries are coupled |
| **2** | 74 | Sync and async APIs | `sync_playwright()` / `async_playwright()`; never mix sync and async objects |
| **3** | 104 | Browser engines | Chromium/Firefox/WebKit; pinned tested revisions; channels |
| **4** | 118 | BrowserType | `launch`, `launch_persistent_context`, `connect`, `connect_over_cdp`; CDP is lower fidelity |
| **5** | 134 | Browser | create contexts/pages, inspect, close, version, CDP sessions; the canonical launch→context→page→close shape |
| **6** | 160 | BrowserContext: isolation boundary | what a context isolates (cookies, storage, permissions, pages, geolocation, locale/tz, proxy, storage state); one per test/identity/tenant |
| **7** | 184 | Context configuration | the full option list — viewport, scale, mobile/touch, locale, timezone, geolocation, permissions, color scheme, UA, headers, credentials, offline, proxy, client certs, service-worker policy, storage state, base URL |
| **8** | 212 | Pages and navigation | `goto`/`reload`/`go_back`/`set_content`/`content`/`title`/`url`; load states `commit`/`domcontentloaded`/`load`/`networkidle`; no `sleep()` |
| **9** | 235 | Locator-first design | a Locator is a **re-resolving query**, not a node; `get_by_role`/`get_by_label`/`get_by_text`/`get_by_test_id`/`locator` |
| **10** | 255 | Locator strategies | the nine families ranked — role+name, label, placeholder, text, alt, title, test id, CSS, XPath as escape hatch |
| **11** | 273 | Locator filtering and composition | `.filter()`, `.and_()`/`.or_()`, `.first`/`.last`/`.nth()`, descendant and frame locators |
| **12** | 287 | Strictness | ambiguous resolution fails by design; do not reflexively append `.first` |
| **13** | 295 | Auto-waiting / actionability | uniqueness, visibility, stability, receives-events, enabled/editable — why hand-written polling is wrong |
| **14** | 309 | Actions | click/hover/fill/press/check/select/focus/drag/tap/set_input_files/scroll; **1.62 adds a `scroll` option** to disable auto scroll-into-view |
| **15** | 330 | `Locator.wait_for_function()` — 1.62 | locator-level predicate wait; prefer built-in assertions, keep custom JS the exception |
| **16** | 340 | Assertions | `expect(...)` web-first retrying assertions vs `assert is_visible()` + sleeps |
| **17** | 362 | Frames and iframes | `Frame` vs `FrameLocator`; iframe DOM is not in the main frame's selector scope |
| **18** | 372 | Shadow DOM | locators pierce **open** shadow roots; closed roots stay inaccessible |
| **19** | 380 | JavaScript evaluation | `page.evaluate`/`locator.evaluate`/`evaluate_handle`; serialize compact primitives, not page objects |
| **20** | 397 | ElementHandle / JSHandle | remote objects with explicit lifetime; **handles go stale, locators do not**; when handles are justified |
| **21** | 410 | Downloads | `page.expect_download()`; suggested filename; save to a controlled path; treat downloads as untrusted, never execute |
| **22** | 429 | File uploads and file chooser | `set_input_files()`; file-chooser events; the API bypasses OS dialogs |
| **23** | 437 | Dialogs | register the handler **before** the triggering action; unhandled modals block the page |
| **24** | 445 | Popups / new pages | `page.expect_popup()` / `context.expect_page()`; the popup shares the context |
| **25** | 453 | Network observation | `request`/`response`/`requestfinished`/`requestfailed`; **404/500 are completed responses, not failures**; what to capture |
| **26** | 468 | Routing and interception | `page.route()` / `context.route()`; continue/fulfill/abort/fallback; **routing disables HTTP cache**; service workers bypass interception |
| **27** | 484 | HAR replay / recording | `route_from_har()` and miss policy; deterministic tests; HAR is not a full browser-state snapshot |
| **28** | 497 | WebSockets | `WebSocket` frame events; `route_web_socket()` / `WebSocketRoute`; set routing before the socket is created |
| **29** | 513 | APIRequestContext | in-browser HTTP client; context-associated shares cookies, standalone is isolated; **defers to HTTPX2 for general high-throughput acquisition** |
| **30** | 529 | `APIResponse.timing` — 1.62 | resource timing for diagnostics; not the same measurement system as navigation performance APIs |
| **31** | 537 | Cookies and storage state | `storage_state()`; **treat storage-state files as credentials** — out of source control, least privilege, rotate, scope per environment |
| **32** | 551 | Permissions and geolocation | context-level permission grants; headless clipboard is now OS-isolated; browser ≠ OS permissions |
| **33** | 561 | Device and environment emulation | viewport, mobile/touch, locale, timezone, color scheme, reduced motion, geolocation, UA, device descriptors; emulation ≠ real hardware |
| **34** | 578 | Clock | `context.clock` — install fake time, fix/set time, fast-forward, drive timers; replaces long real sleeps |
| **35** | 588 | WebAuthn / Credentials | virtual authenticator and credential APIs for passkey tests; distinct from HTTP Basic credentials |
| **36** | 600 | Screenshots | page and element capture; **1.62 adds WebP** (quality 100 = lossless); full-page vs viewport, masking, clipping, scale; screenshots are observations, not extraction |
| **37** | 624 | PDF | Chromium page PDF; print rendering, not an HTML-storage substitute |
| **38** | 632 | Video | context video recording; storage/CPU cost; retain-on-failure in CI |
| **39** | 640 | Tracing / Trace Viewer | actions, DOM snapshots, screenshots, network metadata; the primary forensic artifact; storage and sensitive-data cost at volume |
| **40** | 654 | ARIA snapshots and AI-oriented snapshots | element refs and iframe snapshots; lower-token than raw HTML for agent navigation; an accessibility view, not a DOM serialization |
| **41** | 664 | Codegen | `playwright codegen` for exploration and locator discovery; review the output |
| **42** | 677 | Inspector and debugging | `PWDEBUG=1`, headed mode, slow motion, devtools, trace viewer, verbose logs |
| **43** | 692 | Pytest plugin | `pytest-playwright` fixtures (`playwright`/`browser_type`/`browser`/`context`/`page`) and CLI flags; **flags apply to plugin fixtures, not hand-built contexts** |
| **44** | 718 | Parallelism | `pytest-xdist`; scale by workers, browser reuse, isolated contexts; watch RAM/CPU/process count/destination load |
| **45** | 738 | CI and containers | documented deps or the matching image; pin version + binaries; shared memory; sandbox settings; artifacts on failure |
| **46** | 752 | Browser security boundary | Playwright is **not** a sandbox for your application; egress constraints, OS/container isolation, controlled downloads, care with `evaluate()` return values; do not disable the browser sandbox casually |
| **47** | 768 | Acquisition escalation policy | the canonical list of when HTTPX2 suffices vs when a browser is genuinely required |
| **48** | 785 | Extraction handoff | `page.content()` → selectolax / lxml / extruct; do not keep a browser alive for offline traversal |
| **49** | 802 | Determinism playbook | what to stabilize: browser version, viewport, locale/tz, clock, permissions, geolocation, routing, service workers, storage state, fixtures/HAR, semantic locators |
| **50** | 819 | LLM-agent decision rules | search current APIs before writing polling/DOM loops/sleeps/custom interception; the built-in ladder `Locator → assertion/expect_event → routing → storage state → Clock → APIRequestContext → trace/HAR` |
| **51** | 842 | Testing matrix | 22-item checklist: engines, headless/headed, clean vs reused auth, popup, iframe, shadow DOM, download, upload, dialog, redirect, failure-vs-HTTP-error, route/fulfill/abort, service worker, WebSocket, timeout, artifacts, locale/tz, CI container |
| **52** | 869 | Anti-pattern inventory | 11 items — `time.sleep()`, unstable generated-class selectors, stored handles, shared context across identities, Playwright for every fetch, browser-per-URL, ignoring service workers, disabled sandbox, storage state in VCS, unclosed contexts, 404-as-`requestfailed` |

### §1.3 `lxml_python_advanced_reference_6.1.1.md` — section index (§0-§56)

Front matter: title `1`, `## Version / source anchors` `3`. Numbered sections start at line `21`. Flat.

| § | Line | Title | Key content |
|---|------|-------|-------------|
| **0** | 21 | Capability map | the full facility list — XML/HTML parsing, ElementTree+ API, namespaces, XPath 1.0, XSLT 1.0, DTD/RelaxNG/XSD/Schematron, C14N, XInclude, SAX, pull/iterparse, targets, resolvers, extension functions, custom classes, objectify, HTML helpers, builders, cssselect |
| **1** | 52 | Installation and binary dependencies | wheels vs source builds; `LXML_VERSION`/`LIBXML_VERSION`/`LIBXSLT_VERSION`; **behavior depends on the native library version** |
| **2** | 75 | Core modules | `etree`, `html`, `objectify`, `builder`, `sax`, `isoschematron`, `cssselect`, `doctestcompare`; **`lxml.html.clean` is no longer bundled** — separate `lxml_html_clean` project |
| **3** | 92 | XML tree construction | `Element`/`SubElement`/`ElementTree`; `getparent`/`getnext`/`getprevious`/`getroottree` |
| **4** | 113 | Element vs ElementTree semantics | operation root vs document; absolute-XPath and serialization differences; be explicit for complex transforms |
| **5** | 123 | XML parsing | `XMLParser`, `fromstring`, `parse`; the nine option families to choose explicitly |
| **6** | 144 | **Security: XML parser defaults** | XXE, DTD retrieval, network access, entity expansion, huge/deep trees, XInclude; **6.1 default is `resolve_entities="internal"`**; the recommended untrusted posture `resolve_entities=False, load_dtd=False, no_network=True, huge_tree=False` |
| **7** | 174 | HTML parsing | `html.fromstring`, `etree.HTMLParser`; repair means serialization ≠ source; selectolax may be better for CSS-only extraction |
| **8** | 190 | Parser recovery | `recover=True` trades correctness guarantees for resilience; never in a validation pipeline |
| **9** | 198 | Parser targets | custom event-receiving targets that build compact results instead of a DOM |
| **10** | 211 | Feed parser | incremental `.feed()`/`.close()`; close on EOF, propagate errors, bound input, never mix documents |
| **11** | 225 | iterparse | the primary large-XML tool; the canonical `elem.clear()` + delete-previous-siblings memory discipline |
| **12** | 245 | XMLPullParser | application-controlled incremental parsing for network streams and non-blocking input |
| **13** | 256 | Iteration and axes | `iter`/`iterchildren`/`iterdescendants`/`iterancestors`/`itersiblings` — cheaper than compiling XPath for simple relationships |
| **14** | 269 | Namespaces | Clark notation `{uri}tag`; `namespaces=` prefix maps; **the document's prefix is not the namespace identity** |
| **15** | 288 | QName | construct/decompose qualified names instead of string-building `{uri}local` |
| **16** | 296 | XPath | node sets, booleans, numbers, strings, variables, namespaces, extension functions; **the main reason to choose lxml over selectolax** |
| **17** | 315 | Compiled XPath | `etree.XPath(...)` compiled once, reused across documents with variable binding |
| **18** | 327 | XPathEvaluator | evaluator objects for shared context/namespace config; prefer compiled `XPath` unless state helps |
| **19** | 335 | EXSLT and regex | availability depends on native library support — treat as a runtime capability |
| **20** | 343 | XSLT | `etree.XSLT(tree)`; compile once and reuse; parameters, extension functions, external resources |
| **21** | 361 | XSLT security | untrusted XSLT is code-like; control file I/O, network, resolvers, extension functions, resource limits; isolate |
| **22** | 376 | XSLT parameters | `XSLT.strparam()` for literal strings vs XPath expressions — avoids quoting/injection errors |
| **23** | 384 | Validation overview | DTD, RelaxNG, XMLSchema, Schematron; the `validate`/`error_log`/`assertValid` pattern; compile validators once |
| **24** | 406 | DTD | during-parse or `etree.DTD`; validation and default attributes vs entity/resource security cost |
| **25** | 416 | Relax NG | `etree.RelaxNG`; includes/imports resolve resources — configure resolver/network policy |
| **26** | 429 | XML Schema | `etree.XMLSchema`; `error_log` diagnostics; well-formed ≠ schema-valid |
| **27** | 442 | ISO Schematron | `lxml.isoschematron.Schematron` via the reference XSLT skeleton flow; richer than the legacy facility |
| **28** | 456 | Legacy `etree.Schematron` caution | depends on native libxml2 features and is disappearing; prefer `isoschematron` |
| **29** | 464 | Error logs | do not collapse to `str(exc)` — persist domain/type, line/column, level, message, filename |
| **30** | 479 | Serialization | `tostring` options — method, encoding, declaration, pretty print, tails, doctype; **pretty-printing is not canonicalization** |
| **31** | 502 | Canonical XML (C14N) | canonicalization/signature workflows; know which C14N version and options the protocol needs |
| **32** | 515 | CDATA | construction; parsed semantics usually do not distinguish CDATA from text — only serialization does |
| **33** | 523 | Comments and processing instructions | preserved or removed by parser settings; matters for signatures and document preservation |
| **34** | 531 | XInclude | loads external resources — same resolution policy as DTD/XSLT/schema imports |
| **35** | 539 | Custom resolvers | offline catalogs, controlled includes, deterministic tests, blocking network, remapping identifiers — better than monkeypatching |
| **36** | 554 | XML catalogs | libxml2 catalog facilities for offline resolution in standards-heavy systems |
| **37** | 562 | Extension XPath/XSLT functions | Python callables in XPath/XSLT; couples stylesheets to Python, hurts portability and performance — use sparingly |
| **38** | 575 | Custom element classes | class-lookup schemes for tag/namespace-specific behavior; avoid ORM-like abstractions |
| **39** | 585 | objectify | data-binding-style access; caution with lexical distinctions, complex namespaces, exact manipulation |
| **40** | 599 | `lxml.html` | document/fragment parsing, link iteration/rewriting, forms, helpers, builders, diff; **sanitization is not included** |
| **41** | 615 | Link handling | known link-bearing attributes; 6.1.1 fixed the `xlink:href` omission — link extraction must account for namespaces and embedded vocabularies, not just `<a href>` |
| **42** | 625 | CSS selectors | `lxml.cssselect` translates CSS→XPath via the separate `cssselect` package; selectolax may be faster for CSS-only HTML |
| **43** | 633 | Builder APIs | `lxml.builder.E` and HTML builders — correct escaping and namespace construction without string concatenation |
| **44** | 643 | SAX interoperability | bridging to SAX-style APIs; otherwise native etree/pull parsing is simpler |
| **45** | 651 | Threading | GIL release in native operations; keep parser/tree work per-worker; never mutate one tree concurrently; measure first |
| **46** | 664 | Memory management | native + Python memory for retained trees; iterparse, clear, keep primitives, release references; for HTML crawls parse→extract→discard |
| **47** | 681 | Performance | compile XPath/XSLT/validators once; stream large XML; avoid cross-document node moves and Python callbacks in hot loops |
| **48** | 693 | Node movement / copying | moving between documents incurs native bookkeeping; be explicit about move vs duplicate |
| **49** | 701 | Unicode / bytes | **pass bytes when encoding declarations must be honored**; a decoded `str` makes the XML declaration inconsistent or irrelevant |
| **50** | 709 | lxml vs stdlib ElementTree | choose lxml for axes, XPath, XSLT, validation, C14N, resolvers, native performance; stdlib for minimal dependencies and smaller surface |
| **51** | 727 | lxml vs selectolax | the explicit division: selectolax is HTML/CSS/speed; lxml is XML+HTML, XPath, validation, transformation, namespaces, C14N, resolution |
| **52** | 748 | Security floor: 6.1.1 | `xlink:href` recognition fix + patched libxslt in Linux wheels; do not downgrade below it for untrusted markup |
| **53** | 758 | 7.0 prerelease posture | alphas exist; do not design against them; re-test parser defaults, native compatibility, PyPy/free-threading, removed APIs |
| **54** | 770 | Testing matrix | 20-item checklist: declarations, malformed rejection, recovery, entity/DTD policy, namespaces, XPath variables, compiled reuse, XSLT params and resource denial, four validators, C14N bytes, XInclude, iterparse, resolvers, HTML repair, SVG/MathML links |
| **55** | 798 | LLM-agent execution playbook | *(named differently from the other five docs)* the built-in ladder `parse → iterparse/pull → XPath/compiled → XSLT → validator → C14N → resolver/catalog → lxml.html helper → custom target/class/function` |
| **56** | 818 | Anti-pattern inventory | 11 items — entity resolution on untrusted XML, network on by default, regex XML, hand-written XPath logic, per-record recompilation, DOM where iterparse fits, pretty-print as canonicalization, untrusted XSLT as data, `<a href>`-only link assumptions, `lxml.html.clean` as bundled sanitizer, ignoring native library versions |

### §1.4 `selectolax_python_advanced_reference_0.4.11.md` — section index (§0-§40)

Front matter: title `1`, `## Version / source anchors` `3`. Numbered sections start at line `21`. Flat.

| § | Line | Title | Key content |
|---|------|-------|-------------|
| **0** | 21 | What selectolax is | HTML5 parsing, CSS selectors, text extraction, navigation, mutation, fragments, serialization — and explicitly **not** an XML schema system, XPath engine, JS runtime, or browser |
| **1** | 43 | Backend choice: Lexbor vs Modest | `selectolax.lexbor.LexborHTMLParser` vs `selectolax.parser.HTMLParser`; **Lexbor for new code**; node classes are not interchangeable |
| **2** | 56 | Installation | `pip install selectolax`; `[cython]` extra for non-wheel builds; 0.4.11 ships free-threaded wheels |
| **3** | 72 | LexborHTMLParser construction | constructor with `is_fragment`/`fragment_tag`/`fragment_namespace`; use explicit fragment mode for fragments |
| **4** | 95 | Root/head/body | `root`/`head`/`body`/`html`/`inner_html`/`raw_html`; **parsed serialization ≠ input bytes** |
| **5** | 114 | CSS selection | `css`/`css_first(default=)`/`css_matches`/`any_css_matches`; modern selector support plus `:lexbor-contains(...)` |
| **6** | 131 | Selector strictness | `css_first(..., strict=True)` for uniqueness contracts vs non-strict first-match for speed; never silently first-match identity-critical fields |
| **7** | 148 | Advanced selector helper | `select()` returns a chainable selector object — use it instead of hand-rolled loops over all nodes |
| **8** | 156 | **LexborNode lifetime** | nodes belong to a parser; clones stay tied to parser-owned memory; **never cache nodes beyond the parser** — extract primitives |
| **9** | 166 | Node navigation | parent, first/last child, siblings, traversal, parser reference; cheaper than re-querying CSS |
| **10** | 182 | Node identity/type | element/text/comment/document indicators — use them instead of inferring from serialization |
| **11** | 194 | Tag and ID properties | tag name, `id`, attributes; test attribute case/serialization on target documents |
| **12** | 205 | **Attributes: copy vs live mapping** | `node.attributes` is a dict **snapshot**; `node.attrs` is a **live mutating** interface; empty HTML attributes may be `None` |
| **13** | 223 | Text extraction | `node.text(deep, separator, strip, skip_empty)`; each combination changes token boundaries — it is a policy decision |
| **14** | 244 | merge_text_nodes | merges adjacent text nodes after unwrapping, before separator-based extraction; the canonical unwrap→merge→text pipeline |
| **15** | 260 | Raw text-node value | `raw_value` exposes original bytes for text nodes when entity spelling matters |
| **16** | 268 | Comment content | `comment_content`; check node type first |
| **17** | 276 | Script helpers | `scripts_contain(query)` / `script_srcs_contain(queries)`, cached on first use; fine for tech fingerprinting, **not a security verdict** |
| **18** | 288 | Tag lookup | `tags(name)` for efficient direct lookup when selector expressiveness is unnecessary |
| **19** | 296 | Fragment parsing | empty fragments, fragment tag/namespace, multiple root-level nodes; for snippets, API-field HTML, templates, mutation payloads |
| **20** | 312 | Node creation | `tree.create_node("span")` — parser-owned creation instead of raw HTML concatenation |
| **21** | 324 | Insertion | insert before/after/as-child; **string input is treated as text and escaped** — pass a Node for real markup. A safety property; do not defeat it |
| **22** | 339 | replace_with | text or node replacement; understand ownership/movement; clone for independent identity |
| **23** | 349 | Decompose / remove | `decompose()` (optionally recursive), `remove()` alias; historical double-removal crashes are fixed but keep mutation state coherent |
| **24** | 357 | strip_tags | `strip_tags(tags, recursive=False)`; decide whether children survive; do not strip broadly before metadata extraction |
| **25** | 369 | unwrap / unwrap_tags | removes wrappers, preserves descendants; follow with `merge_text_nodes()` |
| **26** | 379 | inner_html | readable and settable; setting **reparses and replaces children** — not a textual splice |
| **27** | 389 | Serialization | `.html`, `.inner_html`, `html_pretty()`, `inner_html_pretty()`; the pretty option surface; pretty modes are diagnostics unless formatting is contractual |
| **28** | 412 | HTML5 repair semantics | implied structure, repaired nesting, normalized attributes, implied nodes — `input bytes != parsed serialization`; keep original bytes if fidelity matters |
| **29** | 430 | Encoding | prefer bytes + HTTP charset metadata; **decoding decided before passing `str` cannot be undone**; the four-step crawler policy |
| **30** | 442 | Modest backend | still available; do not select it from habit; verify method availability and behavior per backend; standardize on one |
| **31** | 452 | Free-threading and concurrency | free-threaded CPython support; keep parser/node instances job-local; parse independent documents in independent tasks, never mutate one tree from many workers |
| **32** | 464 | Performance | parse once, query the tree, non-strict first-match where valid, tag/script caches, extract primitives then release, no reparse cycles |
| **33** | 478 | Memory and large documents | bound downloaded bytes, pages in flight, retained trees, extracted text — a fast parser is still an exhaustion vector |
| **34** | 492 | When to use lxml instead | XPath, XML namespaces, DTD/XSD/RelaxNG/Schematron, XSLT, C14N, streaming XML, custom resolvers, rich XML semantics |
| **35** | 508 | When to use Playwright instead | JS execution, hydrated DOM, browser storage/cookies, interaction, browser network/WebSocket; and the canonical `Playwright → page.content() → selectolax` pattern |
| **36** | 525 | Error handling | `SelectolaxError`; do not rely on historical segfault behavior as a contract; validate selectors in tests before large crawls |
| **37** | 535 | 0.4.11-specific changes | CSS-selector NULL checks, html5test serialization fix, Lexbor update; recent 0.4.x churn is itself a reason to pin |
| **38** | 555 | Testing strategy | 16-item fixture list: valid/malformed HTML, fragments, duplicates, unicode selectors, empty attributes, comments, entities/`raw_value`, script/style, unwrap+merge, double decompose, `inner_html` set, multi-root, large-but-bounded, selector syntax errors, serialization snapshots |
| **39** | 580 | LLM-agent decision rules | check for a built-in before writing loops — CSS query, first-match, matching, selector helper, tag lookup, text extraction, script helper, unwrap/strip, text merge, insert/replace, fragment parser, pretty serialization |
| **40** | 601 | Anti-pattern inventory | 10 items — regex HTML, per-field reparse, Modest by habit, serialization-equals-bytes, nodes past parser lifetime, live `.attrs` for read-only, stripping before metadata extraction, careless `separator=" "`, unlimited HTML size, selectolax where XPath/validation is required |

### §1.5 `tldextract_python_advanced_reference_5.3.2.md` — section index (§0-§43)

Front matter: title `1`, `## Version / source anchors` `3`. Numbered sections start at line `20`. Flat except `§35`, which has three `##` subsections.

| § | Line | Title | Key content |
|---|------|-------|-------------|
| **0** | 20 | Purpose | the one question it answers — where is the public/registry suffix boundary; and what it is **not**: URL validator, DNS resolver, IDNA canonicalizer, allowlist engine |
| **1** | 45 | Why naive dot splitting is wrong | multi-label suffixes, wildcards, exceptions; `example.co.uk` vs `example.org.kg` |
| **2** | 61 | Installation | pure Python; bundled PSL snapshot plus optional remote update |
| **3** | 71 | One-shot API | module-level `extract` uses a default extractor; instantiate `TLDExtract` for production policy control |
| **4** | 83 | TLDExtract constructor | `cache_dir`, `suffix_list_urls`, `fallback_to_snapshot`, `include_psl_private_domains`, `extra_suffixes`, `cache_fetch_timeout` — these determine data provenance and therefore results |
| **5** | 102 | Explicit production extractor | the two reproducible shapes: `suffix_list_urls=()` + snapshot, or an app-owned `file://` PSL with `fallback_to_snapshot=False` |
| **6** | 127 | ExtractResult fields | `subdomain`, `domain`, `suffix`, `is_private`, `registry_suffix` |
| **7** | 141 | `suffix` | the **effective** public suffix under the current private-domain policy — Blogspot-style hosts yield `com` or `blogspot.com` depending on it |
| **8** | 151 | `registry_suffix` | the registry-controlled suffix, **unaffected** by private-domain inclusion — the stable registrar-level boundary |
| **9** | 159 | `is_private` | whether the matched effective suffix came from the PSL private section |
| **10** | 167 | top_domain_under_public_suffix | domain + effective public suffix; **the preferred replacement** for `registered_domain` |
| **11** | 179 | registered_domain deprecation | deprecated, slated for removal next major; do not introduce new code using it |
| **12** | 193 | top_domain_under_registry_suffix | stable with respect to private-domain semantics; for tenant isolation vs billing/ownership boundaries |
| **13** | 208 | fqdn | rejoins a valid suffix-bearing decomposition; **not a canonical DNS name** |
| **14** | 216 | reverse_domain_name | reverse-DN notation; 5.3.2 fixed stray dots on empty components; not a canonical identity |
| **15** | 226 | ExtractResult is a dataclass | since v5 — **no `r[1]`, no tuple unpacking**; use named fields |
| **16** | 242 | extract_str | `extractor.extract_str(url, include_psl_private_domains=None, session=None)`; the optional `requests.Session` is for PSL fetching, not application HTTP |
| **17** | 256 | extract_urllib | takes `urlsplit` output; **an instance method, not module-level** — 5.3.2 corrected the docs on this |
| **18** | 273 | URL validation boundary | lenient by design; extraction success implies nothing about validity, scheme safety, DNS validity, reachability, or publicness |
| **19** | 294 | Unknown suffix behavior | tldextract does **not** apply the PSL implicit wildcard fallback — unknown final labels leave `suffix` empty **by design**, so callers can detect internal/invalid names. Cover it in tests |
| **20** | 306 | IP addresses | IP-like hosts land in `domain` with empty `suffix`; use `ipaddress` for real classification |
| **21** | 314 | localhost / internal names | populated `domain`, empty `suffix` — this shape means "no PSL suffix matched", **not** "safe internal host" |
| **22** | 324 | Public vs private PSL sections | `com`/`co.uk` vs `blogspot.com`/`github.io`; the choice changes tenant and security semantics — document it |
| **23** | 345 | include_psl_private_domains | constructor default false; per-call override where supported; **switching policy mid-product makes stored domain facts incomparable** |
| **24** | 361 | PSL data sources | `suffix_list_urls` is ordered and **first-success wins** — it does not merge lists; `file://` supported |
| **25** | 371 | Bundled snapshot | resilient offline behavior at the cost of PSL freshness; choose reproducibility vs freshness vs availability explicitly |
| **26** | 385 | Cache | disk cache by default; `cache_dir` or `TLDEXTRACT_CACHE`; `cache_dir=None` disables; decide baked vs ephemeral vs mounted in containers |
| **27** | 401 | Cache update | `extractor.update(fetch_now=True)`; `tldextract --update`; refresh after package upgrades |
| **28** | 419 | Cache fetch timeout | `cache_fetch_timeout` / `TLDEXTRACT_CACHE_TIMEOUT`; never let a parse block on a remote update in a latency-sensitive path — warm outside the critical path |
| **29** | 429 | Custom requests.Session | for proxy, custom CA, internal mirror auth, network policy; **for suffix-list acquisition only** |
| **30** | 443 | extra_suffixes | private namespace rules that act like suffixes; they alter output as materially as a PSL update — document as configuration |
| **31** | 457 | tlds property | the effective suffix list under current policy; for diagnostics, tests, cache-warmup confirmation — do not reimplement lookup from it |
| **32** | 470 | **IDNA / Punycode** | 5.3.2 documents that tldextract does **not** validate or canonicalize hostname representation; U-labels, A-labels, and case may all differ. The four-step caller responsibility: validate → normalize IDNA → normalize case → compare/store |
| **33** | 489 | Unicode dots | internationalized separators are accounted for in parsing, but canonicalization remains the caller's job |
| **34** | 497 | Security uses and non-uses | appropriate as *one input* to a registrable-boundary policy; **insufficient alone** for SSRF, phishing, allowlists, cookie security, same-site, ownership trust |
| **35** | 520 | Reproducibility strategies | `## 522` frozen bundled snapshot · `## 530` application-owned PSL file (pin the hash, use `file://`) · `## 536` auto-updating PSL (freshness over reproducibility — persist version/hash with facts) |
| **36** | 544 | Concurrency | cache locking exists, but avoid many workers doing first-use remote fetches; warm the cache and reuse an extractor per worker |
| **37** | 554 | Performance | trie-based lookup, fast after init; reuse the extractor, warm the cache, no network on the hot path, prefer `extract_urllib` when `urlsplit` already ran |
| **38** | 566 | CLI | `tldextract <url>`, `--update`, `--help`; diagnostics only — use the typed API for machine integration |
| **39** | 580 | Data-model recommendation | the twelve fields to store separately (raw/normalized URL, raw/canonical hostname, subdomain, domain, public suffix, registry suffix, `is_private`, both top-domain forms, PSL policy, PSL snapshot/version/hash); **do not collapse into one `domain` field** |
| **40** | 604 | 5.3.x migration points | 5.3.0 new fields + deprecation · 5.3.1 Python 3.9 dropped, 3.14 added · 5.3.2 `reverse_domain_name` fix, `extract_urllib` doc correction, cache-dir docs, hostname-representation limits, wildcard clarification |
| **41** | 625 | Testing matrix | 22-item checklist: basic and multi-label suffixes, nesting, wildcard and exception rules, private included/excluded, registry-suffix stability, unknown suffix, localhost, IPv4/IPv6, unicode/punycode/uppercase, trailing dot, extra suffixes, local PSL file, no-network snapshot, fetch failure fallback, cache update, empty-component reverse DN |
| **42** | 654 | LLM-agent decision rules | the four-step order — parse URL, normalize hostname, call tldextract, choose public vs registry boundary; never `host.split(".")[-2:]` |
| **43** | 670 | Anti-pattern inventory | 9 items — `suffix` as TLD, `domain` as registrable domain, deprecated `registered_domain`, remote fetch on a latency path, unicode/punycode comparison without normalization, tldextract as URL validation, as SSRF protection, unversioned policy changes, missing PSL provenance |

### §1.6 `extruct_python_advanced_reference_0.18.0.md` — section index (§0-§30)

Front matter: title `1`, `## Version / source anchors` `3`. Numbered sections start at line `18`. Flat. The smallest doc in the pack.

| § | Line | Title | Key content |
|---|------|-------|-------------|
| **0** | 18 | Purpose | the six supported metadata families; use **after** acquisition — it is not an HTTP client or browser |
| **1** | 35 | Installation | `pip install extruct`; `[cli]` extra pulls acquisition dependencies — keep acquisition and extraction separate in application code |
| **2** | 51 | Primary API | `extruct.extract(html, base_url=final_url)`; returns a dict keyed by syntax, each in that syntax's own representation |
| **3** | 68 | Input forms | HTML text/bytes, or an already-parsed lxml tree; **selectolax trees are not accepted** — keep the bytes or build an lxml tree |
| **4** | 80 | `base_url` | always supply it when relative URLs may appear; **use the final post-redirect URL** unless the contract intentionally says otherwise |
| **5** | 94 | Syntax selection | `json-ld`, `microdata`, `opengraph`, `microformat`, `rdfa`, Dublin Core; select only what is needed — do not parse every syntax at volume |
| **6** | 110 | JSON-LD | locates and parses embedded script data; historical robustness (null JSON-LD, control characters, script/comment wrappers, JS-style comments); **parsing is not trust** |
| **7** | 124 | JSON-LD shape variability | single object, array, `@graph`, multiple scripts, duplicate entities, malformed next to valid — normalize downstream; extruct does not reconcile ontology |
| **8** | 138 | Microdata | `itemscope`/`itemtype`/`itemprop`/`itemid`/`itemref` and value extraction by element type; 0.18.0 adds `valueRequired`/`valueName`; do not hand-walk the DOM |
| **9** | 154 | Microdata `return_html_node` | returns the source node for provenance; only when node lifetime/tree ownership is controlled — never serialize parser nodes into an application API |
| **10** | 166 | Open Graph | core `og:*`, namespaces, type-specific metadata, duplicates, optional array behavior; **OG on the public web is inconsistent** — preserve duplicates and order if you need them |
| **11** | 179 | `with_og_array` | array-oriented OG extraction for repeated properties (images); test the exact shape downstream expects |
| **12** | 189 | Microformats | via `mf2py`; a distinct vocabulary — do not force it into schema.org without an explicit mapping layer |
| **13** | 197 | RDFa | built on RDF tooling and **described as experimental**; more dependencies, more CPU/memory, rdflib version coupling — enable only with a real consumer |
| **14** | 211 | Dublin Core | DC-HTML-2003-style metadata; meaningful on publishing/library/document sites, rare on commercial ones |
| **15** | 221 | `uniform` output | maps several syntaxes toward a common template — convenience only; for high fidelity keep syntax-specific output, normalize separately, retain provenance |
| **16** | 234 | Error policy | `strict` / `log` / `ignore` — a deliberate choice. A crawler usually wants one malformed syntax not to discard the others, with parse errors observable |
| **17** | 252 | Individual extractor classes | per-syntax extractors centered on `extract_items()` over an lxml document; for single-syntax use or custom orchestration. Prefer top-level `extract()` otherwise |
| **18** | 265 | Parsed-tree reuse | since 0.15 — `HTML bytes → parse lxml once → XPath extraction + extruct extraction`; saves a pass and makes both stages see the same repaired tree |
| **19** | 282 | selectolax coexistence | run selectolax for DOM/content and extruct separately from the original HTML; do not serialize a selectolax tree and re-parse it into lxml |
| **20** | 296 | Playwright coexistence | `extruct.extract(page.content(), base_url=page.url)`; JS can inject or modify JSON-LD, and `page.content()` is post-DOM serialization — choose server-source vs rendered deliberately |
| **21** | 312 | URL resolution | `@id`, images, canonicals, OG URLs, relative resources; preserve raw value, resolved value, and source page URL when provenance matters |
| **22** | 329 | Duplicate entities | never dedupe by naive JSON equality — consider `@id`, canonical URL, type, normalized identifiers, source syntax, block order. **Entity resolution belongs outside extruct** |
| **23** | 345 | Schema.org normalization | extruct does not validate against schema.org constraints; validate separately, tolerate extensions, version your normalization rules |
| **24** | 356 | Trust boundary | embedded metadata is attacker-controlled: huge blobs, deep nesting, odd Unicode, URLs to private resources, misleading types, HTML/JS strings, RDF expansion. **Never execute values from metadata** |
| **25** | 373 | Resource limits | bound HTML bytes, render time, block count, JSON-LD block size, extracted object count, normalization depth — extruct is not your only anti-exhaustion boundary |
| **26** | 387 | CLI | fetches and emits metadata; good for inspection, experiments, fixtures; production should separate acquisition from extraction |
| **27** | 400 | Version posture | 0.18.0 is old relative to the rest of the pack — pin it, run a metadata corpus on upgrades, watch rdflib/mf2py/lxml compatibility |
| **28** | 412 | Testing corpus | 18-item fixture list: JSON-LD object/array/`@graph`/multi-script/malformed/null, Microdata nesting and `itemref`, action `valueRequired`/`valueName`, duplicate OG images, OG namespaces, Microformats, RDFa, Dublin Core, relative URLs + base, `<base>` element, cross-syntax duplicate entity, large-but-bounded block, Playwright-rendered DOM |
| **29** | 440 | LLM-agent decision rules | use extruct instead of hand-writing JSON-LD regex, Microdata walkers, OG namespace parsers, RDFa or Microformats parsers; choose syntaxes explicitly; keep the `extraction → raw records → validation → entity resolution → canonical model` chain separate |
| **30** | 462 | Anti-pattern inventory | 10 items — `<script type="application/ld+json">` regex, omitted `base_url`, treating syntax outputs as identical, RDFa with no consumer, discarding valid syntaxes on one malformed block, metadata as authoritative truth, fetching metadata URLs without SSRF policy, redundant reparsing, conflating rendered and source metadata, deduping without provenance |

---

## §2 — Cross-document authority matrix

Legend: **✅ authoritative** · **🔁 cross-cut or summary treatment** · **—** not covered. Aliases per §1. When two documents cover a topic, the **Authoritative** column names the winner and why. The pack's own cross-links (`extruct §19`/`§20`, `slax §34`/`§35`, `lxml §51`, `pw §47`/`§48`) already agree with these calls — this section formalizes them and fills the gaps they leave.

### Acquisition

| Topic | httpx2 | pw | Authoritative |
|---|---|---|---|
| Fetching bytes over HTTP | §0-§14 ✅ | §29 🔁 | **httpx2** — `pw §29` itself says HTTPX2 is the better default for general high-throughput acquisition outside browser workflows |
| Executing JavaScript / hydrated DOM | — | §0-§19 ✅ | **pw** — the only doc that covers a JS runtime |
| Deciding between the two | §34 🔁 | §47 ✅ | **pw §47** — the fuller escalation list; `httpx2 §34` states the same rule from the other side (§3 tree 1) |
| Cookies and session identity | §8 ✅ (client cookie jar) | §6, §31 ✅ (browser context + storage state) | **Both, per layer** — they are different state stores and do not interchange |
| Redirects and final URL | §10 ✅ | §8, §25 🔁 | **httpx2 §10** — chain, `history`, `next_request`, and the rule to revalidate every hop |
| WebSockets | §22 ✅ (client sessions) | §28 ✅ (in-page sockets + routing) | **Both, different objects** — httpx2 for a client you own, pw for sockets the page opens |

### Parsing and extraction

| Topic | slax | lxml | extruct | pw | Authoritative |
|---|---|---|---|---|---|
| HTML parsing + CSS selection | §3-§7 ✅ | §7, §42 🔁 | — | §9-§11 🔁 (in-browser) | **slax** — `lxml §51` and `lxml §42` both defer for CSS-only HTML work |
| XPath | — (§0: explicitly not an XPath engine) | §16-§18 ✅ | — | §10 🔁 (escape hatch) | **lxml §16/§17** — the only real XPath engine in the pack |
| XML, namespaces, feeds, sitemaps | — (§34) | §5, §11, §14, §15 ✅ | — | — | **lxml** — sole owner |
| Schema validation (DTD/RelaxNG/XSD/Schematron) and C14N | — | §23-§28, §31 ✅ | — | — | **lxml** — sole owner |
| Streaming a large document | §33 🔁 (bounds only) | §10-§12 ✅ | — | — | **lxml §11** `iterparse` — the only incremental parser here |
| Text extraction from markup | §13, §14 ✅ | §40 🔁 | — | §19 🔁 | **slax §13/§14** — the explicit policy surface (`deep`/`separator`/`strip`/`skip_empty`, then `merge_text_nodes`) |
| DOM mutation and serialization | §20-§27 ✅ | §30-§33, §43, §48 ✅ | — | — | **Both, per format** — slax for HTML trees, lxml for XML/C14N/builders |
| Embedded structured metadata | §17 🔁 (script substring helpers) | §40, §41 🔁 (link/HTML helpers) | §5-§15 ✅ | — | **extruct** — `slax §17` is fingerprinting, not parsing; `lxml.html` has no metadata vocabulary |
| Reusing one parse across stages | §32 🔁 | §7 🔁 | §18 ✅ | — | **extruct §18** — states the `bytes → one lxml parse → XPath + extruct` pattern; note extruct **cannot** consume a selectolax tree (`extruct §3`, `§19`) |

### URLs, hosts, identity

| Topic | httpx2 | tldx | extruct | Authoritative |
|---|---|---|---|---|
| URL parsing and manipulation | §5, §6 ✅ | §17 🔁 (`extract_urllib`) | — | **httpx2 §5** — the structured `URL` type; `tldx` consumes a parsed URL, it does not model one |
| Registrable domain / public suffix boundary | — | §6-§12 ✅ | — | **tldx** — sole owner |
| Public vs registry suffix, private PSL policy | — | §7, §8, §22, §23 ✅ | — | **tldx §8** — `registry_suffix` is the boundary that survives a policy change |
| Resolving relative URLs found in content | §5 🔁 | — | §4, §21 ✅ | **extruct §4/§21** — `base_url` semantics and the raw/resolved/source triple |
| **IDNA / hostname canonicalization** | **§5** ✅ *(stronger than the doc states)* | §32, §33 — **explicitly disclaimed** | — | **httpx2 §5**, verified by probe: `httpx2/_urls.py` imports `idna` and normalizes `URL.raw_host` to a lowercased A-label using **IDNA 2008 / UTS-46 nontransitional** (`faß.de` → `xn--fa-hia.de`). Use `str(URL(u).raw_host,'ascii')` as canonical identity and feed that to tldextract. **Never** stdlib `encodings.idna` — it returns `fass.de` (IDNA 2003), a different registrable domain. Caveat: pure-ASCII hosts bypass IDNA validation entirely, so call `idna.encode(h, uts46=True)` when label validity is contractual |
| What a successful parse does **not** prove | §5, §31 ✅ | §18, §34 ✅ | — | **Both** — `httpx2 §5` (parse ≠ safety) and `tldx §18` (extraction ≠ validity) state the same boundary at different layers |

### Data handling

| Topic | httpx2 | slax | lxml | extruct | Authoritative |
|---|---|---|---|---|---|
| Bytes vs `str`, encoding decisions | §9, §26 ✅ | §29 ✅ | §49 ✅ | §3 🔁 | **All three, per layer** — httpx2 owns the wire and decompression, slax owns HTML charset policy, lxml owns XML declarations. The shared rule: decide once, at the earliest boundary (§3 tree 9) |
| Serialization ≠ source bytes | — | §4, §28 ✅ | §7 🔁 | — | **slax §28** — the fullest statement of HTML5 repair semantics; `lxml §7` says the same in one line |
| Bounding input size | §11, §31 ✅ | §33 ✅ | §46 ✅ | §25 ✅ | **All four, per layer** — bound at acquisition first (`httpx2 §31`); parser-level bounds are the second line, not the first |

### Reliability, testing, security

| Topic | httpx2 | pw | slax | lxml | extruct | tldx | Authoritative |
|---|---|---|---|---|---|---|---|
| Timeouts and waiting | §12 ✅ (four-component budget) | §13 ✅ (actionability auto-wait) | — | — | — | §28 🔁 (PSL fetch only) | **Both, different models** — httpx2 budgets network phases; pw waits on DOM state. Neither substitutes for the other |
| Retries | §25 ✅ | — | — | — | — | — | **httpx2 §25** — and its first statement is that no universal retry switch exists |
| Failure classification | §24 ✅ | §25 🔁 | §36 🔁 | §29 ✅ (error logs) | §16 🔁 (error policy) | — | **httpx2 §24** for network; **lxml §29** for structured parse diagnostics; `pw §25` for the 404-is-not-a-failure distinction |
| Test doubles / fixtures | §20, §28, §29 ✅ (transport layer) | §27 ✅ (HAR, browser layer) | — | — | §28 🔁 (corpus) | — | **Both, different layers** — `MockTransport` intercepts below the client; HAR replays below the browser (§3 tree 8) |
| Concurrency model | §3 ✅ (async, pools) | §44 ✅ (workers, contexts) | §31 ✅ (free-threading) | §45 ✅ (GIL, per-worker trees) | — | §36 ✅ (cache locking) | **Each for its own library** — no shared model; the common rule is that parser/tree/context state is job-local |
| Untrusted-input security | §31 ✅ | §46 ✅ | §33 🔁 | §6, §21 ✅ | §24, §25 ✅ | §34 ✅ | **Each for its own layer** — and all five say the same thing: **none of them is a security boundary by itself.** `httpx2 §31` "not an SSRF firewall"; `pw §46` "not a sandbox"; `tldx §34` "insufficient alone" |
| Anti-pattern inventory | §36 | §52 | §40 | §56 | §30 | §43 | **All six** — always the last section (SKILL §Navigation) |

---

## §3 — Decision trees

Ten trees. Citations name the section that settles the branch.

### 1. Acquisition: HTTP or browser?

```text
Does the data exist in the server-rendered HTML or a public API response?
  -> yes: use httpx2                                              (httpx2 §34; pw §47)
Is JavaScript execution required to produce the content?
  -> yes: Playwright                                              (pw §47)
Does it need browser cookies / localStorage / session state?
  -> yes: Playwright                                              (pw §6, §31)
Does it require interaction (click, scroll, form) to reveal data?
  -> yes: Playwright                                              (pw §14, §47)
Do you need to observe or intercept the browser's own requests?
  -> yes: Playwright                                              (pw §25, §26)
Is the in-page WebSocket traffic the payload?
  -> yes: Playwright                                              (pw §28)
Otherwise
  -> httpx2                                                       (httpx2 §34)
Escalation is one-way and expensive. Before keeping a source on the browser
path, replay the observed endpoint from a clean httpx2 client and compare.  (pw §47)
```

### 2. Which parser?

```text
Is the payload XML (sitemap, RSS/Atom, namespaced feed)?
  -> lxml                                                         (lxml §5, §14; slax §34)
Do you need XPath, XSLT, DTD/RelaxNG/XSD/Schematron, or C14N?
  -> lxml                                                         (lxml §16, §20, §23, §31)
Is the document too large to hold as a tree?
  -> lxml iterparse / XMLPullParser                               (lxml §11, §12)
Is it ordinary HTML and the selectors are CSS?
  -> selectolax (Lexbor)                                          (slax §1, §5; lxml §51)
Do you want embedded JSON-LD / Microdata / OG / RDFa / µf / DC?
  -> extruct                                                      (extruct §5)
Is the DOM only reachable inside a live page?
  -> Playwright locators, then hand page.content() off            (pw §48; slax §35)
Do not parse the same HTML with two parsers without a demonstrated reason.
extruct cannot consume a selectolax tree — keep the original bytes.   (extruct §3, §19)
```

### 3. Which extruct syntaxes to enable?

```text
Do you need schema.org entities from job/product/article pages?
  -> json-ld                                                      (extruct §6, §7)
Did you observe Microdata on the actual target pages?
  -> add microdata (0.18.0 handles valueRequired/valueName)       (extruct §8)
Do you need social/preview metadata, possibly repeated?
  -> opengraph (+ with_og_array for repeated properties)          (extruct §10, §11)
Is there a real consumer for RDF graph semantics?
  -> rdfa; otherwise DO NOT enable it                             (extruct §13)
Is the target a publishing/library/document site?
  -> dublin core; rare on commercial sites                        (extruct §14)
Tempted by `uniform` to flatten it all?
  -> only if its information model IS your model; otherwise keep
     syntax-specific output + your own normalization + provenance (extruct §15)
Enabling every syntax at crawl volume is the default mistake. RDFa is the
most expensive: more dependencies, more CPU/memory, rdflib coupling.  (extruct §5, §13)
```

### 4. PSL and domain policy

```text
Must results reproduce years later, byte for byte?
  -> app-owned PSL file via file://, pin the hash,
     fallback_to_snapshot=False                                   (tldx §5, §24, §35)
Need offline resilience with acceptable staleness?
  -> suffix_list_urls=(), fallback_to_snapshot=True               (tldx §5, §25, §35)
Need maximum freshness?
  -> auto-update, and persist PSL version/hash with every fact    (tldx §27, §35)
Which boundary does the application actually mean?
  -> "the site as a tenant sees it"  -> suffix / top_domain_under_public_suffix
                                                                  (tldx §7, §10)
  -> "who registered it"             -> registry_suffix / top_domain_under_registry_suffix
                                                                  (tldx §8, §12)
Are hosted-service suffixes (github.io, blogspot.com) their own sites?
  -> include_psl_private_domains=True                             (tldx §22, §23)
Never change include_psl_private_domains without versioning stored domain
facts — the old and new values are not comparable.                (tldx §23, §43)
```

### 5. Which timeout fired, and what does it mean?

```text
ConnectTimeout   -> DNS/TCP/TLS setup never completed; host or network problem
ReadTimeout      -> connection established, server too slow or stalled mid-body
WriteTimeout     -> could not push the request body fast enough
PoolTimeout      -> no connection slot available — LOCAL saturation, not the
                    remote host's fault; do not blindly retry it     (httpx2 §12, §13)
                 -> raise Limits, or lower your own concurrency
Set the four components separately. A 30s read may be fine while a 30s pool
wait always means you are over-subscribed.                        (httpx2 §12)
```

### 6. Should this be retried?

```text
Is it a connection-establishment failure?
  -> transport-level retries cover this class                     (httpx2 §20, §25)
Is it a TimeoutException / NetworkError on an idempotent method?
  -> application retry layer, with backoff + jitter + total budget (httpx2 §25)
Is it an HTTPStatusError?
  -> only retry the codes you chose; honor Retry-After            (httpx2 §24, §25)
Is the request body a stream or non-rewindable iterator?
  -> it cannot be replayed; do not retry (also breaks auth challenge
     flows that must resend the body)                             (httpx2 §4, §18)
Is the operation non-idempotent?
  -> do not auto-retry without a protocol-level idempotency key   (httpx2 §25)
Is it PoolTimeout?
  -> fix concurrency, do not retry                                (httpx2 §13)
There is no universal retry switch. Define retryable methods, exceptions,
statuses, backoff, Retry-After, budget, and body replayability.   (httpx2 §25)
```

### 7. Stream or buffer?

```text
Is the response size unknown or potentially large?
  -> client.stream() + iter_bytes/aiter_bytes, inside a context manager
                                                                  (httpx2 §11)
Is it a large XML document?
  -> iterparse, clearing elements and deleting prior siblings     (lxml §11, §46)
Does data arrive incrementally under your control?
  -> feed parser or XMLPullParser                                 (lxml §10, §12)
Is it small and needed whole?
  -> ordinary buffered read
Streamed responses MUST be closed; .content on an unread stream is not the
body. Bound bytes at acquisition — parser-level bounds are the second line. (httpx2 §11, §31; slax §33)
```

### 8. Which test double?

```text
Testing your own WSGI app?      -> WSGITransport + Client         (httpx2 §20, §28)
Testing your own ASGI app?      -> ASGITransport + AsyncClient
                                   (lifespan is a separate concern)(httpx2 §20, §28)
Testing an ASGI WebSocket app?  -> ASGIWebSocketTransport         (httpx2 §22, §28)
Faking an external HTTP API?    -> MockTransport with a handler   (httpx2 §20, §29)
Reproducing browser network behavior deterministically?
                                -> route_from_har(), with an explicit
                                   miss policy                    (pw §27)
Need the page but not the network?
                                -> page.route() to fulfill/abort  (pw §26)
Testing a parser?               -> raw fixture bytes, per the doc's own
                                   corpus lists      (slax §38; lxml §54; extruct §28)
HAR is not a full browser-state snapshot, and routing disables the HTTP
cache while service workers can bypass interception entirely.     (pw §26, §27)
```

### 9. Bytes or `str`?

```text
At the wire (httpx2)
  -> keep .content (bytes) when fidelity matters; .text applies decoding
     logic and response.encoding may be inferred                  (httpx2 §9)
  -> if a signature or hash covers the encoded wire bytes, automatic
     content-decoding is NOT the representation you want          (httpx2 §26)
Into an HTML parser (selectolax)
  -> pass bytes + the HTTP charset; a decoding decision already made
     by handing it `str` cannot be undone                         (slax §29)
Into an XML parser (lxml)
  -> pass bytes so the XML encoding declaration is honored; a decoded
     `str` makes that declaration inconsistent or irrelevant       (lxml §49)
Into extruct
  -> HTML text/bytes, or reuse an existing lxml tree              (extruct §3, §18)
Retain the original response bytes regardless: no HTML5 parser round-trips
to them, so provenance needs the source.        (slax §4, §28; lxml §7)
```

### 10. Browser, context, or page?

```text
One browser process per worker; a browser is expensive              (pw §0, §44)
  |
  +-- One BrowserContext per isolated identity: per test, per scrape
  |   identity, per tenant, per credential boundary                 (pw §6)
  |     |
  |     +-- Pages and popups within one context share its state     (pw §24)
  |
  +-- Need a persistent on-disk profile?
        -> launch_persistent_context(), and only then               (pw §4)
Do not mutate one long-lived context into many identities, and do not launch
a browser per URL. Close contexts; storage state is a credential.  (pw §5, §7, §31, §52)
```

---

## §4 — Operating rules

Rules 1-8 are the key invariants in SKILL §Key invariants, restated here for completeness; 9-21 add the per-library specifics; 22-23 are navigational.

1. **HTTPX-family clients do not follow redirects by default.** Set `follow_redirects=True` deliberately, read the chain from `response.history`, and re-apply URL policy to every redirect target, not just the first. (httpx2 §10, §31)

2. **One long-lived client per policy boundary — never one per request.** The client owns pool, cookies, auth, TLS, and routing; scope it per proxy/TLS/auth/tenant. Use context managers. Streamed responses must be closed, and `.content` on an unread streaming response is not the body. (httpx2 §3, §11, §36)

3. **Playwright's isolation unit is `BrowserContext`, not `Browser`.** Locators re-resolve, `ElementHandle`s go stale. Prefer locator APIs and web-first `expect(...)` assertions; never `time.sleep()` for DOM readiness. (pw §0, §6, §9, §16, §20)

4. **selectolax nodes die with their parser** — extract Python primitives before releasing the tree. `node.attributes` is a value copy; `node.attrs` is a live mutating interface. Use the copy for read-only intent. (slax §8, §12)

5. **Parsed serialization ≠ source bytes for any HTML5 parser.** Implied structure, repaired nesting, and normalized entities all mean the tree is not the input. Retain the original bytes when fidelity or provenance matters. (slax §4, §28; lxml §7)

6. **lxml needs explicit parser policy for untrusted input.** Start from `resolve_entities=False, load_dtd=False, no_network=True, huge_tree=False` and re-enable only what a use case requires — 6.1's `resolve_entities="internal"` default is an improvement, not a policy. `lxml.html.clean` is no longer bundled (separate `lxml_html_clean` project). Compile XPath, XSLT, and validators once and reuse them. (lxml §2, §6, §17, §40, §47)

7. **extruct requires `base_url` set to the final post-redirect URL.** It extracts but does not validate against schema.org and does not reconcile duplicate entities — keep normalization, validation, and entity resolution downstream, retaining source syntax and provenance. (extruct §4, §21, §22, §23)

8. **tldextract's `suffix` is not a TLD and `domain` is not a registrable domain.** Use `top_domain_under_public_suffix`; `registered_domain` is deprecated. `ExtractResult` is a dataclass — no indexing, no tuple-unpacking. (tldx §7, §10, §11, §15, §43)

9. **`PoolTimeout` is a local saturation signal, not a remote failure.** Raise `Limits` or lower your own concurrency; retrying it makes the pressure worse. Set connect/read/write/pool budgets separately. (httpx2 §12, §13)

10. **There is no universal retry switch.** Transport `retries` cover connection establishment only. An application retry layer must define methods, idempotency, exception classes, status codes, backoff and jitter, `Retry-After`, a total budget, and body replayability — a streamed body cannot be replayed. (httpx2 §4, §18, §25)

11. **Catch narrowly.** `HTTPError → RequestError → TransportError → TimeoutException` and `HTTPStatusError` mean different things; collapsing them into "network failure" destroys diagnosis. (httpx2 §24, §36)

12. **An HTTPS destination usually uses an `http://` proxy URL** — HTTPS is tunneled through the proxy with CONNECT. Do not assume `https://proxy` is supported. Separately, `trust_env=False` if inherited `HTTP_PROXY`/`NO_PROXY` must not silently reroute traffic. (httpx2 §16, §17)

13. **HTTP 404 and 500 are completed responses, not `requestfailed` events.** Treating a status error as a network failure misclassifies the source's actual behavior. (pw §25, §52)

14. **Routing disables the HTTP cache, and service workers can bypass interception.** If deterministic routing or HAR replay is required, set the service-worker policy on the context and register routes before the sockets or pages exist. (pw §7, §26, §27, §28)

15. **Storage state is a credential.** Keep `storage_state()` files out of source control, scope them to an environment, use least-privilege accounts, and rebuild them when they expire. (pw §31, §52)

16. **`page.content()` is post-DOM serialization, not the network HTML.** Page JavaScript can inject or rewrite JSON-LD, so decide explicitly whether you want server-source or rendered metadata — and do not keep a browser alive to do offline traversal a parser does cheaper. (pw §48; extruct §20)

17. **selectolax escapes string input on insertion by design** — pass a `Node` when you actually mean markup. This is a safety property; do not defeat it with hand-built concatenation. After `unwrap_tags`, call `merge_text_nodes()` before separator-based text extraction, or artificial separators appear. (slax §14, §21, §25)

18. **Text extraction is a policy decision, not a getter.** `deep`, `separator`, `strip`, and `skip_empty` each change token boundaries; pick a combination deliberately and test it. Likewise `css_first(strict=True)` where the contract expects uniqueness — never use silent first-match semantics for identity-critical fields. (slax §6, §13)

19. **extruct's `uniform` mode is convenience, not fidelity**, and its `errors` mode (`strict`/`log`/`ignore`) is a deliberate choice: a crawler usually wants one malformed syntax not to discard the valid ones, with parse errors still observable. Embedded metadata is attacker-controlled — never execute values from it, and bound block count, block size, nesting, and object count. (extruct §15, §16, §24, §25)

20. **tldextract's quiet contracts.** `extract_urllib` is an **instance** method, not module-level. An empty `suffix` for an unrecognized final label is deliberate — tldextract does not apply the PSL's implicit wildcard fallback, so callers can detect internal or invalid names. `suffix_list_urls` is first-success, not merged. Never change `include_psl_private_domains` without versioning stored facts. (tldx §17, §19, §23, §24)

21. **None of these six is a security boundary.** httpx2 is "a transport client, not an SSRF firewall" (§31); Playwright is "not a security sandbox for your application" (§46); tldextract is explicitly "insufficient alone" for SSRF, phishing, allowlists, or same-site decisions (§34). Bound size, time, and expansion at acquisition; parser-level limits are the second line, not the first. And **no document in this pack canonicalizes a hostname** — `tldx §32` hands that back to the caller. (httpx2 §31; pw §46; tldx §32, §34; lxml §6; extruct §24)

22. **All six documents number their sections `# N. Title`.** A bare `§13` is ambiguous across six files — always cite with an alias, and always scope greps to one filename. (§5)

23. **Every document's last section is its anti-pattern inventory**, and five of six carry a testing section under three different titles. Load the anti-pattern inventory before drafting code and the testing section before writing tests. (SKILL §Navigation)

---

## §5 — Navigation hazards

This pack is unusually uniform, which helps reading and hurts searching.

- **The H1 prefix collides across all six files.** `grep '^# 13\.' docs/library_ref/*.md` returns six unrelated hits — `httpx2 §13` pooling, `pw §13` auto-waiting, `lxml §13` axes, `slax §13` text extraction, `tldx §13` `fqdn`, `extruct §13` RDFa. Always scope to one filename; always write the alias in citations.

- **Section numbers are per-document and not stable across the pack.** There is no shared numbering scheme. `§0` is "Mental model" in httpx2 and pw, "Capability map" in lxml, "What selectolax is" in slax, and "Purpose" in tldx and extruct.

- **The documents are flat.** Only httpx2 has real nesting (11 `##` / 6 `###`, listed inline in §1.1); `tldx §35` has three `##` entries. Sections run 5-40 lines, so `Read(offset, limit)` should take a whole section — paging into the middle of one usually lands mid-list.

- **Filenames embed versions.** `extruct_python_advanced_reference_0.18.0.md` becomes a different filename the moment a doc is refreshed. Glob by prefix (`extruct_python_advanced_reference_*`), not by exact name, when writing anything durable.

- **Closing-section titles are inconsistent.** The testing section is "Testing matrix" (httpx2 §35, pw §51, lxml §54, tldx §41), "Testing strategy" (slax §38), and "Testing corpus" (extruct §28). The agent-rules section is "LLM-agent decision rules" in five docs but **"LLM-agent execution playbook"** in lxml (§55). Grep `^# [0-9]*\. Testing` and `^# [0-9]*\. LLM-agent`, never an exact phrase. Only the anti-pattern inventory is named identically in all six.

- **Version drift: two documents lag the installed packages.** The httpx2 doc anchors 2.9.1 while `uv.lock` installs **2.12.0** — three minor releases, unaudited. `httpx2 §32` (HTTPX→HTTPX2 migration posture) and `§33` (the 2.9.1 `alias_httpx()` fix) are the most version-coupled sections in that document; verify either against the installed package before relying on it. The lxml doc anchors 6.1.1 while the lockfile installs **6.1.2** — treat `lxml §52`'s security floor as a floor. The other four match exactly. Full table: SKILL §Version anchors.

- **The pack's internal cross-links are real and mutually consistent** — `extruct §19`/`§20`, `slax §34`/`§35`, `lxml §51`, `pw §47`/`§48` were written to agree with each other. Where they are silent or where two documents both claim a topic, §2's **Authoritative** column is the tiebreak; where §2 says *no owner* (hostname canonicalization), nothing in this repository covers it.

- **Two documented claims are wrong against the installed versions** — verified by probe,
  not inference. (1) `extruct §4`'s `base_url` does **not** resolve JSON-LD or Open Graph
  URLs, nor microdata `itemid`; only microdata `itemprop` and RDFa `@id` resolve. (2)
  `extruct §16`'s `errors="log"` does **not** isolate a malformed block — one bad
  `<script>` drops the entire syntax key, because extruct materializes `list(extract(...))`
  per syntax. Drive `JsonLdExtractor().extract_items()` per node for per-block resilience.
  A third: `slax §29`'s "prefer bytes" is wrong for Lexbor, which ignores `<meta charset>`
  on bytes input — decode first.

- **Adjacent libraries have no reference documents here.** Robots/Protego, Tenacity, feedparser, `idna`, and `ipaddress` are named as needed by the surrounding architecture but have no deep-dive in `docs/library_ref/`. `jsonschema_python_advanced_reference_4.26.0.md` does exist and is a pinned direct dependency, but **no skill routes it** — read it directly. Pydantic is routed by the sibling skill `fastmcp-pydantic-ref`.
