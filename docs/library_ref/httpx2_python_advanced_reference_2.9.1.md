# HTTPX2 2.9.1 — advanced technical reference / feature and deployment catalog for LLM coding agents

## Version / source anchors

**Release anchor:** `httpx2==2.9.1`, released 2026-07-24.  
**Python floor:** Python 3.10+.  
**Stewardship:** Pydantic Services; continuation of the HTTPX design under the HTTPX2 package name.  
**License:** BSD-3-Clause.

### Source-of-truth hierarchy

1. released `httpx2 2.9.1` package and tagged source;
2. current HTTPX2 developer-interface documentation;
3. HTTPX2 advanced-usage documentation;
4. release/changelog notes;
5. `httpcore2` documentation for transport internals.

Primary sources:
- https://pypi.org/project/httpx2/2.9.1/
- https://httpx2.pydantic.dev/
- https://github.com/pydantic/httpx2
- https://httpx2.pydantic.dev/advanced/transports/
- https://httpx2.pydantic.dev/advanced/extensions/
- https://httpx2.pydantic.dev/websockets/
- https://httpx2.pydantic.dev/exceptions/

## Capability inventory

HTTPX2 provides:

- synchronous and asynchronous clients;
- HTTP/1.1 and optional HTTP/2;
- connection pooling and keep-alive;
- strict default timeouts;
- streaming uploads and downloads;
- browser-style TLS verification using the current HTTPX2 trust-stack;
- cookies and redirects;
- Basic/Digest/custom authentication;
- proxying, SOCKS, mounts, and environment-variable routing;
- WSGI and ASGI in-process transports;
- custom and mock transports;
- low-level request/response extensions;
- native WebSocket sessions in HTTPX2;
- optional Brotli and Zstandard decoding;
- CLI support;
- fully typed public APIs.

The package depends on `httpcore2`, `h11`, `anyio`, `truststore`, and `idna`. Optional capabilities add `h2`, `socksio`, `wsproto`, CLI packages, Brotli support, and Zstandard support.

---

# 0. Mental model and architecture

HTTPX2 separates four layers:

```text
application
  -> Client / AsyncClient
      -> Request / Response
          -> Transport
              -> httpcore2 / network backend
```

The `Client` owns connection pools, cookie state, default headers, authentication, routing, TLS configuration, limits, and lifecycle. `Request` and `Response` are message objects. A transport is the low-level adapter that actually sends a request.

**Agent rule:** do not bypass the client layer unless the use case genuinely requires transport-level control.

---

# 1. Installation and extras

```bash
pip install httpx2
pip install 'httpx2[http2]'
pip install 'httpx2[socks]'
pip install 'httpx2[ws]'
pip install 'httpx2[cli]'
pip install 'httpx2[brotli,zstd]'
```

Current optional features:

| Extra | Capability |
|---|---|
| `http2` | HTTP/2 via `h2` |
| `socks` | SOCKS proxying via `socksio` |
| `ws` | native WebSockets via `wsproto` |
| `cli` | command-line client |
| `brotli` | Brotli response decoding |
| `zstd` | Zstandard support on Python versions needing third-party package |

On Python 3.14+, HTTPX2 can use stdlib Zstandard support when available.

### Production dependency rule

Pin `httpx2` explicitly in applications that rely on exact transport behavior. If code depends on low-level trace-extension event names, also pin the compatible `httpcore2` line because those events are not a long-term protocol.

---

# 2. Top-level request API

The convenience surface mirrors Requests-style usage:

```python
import httpx2

r = httpx2.get("https://example.com")
r = httpx2.post("https://example.com/items", json={"x": 1})
r = httpx2.request("PATCH", url, content=b"...")
```

Common top-level verbs include `get`, `options`, `head`, `post`, `put`, `patch`, and `delete`, plus generic `request`.

Use top-level functions for scripts and truly one-off calls. They do not provide the connection reuse of a long-lived client across repeated requests.

**Anti-pattern:** repeatedly calling top-level helpers inside a crawler loop.

---

# 3. Client lifecycle

Canonical sync usage:

```python
with httpx2.Client(
    base_url="https://api.example.com",
    headers={"User-Agent": "my-app/1"},
    timeout=10.0,
) as client:
    response = client.get("/v1/items")
```

Canonical async usage:

```python
async with httpx2.AsyncClient(base_url="https://api.example.com") as client:
    response = await client.get("/v1/items")
```

Important lifecycle points:

- closing a client releases pooled connections;
- context managers are preferred;
- do not instantiate `AsyncClient` repeatedly in a hot async loop;
- use one appropriately scoped client per policy boundary (proxy/TLS/auth/trust/tenant), not necessarily one process-global client.

### Concurrency

`AsyncClient` supports `asyncio` and Trio through AnyIO-aware internals. Do not mix sync and async transports. A sync `Client` expects `BaseTransport`; an `AsyncClient` expects `AsyncBaseTransport`.

---

# 4. Request construction

Use:

```python
request = client.build_request(
    "POST",
    "/items",
    params={"page": 2},
    headers={"X-Trace": "abc"},
    json={"name": "example"},
)
response = client.send(request)
```

Request body choices:

- `content=` for bytes/str/iterable content;
- `json=` for JSON;
- `data=` for form-encoded data or appropriate form structures;
- `files=` for multipart files.

For methods whose convenience functions intentionally omit request-body parameters (notably GET/DELETE/HEAD/OPTIONS), use generic `request()` when a body is really required.

### Chunked request bodies

Iterators / async iterators can be used for streamed request content. Avoid buffering multi-GB payloads just to hand bytes to the client.

---

# 5. URL model

`httpx2.URL` is a structured URL object. Response URLs are URL objects, not raw strings.

```python
url = httpx2.URL("https://example.com/a?x=1")
str(url)
```

Operational rules:

- use `str(url)` at serialization boundaries;
- avoid hand-concatenating query strings;
- use `base_url` for API clients;
- treat URL normalization and application security validation as separate from “it parsed successfully.”

Internationalized domain names are supported through IDNA handling.

---

# 6. Query parameters

Use mappings, sequences of pairs, or `QueryParams` where duplicate keys/order matter.

```python
params = httpx2.QueryParams([("tag", "a"), ("tag", "b")])
client.get("/search", params=params)
```

Do not collapse multi-valued parameters into a plain dict when duplicate-key semantics are required.

---

# 7. Headers

`Headers` is a case-insensitive multi-dict-like abstraction that preserves HTTP header semantics.

Key cautions:

- header names are case-insensitive;
- repeated headers can matter;
- automatic headers may be added by HTTPX2;
- never log credentials, cookies, or authorization headers by default.

---

# 8. Cookies

Persistent cookies belong on a `Client`.

```python
client = httpx2.Client(cookies={"session": "..."})
```

HTTPX2 intentionally has stricter cookie state expectations than Requests. For repeated/redirecting flows, set client-level cookies rather than treating cookies as ad-hoc per-request mutable state.

`CookieConflict` can occur when name-only lookup is ambiguous across domains/paths.

---

# 9. Response model

Core properties:

```python
response.status_code
response.headers
response.url
response.history
response.content
response.text
response.encoding
response.request
response.extensions
response.cookies
```

Common methods:

```python
response.json()
response.raise_for_status()
```

Status helpers distinguish informational, success, redirect, client-error, and server-error classes. `raise_for_status()` raises `HTTPStatusError` for error statuses.

### Encoding

Prefer bytes when fidelity matters. `.text` applies decoding logic; `response.encoding` may be inferred/selected. For arbitrary binary responses, do not round-trip through text.

---

# 10. Redirects

HTTPX-family behavior does not follow redirects by default unless configured.

```python
client = httpx2.Client(follow_redirects=True)
```

Use `response.history` for the redirect chain and `response.next_request` when controlling redirects manually.

Security rule: for crawlers, validate every redirect target, not just the initial URL. SSRF policy must be re-applied after redirects and DNS changes.

---

# 11. Streaming downloads

Sync:

```python
with client.stream("GET", url) as response:
    response.raise_for_status()
    for chunk in response.iter_bytes():
        ...
```

Async:

```python
async with client.stream("GET", url) as response:
    async for chunk in response.aiter_bytes():
        ...
```

Streaming APIs include byte, raw, text, and line iteration plus explicit read operations.

Lifecycle rule: streamed responses must be closed. Context-manager usage is the safest default.

**Anti-pattern:** accessing `.content` on a not-yet-read streaming response and assuming the body is buffered.

Relevant stream exceptions include `StreamConsumed`, `StreamClosed`, `ResponseNotRead`, and `RequestNotRead`.

---

# 12. Timeouts

HTTPX2 has explicit timeout semantics for:

- connect;
- read;
- write;
- pool acquisition.

```python
timeout = httpx2.Timeout(
    connect=5.0,
    read=15.0,
    write=15.0,
    pool=5.0,
)
client = httpx2.Client(timeout=timeout)
```

A scalar applies a common timeout policy. `None` disables a timeout component / can disable timeouts depending on placement.

Exception mapping:

- `ConnectTimeout`
- `ReadTimeout`
- `WriteTimeout`
- `PoolTimeout`

Agent rule: use differentiated timeout budgets for production acquisition. A 30-second read may be acceptable while a 30-second pool wait usually indicates saturation.

---

# 13. Resource limits and pooling

Use `httpx2.Limits` to control connection pool behavior.

Typical concerns:

- maximum total connections;
- maximum keep-alive connections;
- keep-alive expiry.

Pool sizing should reflect target concurrency and remote-host fan-out. Excessively high connection counts can exhaust local file descriptors or overwhelm destinations.

A `PoolTimeout` is a useful saturation signal; do not blindly retry it without considering local concurrency pressure.

---

# 14. HTTP/2

Install the extra and enable on a client where appropriate.

```python
client = httpx2.Client(http2=True)
```

Benefits can include multiplexing and improved connection utilization, but HTTP/2 is not automatically “faster” for every workload.

Inspect `response.http_version` / response extensions when protocol identity matters.

Operational caveats:

- server support determines negotiated protocol;
- proxies can alter negotiation;
- benchmark the actual target workload;
- do not assume request completion order under multiplexing.

---

# 15. TLS / SSL

HTTPX2 performs TLS verification by default and uses its current trust-stack, including `truststore` integration.

Rules:

- keep verification enabled;
- pass a deliberate SSL context for custom CAs/mTLS;
- isolate distinct TLS policies in separate client instances;
- do not make `verify=False` a generic fix for certificate errors.

For certificate pinning or custom hostname rules, design and test the trust boundary explicitly rather than relying on incidental transport internals.

---

# 16. Environment variables

HTTPX2 honors standard environment variables by default, including proxy variables and certificate-related environment conventions documented by the project.

Disable environment-derived behavior:

```python
client = httpx2.Client(trust_env=False)
```

This matters in hermetic services and agents: inherited `HTTP_PROXY`, `HTTPS_PROXY`, `ALL_PROXY`, and `NO_PROXY` values can silently change network routing.

---

# 17. Proxies

Simple proxy:

```python
client = httpx2.Client(proxy="http://proxy.example:8080")
```

Different routing policies use mounts and explicit transports.

```python
mounts = {
    "http://": httpx2.HTTPTransport(proxy="http://proxy-a:8080"),
    "https://": httpx2.HTTPTransport(proxy="http://proxy-b:8080"),
}
client = httpx2.Client(mounts=mounts)
```

SOCKS requires the `socks` extra.

Important operational detail: an HTTPS destination commonly still uses an `http://` proxy URL because HTTPS is tunneled through an HTTP proxy with CONNECT. HTTPS-to-proxy support has separate limitations; do not assume `https://proxy` is universally supported.

---

# 18. Authentication

Built-in patterns include Basic and Digest authentication plus custom auth classes.

For application-specific schemes, implement the HTTPX2 authentication flow rather than wrapping every call manually.

Auth design rules:

- avoid logging credentials;
- understand whether auth must replay request bodies;
- streamed/non-rewindable bodies complicate retry/auth challenge flows;
- scope tokens and auth objects to the intended host/policy boundary.

---

# 19. Event hooks

Clients support request and response event hooks.

Use cases:

- observability;
- request IDs;
- metrics;
- standardized header injection;
- response policy checks.

Do not implement complex retry orchestration by recursively issuing requests inside hooks unless you have explicitly modeled reentrancy and streaming behavior.

Async clients require async-compatible hook functions where the API expects awaited work.

---

# 20. Transports

## HTTPTransport / AsyncHTTPTransport

Instantiate directly for low-level network options such as:

- connection retries for connect failures;
- local address binding;
- Unix domain sockets;
- proxy transport placement.

Transport-level `retries` are intentionally narrow: they cover connection establishment classes, not arbitrary response/status retry policy.

For status-based backoff, use an explicit retry layer.

## WSGITransport

Directly call WSGI applications without opening a network socket.

Useful for:

- unit/integration tests;
- service adapters;
- deterministic local testing.

## ASGITransport

Async equivalent for ASGI applications.

Important: lifespan startup/shutdown is a separate concern; use an appropriate lifespan manager when the app requires lifecycle events.

## MockTransport

Return deterministic responses based on a handler.

```python
def handler(request):
    return httpx2.Response(200, json={"ok": True})

client = httpx2.Client(transport=httpx2.MockTransport(handler))
```

## Custom transports

Subclass `BaseTransport` or `AsyncBaseTransport` and implement the corresponding request handler.

Use custom transports for cross-cutting wire behavior that genuinely belongs below the client layer.

---

# 21. Mount-based routing

HTTPX2 can route by:

- scheme;
- host/domain;
- wildcard host;
- port;
- combinations.

Examples include proxy exceptions, custom transports for one service, and HTTP-vs-HTTPS differences.

Routing belongs in client construction, not in a giant conditional around every request.

---

# 22. Native WebSockets in HTTPX2

HTTPX2 adds native WebSocket support via the `ws` extra.

```bash
pip install 'httpx2[ws]'
```

Sync:

```python
with httpx2.Client() as client:
    with client.websocket("wss://example.com/ws") as ws:
        ws.send_text("hello")
        message = ws.receive_text()
```

Async:

```python
async with httpx2.AsyncClient() as client:
    async with client.websocket("wss://example.com/ws") as ws:
        await ws.send_json({"op": "ping"})
        message = await ws.receive_json()
```

Capabilities include:

- text and binary frames;
- JSON convenience methods;
- receive timeout;
- subprotocol negotiation;
- keepalive pings;
- handshake response access;
- close/disconnect metadata;
- sync and async sessions.

Configurable session concerns include:

- message/chunk size behavior;
- inbound queue size;
- ping interval;
- ping timeout.

WebSocket exceptions live in `httpx2.websockets`, including upgrade, disconnect, invalid-type, and network errors.

### ASGIWebSocketTransport

HTTPX2 provides an ASGI WebSocket transport for testing ASGI applications without a listening server.

This is a major difference from older HTTPX-era designs that typically relied on a separate `httpx-ws` integration.

---

# 23. Request / response extensions

Extensions are an intentionally low-level, partly transport-specific escape hatch.

Request extensions include:

- `trace`;
- `sni_hostname`;
- `timeout`;
- `target`.

Response extensions include:

- `http_version`;
- `reason_phrase`;
- `stream_id`;
- `network_stream`.

## Trace extension

Can expose connection/TLS/HTTP state events. Use for diagnostics and instrumentation, but treat exact event names as version-coupled to `httpcore2`.

## SNI override

`sni_hostname` allows connection to an explicit IP while verifying a different certificate hostname.

This is security-sensitive and should be used only when the Host header, SNI value, and connection target are intentionally separated.

## Target override

Allows non-standard HTTP request-target forms such as `OPTIONS *`, CONNECT targets, and unusual escaping.

## network_stream

Exposes underlying network stream operations and socket/TLS metadata. This is an advanced escape hatch, not an application-level API.

---

# 24. Exceptions

High-level hierarchy:

```text
HTTPError
├── RequestError
│   ├── TransportError
│   │   ├── TimeoutException
│   │   ├── NetworkError
│   │   ├── ProtocolError
│   │   ├── ProxyError
│   │   ├── UnsupportedProtocol
│   │   └── SSEError
│   ├── DecodingError
│   └── TooManyRedirects
└── HTTPStatusError
```

Separate validation/state errors include `InvalidURL`, `CookieConflict`, and stream-state exceptions.

Catch narrowly enough to preserve diagnosis:

```python
try:
    r = client.get(url)
    r.raise_for_status()
except httpx2.TimeoutException:
    ...
except httpx2.HTTPStatusError as exc:
    ...
except httpx2.RequestError as exc:
    ...
```

Do not catch `Exception` and label everything “network failure.”

---

# 25. Retries

There is no universal “retry all failures” switch at the application layer.

Transport retries cover connection-establishment cases. For higher-level retries define:

- retryable methods;
- idempotency requirements;
- retryable exception classes;
- retryable status codes;
- backoff and jitter;
- `Retry-After`;
- total attempt/time budget;
- request-body replayability.

Never retry non-idempotent operations automatically without a protocol-level idempotency strategy.

---

# 26. Compression and decoding

HTTPX2 automatically handles common response content encodings when support is installed.

Optional support:

- Brotli;
- Zstandard.

Keep raw-wire requirements separate from decoded-body requirements. If a signature/hash applies to encoded wire bytes, automatic content decoding may not be the representation you want.

---

# 27. CLI

Install:

```bash
pip install 'httpx2[cli]'
```

Use the CLI for interactive diagnostics, not as a replacement for an application integration. Pin versions in scripts if CLI output is parsed.

---

# 28. In-process application testing

Canonical choices:

- WSGI app → `WSGITransport` + `Client`;
- ASGI app → `ASGITransport` + `AsyncClient`;
- ASGI WebSocket app → `ASGIWebSocketTransport`;
- external API mock → `MockTransport`.

This avoids flaky loopback sockets and makes transport policy explicit.

---

# 29. Mocking ecosystem

Native `MockTransport` is sufficient for many tests. Third-party packages may add pytest fixtures, request assertions, recording, caching, or richer route DSLs.

Prefer the native transport when you only need deterministic request→response mapping.

---

# 30. Performance playbook

1. Reuse clients.
2. Stream large bodies.
3. Set bounded connection pools.
4. Use async when you have high I/O concurrency and the rest of the stack is async.
5. Avoid unbounded fan-out.
6. Prefer HTTP/2 only after validating target support and workload benefit.
7. Keep DNS/proxy/TLS policy stable within a client.
8. Measure pool waits, connect time, first byte, download time, and status outcomes separately.

---

# 31. Security playbook

For untrusted URLs:

- allowlist schemes;
- reject embedded credentials unless explicitly supported;
- resolve and validate destinations against SSRF policy;
- revalidate redirects;
- consider DNS rebinding;
- block link-local, loopback, private, metadata-service, and other forbidden ranges as required;
- bound response size and time;
- keep TLS verification enabled;
- bound decompression expansion;
- never trust `Content-Type` alone;
- redact authorization/cookie/proxy credentials from logs.

HTTPX2 is a transport client, not an SSRF firewall.

---

# 32. HTTPX → HTTPX2 migration posture

HTTPX2 intentionally continues the HTTPX design, but it is a distinct package and evolving line.

Key migration rules:

- change imports deliberately rather than relying on an accidental transitive `httpx`;
- verify third-party integrations against HTTPX2;
- replace HTTPX-only mocking adapters with HTTPX2-compatible ones where necessary;
- recognize new native WebSocket capability;
- validate custom transports against `httpcore2`;
- test package-specific aliases/shims only if your application intentionally uses them.

Do not assume “same API shape” means every plugin using private HTTPX internals will work unchanged.

---

# 33. 2.9.1-specific release note

The 2.9.1 package fixes HTTP core aliasing behavior in the HTTPX compatibility helper (`alias_httpx()`), ensuring `httpcore` imports are redirected to `httpcore2` in that compatibility path.

Treat compatibility aliasing as migration infrastructure, not the preferred steady-state import model for new code.

---

# 34. LLM-agent decision rules

Use HTTPX2 when:
- a real browser is unnecessary;
- you need efficient repeated HTTP;
- you need sync/async parity;
- you need HTTP/2, transport mounting, in-process ASGI/WSGI, or native WebSockets.

Do not escalate to Playwright just to fetch HTML that HTTPX2 can retrieve reliably.

Before implementing custom code, check built-ins in this order:

```text
Client option
-> request option
-> event hook
-> transport/mount
-> extension
-> custom transport
```

Only move downward when the higher-level layer cannot express the requirement.

---

# 35. Testing matrix

```text
[ ] HTTP/1.1
[ ] HTTP/2 where enabled
[ ] redirects allowed/blocked
[ ] DNS/connect timeout
[ ] read timeout
[ ] write timeout
[ ] pool timeout
[ ] proxy path
[ ] no-proxy path
[ ] TLS validation failure
[ ] streaming close/early abort
[ ] large response limit
[ ] decompression
[ ] duplicate headers/query params
[ ] authentication challenge
[ ] retryable and non-retryable failures
[ ] WSGI/ASGI transport if used
[ ] WebSocket clean close
[ ] WebSocket server disconnect
[ ] WebSocket timeout/ping failure
[ ] SSRF-denied targets and redirect targets
```

# 36. Anti-pattern inventory

- creating a new client for every request in a loop;
- disabling TLS verification globally;
- treating all `HTTPError` subclasses as equivalent;
- automatic retries without idempotency analysis;
- reading huge bodies eagerly;
- unbounded async fan-out;
- accepting proxy environment variables unintentionally;
- using low-level extensions when a documented client option exists;
- relying on exact trace event names without pinning lower-layer versions;
- assuming WebSocket messages are bounded;
- assuming URL parse success equals URL safety.
