---
artifact: design-dossier
design_id: career-source-acquisition
version: v1
date: 2026-08-20
status: accepted
baseline_commit: 4daded6d8cc1764cd2e38491ea035468a039950b
working_tree_digest: 2eb3cbd5489d433a
primary_scope:
  - src/jobsquery/
  - docs/designs/
  - docs/plans/
doctrine_path: docs/library_ref/semantic_design_principles_holistic.md
---

# Career-Source Acquisition — Target Design v1

## 1. Executive decision

Build JobsQuery as a **ports-and-adapters pipeline with a pure functional core, a single
narrow-authority egress gate, and declarative source-family manifests**, adopting the two
proposal documents' *policy* layer nearly intact while replacing their *contract* and
*adapter-protocol* layer.

Five decisions define the design:

1. **One egress gate.** Every byte the system fetches — HTTP or in-browser — passes one
   `HostPolicyGate`. Playwright traffic is routed through the same decision function via
   `context.route()`, so browser navigation cannot bypass scheme, IDNA, PSL, IP-class,
   robots, redirect, or size policy. The proposal left browser egress ungoverned.
2. **A pure core.** Extraction, pagination guarding, completeness classification,
   normalization, identity resolution, lifecycle reconciliation, and mandate matching are
   pure functions of bytes and records — no network, no clock, no database. This is what
   makes the proposal's own fixture-first mandate achievable rather than aspirational.
3. **Completeness is derived, never authored.** Adapters emit structured evidence across
   the ten dimensions the proposal itself defines; one pure `classify()` owns every
   verdict. The proposal let each adapter self-certify `EXACT`.
4. **Shared execution, not a configuration language.** Recurring board families are
   expressed as per-family `BoardSpec` **Python literals** consumed by one shared
   `run_board()`, so pagination, identity, and field mapping are reused without inventing a
   DSL. Irregular behaviour is a callable in the spec, not a schema extension. Onboarding a
   new employer changes registry data; onboarding a new board family adds one small module.
   **This replaces an earlier YAML-manifest design that an adversarial review falsified** —
   see D-10 and LD-11.
5. **Reconnaissance is a run mode, not a second system.** `CrawlRun.mode = recon |
   production` replaces an entire parallel contract family whose two published shapes
   already disagreed with each other.

**What is preserved:** the 25 non-negotiable invariants, the ten completeness dimensions
and their semantics, the browser-escalation ladder and its six-condition promotion rule,
the fixture-capture protocol, the 21-category failure taxonomy, the source-classification
model, and every fixed input — the 30 Wave 1 institutions, the anchor batch, the mandate
and geography vocabularies, and the web-source ladder.

**What is rejected:** nine structural defects in the proposal's contracts (§3), four
proposed storage dependencies the stdlib or SQLite already cover (§5), a crawl-framework
substrate (§6.3), and — after review — a declarative manifest DSL (§6.6).

**Where the effort goes.** Spec §5 ranks Workday · Oracle CX · Taleo · iCIMS · SAP
SuccessFactors · Eightfold · Phenom · Avature · proprietary sites as **Priority 1**
("likely to determine coverage of the largest target employers") and the eight uniform
board APIs as **Priority 2** ("**less likely to dominate large-bank coverage**"). The
generalization worth finding is therefore in Priority 1, where all nine share one shape —
*browser-discover an endpoint, then replay it over HTTP with a faceted query* — not in a
field-map abstraction over Priority 2.

**The measure of success** is unchanged from the proposal and worth restating, because it
determines what the proof strategy must show: not the sophistication of role matching, but
whether the system can make a **credible, auditable statement that it has repeatedly
captured the complete public job inventory of every employer in its declared universe —
and can show exactly where that statement remains uncertain.**

## 2. Constraints and target invariants

### 2.1 Fixed inputs (not open to design deviation)

The user has fixed two inputs. They are reproduced as **data**, owned by the source
registry, never as code constants.

**Wave 1 institution universe — 30 employment brands** (recon §4.2). Group A, large
U.S. banks and capital markets (12): JPMorganChase · Citi · Goldman Sachs · Morgan
Stanley · Bank of America · BNY · Wells Fargo · Capital One · HSBC · Barclays ·
Deutsche Bank · UBS. Group B, asset management, alternatives, custody (6): BlackRock ·
Blackstone · State Street · Apollo Global Management · KKR · TIAA. Group C, insurance
(6): MetLife · New York Life · AIG · Prudential Financial · Chubb · Guardian Life.
Group D, payments, market infrastructure, public financial institutions (6): American
Express · Mastercard · DTCC · Intercontinental Exchange/NYSE · Nasdaq · Federal Reserve
Bank of New York.

**Anchor batch — first 12, in order** (recon §4.4): JPMorganChase, Citi, Goldman Sachs,
Morgan Stanley, BNY, BlackRock, MetLife, New York Life, American Express, DTCC,
Intercontinental Exchange/NYSE, Federal Reserve Bank of New York.

**Wave 2 expansion — 15 brands** (recon §4.3): BNP Paribas · Société Générale · MUFG ·
Mizuho · RBC · TD · Santander · Macquarie · Visa · S&P Global · Moody's · Marsh
McLennan · MassMutual · Invesco · Franklin Templeton.

**Web-source ladder** (spec §4.2, §4.7) — the vendor-independent surfaces every brand is
checked against, in order: `robots.txt` → sitemap index → URL sitemap → RSS/Atom → XML
job feed → JSON feed → JSON-LD `JobPosting` → Microdata → canonical link → Open Graph →
static listing HTML → static detail HTML → embedded script state → public XHR/`fetch` →
public GraphQL → browser DOM.

**ATS families with dedicated detection**, reproduced in full from spec §4.3–§4.6 —
**fifteen** enterprise families, not the five an earlier draft of this design listed:

- *Priority 2, uniform public board APIs* (spec §4.3): Greenhouse · Lever · Ashby ·
  Workable · Recruitee · Personio · Teamtailor · SmartRecruiters.
- *Priority 1, core enterprise ATS* (spec §4.4, all fifteen rows): **Workday Recruiting ·
  Oracle Recruiting Candidate Experience · Oracle Taleo · SAP SuccessFactors Recruiting ·
  iCIMS · Jobvite · Dayforce · ADP Recruiting · Cornerstone (`*.csod.com`) · IBM
  Kenexa/BrassRing · PageUp · UKG Recruiting · Paylocity · Paycom · Avature.**
- *Candidate-experience overlays* (spec §4.5): Phenom · Eightfold · Beamery · Radancy ·
  Avature · Symphony Talent/SmashFly.
- *Long-tail detection catalog* (spec §4.6), routed to generic handling, never dedicated
  adapters: BambooHR · Pinpoint · Comeet · Breezy HR · JazzHR/ApplyToJob · JobScore ·
  ApplicantPro · ClearCompany · ApplicantStack · TalentReef · Fountain · Workstream ·
  Crelate · Zoho Recruit · Manatal · Hireology · Rippling Recruiting · Oracle NetSuite.

Spec §4.4 also records three enumeration-shaping facts that the adapter design must carry:
Workday and Oracle both support **multiple external career sites per employer** (so one
brand may need several boards for full coverage); Oracle's **location-facet counts count
location occurrences, not requisitions** (a trap for count reconciliation); and iCIMS's
XML feed requires OAuth approval, so the **public employer portal is the source**, never an
assumed partner feed.

**Mandate and geography probe vocabularies** (recon §5) — 18 high-signal terms, 13
adjacent terms, `NYC_CORE` (7 values), `NYC_METRO_EXTENDED` (5 + open), remote taxonomy
(`remote_us`, `remote_ny`, `remote_unspecified`, `hybrid_nyc`, `unknown`). These are
**validation queries applied after enumeration, never ingestion filters** — the single
most load-bearing constraint in the whole system.

**Audit-only sources** (spec §4.8): LinkedIn, Google/Bing job search, Indeed, Glassdoor,
eFinancialCareers, Built In NYC, ZipRecruiter, Common Crawl, Wayback. Used for
miss-discovery and coverage audit only, never as a source of record. Automated LinkedIn
scraping is out of scope.

### 2.2 Environment constraints (observed)

| Fact | Evidence |
|---|---|
| Python 3.14.7; pure-Python project, no Rust | `.venv/bin/python -V`; `pyproject.toml` `requires-python` |
| Greenfield: 2 LOC of source, no tests, no gates, no CI | `rg --files src/`; `rg -n '^(def\|class\|async def) ' src/` |
| **11** direct deps declared as `>=` **floors, not pins**; **52** packages resolved; **no dev-dependency group**, and `pytest`/`ruff`/`pyrefly` are declared as **runtime** deps *(amended 20:26:42 — was 8 deps / 44 packages when this design was accepted)* | `pyproject.toml`; `grep -c '^name = ' uv.lock` — re-tiering and exact pinning are a G1 deliverable (§8.4, LD-19) |
| No `pytest`, `ruff`, `justfile`, `rules/`, `sgconfig.yml` | `ls`; `.venv/bin/` listing |

### 2.3 Target invariants

Spec §15 states 25 non-negotiable invariants. Sixteen are adopted as written, several
are merged, and seven are added; they are restated below as **`I-nn`** so plan packets can cite them. Each is written to be
mechanically checkable rather than aspirational; §6 names the check.

| ID | Invariant | Doctrine |
|---|---|---|
| **I-01** | Enumeration retrieves **all** public postings for a source. Mandate and geography vocabularies are applied only to already-stored postings. A search-definition change never requires a recrawl. | P2, P21 |
| **I-02** | Official employer-controlled sources are the only source of record. Aggregators are audit-only. | P8 |
| **I-03** | Every acquisition writes an immutable, content-addressed raw artifact before any parsing occurs. Every canonical posting traces to the artifacts that produced it. | P19, P27 |
| **I-04** | Acquisition, parsing, and normalization are separately invocable. Parsing runs offline from stored bytes with no network access. | P17, P30 |
| **I-05** | Every **client-originated and page-originated** request — HTTP and browser alike — passes the single `HostPolicyGate`, enforced by an allow-list import gate and a socket-level egress assertion, not by a name deny-list. *(Restated after review: "every byte" was unprovable — Chromium's own process egress never reaches a route handler.)* | P8 |
| **I-06** | Direct HTTP is preferred; browser acquisition requires a recorded justification per source, and every browser-discovered endpoint is tested for HTTP replay. | P5 |
| **I-07** | Completeness is **derived from structured evidence**, never authored. HTTP success alone never implies completeness. | P11, P10 |
| **I-08** | A zero-job result is publishable only from a run whose derived completeness admits it. Otherwise it is an anomaly, not a fact. | P23 |
| **I-09** | Incomplete or failed runs cannot close jobs. Lifecycle transitions to `closed` require an enumeration-complete run. | P23, P24 |
| **I-10** | Enumeration completeness and content-refresh completeness are separate verdicts. A failed detail fetch never overwrites good content with nulls. | P19, P23 |
| **I-11** | Job identity derives from a stable public source identifier scoped to its career source — never from URL, row order, or list position. **Where an overlay fronts a different ATS, both the overlay's source job ID and the underlying ATS/requisition ID are preserved** (spec §4.5). | P13 |
| **I-12** | All locations are preserved as a list; the raw source location string is never replaced by its normalized form. | P12, P27 |
| **I-13** | Spec §4.5's "critical modeling rule" is preserved in full: **presentation platform · enumeration interface · detail interface · application platform · canonical employer** are five separately recorded facts, not two. An overlay's enumeration interface is routinely not the ATS behind its apply link. | P29 |
| **I-14** | Adapters extend the system additively. Onboarding a new employer changes registry **data**; onboarding a new uniform ATS family changes a **manifest**; only irregular platforms may add code. | P31, §12 litmus |
| **I-15** | Every recurring adapter passes the shared contract-test suite against offline fixtures. | P30 |
| **I-16** | Every durable record validates under both its Pydantic model and an independent Draft 2020-12 `jsonschema` check. Schema references resolve locally; network retrieval is disabled. | P29, P16 |
| **I-17** | Source drift produces recorded degradation and review, never silent data loss. | P23, P28 |
| **I-18** | Every failure is classified into exactly one category of the failure taxonomy. No generic "run failed". | P23 |
| **I-19** | Re-running any stage with the same effective inputs is idempotent and does not corrupt state. | P24 |
| **I-20** | Domain facts record the PSL policy and snapshot identity that produced them; hostname canonicalization is explicit and separate from PSL decomposition. | P25, P13 |
| **I-21** | No credential, authenticated session, application submission, CAPTCHA bypass, stealth tooling, or access-control circumvention exists anywhere in the system. | P8 |
| **I-22** | Embedded structured metadata **never** proves complete enumeration. It is a field source only; enumeration completeness comes from pagination and count evidence (spec §15.16). | P23, P27 |
| **I-23** | Personalized or recommendation-driven result sets are **never** treated as the full job inventory. A context that may be personalized cannot produce a closure-eligible run (spec §15.17, §10.8). | P23 |

## 3. Why the proposal is reshaped, not adopted

The two proposal documents are **policy-strong and structure-weak**. Their invariants (spec
§15), completeness semantics (§10), browser-escalation machine (§8.4), fixture protocol
(recon §10), and source classification (recon §7) are excellent and are adopted almost
intact. Their *contracts* (spec §7) and *adapter protocol* (§7.10) contain defects that
would make several of their own invariants unimplementable. Those defects, not preference,
drive every deviation in §4.

**One row below is a withdrawal.** D-2 originally asserted a contradiction that does not
exist in the source document. It is kept, struck and corrected, rather than quietly
deleted — a design that trades on precision about someone else's contracts owes the same
standard to its own.

| # | Defect (observed in the proposal) | Consequence | Resolution |
|---|---|---|---|
| **D-1** | `CareerSource.employer_id` is single-valued, but §7.1 requires "a shared career source must not be crawled repeatedly per subsidiary". | Shared boards across subsidiaries are **unrepresentable**; the invariant cannot be satisfied. | Many brands ↔ one source via an explicit link table with per-brand scope. |
| **D-2** | `RawArtifact` mixes a per-run fetch record with content addressing in one contract. *(Correction after review: this is **not** a contradiction — §7.4's `raw_content_reference` is a pointer, so the row is per-run while bytes are shared. The original claim that dedup "blocks itself" was wrong and is withdrawn.)* | The blob/observation boundary is implicit, and the two hashes it needs are easy to lose. | Make it explicit: **`ContentBlob`** (content-addressed, immutable, run-free, carrying **both** `wire_hash` and `decoded_hash`) and **`FetchObservation`** (per-run). |
| **D-3** | `enumerate_jobs()` returns a whole `EnumerationResult`; the caller never sees `EnumerationPage`. | §10.4's 11 pagination safeguards must be re-implemented inside every adapter — violating invariant #5 ("adapters cannot bypass shared policy") for the *most* safety-critical policy. | Enumeration is an **async iterator of `EnumerationPage`**. Safeguards run in shared policy over the stream. |
| **D-4** | `assess_completeness()` is an adapter method returning a verdict. | Each adapter can self-certify `EXACT`. Completeness is global policy (§10.1) delegated to the least trusted layer. | Adapters emit **evidence only**. A single pure `classify()` owns every verdict. |
| **D-5** | `PARTIAL`, `FAILED`, `BLOCKED`, `INDETERMINATE` each name a source state (§8.1), a run terminal (§8.2), **and** a completeness class (§10.1). | Three different claims share one token; no artifact can disambiguate them. | Three disjoint namespaced enums: `SourceState`, `RunState`, `Completeness`. No bare tokens anywhere. |
| **D-6** | `CompletenessAssessment` has 7 status fields; §10.2 defines 10 dimensions. Listing-coverage, personalization, truncation and stability have no field — yet §10.8 requires "run not personalized" as an independent closure precondition. | The closure predicate cannot be evaluated from the contract. | `CompletenessEvidence` carries all **10** dimensions 1:1, each with a value vocabulary. |
| **D-7** | Four contracts are referenced but never defined: **`CanonicalJob`**, **`CrawlRun`**, **`SourceObservation`**, **`LegalEntity`** — plus `ProbeResult`, `CrawlRunEvidence`, `DetailAcquisitionResult`, and all five gateway return types. Invariant #19 (traceability) is therefore unsatisfiable. | The system's *output* contract is undefined; §16's 25-field dataset is its only specification. | All defined in §4.2. `CanonicalJob` is derived from §16's 25 fields. |
| **D-8** | `board_scope` is a flat 9-value enum conflating organizational reach, candidate population, and escape hatches — while `regions`/`locales` encode geography a second time. §10.2 asks for five axes. | Scope is the gate on every employer-level completeness claim, and it cannot express what the gate needs. | Structured `SourceScope` object: reach × population × regions × locales × languages. |
| **D-9** | Capability flags are declared per-adapter **class**, but §9.4/§9.5 describe per-tenant probing (Workable modes, Recruitee access mode, browser necessity). | A class-level flag cannot describe a per-tenant fact. | Capabilities move to `ProbeResult`/`CareerSource` (per-instance). The manifest declares defaults. |
| **D-10** | The sixteen adapter specs repeat pagination, identity, and field-mapping structure. | Sixteen near-identical control loops; §10.4's safeguards duplicated per adapter. | **One shared `run_board(spec)`** with per-family `BoardSpec` literals. *(An earlier version of this row claimed eight families "differ **only**" along eight declarative axes and proposed a YAML DSL. Review falsified it — five of the eight need behaviour a schema cannot carry. See §6.6.)* |

| **D-11** | §7.9's `closure_allowed` is presented as derivable from one run's assessment, but §10.8's predicate depends on §10.9's twelve **cross-run** anomaly rules, which have no contract anywhere. | The product's central safety predicate is uncomputable as specified. *(An earlier draft of this design reproduced the same error.)* | `classify(evidence, prior, anomalies)`; new `SourceAnomaly` and `PriorRunSummary` contracts; `closure_allowed = False` when no prior exists. |

Two internal contradictions are resolved by decision rather than reshaping:

- **§10.6 step 6 ("include an unknown partition *where possible*") vs `PARTITIONED`'s
  "no unknown category is omitted".** Decision: `PARTITIONED` requires demonstrated
  partition coverage — either an unknown/uncategorized partition was enumerated, **or**
  the union was independently reconciled against an advertised total. Absent both, the
  verdict is `INDETERMINATE`. Partitioning by NYC remains prohibited (§10.6).
- **§10.5's valid-zero bar is unreachable for `TERMINAL`-class sources**, which by
  definition have no trustworthy total. Decision: a valid zero requires *either* an
  advertised zero *or* an explicit empty collection *or* a normally-terminated
  enumeration on a source whose prior verdict was `TERMINAL`, plus the other five §10.5
  conditions. Mass closure from a zero still requires a second confirming run when the
  prior complete run was non-empty.

Two vocabularies are unified rather than kept in parallel. The recon plan's completeness
grades **A/B/C/D/M/X** and the spec's eight **classes** describe the same judgement. Both
become pure projections of one `CompletenessEvidence` record:
`A≡EXACT`, `B≡TERMINAL`, `C≡INDETERMINATE (heuristic-only enumeration)`,
`D≡PARTIAL`, `M≡MANUAL`, `X≡BLOCKED`, with `PARTITIONED` projecting to `A` when its
coverage is reconciled and `FAILED` projecting to `D`. Neither vocabulary is stored.

## 4. Target architecture

### 4.1 Shape: functional core, imperative shell, one gate

Three rings. Dependencies point **inward only**; the compile-time check is L-01 (§8).

```text
┌────────────────────────────────────────────────────────────────────────────┐
│ SHELL — I/O, orchestration, retries, leases, browser lifecycle (impure)     │
│                                                                            │
│   Scheduler ─▶ CrawlRunner ─▶ PageGuard ─▶ ┌──────────────────────────┐   │
│                                 │  HostPolicyGate   (SOLE egress path) │   │
│                                 │  scheme · credentials · IDNA · PSL   │   │
│                                 │  IP class · robots · size · redirect │   │
│                                 │  revalidation · per-host lease       │   │
│                                 └───────┬──────────────────┬───────────┘   │
│                                  HttpFetcher        BrowserFetcher         │
│                                   (httpx2)      (playwright + route())     │
│                                         └────────┬───────┘                 │
│                                            ContentStore (CAS)              │
└────────────────────────────────────────────────┼───────────────────────────┘
                                                 │ bytes + FetchObservation
┌────────────────────────────────────────────────▼───────────────────────────┐
│ CORE — pure, deterministic, no I/O, no clock, no network                   │
│                                                                            │
│   extract()      bytes + manifest ─▶ EnumerationPage | ParsedSourcePosting │
│   guard()        page stream       ─▶ TruncationSignal[]                   │
│   classify()     CompletenessEvidence ─▶ CompletenessVerdict               │
│   normalize()    ParsedSourcePosting  ─▶ CanonicalJob fields               │
│   resolve()      SourceJobRef[]       ─▶ JobIdentity                       │
│   reconcile()    prior + refs + verdict ─▶ LifecycleTransition[]           │
│   match()        CanonicalJob + probe vocabulary ─▶ MandateMatch[]         │
└────────────────────────────────────────────────┬───────────────────────────┘
                                                 │ typed records
┌────────────────────────────────────────────────▼───────────────────────────┐
│ CONTRACTS — Pydantic v2 models; published as JSON Schema 2020-12           │
└────────────────────────────────────────────────────────────────────────────┘
```

**`guard` is a shell object, not a core function** — a correction from review. Deciding
*when to stop fetching* is control flow over I/O: three of §10.4's eleven safeguards are
inherently impure (a page-count budget, mid-run insertion detection, and the end-of-run
first-page rerun, which is a network request). So `PageGuard` lives in the shell and drives
the stream, while the **eleven signal predicates stay pure** — `signal(page, history) ->
TruncationSignal?` — which is what fixture-testability actually requires. D-3's property
survives: there is still exactly one implementation of the safeguards, shared by every
adapter, rather than sixteen private ones.

**The clock and entropy are injected, never ambient.** `reconcile()` needs `first_seen` /
`last_seen` and the retry `decide()` needs jitter; both take them as parameters. Lifecycle
ordering derives from **run identity**, not wall-clock timestamps, so two workers with
skewed clocks cannot produce a non-monotonic `last_seen`.

Everything else in CORE is a pure function of its arguments. That is what makes the
fixture-first test architecture (spec §12, recon §10) possible at all: `extract`,
`guard`, `classify`, `normalize`, `resolve`, `reconcile` and `match` are tested from
stored bytes with no network, no browser, and no clock. It also satisfies I-04 directly —
parsing is not merely *separable* from acquisition, it is *incapable* of acquiring.

**`HostPolicyGate` is the only egress path (I-05).** The proposal placed robots and host
policy in a component *beside* the gateway, and gave Playwright its own lifecycle. That
leaves browser egress ungoverned: a page navigation issues dozens of requests that never
pass an HTTPX-level check. Here, `BrowserFetcher` installs `context.route("**/*")` and
routes every in-browser request through the same `HostPolicyGate` decision function that
`HttpFetcher` uses, aborting on denial. One policy, two transports, no second path.

### 4.2 Contracts

Reshaped and new contracts only; unchanged ones are adopted from spec §7 as written.

**Identity and registry (durable domain truth — P19).**

| Contract | Shape | Resolves |
|---|---|---|
| `LegalEntity` | `entity_id`, `legal_name`, `jurisdiction`, `regulatory_ids{FFIEC RSSD, FDIC cert, LEI, CIK, CRD, NAIC}`, `brand_id?` | D-7 |
| `EmploymentBrand` | as spec §7.1, minus `legal_entity_ids` (now a reverse FK) | — |
| `BrandSourceLink` | `brand_id`, `source_id`, `scope: SourceScope`, `evidence: DiscoveryEvidence`, `confidence` | **D-1** — many brands ↔ one source |
| `CareerSource` | as spec §7.2 **minus** `employer_id`, `board_scope`, `check_interval`; **plus** `scope: SourceScope`, `schedule_policy_id` | D-1, D-8 |
| `SourceScope` | a **discriminated union** — `UnresolvedScope(evidence_gap)` or `ResolvedScope(reach, population, locales[])` — so "scope not established" is a *different type*, not a flag beside populated fields. `locales` subsumes language+region; the proposal encoded geography twice. | **D-8** |
| `SourceFamilyManifest` | see §4.3 | **D-10** |

Spec §7.2's binding invariant is "an unresolved scope blocks claims of employer-level
completeness". A `resolved: bool` sitting beside populated fields leaves the illegal state
representable — so scope is a union, and `classify()` cannot receive an `UnresolvedScope`
and return `enumeration_complete=True`, because the types forbid it (P12).

**Acquisition (temporal control truth + immutable evidence).**

| Contract | Shape | Resolves |
|---|---|---|
| `ContentBlob` | **`wire_hash`** (identity of the bytes actually received — the I-03 immutability anchor and §8.4 reproducibility oracle) **and `decoded_hash`** (the dedup key, stable when a CDN switches gzip→br), plus `byte_length`, `stored_encoding`, `stored_at`. **No `run_id`, no URL.** | **D-2** — a single hash fails dedup on any content-encoding change |
| `FetchObservation` | `observation_id`, `run_id`, `source_id`, `request{method,url,safe_headers,body_hash}`, `response{status,headers,final_url,redirect_chain,content_type}`, `wire_hash → ContentBlob`, `transport: http\|browser_network\|browser_dom`, `role: robots\|sitemap\|listing\|detail\|rendered\|har\|trace\|script_config\|error`, `fetched_at` | **D-2**, and splits spec §7.4's `artifact_type` into orthogonal `role` × `content_type` |
| `CrawlRun` | `run_id`, `source_id`, `state: RunState`, `planned_at`, `lease{holder,expires_at}`, `mode: recon\|production`, `counters`, `failures[]` | **D-7** |
| `EnumerationPage` | as spec §7.6, **plus** `partition_key`, `observed_at`; **yielded**, not accumulated | **D-3** |
| `SourceObservation` | `source_id`, `run_id`, `source_job_id`, `observed_at`, `listing_position`, `partition_key`, `page_ordinal`, `blob_hash` — one row per (job, source, run) | **D-7**; makes §10.7's "preserve both observations" representable |

**Judgement (pure, derived).**

| Contract | Shape | Resolves |
|---|---|---|
| `CompletenessEvidence` | **10 fields, 1:1 with spec §10.2**: `scope`, `pagination`, `count`, `identity`, `listing_coverage`, `detail_coverage`, `parsing`, `personalization`, `truncation`, `stability` — each a small enum, plus `signals: TruncationSignal[]`, `advertised_total?`, `unique_ids`, `failed_pages[]`, `partitions[]` | **D-6** |
| `CompletenessVerdict` | **derived, never authored**: `completeness: Completeness`, `enumeration_complete`, `content_refresh_complete`, `reconciliation_allowed`, `closure_allowed`, `blocking_reasons[]`, `classifier_version` | **D-4** |
| `SourceAnomaly` | spec §10.9's **twelve** rules as named members (`count_zero`, `count_drop_beyond_threshold`, `count_rise_implausible`, `branding_changed`, `source_identifier_missing`, `selector_counts_shifted`, `json_shape_changed`, `sentinel_job_missing`, `pagination_terminated_early`, `all_locations_null`, `all_descriptions_empty`, `all_source_ids_changed`) + the thresholds they compare against | **D-11** — I-17 previously had no contract home |
| `PriorRunSummary` | `run_id`, `unique_id_count`, `source_identity_fingerprint`, `terminal_condition`, `selector_counts`, `sentinel_job_ids[]` — the minimum cross-run state `classify()` needs | **D-11** |
| `FailureRecord` | `category` (one of the 21 recon §16 categories), `stage`, `retryable`, `message`, `reference`, `resolution` | I-18 |

**Output (the contract §16 implied but never wrote).**

`CanonicalJob` — the 25 fields of spec §16, with three structural rules: every
nontrivially-normalized field keeps its raw counterpart (`locations_raw[]` alongside
`locations[]`, `compensation_raw` alongside `compensation`); `observations[]` links to
`SourceObservation` rows so I-03 traceability is a foreign key rather than a promise; and
`mandate_matches[]`/`geo_class` are **derived projections, recomputable without recrawl**
(I-01).

**`classify()` is cross-run, or `closure_allowed` is a lie.** Spec §10.8's closure predicate
has five conjuncts, and two of them — *source not anomalous* and *source identity
unchanged* — are defined by §10.9's twelve **comparative** rules. A function of one run's
evidence cannot evaluate them, so the signature is:

```python
def classify(evidence: CompletenessEvidence,
             prior: PriorRunSummary | None,
             anomalies: SourceAnomaly) -> CompletenessVerdict: ...   # still pure
```

Still pure — cross-run state is passed in, not fetched. **`closure_allowed` is `False`
whenever `prior is None`**, which makes a first run structurally incapable of closing a job
and a `reconcile()` that receives a verdict computed without a prior refuses it. This
corrects an error in an earlier draft that made `classify()` a function of single-run
evidence while promising a predicate it could not compute (D-11).

**Contracts referenced above whose shapes belong to the implementation plan, not here.**
Naming them explicitly, because this design's own D-7 complaint is that the proposal
referenced contracts it never defined — and an earlier draft of this design did the same:
`DiscoveryEvidence`, `ProbeResult`, `TruncationSignal`, `SourceJobRef`, `JobIdentity`,
`LifecycleTransition`, `MandateMatch`, `ExtractContext`, `RunEvidence`, `Fetcher`,
`RobotsDecision`, `SourceState`, `RunState`, `FieldAvailability`, `ProbeVocabulary`, and
the twelve `resolution_state` / three `resolution_confidence` values from recon §7.2/§7.3.
Their *existence and role* are design commitments; their *field lists* are a G1 deliverable.
**G1's exit condition enumerates all of them by name** (§7) — it does not say "all §4.2
contracts", which would have passed with the output contract undefined.

**Vocabulary namespacing (D-5).** `SourceState`, `RunState`, and `Completeness` are three
disjoint enums. No artifact stores a bare `PARTIAL`; it stores `RunState.PARTIAL` or
`Completeness.PARTIAL`. L-02 (§8) fails the build on a bare collision token.

### 4.3 Adapters: one shared board runner, no configuration language

An earlier version of this design made eight uniform ATS families **declarative YAML
manifests** executed by a generic interpreter. Adversarial review falsified it on evidence
(§6.6): five of the eight need behaviour a field-map schema cannot express, and spec §5
ranks all eight as **Priority 2 — "less likely to dominate large-bank coverage."** The
abstraction was aimed at the wrong half of the portfolio. What survives is the *reuse*, not
the *DSL*.

**One shared runner.** `run_board(spec, src, fetch)` owns the control loop that every board
family repeats — request, paginate, accumulate identity, yield pages — and is the single
place §10.4's eleven safeguards attach. Each family supplies a `BoardSpec`: a small frozen
Python object whose fields are values where behaviour is uniform and **callables where it
is not**.

```python
GREENHOUSE = BoardSpec(
    family="greenhouse",
    detect=greenhouse_detect,          # host list OR embedded board token in a careers link
    endpoint=lambda t: f"https://boards-api.greenhouse.io/v1/boards/{t}/jobs?content=true",
    paginate=NoPagination(),
    collection=lambda doc: doc["jobs"],
    identity=lambda j: SourceJobId(str(j["id"]), requisition=j.get("requisition_id")),
    fields=greenhouse_fields,          # returns raw AND normalized per field (I-12)
    advertised_total=None,             # see below — no *independent* total
)
```

This keeps every property that mattered — one control loop, one safeguard implementation,
one contract-test suite, one file per family — while the awkward cases stay expressible:
Lever advances `skip` by **records returned, not by `limit`**, and runs on both global and
EU hosts; Ashby resolves identity through a **three-level preference ladder** (posting ID →
job-URL path → apply-URL path); Teamtailor must detect **silent first-100 truncation** on
**custom career domains** that no static host list can enumerate; Workable enumerates by
**filtering a multi-tenant global XML feed**, with two-valued completeness; Recruitee runs a
**five-step branching probe** whose completeness semantics differ per branch; Personio needs
**multi-language fan-out with a join on position ID**; and SmartRecruiters' documented API
**requires a key**, so its strategy is probe-and-fallback with no stable URL. Each is a
function. None is a schema extension, and there is no interpreter to version, no `L-03`
review rule, and no "badly-specified programming language" failure mode to mitigate.

**Priority 1 is where the generalization actually is.** Workday, Oracle CX, Taleo, iCIMS,
SAP SuccessFactors, Eightfold, Phenom, Avature and proprietary sites all share one shape:
*browser-discover a public endpoint, prove it replays over plain HTTP, then drive it with a
faceted query and partition when it caps.* That is the abstraction worth building —
`run_faceted_endpoint(spec)` — and it is where the 30 Wave 1 institutions live. It is
sequenced **before** the Priority 2 runner (§7).

**The port (D-3, D-4, D-9):**

```python
class SourceAdapter(Protocol):
    family: str
    async def probe(self, src: CareerSource, fetch: Fetcher) -> ProbeResult: ...
    def enumerate(self, src, fetch) -> AsyncIterator[EnumerationPage]: ...   # yields
    async def fetch_detail(self, src, ref, fetch) -> FetchObservation: ...
    def extract(self, blob: bytes, ctx: ExtractContext) -> ParsedSourcePosting: ...  # PURE
    def evidence(self, run: RunEvidence) -> CompletenessEvidence: ...        # evidence, not verdict
```

`enumerate` yields pages so the §10.4 safeguards apply uniformly in shared policy —
restoring invariant #5 for the policy that matters most. `extract` is pure and takes bytes,
so every parser is fixture-testable. `evidence` cannot certify anything; only `classify()`
produces a verdict, and only with cross-run inputs (§4.2).

**`interface_pattern` (S0–S7) is a per-deployment fact, not a family fact.** Recon §7 opens
by warning that two deployments of the same ATS can expose materially different public
interfaces. It is therefore carried on `ProbeResult`/`CareerSource`, never on the
`BoardSpec` — the class-vs-instance error this design diagnosed as D-9.

### 4.4 Reconnaissance is a mode, not a second system

The proposal defines a full parallel contract family for reconnaissance (recon §9:
observation record, seven subrecords) whose fields duplicate the production contracts —
and whose two shapes already disagree with each other (`enumerated_unique_count` vs
`enumerated_count` + `unique_id_count`; `pages_or_cursors` vs `page_count` + `cursor_count`).

Decision: **`CrawlRun.mode = recon | production`**. Both modes run the same pipeline
through the same gate and emit the same contracts. Recon mode additionally (a) captures the
full artifact set including HAR and trace, (b) writes a `FieldAvailability` assessment
(recon §8 stage 7), and (c) does not reconcile lifecycle. The recon "observation record"
becomes a **view** joining `CareerSource` + `CrawlRun` + `CompletenessEvidence` +
`FieldAvailability` — derived, not stored twice. This deletes an entire contract family,
removes the §9.2/§9.3 disagreement by construction, and means every recon fixture is
already a production fixture.

### 4.5 Storage

| Concern | Mechanism | Rationale |
|---|---|---|
| Raw bytes | Content-addressed files, `blake2b` name, `compression.zstd` framed | I-03; stdlib only (LD-09) |
| Registry, runs, jobs, observations | **SQLite** (WAL), stdlib `sqlite3` | Single-writer scheduler, small universe (~10² brands, ~10⁵ postings); zero operational surface |
| Full-text search | **SQLite FTS5** over title + description text | I-01: mandate vocabulary is a query, not an ingestion filter |
| Export | JSONL + CSV via stdlib; JSON Schema-validated | I-16, recon §19.2 |

Rejected: DuckDB, Polars, Parquet, `zstandard`, `blake3`, SQLAlchemy — see LD-09/LD-10.
A dataset this size does not need an analytical engine, and every one of them is a
dependency whose capability the stdlib or SQLite already covers at this scale.

### 4.6 Concurrency and resource ownership (P22)

**The interpreter is GIL-enabled.** Verified: `sysconfig.get_config_var("Py_GIL_DISABLED")
== 0` and `sys._is_gil_enabled() == True` on 3.14.7. selectolax 0.4.11 ships free-threaded
wheels and lxml releases the GIL in native sections, but neither changes this build. The
design is therefore **I/O-concurrent, not CPU-parallel**: `asyncio.TaskGroup` over many
in-flight fetches, with parsing on the same loop because parse cost per document is small
relative to network latency. If parse CPU ever dominates, the escape hatch is 3.14's
`concurrent.futures.InterpreterPoolExecutor` (subinterpreters) rather than threads — noted,
not adopted.

**Playwright must use the async API.** Verified: `playwright.sync_api.sync_playwright()`
inside a running event loop raises `Error: It looks like you are using Playwright Sync API
inside the asyncio loop.` The sync API is a greenlet shim over the same async core. With an
asyncio orchestrator the design uses `playwright.async_api` exclusively; if a sync context
is ever unavoidable it is isolated in a **separate process**, never a thread.

| Resource | Owner | Lifecycle |
|---|---|---|
| `AsyncClient` (pool, cookies, TLS, routing) | one per policy boundary, held by `HostPolicyGate` | context manager for process lifetime |
| `Browser` process | one per worker | started lazily on first escalation, closed at worker exit |
| `BrowserContext` | one per source investigation | created and **closed** per source; never reused across identities |
| Parser tree | `extract()` call frame | never escapes; primitives returned (LD-03) |
| Source lease | scheduler | time-bounded, renewable, reclaimed on expiry |
| SQLite write connection | single writer | WAL; async workers queue writes through it |

**Per-host budget** is two separate mechanisms, deliberately: `anyio.CapacityLimiter` bounds
*concurrency*, and the C-8 token bucket bounds *rate*. Conflating them is why crawlers
either idle or hammer.

**Browser binaries are a deployment step.** `python -m playwright install` is required and
the package and binary versions are coupled (`pw §1`); the binaries are not present in the
current environment. This belongs in the G2 gate, not in a developer's README.

### 4.7 Query, export, and coverage audit

The pipeline does not end at `CanonicalJob`. Spec §16's steps 8–11 are the deliverable, and
they are what I-01 exists to protect.

**Mandate and geography are queries over stored postings.** The 18 high-signal terms, 13
adjacent terms, `NYC_CORE`, `NYC_METRO_EXTENDED`, and the five-value remote taxonomy
(§2.1) live in a **versioned probe vocabulary** — a data artifact, not code and not an
ingestion filter. `match()` is pure: `(CanonicalJob, ProbeVocabulary) -> MandateMatch[]`,
recording *which* term matched and *where* (title vs description), so a result is
explainable. Changing the vocabulary re-runs `match()` over stored rows; it never triggers
a recrawl (I-01, spec §15.25). FTS5 indexes title and description text; multi-word probes
(`"lean six sigma"`, `"process mining"`) use FTS5 phrase queries, and the vocabulary's
version is stored with every match set.

**Geography is reported, never silently merged.** `NYC_CORE` and `NYC_METRO_EXTENDED` are
separate counts in every output (recon §5.2). Unknown-location postings are **included and
labelled**, not dropped — dropping them would hide exactly the coverage gap the system
exists to measure. A remote posting is not assumed to permit New York residence unless the
posting says so.

**Export.** JSONL (machine) and CSV (human review), both schema-validated on the way out
(I-16). The recon §19.2 dataset names are adopted: `institutions.jsonl`,
`career_sources.jsonl`, `source_observations.jsonl`, `source_family_catalog.json`,
`fixture_manifest.jsonl`, `known_gaps.jsonl`.

**Coverage audit is a first-class output, not a report.** For every brand in the universe
the auditor emits exactly one terminal status — `healthy` · `degraded` · `manual_only` ·
`blocked` · `no_public_source_observed` · `documented_gap` — and **fails the build if any
brand has none** (§8.3). Audit-only sources (LinkedIn, Indeed, Google, Glassdoor,
eFinancialCareers, Built In NYC, Common Crawl, Wayback) feed a **miss-discovery** channel:
externally-observed postings are reconciled against the official inventory to produce
`known_gaps.jsonl`. They are never a source of record (I-02), and LinkedIn is manual-import
only (I-21, spec §15.20).

This is the closing of the loop the proposal names as its measure of quality: the system
can state what it captured, and show exactly where that statement is uncertain.

## 5. Library and platform decisions

Every decision below was verified against the **installed** version, not the reference
documentation, because two reference documents lag the lockfile: the httpx2 doc anchors
2.9.1 while 2.12.0 is installed, and the lxml doc anchors 6.1.1 while 6.1.2 is installed.
Probe command for the whole set: `.venv/bin/python -c` import/signature/behaviour checks
recorded per record. Reference sections are cited via the `web-acquisition-lib-ref` skill.

**Installed baseline (probed):** Python 3.14.7 · httpx2 2.12.0 · playwright 1.62.0 ·
lxml 6.1.2 (libxml2 2.14.6, libxslt 1.1.43) · selectolax 0.4.11 · tldextract 5.3.2 ·
extruct 0.18.0 · pydantic 2.13.4 · jsonschema 4.26.0 · idna 3.19 · SQLite 3.53.4.

### LD-01 — httpx2 adopt as the sole HTTP transport

**Decision:** adopt
**Version basis:** 2.12.0 (installed). Verified present: `Client`, `AsyncClient`, `Limits`,
`Timeout`, `MockTransport`, `HTTPTransport`, `URL`, `QueryParams`, `Headers`, `event_hooks`.
Verified `Timeout(connect=,read=,write=,pool=)` four-component construction; verified
`Client.follow_redirects` **defaults to `False`**; verified exception chain
`ConnectTimeout → TimeoutException → TransportError → RequestError → HTTPError`.
**Displaces:** all bespoke HTTP handling. Built-ins used instead of custom code —
`Limits` for per-host pooling (not a hand-rolled semaphore), `Timeout` for the four-phase
budget (not one wall-clock), `event_hooks` for metrics (not wrapper functions),
`client.stream()` + `iter_bytes()` for bounded reads (not `.content`), `response.history`
for redirect chains, `MockTransport` for the whole §8.2 transport matrix (no live server).
**Probed corrections to the 2.9.1 reference:**
- `SUPPORTED_DECODERS = {identity, gzip, deflate, zstd}` — **Brotli is absent** and the
  default `Accept-Encoding` omits `br`. CDNs fronting career sites commonly prefer Brotli,
  so **`httpx2[brotli]` is required** (LD-15). `zstd` needs no package: httpx2 2.12.0
  imports the 3.14 stdlib `compression.zstd`.
- `Client(http2=True)` raises `ImportError` — **`httpx2[http2]` (the `h2` package) is
  required** if HTTP/2 is adopted at all.
- 2.12.0 adds `Client.sse()`, `Client.query()`, `Origin`, `Proxy`, `FunctionAuth`,
  `NetRCAuth`, `create_ssl_context()`, and `HTTPTransport(socket_options=)`. No removals.
- `Response.num_bytes_downloaded` is a live counter during a stream — the built-in that
  implements the byte ceiling. Not named in the reference doc.
- `create_ssl_context()` returns a `truststore.SSLContext` (OS trust store) by default.
- Event hooks fire **once per redirect hop**, which is what makes per-hop policy
  observable.

**Three things httpx2 does not provide** (custom code, §5.1): there is no HTTP cache layer,
so conditional requests are hand-managed; `follow_redirects=True` yields `response.history`
but no veto point, while a manual `next_request` walk yields the veto but **drops
`history`** — the design uses `follow_redirects=True` plus a response hook that raises on a
disallowed hop; and decompression bounding beyond `MAX_DECODE_CHUNK_SIZE` (1 MiB/chunk) and
`max_decode_links=5` is ours.
**Risk:** the 2.9.1→2.12.0 gap is unaudited. Mitigation: every API above was probed against
2.12.0; `httpx2` is imported directly, never through `alias_httpx()`.
**Validation:** the `MockTransport` matrix in §8.2 must pass, including private-address
redirect rejection and oversized-body abort.

### LD-02 — Playwright adopt, in a bounded role

**Decision:** adopt
**Version basis:** 1.62.0. Role restricted to: network reconnaissance, endpoint promotion
testing, and production acquisition only where a source is proven irreducible to HTTP.
**Displaces:** any custom rendering, polling, or DOM-wait code. Built-ins used instead:
`BrowserContext` as the isolation unit (`pw §6`); `context.route()` to enforce
`HostPolicyGate` on in-browser egress — the mechanism that makes I-05 achievable at all;
`route_from_har()` for deterministic replay (`pw §27`); `expect(...)` web-first assertions
and locator auto-waiting instead of sleeps (`pw §13`, `§16`); tracing on failure (`pw §39`);
`page.content()` as the sole handoff to the pure core (`pw §48`).
**Risk:** browser dependence spreads silently and becomes the default path. Mitigation:
`CareerSource.acquisition_mode` records browser dependence per source; every
browser-discovered endpoint must pass the six-condition promotion rule (spec §8.4) before a
source is allowed to stay browser-backed; G7 reports browser-backed source count.
**Validation:** an integration test proves a `context.route()` denial aborts the request —
no in-browser egress reaches the network without a gate decision.

### LD-03 — selectolax (Lexbor) adopt for HTML

**Decision:** adopt
**Version basis:** 0.4.11. Verified `LexborHTMLParser`, `css_first(..., strict=True)`,
`text(deep=, separator=, strip=, skip_empty=)`, `merge_text_nodes()`, and that
`.attributes` (copy) and `.attrs` (live) are distinct.
**Displaces:** all HTML string surgery and regex extraction. Built-ins used instead:
`strict=True` for identity-critical single-match fields (`slax §6`); the four text knobs as
explicit extraction policy rather than `.strip()` chains (`slax §13`); `merge_text_nodes()`
after unwrapping (`slax §14`); fragment parsing for API-embedded HTML (`slax §19`).
**Probed corrections to the reference:**
- **`slax §29`'s "prefer bytes" is wrong for Lexbor.** Probe: ISO-8859-1 bytes carrying
  `<meta charset="iso-8859-1">` parsed to `caf\ufffd`; the same content decoded to `str`
  first parsed to `café`. **Lexbor does not honour `<meta charset>` on bytes input.** The
  design therefore **decodes before parsing**, using `Response.charset_encoding` (declared
  HTTP charset) and the built-in `Client(default_encoding=<callable>)` hook for a sniffer.
  The original bytes are still what the artifact store keeps (I-03).
- `text()` **includes `<script>` and `<style>` content** — `<div>a<span>b</span></div>
  <script>X</script>` yields `abX`. Scope the selector or `strip_tags` first.
- `separator` defaults to `''`, not `' '`.
- `script_srcs_contain()` requires a **tuple**; a list raises `TypeError`.
- Nodes hold a strong reference to their parser, so the practical risk of caching one is
  **whole-tree memory retention**, not use-after-free.
**Risk:** silent mojibake in extracted fields — the failure mode that looks like data rather
than an error. Mitigation: the decode step is a single choke point with its own golden
fixtures (Latin-1, UTF-8 with and without BOM, declared-vs-actual mismatch).
**Validation:** adapter contract suite parses every fixture and asserts no parser object
escapes `extract()`; the encoding fixture set must round-trip without replacement chars.

### LD-04 — lxml adopt for XML, sitemaps and feeds

**Decision:** adopt
**Version basis:** 6.1.2 / libxml2 2.14.6. Verified the untrusted-input parser posture
`XMLParser(resolve_entities=False, load_dtd=False, no_network=True, huge_tree=False)`
constructs and is accepted; verified `etree.iterparse` and compiled `etree.XPath`.
**Displaces:** custom XML handling and any regex over markup. Built-ins used instead:
`iterparse` with element clearing for bounded-memory large sitemaps and feeds (`lxml §11`,
`§46`); compiled `XPath` objects reused across documents rather than re-parsed per record
(`lxml §17`, `§47`); structured `error_log` retained instead of `str(exc)` (`lxml §29`).
**Risk:** untrusted XML is an XXE/expansion surface, and behaviour tracks the native
libxml2 version, not the wheel (`lxml §1`). Mitigation: one shared parser factory is the
only construction site; L-04-style rule forbids `etree.XMLParser(` elsewhere; the probed
libxml2 version is recorded in the fixture manifest.
**Validation:** a fixture with an external entity must fail closed; a 100k-entry sitemap
must parse within a fixed memory bound.

### LD-05 — extruct adopt, JSON-LD only by default

**Decision:** adopt
**Version basis:** 0.18.0. Verified `extract()` accepts `syntaxes=`, `uniform=`, `errors=`;
verified `syntaxes=["json-ld"]` with `base_url=` returns syntax-keyed output.
**Displaces:** any hand-written JSON-LD `<script>` regex, Microdata walker, or Open Graph
parser. Built-ins used: `syntaxes=` to pay only for what is consumed (`extruct §5`);
parsed-tree reuse with an existing `lxml.html.HtmlElement` (`extruct §18`) — extruct
**cannot** consume a selectolax tree, so the metadata path parses with lxml.

**Two probed findings that overturn the obvious reading of the reference — both change the
design:**

1. **`base_url` resolves far less than `extruct §4` implies.** Probe with
   `base_url="https://final.example/page"`: Microdata `itemprop` URLs **are** resolved and
   RDFa `@id`s **are** resolved, but **JSON-LD is not resolved at all**, **Open Graph is
   not resolved**, and Microdata **`itemid` is not resolved**. A document's own
   `<base href>` is also not applied. Since JSON-LD is the primary metadata path for
   `JobPosting`, **URL resolution is ours**: `httpx2.URL(final_url).join(value)`, retaining
   raw and resolved values side by side (I-12 pattern, `extruct §21`).
2. **`errors="log"` does not give per-block resilience.** Probe: one malformed
   `<script type="application/ld+json">` among three caused the **entire `json-ld` key to
   be absent** — both valid blocks were lost, because `extruct` materializes
   `list(extract(...))` per syntax and the generator dies on the first bad block.
   Consequences adopted: always read `result.get(syntax, [])`, never `result[syntax]`; and
   the JSON-LD path drives **`extruct.JsonLdExtractor().extract_items()` per `<script>`
   node**, so a malformed block is isolated to itself. Probe confirmed blocks 0 and 2
   extract while block 1 raises alone.

**Risk:** JSON-LD is attacker-controlled and proves nothing about enumeration completeness
(I-01, spec §15.16). Mitigation: metadata is a *field source*, never an enumeration source;
`uniform=` is not used (`extruct §15`); Microdata is enabled per-source only where observed
(recon ADR-04); block count and block size are bounded (`extruct §25`).
**Validation:** a fixture with a malformed JSON-LD block adjacent to two valid ones must
yield **both** valid blocks plus one recorded `structured_data_conflict` failure — a test
the naive `errors="log"` implementation demonstrably fails.

### LD-06 — tldextract adopt with a pinned offline PSL

**Decision:** adopt
**Version basis:** 5.3.2. Verified `TLDExtract(suffix_list_urls=(), fallback_to_snapshot=True)`
resolves offline and returns `top_domain_under_public_suffix` and `registry_suffix`
(probe: `careers.jpmorganchase.com` → `careers` / `jpmorganchase` / `com`).
**Displaces:** every form of label splitting. Built-ins used instead:
`top_domain_under_public_suffix` (never the deprecated `registered_domain`, `tldx §11`);
`registry_suffix` as the policy-stable boundary (`tldx §8`); `extract_urllib` on an existing
`urlsplit` result (`tldx §17`); a single reused extractor instance (`tldx §37`).
**Probed hazard:** the default `cache_dir` is
`~/.cache/python-tldextract/3.14.7.final__.venv__6d4b72__tldextract-5.3.2` — it embeds the
Python version **and a venv hash**, so it changes silently across environments. The design
pins `cache_dir` explicitly. Also verified: `.tlds` is a property (6,871 entries), not a
callable, and `ExtractResult` exposes `ipv4`/`ipv6` properties the reference doc omits.
**Risk:** PSL drift silently changes stored domain facts. Mitigation: remote fetch disabled;
`cache_dir` pinned; PSL policy and snapshot identity persisted with every domain fact (I-20).
**Validation:** golden test over vendor hosts, private-suffix hosts (`github.io`), IDN
hosts, and unknown suffixes — asserting an empty `suffix` for unknown labels is treated as
a signal, not an error (`tldx §19`).

### LD-07 — hostname canonicalization: `httpx2.URL.raw_host`; never stdlib `encodings.idna`

**Decision:** adopt (built-in); promote `idna` to a declared direct dependency for explicit
label validation only
**Version basis:** httpx2 2.12.0, idna 3.19 (already in `uv.lock`).

The `web-acquisition-lib-ref` authority matrix records hostname canonicalization as having
**no owner** in the acquisition pack, because `tldx §32` explicitly disclaims it and
`httpx2 §5` mentions IDNA only in passing. **Probing overturns that verdict.**
`httpx2/_urls.py` imports `idna` and normalizes `URL.raw_host` to a lowercased A-label:

| Input | `raw_host` | `host` |
|---|---|---|
| `https://BÜCHER.de:443/p` | `b'xn--bcher-kva.de'` | `bücher.de` (default port dropped) |
| `https://日本語.jp/` | `b'xn--wgv71a119e.jp'` | — |
| `https://xn--fa-hia.de/` | — | round-trips to `faß.de` |
| **`faß.de`** | **`b'xn--fa-hia.de'`** | the **IDNA 2008 / UTS-46 nontransitional** answer |

**The decisive negative result:** stdlib `'faß.de'.encode('idna')` returns **`b'fass.de'`** —
IDNA 2003 transitional mapping, which is *a different registrable domain*. For an allowlist
of bank domains that is a security-relevant divergence, so `encodings.idna` is banned
outright (L-05 deny-list).

**Canonical hostname is therefore `str(httpx2.URL(url).raw_host, "ascii")`**, and that
A-label — not the display form — is what feeds tldextract (verified:
`xn--bcher-kva.de` → `suffix='de'`). **Caveat:** pure-ASCII hosts bypass IDNA validation
entirely — a 70-character label was accepted although DNS caps labels at 63. Where label
validity is contractual the design calls `idna.encode(host, uts46=True)` directly, which is
why `idna` is promoted from transitive to declared.
**Risk:** comparing a U-label against an A-label, or two normalizations disagreeing.
Mitigation: one canonicalization function, one storage form (A-label), display form derived.
**Validation:** property test — for any host, `URL(url).raw_host` is idempotent under
re-parsing, agrees with `idna.encode(uts46=True)`, and **disagrees with
`encodings.idna` on `faß.de`** (an assertion that the ban is still warranted).

### LD-08 — Pydantic v2 + jsonschema, dual validation

**Decision:** adopt
**Version basis:** pydantic 2.13.4, jsonschema 4.26.0.
**Displaces:** hand-written validation and any string-parsed error handling. Built-ins used
instead: Pydantic models as the parse-don't-validate boundary (P11) with `model_json_schema()`
emitting Draft 2020-12; `TypeAdapter` for record collections; `jsonschema.Draft202012Validator`
with `check_schema()` and a **local** `referencing` registry for independent verification of
every exported artifact; structured `ValidationError` objects, never parsed text.
**Risk:** the two validators disagree, or a `$ref` resolves over the network. Mitigation:
network retrieval is disabled and references are preloaded (recon ADR-05); a round-trip test
asserts every model's instances validate under both engines.
**Validation:** `just contracts` regenerates schemas; a JSONL export produced from Pydantic
models validates independently through `Draft202012Validator` (recon ADR-05 acceptance rule).

### LD-09 — stdlib storage; reject four proposed dependencies

**Decision:** adopt stdlib / **reject** DuckDB, Polars, Parquet, `zstandard`, `blake3`
**Version basis:** probed on 3.14.7 — SQLite **3.53.4 with FTS5 available**;
`compression.zstd` present (PEP 784, new in 3.14) with verified round-trip;
`hashlib.blake2b(digest_size=32)` verified.
**Displaces:** four dependencies the proposal recommended (spec §13.7). Python 3.14's
stdlib now covers Zstandard natively, `blake2b` is a fast keyed content hash, and SQLite
FTS5 covers the broad-mandate search requirement (WP-20). The universe is ~10² brands and
~10⁵ postings — three orders of magnitude below where an analytical engine earns its
operational cost.
**Risk:** an analytical workload later outgrows SQLite. Mitigation: exports are JSONL/CSV,
so DuckDB or Polars can be added *downstream* without touching the core; this decision is
cheaply reversible and is recorded as such.
**Validation:** FTS5 query returns expected mandate matches over the anchor corpus; a
zstd-compressed blob round-trips to byte-identical content (I-03).

### LD-10 — robots: stdlib `urllib.robotparser` + a thin wrapper; reject Protego

**Decision:** adopt stdlib + wrap; **reject** Protego
**Version basis:** Python 3.14.7. A nine-case RFC 9309 conformance suite was run against
`urllib.robotparser`. Recon ADR-07 declined to assume the stdlib was adequate "without
testing"; it has now been tested.

| RFC 9309 requirement | Result |
|---|---|
| §2.2.2 longest-match precedence (`Disallow: /x/` vs `Allow: /x/page.html`) | **PASS**, order-independent |
| §2.2.2 longer `Disallow` beats shorter `Allow` | **PASS** |
| §2.2.3 `*` and `$` wildcards (`/*.pdf$`) | **PASS** |
| §2.2.1 case-insensitive UA matching | **PASS** |
| §2.2.1 merging repeated groups for one UA | **PASS** |
| §2.2.2 percent-encoding equivalence (`/f%6Fo` ≡ `/foo`) | **PASS** |
| §2.2.2 empty `Disallow:` = allow-all | **PASS** |
| §2.3.1.3/§2.3.1.4 fetch-status rules (401/403 → deny-all, 4xx → allow-all, 5xx → deny-all) | **PASS** — the source cites the RFC sections by number |
| §2.3 UTF-8 BOM tolerance | **FAIL** — a leading `\ufeff` breaks the first `User-agent` line, the file is ignored, and the result is allow-all |

All three *hard* matching rules are correct; the module's poor reputation is out of date.
Three further defects found by source inspection: UA matching is **substring containment**
(`if agent in useragent`), so a crawler named `mybot` is captured by a `User-agent: bot`
group; `_find_entry` returns the **first** applying group rather than the longest-matching
product token; and `Crawl-delay`/`Request-rate` use `str.isdecimal()`, so `Crawl-delay: 4.5`
silently becomes `None`.

**The wrapper (≈30 lines) therefore owns:** BOM strip; exact/longest UA-token group
resolution before delegating; fractional `Crawl-delay` parsing; a 500 KiB input bound
(RFC 9309 §2.5). It also owns the **fetch**, because `RobotFileParser.read()` uses
`urllib.request.urlopen` — a separate HTTP stack that would bypass the gate's timeouts, TLS
posture, proxy, UA, and SSRF policy entirely (I-05). The design fetches `robots.txt` through
`HostPolicyGate`, applies the §2.3.1 status rules, and calls `parser.parse(text.splitlines())`.
**Displaces:** Protego. It would fix the UA-token rules natively but still requires our own
fetch and status logic, and adds a dependency for four cases the wrapper already covers.
**Four further defects found by adversarial probing — and they change the port, not the
library choice:**

| Probe | Result | Direction |
|---|---|---|
| `Disallow: /a$` vs URL `/a#frag` | **allowed** — the fragment is included in the compared string; RFC 9309 §2.2.2 compares path+query only | **fail-open** |
| `Disallow: /a%2Fb` vs `/a/b` | **blocked** — `normalize()` unquotes then requotes, collapsing the reserved octet | fail-closed, false `BLOCKED` |
| `User-agent: bot` vs token `JobsQueryBot/1.0` | **blocked** — substring containment, not token match | fail-closed, false `BLOCKED` |
| `can_fetch(ua, "https://otherhost/private")` | judged against **this** robots.txt; the host is discarded | **fail-open across redirects** |

The last is a defect in **this design's port signature**, not in the library: a
`decide(url, ua)` shape cannot tell that a redirect crossed origins, and §4.1 explicitly
gates cross-host redirects. False `BLOCKED` matters more than usual here because it
corrupts the coverage register, which is the product. Protego shares the fragment and
token issues, so swapping libraries fixes none of this.

**Revised port:** `RobotsPolicy.decide(origin, path_and_query, product_token,
fetch_outcome) -> RobotsDecision`. The wrapper strips fragments, preserves reserved
percent-encodings, does exact product-token matching before delegating, and — because the
gate cannot use `read()` (it calls `urllib.request.urlopen`, bypassing every gate control) —
**owns the RFC 9309 §2.3.1 status matrix itself**: 401/403 → deny-all, other 4xx →
allow-all, 5xx → deny-all, plus timeout, oversize, and "robots.txt returned HTML" cases and
a cache TTL. That matrix is a **G2 deliverable**, not a library property.
**Risk:** an unusual `robots.txt` exercises a defect the suite missed. Mitigation: the port
is one method; swapping implementations is a single-file change.
**Validation:** the nine-case suite plus these four defects and the 4xx/5xx/timeout/oversize
matrix become regression tests;
SPK-08 extends the corpus with live `robots.txt` from the anchor batch.

### LD-11 — shared board runner: build; **manifest DSL rejected**

**Decision:** build `run_board(spec)` with per-family `BoardSpec` literals; **reject** a
declarative manifest schema and interpreter
**Version basis:** n/a — no library models ATS board enumeration.
**Status:** the *reuse* decision is firm; the *portfolio scope* is **provisional pending
SPK-06**, exactly as LD-10 is provisional pending SPK-08. Recon ADR-09 and SPK-06 exist to
decide dedicated-vs-configured adapters on evidence; this record must not pre-empt them.
ADR-09's preliminary rule (dedicated adapter when a family covers ≥2 Wave 1 institutions,
or needs nontrivial pagination/detail behaviour) is **adopted**, not contradicted.

**Why the manifest DSL was rejected.** An earlier version of this design made the eight
Priority-2 families declarative YAML. Checking each against spec §9.1–§9.8 falsified the
premise that they "differ only along eight axes": Lever advances `skip` by records
returned; Ashby needs a three-level identity coalesce; Workable filters a multi-tenant
global feed with two-valued completeness; Recruitee branches five ways with per-branch
completeness semantics; Personio fans out across languages and joins on position ID;
Teamtailor must detect first-100 truncation on custom domains; SmartRecruiters has no
stable URL. **Five of eight need escape hatches on day one** — which is the definition of a
wrong abstraction, and exactly the "badly-specified programming language" failure the
earlier record proposed to mitigate with a review rule (`L-03`, now deleted).
**Displaces:** sixteen private control loops and sixteen private §10.4 implementations —
the property that actually mattered, and one that Python literals deliver as well as YAML.
**Risk:** `BoardSpec` callables drift into unreviewable per-family behaviour. Mitigation:
every family passes the same contract suite (§8.2); the callable surface is narrow and
typed; `run_board` owns the loop, so a family cannot bypass a safeguard by construction.
**Validation:** each family's `BoardSpec` passes the full adapter contract suite offline;
SPK-06 confirms or reverses the configured-vs-dedicated split before G5.

### LD-12 — retry policy: build small; reject Tenacity

**Decision:** build, **reject** Tenacity
**Version basis:** httpx2 2.12.0. `httpx2 §25` states plainly that no universal retry
switch exists and that transport-level `retries` cover connection establishment only.
**Displaces:** a dependency that would supply only the easy half. Backoff and jitter are a
dozen lines; the parts that matter here — `Retry-After` honouring, the retryable/
non-retryable status and exception partition (spec §8.3's 7 + 8 cases), request-body
replayability, and per-host budget interaction with the lease manager — are domain policy
that Tenacity does not model.
**Risk:** hand-rolled backoff is a classic source of thundering herds. Mitigation: the
policy is a pure function `decide(attempt, outcome) -> Retry | Stop` in the core, property-
tested for monotonic backoff and bounded total budget, with the whole §8.3 matrix driven
through `MockTransport`.
**Validation:** a 429 with `Retry-After: 120` must produce a single delayed retry, not an
immediate one; a `PoolTimeout` must **not** retry (`httpx2 §13`).

### LD-13 — feedparser: defer

**Decision:** defer
**Version basis:** not in `uv.lock`. lxml (LD-04) parses RSS and Atom as namespaced XML,
which is sufficient for the feed shapes observed so far.
**Risk:** real-world feeds are famously malformed and feedparser's tolerance is its whole
value proposition. Mitigation: add it if and only if a Wave 1 source produces a feed lxml
cannot parse — recorded as a coverage-gap entry, not a silent workaround.
**Validation:** n/a until triggered.

### LD-14 — test and gate stack: adopt as dev dependencies

**Decision:** adopt
**Version basis:** **none of these are currently installed** — the project has no test
framework, linter, or task runner at all (`ls .venv/bin`). Required: `pytest`,
`pytest-asyncio`, `pytest-playwright`, `hypothesis`, `ruff`, `ast-grep`, and `just`.
**Displaces:** nothing; it establishes the proof infrastructure §8 depends on.
**Risk:** treating gates as optional because the design is greenfield. Mitigation: G1 does
not pass without `just contracts`, `just lint`, `just rules`, and `just test` existing and
green.
**Validation:** the final gate matrix in the implementation plan is a list of `just`
recipes; each must exist and pass at HEAD.


### LD-15 — httpx2 extras: `brotli` required, `http2` conditional

**Decision:** adopt `httpx2[brotli]`; defer `httpx2[http2]`
**Version basis:** probed on 2.12.0 — `SUPPORTED_DECODERS = {identity, gzip, deflate, zstd}`;
default `Accept-Encoding: gzip, deflate, zstd`, **no `br`**. `Client(http2=True)` raises
`ImportError: the 'h2' package is not installed`.
**Rationale:** career sites sit behind CDNs that commonly negotiate Brotli; without the
extra the client silently never offers it, costing bandwidth on every listing page. HTTP/2
is deferred because `httpx2 §14` is explicit that it is not automatically faster and
negotiation depends on the origin — SPK-01 measures it per source family before adoption.
**Validation:** a probe asserts `br` appears in the default `Accept-Encoding` after the
extra is installed; HTTP/2 adoption requires an SPK-01 measurement, not a preference.

### LD-16 — per-host rate limiting: build; reject `aiolimiter`

**Decision:** build (custom)
**Version basis:** verified gap. `asyncio` offers `Semaphore`/`BoundedSemaphore`
(concurrency, not rate); `anyio` 4.14.2 offers `CapacityLimiter` (also concurrency);
httpx2 has no rate parameter. **Nothing pinned provides rate limiting.**
**Displaces:** a single-purpose ~100-line dependency. The design implements a token bucket
keyed on `URL.raw_host` (~60 lines over `asyncio.Lock` + `loop.time()`) that honours both
`Crawl-delay` (LD-10) and `Retry-After` (LD-12) — neither of which a generic limiter models.
`anyio.CapacityLimiter` is still used for per-host *concurrency*, which it does model.
**Risk:** a hand-rolled limiter that leaks or deadlocks under cancellation. Mitigation: it
is a pure decision function `next_allowed_at(host, now, history) -> float` in the core with
a thin async wrapper in the shell, property-tested for monotonicity and bounded wait.
**Validation:** property test — no host ever exceeds its configured rate over any window;
`Crawl-delay: 4.5` is honoured as 4.5s, not 4s.

### LD-17 — test stack: `pytest` + `anyio` plugin; not `pytest-asyncio`

**Decision:** adopt `pytest`, `pytest-playwright`, `hypothesis`, `ruff`, `ast-grep`, `just`;
**reject** `pytest-asyncio`
**Version basis:** verified absent — `import pytest` raises `ModuleNotFoundError`. There is
no test infrastructure of any kind. But **`anyio` 4.14.2 is already in the lock and ships
`anyio.pytest_plugin`**, and httpx2 is AnyIO-based — so async tests need no new package
beyond `pytest` itself. `hypothesis` is adopted narrowly, for the URL/IDNA/PSL and
`classify()` property tests (§8.2), not for fixture-driven parser tests.
**Risk:** treating gates as optional because the project is greenfield. Mitigation: G1 does
not pass without `just contracts`, `just lint`, `just rules`, `just test` existing and green.

### LD-18 — analytics: SQLite now, DuckDB only on measured need

**Decision:** retain SQLite; **defer** DuckDB; **reject** Polars
**Version basis:** SQLite 3.53.4 probed with FTS5, JSON1 (`json_extract`), STRICT tables,
generated columns, window functions, `RETURNING`, UPSERT, `blobopen()` incremental BLOB I/O,
and `threadsafety=3`. That covers per-run and moderate cross-run analytics for a universe of
~10² brands and ~10⁵ postings.
**Risk:** a later cross-run analytical workload outgrows it. Mitigation: exports are
JSONL/CSV, so DuckDB attaches *downstream* without touching the core — a reversible,
data-triggered decision. Polars is rejected outright as redundant with DuckDB.
**Note:** `sqlite3.version` was **removed** in Python 3.14; use `sqlite3.sqlite_version`.

### LD-19 — type checker: pyrefly (amendment, 2026-08-20)

**Decision:** adopt
**Version basis:** `pyrefly 1.2.0`, verified installed (`.venv/bin/pyrefly --version`).
**Recorded as an amendment** because the dependency set changed after this design was
accepted: `pyrefly`, `pytest`, and `ruff` were added to `pyproject.toml` at `20:26:42` on
2026-08-20 as **runtime** declarations, taking the direct set from eight to eleven and the
lockfile from 44 to 52 packages. LD-14 and LD-17 named no type checker at all, so
`just typecheck` — a gate the implementation plan cites in twelve packets — had nothing
behind it.
**Displaces:** nothing; it records a tool that was already pinned.
**Risk:** dev tooling shipping as a runtime dependency of the published package.
Mitigation: the implementation plan's `WP01` re-tiers all three into `[dependency-groups]
dev` as an explicit required change with a parsed (not grepped) acceptance check.
**Validation:** `just typecheck` runs `pyrefly` over `src/` and exits 0.

### 5.1 Custom code this design owns

Nothing pinned covers these, and each is small, isolated, and testable. They are listed
here so the implementation plan cannot mistake them for library behaviour.

| # | Custom component | Why | Home |
|---|---|---|---|
| C-1 | Conditional-request store (ETag / `Last-Modified` → `If-None-Match` / `If-Modified-Since`, 304 handling) | httpx2 has no cache layer; `hishel` targets `httpx`, not `httpx2` | shell, ~50 lines over `sqlite3` |
| C-2 | Per-hop redirect provenance + SSRF revalidation | `follow_redirects=True` gives `history` but no veto; a manual walk gives the veto but drops `history` | `HostPolicyGate` + response hook |
| C-3 | Decompression-expansion bound | built-ins stop at 1 MiB/chunk and 5 chained encodings | `HttpFetcher` |
| C-4 | Charset resolution before HTML parse | Lexbor ignores `<meta charset>` on bytes (LD-03) | shell, wired to `Client(default_encoding=)` |
| C-5 | Per-block JSON-LD extraction | `errors="log"` discards a whole syntax on one bad block (LD-05) | core `extract()` |
| C-6 | URL resolution for JSON-LD / OG / microdata `itemid` | `base_url` does not resolve them (LD-05) | core `normalize()` |
| C-7 | Robots wrapper (BOM, UA-token group, fractional `Crawl-delay`, 500 KiB bound, gated fetch) | four stdlib defects + `read()` bypasses the gate (LD-10) | `RobotsPolicy` port |
| C-8 | Per-host token bucket | nothing pinned provides *rate* limiting (LD-16) | core decision fn + shell wrapper |
| C-9 | SSRF predicate | see §9.2 — the correct test is `not is_global` | `HostPolicyGate` |
| C-10 | Shared board runner `run_board(spec)` + `run_faceted_endpoint(spec)` | no library models ATS board enumeration (LD-11). *(This row said "manifest evaluator" before §6.4 falsified the DSL; the component survived, the configuration language did not.)* | adapters |
| C-11 | Retry policy | httpx2 has no universal switch (LD-12) | core decision fn |

Eleven small components, each with a named oracle in §8 — against sixteen bespoke adapters,
a hand-built crawler framework, and four storage dependencies in the proposal.

## 6. Alternatives and clean-sheet challenge

Three materially different designs were developed. The clean-sheet question —
*would this still be the preferred design if the proposal did not exist?* — is answered
in §6.4.

### 6.1 Alternative A — adapter-per-family (the proposal as written)

Sixteen bespoke Python adapters, each owning its own pagination handling and its own
`assess_completeness`; a separate reconnaissance contract family; `RawArtifact` as a
single table.

**Where it wins.** Nothing is indirect: reading `greenhouse.py` tells you exactly what
happens. No manifest schema to design, no interpreter to debug. Irregular behaviour never
has to be forced into a declarative shape.

**Why it is rejected.** It cannot satisfy its own invariants. Shared subsidiary boards are
unrepresentable (D-1); the artifact table contradicts its dedup rule (D-2); the eleven
pagination safeguards become sixteen private implementations, violating invariant #5 for
the highest-risk policy in the system (D-3); adapters self-certify completeness (D-4).
Change surface scales linearly with ATS families forever, and the doctrine litmus test
(§12) fails on the first new employer that brings a new uniform board.

### 6.2 Alternative B — shared board runner with per-family specs (**selected**)

As specified in §4.3. One `run_board(spec)` owns the control loop and the §10.4 safeguards;
each family supplies a `BoardSpec` of values and callables; Priority-1 enterprise platforms
get `run_faceted_endpoint(spec)`; recon is a run mode; completeness is derived centrally
from cross-run inputs; one gate governs both transports.

**Where it wins.** One implementation of the safeguards (D-3, D-4). One file per family.
The pure core makes the fixture-first mandate (spec §12) achievable. Two contract families
collapse into one (§4.4). And irregular behaviour — five of eight Priority-2 families need
it on day one — is a typed callable rather than a schema extension.

**What it costs.** `BoardSpec` callables can drift into unreviewable per-family behaviour.
Mitigations: `run_board` owns the loop, so a family cannot bypass a safeguard by
construction; every family passes the same contract suite; the callable surface is narrow
and typed.

### 6.3 Alternative C — adopt a crawl framework (Scrapy) as the substrate

Genuinely material, and the proposal's ADR-01 considered it. Scrapy supplies, off the
shelf: a scheduler, per-host concurrency and AutoThrottle, a robots middleware, a retry
middleware, a duplicate-request filter, an HTTP cache, feed exports, and a mature spider
contract-test facility. That is a large share of what WP-04, WP-05 and WP-14 hand-build.

**Why it is rejected.**

1. **Completeness accounting is the product, and Scrapy has no concept of it.** Scrapy
   optimizes throughput over a frontier; JobsQuery must prove that a *bounded* set was
   exhaustively enumerated, per source, with evidence. The eleven pagination safeguards,
   partition coverage, advertised-count reconciliation, and closure eligibility would all
   be written anyway — inside a framework whose scheduler actively hides request ordering.
2. **The safety story inverts.** I-05 requires one gate over both HTTP and browser egress.
   Scrapy's policy lives in a middleware chain that Playwright traffic does not traverse,
   so the design would carry two policy paths — precisely the defect §4.1 exists to remove.
3. **Dependency direction.** Spiders are framework subclasses; domain logic inside them
   depends outward (P5, P6 violation). The pure-core testability that makes §12 feasible
   would be lost.
4. **It is not in the pinned set**, and it brings Twisted-era reactor semantics into an
   otherwise plain-asyncio, Python-3.14 process alongside Playwright's own event loop.

**What is adopted from it anyway:** AutoThrottle's idea (adaptive per-host delay driven by
observed latency) informs the scheduler's per-host lease policy, and Scrapy's spider
contract tests informed the shared adapter contract suite (§8).

### 6.4 Alternative D — declarative manifest DSL (**developed, then falsified**)

This was the selected design in an earlier draft of this dossier, and it is recorded here
rather than deleted, because the reason it failed is more useful than the fact that it did.

**The proposal:** the eight Priority-2 board families become versioned YAML manifests —
host signatures, endpoint template, pagination protocol, collection path, identity path,
field map, advertised-count path — executed by one generic interpreter, governed by a
schema-change review rule.

**How it was falsified.** Checking each family against spec §9.1–§9.8 rather than against
the summary in §4.3: Lever advances `skip` by *records returned*, not by `limit`, and needs
a region variable the schema had no slot for; Ashby resolves identity through a three-level
preference ladder; Workable enumerates by filtering a multi-tenant global XML feed and has
two-valued completeness; Recruitee branches five ways with per-branch completeness
semantics; Personio fans out across languages and joins on position ID; Teamtailor must
detect silent first-100 truncation on **custom** career domains that no static host list
can enumerate; SmartRecruiters' documented API requires a key, so it has no stable URL to
put in the template. **Five of eight need an escape hatch immediately.**

**Two compounding errors, both worth naming.** First, the single worked example was wrong
in four ways — one `tenant_from` rule could not serve the three hosts it declared; the field
map carried only the *normalized* location, violating I-12; `interface_pattern` sat at
family level, which is this design's own D-9 error; and an `advertised_total` serialized in
the same response as the array it counts is **not independent evidence**, so `EXACT` — the
system's strongest claim — would have been granted on self-consistent data. Second, spec §5
ranks all eight families as **Priority 2, "less likely to dominate large-bank coverage."**
The abstraction was aimed at the half of the portfolio that does not move Wave 1 coverage,
while the half that does was left as ungeneralized bespoke code.

**What survives.** The reuse property, which never required YAML. **What is deleted:** the
schema, the interpreter, the `SourceFamilyManifest` contract, and the `L-03` governance
rule invented to contain it. **What it cost to find:** one adversarial review — which is
the argument for running one before, not after, a design is accepted.

**Consequence for `EXACT`.** `CompletenessEvidence` gains
`total_is_independent: bool`, and `EXACT` requires it. A count returned alongside the
collection it describes yields `TERMINAL`, not `EXACT`.

### 6.5 Clean-sheet challenge

*Would this design be preferred if the proposal did not exist?* **Yes**, with two
qualifications. The pipeline shape (universe → brand → source discovery → detection →
enumeration → artifacts → parse → normalize → reconcile → query) is not an artifact of the
proposal; it is the natural decomposition of the problem, and any clean-sheet attempt
reproduces it. The proposal's real contribution is its **policy layer** — the 25
invariants, the completeness dimensions, the escalation ladder and its promotion rule, the
failure taxonomy, the fixture protocol — which a clean-sheet design would have had to
discover the hard way. That layer is adopted nearly intact and is the reason this is a
reshaping rather than a rewrite.

**Qualification 1:** nothing in the *structure* of the proposal survives unchallenged.
Every §7 contract is reshaped or replaced, the adapter protocol is redesigned, and one
entire contract family is deleted. No structure is retained because it exists.

**Qualification 2 — the policy layer was exempted from challenge, and it should not have
been.** By declaring §15/§10/§8.4 "excellent", an earlier draft skipped the clean-sheet
question that matters most for a greenfield repo: not *is this policy right* but *is it the
right size*. Adopted with no reduction: 23 invariants · 10 completeness dimensions · 8
classes · 6 A–X grades · 8 interface patterns · 12 resolution states · 3 confidence states ·
21 failure categories · 11 pagination safeguards · 12 anomaly rules · a 25-field output —
roughly **150 enumerated vocabulary items governing a system that has not yet enumerated one
Greenhouse board**, with G6 freezing the schema.

Decision: **a minimum-viable-vocabulary pass runs before G1.** Each vocabulary is reduced to
the members the anchor batch actually exercises, with the remainder recorded as *reserved*
and admitted by schema version when a real source demands them. The invariants are **not**
reduced — they are policy, cheap to hold, and each one has a named oracle. The enumerations
are, because an unexercised enum member is an untested branch that reads like a
requirement.

### 6.6 Legacy disposition

Inventory generated by:
`rg --files src/ docs/ --glob '!docs/library_ref/**'` and
`rg -n '^(def|class|async def) ' src/ -g '*.py'`.

| Surface | Disposition | Justification |
|---|---|---|
| `src/jobsquery/__init__.py::main` (2 LOC) | **replace** | Toolchain seed. Becomes the CLI entry point. |
| `docs/…Acquisition Design Specification.md` (§1–§13, §15, §16) | **preserve** as policy source | Invariants, completeness semantics, escalation ladder, fixture protocol, source inventory. Cited, not restated. |
| `docs/…Acquisition Design Specification.md` §7 contracts, §7.10 protocol | **replace** | D-1…D-10. Superseded by §4.2/§4.3. |
| `docs/…Acquisition Design Specification.md` §14 (WP-01…WP-22) | **reshape** | Good dependency ordering; repacketized in the implementation plan against the new contracts (notably: a completeness-evaluator packet, which §14 lacks). |
| `docs/…Reconnaissance and Architecture Decision Plan.md` §4, §5, §7, §8, §10, §12, §13, §16 | **preserve** | Fixed inputs, classification model, workflow, fixtures, spikes, priorities, failure taxonomy. |
| `docs/…Reconnaissance…Plan.md` §9 observation contract | **reshape** *(was: delete)* | The §9.2/§9.3 disagreement is a naming inconsistency between an example and a field list, not a reason to drop a deliverable. §9.3's field names win. The *contract family* collapses into the run-mode view (§4.4), but the **deliverable survives**: recon §6.3's source-observation dataset and §19.2's `source_observations.jsonl` are handoff obligations that §12/§13/§16 — all marked preserve — feed. `search_checks` and `resolution_confidence` are carried on the view. |
| `docs/…Reconnaissance…Plan.md` §11 ADR-01…ADR-12 | **reshape** | Preliminary decisions; superseded by the LD records in §5, which carry verified version evidence they lacked. |

There is **no legacy code to encapsulate, migrate, or decommission**. The 2-LOC seed is the
entire executable inheritance.

## 7. Transition and cutover

There is no migration. The system is greenfield (§6.5), so "transition" means **build
order under one constraint**: contracts and the gate must exist before any network traffic,
because reconnaissance produces the fixture corpus that everything downstream is tested
against, and a fixture captured under a schema that later changes is a fixture lost.

Recon §18's seven gates are adopted, re-cut against the target contracts:

| Gate | Exit condition (all executable) |
|---|---|
| **G1 — Contracts frozen** | Every contract in §4.2 **and every contract named in §4.2's "shapes belong to the plan" list** exists as a Pydantic model — the exit condition enumerates them individually. `just contracts` regenerates JSON Schema 2020-12 (with `$schema` injected — Pydantic omits it); every model round-trips through `Draft202012Validator`; `Completeness`/`RunState`/`SourceState` are disjoint (L-02); dependency floors are replaced by exact pins. **No network code merges before G1.** |
| **G2 — Gate and store** | `HostPolicyGate` + `ContentStore` + `HttpFetcher` pass the full `MockTransport` matrix (§8.2) including private-address redirect rejection; `BrowserFetcher` proves in-browser routing through the same gate. |
| **G3 — Core is pure** | `extract`/`guard`/`classify`/`normalize`/`resolve`/`reconcile`/`match` implemented and property-tested; L-01 proves core imports no I/O module. |
| **G4 — Anchor batch recon** | The 12 anchor institutions observed in `mode=recon`; fixture corpus + manifest captured; every institution lands in exactly one resolution state. |
| **G5 — Spikes resolved** | Recon §12's twelve spikes closed; each LD record in §5 either confirmed or amended with the evidence. |
| **G6 — Wave 1 complete + repeat** | Remaining 18 Wave 1 brands observed twice; stability dimension populated; schema **frozen** — new facts require a schema version, not a field edit. |
| **G7 — Portfolio decision** | Adapter portfolio meets recon §13.3: ≥80% of Wave 1 brands on automated healthy sources, ≥90% of observed Wave 1 job volume, every anchor institution resolved or explicitly classified manual/blocked. |

**Ordering constraint that the proposal's §14 gets wrong.** Spec §14 sequences WP-13
(identity/lifecycle) and WP-14 (anomaly/no-silent-zero) *after* the first six adapters,
yet ships those adapters with acceptance criteria that reference closure behaviour. And no
packet owns the completeness evaluator at all — it is left implicit in the adapter SDK. In
this design the evaluator is a **G3 deliverable**, before any adapter exists, because
every adapter's contract test asserts against its verdicts.

**Intermediate states.** Exactly one is permitted: between G4 and G7 a source may be
`resolution_state = investigating` with a hand-written manifest that has not yet passed
the contract suite. Exit invariant: at G7 every source is in a terminal resolution state or
recorded in the coverage-gap register. No source is silently carried.

## 8. Proof strategy

The design is not proved by the existence of modules. Each claim below names the oracle
that distinguishes a correct implementation from a plausible-looking one.

### 8.1 Structural governance (executable, in `rules/` + `just`)

| ID | Rule | Check |
|---|---|---|
| **L-01** | Core imports are **allow-listed**, not deny-listed. `jobsquery.core.*` may import only the stdlib data modules, `pydantic`, and sibling core modules. A deny-list was the original design and review falsified it: `extract()` is in core and uses `extruct`, whose resolved tree pulls **`rdflib`, `pyrdfa3`, `requests`, `urllib3`, `requests-file`** — `rdflib.Graph.parse(location=…)` is live egress that no enumerated ban would have named. `urllib.request` was likewise unnamed. | import-gate rule over the allow-list + **L-01b** runtime assertion |
| **L-01b** | **Socket-level egress assertion.** A test monkeypatches `socket.socket` and `socket.getaddrinfo`, runs the entire core module tree over the full fixture corpus, and asserts **zero** calls. This catches transitive egress by behaviour rather than by name — the only form of this claim that is actually provable. | `pytest` test, part of `just rules` |
| **L-02** | No bare collision token. `PARTIAL`/`FAILED`/`BLOCKED`/`INDETERMINATE` appear only as members of a namespaced enum. | `ast-grep` rule with fixtures in `rule-tests/` |
| **L-04** | No egress outside the gate: only `jobsquery.shell.gate` may construct `httpx2.Client`/`AsyncClient`, call `browser.new_context`, or call `urllib.request.*`. A name rule proves absence of *spellings*, not of egress — it is retained as a fast tripwire, with **L-01b** as the actual proof. | `ast-grep` rule, zero-hit (tripwire) |
| **L-07** | `encodings.idna` is never imported anywhere. It maps `faß.de` → `fass.de` (IDNA 2003), a **different registrable domain** (LD-07). | `rg` deny-list, zero-hit |
| **L-05** | No credential, storage-state persistence, or submission path exists (I-21). | `rg` deny-list over the tree: `storage_state(`, `set_extra_http_headers.*[Aa]uthorization`, apply-submit verbs |
| **L-06** | Contracts are acyclic and dependency direction is inward. | **pytest** import-graph test — `ast-grep` is a per-file matcher and cannot compute a cross-file cycle |
| **L-08** | `etree.XMLParser(` is constructed only in the shared safe-parser factory. | `ast-grep` rule, zero-hit outside the factory |
| **L-09** | `LexborHTMLParser(` is constructed only from a `str`-typed value — never bytes (LD-03: Lexbor ignores `<meta charset>` on bytes, producing silent mojibake). | `ast-grep` rule |
| **L-10** | No `raise` in the shell escapes without a `FailureCategory` (I-18). | `ast-grep` rule |
| **L-11** | No adapter defines `assess_completeness` or constructs a `CompletenessVerdict` (D-4). | `ast-grep` rule, zero-hit |

### 8.2 Behavioural oracles

**Pure core — property and golden tests, no network.**

- `classify()` — property test over generated `CompletenessEvidence`: `EXACT` is
  unreachable without an advertised total that equals unique-ID count with pagination
  exhausted and no truncation signal (spec §10.1); `closure_allowed` implies all five
  §10.8 conjuncts; `content_refresh_complete` never gates `reconciliation_allowed` (§10.3).
  Golden table: every one of the 8 classes reached by at least one evidence vector, and
  each A/B/C/D/M/X projection asserted.
- `signal()` — one fixture per §10.4 safeguard (11); the stream-driving `PageGuard` and the
  end-of-run first-page rerun are tested in the shell against `MockTransport`.
  A repeated cursor, a non-advancing offset, and an exceeded advertised total must each
  produce a distinct signal, not a shared "anomaly".
- `reconcile()` — state-machine property test (spec §8.5): **no input sequence containing
  an incomplete run ever produces a `CLOSED` transition** (I-09). This is the single most
  important test in the system and is written before any adapter.
- `normalize()` — golden fixtures asserting `locations[]` is never collapsed (I-12) and
  raw values survive alongside normalized ones.
- `extract()` — per-family fixture corpus (recon §10.2): first/middle/terminal page, empty
  result, multi-location job, malformed record, closed job, JSON-LD array/`@graph`/
  multi-block/malformed-adjacent-valid.

**Shell — deterministic transport tests.** `httpx2.MockTransport` matrix (spec §12.4):
redirect chains, 304, 408, 429 + `Retry-After`, 500/503, read timeout, connect failure,
oversized body, unexpected content type, compressed bodies, duplicate query params,
conditional headers, stream closure. **SSRF is deliberately *not* on this list:**
`MockTransport` replaces the transport and never resolves a name, so it can only prove that
the predicate rejects a literal private address — not that a public hostname resolving to a
private one is caught, which is the actual risk. That is proved separately against the
custom resolving transport with a stub resolver. Browser paths use
Playwright HAR replay (`route_from_har`) with an explicit miss policy, service workers
blocked, fixed viewport/locale/timezone.

**Adapter contract suite (I-15).** Every adapter — manifest-driven or coded — passes the
same suite: detection, empty board, single item, exact-full-page, short terminal page,
multi-page, repeated-page rejection, duplicate source IDs, identity stability across two
runs, malformed record tolerance, and evidence emission that classifies to the expected
verdict. A manifest is not "done" until its fixtures make the suite green offline.

### 8.2b Recovery, concurrency, and lease oracles

Recon **SPK-12** (scheduler recovery: idempotence and resumability) and **ADR-10**'s
validation experiment are preserved in §6.5 but had **no oracle** in an earlier draft, so
I-19 was asserted with nothing that could falsify it. Named now:

- **Kill-9 mid-enumeration, then resume.** A crash leaves `SourceObservation` rows and
  blobs written with **no `CompletenessEvidence`** — so `classify()` never ran. The
  defined behaviour: `reconcile()` **refuses a null verdict** and the run resumes or is
  discarded; it never closes a job (I-09). Transaction boundary is **per page**.
- **Lease expiry interleave.** `CrawlRun.lease` carries a monotonically increasing
  **`lease_epoch` fencing token**, and every write is conditional on it. Test: holder A
  stalls past `expires_at`, holder B acquires and writes, A resumes — A's writes must be
  **rejected**, not merged. Without the token this is undetectable, which is why it is a
  contract field and not an operational note.
- **SQLite writer serialization.** "Single-writer" is a mechanism, not an assertion: one
  dedicated writer task consuming a queue, WAL mode, explicit `busy_timeout`, and
  `PRAGMA` setup recorded in one place. Test: N sources enumerate concurrently against one
  writer with zero `SQLITE_BUSY` escapes.
- **ATS vendor change between runs.** Trips four §10.9 rules at once
  (`branding_changed`, `source_identifier_missing`, `json_shape_changed`,
  `all_source_ids_changed`); every job appears closed and an identical set appears new.
  Test: the `SourceAnomaly` guard must block closure and mark the source `DEGRADED` —
  this is the design's most damaging realistic failure and its guard is now testable.
- **A `BoardSpec` that runs clean and returns nothing.** Fixtures cannot catch a
  `collection` accessor that resolves empty against a live board, because the same broken
  spec produces the evidence I-08 would check. Oracle: spec §12.7's **live smoke test**
  against a known-non-empty board is an acceptance criterion for every family, run outside
  the offline suite.

### 8.3 Negative and zero-state evidence

- **No silent zero (I-08):** a test asserts that a run producing zero jobs with evidence
  insufficient for a valid zero (§10.5) raises an anomaly and does **not** write a
  zero-job fact.
- **No second network path (I-05):** L-04 zero-hit, plus an integration test that revokes
  the gate and asserts every fetcher fails closed.
- **No employer silently omitted (recon §20):** the coverage auditor asserts that
  `|Wave 1 brands| == |brands in exactly one terminal resolution state|`, failing the
  build on any brand with no recorded status.

### 8.4 Reproducibility

Every derived fact records what produced it: `parser_version`, `classifier_version`,
manifest version, PSL policy + snapshot identity (I-20), and library versions in the
fixture manifest. Re-running any stage over the same blobs reproduces byte-identical
outputs — asserted by a golden-hash test over the anchor-batch corpus.

## 9. Failure semantics, trust boundary, and observability

### 9.1 Failure taxonomy (P23, I-18)

Recon §16's 21 categories are adopted verbatim as the `FailureCategory` enum and are the
**only** way a failure may be recorded. There is no generic "run failed". Each category
carries a fixed `retryable` disposition and a fixed owner stage, so that a failure record
is actionable without reading prose:

`institution_identity_failure` · `employment_brand_mapping_failure` · `official_domain_failure` ·
`career_source_discovery_failure` · `source_scope_ambiguity` · `robots_prohibition` ·
`access_blocked` · `authentication_required` · `http_fetch_failure` ·
`browser_navigation_failure` · `network_endpoint_not_replayable` · `listing_parse_failure` ·
`detail_parse_failure` · `pagination_failure` · `advertised_count_mismatch` ·
`source_id_collision` · `location_parse_failure` · `structured_data_conflict` ·
`fixture_capture_failure` · `schema_validation_failure` · `change_detection_ambiguity`

The retry partition is spec §8.3's, corrected for one layer leak: **"parser contract
failure" is removed from the HTTP-request state machine**, where the proposal placed it —
parsing happens downstream of the request and cannot be a transport outcome.

### 9.2 Trust boundary (P8, I-05, I-21)

Every URL, every byte, and every piece of embedded metadata is untrusted input.

| Threat | Control | Owner |
|---|---|---|
| SSRF, private/link-local/metadata/CGNAT targets | scheme allowlist · credential rejection · A-label canonicalization (LD-07) · `ipaddress` classification of the **resolved address** using **`not addr.is_global`** · re-applied to **every** redirect hop. **Mechanism:** a custom `AsyncBaseTransport` that resolves, validates, then connects — httpx2 exposes no pre-connect hook, so this cannot be a client option and cannot be proved by `MockTransport` (which replaces the transport and never resolves). | custom transport in `HostPolicyGate` |
| Browser egress bypassing policy | `context.route("**/*")` → same gate decision → abort on deny, **with `service_workers="block"` on the production context, not only in tests** — service workers bypass route interception (`pw §26`, `§7`). Chromium's own process traffic (component updates, safe-browsing, OCSP) never reaches a handler and is out of scope for this control; it is bounded at the container's egress policy instead. | `BrowserFetcher` |
| XXE, entity expansion, external resource loading | one shared lxml parser factory with entities, DTD, network, and huge-tree all disabled | parser factory (LD-04) |
| Decompression bombs, oversized bodies | streamed reads with a byte ceiling; decompression expansion bounded | `HttpFetcher` |
| Hostile embedded metadata | metadata is a field source only, never an enumeration source; values are never executed, never fetched without a fresh gate decision | core `extract()` |
| Credential leakage | safe-header allowlist on persist; no storage-state persistence; L-05 deny-list in CI | `ContentStore`, L-05 |

**The SSRF predicate is `not addr.is_global` — not `addr.is_private`.** This was verified
and it is the single most consequential correction in the security design: `100.64.0.1`
(RFC 6598 carrier-grade NAT) reports `is_private=False` **and** `is_global=False`, so a
private-address check lets it through. `169.254.169.254` (the cloud metadata endpoint)
reports `is_link_local=True, is_private=True, is_global=False`, and `192.0.0.170` reports
`is_private=True`. Only `is_global` covers the full non-routable space in one test.
`tldx §34` and `httpx2 §31` both state plainly that neither library is an SSRF boundary;
this predicate is ours (C-9).

Three things the system deliberately **cannot** do (I-21): authenticate, submit an
application, or circumvent an access control. There is no code path to add them without
crossing a governance rule.

### 9.3 Observability (P28)

Structured records aligned to artifact and stage boundaries, distinct from provenance:
per-run counters (requests, bytes, pages, retries, 429s, timeouts, parse failures,
duration), per-source health history, and the six recon §15 metric families as derived
views over stored facts — not separately maintained counters. **Provenance** answers "what
produced this job record" (`SourceObservation` → `FetchObservation` → `ContentBlob`);
**observability** answers "what did this run do". They are not merged.

## 10. Doctrine conformance

Assessed against `docs/library_ref/semantic_design_principles_holistic.md`. The doctrine
targets a semantic-compiled engineering platform; principles concerning compilation passes,
multi-IR lowering, solver runtimes, and workbench projection (**P9, P14, P15, P18, P20,
P26**) are **N/A** to an acquisition system and are not force-fitted.

| Principle | Status | Where |
|---|---|---|
| P2 Separation of concerns · P5 Dependency direction · P6 Ports and adapters | **Advances** | §4.1 three rings; L-01/L-06 |
| P8 Trust boundaries, least privilege | **Advances** | §9.2 single gate; L-04, L-05 |
| P10 Declarative single-sourcing | **Advances** | §4.3 manifests; one classification rule (§4.2) |
| P11 Parse, don't validate · P12 Illegal states unrepresentable | **Advances** | Pydantic boundary; `CompletenessVerdict` cannot be authored |
| P13 Stable semantic identity | **Advances** | I-11; identity never from URL or list position |
| P16 Design by contract · P29 Versioned public contracts | **Advances** | §4.2; dual validation (LD-08); G1 freeze |
| P17 Functional core, imperative shell | **Advances** | §4.1 — the load-bearing structural decision |
| P19 Durable domain vs temporal control truth | **Advances** | `ContentBlob`/`SourceObservation` vs `CrawlRun`/lease |
| P22 Ownership and lifecycle | **Advances** | client, browser context, parser tree, lease each have one owner |
| P23 Explicit failure semantics | **Advances** | §9.1, 21 categories, no generic failure |
| P24 Idempotency | **Advances** | I-19; content-addressed writes are naturally idempotent |
| P25 Reproducibility and hermeticity | **Advances** | §8.4; PSL/parser/classifier versions recorded |
| P27 Provenance · P28 Observability | **Advances** | §9.3, kept distinct |
| P30 Testability | **Advances** | §8; the pure core is what makes fixture-first achievable |
| P31 Additive extensibility + executable governance | **Advances**, scoped honestly | §8's L-rules are executable. On the **§12 litmus**: for this declared universe, new scope arrives largely as **Priority-1 enterprise sources, which are code**. The litmus is met for employers (data) and for recurring board families (one small module), not universally. Doctrine §8's YAGNI clause is what killed the manifest DSL (§6.4) — an abstraction layer needs a near-term second use case, and spec §5 says the near-term case is elsewhere. |
| P21 Command–query separation | **Risk — mitigated** | The scheduler both reads due-sources and writes leases. Confined to one component with an explicit lease contract; queries elsewhere are side-effect-free. |
| P1 Information hiding · P3 SRP · P4 Cohesion · P7 Acyclic | **Maintains** | Conventional module discipline; L-06 enforces acyclicity |

**Anti-principles checked directly.** No family-specific execution branches in core (the
manifest engine is generic; Tier-2 adapters implement the same port). No truth distributed
across UI or vendor objects. No duplicate rule definitions — the completeness rule exists
once (D-4), and the two completeness vocabularies are projections, not stores (D-5). No
workflow controller holding domain semantics — state machines carry *control* truth only;
domain rules live in the pure core. No identity from row order (I-11). No side-write path
outside the gate (L-01b). No diffuse privilege (§9.2).

**One anti-principle was committed, and is now corrected.** *"Hidden duplicate rule
definitions"* — §3 keeps two completeness vocabularies (A–X grades and the eight classes) as
"pure projections", but an earlier draft described the mapping only in prose here. It is now
a **single-sourced projection table** owned by the classifier module and cited by both the
recon view and the §7 gates. P10 requires the rule to exist once; prose in a design document
is not once.

**§12 litmus test, stated honestly.** New scope arrives as: a registry row (new employer),
a fixture set (new observed shape), or a `BoardSpec` module behind an unchanged port (new
board family). None requires a core or shell edit. **But for this declared universe, the
scope that moves Wave 1 coverage is Priority-1 enterprise sources, and those are code** —
`run_faceted_endpoint(spec)` generalizes their shared shape, but each still carries real
per-platform behaviour. The litmus is met for employers and board families; it is *partially*
met for the enterprise platforms, and the design says so rather than claiming otherwise.
What keeps it true is the port boundary and the shared contract suite, not a governance rule
over a schema — the schema, and the `L-03` rule invented to contain it, are both gone (§6.4).

## 11. Assumptions to validate, and what would reopen this design

Labelled per `evidence-policy.md` §1. Each has an owner gate and a stated consequence.

| # | Assumption | Validate at | If false |
|---|---|---|---|
| **A-1** | Eight uniform ATS families reduce to the eight declarative axes without escape hatches. | G4/G5 (SPK-06) | If ≥3 need escapes, revert those families to Tier-2 adapters. The manifest engine survives for the rest; only its scope shrinks. Recorded replan trigger. |
| **A-2** | Greenhouse/Lever/Ashby field paths in §4.3 are as illustrated. | G4 — anchor-batch fixtures | Paths are corrected in the manifest; no design change. |
| **A-3** | `context.route("**/*")` intercepts every in-browser request, including navigation and preflight, with `service_workers='block'`. | G2 | If any egress class escapes routing, I-05 needs a second mechanism (proxy-level egress control), which is a material design change. |
| **A-4** | stdlib `urllib.robotparser` + the C-7 wrapper is RFC-9309-adequate for this source set. | G5 (SPK-08) | Adopt Protego behind the existing `RobotsPolicy` port — one file. |
| **A-5** | SQLite + FTS5 is adequate for the mandate vocabulary and cross-run analytics at this volume. | G6 | Attach DuckDB downstream of the JSONL export. Core untouched. |
| **A-6** | Career-site `robots.txt` in the target set permits the enumeration paths at all. | G4 | A `robots_prohibition` source becomes `manual_only`; this is a coverage outcome, not a failure — and it is precisely what the audit is for. |
| **A-7** | The GIL-enabled interpreter leaves parse CPU below network latency. | G6 | `InterpreterPoolExecutor` for parsing; no contract change (the core is already pure). |

**What would reopen the design outright** (not merely amend it): evidence that per-source
completeness cannot be established for a majority of Wave 1 brands by any method
(invalidates the product premise, not the architecture); a legal or access-policy change
making official-source enumeration untenable; or a finding that the pure-core split
materially prevents an adapter from functioning — the one structural bet with no cheap
exit.

## 12. Review record

An independent challenger with fresh context reviewed this dossier against the source
proposals and the doctrine, and against live probes of the pinned libraries. It returned
four blocking and seven material findings. **Nine changed the design; two were rejected in
part.** The changes are recorded inline above rather than as a changelog, but the material
ones are indexed here because a reader deserves to know which parts of this document
survived challenge and which were rewritten under it.

| Finding | Verdict | Where the design changed |
|---|---|---|
| Manifest DSL fits 5 of 8 families, and targets the portfolio half spec §5 calls "less likely to dominate large-bank coverage" | **Upheld** — verified verbatim in spec §5 and §9.1–§9.8 | §1.4, D-10, §4.3 rewritten to `BoardSpec` + `run_board`; LD-11 rewritten and made provisional pending SPK-06; `L-03` deleted; §6.4 records the falsified alternative |
| §2.1 truncated spec §4.4 from fifteen enterprise families to five | **Upheld** — §4.4 has fifteen rows | §2.1 restored in full; §4.5's five-way modeling rule restored to I-13; dual overlay identity added to I-11 |
| `closure_allowed` uncomputable from single-run evidence | **Upheld** | `classify(evidence, prior, anomalies)`; `SourceAnomaly` + `PriorRunSummary` contracts added (D-11); `closure_allowed=False` when no prior |
| I-05 enforced by deny-lists with proven holes | **Upheld** | I-05 restated to a provable claim; L-01 becomes an allow-list; **L-01b** socket-egress assertion added; `service_workers="block"` moved to the production context; SSRF mechanism named as a resolving transport; the MockTransport SSRF claim withdrawn |
| This design commits its own D-7 (undefined contracts) | **Upheld** | Sixteen contracts named in §4.2; G1's exit condition enumerates them |
| The worked manifest example is wrong four ways | **Upheld** | Example deleted with the DSL; `total_is_independent` added to `CompletenessEvidence` so `EXACT` cannot rest on a self-consistent count |
| LD-10 tested what works, missed four real robots failures | **Upheld** | Four defects tabled; port reshaped to `decide(origin, path_and_query, product_token, fetch_outcome)`; RFC 9309 §2.3.1 status matrix made a G2 deliverable |
| D-2 is a manufactured defect | **Upheld** | D-2 withdrawn and corrected in place; `ContentBlob` gains both `wire_hash` and `decoded_hash` |
| `guard()` cannot be pure; jitter in core; `SourceScope` over-modeled | **Upheld** | `PageGuard` moved to the shell over pure `signal()` predicates; clock and entropy injected; `SourceScope` becomes a discriminated union |
| Failure modes unspecified; SPK-12/ADR-10 have no oracle | **Upheld** | §8.2b added: crash-resume, lease fencing token, SQLite writer serialization, vendor-change anomaly, live smoke test |
| Policy layer exempted from challenge; ~150 vocabulary items | **Upheld in part** | A minimum-viable-vocabulary pass added before G1 — but for *enumerations only*. The **invariants are not reduced**: they are cheap to hold and each has an oracle. Reducing them was not proposed and would not be accepted. |
| Recon §9 deleted on a naming inconsistency | **Upheld in part** | Disposition changed `delete` → `reshape`; the *contract family* still collapses into the run-mode view, but the **deliverable** (`source_observations.jsonl`, `search_checks`, `resolution_confidence`) is restored |

Three accuracy defects were also corrected: "8 direct deps pinned" (they are `>=` floors,
and there is no dev-dependency group), "the other five §10.5 conditions" (there are seven,
so six remain), and the invariant arithmetic. Spec §15's #16 and #17 — embedded metadata
does not prove enumeration, and personalized results are not the inventory — had been lost
in consolidation and are restored as **I-22** and **I-23**. They guard the two most likely
silent failures in the system.

**What this cost, and what it argues.** The manifest engine was the design's headline claim
for two drafts. It was falsified by reading the eight family specifications the design cited
but had summarized, and by one sentence in spec §5 that the design never quoted. The
cheapest place to find that was here; the most expensive would have been after G3 built the
interpreter. An adversarial review before acceptance is not ceremony.

---

## Acceptance

**accepted-with-named-assumptions.**

The four blocking findings are resolved in this revision. The design's core commitments —
one egress gate, a pure core, derived completeness, recon as a run mode, and shared
execution without a configuration language — are unchanged or strengthened by challenge.

Accepted subject to the seven assumptions in §11, of which three are load-bearing and carry
explicit reversal paths: **A-1** (`BoardSpec` scope, reversed by SPK-06), **A-3**
(`context.route()` coverage, which if it fails moves I-05's enforcement to container egress
policy), and **A-4** (robots conformance, reversed by SPK-08 behind a one-method port).

Two items are deferred to the implementation plan rather than blocking here: the field
shapes of the sixteen contracts named in §4.2, and the minimum-viable-vocabulary pass on the
enumerations — both are G1 exit conditions with named checks, and neither changes the
architecture.

**Not accepted, and deliberately so:** any claim that the Priority-2 board families are
solved. Spec §5 places Wave 1 coverage in Priority 1, SPK-06 has not run, and this design
now says so in three places rather than assuming otherwise.
