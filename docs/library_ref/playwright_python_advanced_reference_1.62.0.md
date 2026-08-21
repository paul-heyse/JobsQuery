# Playwright for Python 1.62.0 — advanced technical reference / feature and deployment catalog for LLM coding agents

## Version / source anchors

**Release anchor:** `playwright==1.62.0`, released 2026-07-31.  
**Reference date:** 2026-08-20.  
**Primary role:** deterministic browser automation and browser-mediated acquisition.

Sources:
- https://pypi.org/project/playwright/
- https://playwright.dev/python/docs/
- https://playwright.dev/python/docs/api/class-playwright
- https://playwright.dev/python/docs/release-notes
- https://github.com/microsoft/playwright-python

Version 1.62 adds WebP screenshots, action `scroll` control, `Locator.wait_for_function()`, and `APIResponse.timing`.

---

# 0. Mental model

```text
Playwright
  -> BrowserType (chromium/firefox/webkit)
      -> Browser
          -> BrowserContext
              -> Page
                  -> Frame
                      -> Locator
```

Secondary object families include:

- Request / Response / Route;
- WebSocket / WebSocketRoute;
- APIRequestContext / APIResponse;
- Download / FileChooser / Dialog;
- JSHandle / ElementHandle;
- Worker / ServiceWorker;
- CDPSession (Chromium);
- Clock;
- Tracing;
- Credentials / WebAuthn;
- selectors and accessibility/ARIA surfaces.

**Core design rule:** a browser process is expensive; browser contexts are the normal isolation unit.

---

# 1. Installation

```bash
pip install playwright
python -m playwright install
```

Install browser OS dependencies on Linux/CI:

```bash
python -m playwright install --with-deps
```

Install selected engines:

```bash
python -m playwright install chromium
python -m playwright install firefox webkit
```

The Python package version and managed browser binaries are coupled. Upgrade both together.

---

# 2. Sync and async APIs

Sync:

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto("https://example.com")
    browser.close()
```

Async:

```python
from playwright.async_api import async_playwright

async with async_playwright() as p:
    browser = await p.chromium.launch()
    page = await browser.new_page()
    await page.goto("https://example.com")
    await browser.close()
```

Do not mix sync objects with async objects.

---

# 3. Browser engines

Playwright manages:

- Chromium;
- Firefox;
- WebKit.

Browser behavior is close to the corresponding engines, but Playwright ships tested revisions. For reproducible acquisition, pin Playwright and install its matching browser revisions.

Browser channels can target supported installed channels where documented.

---

# 4. BrowserType

Primary operations:

- `launch()`;
- `launch_persistent_context()`;
- `connect()` to a Playwright server;
- `connect_over_cdp()` for Chromium-family CDP connectivity;
- executable-path/channel selection.

Use `launch_persistent_context()` only when persistent browser profile semantics are required. The normal default is ephemeral `Browser` + isolated `BrowserContext`.

CDP connectivity is lower fidelity than the native Playwright protocol for some features.

---

# 5. Browser

Important capabilities:

- create contexts;
- create convenience pages;
- inspect contexts;
- close or disconnect;
- version information;
- CDP sessions where supported.

Prefer:

```python
browser = p.chromium.launch()
context = browser.new_context()
page = context.new_page()
...
context.close()
browser.close()
```

rather than using a single shared default state for unrelated tasks.

---

# 6. BrowserContext: isolation boundary

A context isolates:

- cookies;
- local/session storage;
- permissions;
- pages/popups;
- geolocation;
- locale/timezone;
- proxy and many emulation settings;
- authentication/storage state.

Non-persistent contexts do not write ordinary browsing state to disk.

Use one context per:
- test;
- scrape identity;
- tenant;
- credential boundary;
- isolation-sensitive job.

---

# 7. Context configuration

Common context options include:

- viewport / screen;
- device scale factor;
- mobile/touch flags;
- locale;
- timezone;
- geolocation;
- permissions;
- color scheme;
- reduced motion;
- forced colors;
- user agent;
- extra HTTP headers;
- HTTP credentials;
- offline state;
- proxy;
- client certificates where supported;
- service worker policy;
- storage state;
- base URL.

Keep context construction declarative. Avoid mutating a long-lived context into many unrelated identities.

---

# 8. Pages and navigation

Core page operations:

- `goto()`;
- `reload()`;
- `go_back()` / `go_forward()`;
- `set_content()`;
- `content()`;
- `title()`;
- `url`;
- close/bring-to-front.

Navigation wait conditions include load-state concepts such as:
- `commit`;
- `domcontentloaded`;
- `load`;
- `networkidle` where supported.

Do not use arbitrary `sleep()` as a substitute for state-based waits.

---

# 9. Locator-first design

A Locator is a re-resolving query, not a stale DOM node.

Preferred patterns:

```python
page.get_by_role("button", name="Submit")
page.get_by_label("Email")
page.get_by_text("Welcome")
page.get_by_test_id("save")
page.locator("article.product")
```

Locators support chaining/filtering and auto-waiting.

**Agent rule:** use Locator APIs unless a lower-level handle is specifically needed.

---

# 10. Locator strategies

High-signal locator families:

1. role + accessible name;
2. label;
3. placeholder;
4. text;
5. alt text;
6. title;
7. test id;
8. CSS;
9. XPath as an escape hatch.

Prefer user-visible/accessibility semantics for tests. For scraping, CSS can be appropriate when the site structure is the contract.

---

# 11. Locator filtering and composition

Use:

- `.filter(...)`;
- `.and_()` / `.or_()` where available;
- `.first`, `.last`, `.nth()`;
- descendant locators;
- frame locators.

Avoid large, brittle CSS/XPath expressions when a composable locator expresses the relationship more robustly.

---

# 12. Strictness

Actions that expect one element fail if a locator resolves ambiguously. This catches selector drift early.

Do not reflexively append `.first` to silence strictness failures; determine whether multiple matches are actually acceptable.

---

# 13. Auto-waiting / actionability

Before actions, Playwright waits for relevant actionability conditions such as:

- resolved uniqueness;
- visibility;
- stability;
- receiving events;
- enabled/editable state.

This is why Playwright actions generally should not be preceded by hand-written polling.

---

# 14. Actions

Typical Locator actions:

- click / dblclick;
- hover;
- fill / clear;
- type-like keyboard input where needed;
- press;
- check / uncheck;
- select option;
- focus / blur;
- drag;
- tap;
- set input files;
- scroll-related actions.

1.62 adds a `scroll` option on actions so callers can disable Playwright's automatic scroll-into-view when appropriate.

---

# 15. Locator.wait_for_function() — 1.62

Version 1.62 adds a locator-level function wait.

Use it when the desired readiness condition is a predicate on the matching element that is not represented by built-in states/assertions.

Prefer a built-in assertion/wait when one exists; custom JS conditions should remain the exception.

---

# 16. Assertions

Playwright's `expect(...)` assertions provide web-first retrying assertions.

Examples:

```python
expect(locator).to_be_visible()
expect(locator).to_have_text("Ready")
expect(page).to_have_url(...)
```

They repeatedly re-check until success/timeout and are generally more stable than:

```python
assert locator.is_visible()
```

followed by manual sleeps.

---

# 17. Frames and iframes

Use `Frame` or, preferably for element targeting, `FrameLocator`.

Do not assume iframe DOM is part of the main frame's selector scope.

Frame lifecycle and navigation are independent enough that acquisition code should identify which frame actually owns the data.

---

# 18. Shadow DOM

Locator engines pierce open shadow roots for many standard locator operations. Closed shadow roots remain inaccessible through normal DOM APIs.

Do not add custom JavaScript traversal until built-in locator behavior is proven insufficient.

---

# 19. JavaScript evaluation

Use:

- `page.evaluate()`;
- `locator.evaluate()`;
- `evaluate_handle()` / JS handles.

Use evaluation for:
- browser-only state;
- structured extraction that is most naturally computed in-page;
- instrumenting app behavior.

Avoid dumping arbitrary page objects across the protocol. Serialize compact primitives/dicts instead.

---

# 20. ElementHandle / JSHandle

Handles represent remote browser objects and have explicit lifetime/disposal considerations.

Prefer locators because locators re-resolve after DOM mutations. A stored ElementHandle can become detached/stale.

Handles are justified when:
- interacting with a specific JS object identity;
- using APIs that require handles;
- passing a fixed node into JS evaluation.

---

# 21. Downloads

Use event-scoped download handling.

```python
with page.expect_download() as info:
    page.get_by_text("Download").click()
download = info.value
```

Inspect suggested filename and save to an explicit controlled path.

Security:
- treat downloaded files as untrusted;
- do not execute them;
- enforce size/type policy separately.

---

# 22. File uploads and file chooser

Use `set_input_files()` for known inputs. Use file chooser events when the app dynamically opens one.

Avoid relying on OS-native file dialogs; Playwright's API bypasses them.

---

# 23. Dialogs

Dialogs include alert/confirm/prompt/beforeunload-like flows.

Register a handler before the action that can trigger the dialog. Unhandled modal dialogs can block the page.

---

# 24. Popups / new pages

Use `page.expect_popup()` for an action expected to open a popup, or `context.expect_page()` at context scope.

The popup belongs to the same browser context.

---

# 25. Network observation

Relevant events:

- `request`;
- `response`;
- `requestfinished`;
- `requestfailed`.

HTTP 404/500 responses are still completed HTTP responses, not network failures.

Capture final URLs, headers, status, resource type, initiator-related information where exposed, and response bodies selectively.

---

# 26. Routing and interception

Use `page.route()` or `browser_context.route()`.

A matching route stalls until the handler:
- continues;
- fulfills;
- aborts;
- falls back as supported.

Context routes apply broadly; page routes can take precedence.

Routing disables HTTP cache for affected behavior. Service workers can bypass normal interception, so block service workers when deterministic routing/HAR behavior requires it.

---

# 27. HAR replay / recording

`route_from_har()` can serve requests from HAR data and define behavior for misses.

Use HAR replay for:
- deterministic tests;
- isolating front-end behavior;
- reproducing network scenarios.

Do not confuse HAR with a complete browser-state snapshot.

---

# 28. WebSockets

`WebSocket` objects expose browser WebSocket connections and frame events.

`BrowserContext.route_web_socket()` / related WebSocketRoute APIs can intercept and modify WebSocket traffic.

Set routing before creating the relevant sockets/pages.

This is useful for:
- deterministic tests;
- protocol inspection;
- message filtering;
- mocked browser-side streaming.

---

# 29. APIRequestContext

Playwright includes an HTTP API client useful for test setup and browser-coupled workflows.

A context-associated `APIRequestContext` can share cookies with its BrowserContext. A standalone context is isolated.

Use cases:
- authenticate by API then browse;
- seed server state;
- check post-conditions;
- make network calls whose cookie coupling to the browser is important.

For general-purpose high-throughput HTTP acquisition outside browser workflows, HTTPX2 remains the better architectural default.

---

# 30. APIResponse.timing — 1.62

Version 1.62 exposes response resource timing through `APIResponse.timing`.

Use it for diagnostic/test timing data, but do not assume browser-navigation performance APIs and APIRequestContext timing are identical measurement systems.

---

# 31. Cookies and storage state

Context APIs manage cookies. `storage_state()` can persist reusable authentication state including supported storage domains.

Treat storage-state files as credentials.

Rules:
- keep them out of source control;
- use least-privilege accounts;
- rotate/rebuild when expired;
- scope state to the correct environment.

---

# 32. Permissions and geolocation

Context-level permissions allow deterministic tests for APIs such as geolocation, notifications, clipboard, etc.

Headless clipboard behavior changed in recent Playwright: headless clipboard is isolated from the OS, improving test isolation.

Do not assume browser permissions equal OS permissions.

---

# 33. Device and environment emulation

Playwright supports:
- viewport;
- mobile/touch;
- locale;
- timezone;
- color scheme;
- reduced motion;
- geolocation;
- user agent;
- predefined device descriptors.

Emulation changes browser-facing behavior but does not magically reproduce all hardware/network characteristics of a real device.

---

# 34. Clock

`BrowserContext.clock` can control time for pages/frames in the context.

Capabilities include installing fake time, setting fixed/system time, fast-forwarding, and running timer-driven behavior.

Use it instead of long real-time sleeps in time-dependent tests.

---

# 35. WebAuthn / Credentials

Recent Playwright versions expose virtual WebAuthn authenticator/credential facilities through context credentials APIs.

Use for:
- passkey/WebAuthn tests;
- deterministic credential creation/get flows.

Keep this separate from ordinary HTTP Basic credentials.

---

# 36. Screenshots

`page.screenshot()` and `locator.screenshot()` support page/element capture.

Version 1.62 adds WebP.

```python
page.screenshot(path="page.webp", quality=50)
```

Quality 100 is lossless for WebP; lower values are lossy.

Other concerns include:
- full page vs viewport;
- masking;
- animations/caret;
- clipping;
- scale;
- background handling.

Screenshots are observations, not a reliable DOM extraction format.

---

# 37. PDF

Chromium supports page PDF generation where documented.

Use for print rendering, not as a substitute for HTML storage if machine extraction is the primary use case.

---

# 38. Video

Browser contexts can record video when configured. Video is useful for debugging failures but adds substantial storage/CPU overhead.

Enable selectively in CI, commonly retain-on-failure through pytest integration.

---

# 39. Tracing / Trace Viewer

Tracing can capture:
- actions;
- DOM snapshots;
- screenshots;
- network metadata and related diagnostics.

Use tracing as the primary forensic artifact for flaky browser workflows.

Do not leave full tracing permanently enabled in very high-volume production acquisition without understanding storage overhead and sensitive-data capture.

---

# 40. ARIA snapshots and AI-oriented snapshots

Recent versions expose ARIA snapshots, including an AI-oriented mode with element references and iframe snapshots.

This can be valuable for agent navigation because it is lower-token and semantically richer than raw HTML for some tasks.

Treat it as an accessibility representation, not a canonical DOM serialization.

---

# 41. Codegen

`playwright codegen` records interactions and proposes locators/code.

Use as:
- exploration;
- locator discovery;
- initial scaffolding.

Review generated code; do not assume it is the optimal abstraction for a maintainable suite.

---

# 42. Inspector and debugging

`PWDEBUG=1` activates Inspector-oriented debugging.

Other useful tools:
- headed mode;
- slow motion;
- browser devtools where explicitly configured;
- trace viewer;
- verbose Playwright logs.

Debug state-based problems with events/locators/traces before adding sleeps.

---

# 43. Pytest plugin

Install `pytest-playwright` for fixtures and CLI integration.

Typical fixtures:
- `playwright`;
- `browser_type`;
- `browser`;
- `context`;
- `page`.

Common CLI controls:
- browser;
- headed;
- browser channel;
- slowmo;
- device;
- output;
- tracing;
- video;
- screenshots.

CLI options apply to plugin-created default fixtures, not arbitrary contexts you manually construct unless you propagate options.

---

# 44. Parallelism

Pytest parallelism commonly uses `pytest-xdist`.

Scale by:
- processes/workers;
- browser reuse;
- isolated browser contexts.

Watch:
- RAM;
- CPU;
- browser-process count;
- destination load;
- test data contention.

More browser workers are not always faster.

---

# 45. CI and containers

On Linux use Playwright's documented system dependency install or matching container image.

Production/CI principles:
- pin Playwright version;
- pin compatible browser images/binaries;
- use enough shared memory;
- understand sandbox settings;
- collect trace/screenshot artifacts on failure;
- do not run untrusted browser content with unnecessarily broad host/container privileges.

---

# 46. Browser security boundary

Playwright automates a real browser; it is not a security sandbox for your application.

For untrusted targets:
- constrain network egress if needed;
- isolate OS/container identity;
- keep downloads controlled;
- do not expose local file paths/secrets through page scripts;
- be careful with `page.evaluate()` and values returned from untrusted pages;
- apply SSRF policy to browser network access at infrastructure/routing layer if required.

Browser sandbox flags should not be disabled casually.

---

# 47. Acquisition escalation policy

Prefer HTTPX2 when:
- server-rendered HTML/API data is sufficient;
- browser JS is unnecessary.

Use Playwright when:
- JS must execute;
- content requires browser cookies/storage;
- browser-only anti-bot/session behavior is required;
- interactions reveal data;
- browser network requests must be observed/intercepted;
- WebSocket behavior is in-page;
- DOM after app hydration is the desired artifact.

---

# 48. Extraction handoff

For static post-load processing:

```python
html = page.content()
```

Then hand HTML to:
- selectolax for fast HTML parsing;
- lxml for XPath/rich markup;
- extruct for embedded metadata.

Do not keep the browser alive solely to do offline DOM traversals that a parser can do cheaper.

---

# 49. Determinism playbook

Stabilize:
- browser version;
- viewport/device;
- locale/timezone;
- clock where needed;
- permissions;
- geolocation;
- network routing;
- service-worker policy;
- storage state;
- server fixtures/HAR;
- test IDs / semantic locators.

---

# 50. LLM-agent decision rules

When a user asks for browser automation, search current Playwright APIs before writing:
- manual polling;
- DOM query loops;
- arbitrary sleeps;
- custom network interception;
- custom clock monkeypatching;
- browser-profile file surgery.

Prefer built-in:
```text
Locator
-> web-first assertion / expect_event
-> context/page routing
-> storage state
-> Clock
-> APIRequestContext
-> trace/HAR
```

---

# 51. Testing matrix

```text
[ ] Chromium
[ ] Firefox if supported requirement
[ ] WebKit if supported requirement
[ ] headless
[ ] headed smoke
[ ] clean context
[ ] reused auth state
[ ] popup
[ ] iframe
[ ] open shadow DOM
[ ] download
[ ] upload
[ ] dialog
[ ] redirect/navigation
[ ] request failure vs HTTP error
[ ] route/fulfill/abort
[ ] service worker interaction
[ ] WebSocket traffic
[ ] timeout path
[ ] screenshot/trace artifact generation
[ ] timezone/locale-sensitive behavior
[ ] CI container
```

# 52. Anti-pattern inventory

- `time.sleep()` for DOM readiness;
- CSS selectors based on unstable generated classes when semantic locators exist;
- storing ElementHandles across rerenders;
- sharing one context across unrelated authenticated identities;
- using Playwright for every HTTP fetch;
- starting a whole browser per URL in high-volume crawlers;
- ignoring service workers while expecting full route interception;
- disabling browser sandbox without a deployment reason;
- persisting storage state in source control;
- failing to close contexts/browsers;
- treating 404 as a `requestfailed` event.
