# tldextract 5.3.2 — advanced technical reference / feature and deployment catalog for LLM coding agents

## Version / source anchors

**Release anchor:** `tldextract==5.3.2`, released 2026-08-08.  
**Python floor:** >=3.10.  
**Status:** Production/Stable.  
**Data model:** Mozilla/Public Suffix List (PSL) public + private sections, with configurable policy.

Sources:
- https://pypi.org/project/tldextract/5.3.2/
- https://github.com/john-kurkowski/tldextract
- https://github.com/john-kurkowski/tldextract/releases
- https://github.com/john-kurkowski/tldextract/blob/master/CHANGELOG.md

5.3.2 fixes `reverse_domain_name` edge cases and explicitly documents hostname-representation and implicit-PSL-wildcard behavior.

---

# 0. Purpose

tldextract answers:

> Given a hostname-like input and a Public Suffix List, where is the public/registry suffix boundary?

Example:

```python
import tldextract

r = tldextract.extract("https://forums.news.bbc.co.uk/")
r.subdomain  # "forums.news"
r.domain     # "bbc"
r.suffix     # "co.uk"
```

It is not:
- a strict URL validator;
- a DNS resolver;
- an IDNA canonicalizer;
- a security allowlist engine.

---

# 1. Why naive dot splitting is wrong

Registrable boundaries vary:

```text
example.com
example.co.uk
example.org.kg
```

The PSL includes multi-label suffixes and wildcard/exception rules. A regex or “last two labels” rule is incorrect.

Use PSL-aware logic.

---

# 2. Installation

```bash
pip install tldextract==5.3.2
```

Pure-Python distribution; the package includes a bundled PSL snapshot and can update from remote PSL sources.

---

# 3. One-shot API

```python
r = tldextract.extract("https://sub.example.co.uk/path")
```

Top-level `extract` uses a module-level default extractor.

For production policy control, instantiate `TLDExtract` explicitly.

---

# 4. TLDExtract constructor

Important options:

```python
tldextract.TLDExtract(
    cache_dir=...,
    suffix_list_urls=...,
    fallback_to_snapshot=True,
    include_psl_private_domains=False,
    extra_suffixes=(),
    cache_fetch_timeout=...,
)
```

These options determine data provenance and therefore extraction results.

---

# 5. Explicit production extractor

Reproducible offline-style configuration:

```python
extractor = tldextract.TLDExtract(
    suffix_list_urls=(),
    fallback_to_snapshot=True,
)
```

This disables remote fetching and uses the bundled snapshot.

For a fully application-controlled PSL:

```python
extractor = tldextract.TLDExtract(
    suffix_list_urls=["file:///app/data/public_suffix_list.dat"],
    fallback_to_snapshot=False,
    cache_dir="/app/cache/tldextract",
)
```

---

# 6. ExtractResult fields

Current fields include:

- `subdomain`;
- `domain`;
- `suffix`;
- `is_private`;
- `registry_suffix`.

`registry_suffix` was added in 5.3.x to distinguish registry/public suffix behavior from an optionally included private suffix.

---

# 7. `suffix`

`suffix` is the effective public suffix under the extractor's `include_psl_private_domains` policy.

With private domains excluded (default), a Blogspot-style hostname may produce `com` as suffix.

With private domains included, it may produce `blogspot.com`.

---

# 8. `registry_suffix`

`registry_suffix` identifies the registry-controlled suffix under which registration occurs and is intentionally unaffected by the private-domain inclusion policy.

Use it when you need a stable registrar-level boundary even if application logic treats PSL private domains specially.

---

# 9. `is_private`

`is_private` indicates that the effective matched suffix comes from the PSL private section.

If private domains are excluded, this is normally false because those private rules do not become effective suffixes.

---

# 10. top_domain_under_public_suffix

```python
r.top_domain_under_public_suffix
```

Returns the domain + effective public suffix when both exist.

This is the preferred replacement for the older ambiguous `registered_domain` name.

---

# 11. registered_domain deprecation

`registered_domain` is deprecated and scheduled for removal in the next major version.

Use:

```python
r.top_domain_under_public_suffix
```

Do not introduce new code using `registered_domain`.

---

# 12. top_domain_under_registry_suffix

```python
r.top_domain_under_registry_suffix
```

Returns the top domain under the registry suffix and is stable with respect to private-domain inclusion semantics.

This is useful when:
- tenant isolation uses PSL private suffixes;
- billing/ownership logic uses registry boundaries;
- you need both concepts simultaneously.

---

# 13. fqdn

`r.fqdn` rejoins the full extracted hostname when it represents a valid suffix-bearing domain decomposition.

Do not treat it as a canonical DNS name. The package explicitly does not normalize all hostname representations.

---

# 14. reverse_domain_name

`r.reverse_domain_name` emits reverse-domain-name notation suitable for namespace-style organization.

5.3.2 fixes stray-dot behavior when suffix/domain components are empty.

Do not use reverse-domain-name output as a cryptographic/canonical hostname identity without a separate normalization contract.

---

# 15. ExtractResult is a dataclass

Since major version 5, `ExtractResult` is a dataclass, not a namedtuple.

Do not:
```python
r[1]
a, b, c = r
```

Use named fields/properties.

This is more robust and self-documenting.

---

# 16. extract_str

Instance method:

```python
extractor.extract_str(url, include_psl_private_domains=None, session=None)
```

It accepts an optional `requests.Session` used when a PSL fetch is needed.

Application HTTP acquisition and PSL-update networking are separate concerns; a custom session lets you control the latter.

---

# 17. extract_urllib

For already parsed URLs:

```python
from urllib.parse import urlsplit

parts = urlsplit(url)
r = extractor.extract_urllib(parts)
```

Important: `extract_urllib` is an instance method, not a module-level helper. 5.3.2 documentation explicitly corrected this point.

This is preferable when your URL-validation pipeline already uses `urlsplit`.

---

# 18. URL validation boundary

tldextract intentionally accepts lenient “URL-like” strings.

Therefore:

```python
tldextract.extract(user_input)
```

must not be interpreted as:
- URL is syntactically valid;
- scheme is safe;
- hostname is DNS-valid;
- destination is public;
- host exists.

Validate before/around extraction according to application needs.

---

# 19. Unknown suffix behavior

Important 5.3.2-documented behavior:

tldextract does **not** apply the PSL's implicit wildcard fallback the same way a formal PSL consumer might when no explicit suffix matches.

For unknown/unlisted final labels, `suffix` remains empty so callers can distinguish unknown/internal/invalid names.

This is a deliberate semantic feature and should be covered by tests.

---

# 20. IP addresses

IP-like hosts are returned in the `domain` field with empty suffix rather than being treated as domain names.

Still use `ipaddress` for actual IP validation/classification.

---

# 21. localhost / internal names

Internal names can produce:
- `domain` populated;
- `suffix` empty.

Do not infer “safe internal host” from this shape. It only means no configured PSL suffix matched.

---

# 22. Public vs private PSL sections

Public section examples:
- `com`;
- `co.uk`.

Private section examples:
- hosted-service suffixes such as `blogspot.com` or `github.io`.

Private-domain choice changes tenant/security semantics.

For cookie-like/site isolation:
- private domains may be important.

For registrar ownership:
- registry suffix may be the right abstraction.

Document the choice.

---

# 23. include_psl_private_domains

Constructor default is false.

```python
extractor = tldextract.TLDExtract(
    include_psl_private_domains=True
)
```

The call can also override the choice where supported.

Avoid switching policy casually inside one data product because stored “domain” facts then become incomparable.

---

# 24. PSL data sources

`suffix_list_urls` is an ordered sequence. The implementation tries sources and uses the first successful suffix-list response rather than merging arbitrary remote lists.

Sources can include `file://` URLs.

Use explicit source configuration for reproducibility.

---

# 25. Bundled snapshot

If remote/cached data is unavailable and `fallback_to_snapshot=True`, the bundled snapshot is used.

This is excellent for resilient offline behavior but means results can lag the current PSL.

Choose:
- reproducibility;
- freshness;
- availability
as an explicit product policy.

---

# 26. Cache

By default tldextract caches suffix-list data on disk.

Configure through:
- `cache_dir`;
- `TLDEXTRACT_CACHE` environment variable.

Set `cache_dir=None` to disable disk caching.

For containers:
- decide whether cache should be baked, ephemeral, or mounted;
- do not rely on a writable home directory accidentally.

---

# 27. Cache update

Programmatically:

```python
extractor.update(fetch_now=True)
```

CLI:

```bash
tldextract --update
```

After package upgrades, the project changelog has historically recommended refreshing cache so behavior reflects expected current PSL data.

---

# 28. Cache fetch timeout

`cache_fetch_timeout` controls suffix-list fetch timeout and can be set via `TLDEXTRACT_CACHE_TIMEOUT`.

Do not let a domain-parsing operation unexpectedly block for long remote updates in latency-sensitive request paths.

For production ingestion, warm/cache PSL data outside the critical path.

---

# 29. Custom requests.Session

PSL fetching can receive a custom `requests.Session`.

Use it for:
- proxy;
- custom CA;
- auth to internal PSL mirror;
- network policy.

This session is for suffix-list acquisition, not for parsing user URLs.

---

# 30. extra_suffixes

```python
extractor = tldextract.TLDExtract(
    extra_suffixes=["corp", "service.internal"]
)
```

Use for private namespace rules that should act like suffixes.

Document them as application configuration; they alter semantic output as materially as PSL updates do.

---

# 31. tlds property

`extractor.tlds` exposes the current effective suffix list according to configured private-domain/extra-suffix policy.

Useful for:
- diagnostics;
- tests;
- cache warmup confirmation.

Avoid copying it into an ad-hoc lookup algorithm; just call the extractor.

---

# 32. IDNA / Punycode

5.3.2 explicitly documents that tldextract does not fully validate/canonicalize hostname representation.

Equivalent hostnames may appear as:
- Unicode U-labels;
- ASCII-compatible Punycode/A-labels;
- differing case.

If host equality or security decisions matter:
1. validate labels;
2. normalize to a chosen IDNA form;
3. normalize case;
4. then compare/store.

Do not use tldextract output text alone as canonical identity.

---

# 33. Unicode dots

Modern versions account for internationalized dot separators in parsing behavior, but hostname canonicalization is still the caller's responsibility.

Keep URL-parser/IDNA normalization tests separate from PSL-boundary tests.

---

# 34. Security uses and non-uses

Appropriate:
- determine registrable/site boundary as one input to policy.

Insufficient alone for:
- SSRF;
- phishing detection;
- allowlist matching;
- cookie security;
- same-site decisions;
- trust of organization ownership.

Security policy usually also needs:
- canonical hostname;
- DNS/IP resolution;
- scheme/port;
- redirect chain;
- private/public IP classification;
- certificate/identity context.

---

# 35. Reproducibility strategies

## Frozen bundled snapshot

```python
TLDExtract(suffix_list_urls=(), fallback_to_snapshot=True)
```

Behavior changes only with package version.

## Application-owned PSL file

Pin the exact PSL file hash and use `file://`.

Best when domain-boundary facts must reproduce years later.

## Auto-updating PSL

Best freshness, lower reproducibility.

Persist PSL version/hash with generated facts if results must be auditable.

---

# 36. Concurrency

The package uses cache locking mechanisms for multiprocess/thread scenarios.

Still avoid causing many workers to perform first-use remote PSL fetching simultaneously. Warm the cache/extractor before high-concurrency work.

Reuse an extractor instance in a worker/process when policy is constant.

---

# 37. Performance

PSL lookup uses an internal trie and is fast after initialization.

High-value optimizations:
- reuse `TLDExtract`;
- warm cache;
- avoid network update on hot path;
- avoid reparsing full URLs if `urlsplit` already exists—use `extract_urllib`.

---

# 38. CLI

Examples:

```bash
tldextract https://forums.bbc.co.uk
tldextract --update
tldextract --help
```

CLI is useful for diagnostics and scripts. For durable machine integration use the Python API and typed fields.

---

# 39. Data-model recommendation

For stored URL facts, keep distinct fields:

```text
raw_url
normalized_url
raw_hostname
canonical_hostname
subdomain
domain
public_suffix
registry_suffix
is_private_suffix
top_domain_under_public_suffix
top_domain_under_registry_suffix
psl_policy
psl_snapshot/version/hash
```

Do not collapse all concepts into a field named `domain`.

---

# 40. 5.3.x migration points

5.3.0:
- `registry_suffix`;
- `top_domain_under_public_suffix`;
- `top_domain_under_registry_suffix`;
- deprecation of `registered_domain`.

5.3.1:
- Python 3.9 dropped;
- Python 3.14 support.

5.3.2:
- `reverse_domain_name` stray-dot fix;
- corrected `extract_urllib` docs;
- clearer cache-dir docs;
- hostname representation limitations;
- implicit wildcard behavior clarified.

---

# 41. Testing matrix

```text
[ ] example.com
[ ] example.co.uk
[ ] deeply nested subdomain
[ ] PSL wildcard rule
[ ] PSL exception rule
[ ] private suffix excluded
[ ] private suffix included
[ ] registry_suffix same across private policy
[ ] unknown suffix
[ ] localhost
[ ] IPv4
[ ] IPv6 URL
[ ] Unicode hostname
[ ] Punycode equivalent
[ ] uppercase hostname
[ ] trailing dot if allowed by input policy
[ ] custom extra suffix
[ ] local PSL file
[ ] no-network bundled snapshot
[ ] remote fetch failure + fallback
[ ] cache update
[ ] reverse_domain_name empty-component cases
```

---

# 42. LLM-agent decision rules

Before writing domain-splitting code:
1. validate/parse URL with the URL layer;
2. normalize hostname representation if required;
3. call tldextract;
4. choose public vs registry boundary deliberately.

Never write:
```python
host.split(".")[-2:]
```
for registrable-domain logic.

---

# 43. Anti-pattern inventory

- treating `suffix` as synonymous with TLD;
- treating `domain` as full registrable domain;
- using deprecated `registered_domain`;
- allowing default remote PSL fetch in a latency-critical path unintentionally;
- comparing Unicode/Punycode output without normalization;
- using tldextract as URL validation;
- using tldextract as SSRF protection;
- changing `include_psl_private_domains` policy without versioning stored facts;
- failing to record PSL provenance in reproducibility-sensitive datasets.
