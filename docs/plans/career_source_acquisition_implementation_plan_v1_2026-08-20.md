---
artifact: implementation-plan
plan_id: career-source-acquisition
version: v1
date: 2026-08-20
status: audited
design_path: docs/designs/career_source_acquisition_design_v1_2026-08-20.md
design_version: v1
baseline_commit: 4daded6d8cc1764cd2e38491ea035468a039950b
state_path: docs/plans/state/career-source-acquisition_state.json
cutover: false
---

# Career-Source Acquisition — Implementation Plan v1

## 1. Outcome and non-goals

**Outcome.** A system that, for a declared universe of 30 Wave 1 financial-institution
employment brands, discovers every official public career source, enumerates every public
posting from it, preserves the raw bytes, normalizes postings into a canonical model, and
reconciles active/closed lifecycle **only from runs whose completeness evidence permits
it** — then answers NYC + internal-operations mandate queries over the stored set and
reports, per employer, exactly where coverage remains uncertain.

The measure of done is not module existence. It is the design's stated bar: a **credible,
auditable statement of complete public inventory capture per employer, with its uncertainty
visible**.

**Non-goals** (design §2.1, spec §15.20–21): role scoring or resume matching; application
submission; authenticated or credentialed access; LinkedIn scraping; CAPTCHA bypass,
stealth tooling, or access-control circumvention; building an adapter for every ATS that
exists.

**Cutover:** `false`. The repository contains 2 LOC of executable inheritance
(`src/jobsquery/__init__.py::main`). There is no legacy authority, no dual-write window,
and no migration. `DB01` covers the only deletion.

## 2. Source design and declared inputs

Design: `docs/designs/career_source_acquisition_design_v1_2026-08-20.md` v1, `status:
accepted` (accepted-with-named-assumptions; the seven assumptions in design §11 are carried
into packet replan triggers below).

Declared inputs — computed once at planning time; thereafter recomputed only by tooling
per `artifact-schemas.md` §8. Never hand-edited.

| path | sha256 |
|---|---|
| `docs/designs/career_source_acquisition_design_v1_2026-08-20.md` | `6031caa20b3c051a3b827dbed153d024df342c8d198e95a75bcf3184b376dce4` |
| `docs/Comprehensive Career-Source Inventory and Detailed Acquisition Design Specification.md` | `120e1f8622a6f6504bb6b2c98c834bd4a23a1f343a38a004c6da29be9c214c9d` |
| `docs/Target Career-Source Reconnaissance and Architecture Decision Plan.md` | `99162deae0325fbdfc2cd4c0cca3f1d58dc3501f1d86c65df10e31e4087a1906` |
| `docs/library_ref/semantic_design_principles_holistic.md` | `bb0f28e54f701aa932cddb59fe5d9464b304ed59443f0280377e8c4d9a9d1892` |
| `pyproject.toml` | `f3c64b3e26a2b231121fbcd8ed093b864b0297d3791018c86a04c644c4ef1cf4` |
| `uv.lock` | `387351c2b66f955ee0cc5c84d34121c4d66b85213b8f67fd956f4376bc7810b1` |

**Staleness probe.** `git rev-parse HEAD` = `4daded6…`, unchanged from the design baseline.
`rg -n '^(def|class|async def) ' src/` returns one hit.

**Drift did occur mid-planning, and it is material.** `pyproject.toml` and `uv.lock` were
modified at `20:26:42` while this plan was being written — not by the plan. The dependency
set is now **eleven** direct declarations, not the eight the design's §2.2 table recorded,
and `uv.lock` resolves **52** packages, not 44. Three additions arrived as **runtime**
dependencies of the published package:

```
pyrefly>=1.2.0    pytest>=9.1.1    ruff>=0.16.4
```

All three are installed (`pyrefly 1.2.0`, `pytest 9.1.1`, `ruff 0.16.4`). This changes
`WP01` substantially — see that packet — and supersedes design §2.2's environment table,
which is amended by reference rather than silently contradicted.

**Toolchain probe (re-run after the drift).** `just` 1.58.0, `uv` 0.12.5, `ast-grep`, and
now `ruff`/`pytest`/`pyrefly` all present. Playwright browser binaries **absent**
(`~/Library/Caches/ms-playwright` does not exist). There is still **no justfile** — the gate
registry does not exist and is created by `WP01`.

## 3. Global target invariants

The plan is bound by design §2.3's **I-01 … I-23**. Packets cite them individually; four
govern every packet and are restated because violating them is a stop condition:

- **I-01** — enumeration is unfiltered; mandate and geography are queries over stored
  postings. No packet may introduce an ingestion-time filter.
- **I-05** — every client- and page-originated request passes `HostPolicyGate`. Proven by
  the `L-01b` socket assertion, not by a name deny-list.
- **I-07 / I-09** — completeness is derived from evidence; incomplete runs never close a
  job.
- **I-21** — no credential, session, submission, or circumvention path exists anywhere.

---

## 4. Work packets

Twenty-five packets (WP19 split into three after review). Each is dependency-closed and locally provable. Load one at a time.

Two conventions apply throughout and are not repeated per packet:

- **Preflight**: this is a greenfield tree. Until a packet's dependencies have landed, its
  preflight query legitimately returns nothing. The executor runs it anyway and records
  the result — an unexpected hit means another packet has already claimed the surface.
- **Fixtures**: every fixture carries the recon §10.5 manifest entry (artifact id,
  institution, source, type, URL, capture method, timestamp, status, content type, content
  hash, library versions, redaction status, expected result, related tests). A fixture that
  no test consumes offline is not complete (recon §10.6).

---

## WP01 — Toolchain, gate registry, and executable governance

### Outcome
`just --list` prints the gate registry; `just lint`, `just typecheck`, `just test`,
`just rules`, `just contracts` exist and pass. Dev tooling is in a dev group, not shipped as
a runtime dependency of the published package. Dependency floors become exact pins. The
governance rules exist, **split by the instrument each actually needs**. The seed `main()`
is gone.

### Dependencies
None. First packet.

### Target Invariants
Enables proof of I-05, I-16, I-19, I-21 later. Design §8.1 (L-rules), §8.4 (reproducibility
depends on exact pins), LD-14, LD-17.

### Design and Library References
Design §7 G1, §8.1, §8.4, LD-14, LD-17, §6.6 legacy disposition. **Amends design §2.2's
environment table** — see §2's drift record.

### Change Surface
#### Preflight Query
```bash
rg -n '^\s*"' pyproject.toml | sed -n '/dependencies/,/]/p'   # confirm the 11 declarations
grep -c '^name = ' uv.lock                                     # expect 52
rg -n '^(def|class|async def) ' src/ -g '*.py'                 # expect exactly main()
command -v just uv ast-grep && .venv/bin/pytest --version && .venv/bin/ruff --version \
  && .venv/bin/pyrefly --version
```
#### Known Touch (verified this session)
`pyproject.toml` (11 declarations → 8 runtime pins + a dev group), `uv.lock`,
`src/jobsquery/__init__.py` (delete `main`), new: `justfile`, `rules/`, `rule-tests/`,
`sgconfig.yml`, `tests/`.

### Required Changes
1. **Re-tier three dependencies.** `pytest>=9.1.1`, `ruff>=0.16.4`, and `pyrefly>=1.2.0`
   are currently declared in `[project].dependencies` — i.e. a test framework, a linter,
   and a type checker install with the published package. Move all three to
   `[dependency-groups] dev` and add `pytest-playwright` and `hypothesis` there.
   **Do not add `pytest-asyncio`**: `anyio` 4.14.2 is already resolved and ships
   `anyio.pytest_plugin`, and httpx2 is AnyIO-based (LD-17).
2. **Exact pins.** Replace `>=` with `==` for the **eight** runtime dependencies (extruct,
   httpx2, jsonschema, lxml, playwright, pydantic, selectolax, tldextract) and promote
   `idna` to a declared direct dependency (LD-07). Dev-group entries are pinned too —
   §8.4's reproducibility claim covers the toolchain, not only the runtime.
3. **`pyrefly` is the type checker.** It is what `just typecheck` runs. Design LD-14/LD-17
   name no type checker at all, so this packet also records **LD-19** as a design amendment
   (see §9): *adopt pyrefly 1.2.0, already pinned and installed; `just typecheck` runs it
   over `src/`.* Without this, `just typecheck` is a gate cited by twelve packets that
   nothing creates.
4. **Author the justfile** as the gate registry (§7 lists the recipes). `just test` must
   tolerate **exit code 5** (`NO_TESTS_COLLECTED`) — verified: `pytest` on a tree with no
   tests returns 5, not 0. Only WP01 relies on that tolerance.
5. **Author the governance rules, split by instrument.** They are not six `ast-grep` rules:

   | Rule | Instrument | Why |
   |---|---|---|
   | `L-01` core import allow-list | `ast-grep` | per-file import matching |
   | `L-02` no bare collision token | `ast-grep` | per-file symbol matching |
   | `L-04` egress construction sites | `ast-grep` | per-file call matching |
   | `L-05`, `L-07` deny-lists | **`rg`** | design §8.1 specifies text search; `ast-grep` cannot see strings, comments, or config |
   | `L-06` acyclic import graph | **`pytest`** | a cross-file graph property; `ast-grep` is a per-file structural matcher and cannot compute a cycle |
   | `L-01b` socket-egress assertion | **`pytest`** | a runtime behavioural assertion |
   | `L-08`–`L-11` | `ast-grep` | added by WP09/WP14/WP23; registered in design §8.1 |

   Each `ast-grep` rule gets `rule-tests/` fixtures proving it fires on a planted violation
   and passes on clean code. **`L-01b` is only reserved here — it is authored in `WP06`**,
   because its definition requires the core module tree and the full fixture corpus, neither
   of which exists yet. `just rules` therefore skips `L-01b` until WP06 and the recipe says
   so explicitly rather than silently passing an empty check.
6. Delete `main()`; the CLI entry point arrives in WP22, not as a stub now.

### Legacy Disposition and Decommission
`DB01` — `src/jobsquery/__init__.py::main`. Design §6.6 disposition **replace**. Safe
immediately; nothing imports it.

### Acceptance Checks
#### Behavioral
- `just lint` — exit 0
- `just typecheck` — exit 0 (pyrefly over `src/`)
- `just test` — exit 0 **or 5**; the recipe maps 5 → 0 for this packet only
- `just rules` — exit 0
#### Structural
- `just rules` runs three `ast-grep` rules with `rule-tests/` proof, two `rg` deny-lists,
  and the `L-06` import-graph test — and reports `L-01b` as *pending WP06*
#### Negative / Zero-State
- `rg -n 'def main' src/` → zero hits (DB01)
- `python -c "import tomllib,sys; d=tomllib.load(open('pyproject.toml','rb'))['project']['dependencies']; sys.exit(any('>=' in x or '<' in x for x in d))"` — exit 0.
  A bare `rg -n '>=' pyproject.toml` would permanently hit `requires-python` and the build
  backend, so the check parses the table rather than grepping the file.
- `python -c "import tomllib,sys; d=tomllib.load(open('pyproject.toml','rb'))['project']['dependencies']; sys.exit(any(x.split('==')[0] in {'pytest','ruff','pyrefly'} for x in d))"` — exit 0; dev tooling is not a runtime dependency
#### Operational
- `uv sync --frozen` — exit 0; `uv run python -c "import jobsquery"` — exit 0

### Edit-Local Gates
`ruff format` and `ruff check` on changed files.

### Packet-Local Gates
`just lint`, `just typecheck`, `just rules`, `just test`.

### Integration Milestone
`M01`.

### Replan Triggers
`ast-grep` cannot express the L-01 allow-list over Python imports → implement it as a second
pytest import-graph assertion alongside L-06 (adaptation). `pyrefly` 1.2.0 cannot analyse
Python 3.14.7 syntax → substitute a type checker and amend LD-19 (**plan revision**).

### Rollback or Recovery
Revert the commit. No persisted state exists.

## WP02 — Contracts and schema publication  ·  **G1**

### Outcome
Every contract named in design §4.2 exists as a Pydantic v2 model with
`ConfigDict(extra="forbid")`; `just contracts` regenerates JSON Schema 2020-12 into
`contracts/` with `$schema` injected; every model round-trips through
`Draft202012Validator`; `SourceState`, `RunState`, and `Completeness` are provably disjoint.
**No network code merges before this packet is complete.**

### Dependencies
`WP01`.

### Target Invariants
I-16 (dual validation, local `$ref` only), I-12 (raw preserved beside normalized), I-13
(five-way platform split), I-11 (dual identity for overlay chains), I-18 (21-category
taxonomy), I-22, I-23. Design §4.2, §7 G1.

### Design and Library References
Design §4.2 (all contracts), §4.2's "shapes belong to the plan" list, D-1, D-2, D-5, D-6,
D-7, D-8, D-11, LD-08. Recon §7.2 (12 resolution states), §7.3 (3 confidence states), §16
(21 failure categories). Spec §10.2 (10 dimensions), §10.9 (12 anomaly rules), §16 (the
25-field `CanonicalJob`).

### Change Surface
#### Preflight Query
```bash
rg -n 'BaseModel|ConfigDict' src/ -g '*.py'         # expect zero
ls contracts/ 2>/dev/null                            # expect absent
```
#### Known Touch (verified this session)
New: `src/jobsquery/contracts/`, `contracts/` (generated), `tests/contracts/`.

### Required Changes
1. **Registry contracts**: `LegalEntity`, `EmploymentBrand`, `BrandSourceLink` (D-1: many
   brands ↔ one source), `CareerSource`, `SourceScope` as a **discriminated union**
   `UnresolvedScope | ResolvedScope` (D-8, P12).
2. **Acquisition contracts**: `ContentBlob` with **both** `wire_hash` and `decoded_hash`
   (D-2), `FetchObservation`, `CrawlRun` with `lease{holder, expires_at, lease_epoch}` —
   the fencing token is a contract field, not an operational note (design §8.2b),
   `EnumerationPage`, `SourceObservation`.
3. **`FailureCategory`** — recon §16's 21 categories, with `FailureRecord` structurally
   unconstructable without one (I-18). Placed here, not in `WP23`, so every shell packet
   from `WP07` on classifies as it goes.
4. **Judgement contracts**: `CompletenessEvidence` with **10** dimensions 1:1 with spec
   §10.2 plus `total_is_independent: bool` (design §6.4), `CompletenessVerdict` (derived
   only), `SourceAnomaly` with spec §10.9's twelve named rules, `PriorRunSummary`,
   `FailureRecord` over the 21 categories.
5. **Output contract**: `CanonicalJob` — spec §16's 25 fields, with `locations_raw[]`
   beside `locations[]`, `compensation_raw` beside `compensation`, `observations[]` as a
   foreign key (I-03), and `mandate_matches[]`/`geo_class` marked derived-recomputable.
6. **The sixteen plan-shaped contracts** named in design §4.2: `DiscoveryEvidence`,
   `ProbeResult`, `TruncationSignal`, `SourceJobRef`, `JobIdentity`, `LifecycleTransition`,
   `MandateMatch`, `ExtractContext`, `RunEvidence`, `Fetcher`, `RobotsDecision`,
   `SourceState`, `RunState`, `FieldAvailability`, `ProbeVocabulary`, and recon §7.2/§7.3's
   `resolution_state` / `resolution_confidence`.
7. **Minimum-viable-vocabulary pass** (design §6.5 qualification 2): each *enumeration* is
   reduced to members the anchor batch will exercise; the remainder are recorded as
   `reserved` and admitted only by schema version. **Invariants are not reduced.**
8. `just contracts` emits `contracts/*.schema.json` with `$schema` injected (Pydantic's
   `model_json_schema()` omits it — LD-08), using `mode="serialization"` for published
   shapes.

### Legacy Disposition and Decommission
None. Recon §9's observation contract is **reshaped, not deleted** (design §6.6): §9.3's
field names win, and the deliverable survives as a view (WP16).

### Acceptance Checks
#### Behavioral
- **`tests/contracts/test_registry_complete.py::test_all_named_contracts_exist`** — driven
  by a **literal list** of every contract named in §4.2 and in its "shapes belong to the
  plan" list. Design §7's G1 is explicit that the exit condition "enumerates them
  individually — it does not say 'all §4.2 contracts', which would have passed with the
  output contract undefined." A registry-iterating test is green if `CanonicalJob` was never
  written; this one is not.
- `tests/contracts/test_roundtrip.py::test_every_model_validates_under_both_engines` — each
  model's instances validate under Pydantic **and** `Draft202012Validator`
- `tests/contracts/test_scope_union.py::test_unresolved_scope_cannot_claim_completeness` —
  the union makes the illegal state a type error, not a flag
#### Structural
- `just rules` — `L-02` proves `SourceState`/`RunState`/`Completeness` share no bare token
- `just contracts` — regenerates cleanly; `git diff --exit-code contracts/` is empty
#### Negative / Zero-State
- `tests/contracts/test_no_network_refs.py` — no schema resolves a remote `$ref`; the
  `referencing` registry is local-only
- `rg -n 'registered_domain' src/` → zero hits (deprecated, LD-06)
#### Operational
- `just contracts && just test` — exit 0

### Edit-Local Gates
`ruff check` on changed files; `uv run python -c "from jobsquery.contracts import *"`.

### Packet-Local Gates
`just contracts`, `just test`, `just rules`, `just typecheck`.

### Integration Milestone
`M01` — **G1 closes here.**

### Replan Triggers
Pydantic 2.13.4 cannot express the `SourceScope` discriminated union with the required
JSON-Schema output (fall back to a sealed base class + validator, recorded as adaptation).
The `jsonschema` `format` gap (LD-08) proves load-bearing → add the `jsonschema[format]`
extra, which is a **plan revision** because it adds a dependency.

### Rollback or Recovery
Revert. `contracts/` is generated, not authored.

### Design-Bearing Contracts and Exemplars
```python
# The one signature the whole product depends on (design §4.2, D-11).
def classify(evidence: CompletenessEvidence,
             prior: PriorRunSummary | None,
             anomalies: SourceAnomaly) -> CompletenessVerdict: ...
# closure_allowed is False whenever prior is None — a first run cannot close a job.
```

---

## WP03 — URL and hostname policy (pure core)

### Outcome
One canonicalization function turns any URL into a stable identity: A-label host, PSL
decomposition with recorded provenance, and an SSRF verdict on a resolved address. Golden
tests cover IDN, Punycode, private suffixes, unknown suffixes, IPs, and the CGNAT trap.

### Dependencies
`WP02`.

### Target Invariants
I-20 (PSL policy + snapshot recorded with every domain fact), I-05 (the predicate the gate
enforces), I-02.

### Design and Library References
Design §9.2 (trust boundary table), C-9, LD-06, LD-07, L-07. `tldx §5`, `§8`, `§19`,
`§26`, `§39`; `httpx2 §5`.

### Change Surface
#### Preflight Query
```bash
rg -n 'tldextract|TLDExtract|raw_host|ip_address' src/ -g '*.py'   # expect zero
```
#### Known Touch (verified this session)
New: `src/jobsquery/core/urlpolicy.py`, `tests/core/test_urlpolicy.py`.

### Required Changes
1. `canonical_host(url) -> ALabel` = `str(httpx2.URL(url).raw_host, "ascii")`. **Never**
   `encodings.idna` — it maps `faß.de` → `fass.de`, a different registrable domain (LD-07,
   enforced by `L-07`). Where label validity is contractual, call
   `idna.encode(host, uts46=True)`, because pure-ASCII hosts bypass httpx2's IDNA path.
2. `decompose(a_label) -> DomainFacts` using a **module-level singleton**
   `TLDExtract(suffix_list_urls=(), fallback_to_snapshot=True, cache_dir=<pinned>)`. The
   default `cache_dir` embeds the Python version and a venv hash and is therefore
   non-reproducible (LD-06). Store the twelve fields of `tldx §39`, never one `domain`
   column. Empty `suffix` for an unknown label is a **signal**, not an error (`tldx §19`).
3. `is_egress_allowed(resolved: ip_address) -> bool` = **`not addr.is_global`** rejects.
   `is_private` is insufficient: `100.64.0.1` (RFC 6598 CGNAT) is neither private nor
   global (design §9.2).
4. Scheme allow-list and embedded-credential rejection.

### Legacy Disposition and Decommission
None.

### Acceptance Checks
#### Behavioral
- `tests/core/test_urlpolicy.py::test_idna_golden` — the recon §41-style matrix: IDN host,
  Punycode equivalent, uppercase, trailing dot, private suffix (`github.io`), unknown
  suffix, `localhost`, IPv4, IPv6
- `::test_cgnat_and_metadata_rejected` — `100.64.0.1`, `169.254.169.254`, `192.0.0.170`,
  `::1` all rejected
- `::test_psl_provenance_recorded` — every `DomainFacts` carries PSL policy + snapshot id
- Hypothesis property: `canonical_host` is idempotent under re-parsing and agrees with
  `idna.encode(uts46=True)`
#### Structural
- `just rules` — `L-01` (core allow-list) and `L-07` both pass
#### Negative / Zero-State
- `rg -n "encodings.idna|\.encode\(.idna.\)" src/` → zero hits
- `tests/core/test_urlpolicy.py::test_stdlib_idna_still_diverges` — asserts
  `'faß.de'.encode('idna') != idna.encode('faß.de', uts46=True)`, so the ban stays justified
#### Operational
- No network: `L-01b` covers this module in WP06's assertion scope.

### Edit-Local Gates
`ruff check`; one unit test.

### Packet-Local Gates
`just test tests/core/test_urlpolicy.py`, `just rules`, `just typecheck`.

### Integration Milestone
`M02`.

### Replan Triggers
`httpx2.URL.raw_host` behaviour differs from the probed 2.12.0 result on any Wave 1 host.

### Rollback or Recovery
Revert. Pure module, no persisted state.

---

## WP04 — Robots policy port and RFC 9309 wrapper

### Outcome
`RobotsPolicy.decide(origin, path_and_query, product_token, fetch_outcome) ->
RobotsDecision` correctly answers the nine-case RFC 9309 suite **and** the four defects
adversarial probing found in the stdlib parser. The RFC 9309 §2.3.1 status matrix is owned
here, not by the library.

### Dependencies
`WP03`.

### Target Invariants
I-05 (robots is a gate concern), I-02, I-21.

### Design and Library References
Design LD-10 (the nine-case table and the four defects), C-7, §7 G2. Recon ADR-07.

### Change Surface
#### Preflight Query
```bash
rg -n 'robotparser|RobotFileParser|Protego' src/ -g '*.py'   # expect zero
```
#### Known Touch (verified this session)
New: `src/jobsquery/shell/robots.py`, `tests/shell/test_robots.py`.

### Required Changes
1. Wrapper over `urllib.robotparser` owning: **BOM strip** (the one hard RFC failure —
   a leading `﻿` makes the file parse as empty and the result allow-all); **exact
   longest product-token group resolution** before delegating (the stdlib does substring
   containment, so `mybot` matches a `User-agent: bot` group); **fragment stripping**
   before comparison (a fragment makes `Disallow: /a$` fail **open**); **reserved
   percent-encoding preservation** (`normalize()` collapses `%2F` into `/`, producing a
   false `BLOCKED`); **fractional `Crawl-delay`** parsing; a **500 KiB** input bound
   (RFC 9309 §2.5).
2. **The origin parameter is why this port shape exists.** `can_fetch(ua, url)` judges any
   URL against whatever robots.txt was last parsed — so a cross-host redirect is evaluated
   against the wrong origin, failing **open**. `decide()` takes the origin explicitly.
3. **Own the §2.3.1 status matrix**: 401/403 → deny-all, other 4xx → allow-all, 5xx →
   deny-all, plus timeout, oversize, and "robots.txt returned HTML". The wrapper must not
   call `RobotFileParser.read()` — it uses `urllib.request.urlopen`, a second HTTP stack
   that bypasses every gate control (I-05).
4. Decision cache with a TTL.

### Legacy Disposition and Decommission
None.

### Acceptance Checks
#### Behavioral
- `tests/shell/test_robots.py::test_rfc9309_conformance` — all nine cases from design LD-10
- `::test_four_stdlib_defects` — BOM, fragment fail-open, `%2F` collapse, substring UA
- `::test_cross_origin_redirect_uses_correct_robots` — the port-shape defect
- `::test_status_matrix` — 401/403/404/500/timeout/oversize/HTML-body
#### Structural
- `just rules` — `L-04` proves no module outside the gate calls `urllib.request.*`
#### Negative / Zero-State
- `rg -n 'RobotFileParser\(\)\.read\(|\.read\(\)' src/jobsquery/shell/robots.py` → zero hits
- `rg -n 'protego' pyproject.toml uv.lock` → zero hits (LD-10 rejects it)
#### Operational
- Robots fetch is routed through the gate in WP06; asserted there.

### Edit-Local Gates
`ruff check`; the conformance test.

### Packet-Local Gates
`just test tests/shell/test_robots.py`, `just rules`.

### Integration Milestone
`M02`.

### Replan Triggers
A Wave 1 `robots.txt` exercises a stdlib defect not in the suite → adopt Protego behind the
unchanged port (a **plan revision**: adds a dependency; the port shape does not change).
This is design assumption **A-4**, resolved by `WP17`/SPK-08.

### Rollback or Recovery
Revert. The port is one method; no consumer knows the implementation.

---

## WP05 — Content store (content-addressed, immutable)

### Outcome
`ContentStore.put(wire_bytes, headers) -> ContentBlob` writes zstd-framed bytes once,
addressed by `wire_hash`, and returns both hashes. Re-putting identical content is a no-op.
Nothing mutates a stored blob.

### Dependencies
`WP02`.

### Target Invariants
I-03 (immutable artifact before any parsing; every canonical job traces to it), I-19
(idempotent by construction), I-16.

### Design and Library References
Design §4.5, D-2 (both hashes), LD-09 (stdlib only), §8.4.

### Change Surface
#### Preflight Query
```bash
rg -n 'blake2b|compression.zstd|ContentBlob' src/ -g '*.py'   # expect zero
```
#### Known Touch (verified this session)
New: `src/jobsquery/shell/store.py`, `tests/shell/test_store.py`.

### Required Changes
1. `wire_hash = blake2b(wire_bytes, digest_size=32)` — the identity and immutability
   anchor, and the §8.4 reproducibility oracle. `decoded_hash` over post-content-encoding
   bytes — the dedup key, stable when a CDN switches gzip→br. **A single hash fails dedup
   on any content-encoding change** (D-2).
2. `compression.zstd` (stdlib, PEP 784, verified on 3.14.7) — **not** the `zstandard`
   package (LD-09).
3. Safe-header allow-list on persist; browser storage state never stored (design §9.2,
   recon §10.4 redaction list).
4. Write-once semantics: a second `put` of identical `wire_hash` returns the existing blob
   without rewriting.

### Legacy Disposition and Decommission
None.

### Acceptance Checks
#### Behavioral
- `tests/shell/test_store.py::test_roundtrip_byte_identical` — zstd round-trip is exact
- `::test_dedup_survives_encoding_change` — same document served gzip then br yields **two**
  `wire_hash` values and **one** `decoded_hash`
- `::test_put_is_idempotent` — I-19
#### Structural
- `just rules` — `L-05` proves no credential/storage-state persistence path
#### Negative / Zero-State
- `rg -n 'import zstandard|from zstandard' src/` → zero hits
- `::test_blob_is_immutable` — a write attempt to a stored blob raises
#### Operational
- `::test_store_bounds_input` — a blob exceeding the configured ceiling is rejected before
  write

### Edit-Local Gates
`ruff check`; the round-trip test.

### Packet-Local Gates
`just test tests/shell/test_store.py`, `just rules`.

### Integration Milestone
`M02`.

### Replan Triggers
`compression.zstd` unavailable on the deployment interpreter → fall back to `lzma`
(adaptation) or add `zstandard` (revision).

### Rollback or Recovery
Revert. Blobs written during development are content-addressed and orphan safely.

---

## WP06 — HostPolicyGate and the resolving transport

### Outcome
A single `HostPolicyGate` is the only way to reach the network. It resolves a hostname,
validates the **resolved address**, applies robots, acquires a per-host rate slot, and only
then connects. `L-01b` proves the core makes zero socket calls under the full fixture
corpus.

### Dependencies
`WP03`, `WP04`, `WP05`.

### Target Invariants
**I-05** (the packet that makes it true), I-02, I-21, I-20.

### Design and Library References
Design §4.1, §9.2, C-2, C-8, C-9, L-01, L-01b, L-04, LD-01, LD-16. `httpx2 §20` (custom
transports), `§31` (security playbook), `§13` (pool as saturation signal).

### Change Surface
#### Preflight Query
```bash
rg -n 'httpx2\.(Async)?Client\(|AsyncBaseTransport|getaddrinfo' src/ -g '*.py'
ast-grep --pattern 'httpx2.$_(...)' src/            # locate any construction site

# BLOCKING PROBE — run and record BEFORE any edit in this packet.
# I-05's entire mechanism assumes a custom AsyncBaseTransport can observe and gate the
# RESOLVED address before connect. LD-01 probed Client/Limits/Timeout/MockTransport/URL/
# event_hooks/num_bytes_downloaded — it never probed this. `httpx2 §20` documents custom
# transports as request handlers; connection establishment lives in httpcore2, and
# `httpx2 §31` says plainly that httpx2 "is not an SSRF firewall".
.venv/bin/python - <<'EOF'
import inspect, httpx2, httpcore2
print(inspect.getsource(httpx2.AsyncHTTPTransport.handle_async_request)[:2000])
print([n for n in dir(httpcore2) if 'ackend' in n or 'onnect' in n])
EOF
```
If the probe shows no pre-connect seam, **stop and escalate** — see Replan Triggers.
#### Known Touch (verified this session)
New: `src/jobsquery/shell/gate.py`, `src/jobsquery/shell/ratelimit.py`,
`src/jobsquery/core/ratelimit.py`, `tests/shell/test_gate.py`, `tests/test_egress.py`.

### Required Changes
1. **Custom `AsyncBaseTransport`** that resolves → validates (`WP03.is_egress_allowed`) →
   connects. httpx2 exposes **no pre-connect hook**, so this cannot be a client option, and
   `MockTransport` cannot prove it because it replaces the transport and never resolves
   (design §8.2). Validation is re-applied on **every** redirect hop.
2. Per-host **rate** limiting (C-8, LD-16): a pure `next_allowed_at(host, now, history) ->
   float` in core with a thin async wrapper. `anyio.CapacityLimiter` handles per-host
   **concurrency** — the two are different mechanisms and conflating them is why crawlers
   either idle or hammer. Honours `Crawl-delay` (WP04) and `Retry-After` (WP07).
3. Robots evaluated through `WP04`'s port, with the robots fetch itself passing through
   this gate.
4. **`L-01b` — the socket-level egress assertion.** A test monkeypatches `socket.socket`
   and `socket.getaddrinfo`, runs the entire core module tree over the fixture corpus, and
   asserts **zero** calls. This is the only provable form of I-05: a name deny-list cannot
   see `extruct`'s transitive `rdflib`/`requests`/`urllib3`, and `rdflib.Graph.parse(
   location=…)` is live egress.

### Legacy Disposition and Decommission
None.

### Acceptance Checks
#### Behavioral
- `tests/shell/test_gate.py::test_ssrf_resolved_address` — a **public hostname resolving to
  a private address** is rejected, using a stub resolver. This is the actual risk; a literal
  private address is the easy case.
- `::test_redirect_hop_revalidated` — a redirect to a private target is rejected mid-chain
- `::test_rate_limit_honours_fractional_crawl_delay` — `Crawl-delay: 4.5` waits 4.5s
- Hypothesis property: no host exceeds its configured rate over any window
#### Structural
- `just rules` — `L-04` proves only `shell.gate` constructs a client
#### Negative / Zero-State
- **`tests/test_egress.py::test_core_makes_zero_socket_calls`** (`L-01b`) — the load-bearing
  negative proof for I-05
- `::test_gate_revoked_fails_closed` — with the gate revoked, every fetcher raises rather
  than falling back to a direct client
- **`::test_validated_address_is_the_connected_address`** — the address the policy approved
  is the address the socket connects to. Without this, a re-resolve between validate and
  connect (DNS rebinding, which `httpx2 §31` names explicitly as the caller's problem) walks
  straight through a stub-resolver test that looks green.
#### Operational
- `::test_pool_timeout_is_not_retried` — `PoolTimeout` is local saturation (`httpx2 §13`)

### Edit-Local Gates
`ruff check`; `test_ssrf_resolved_address`.

### Packet-Local Gates
`just test tests/shell/ tests/test_egress.py`, `just rules`, `just typecheck`.

### Integration Milestone
`M02`.

### Replan Triggers
The preflight probe shows no pre-connect seam in a custom `AsyncBaseTransport` on httpx2
2.12.0. Two escalations, and neither is an adaptation:
- drop to an `httpcore2` network backend → **plan revision**, because it promotes
  `httpcore2` from transitive (`uv.lock`) to a **direct** dependency, and §9 classifies a
  dependency change as a revision, not an adaptation;
- enforce egress at the container instead → **design reopening** (assumption **A-3**),
  because I-05's stated mechanism moves.

This is the most load-bearing invariant in the system resting on the one API the design
never probed, which is why the probe is blocking rather than advisory.

### Rollback or Recovery
Revert. No persisted state.

---

## WP07 — HttpFetcher: streaming, conditional requests, retry, decoding

### Outcome
`HttpFetcher.fetch(request) -> FetchObservation` streams with a byte ceiling, honours
conditional requests, classifies failures narrowly, retries only what is retryable, and
hands correctly-decoded text to the parser layer. The full `MockTransport` matrix passes.

### Dependencies
`WP06`.

### Target Invariants
I-03 (artifact written before parsing), I-19, I-18 (narrow failure classification), I-10.

### Design and Library References
Design LD-01, LD-12, LD-15, C-1, C-3, C-4, C-11, §8.2. `httpx2 §9`–`§13`, `§24`, `§25`,
`§26`.

### Change Surface
#### Preflight Query
```bash
rg -n 'client.stream|iter_bytes|If-None-Match|Retry-After' src/ -g '*.py'   # expect zero
```
#### Known Touch (verified this session)
New: `src/jobsquery/shell/http.py`, `src/jobsquery/core/retry.py`,
`tests/shell/test_http_matrix.py`.

### Required Changes
1. One long-lived `AsyncClient` per policy boundary with `Limits` and four-component
   `Timeout` (LD-01). `follow_redirects=True` **plus a response event hook that raises on a
   disallowed hop** — the manual `next_request` walk yields the veto but **drops
   `response.history`**, and per-hop provenance needs both (C-2).
2. Byte ceiling enforced with `Response.num_bytes_downloaded` during the stream (LD-01).
3. **Conditional-request store (C-1)**: httpx2 has no cache layer. Persist
   ETag/`Last-Modified`, send `If-None-Match`/`If-Modified-Since`, handle 304 as a normal
   response. `hishel` is rejected — it targets `httpx`, not `httpx2` (LD-01).
4. **Decompression-expansion bound (C-3)** — built-ins stop at 1 MiB/chunk and 5 chained
   encodings, which is not a total-expansion bound.
5. **Charset resolution before parsing (C-4)**: Lexbor ignores `<meta charset>` on bytes
   input, so decode first, wiring `Client(default_encoding=<callable>)` to a sniffer. The
   **original bytes still go to the store** (I-03).
6. **Retry policy (C-11, LD-12)**: pure `decide(attempt, outcome, now, jitter) -> Retry |
   Stop` in core with **injected entropy and clock** — jitter is nondeterministic and must
   not hide inside a "pure" function (design §4.1). `PoolTimeout` never retries.
7. Add the `httpx2[brotli]` extra (LD-15) — the default `Accept-Encoding` omits `br`.

### Legacy Disposition and Decommission
None.

### Acceptance Checks
#### Behavioral
- `tests/shell/test_http_matrix.py` — the design §8.2 matrix via `MockTransport`: redirect
  chains, 304, 408, 429 + `Retry-After`, 500/503, read timeout, connect failure, oversized
  body, unexpected content type, compressed bodies, duplicate query params, conditional
  headers, stream closure
- `::test_retry_after_honoured` — `Retry-After: 120` produces one **delayed** retry
- `::test_charset_latin1_no_mojibake` — the decode step over the C-4 fixture set (Latin-1,
  UTF-8 ± BOM, declared-vs-actual mismatch). **This proves the decode, not the parser
  defect**; the parser-level oracle lives in `WP09`
#### Structural
- `just rules` — `L-04` still passes
#### Negative / Zero-State
- **SSRF is deliberately absent from this matrix** — `MockTransport` never resolves. It is
  proved in `WP06` against the resolving transport with a stub resolver.
- `rg -n 'hishel|tenacity' pyproject.toml uv.lock` → zero hits
#### Operational
- `::test_byte_ceiling_aborts_stream` — an oversized body aborts mid-stream, not after

### Edit-Local Gates
`ruff check`; one matrix case.

### Packet-Local Gates
`just test tests/shell/`, `just rules`, `just typecheck`.

### Integration Milestone
`M02`.

### Replan Triggers
`Client(default_encoding=)` does not accept a callable on 2.12.0 → decode outside the client
(adaptation). **Also unprobed:** `Response.charset_encoding` appears nowhere in the httpx2
2.9.1 reference and was not in LD-01's probe set — verify it on 2.12.0 in this packet's
preflight before relying on it for C-4.

### Rollback or Recovery
Revert. The conditional-request store is a cache; dropping it costs bandwidth, not
correctness.

---

## WP08 — BrowserFetcher  ·  **G2**

### Outcome
Playwright runs async-only, one browser per worker, an ephemeral context per source, every
in-page request routed through `HostPolicyGate`, service workers blocked in **production**,
HAR recorded, trace on failure, and no storage state persisted.

### Dependencies
`WP06`, `WP07`.

### Target Invariants
I-05, I-06 (browser requires recorded justification), I-21, I-03.

### Design and Library References
Design LD-02, §4.6 (async-only, ownership table), §9.2. `pw §2`, `§6`, `§7`, `§26`, `§27`,
`§31`, `§39`, `§48`.

### Change Surface
#### Preflight Query
```bash
rg -n 'sync_playwright|storage_state|new_context' src/ -g '*.py'
```
#### Known Touch (verified this session)
New: `src/jobsquery/shell/browser.py`, `tests/shell/test_browser_routing.py`.

### Required Changes
1. **`playwright.async_api` exclusively.** `sync_playwright()` inside a running loop raises
   — the sync API is a greenlet shim over the same core (design §4.6). If a sync context is
   ever unavoidable it goes in a separate **process**, never a thread.
2. `context.route("**/*")` → the same `HostPolicyGate` decision → `route.abort()` on deny.
3. **`service_workers="block"` on the production context** — not only in HAR tests. Service
   workers bypass route interception (`pw §26`). An earlier design draft blocked them only
   where it was easy.
4. Ephemeral context per source investigation, closed after; fixed viewport/locale/timezone;
   **storage state never persisted** (`pw §31`, I-21).
5. HAR record with a URL filter and `record_har_content`; tracing started on every run and
   **retained only on failure** (`pw §39`).
6. `page.content()` is the sole handoff to the core — and is post-DOM serialization, not
   network bytes (I-03 keeps the network bytes separately).
7. Browser binary installation is a **G2 deployment step**: `just browsers` runs
   `python -m playwright install --with-deps chromium`. Package and binary versions are
   coupled (`pw §1`).

### Legacy Disposition and Decommission
None.

### Acceptance Checks
#### Behavioral
- `tests/shell/test_browser_routing.py::test_route_denial_aborts` — a denied host aborts;
  the page never reaches it
- `::test_har_replay_deterministic` — `route_from_har(not_found="abort")` replays a captured
  listing workflow with no live network
#### Structural
- `just rules` — `L-04` proves only `shell.gate`/`shell.browser` call `new_context`
#### Negative / Zero-State
- `rg -n 'sync_playwright' src/` → zero hits
- `rg -n 'storage_state\(' src/` → zero hits (I-21, `L-05`)
- `::test_service_workers_blocked_in_production` — the production context factory sets
  `service_workers="block"`
#### Operational
- `just browsers` — exit 0; `::test_browser_version_recorded` — the fixture manifest carries
  the browser version (recon §10.5)

### Edit-Local Gates
`ruff check`; the routing test.

### Packet-Local Gates
`just test tests/shell/test_browser_routing.py`, `just rules`.

### Integration Milestone
`M02` — **G2 closes here.**

### Replan Triggers
`context.route()` proves not to intercept some egress class (navigation, preflight) →
design assumption **A-3** fails and I-05's enforcement moves to container egress policy: a
**design reopening**, not an adaptation.

### Rollback or Recovery
Revert. Contexts are ephemeral; nothing persists.

---

## WP09 — Pure extraction primitives

### Outcome
`extract()` turns stored bytes into typed records with zero I/O: HTML via selectolax, XML
and sitemaps via lxml, embedded metadata via extruct with per-block JSON-LD resilience and
our own URL resolution. Every primitive is proved from the six reference documents' own
fixture corpora.

### Dependencies
`WP02`, `WP07` (for the charset contract).

### Target Invariants
I-04 (parsing cannot acquire), I-12 (raw preserved), I-22 (metadata never proves
enumeration), I-03.

### Design and Library References
Design LD-03, LD-04, LD-05, C-5, C-6, §4.1. `slax §5`/`§6`/`§13`/`§14`/`§29`/`§38`;
`lxml §6`/`§11`/`§17`/`§29`/`§54`; `extruct §4`/`§16`/`§18`/`§28`. Spec §11.

### Change Surface
#### Preflight Query
```bash
rg -n 'LexborHTMLParser|iterparse|extruct.extract|JsonLdExtractor' src/ -g '*.py'
```
#### Known Touch (verified this session)
New: `src/jobsquery/core/extract/`, `tests/core/extract/`, `tests/fixtures/`.

### Required Changes
1. **HTML (LD-03)**: `LexborHTMLParser` on **decoded `str`** (Lexbor ignores
   `<meta charset>` on bytes — probed). `css_first(strict=True)` for identity-critical
   fields. `text(deep, separator, strip, skip_empty)` as an explicit policy, noting
   `separator` defaults to `''` and `text()` **includes `<script>`/`<style>` content** —
   scope the selector or `strip_tags` first. `merge_text_nodes()` after `unwrap_tags()`.
   No parser node escapes the call frame (`slax §8`).
2. **XML (LD-04)**: one shared parser factory —
   `XMLParser(resolve_entities=False, load_dtd=False, no_network=True, huge_tree=False)` —
   as the **only** construction site. `iterparse` with element clearing for large sitemaps.
   Compiled `etree.XPath` objects reused. Structured `error_log` retained, never
   `str(exc)`. **lxml 6.1.2 defaults `decompress=False`** — a gzipped sitemap must be
   wrapped in `gzip.GzipFile`, which is desirable because it keeps decompression under our
   byte budget.
3. **Metadata (LD-05)** — two probed corrections that are the substance of this packet:
   - **Per-block JSON-LD (C-5).** `errors="log"` does **not** isolate a bad block: extruct
     materializes `list(extract(...))` per syntax, so one malformed `<script>` drops the
     **entire** `json-ld` key and every valid block with it. Drive
     `JsonLdExtractor().extract_items()` per `<script>` node. Always read
     `result.get(syntax, [])`, never `result[syntax]`.
   - **URL resolution (C-6).** `base_url` resolves microdata `itemprop` and RDFa `@id`, but
     **not JSON-LD, not Open Graph, not microdata `itemid`** — and JSON-LD is the primary
     `JobPosting` path. Resolve with `httpx2.URL(final_url).join(v)`, keeping raw and
     resolved side by side (I-12).
   - Syntaxes are opt-in; the default enables all six including RDFa, which pulls
     `rdflib`/`pyrdfa3` and is experimental (`extruct §13`).
4. Description sectioning and location extraction per spec §11.5/§11.6 — **all** locations
   preserved as a list, raw never replaced (I-12).
5. **RSS and Atom parse as namespaced XML through lxml** — `feedparser` stays deferred
   (LD-13). It is added only if a Wave 1 source produces a feed lxml cannot parse, and then
   as a recorded coverage-gap entry rather than a silent workaround.

### Legacy Disposition and Decommission
None.

### Acceptance Checks
#### Behavioral
- `tests/core/extract/test_jsonld.py::test_malformed_block_isolated` — **three blocks, the
  middle one malformed → both valid blocks returned plus one recorded
  `structured_data_conflict`.** The naive `errors="log"` implementation demonstrably fails
  this; it is the packet's defining test.
- `::test_jsonld_urls_resolved` — relative `@id`, image, and apply URLs resolve against the
  final URL, with raw values retained
- `tests/core/extract/test_html.py` — the `slax §38` 16-item fixture corpus
- `tests/core/extract/test_xml.py` — the `lxml §54` matrix incl. external-entity denial and
  a bounded-memory 100k-entry sitemap
- `::test_locations_never_collapsed` — I-12
- **`::test_lexbor_bytes_vs_str_diverges`** — the golden for LD-03's headline probe: the
  *same* ISO-8859-1 document with `<meta charset>` yields `caf\ufffd` from bytes and `café`
  from `str`. `WP07`'s decode test is a decode-level assertion in a packet that owns no
  parser; this is the only place the defect is observable, and without it C-4 has no oracle.
#### Structural
- `just rules` — `L-01` (core allow-list) passes with `extruct` in scope
- **L-08**: `etree.XMLParser(` appears only in the shared factory
- **L-09**: `LexborHTMLParser(` is constructed only from a `str`-typed value
  — nothing structurally forbade passing bytes, which is how the mojibake gets in
#### Negative / Zero-State
- `::test_external_entity_denied` — an XXE fixture yields no external content
- `rg -n "extruct.extract\(.*uniform=True" src/` → zero hits (LD-05)
- `rg -n "result\[[\"']json-ld[\"']\]" src/ ; test $? -eq 1` — quote-agnostic; must use `.get`
#### Operational
- `tests/test_egress.py` (`L-01b`) now covers `extruct`'s transitive `rdflib`/`requests` —
  **zero socket calls** over the whole fixture corpus

### Edit-Local Gates
`ruff check`; the malformed-block test.

### Packet-Local Gates
`just test tests/core/extract/`, `just rules`, `just typecheck`.

### Integration Milestone
`M03`.

### Replan Triggers
`JsonLdExtractor.extract_items()` is not importable or its signature differs on extruct
0.18.0 → hand-parse `<script>` nodes and `json.loads` each (adaptation; the per-block
requirement is unchanged).

### Rollback or Recovery
Revert. Pure module.

---

## WP10 — Completeness evaluator  ·  **G3 core**

### Outcome
`classify(evidence, prior, anomalies) -> CompletenessVerdict` is the single owner of every
completeness judgement. `signal()` implements all eleven §10.4 safeguards as pure
predicates. Both A–X grades and the eight classes are projections of one record.

### Dependencies
`WP02`.

### Target Invariants
**I-07** (derived, never authored), **I-08** (no silent zero), **I-09** (incomplete runs
cannot close), I-10 (enumeration vs content are separate verdicts), I-17, I-22, I-23.

### Design and Library References
Design §4.2 (`classify` signature, D-11), §6.4 (`total_is_independent`), §8.2, D-4, D-6.
Spec §10.1–§10.9. Recon §8 stage 6 (A–X grades).

### Change Surface
#### Preflight Query
```bash
rg -n 'CompletenessEvidence|classify\(|assess_completeness' src/ -g '*.py'
```
#### Known Touch (verified this session)
New: `src/jobsquery/core/completeness.py`, `tests/core/test_completeness.py`.

### Required Changes
1. `signal(page, history) -> TruncationSignal | None` — the eleven §10.4 safeguards as
   **distinct** signals: repeated cursor, repeated page URL, repeated ordered ID
   fingerprint, non-advancing offset, empty middle page, page-count safety bound,
   advertised total reached early, advertised total exceeded, cross-page duplicates, sort
   instability, mid-run insertion. A repeated cursor and an exceeded total must not collapse
   into a shared "anomaly".
2. `anomalies(current, prior) -> SourceAnomaly` — spec §10.9's **twelve** comparative rules.
3. `classify(evidence, prior, anomalies) -> CompletenessVerdict`. **Cross-run by
   signature**: §10.8's closure predicate needs *source not anomalous* and *identity
   unchanged*, both of which are comparative. `closure_allowed` is `False` whenever
   `prior is None` (D-11).
4. `EXACT` additionally requires `total_is_independent` — a count serialized in the same
   response as the array it counts is not independent evidence, and would otherwise grant
   the system's strongest claim on self-consistent data (design §6.4).
5. The **single-sourced projection table** mapping evidence → eight classes → A–X grades.
   P10 requires the rule to exist once; prose is not once (design §10).
6. Valid-zero predicate per spec §10.5 as resolved in design §3 — including the `TERMINAL`
   accommodation, since a source with no trustworthy total can still legitimately be empty.

### Legacy Disposition and Decommission
None. This packet **replaces** the proposal's per-adapter `assess_completeness` (D-4)
before any adapter exists, so no adapter ever ships with self-certification.

### Acceptance Checks
#### Behavioral
- `tests/core/test_completeness.py::test_exact_requires_independent_total` — Hypothesis:
  `EXACT` is unreachable without an advertised total that is independent, equals unique-ID
  count, with pagination exhausted and no truncation signal
- `::test_closure_requires_all_five_conjuncts` — Hypothesis over §10.8
- `::test_closure_false_without_prior` — D-11; a first run cannot close
- `::test_content_refresh_does_not_gate_reconciliation` — §10.3
- `::test_eleven_signals_are_distinct` — one fixture per safeguard, asserting distinct
  signal identity
- `::test_projection_table_total` — every one of the eight classes is reached by at least
  one evidence vector, and each A–X projection asserted
- **`::test_twelve_anomaly_rules_distinct`** — one fixture per spec §10.9 rule, each
  producing a distinct member. WP10.2 builds the whole cross-run anomaly machinery — the
  machinery D-11 exists to supply — and an earlier draft proved none of it.
- **`::test_vendor_change_blocks_closure_and_degrades`** — design §8.2b names an ATS vendor
  change between runs "the design's most damaging realistic failure": it trips
  `branding_changed`, `source_identifier_missing`, `json_shape_changed` and
  `all_source_ids_changed` at once, every job appears closed and an identical set appears
  new. The guard must block closure and mark the source `DEGRADED`. §8.2b asserted this was
  "now testable"; this is the test.
#### Structural
- `just rules` — `L-01` (pure core), `L-02` (`Completeness` tokens namespaced)
#### Negative / Zero-State
- `rg -n '(async )?def assess_completeness' src/ ; test $? -eq 1` — **no adapter may produce a verdict**
- `::test_verdict_cannot_be_constructed_directly` — `CompletenessVerdict` is only
  obtainable from `classify()`
#### Operational
- Pure: covered by `L-01b`.

### Edit-Local Gates
`ruff check`; the closure property test.

### Packet-Local Gates
`just test tests/core/test_completeness.py`, `just rules`, `just typecheck`.

### Integration Milestone
`M03`.

### Replan Triggers
A Wave 1 source produces evidence no class expresses → add a class **by schema version**,
never by widening `INDETERMINATE` (which the design already flags as over-loaded).

### Rollback or Recovery
Revert. Pure module; no verdicts are persisted before `WP13`.

---

## WP11 — Identity, normalization, and lifecycle reconciliation

### Outcome
Job identity is stable across re-crawls and never derived from URL or list position;
normalization preserves every raw value; and **no input sequence containing an incomplete
run can produce a `CLOSED` transition.**

### Dependencies
`WP09`, `WP10`.

### Target Invariants
**I-09**, **I-11** (incl. dual overlay identity), I-12, I-13 (five-way platform split),
I-10, I-19.

### Design and Library References
Design §4.1 (clock injected; ordering by run identity), §4.2 (`CanonicalJob`), §8.2.
Spec §8.5 (job lifecycle), §10.3, §13 (identity hierarchy), §16 (25 fields), §4.5.

### Change Surface
#### Preflight Query
```bash
rg -n 'def reconcile|LifecycleTransition|first_seen' src/ -g '*.py'
```
#### Known Touch (verified this session)
New: `src/jobsquery/core/identity.py`, `src/jobsquery/core/normalize.py`,
`src/jobsquery/core/lifecycle.py`, `tests/core/test_lifecycle.py`.

### Required Changes
1. `resolve()` — spec §13's identity ladder: same source ID → same employer + requisition
   ID → same canonical apply URL → same employer + normalized content fingerprint → fuzzy
   candidate → manual review. **Never** URL, row order, or list position (I-11). Where an
   overlay fronts another ATS, **both** identifiers are preserved (I-11, spec §4.5).
2. `normalize()` — the 25 `CanonicalJob` fields, every nontrivially-normalized field
   carrying its raw counterpart, and the five-way platform split of I-13 (presentation ·
   enumeration interface · detail interface · application · canonical employer).
3. `reconcile(prior, refs, verdict, clock) -> LifecycleTransition[]` — spec §8.5 with the
   design's corrections: `UNKNOWN` is a declared state; `REPOSTED`/`SUPERSEDED` are
   **relationships**, modelled as edges rather than states. **Clock injected**; lifecycle
   ordering derives from **run identity**, not wall-clock, so skewed workers cannot produce
   non-monotonic `last_seen` (design §4.1).
4. A failed detail fetch **never** overwrites good content with nulls (I-10, spec §10.3).

### Legacy Disposition and Decommission
None.

### Acceptance Checks
#### Behavioral
- **`tests/core/test_lifecycle.py::test_incomplete_run_never_closes`** — Hypothesis over
  generated run sequences: **no sequence containing an incomplete run yields a `CLOSED`
  transition.** Design §8.2 names this the single most important test in the system; it is
  written before any adapter exists.
- `::test_n_absence_threshold_respected` — closure needs N complete absent observations
- `::test_failed_detail_preserves_prior_content` — I-10
- `::test_identity_stable_across_reorder` — I-11
- `::test_clock_skew_cannot_reorder` — two workers, skewed clocks, monotonic `last_seen`
#### Structural
- `just rules` — `L-01` proves no `time`/`datetime.now` import in core
#### Negative / Zero-State
- `rg -n 'datetime\.now\(|time\.time\(' src/jobsquery/core/` → zero hits
- `::test_url_is_not_identity` — two postings differing only by URL are one job
#### Operational
- Pure: covered by `L-01b`.

### Edit-Local Gates
`ruff check`; the incomplete-run property test.

### Packet-Local Gates
`just test tests/core/`, `just rules`, `just typecheck`.

### Integration Milestone
`M03`.

### Replan Triggers
Spec §8.5's per-source-family N threshold proves unsafe for a real source → the threshold
moves to policy configuration with a floor, recorded as adaptation.

### Rollback or Recovery
Revert. No lifecycle state is persisted before `WP12`.

---

## WP12 — Persistence, single-writer, and the lease  ·  **G3**

### Outcome
SQLite holds the registry, runs, observations, and jobs behind **one** writer task; the
lease carries a fencing token that makes a stale writer's writes rejectable; FTS5 indexes
title and description; a kill-9 mid-enumeration resumes without closing a job.

### Dependencies
`WP02`, `WP05`, `WP11`.

### Target Invariants
I-03, I-19 (idempotent, resumable), I-09, I-16.

### Design and Library References
Design §4.5, §4.6 (ownership table), §8.2b (recovery oracles), LD-09, LD-18.
Recon SPK-12, ADR-10.

### Change Surface
#### Preflight Query
```bash
rg -n 'sqlite3|CREATE TABLE|fts5|busy_timeout' src/ -g '*.py'
```
#### Known Touch (verified this session)
New: `src/jobsquery/shell/db/`, `tests/shell/test_persistence.py`,
`tests/shell/test_recovery.py`.

### Required Changes
1. Schema with **STRICT tables**, WAL, explicit `busy_timeout`, and `PRAGMA` setup in one
   place. `sqlite3.version` was **removed in 3.14** — use `sqlite3.sqlite_version` (LD-18).
2. **One dedicated writer task consuming a queue.** "Single-writer" is a mechanism, not an
   assertion (design §8.2b): stdlib `sqlite3` is blocking and thread-affine, and WAL permits
   one writer with `SQLITE_BUSY` for the rest, while ADR-10 mandates `asyncio.TaskGroup`
   with bounded per-host concurrency.
3. **Lease fencing.** `lease_epoch` increments monotonically on acquisition and **every
   write is conditional on it**. Without it, a holder that stalls past `expires_at` and
   resumes cannot detect it lost the lease, and its stale pages merge silently.
4. Transaction boundary is **per page** (design §8.2b), so a crash leaves observations with
   no `CompletenessEvidence` — a state `reconcile()` must refuse rather than interpret.
5. FTS5 over title + description text, supporting phrase queries for multi-word probes such
   as `"lean six sigma"` and `"process mining"` (recon §5.1).

### Legacy Disposition and Decommission
None.

### Acceptance Checks
#### Behavioral
- `tests/shell/test_persistence.py::test_fts5_phrase_query` — multi-word mandate terms match
- `::test_writer_serializes_n_sources` — N sources enumerate concurrently against one writer
  with **zero `SQLITE_BUSY` escapes**
#### Structural
- `just rules` — `L-06` (acyclic, inward dependency direction)
#### Negative / Zero-State
- **`tests/shell/test_recovery.py::test_kill9_midrun_resumes_without_closing`** — SPK-12's
  oracle: observations exist, no verdict exists, `reconcile()` **refuses** and no job closes
- **`::test_stale_lease_writes_rejected`** — holder A stalls past expiry, B acquires and
  writes, A resumes → **A's writes are rejected**, not merged
- `::test_duplicate_run_attempt_is_idempotent` — I-19
- `rg -n 'sqlite3\.version\b' src/` → zero hits
#### Operational
- `::test_wal_and_busy_timeout_set` — PRAGMA state asserted at connection open

### Edit-Local Gates
`ruff check`; the fencing test.

### Packet-Local Gates
`just test tests/shell/test_persistence.py tests/shell/test_recovery.py`, `just rules`.

### Integration Milestone
`M03` — **G3 closes here.**

### Replan Triggers
Writer throughput becomes the bottleneck at Wave 1 scale → batch writes (adaptation).
Cross-run analytics outgrow SQLite → attach DuckDB **downstream of the JSONL export**,
never in the core (design assumption **A-5**, LD-18).

### Rollback or Recovery
Revert schema and re-create; no production data exists during the build.

---

## WP13 — CrawlRunner, PageGuard, and scheduler

### Outcome
A run advances through the `RunState` machine under a lease, streams pages through
`PageGuard`, writes observations per page, computes evidence once at the end, and honours
`mode = recon | production`.

### Dependencies
`WP07`, `WP08`, `WP10`, `WP12`.

### Target Invariants
I-04, I-05, I-07, I-09, **I-17**, I-19.

### Design and Library References
Design §4.1 (`PageGuard` is shell over pure `signal()`), §4.4 (recon is a mode), §4.6
(ownership), §8.2b. Spec §8.1 (**source lifecycle**), §8.2 (run state machine), §10.4.

### Change Surface
#### Preflight Query
```bash
rg -n 'RunState|PageGuard|acquire_lease' src/ -g '*.py'
```
#### Known Touch (verified this session)
New: `src/jobsquery/shell/runner.py`, `src/jobsquery/shell/guard.py`,
`src/jobsquery/shell/scheduler.py`, `tests/shell/test_runner.py`.

### Required Changes
1. **`PageGuard` is a shell object over pure predicates.** Deciding *when to stop fetching*
   is control flow over I/O; three of the eleven safeguards are inherently impure (a
   page-count budget, mid-run insertion detection, and the end-of-run first-page rerun,
   which is a network request). The eleven `signal()` predicates stay in core (design §4.1,
   WP10). D-3's property is preserved: **one** implementation, shared by every adapter.
2. `RunState` machine per spec §8.2 with the design's corrections — the off-by-one
   transition labels fixed, `RECONCILING` given a terminal, and terminals namespaced so
   `RunState.PARTIAL` is never confused with `Completeness.PARTIAL` (D-5).
3. `mode = recon | production` (§4.4): recon additionally captures HAR and trace and writes
   `FieldAvailability`, and **does not reconcile lifecycle**.
4. Scheduler: due-source selection, lease acquisition with `lease_epoch`, per-host
   concurrency via `anyio.CapacityLimiter`, crash recovery, abandonment detection.
5. **`SourceState` machine** (spec §8.1) driven by run outcomes: `UNRESOLVED → CANDIDATE →
   VERIFIED → HEALTHY ⇄ DEGRADED`, with exits to `BLOCKED`, `MANUAL_ONLY`, `RETIRED`,
   `REJECTED`. The design's corrections apply: `FAILED` is given transitions (spec §8.1
   leaves it orphaned), recovery edges out of `BLOCKED`/`MANUAL_ONLY`/`FAILED` exist because
   real sources unblock, and `VERIFIED → DEGRADED` is reachable without passing through
   `HEALTHY`. **This is where I-17 becomes real**: drift produces a recorded `DEGRADED`
   transition and a review item, never silent data loss.

### Legacy Disposition and Decommission
None.

### Acceptance Checks
#### Behavioral
- `tests/shell/test_runner.py::test_guard_stops_on_repeated_cursor` — the stream halts and
  records a distinct signal
- `::test_first_page_rerun_detects_churn` — §10.4's end-of-run rerun
- `::test_recon_mode_does_not_reconcile` — §4.4
#### Structural
- `just rules` — `L-01` proves `signal()` stayed pure while `PageGuard` did not migrate into
  core
#### Negative / Zero-State
- `::test_adapter_cannot_bypass_guard` — an adapter yielding pages directly still passes
  through `PageGuard`; there is no second path (D-3, invariant #5)
- *(moved to `WP14`, which creates `src/jobsquery/adapters/`; an `rg` against a directory that does not yet exist errors with exit 2, which is not a zero-hit proof)*
#### Operational
- `::test_lease_renewed_during_long_run`
- `tests/shell/test_source_health.py::test_drift_degrades_not_deletes` — I-17; an anomalous
  run moves the source to `DEGRADED` with a review item and **does not** discard prior facts
- `::test_blocked_source_can_recover` — the missing recovery edge

### Edit-Local Gates
`ruff check`; the guard test.

### Packet-Local Gates
`just test tests/shell/test_runner.py`, `just rules`, `just typecheck`.

### Integration Milestone
`M04`.

### Replan Triggers
`PageGuard` cannot express a safeguard without adapter cooperation → the safeguard moves to
the adapter **only** with a recorded justification; more than one such move is a **plan
revision**, because D-3 is the reason this design exists.

### Rollback or Recovery
Revert. Runs are resumable by `WP12`.

---

## WP14 — Adapter port and shared contract harness

### Outcome
`SourceAdapter` exists as a Protocol; `run_board(spec, src, fetch)` and
`run_faceted_endpoint(spec, src, fetch)` own the two control loops; the six-condition
promotion rule is enforceable; and one contract suite runs every adapter against offline
fixtures. A demo adapter of each shape passes all scenarios.

### Dependencies
`WP13`.

### Target Invariants
**I-15** (every recurring adapter passes the shared suite), I-14 (additive extensibility),
I-04, I-07.

### Design and Library References
Design §4.3 (the port and `BoardSpec`), **C-10** (the shared board runner — the component
that survived §6.4's rejection of the configuration language), D-3, D-4, D-9, LD-11.
Spec §12.3, §7.10. Recon ADR-09, SPK-06.

### Change Surface
#### Preflight Query
```bash
rg -n 'class SourceAdapter|run_board|BoardSpec' src/ -g '*.py'
```
#### Known Touch (verified this session)
New: `src/jobsquery/adapters/port.py`, `src/jobsquery/adapters/board.py`,
`tests/adapters/contract_suite.py`.

### Required Changes
1. The port exactly as design §4.3 states: `enumerate` **yields** `EnumerationPage` (D-3),
   `extract` is **pure and takes bytes** (fixture-testable), `evidence` returns evidence and
   **cannot certify** (D-4).
2. `run_board(spec)` owns request → paginate → accumulate identity → yield. Each family
   supplies a `BoardSpec` of values, **with callables where behaviour is irregular** —
   `detect`, `endpoint`, `paginate`, `collection`, `identity`, `fields`,
   `advertised_total`.
2b. **`run_faceted_endpoint(spec)` and the six-condition promotion rule ship here too**, not
   in `WP18`. All 30 Wave 1 institutions are Priority-1 sources (design §1, §4.3), so
   `WP16`'s anchor reconnaissance cannot enumerate a single one of them on `run_board`, and
   I-06 forbids leaving a source browser-backed until spec §8.4's six conditions are checked
   from a clean process. `WP18` then adds Workday's tenant discovery and partitioning on top
   of a runner that already exists, rather than owning the runner itself.
3. **`interface_pattern` (S0–S7) lives on `ProbeResult`/`CareerSource`, never on the
   `BoardSpec`** — recon §7 warns that two deployments of one ATS expose different public
   interfaces, and putting it on the family is the class-vs-instance error the design
   diagnosed as D-9.
4. The shared contract suite (spec §12.3): detection · empty board · single item ·
   exact-full-page · short terminal page · multi-page · repeated-page rejection · duplicate
   source IDs · identity stability across two runs · malformed-record tolerance · evidence
   emission classifying to the expected verdict.

### Legacy Disposition and Decommission
None. The `SourceFamilyManifest` contract, its schema, its interpreter, and the `L-03`
governance rule are **never created** — design §6.4 falsified them before implementation.

### Acceptance Checks
#### Behavioral
- `tests/adapters/contract_suite.py` — all eleven scenarios, parameterized over every
  registered adapter
- `::test_demo_board_adapter_passes_suite` and `::test_demo_faceted_adapter_passes_suite`
- `::test_adapter_cannot_bypass_guard` — moved here from `WP13`: an adapter yielding pages
  directly still traverses `PageGuard`; there is no second path (D-3, invariant #5)
- `::test_promotion_rule_all_six_conditions` — a source failing any one of spec §8.4's six
  conditions stays browser-backed (I-06); hoisted here so `WP16` can satisfy I-06
#### Structural
- `just rules` — **L-11** proves no adapter defines `assess_completeness` or constructs a
  `CompletenessVerdict` (D-4)
#### Negative / Zero-State
- `rg -n 'SourceFamilyManifest|manifest.schema.json' src/ contracts/` → zero hits
- `rg -n 'interface_pattern' -g '**/spec.py' src/jobsquery/adapters/ ; test $? -eq 1` — glob handled by `rg`, not the shell (a bare `*/spec.py` aborts under zsh before WP20 creates those files)
#### Operational
- The suite runs fully offline; `L-01b` covers `extract`.

### Edit-Local Gates
`ruff check`; the demo adapter suite run.

### Packet-Local Gates
`just test tests/adapters/`, `just rules`, `just typecheck`.

### Integration Milestone
`M04`.

### Replan Triggers
The contract suite cannot be expressed generically across manifest-driven and coded
adapters → **plan revision**; I-15 is not negotiable.

### Rollback or Recovery
Revert. No adapters depend on it yet.

---

## WP15 — Discovery: employer universe, brands, and career sources

### Outcome
The 30 Wave 1 brands and their legal entities are ingested as **data**; for each brand,
official career-source candidates are discovered from the employer's own domain with
recorded evidence and confidence; no source is `VERIFIED` without official-link evidence.

### Dependencies
`WP09`, `WP12`, `WP13`.

### Target Invariants
I-02 (official sources only), I-14 (new employer = data), I-20, I-13.

### Design and Library References
Design §2.1 (the fixed inputs, reproduced as data), §4.7. Spec §4.1, §4.2, §4.7, §7.1,
§7.2, WP-07/WP-08. Recon §7.2, §7.3, §8 stages 1–2.

### Change Surface
#### Preflight Query
```bash
rg -n 'FFIEC|GLEIF|career_source|discover' src/ data/ -g '*.py' -g '*.json'
```
#### Known Touch (verified this session)
New: `data/universe/` (seed registry), `src/jobsquery/discovery/`,
`tests/discovery/`.

### Required Changes
1. Seed the registry from design §2.1: **30 Wave 1 brands** (12 + 6 + 6 + 6), the **12-brand
   anchor batch in order**, and the **15 Wave 2 brands** — as versioned data files, never
   code constants (I-14).
2. Universe ingestion adapters for FFIEC NIC, FDIC, OCC, NYDFS, SEC IAPD, FINRA, NAIC,
   GLEIF, EDGAR and manual inclusions (spec §4.1), preserving originals and recording
   `inclusion_reason` + `universe_sources`.
3. Brand resolution: many legal entities → one brand; **a shared board is crawled once**,
   which `BrandSourceLink` now makes representable (D-1).
4. Source discovery over the spec §4.2 surface list and the §4.7 standards ladder:
   homepage/footer links, `robots.txt`, declared sitemaps, sitemap indexes, conventional
   paths, redirects, vendor-host detection, embedded script config, JSON-LD, iframes.
5. Long-tail detection (spec §4.6, 18 vendors) routes to **generic handling**, never a
   dedicated adapter.
6. Recon §7.2's twelve `resolution_state` values and §7.3's three confidence states.
   **Low confidence is never `resolved`** — search-engine evidence may nominate a candidate,
   but acceptance requires an official employer-controlled link.

### Legacy Disposition and Decommission
None.

### Acceptance Checks
#### Behavioral
- `tests/discovery/test_registry_seed.py::test_wave1_is_thirty_brands` — and the anchor
  batch is twelve, **in the specified order**
- `tests/discovery/test_discovery.py::test_vendor_host_detection` — fixture per ATS family
- `::test_shared_board_linked_not_duplicated` — D-1
#### Structural
- `just rules` — `L-06`
#### Negative / Zero-State
- `::test_low_confidence_never_resolved` — recon §7.3
- `rg -n 'JPMorgan|Goldman' src/ -g '*.py'` → zero hits — **the universe is data, not code**
  (I-14)
#### Operational
- Discovery runs through the gate; `L-01b` unaffected (discovery is shell).

### Edit-Local Gates
`ruff check`; the seed test.

### Packet-Local Gates
`just test tests/discovery/`, `just rules`.

### Integration Milestone
`M04`.

### Replan Triggers
A regulatory dataset is unavailable or unusable → that brand is seeded manually with a
recorded `inclusion_reason` (adaptation; the universe is fixed regardless).

### Rollback or Recovery
Revert. Registry data is versioned and re-seedable.

---

## WP16 — Anchor-batch reconnaissance execution  ·  **G4**

### Outcome
All twelve anchor institutions observed in `mode=recon`. Every one lands in exactly one
resolution state, with a fixture corpus, a completeness grade, and a field-availability
assessment. The recon observation dataset is produced **as a view**, not a second contract.

### Dependencies
`WP14`, `WP15`.

**Resolving the circularity — and the anchor batch needs the *faceted* runner, not the
board runner.** Recon stage 6 validates enumeration, enumeration needs a runner, and the
adapters come after the spikes recon feeds. An earlier draft answered this by pointing at
`WP14`'s `run_board` — **which is wrong**: `run_board` is the Priority-2 uniform-board loop,
and design §1/§4.3 place all 30 Wave 1 institutions in **Priority 1**, whose shape is
`run_faceted_endpoint`. No anchor institution is a `run_board` source.

The cycle is broken by **hoisting the minimum into `WP14`**, not by prose: `WP14` now
delivers `run_faceted_endpoint` and the spec §8.4 **six-condition promotion rule** alongside
`run_board`, because I-06 requires every browser-discovered endpoint to be replay-tested
before a source may stay browser-backed — an obligation this packet cannot otherwise meet.
`WP18` then hardens Workday on top of a runner that already exists.

Within that, design §7's single permitted intermediate state applies: between G4 and G7 a
source may sit at `resolution_state = investigating` with a provisional spec **that has not
yet passed the contract suite**. Provisional specs are exploratory instruments; `WP18`–`WP20`
harden them. Exit invariant: at G7 every source is in a terminal resolution state or in the
coverage-gap register, and no provisional spec survives.

### Target Invariants
I-02, I-06, I-07, I-16, I-17.

### Design and Library References
Design §4.4 (recon is a mode), §7 G4, §6.6 (recon §9 reshaped, deliverable preserved).
Recon §8 (eight stages), §10 (fixture protocol), §19.2 (datasets), §21 (first batch order).

### Change Surface
#### Preflight Query
```bash
ls data/fixtures/ 2>/dev/null; rg -n 'FieldAvailability' src/ -g '*.py'
```
#### Known Touch (verified this session)
New: `data/fixtures/<institution>/`, `src/jobsquery/recon/view.py`,
`tests/recon/test_observation_view.py`.

### Required Changes
1. Run recon §8's eight stages over the anchor batch in the §21 order: JPMorganChase, Citi,
   Goldman Sachs, Morgan Stanley, then the remaining eight.
2. Emit recon §19.2's datasets — `institutions.jsonl`, `career_sources.jsonl`,
   **`source_observations.jsonl`**, `source_family_catalog.json`, `fixture_manifest.jsonl`,
   `known_gaps.jsonl`. The observation record is a **view** joining `CareerSource` +
   `CrawlRun` + `CompletenessEvidence` + `FieldAvailability`, carrying `search_checks` and
   `resolution_confidence`; **§9.3's field names win** over §9.2's example (design §6.6).
3. Capture the fixture corpus per recon §10.2 with full manifests and **redaction** — no
   cookies, session or anti-forgery tokens, authorization headers, or browser storage state.
4. `FieldAvailability` per recon §8 stage 7: each field classified `structured` ·
   `embedded_metadata` · `html_extracted` · `description_only` · `absent` · `conflicting`.

### Legacy Disposition and Decommission
None.

### Acceptance Checks
#### Behavioral
- `tests/recon/test_observation_view.py::test_view_validates_under_both_engines` — I-16
- `::test_every_anchor_has_exactly_one_resolution_state` — twelve for twelve
#### Structural
- `just contracts` — the exported datasets validate against published schemas
#### Negative / Zero-State
- `tests/recon/test_redaction.py::test_no_credentials_in_fixtures` — `rg` deny-list over
  `data/fixtures/` for cookie, authorization, token, and storage-state patterns → zero hits
- `::test_no_failed_run_recorded_as_valid_zero` — recon §20; spec §10.5
#### Operational
- `just recon-anchor` — exit 0; every fixture is consumed by at least one offline test
  (recon §10.6's replay rule)

### Edit-Local Gates
`ruff check`; the view validation test.

### Packet-Local Gates
`just test tests/recon/`, `just contracts`, `just rules`.

### Integration Milestone
`M04` — **G4 closes here.**

### Replan Triggers
An anchor institution cannot be observed at all (blocked, or no public source) → it is
recorded in `known_gaps.jsonl` with a terminal state. That is a **coverage outcome, not a
plan failure** (design assumption **A-6**).

### Rollback or Recovery
Recon is read-only against the network and additive to storage. Re-runnable.

---

## WP17 — Architecture spikes  ·  **G5**

### Outcome
Recon §12's twelve spikes are closed against real anchor-batch evidence, and every
provisional library decision in design §5 is either confirmed or amended with the evidence
that settled it.

### Dependencies
`WP16`.

### Target Invariants
I-06, I-07, I-15. Design assumptions **A-1 … A-7**.

### Design and Library References
Design §11 (assumptions and reversal paths), LD-10 (provisional), LD-11 (provisional).
Recon §12 (SPK-01…SPK-12), ADR-01…ADR-12.

### Change Surface
#### Preflight Query
```bash
rg -n 'SPK-0|SPK-1' docs/ ; ls docs/spikes/ 2>/dev/null
```
#### Known Touch (verified this session)
New: `docs/spikes/`, updates to `docs/designs/…_v1_….md` §5/§11 recorded as an amendment
note (never an overwrite of an accepted record).

### Required Changes
Run and record the twelve spikes. Four settle design assumptions and are the reason this
packet gates the adapter work:

| Spike | Settles | Consequence if it fails |
|---|---|---|
| **SPK-06** adapter granularity | **A-1**, **LD-11** | If ≥3 of the eight Priority-2 families still need escape hatches inside `run_board`, those families become fully coded adapters. `WP20` shrinks; the port is unchanged. |
| **SPK-08** robots | **A-4**, **LD-10** | Adopt Protego behind the unchanged `RobotsPolicy` port — a **plan revision** (new dependency), not a design change. |
| **SPK-02** browser endpoint promotion | **A-3**, I-06 | Determines how many sources stay browser-backed; feeds `WP18`. |
| **SPK-05** enumeration proof | I-07 | Confirms the `classify()` rubric against real boards. |

`SPK-01` (HTTP baseline) additionally decides whether HTTP/2 is adopted, which is the open
half of **LD-15** — `httpx2[http2]` is deferred until measured, because `httpx2 §14` is
explicit that HTTP/2 is not automatically faster.

### Legacy Disposition and Decommission
None.

### Acceptance Checks
#### Behavioral
- `docs/spikes/SPK-NN.md` exists for all twelve, each stating question · method · evidence ·
  decision · reconsideration trigger
#### Structural
- Each spike's decision cites the LD or assumption it settles
#### Negative / Zero-State
- No spike is recorded as "deferred" without a named trigger and owner
#### Operational
- `just spikes` — replays every spike's offline portion from fixtures

### Edit-Local Gates
n/a (documentation and measurement packet).

### Packet-Local Gates
`just test`, `just rules` (must remain green — spikes may add fixtures, not bypass gates).

### Integration Milestone
`M05` — **G5 closes here.**

### Replan Triggers
**SPK-06 reversing LD-11 is the single highest-probability replan in this plan.** It is
anticipated, bounded, and cheap: `WP20`'s scope changes, `WP14`'s port does not.

### Rollback or Recovery
n/a.

---

## WP18 — Priority 1: faceted-endpoint runner and Workday

### Outcome
Workday — the highest-coverage single family for Wave 1 — passes the contract suite on
`run_faceted_endpoint`, with tenant/site discovery, count reconciliation, and partitioning
under a hard cap.

### Dependencies
`WP17`.

### Target Invariants
I-06 (browser-discovered endpoints tested for HTTP replay), I-01, I-07, I-15.

### Design and Library References
Design §4.3 ("Priority 1 is where the generalization actually is"), §1 ("Where the effort
goes"). Spec §5 Priority 1, §8.4 (the six-condition promotion rule), §9.12 (Workday),
§10.6 (hard caps and partitioning). Recon SPK-02.

### Change Surface
#### Preflight Query
```bash
rg -n 'myworkdayjobs|faceted|partition_key' src/ -g '*.py'
```
#### Known Touch (verified this session)
New: `src/jobsquery/adapters/faceted.py`, `src/jobsquery/adapters/workday/`,
`tests/adapters/test_workday.py`.

### Required Changes
1. Workday on `run_faceted_endpoint` (the runner and the promotion rule land in `WP14`).
2. **The six-condition promotion rule** (spec §8.4) applied to Workday tenants before any is
   allowed to remain browser-backed: same active job IDs · same board scope · identical pagination ·
   no browser credential or private token · public tenant/site identifiers only · within the
   source's access policy. Verified **from a clean process**.
3. Workday: tenant/site discovery, **multiple external career sites per employer**
   (spec §4.4 — one brand may need several boards for full coverage), anonymous faceted
   search, count reconciliation, detail retrieval, HTTP promotion, partition fallback per
   §10.6 — and **partitioning by NYC is prohibited**, because it would make employer-level
   coverage unauditable (I-01, spec §10.6).
4. Scope verification: an unresolved scope blocks employer-level completeness claims
   (`UnresolvedScope`, WP02).

### Legacy Disposition and Decommission
None.

### Acceptance Checks
#### Behavioral
- `tests/adapters/test_workday.py` — the full `WP14` contract suite, offline
- `::test_workday_tenant_promotion` — each Workday tenant is replay-tested (the generic rule is proved in `WP14`)
- `::test_partition_union_deduplicated` — §10.6; `PARTITIONED` requires demonstrated
  coverage (design §3's resolution of the §10.6/§10.1 contradiction)
#### Structural
- `just rules` — no `assess_completeness`; `interface_pattern` not on the spec
#### Negative / Zero-State
- `::test_nyc_partitioning_rejected` — a spec partitioning by NYC fails validation
- `rg -n 'requisition.*count' src/jobsquery/adapters/` reviewed — Oracle's facet-count trap
  is documented at `WP19`, not silently generalized here
#### Operational
- `::test_multi_site_employer_enumerated_completely`

### Edit-Local Gates
`ruff check`; the contract suite for Workday.

### Packet-Local Gates
`just test tests/adapters/test_workday.py`, `just rules`, `just typecheck`.

### Integration Milestone
`M06`.

### Replan Triggers
Workday's public search cannot be replayed over HTTP from a clean process for a Wave 1
tenant → that tenant is browser-backed with recorded justification (I-06), not a plan
failure.

### Rollback or Recovery
Revert the adapter; the runner is independently useful for `WP19`.

---

## WP19 — Priority 1: Oracle Recruiting CX and Taleo

### Outcome
Oracle CX and Taleo enumerate completely on `run_faceted_endpoint`, with the documented
facet-count trap handled, or are explicitly classified manual/blocked.

### Dependencies
`WP18`.

### Target Invariants
I-06, I-13, I-15, I-01.

### Design and Library References
Design §2.1 (fifteen enterprise families). Spec §9.13, §4.4 (multiple external sites;
location-facet counts).

### Change Surface
#### Preflight Query
```bash
rg -in 'oracle|taleo|careersection' src/ -g '*.py'
```
#### Known Touch (verified this session)
New: `src/jobsquery/adapters/{oracle,taleo}/`, `tests/adapters/test_oracle.py`.

### Required Changes
1. Oracle CX: site codes, **multiple external career sites per employer**, blind search
   (its public search works with no criteria, which is ideal for enumeration).
2. **The facet-count trap**: Oracle warns that location-facet counts count *location
   occurrences*, not requisitions (spec §4.4). Treating one as an advertised total would
   grant a false `EXACT`.
3. Taleo kept a **separate family** from Oracle CX — different pagination, different
   `/careersection/` shape.

### Legacy Disposition and Decommission
None.

### Acceptance Checks
#### Behavioral
- Contract suite green for both families
- `::test_oracle_facet_count_not_treated_as_total` — the documented trap
- `::test_multi_site_employer_union_complete`
#### Structural
- `just rules` — `L-02`; no adapter constructs a verdict
#### Negative / Zero-State
- `rg -in 'oauth|api_key|client_secret' src/jobsquery/adapters/oracle src/jobsquery/adapters/taleo ; test $? -eq 1` (I-21)
#### Operational
- `acquisition_mode` and browser dependence recorded per source (I-06)

### Edit-Local Gates
`ruff check`; the facet-count test.

### Packet-Local Gates
`just test tests/adapters/test_oracle.py`, `just rules`, `just typecheck`.

### Integration Milestone
`M06`.

### Replan Triggers
Neither family's public search replays from a clean process → browser-backed with recorded
justification (I-06).

### Rollback or Recovery
Revert per family.

---

## WP19b — Priority 1: SAP SuccessFactors and iCIMS

### Outcome
Both families enumerate from **public** surfaces only — no partner feed, no credential — or
are explicitly classified.

### Dependencies
`WP18`.

### Target Invariants
**I-21** (no credentialed path), I-06, I-13, I-15.

### Design and Library References
Design §2.1, I-21. Spec §9.14, §9.15, §4.4.

### Change Surface
#### Preflight Query
```bash
rg -in 'successfactors|icims|career site builder' src/ -g '*.py'
```
#### Known Touch (verified this session)
New: `src/jobsquery/adapters/{successfactors,icims}/`.

### Required Changes
1. SAP SuccessFactors: Career Site Builder domains, job-search network calls, multi-language.
2. **iCIMS: the public employer portal only.** iCIMS publishes a standardized XML job-board
   feed, but consuming vendors must authenticate via OAuth **and be approved** (spec §4.4).
   The design forbids assuming that access (I-21), so the portal is the source.
3. Custom career domains for both; scope verification per source.

### Legacy Disposition and Decommission
None.

### Acceptance Checks
#### Behavioral
- Contract suite green for both
- `::test_icims_uses_public_portal_not_feed`
#### Structural
- `just rules`
#### Negative / Zero-State
- `rg -in 'oauth|api_key|client_secret|partner.feed' src/jobsquery/adapters/icims ; test $? -eq 1` — I-21
#### Operational
- Browser dependence recorded per source

### Edit-Local Gates
`ruff check`.

### Packet-Local Gates
`just test tests/adapters/`, `just rules`.

### Integration Milestone
`M06`.

### Replan Triggers
A Wave 1 employer's iCIMS portal is unusable without approval → `manual_only` with a
coverage-gap entry (A-6).

### Rollback or Recovery
Revert per family.

---

## WP19c — Priority 1: candidate-experience overlays

### Outcome
Overlay platforms are enumerated from a clean context, both identifiers are preserved, and
a recommendation set can never be mistaken for the inventory.

### Dependencies
`WP18`.

### Target Invariants
**I-23** (recommendations are not the inventory), **I-11** (dual identity), I-13, I-15.

### Design and Library References
Design §2.1 (overlays incl. Avature and Symphony Talent/SmashFly), I-11, I-13, I-23.
Spec §9.16, §4.5 (the five-way modeling rule).

### Change Surface
#### Preflight Query
```bash
rg -in 'phenom|eightfold|beamery|radancy|avature|smashfly|symphony' src/ -g '*.py'
```
#### Known Touch (verified this session)
New: `src/jobsquery/adapters/overlay/`.

### Required Changes
1. Phenom, Eightfold, Beamery, Radancy, Avature, Symphony Talent/SmashFly.
2. **Clean-context enumeration and personalization detection.** A personalized or
   recommendation-driven result set is never the inventory (I-23) and cannot produce a
   closure-eligible run.
3. **Both identifiers stored** — the overlay's source job ID *and* the underlying
   ATS/requisition ID (I-11, spec §4.5), plus the five-way platform split (I-13): the
   enumeration interface is routinely not the ATS behind the apply link.
4. The remaining §4.4 families (Jobvite, Dayforce, ADP, Cornerstone, Kenexa/BrassRing,
   PageUp, UKG, Paylocity, Paycom) are **detected and classified** here; adapters are built
   only where a Wave 1 brand actually uses them, per recon ADR-09's ≥2-institution rule.

### Legacy Disposition and Decommission
None.

### Acceptance Checks
#### Behavioral
- Contract suite green per implemented overlay
- `::test_overlay_stores_both_identifiers` — I-11
- `::test_underlying_ats_detected` — I-13
#### Structural
- `just rules` — `L-02` across all new adapters
#### Negative / Zero-State
- **`::test_personalized_context_cannot_close`** — I-23
- `rg -in 'oauth|api_key|client_secret' src/jobsquery/adapters/overlay ; test $? -eq 1`
#### Operational
- Each family records `acquisition_mode` and browser dependence (I-06)

### Edit-Local Gates
`ruff check`; the personalization test.

### Packet-Local Gates
`just test tests/adapters/`, `just rules`, `just typecheck`.

### Integration Milestone
`M06`.

### Replan Triggers
An overlay cannot be enumerated without personalization → `manual_only`; three or more such
cases means the overlay abstraction is wrong (**plan revision**).

### Rollback or Recovery
Revert per family.

## WP20 — Priority 2: board families on `run_board`

### Outcome
The eight uniform board families enumerate completely through `run_board` with per-family
`BoardSpec` literals, each passing the contract suite offline.

### Dependencies
`WP17` (SPK-06 must have settled the scope first).

### Target Invariants
I-14, I-15, I-01, I-07.

### Design and Library References
Design §4.3 (`BoardSpec`, and the five families whose irregularity killed the DSL), §6.4,
LD-11. Spec §9.1–§9.8, §5 Priority 2.

### Change Surface
#### Preflight Query
```bash
rg -n 'greenhouse|lever|ashby|workable|recruitee|personio|teamtailor|smartrecruiters' src/ -i
```
#### Known Touch (verified this session)
New: `src/jobsquery/adapters/boards/<family>/spec.py` ×8, `tests/adapters/boards/`.

### Required Changes
Eight `BoardSpec`s. Five carry irregularity that must be a **callable, not a schema
extension** — this is the concrete reason design §6.4 rejected a manifest DSL:

| Family | The irregularity |
|---|---|
| Greenhouse | detection includes an embedded board token in a careers link, not only a host |
| **Lever** | `skip` advances by **records returned**, not by `limit`; global **and EU** hosts |
| **Ashby** | identity is a three-level ladder: posting ID → job-URL path → apply-URL path |
| **Workable** | enumeration filters a **multi-tenant global XML feed**; completeness is two-valued |
| **Recruitee** | a five-step branching probe with **per-branch** completeness semantics |
| **Personio** | multi-language fan-out, join on position ID, dedupe; tenant **and language** must be verified |
| **Teamtailor** | silent **first-100 truncation** detection, on **custom career domains** |
| **SmartRecruiters** | documented API requires a key → probe-and-fallback, no stable URL |

`advertised_total` is set **only where the count is independent** of the collection
response. Greenhouse's total travels with the array it counts, so it does not qualify:
`total_is_independent=False`, and the verdict is `TERMINAL`, not `EXACT` (design §6.4,
WP10).

### Legacy Disposition and Decommission
None.

### Acceptance Checks
#### Behavioral
- Contract suite green for all eight, offline
- `::test_lever_skip_advances_by_returned` — the specific defect that falsified the DSL
- `::test_teamtailor_first_100_truncation_detected` — a silent cap is a truncation signal
- `::test_personio_language_variants_deduped`
#### Structural
- `just rules` — no adapter constructs a verdict
#### Negative / Zero-State
- `::test_greenhouse_total_not_independent` — asserts `EXACT` is **not** granted on a
  self-consistent count
- `rg --files -g '*.y*ml' src/jobsquery/adapters/ ; test $? -eq 1` — **specs are Python, not
  configuration** (design §6.4). The obvious `rg -n '\.yaml'` searches file *contents* and
  passes with a real `spec.yaml` sitting in the tree.
#### Operational
- **`::test_live_smoke_nonempty`** (spec §12.7) — an acceptance criterion for **every**
  family: a `BoardSpec` whose `collection` accessor resolves empty against a live
  known-non-empty board produces a clean zero that fixtures cannot catch, because the same
  broken spec produced the evidence (design §8.2b). Run outside the offline suite.

### Edit-Local Gates
`ruff check`; per-family contract suite.

### Packet-Local Gates
`just test tests/adapters/boards/`, `just rules`, `just typecheck`.

### Integration Milestone
`M06`.

### Replan Triggers
SPK-06 (`WP17`) already reversed the scope → this packet is re-scoped before it starts.

### Rollback or Recovery
Revert per family.

---

## WP21 — Generic adapters: sitemap, JSON-LD, static HTML

### Outcome
Employers on proprietary or long-tail platforms are enumerated generically, with drift
detection — and embedded metadata is never mistaken for proof of complete enumeration.

### Dependencies
`WP20`.

### Target Invariants
**I-22** (metadata never proves enumeration), I-01, I-07, I-17.

### Design and Library References
Design §2.1 (the §4.7 standards ladder), I-22. Spec §9.9–§9.11, §4.6, §4.7, §5 Priority 3.

### Change Surface
#### Preflight Query
```bash
rg -n 'sitemap|JobPosting|recipe' src/jobsquery/adapters/ -g '*.py'
```
#### Known Touch (verified this session)
New: `src/jobsquery/adapters/generic/{sitemap,jsonld,static}/`.

### Required Changes
1. **Sitemap adapter** — index walking, streamed large sitemaps (`iterparse`), URL
   classification into career/job/other, gzip handling (lxml's `decompress=False`, WP09).
2. **JSON-LD adapter** — `JobPosting` extraction across single/array/`@graph`/multi-block
   shapes, reconciled against HTML, **conflicts preserved rather than silently resolved**
   (recon §8 stage 4). It **must not own enumeration** (spec §9.9): JSON-LD proves what a
   detail page says, never that every job was discovered (I-22).
3. **Static-HTML recipe adapter** — configurable selectors with **selector-drift
   detection** (spec §9.11, §10.9's "HTML selector counts change materially").
4. Long-tail vendors (spec §4.6) route here.

### Legacy Disposition and Decommission
None.

### Acceptance Checks
#### Behavioral
- Contract suite green for all three
- `::test_jsonld_conflicts_preserved` — page vs metadata disagreement is recorded, not
  resolved
- `::test_selector_drift_flagged` — a fixture whose selector count collapses raises an
  anomaly, not an empty result
#### Structural
- `just rules`
#### Negative / Zero-State
- **`::test_jsonld_alone_cannot_claim_enumeration_complete`** — I-22; the verdict is at best
  `INDETERMINATE` without pagination evidence
- `::test_sitemap_bounded_memory` — a large sitemap does not build a full DOM
#### Operational
- `::test_live_smoke_nonempty` per configured source

### Edit-Local Gates
`ruff check`.

### Packet-Local Gates
`just test tests/adapters/generic/`, `just rules`.

### Integration Milestone
`M06`.

### Replan Triggers
A static recipe requires per-source Python → it becomes a coded adapter (adaptation).

### Rollback or Recovery
Revert per adapter.

---

## WP22 — Query, export, and coverage audit  ·  **G7**

### Outcome
Mandate and geography queries run over stored postings and can be changed **without
recrawling**; exports validate independently; and the coverage auditor **fails the build**
if any Wave 1 brand lacks a terminal status.

### Dependencies
`WP19`, `WP19b`, `WP19c`, `WP20`, `WP21`.

### Target Invariants
**I-01** (search definitions change without recrawl), I-02, I-16, I-17. Spec §15.25.

### Design and Library References
Design §4.7, §8.3. Spec §16 (the eleven-step target behaviour), WP-20, WP-21, WP-22.
Recon §5 (probe vocabularies), §13.3 (portfolio target), §15 (six metric families), §19.2.

### Change Surface
#### Preflight Query
```bash
rg -n 'ProbeVocabulary|mandate|NYC_CORE|coverage' src/ -g '*.py'
```
#### Known Touch (verified this session)
New: `src/jobsquery/query/`, `src/jobsquery/audit/`, `src/jobsquery/cli.py`,
`data/vocabularies/`.

### Required Changes
1. **Versioned probe vocabulary as data**: 18 high-signal terms, 13 adjacent terms,
   `NYC_CORE` (7), `NYC_METRO_EXTENDED` (5 + open), and the five-value remote taxonomy —
   never code constants (design §2.1, §4.7).
2. `match(job, vocabulary) -> MandateMatch[]` — pure, recording **which** term matched and
   **where** (title vs description), so a result is explainable. Re-running it over stored
   rows never triggers a fetch (I-01).
3. **Geography is reported, never silently merged**: `NYC_CORE` and `NYC_METRO_EXTENDED` are
   separate counts in every output; unknown-location postings are **included and labelled**,
   not dropped — dropping them would hide the coverage gap the system exists to measure.
   A remote posting is not assumed to permit NY residence unless it says so.
4. Exports: JSONL + CSV, schema-validated on the way out (I-16), using recon §19.2's names.
5. **Coverage auditor**: every Wave 1 brand gets exactly one of `healthy` · `degraded` ·
   `manual_only` · `blocked` · `no_public_source_observed` · `documented_gap`. Audit-only
   sources feed miss-discovery into `known_gaps.jsonl` and are **never** a source of record
   (I-02); LinkedIn is manual-import only (I-21).
6. Report recon §13.3's portfolio thresholds: ≥80% of Wave 1 brands on automated healthy
   sources, ≥90% of observed Wave 1 job volume, every anchor institution resolved or
   explicitly classified.
7. The CLI entry point (`DB01`'s replacement for the deleted seed `main`).

### Legacy Disposition and Decommission
`DB01` closes here: `jobsquery = "jobsquery.cli:main"` replaces the deleted seed. Negative
proof asserted in `WP01`; positive proof here.

### Acceptance Checks
#### Behavioral
- `tests/query/test_mandate.py::test_vocabulary_change_requires_no_refetch` — **I-01's
  defining test**: change the vocabulary, re-run, assert zero fetches (the `L-01b` socket
  monkeypatch is reused here)
- `::test_phrase_terms_match` — `"lean six sigma"`, `"process mining"` via FTS5 phrase query
- `::test_geo_classes_reported_separately` — no silent merge
- `::test_unknown_location_included_and_labelled`
#### Structural
- `just contracts` — exports validate under `Draft202012Validator`
#### Negative / Zero-State
- **`tests/audit/test_coverage.py::test_no_brand_silently_omitted`** — asserts
  `|Wave 1 brands| == |brands in exactly one terminal state|`; **fails the build**
  otherwise (design §8.3, recon §20)
- `rg -in 'linkedin' src/jobsquery/adapters/ ; test $? -eq 1` — case-insensitive (I-21)
- `::test_audit_source_never_source_of_record` — I-02
#### Operational
- `just export` — exit 0; `just audit` — exit 0 and emits the coverage report
- `jobsquery --help` — exit 0

### Edit-Local Gates
`ruff check`; the no-refetch test.

### Packet-Local Gates
`just test tests/query/ tests/audit/`, `just contracts`, `just rules`.

### Integration Milestone
`M06` — **G7 closes here.**

### Replan Triggers
FTS5 phrase matching proves inadequate for the mandate vocabulary → evaluate DuckDB
downstream of the export (design assumption **A-5**, LD-18); the core is untouched either
way.

### Rollback or Recovery
Revert. Query and audit are read-only over stored facts.

---

## WP23 — Observability, metrics, and manual curation

### Outcome
Every run emits structured records aligned to stage boundaries; recon §15's six metric
families are **derived views over stored facts**, not separately maintained counters; the
21-category failure taxonomy is provably exhaustive and mutually exclusive; and manual
annotations coexist with source-derived fields without overwriting them.

### Dependencies
`WP13`, `WP22`.

### Target Invariants
**I-18** (every failure in exactly one category; no generic "run failed"), I-17, I-02,
I-03 (provenance and observability stay distinct).

### Design and Library References
Design **§9.1** (the 21 categories), **§9.3** (observability vs provenance), §4.7.
Recon **§15** (six metric families), §16 (taxonomy). Spec §6 (Manual Annotation Store),
WP-21 (manual curation and audit-source imports).

### Change Surface
#### Preflight Query
```bash
rg -n 'FailureCategory|metrics|annotation|logging' src/ -g '*.py'
rg -n 'FailureRecord\(' src/ -g '*.py'      # every construction site must name a category
```
#### Known Touch (verified this session)
New: `src/jobsquery/obs/`, `src/jobsquery/curation/`, `tests/obs/`, `tests/curation/`.

### Required Changes
1. **Structured observability** (design §9.3): per-run counters — requests, bytes, pages,
   retries, 429s, timeouts, parse failures, duration — plus per-source health history.
   **Provenance and observability are not merged**: provenance answers *what produced this
   job record* (`SourceObservation → FetchObservation → ContentBlob`); observability answers
   *what did this run do*.
2. **Recon §15's six metric families as derived views**, never stored counters: coverage ·
   acquisition · structure · target-search validation · completeness · operational. A view
   recomputes from facts, so a metric can never drift from what happened.
3. **Failure-taxonomy wiring (I-18) — the remaining sweep.** The `FailureCategory` enum and
   the rule that **a `FailureRecord` cannot be constructed without a category** land in
   `WP02` (contract) and are enforced from `WP07` onward, because WP04/WP06/WP07/WP12/WP13/
   WP15 all author failure paths at M02–M04. Retrofitting exhaustive classification across
   the whole shell at M06 is precisely the ordering error this plan criticises in spec §14.
   What remains here is the exhaustiveness proof and the metrics that consume it. The design also removes one layer leak the proposal
   carried: `parser contract failure` is **not** an HTTP-request outcome (spec §8.3 placed
   it there) and must not appear in transport code.
4. **Manual curation** (spec §6, WP-21): tags, notes, and review status stored **beside**
   source-derived fields; a manual annotation never overwrites a parsed value. Audit-source
   imports (LinkedIn manual, Google, Indeed, eFinancialCareers, recruiter email, employer
   alerts) create external observation records reconciled against the official inventory,
   feeding `known_gaps.jsonl` — never a source of record (I-02).

### Legacy Disposition and Decommission
None.

### Acceptance Checks
#### Behavioral
- `tests/obs/test_metrics.py::test_metric_families_are_derived` — mutate a stored fact,
  recompute, assert the metric changes; no counter is written at run time
- `tests/curation/test_annotations.py::test_annotation_never_overwrites_source_field`
- `::test_audit_import_creates_external_observation_only`
#### Structural
- **`tests/obs/test_taxonomy.py::test_taxonomy_exhaustive_and_exclusive`** — every failure
  path maps to exactly one of the 21 categories; a `FailureRecord` cannot be constructed
  without one. This is the check I-18 was previously asserted without.
- `just rules` — **L-10** proves no `raise` in the shell escapes without a category
#### Negative / Zero-State
- `rg -n 'run failed|generic error|Exception\("' src/` → zero hits (I-18)
- `rg -n 'parser_contract_failure' src/jobsquery/shell/http.py` → zero hits (the layer leak)
- `::test_manual_source_never_authoritative` — I-02
#### Operational
- `just metrics` — emits the six families for the current corpus; exit 0

### Edit-Local Gates
`ruff check`; the taxonomy test.

### Packet-Local Gates
`just test tests/obs/ tests/curation/`, `just rules`.

### Integration Milestone
`M06`.

### Replan Triggers
A failure arises that fits no category → add one **by schema version** (recon §16's
taxonomy is a published contract), never by widening an existing category or introducing a
generic bucket.

### Rollback or Recovery
Revert. Observability is derived; curation data is additive and separately stored.

---

## 5. Integration milestones

Six, aligned to the design's gates. Each is a point where packets first interact or risk
accumulates — not a per-packet ceremony.

### M01 — Contracts frozen (design **G1**)
**Packets:** WP01, WP02.
**Evidence:** `just contracts` regenerates cleanly and `git diff --exit-code contracts/` is
empty; every model round-trips through both validators; `L-02` proves the three state
vocabularies are disjoint; dependency floors are exact pins; `rg 'def main' src/` is empty.
**Gate:** `just lint && just typecheck && just contracts && just rules && just test`.
**Why it matters:** no network code merges before this. A fixture captured under a schema
that later changes is a fixture lost, and the whole downstream corpus depends on it.

### M02 — Egress is provably single-path (design **G2**)
**Packets:** WP03, WP04, WP05, WP06, WP07, WP08.
**Evidence:** the full `MockTransport` matrix; SSRF proved against the **resolving**
transport with a stub resolver (not `MockTransport`); `L-01b` reports **zero** socket calls
from the core; `context.route()` denial aborts an in-page request; `just browsers` installs
binaries.
**Gate:** `just lint && just typecheck && just rules && just test && just browsers`.

### M03 — The core is pure and the store survives a crash (design **G3**)
**Packets:** WP09, WP10, WP11, WP12.
**Evidence:** `test_incomplete_run_never_closes` (Hypothesis) green;
`test_malformed_block_isolated` green; `test_kill9_midrun_resumes_without_closing` and
`test_stale_lease_writes_rejected` green; `L-01` allow-list holds with `extruct` in scope.
**Gate:** `just lint && just typecheck && just rules && just test`.
**Why it matters:** the completeness evaluator and the lifecycle reconciler exist **before**
any adapter. The proposal sequenced them after the first six adapters, which would have
shipped adapters whose acceptance criteria referenced closure behaviour that did not exist.

### M04 — First real evidence (design **G4**)
**Packets:** WP13, WP14, WP15, WP16.
**Evidence:** twelve anchor institutions each in exactly one resolution state; fixture
corpus with manifests and redaction proof; recon datasets validate under both engines.
**Gate:** `just lint && just rules && just test && just contracts && just recon-anchor`.

### M05 — Assumptions settled (design **G5**)
**Packets:** WP17.
**Evidence:** twelve spike records; **A-1/LD-11** and **A-4/LD-10** confirmed or amended
with evidence; HTTP/2 decided on measurement.
**Gate:** `just test && just rules && just spikes`.
**Why it matters:** this is the last cheap moment to reverse the two provisional library
decisions. Adapter work starts after it, not before.

### M06 — Portfolio and coverage (design **G6 + G7**)
**Packets:** WP18, WP19, WP19b, WP19c, WP20, WP21, WP22, WP23.
**Evidence:** remaining 18 Wave 1 brands observed twice (stability dimension populated);
schema **frozen** — new facts require a schema version, not a field edit; recon §13.3
thresholds reported; `test_no_brand_silently_omitted` green.
**Gate:** the full final matrix (§6).

## 6. Decommission

### DB01 — the toolchain seed
**Surface:** `src/jobsquery/__init__.py::main` (2 LOC) and the `[project.scripts]` entry
pointing at it.
**Prerequisite:** none for removal (WP01); the replacement CLI lands in WP22.
**Negative proof:** `rg -n 'def main' src/` → zero hits, asserted from WP01 onward.
**Positive proof:** `jobsquery --help` exits 0 at WP22.

There are **no other decommission batches**, because there is no other legacy authority:
the repository's entire executable inheritance is those two lines
(`rg -n '^(def|class|async def) ' src/` returns one hit). Three surfaces the design marked
`replace` or `delete` are **documents**, not code — spec §7's contracts, §7.10's protocol,
and recon §9's observation contract are superseded by design §4.2/§4.3/§4.4 and are never
implemented. The corresponding zero-state proofs are the `rg` assertions in WP14
(`SourceFamilyManifest`, `assess_completeness`) and WP20 (no `.yaml` under `adapters/`).

## 7. Final gate matrix

The justfile is the gate registry (`validation-policy.md` §4) and **does not yet exist** —
WP01 creates it. `just --list` currently reports `error: no justfile found`; `just` 1.58.0,
`uv` 0.12.5, and `ast-grep` are on PATH, while `ruff` and `pytest` are not installed.

Recipes this plan requires WP01 to define:

| Recipe | Covers |
|---|---|
| `just lint` | `ruff format --check` + `ruff check` |
| `just typecheck` | narrow type diagnostics over `src/` |
| `just test` | `pytest` with the `anyio` plugin (**not** `pytest-asyncio`, LD-17) |
| `just rules` | all `ast-grep` rules + `rule-tests/` + the `rg` deny-lists for L-05/L-07 + the `L-01b` socket assertion |
| `just contracts` | regenerate `contracts/*.schema.json` with `$schema` injected; fail on diff |
| `just browsers` | `python -m playwright install --with-deps chromium` |
| `just recon-anchor` | anchor-batch reconnaissance run |
| `just spikes` | replay the offline portion of every spike |
| `just export` | JSONL + CSV export, schema-validated |
| `just audit` | coverage report; non-zero if any brand lacks a terminal state |
| `just smoke` | live non-empty smoke tests (WP20/WP21; outside the offline suite) |
| `just metrics` | emit recon §15's six metric families as derived views (WP23) |

**Final matrix (M06):**
`just lint && just typecheck && just rules && just contracts && just test && just audit`,
with `just smoke` run separately because it touches the network.

**Baseline failures.** The repository has no test suite, so the baseline is trivially
green and every failure from WP01 onward is plan-caused. This is recorded so that a later
executor does not mistake an inherited failure for one it introduced — there are none to
inherit.

**Single-language scope.** This is a pure-Python project (no `Cargo.toml`, no native
extension). A Python-only final gate is correct here, and that is a verified fact rather
than an omission.

## 8. Execution sequence

```text
Generated from each packet's declared Dependencies field. A hand-drawn earlier version
contradicted seven packets; this one is derived, so it cannot.

M01  WP01 ─▶ WP02                                          (G1 · contracts frozen)

M02  WP02 ─▶ WP03 ─▶ WP04 ─┐
     WP02 ─▶ WP05 ─────────┼─▶ WP06 ─▶ WP07 ─▶ WP08        (G2 · one egress path)

M03  WP02 ─▶ WP10 ─────────┐
     WP07 ─▶ WP09 ─────────┴─▶ WP11 ─▶ WP12                (G3 · pure core, crash-safe)
                                       ▲ also WP02, WP05

M04  WP07,WP08,WP10,WP12 ─▶ WP13 ─▶ WP14 ─▶ WP15 ─▶ WP16   (G4 · anchor recon)
                                             ▲ also WP09, WP12

M05  WP16 ─▶ WP17                                          (G5 · assumptions settled)

M06  WP17 ─▶ WP18 ─▶ WP19 / WP19b / WP19c ─┐
     WP17 ─▶ WP20 ─▶ WP21 ─────────────────┴─▶ WP22 ─▶ WP23                (G6+G7 · coverage, observability)
                                       ▲ also WP13
```

**Parallelism — read from the graph, not from intuition.** Genuinely independent:
**WP03 and WP05** (both need only WP02); **WP10** (needs only WP02, so it can start at M02
time and wait for nothing). After WP17 the two adapter chains **WP18→WP19** and
**WP20→WP21** are independent *of each other* and are the natural delegation boundary —
but **WP22 needs all four**, and **WP23 needs WP22**. An earlier draft called WP21 and WP23
independent post-WP17 workstreams; they are not, and scheduling worktrees from that claim
would start them before their inputs exist. WP09 is likewise **not** free to run beside
WP10 — it sits behind WP07 for the charset contract.

Everything on the M01→M02 spine is strictly sequential: each layer is the next layer's
proof surface.

**The one ordering that is not negotiable.** WP10 (completeness) and WP11 (lifecycle) land
at M03, **before** WP14's adapter port and before any adapter. Every adapter's contract test
asserts against `classify()`'s verdicts, and `test_incomplete_run_never_closes` must exist
before code can violate it.

## 9. Plan risks and replan policy

### Distinguishing the three responses

- **Implementation adaptation** — stays inside the accepted design and invariants; recorded
  in execution state; no artifact changes. *Example:* `Client(default_encoding=)` will not
  take a callable, so decoding moves outside the client.
- **Plan revision** — packet boundaries, sequence, dependencies, or proof obligations
  change; a new plan version is written. *Example:* SPK-08 fails and Protego is adopted —
  the port is unchanged, but a dependency and its packet obligations change.
- **Design reopening** — architecture, a public contract, a library decision, or a target
  invariant changes; the design is revised first. *Example:* `context.route()` proves not to
  intercept some egress class, so I-05's enforcement mechanism moves.

### Named risks

| Risk | Probability | Response |
|---|---|---|
| **SPK-06 reverses LD-11** — Priority-2 families need full adapters | **High** — the design already records five of eight as irregular | Plan revision, pre-scoped: WP20 shrinks, WP14's port is untouched. Anticipated and cheap. |
| **A-3** — `context.route()` misses an egress class | Medium | **Design reopening.** I-05 moves to container egress policy. |
| **A-4** — stdlib robots fails an RFC case in the wild | Medium | Plan revision: Protego behind the unchanged one-method port. |
| A Wave 1 brand has no automatable public source | Medium | Neither. A `documented_gap` with a terminal state **is a correct outcome** (A-6). The failure mode to avoid is silent omission, which WP22 fails the build on. |
| Custom resolving transport cannot see the resolved address pre-connect | Low-medium | Drop to an `httpcore2` backend (adaptation), else A-3's path. |
| Vocabulary freeze at M06 proves premature | Medium | Schema version bump; the minimum-viable-vocabulary pass at WP02 exists to make this cheap. |
| Playwright/browser version coupling breaks CI | Low | Pin both; `just browsers` is a gate, not a README step. |

### Standing replan triggers (any packet)

A planned library API is absent or behaves differently from the design's probed basis; the
consumer surface is materially larger than the preflight query suggested; a packet cannot be
left dependency-closed; a target invariant cannot be satisfied by the planned mechanism;
migration would require unbounded dual authority; or security, reliability, or operational
evidence invalidates the approach.

**One trigger overrides all others.** If any change would make an incomplete run capable of
closing a job (I-09), or introduce a second network path (I-05), it is a design reopening
regardless of how small the code change looks.

---

## 10. Audit record

An independent auditor with fresh context reviewed this plan against the design, the
validation policy, and the plan schema, and probed the repository for ground truth. It
returned **five blocking and six material findings**. All eleven changed the plan; two also
amended the design. Indexed here because a reader should know which parts survived review.

| Finding | Verdict | Change |
|---|---|---|
| Three of six declared-input digests did not recompute; "No drift" was false | **Upheld** | Digests recomputed; §2 now records that `pyproject.toml`/`uv.lock` were modified **at 20:26:42, mid-planning** |
| The dependency set is 11, not 8; `pytest`/`ruff`/`pyrefly` are **runtime** deps; lock is 52 packages, not 44 | **Upheld** — verified | `WP01` rewritten: re-tier three deps into a dev group, pin the eight runtime deps, correct every derived count. Design §2.2 amended. |
| `just typecheck` is a gate in twelve packets and nothing creates it — while `pyrefly` sat pinned and uncited | **Upheld** | `WP01.3` names pyrefly; **LD-19** added to the design, which had no type-checker record at all |
| `WP16`'s circularity escape pointed at the **wrong runner** — anchor institutions are Priority 1 (`run_faceted_endpoint`), not Priority 2 (`run_board`) | **Upheld** | `run_faceted_endpoint` + the six-condition promotion rule **hoisted into `WP14`**; `WP18` keeps Workday specifics. The cycle is now broken structurally, not by prose. |
| §8's graph contradicted seven packets' declared dependencies; the parallelism claims contradicted two more | **Upheld** | Graph **regenerated from the `Dependencies` fields** and verified acyclic; parallelism rewritten from the graph |
| `WP01` cannot pass its own checks (`pytest` exits **5** on an empty tree); `L-01b` double-owned; "six ast-grep rules" wrong four ways | **Upheld** — exit 5 verified | Rules split by instrument (`ast-grep` / `rg` / `pytest`); `L-01b` authored only in `WP06`; `just test` maps 5→0 for `WP01` alone |
| Several zero-state checks are malformed shell; two pass with the forbidden artifact present | **Upheld** | Six checks rewritten. `rg -n '\.yaml'` searched *contents*; it is now `rg --files -g '*.y*ml'`. Directory checks moved to the packet that creates the directory. |
| LD-03's headline probe (Lexbor ignores `<meta charset>` on bytes) had **no oracle** in the packet that owns the parser | **Upheld** | `WP09` gains `::test_lexbor_bytes_vs_str_diverges` and rule **L-09** |
| `WP10` builds the twelve-rule anomaly machinery with zero checks; §8.2b's "most damaging realistic failure" had no oracle anywhere | **Upheld** | Two checks added to `WP10`; `FailureCategory` moved from `WP23` to `WP02` so shell packets classify as they go rather than retrofitting at M06 |
| I-05's mechanism rests on an API the design never probed; the `httpcore2` fallback was misclassified as an adaptation | **Upheld** | `WP06` gains a **blocking preflight probe**; the fallback is reclassified as a plan revision; `::test_validated_address_is_the_connected_address` added for DNS rebinding |
| `WP02` and `WP19` are buckets; `WP02`'s G1 check was exactly the one design §7 forbade | **Upheld** | `test_all_named_contracts_exist` added (literal list, not registry iteration); `WP19` split into `WP19`/`WP19b`/`WP19c` |

Four governance rules the plan had added ad hoc without ids — the XML-parser factory rule,
the Lexbor-bytes rule, the failure-category rule, and the no-adapter-verdict rule — are now
minted as **L-08 … L-11** in design §8.1 rather than living only in packet prose.

**What the audit cost, and what it argues.** Three of the five blocking findings were
*ground-truth* errors: a dependency set that changed under the plan while it was being
written, and a gate cited sixteen times with nothing behind it. No amount of internal
consistency checking would have caught them, because the plan was self-consistent — it was
consistent with a repository state that had stopped being true. The lesson for execution is
in §9's standing triggers: re-verify current-tree facts before load-bearing edits, because
the tree moves.
