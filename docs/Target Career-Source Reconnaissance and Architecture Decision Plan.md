# Target Career-Source Reconnaissance and Architecture Decision Plan

## Large Financial Institutions · Internal Operations and Process Optimization · New York City

**Document status:** Execution plan for reconnaissance and architecture decisions  
**Reference date:** August 20, 2026  
**Primary objective:** Determine the smallest reliable direct-source acquisition architecture capable of comprehensively capturing public job postings from large financial institutions with substantial New York City hiring activity.

---

## 1. Executive summary

This plan defines the work required before producing the final, implementation-level design for the job-opportunity harvester.

The reconnaissance has five purposes:

1. **Map the actual career-source landscape** across the large financial institutions most relevant to the search.
2. **Determine how each source can be enumerated completely**, including pagination, job details, locations, and closure behavior.
3. **Build a representative fixture corpus** from the observed source families.
4. **Test and resolve the remaining architecture decisions** using real sources rather than assumptions.
5. **Produce the evidence and machine-readable contracts** needed to write source-specific detailed designs and hand implementation work to an LLM programming agent.

The search focus is deliberately simple:

```text
Industry:
  Large financial institutions

Broad mandate:
  Internal operations, process optimization, process excellence,
  operational excellence, transformation, productivity, workflow redesign,
  process engineering, and closely adjacent mandates

Geography:
  New York City
```

The crawler will still retrieve **all public postings from each target employer**. The mandate and geographic definitions will be used after collection to test search quality and inspect relevant results; they will not constrain acquisition.

The recommended architectural baseline is already sufficiently clear that the reconnaissance should validate it rather than reopen it without evidence:

```text
HTTPX2 direct acquisition
        ↓
Structured endpoint or feed when available
        ↓
Static HTML and embedded structured-data inspection
        ↓
Playwright network reconnaissance when browser execution is necessary
        ↓
Promote reproducible public network calls back into HTTPX2
        ↓
Browser DOM extraction only as a final fallback
```

The supplied reference pack explicitly recommends this separation of acquisition, parsing, structured-metadata extraction, and domain decomposition. It assigns direct HTTP to HTTPX2, browser-mediated acquisition to Playwright, fast HTML parsing to selectolax, XML and XPath work to lxml, embedded metadata extraction to extruct, and public-suffix-aware domain decomposition to tldextract.

---

# 2. Scope of the reconnaissance

## 2.1 In scope

The reconnaissance will determine:

- Which large financial institutions belong in the first target universe.
- Which legal entities and subsidiaries share employment brands and career boards.
- The official careers landing page for each institution.
- Every distinct public external-job source used by each employment brand.
- The source family or ATS vendor where identifiable.
- The actual public interface pattern used by the source.
- Whether listings are available through:
  - Public JSON
  - XML
  - RSS or Atom
  - JSON-LD or Microdata
  - Static HTML
  - JavaScript-backed public network calls
  - Browser-rendered DOM only
- How all jobs can be enumerated.
- How job details are retrieved.
- How locations are represented.
- How New York City jobs can be identified.
- Which useful fields are available and in what form.
- How active jobs disappear or are marked closed.
- Whether the source can be replayed deterministically from saved fixtures.
- Whether a reusable source-family adapter is justified.
- Which sources remain manual, blocked, or unresolved.

## 2.2 Out of scope

The reconnaissance will not:

- Build a sophisticated role-scoring model.
- Assess whether an individual job is attractive.
- Estimate interview probability.
- Match jobs to the resume.
- Automate applications.
- Automate authenticated LinkedIn use.
- Circumvent access controls, CAPTCHAs, or rate limits.
- Build every possible ATS adapter.
- Treat an ATS vendor name alone as proof that one adapter will support all of that vendor’s deployments.
- Infer that a source is complete merely because an HTTP request succeeded.
- Permanently exclude jobs based on title or mandate terms.

---

# 3. Definition of a successful reconnaissance

The reconnaissance is complete when the following questions have evidence-backed answers.

## 3.1 Employer coverage

For every institution in the priority sample:

- What is the official employer or employment brand?
- What legal entities map to it?
- Does it use one public job board or several?
- Are U.S., international, subsidiary, early-career, or acquired-company jobs separated?
- Is each board linked from an official employer-controlled page?
- What portion of the employer’s external jobs appears to be represented?

## 3.2 Acquisition

For every career source:

- Can it be accessed through ordinary HTTP?
- Does it expose structured listing or detail data?
- Is JavaScript required?
- If JavaScript is required, what browser request actually retrieves the jobs?
- Can that request be reproduced through HTTPX2 without browser state?
- Is browser DOM extraction genuinely necessary?

## 3.3 Enumeration

For every source:

- What constitutes a unique source job?
- Is there an advertised total?
- What pagination or cursor mechanism exists?
- How is the terminal page identified?
- Are results capped?
- Are there regional or language partitions?
- Can all source job identifiers be enumerated without applying a search query?
- What evidence supports the conclusion that enumeration was complete?

## 3.4 Data quality

For every source family:

- Which fields are structured?
- Which fields require HTML extraction?
- Which fields exist only in prose?
- Are dates reliable?
- Are multiple locations preserved?
- Is New York City represented consistently?
- Are description sections and bullets retained?
- Are application and canonical job URLs different?
- Are compensation fields structured or textual?

## 3.5 Stability and implementation value

For every recurring source family:

- How many target institutions use it?
- Approximately how much target job volume does it represent?
- Is the interface stable across tenants?
- Can it be tested offline?
- Can a dedicated adapter prove enumeration completeness?
- Is browser execution necessary in production?
- What maintenance burden is likely?

---

# 4. Target institution universe

## 4.1 Universe-construction principle

The initial universe should be based on both:

1. **Institution scale and relevance**
2. **Substantial New York City employment presence**

Regulatory datasets provide a useful starting point but describe legal entities rather than employment brands. The FFIEC National Information Center maintains a current large-holding-company list and structural data for financial institutions. New York’s Department of Financial Services maintains directories spanning banks, insurers, and other supervised institutions. SEC IAPD/Form ADV data covers investment advisers, FINRA provides information on registered broker-dealers, and NAIC provides insurer and subsidiary search resources.

These sources should seed and verify the universe, but the working object for career-source discovery must be the **employment brand**, not every regulated subsidiary.

```text
Regulated legal entities
        ↓
Corporate parents and operating subsidiaries
        ↓
Employment brands
        ↓
Public career sources
```

## 4.2 Wave 1 priority sample

The following 30 institutions form the initial reconnaissance universe.

### A. Large U.S. banks and capital-markets institutions

| Institution | Initial inclusion rationale |
|---|---|
| JPMorganChase | Very large NYC-centered financial institution and high-volume employer |
| Citi | Large NYC-headquartered global bank |
| Goldman Sachs | Major NYC investment bank and asset manager |
| Morgan Stanley | Major NYC investment bank and wealth manager |
| Bank of America | Large NYC corporate and investment-banking presence |
| BNY | NYC-headquartered custody, asset-servicing, and banking institution |
| Wells Fargo | Large bank with meaningful NYC corporate and markets hiring |
| Capital One | Large bank with NYC corporate and technology presence |
| HSBC | Major international bank with substantial NYC operations |
| Barclays | Major international investment-banking presence in NYC |
| Deutsche Bank | Major international bank with substantial NYC operations |
| UBS | Major wealth-management and investment-banking presence in NYC |

### B. Asset management, alternatives, and custody

| Institution | Initial inclusion rationale |
|---|---|
| BlackRock | Large NYC-headquartered asset manager |
| Blackstone | Large NYC-headquartered alternatives manager |
| State Street | Large custody and asset-servicing institution with NYC roles |
| Apollo Global Management | Large NYC-headquartered alternatives manager |
| KKR | Large NYC-headquartered alternatives manager |
| TIAA | Large NYC-headquartered retirement and financial-services institution |

### C. Insurance

| Institution | Initial inclusion rationale |
|---|---|
| MetLife | Large NYC-headquartered insurer |
| New York Life | Large NYC-headquartered life insurer |
| AIG | Large insurer with major NYC headquarters and operations |
| Prudential Financial | Large insurer and financial-services institution with NYC-region hiring |
| Chubb | Large insurer with substantial NYC-area operations |
| Guardian Life | Large NYC-headquartered insurer |

### D. Payments, market infrastructure, and public financial institutions

| Institution | Initial inclusion rationale |
|---|---|
| American Express | Large NYC-headquartered payments and financial-services company |
| Mastercard | Large payments institution headquartered in the NYC region |
| DTCC | Major post-trade market-infrastructure institution with NYC operations |
| Intercontinental Exchange / NYSE | Major exchange and market-infrastructure operator |
| Nasdaq | Major exchange and financial-technology operator with NYC operations |
| Federal Reserve Bank of New York | Major NYC financial-system institution with operations, technology, supervision, and transformation roles |

Official employer career sites remain the authoritative source for current external openings. For example, BlackRock expressly states that all roles for which it is recruiting externally are posted on its career site, while the New York Fed instructs candidates to verify and apply through the official Federal Reserve careers site.

## 4.3 Wave 2 expansion sample

Wave 2 should broaden foreign-bank, financial-information, insurance, and payments coverage after the first source-family map is established.

| Category | Institutions |
|---|---|
| Foreign banks | BNP Paribas, Société Générale, MUFG, Mizuho, RBC, TD, Santander, Macquarie |
| Payments | Visa |
| Ratings and financial information | S&P Global, Moody’s |
| Insurance and related financial services | Marsh McLennan, MassMutual |
| Asset management | Invesco, Franklin Templeton |

Wave 2 should not delay the first architecture decisions. It exists to test whether the adapter portfolio selected from Wave 1 generalizes to adjacent employers.

## 4.4 Anchor batch

The first reconnaissance batch should cover 12 institutions before all 30 are processed:

1. JPMorganChase  
2. Citi  
3. Goldman Sachs  
4. Morgan Stanley  
5. BNY  
6. BlackRock  
7. MetLife  
8. New York Life  
9. American Express  
10. DTCC  
11. Intercontinental Exchange / NYSE  
12. Federal Reserve Bank of New York  

This batch is intentionally diverse across business models and likely career-site architectures. No ATS classification should be assumed in advance; the source interface must be observed.

---

# 5. Role and geography probes

These definitions are not ingestion filters. They are validation queries used after complete board enumeration.

## 5.1 Broad mandate probe

### High-signal title and phrase terms

```text
process improvement
process optimization
process excellence
operational excellence
continuous improvement
business process management
process engineering
process transformation
operations transformation
business transformation
enterprise transformation
operating model
productivity
intelligent automation
workflow redesign
workflow transformation
process mining
lean six sigma
```

### Adjacent terms

```text
strategy and transformation
operations strategy
strategic initiatives
business architecture
COO office
enterprise change
automation center of excellence
service transformation
organizational transformation
business reengineering
operational effectiveness
efficiency
simplification
```

### Use of the terms

The reconnaissance should record:

- Total active jobs
- Jobs whose titles match a high-signal term
- Jobs whose descriptions match a high-signal term
- Jobs matching only an adjacent term
- Jobs that match both the mandate and NYC geography

No automatic negative-term exclusion is needed. False positives are useful because they show how broad the local search results will initially be.

## 5.2 Geography probe

### Default geography: `NYC_CORE`

```text
New York, NY
New York City
Manhattan
Brooklyn
Queens
Bronx
Staten Island
```

### Separately reported geography: `NYC_METRO_EXTENDED`

```text
Jersey City, NJ
Hoboken, NJ
Newark, NJ
White Plains, NY
Stamford, CT
Other explicitly identified NYC commuting locations
```

The default output should not silently combine `NYC_CORE` and `NYC_METRO_EXTENDED`.

### Remote jobs

Remote jobs should be represented separately:

```text
remote_us
remote_ny
remote_unspecified
hybrid_nyc
unknown
```

A remote posting should not be assumed to permit residence in New York unless the posting says so.

---

# 6. Reconnaissance deliverables

The reconnaissance should produce nine concrete deliverables.

## 6.1 Target institution register

One record per employment brand containing:

- Canonical employer name
- Alternate names
- Parent organization
- Relevant regulated entities
- Industry tags
- Official domain
- Priority wave
- Inclusion rationale
- NYC relevance
- Career-source status

## 6.2 Career-source register

One record per distinct public career source containing:

- Employer
- Official careers landing page
- Job-board URL
- Source hostname
- Vendor family, if known
- Public interface pattern
- Employment-brand scope
- Region or business scope
- Discovery evidence
- Verification state
- Current status

## 6.3 Source-observation dataset

A structured record of every reconnaissance run, including acquisition method, pagination, job count, field availability, errors, and completeness evidence.

## 6.4 Source-family catalog

A catalog of observed interface families, distinct from vendor names.

Example:

```text
Vendor family: Oracle Recruiting
Interface pattern: public Candidate Experience JSON endpoints

Vendor family: custom
Interface pattern: server-rendered HTML with JobPosting JSON-LD

Vendor family: Workday
Interface pattern: JavaScript search page backed by public POST search endpoint
```

## 6.5 Fixture corpus

Representative raw listing, detail, metadata, XML, and browser-network artifacts for each important source family.

## 6.6 Architecture-spike results

Structured comparisons of:

- HTTPX2 versus Playwright
- Public endpoint versus browser DOM
- selectolax versus lxml where both could apply
- Embedded metadata versus page extraction
- Generic recipes versus dedicated adapters
- Candidate orchestration and retry policies

## 6.7 ADR pack

A concise Architecture Decision Record for every accepted, rejected, or deferred decision.

## 6.8 Adapter-priority backlog

Source families and employers ranked by:

- Coverage value
- Job-volume value
- Importance to the target search
- Stability
- Completeness confidence
- Implementation effort

## 6.9 Coverage-gap register

Every unresolved or nonautomated employer/source, with:

- Failure category
- Evidence gathered
- Next action
- Manual fallback
- Impact on target-market coverage

---

# 7. Source classification model

The reconnaissance should classify both the **vendor family** and the **interface pattern**.

Vendor identity alone is not enough. Two deployments of the same ATS can expose materially different public interfaces.

## 7.1 Interface-pattern classes

| Class | Interface pattern | Preferred acquisition |
|---|---|---|
| S0 | Public structured listing and detail endpoint | HTTPX2 |
| S1 | Public XML, RSS, or Atom feed | HTTPX2 plus lxml or feedparser |
| S2 | Static detail pages with `JobPosting` structured data | HTTPX2 plus extruct |
| S3 | Static listing and detail HTML | HTTPX2 plus selectolax |
| S4 | JavaScript UI backed by reproducible public JSON or GraphQL calls | Playwright for discovery, then HTTPX2 |
| S5 | Browser-rendered DOM with no reproducible public endpoint | Playwright |
| S6 | Official email alert or manual-only source | Manual or mailbox-assisted |
| S7 | Blocked, authentication-dependent, or unsuitable for automation | Coverage exception |

## 7.2 Source-resolution states

```text
queued
investigating
resolved_structured_http
resolved_static_html
resolved_browser_endpoint
resolved_browser_dom
shared_board
manual_only
blocked
no_public_board_observed
unresolved
retired
```

## 7.3 Confidence states

| Confidence | Required evidence |
|---|---|
| High | Official employer link plus successful source enumeration and confirmed employer branding |
| Medium | Strong source and branding evidence, but enumeration or scope remains uncertain |
| Low | Search-engine or indirect evidence only; not accepted as resolved |

Search-engine results may identify a candidate source, but the source should not be accepted until an official employer-controlled page links to it or equivalent high-confidence evidence is obtained.

---

# 8. End-to-end reconnaissance workflow

## Stage 1 — Establish employer identity

For each target institution:

1. Record the canonical employment brand.
2. Record known parent and subsidiary names.
3. Record the official corporate domain.
4. Map relevant regulated entities to the employment brand.
5. Check whether subsidiaries appear to recruit separately.
6. Assign an institution identifier that remains stable even if the display name changes.

### Evidence

- Regulatory source record
- Official corporate website
- Official careers link
- Parent/subsidiary disclosure where necessary

### Result

```text
Institution resolved
or
Institution requires entity-resolution review
```

---

## Stage 2 — Discover official career sources

Starting from the official corporate domain:

1. Inspect homepage and footer links for:
   - Careers
   - Jobs
   - Join us
   - Opportunities
   - Work with us
2. Retrieve and inspect `robots.txt`.
3. Record declared sitemap URLs.
4. Inspect sitemap indexes and sitemaps for career and job paths.
5. Follow official redirects to vendor-hosted boards.
6. Inspect embedded scripts, widgets, forms, canonical links, and structured metadata.
7. Record every distinct board associated with:
   - Geography
   - Subsidiary
   - Business line
   - Early careers
   - Experienced hires
   - Acquired companies

The Robots Exclusion Protocol is standardized in RFC 9309 and includes rules for crawler directives, errors, and caching. The Sitemap protocol provides an XML format for publishing discoverable URLs and optional modification metadata.

### Result

```text
Official career-source candidates with evidence
```

---

## Stage 3 — Direct HTTP inspection

For every candidate source:

1. Retrieve the landing page with HTTPX2.
2. Preserve:
   - Original URL
   - Final URL
   - Redirect chain
   - Status
   - Selected safe response headers
   - Content type
   - Raw bytes
   - Retrieval timestamp
3. Inspect source HTML with selectolax.
4. Inspect embedded metadata with extruct.
5. Inspect linked or embedded XML feeds.
6. Identify listing links, detail links, scripts, and public data endpoints.
7. Attempt to enumerate jobs without applying title or location filters.
8. Record pagination and advertised counts.
9. Fetch representative detail pages.
10. Compare listing and detail field availability.

HTTPX2 clients should be reused rather than recreated per request, and production retrieval should use bounded timeouts, connection limits, streaming where necessary, narrow exception handling, and an explicit retry policy. The library’s transport retries cover only limited connection cases; broader retry behavior requires application policy. 
### Result

One of:

```text
Direct structured source identified
Static source identified
Browser escalation required
Access unsuitable
```

---

## Stage 4 — Structured-data inspection

For each HTML page:

1. Extract JSON-LD.
2. Identify:
   - A single `JobPosting`
   - Arrays of `JobPosting`
   - `@graph` entries
   - Multiple script blocks
3. Extract Microdata if JSON-LD is absent or incomplete.
4. Record structured-data provenance separately from page-derived values.
5. Compare structured metadata with visible page content.
6. Preserve conflicting values rather than silently choosing one.
7. Pass the final redirected page URL as the extraction base URL.

`extruct` supports JSON-LD, Microdata, Open Graph, Microformats, RDFa, and Dublin Core, but the reconnaissance should ordinarily limit extraction to JSON-LD and Microdata unless another syntax proves useful. It should preserve syntax-specific output rather than relying on `uniform` mode, and it must receive the actual final base URL for relative-link resolution. 
Schema.org defines `JobPosting` as a structured description of a job opening, including properties for locations, remote-location requirements, dates, organization, responsibilities, and other job facts. JSON-LD 1.1 is the standard JSON-based linked-data serialization commonly used to embed those records.

### Result

```text
Structured metadata is sufficient
Structured metadata is supplemental
Structured metadata is absent
Structured metadata conflicts with page
```

---

## Stage 5 — Browser-network reconnaissance

Escalate to Playwright only when direct HTTP does not expose a complete source.

Use a fresh, unauthenticated browser context configured consistently for:

- Locale
- Timezone
- User agent
- Viewport
- Service-worker policy
- Downloads
- Storage
- Permissions

Procedure:

1. Open the official job search.
2. Observe:
   - Requests
   - Responses
   - XHR and `fetch`
   - GraphQL traffic
   - Pagination calls
   - Detail calls
3. Capture the first, middle, and terminal pagination requests.
4. Record any search token, cursor, tenant ID, site ID, locale, or location parameter.
5. Capture a bounded HAR for the relevant workflow.
6. Save rendered HTML only when it contributes information unavailable in the network payload.
7. Attempt to replay candidate public requests through HTTPX2 from a clean process.
8. Determine whether cookies, anti-forgery values, or browser state are truly required.
9. Promote the source to an HTTP adapter whenever the public request is reproducible.
10. Use DOM extraction only when the data cannot be obtained more directly.

Playwright supports browser-isolated contexts, request and response observation, routing, HAR recording and replay, tracing, and deterministic environment configuration. The reference pack recommends HTTPX2 for general acquisition and Playwright only when JavaScript execution, browser state, interaction, or browser network observation is necessary. 
### Browser stopping conditions

Stop and mark the source appropriately if:

- A CAPTCHA must be bypassed.
- Authentication is required for external postings.
- The source blocks automated access and no appropriate public alternative exists.
- Repeated interaction would violate the site’s published rules.
- The source requires proxy rotation or stealth behavior to remain accessible.

### Result

One of:

```text
Public endpoint promoted to HTTPX2
Production browser acquisition justified
DOM-only extraction justified
Manual-only or blocked
```

---

## Stage 6 — Enumeration validation

For every working acquisition path:

1. Enumerate all source job IDs.
2. Record every page or cursor.
3. Compare unique count with advertised count.
4. Check for duplicate source IDs.
5. Validate the terminal pagination condition.
6. Retrieve every required detail record in the test run.
7. Compare source-level location filtering with local location filtering.
8. Search the local result set for the mandate probe.
9. Repeat the enumeration in a later independent run.
10. Explain every count difference as:
    - A real posting change
    - A source inconsistency
    - An incomplete run
    - A parser or deduplication problem

### Completeness grades

| Grade | Meaning |
|---|---|
| A | Advertised count reconciles to unique enumerated IDs; pagination and detail retrieval are complete; repeated run is consistent |
| B | No advertised count, but cursor or page termination is explicit and repeatable; all details retrieved |
| C | Enumeration appears complete but relies on UI or DOM heuristics |
| D | Partial or indeterminate |
| M | Manual-only |
| X | Blocked or unsuitable |

Only Grade A or B sources should be considered fully suitable for a high-confidence production adapter without an explicit exception.

---

## Stage 7 — Field-availability assessment

For each source, classify each field as:

```text
structured
embedded_metadata
html_extracted
description_only
absent
conflicting
```

### Fields to assess

| Field family | Fields |
|---|---|
| Identity | Source job ID, requisition ID, posting ID |
| Employer | Hiring organization, employment brand, subsidiary |
| Position | Title, department, job family, employment type |
| Dates | Date posted, modified date, valid-through date |
| Locations | Raw location, city, region, country, remote type, remote scope |
| Links | Listing URL, detail URL, application URL, canonical URL |
| Description | HTML, normalized text, headings, paragraphs, bullets |
| Compensation | Raw text, minimum, maximum, currency, period |
| Status | Active, unavailable, closed, unknown |
| Other | Language, business line, schedule, travel, category |

The reconnaissance is not required to infer missing values. It should identify what each source actually exposes.

---

## Stage 8 — Repeatability and change observation

Each Wave 1 source should be observed at least twice.

The repeated observation should determine:

- Whether source job IDs remain stable.
- Whether URLs remain stable.
- Whether timestamps change without content changes.
- Whether job ordering changes.
- Whether default location or keyword filters change.
- Whether the board returns personalized results.
- Whether cookies alter result count.
- Whether stale jobs remain visible.
- Whether removed jobs return explicit closure status or simply disappear.

---

# 9. Reconnaissance observation contract

## 9.1 Contract architecture

Use:

- **Pydantic 2.13.4** for internal Python data contracts, parsing, serialization, and JSON Schema generation.
- **JSON Schema Draft 2020-12** for external artifact validation.
- **jsonschema 4.26.0** to validate exported JSON, JSONL records, and AI-returned annotations independently of the internal Python implementation.

Pydantic provides compiled validation, structured errors, JSON-native validation, `TypeAdapter` for non-model record collections, and `model_json_schema()` for contract publication.

The `jsonschema` reference recommends an explicit `$schema` declaration, `Draft202012Validator`, schema validation through `check_schema()`, validator reuse, immutable local reference registries, and no automatic network reference resolution. 
## 9.2 Top-level observation record

```json
{
  "schema_version": "1.0",
  "observation_id": "obs_...",
  "institution_id": "inst_...",
  "source_id": "src_...",
  "observed_at": "2026-08-20T18:30:00Z",

  "institution": {
    "canonical_name": "Example Financial Group",
    "employment_brand": "Example",
    "official_domain": "example.com",
    "priority_wave": 1
  },

  "source": {
    "careers_landing_url": "https://example.com/careers",
    "board_url": "https://jobs.vendor.example/example",
    "vendor_family": "unknown",
    "interface_pattern": "javascript_public_endpoint",
    "employment_scope": "global_external",
    "resolution_state": "resolved_browser_endpoint",
    "resolution_confidence": "high"
  },

  "acquisition": {
    "direct_http_attempted": true,
    "browser_attempted": true,
    "production_method_candidate": "httpx2",
    "browser_required_for_discovery": true,
    "browser_required_for_production": false
  },

  "enumeration": {
    "advertised_count": 842,
    "enumerated_unique_count": 842,
    "pages_or_cursors": 43,
    "terminal_condition": "empty_next_cursor",
    "detail_fetch_count": 842,
    "detail_failure_count": 0,
    "completeness_grade": "A"
  },

  "search_checks": {
    "nyc_core_count": 74,
    "nyc_metro_extended_count": 11,
    "mandate_probe_count": 29,
    "nyc_and_mandate_count": 8
  },

  "fields": {
    "source_job_id": "structured",
    "requisition_id": "structured",
    "title": "structured",
    "locations": "structured",
    "date_posted": "structured",
    "description_html": "structured",
    "compensation": "description_only"
  },

  "artifacts": [],
  "errors": [],
  "notes": []
}
```

## 9.3 Required subrecords

### Institution identity

```text
institution_id
canonical_name
employment_brand
aliases
parent
official_domain
regulatory_source_ids
industry_tags
priority_wave
```

### Source identity

```text
source_id
careers_landing_url
board_url
source_hostname
vendor_family
interface_pattern
tenant_or_site_identifier
region
language
employment_scope
```

### Discovery evidence

```text
evidence_type
source_url
observation
retrieved_at
confidence
artifact_reference
```

### Acquisition result

```text
method
request_summary
response_summary
final_url
redirect_chain
content_type
bytes_received
browser_context_profile
required_state
```

### Enumeration result

```text
advertised_count
enumerated_count
unique_id_count
page_count
cursor_count
terminal_condition
duplicate_ids
failed_pages
failed_details
truncation_signals
completeness_grade
```

### Field assessment

```text
field_name
availability_class
source_path_or_selector
example_value
conflicts
notes
```

### Artifact reference

```text
artifact_id
artifact_type
content_hash
storage_reference
captured_at
redaction_status
source_url
```

### Error

```text
stage
error_class
retryable
message
request_or_page_reference
resolution
```

---

# 10. Fixture-capture protocol

## 10.1 Purpose

Fixtures should support:

- Offline parser tests
- Adapter contract tests
- Pagination tests
- Completeness tests
- Change-detection tests
- Regression testing after source or library upgrades
- Reproduction by an LLM programming agent

## 10.2 Required fixture types

### Direct HTTP sources

Capture:

- Listing first page
- Listing middle page
- Listing terminal page
- Empty-result query
- One ordinary job detail
- One NYC job detail
- One multi-location job
- One job with compensation, when available
- One job without optional fields
- One malformed or unexpected record, when observed
- One unavailable or closed job, when observed

### XML and feed sources

Capture:

- Complete feed or bounded representative subset
- Namespace declarations
- Pagination or feed-index structure
- Empty feed
- Malformed-but-observed item
- Relative URLs
- Encoding declaration

### JSON-LD sources

Capture:

- Single `JobPosting`
- Multiple script blocks
- Array form
- `@graph`
- Relative links
- Duplicate entities
- Malformed block adjacent to a valid block
- Server-source and rendered-source variants when they differ

### Browser sources

Capture:

- Landing-page source HTML
- Rendered HTML where useful
- Bounded HAR containing the listing workflow
- Request and response payloads for:
  - Initial search
  - Next page
  - Terminal page
  - Job detail
- Screenshot only as supporting evidence
- Trace only for complex or failing workflows

Screenshots should not be treated as extraction fixtures. Playwright’s reference explicitly distinguishes screenshots as observations rather than reliable DOM extraction formats and recommends tracing primarily as a forensic artifact.

## 10.3 Source fidelity

Retain original response bytes.

selectolax repairs and normalizes malformed HTML under HTML5 parsing rules, so serialized parser output is not guaranteed to equal the original input. The reference recommends preserving response bytes, parsing once, extracting primitives, and releasing parser-owned nodes.

For XML:

- Retain bytes so encoding declarations remain meaningful.
- Use an lxml parser configured with:
  - `resolve_entities=False`
  - `load_dtd=False`
  - `no_network=True`
  - `huge_tree=False`
- Use `iterparse` for unusually large XML rather than retaining an unbounded DOM.

lxml provides XML parsing, namespaces, XPath, streaming parse, and validation, but its untrusted-XML posture should explicitly disable external entities and network access.

## 10.4 Redaction

Fixtures must remove or avoid retaining:

- Authentication cookies
- Session tokens
- Anti-forgery tokens that function as credentials
- Personal identifiers
- Browser storage state
- Authorization headers
- Proxy credentials
- Unnecessary analytics identifiers

Public tenant IDs, site IDs, page cursors, and ordinary query parameters may be retained when necessary for fixture replay, provided they are not credentials.

## 10.5 Fixture manifest

Every artifact should have a manifest entry containing:

```text
Artifact ID
Institution
Source
Artifact type
Source URL
Capture method
Capture timestamp
HTTP status
Content type
Content hash
Browser version where applicable
Library versions
Redaction status
Expected parser result
Related test cases
```

## 10.6 Replay rule

A fixture is not considered complete until at least one test can consume it offline and produce an expected structured result.

---

# 11. Architecture decision plan

## ADR-01 — Direct acquisition default

### Question

Should the primary acquisition runtime be direct HTTP or a browser-crawler framework?

### Preliminary decision

Use **HTTPX2-first direct acquisition**.

### Evidence supporting the preliminary decision

- Target sources are known employer boards rather than arbitrary open-web pages.
- Enumeration and completeness require explicit source-specific control.
- HTTPX2 provides sync and async clients, connection pooling, bounded timeouts, streaming, HTTP/2, custom transports, and native mocking.
- Playwright remains available for browser-only discovery and fallback.

### Reconnaissance validation

Test HTTPX2 against:

- One public JSON source
- One static HTML source
- One XML source
- One JavaScript-backed public endpoint
- One source with redirects and transient failures
- One empty board

### Acceptance rule

Retain HTTPX2-first unless a crawler framework materially improves:

- Complete enumeration
- Per-host control
- Persisted-run integration
- Browser escalation
- Fixture replay

without obscuring those behaviors.

---

## ADR-02 — Browser role

### Question

Should Playwright be a routine production acquisition layer?

### Preliminary decision

Use Playwright for:

- Network reconnaissance
- JavaScript-only enumeration
- Required interaction
- Browser-only DOM extraction
- Failure forensics

Do not use it for sources that HTTPX2 can retrieve completely.

### Decision evidence

For each browser-observed endpoint:

1. Replay from a clean HTTPX2 client.
2. Compare job IDs and counts.
3. Determine whether browser state is required.
4. Record whether browser execution adds data.

### Acceptance rule

A source remains browser-backed only if direct HTTP cannot reproduce complete results.

---

## ADR-03 — HTML and XML parser division

### Preliminary decision

- **selectolax Lexbor** for ordinary HTML listing and detail parsing.
- **lxml** for XML, sitemaps, namespaces, XPath-heavy extraction, and large XML streaming.
- Do not parse the same HTML with both unless a demonstrated use case requires it.

The supplied references recommend Lexbor for new selectolax code and lxml when XML, XPath, namespaces, streaming, or schema facilities are needed. 
### Validation experiment

Apply both parsers to representative HTML fixtures and compare:

- Parse success
- Job-link discovery
- Field extraction
- Text extraction
- Runtime
- Memory
- Complexity of selectors

### Acceptance rule

Use selectolax unless lxml produces materially better correctness or enables a required XPath operation.

---

## ADR-04 — Embedded structured metadata

### Preliminary decision

Use extruct as a supplemental parser for:

- JSON-LD
- Microdata when needed

Do not enable every supported syntax by default.

### Validation experiment

Across at least 20 job-detail fixtures, compare:

- Structured metadata presence
- Field completeness
- Consistency with visible content
- Parse failures
- Additional runtime
- Utility for canonicalization

### Acceptance rule

JSON-LD extraction remains enabled if it supplies reliable fields or provenance on a meaningful share of source pages. Microdata is enabled only where observed.

---

## ADR-05 — Internal and external schema contracts

### Preliminary decision

Use Pydantic for internal contracts and JSON Schema 2020-12 plus `jsonschema` for published interchange contracts.

### Required behavior

- Every schema declares `$schema`.
- Pydantic-generated schemas are checked before publication.
- External schemas use stable `$id` values where references are needed.
- Referenced schemas are preloaded locally.
- Network schema retrieval is disabled.
- Validation errors are serialized structurally rather than parsed from human-readable text.

### Acceptance rule

A JSONL export produced from Pydantic models must validate independently through `Draft202012Validator`.

---

## ADR-06 — Domain and hostname policy

### Preliminary decision

Use:

1. Standard URL parsing
2. IDNA normalization
3. `tldextract`
4. Explicit storage of:
   - Full hostname
   - Registrable or top-domain boundary
   - Registry suffix
   - Public suffix
   - PSL policy and version

`tldextract` is explicitly not a URL validator, DNS resolver, IDNA canonicalizer, or security allowlist engine. Its reference recommends keeping URL validation and IDNA normalization separate, reusing an extractor, and recording PSL provenance when results must be reproducible. 
### Preliminary PSL policy

Use the bundled pinned PSL snapshot during reconnaissance:

```python
TLDExtract(
    suffix_list_urls=(),
    fallback_to_snapshot=True
)
```

Record the package version and policy.

### Acceptance rule

Domain classification must be deterministic across all fixtures and must not use naive final-two-label splitting.

---

## ADR-07 — Robots policy

### Preliminary decision

Implement robots handling according to RFC 9309.

Evaluate **Protego** as the parser because it supports modern robots conventions; do not assume the standard-library `robotparser` is sufficiently aligned with current RFC behavior without testing.

### Required tests

- User-agent selection
- Allow/disallow precedence
- Wildcards
- End anchors
- Redirected `robots.txt`
- Missing file
- 4xx and 5xx behavior
- Caching
- Sitemap directives
- Oversized file handling

### Acceptance rule

The selected parser must pass the project’s RFC-focused fixture corpus and expose sitemap directives.

---

## ADR-08 — Retry policy

### Preliminary decision

Use HTTPX2 exception classification plus a bounded retry layer.

Evaluate **Tenacity** for:

- Retry predicates
- Exponential backoff
- Jitter
- Attempt limits
- Total-time limits

Tenacity is a general-purpose retry library supporting configurable retry, wait, and stop behavior.

### Retryable candidates

```text
DNS/connect failure
connection reset
read timeout
408
429
selected 5xx responses
```

### Normally non-retryable

```text
ordinary 4xx
invalid URL
robots prohibition
unsupported content type
authentication requirement
parser contract failure
```

### Acceptance rule

Every retry must be observable and bounded, and `Retry-After` must take precedence where supplied.

---

## ADR-09 — Source adapter granularity

### Question

When should a source use:

- A dedicated coded adapter
- A generic configured HTML recipe
- A browser recipe

### Decision experiment

Implement one recurring structured family and two custom HTML sources using both:

1. Generic configuration
2. Dedicated adapter code

Compare:

- Ability to prove completeness
- Error reporting
- Pagination clarity
- Fixture testing
- Maintainability
- Source-specific exceptions

### Preliminary rule

Use a dedicated adapter when a source family:

- Covers at least two Wave 1 institutions, or
- Represents a material share of observed job volume, or
- Is the only route to a strategically important institution, or
- Requires nontrivial pagination or detail behavior

Use generic static recipes for low-volume one-off sources with simple HTML.

---

## ADR-10 — Async orchestration

### Preliminary decision

Use a small application-owned asynchronous orchestration layer around:

- HTTPX2
- `asyncio.TaskGroup`
- Per-host concurrency controls
- Persisted crawl-run state

Do not adopt distributed workflow infrastructure during reconnaissance.

### Validation experiment

Run the anchor batch with:

- Bounded global concurrency
- Bounded per-host concurrency
- Crash and restart simulation
- Retry simulation
- Partial-detail failure
- Duplicate run attempt

### Reconsideration trigger

Evaluate a broader crawler framework only if the custom layer becomes unable to express:

- Due-source scheduling
- Host throttling
- Idempotent run state
- Browser escalation
- Offline fixture testing

without substantial duplicated infrastructure.

---

## ADR-11 — Browser fixture strategy

### Preliminary decision

Use:

- Raw JSON responses whenever possible
- Bounded HAR for network workflow evidence
- Rendered HTML for DOM-only behavior
- Trace only for failure forensics
- Screenshot only for human evidence

Playwright HAR replay can provide deterministic network behavior, but a HAR is not a complete browser-state snapshot. Service workers may need to be blocked for deterministic routing and capture.

---

## ADR-12 — Feed handling

### Preliminary decision

Treat RSS and Atom as opportunistic source types.

Use `feedparser` only if the reconnaissance discovers relevant feeds; otherwise avoid adding it to the core dependency set. Current documentation describes feedparser as a parser for RSS, Atom, and JSON feeds.

---

# 12. Architecture-spike matrix

| Spike | Input sources | Main measurement | Decision produced |
|---|---|---|---|
| SPK-01 Direct HTTP baseline | Structured, HTML, and empty boards | Count completeness, latency, requests, errors | HTTPX2 acquisition template |
| SPK-02 Browser endpoint promotion | At least three JS boards | Browser count versus replayed HTTP count | Endpoint-backed versus browser-backed classification |
| SPK-03 HTML parser comparison | Representative HTML fixtures | Correctness, selector complexity, runtime | selectolax or lxml for each pattern |
| SPK-04 Structured metadata value | At least 20 detail pages | Field completeness and agreement | extruct syntax policy |
| SPK-05 Enumeration proof | Paginated and cursor boards | Count reconciliation and terminal condition | Completeness rubric |
| SPK-06 Adapter granularity | Recurring ATS plus custom boards | Maintainability and proof strength | Dedicated versus configured adapter policy |
| SPK-07 Retry and throttling | Simulated 429, timeout, 5xx | Bounded recovery and observability | Retry policy |
| SPK-08 Robots handling | RFC-focused corpus and live examples | Correct allow/disallow behavior | Robots parser selection |
| SPK-09 Domain normalization | Vendor, employer, private-suffix, IDN hosts | Stable domain facts | URL/IDNA/PSL policy |
| SPK-10 Contract validation | Observation and fixture records | Independent schema validation | Pydantic/JSON Schema publication process |
| SPK-11 Change detection | Two observations per source | Real changes versus acquisition drift | Snapshot and source-health rules |
| SPK-12 Scheduler recovery | Interrupted multi-source run | Idempotence and resumability | Orchestration design |

---

# 13. Source-family implementation priority

## 13.1 Evaluation dimensions

Every observed source family should receive a simple qualitative assessment.

| Dimension | Question |
|---|---|
| Employer coverage | How many Wave 1 institutions use it? |
| Job-volume coverage | How much observed posting inventory does it represent? |
| Search relevance | Does it cover institutions likely to post target roles? |
| Structured access | Are listing and detail fields machine-readable? |
| Completeness confidence | Can complete enumeration be proved? |
| Stability | Are identifiers, pagination, and endpoints repeatable? |
| Production cost | Does it require a browser? |
| Implementation effort | How much source-specific behavior is required? |
| Testability | Can it be reproduced through saved fixtures? |

## 13.2 Priority bands

### Priority A

Implement before the first comprehensive production crawl.

Criteria:

- Covers several Wave 1 institutions, or
- Covers a very high-volume or strategically important institution
- Supports Grade A or B completeness
- Has a reasonably stable public interface

### Priority B

Implement after Priority A.

Criteria:

- Meaningful additional institution coverage
- Manageable implementation effort
- Good but not dominant job-volume contribution

### Priority C

Use generic handling or defer.

Criteria:

- One low-volume institution
- Weak target-search relevance
- High maintenance burden
- Browser-only without substantial coverage value

### Manual exception

Retain a visible manual process when:

- Access is unsuitable
- Automation is blocked
- The employer has very few postings
- The implementation cost is disproportionate

## 13.3 Release-one portfolio target

The first adapter portfolio should aim to cover:

- At least 80% of Wave 1 employment brands through automated healthy sources
- At least 90% of the observed Wave 1 job volume
- Every institution in the anchor batch, unless explicitly classified as manual or blocked

These targets are not claims of universal market completeness. They are decision thresholds for the first implementation portfolio.

---

# 14. Library and protocol register

## 14.1 Core adopted libraries

| Library | Planned role | Status |
|---|---|---|
| HTTPX2 2.9.1 | Direct HTTP, streaming, pooling, transports, mocking | Adopt |
| Playwright 1.62.0 | Browser reconnaissance and fallback | Adopt |
| selectolax 0.4.11 | Primary static HTML parser | Adopt |
| lxml 6.1.1 | XML, sitemap, namespace, and XPath handling | Adopt |
| extruct 0.18.0 | JSON-LD and selective Microdata extraction | Adopt with constrained syntax set |
| tldextract 5.3.2 | PSL-aware hostname decomposition | Adopt |
| Pydantic 2.13.4 | Internal typed contracts and serialization | Adopt |
| jsonschema 4.26.0 | Independent JSON Schema validation | Adopt |

The common runtime floor implied by the supplied stable references is Python 3.10, and all behavior-affecting versions should be locked before implementation. 
## 14.2 Candidate supporting libraries

| Library | Role | Decision posture |
|---|---|---|
| Protego | Modern robots parsing | Evaluate against RFC corpus |
| Tenacity | Bounded retry policy | Evaluate, likely adopt |
| feedparser | RSS and Atom | Add only if observed |
| `idna` | Hostname IDNA/UTS 46 normalization | Adopt if explicit canonicalization is required |
| pytest | Test runner | Adopt |
| pytest-asyncio | Async tests | Adopt |
| Hypothesis | Lifecycle and parser invariants | Adopt in detailed test design |
| Native HTTPX2 MockTransport | Direct HTTP fixture tests | Adopt |
| Playwright HAR replay | Browser-network fixture tests | Adopt selectively |

The Python `idna` package supports IDNA 2008 and UTS 46 compatibility processing, which is useful because tldextract intentionally does not perform full hostname canonicalization.

## 14.3 Protocols and formats

| Protocol or format | Use |
|---|---|
| HTTP/1.1 and HTTP/2 | Source retrieval |
| HTTP redirects and cache validators | Final URL and conditional acquisition |
| Robots Exclusion Protocol, RFC 9309 | Access policy |
| Sitemap XML protocol | Career and job URL discovery |
| JSON-LD 1.1 | Embedded linked job data |
| Schema.org `JobPosting` | Job structured-data vocabulary |
| Microdata | Alternate embedded structured data |
| RSS and Atom | Optional published feeds |
| HAR | Browser network capture and replay |
| JSON Schema Draft 2020-12 | Machine-readable contracts |
| JSONL | Reconnaissance records and AI batch exchange |
| CSV | Human review |
| IDNA and Public Suffix List | Hostname normalization and decomposition |

---

# 15. Measurement framework

## 15.1 Coverage measurements

```text
Wave 1 institutions
Employment brands resolved
Career landing pages resolved
Career sources resolved
Automated healthy sources
Manual-only sources
Blocked sources
Unresolved sources
```

## 15.2 Acquisition measurements

```text
Jobs enumerated
Unique source job IDs
Advertised count
Listing pages or cursors
Detail fetch success
Direct HTTP sources
Browser-discovered but HTTP-replayable sources
Production browser sources
DOM-only sources
```

## 15.3 Structure measurements

Percentage of jobs with:

```text
Title
Source ID
Requisition ID
Location
Posting date
Department
Employment type
Description HTML
Description text
Application URL
Compensation
```

## 15.4 Target-search validation

Per employer and in aggregate:

```text
Active jobs
NYC_CORE jobs
NYC_METRO_EXTENDED jobs
Mandate-probe jobs
NYC_CORE + mandate-probe jobs
Unknown-location jobs
```

## 15.5 Completeness measurements

```text
Grade A sources
Grade B sources
Grade C sources
Indeterminate sources
Count mismatches
Pagination anomalies
Detail failures
Duplicate source IDs
Unexplained zero-job results
```

## 15.6 Operational measurements

```text
Requests per source
Requests per job
Bytes retrieved
Browser pages opened
Retry count
429 count
Timeout count
Run duration
Parser failures
Source changes between runs
```

These are reconnaissance measurements, not optimization targets by themselves.

---

# 16. Failure taxonomy

Every problem should be recorded under one primary category.

```text
institution_identity_failure
employment_brand_mapping_failure
official_domain_failure
career_source_discovery_failure
source_scope_ambiguity
robots_prohibition
access_blocked
authentication_required
http_fetch_failure
browser_navigation_failure
network_endpoint_not_replayable
listing_parse_failure
detail_parse_failure
pagination_failure
advertised_count_mismatch
source_id_collision
location_parse_failure
structured_data_conflict
fixture_capture_failure
schema_validation_failure
change_detection_ambiguity
```

This taxonomy will later become the basis for source health and engineering backlogs.

---

# 17. Security and access controls

## 17.1 URL and network policy

All discovered URLs should be treated as untrusted input.

Required controls include:

- Permit only expected schemes.
- Reject embedded credentials.
- Validate every redirect target.
- Block localhost, link-local, private, and metadata-service addresses.
- Bound response size.
- Bound redirect count.
- Bound decompression expansion.
- Keep TLS verification enabled.
- Do not trust `Content-Type` alone.
- Do not log cookies or authorization information.

HTTPX2 is a transport client, not an SSRF firewall; its own reference calls for explicit URL, redirect, DNS/IP, size, TLS, and logging controls when retrieving untrusted URLs.

## 17.2 XML policy

For untrusted XML:

```text
External entities disabled
DTD loading disabled
Network resolution disabled
Huge-tree mode disabled
Input size bounded
Streaming used for very large documents
```

## 17.3 Browser policy

- Use ephemeral contexts.
- Do not use a personal browser profile.
- Do not persist storage state.
- Keep browser sandboxing enabled.
- Restrict downloads.
- Do not expose local files or secrets.
- Do not use stealth or CAPTCHA-solving tooling.
- Close contexts after each isolated source investigation.

## 17.4 Schema policy

- No arbitrary remote `$ref` retrieval.
- No custom schema keyword that executes I/O.
- No dynamic imports from schema data.
- Validate schema documents before using them.
- Bound schema and instance sizes.

---

# 18. Reconnaissance execution sequence

## Gate 1 — Contract readiness

Complete:

- Observation schema
- Fixture manifest schema
- Error taxonomy
- Completeness grades
- Source classification vocabulary
- Architecture-spike templates

No institution reconnaissance should begin until these records can be validated.

## Gate 2 — Anchor-batch reconnaissance

Process the 12 anchor institutions.

Output:

- First source-family map
- Initial fixture corpus
- Browser-escalation examples
- Preliminary adapter-priority list

## Gate 3 — Core architecture spikes

Complete:

- HTTPX2 acquisition spike
- Browser endpoint-promotion spike
- Parser comparison
- Structured-metadata assessment
- Robots-policy validation
- Schema-validation round trip

Resolve or update the preliminary ADRs.

## Gate 4 — Remaining Wave 1 reconnaissance

Process the other 18 Wave 1 institutions using the standardized workflow.

Do not change observation fields ad hoc. Version the schema if new facts must be represented.

## Gate 5 — Repeat observations

Run a second independent observation against every Wave 1 source.

Resolve:

- Stability
- Source-ID behavior
- Count differences
- Closure signals
- Personalization or cookie effects

## Gate 6 — Adapter portfolio decision

Select:

- Dedicated source-family adapters
- Generic HTML/JSON-LD handling
- Browser-backed adapters
- Manual exceptions

## Gate 7 — Final architecture handoff

Publish:

- ADR pack
- Schemas
- Fixture corpus
- Source-family specifications
- Adapter backlog
- Coverage report
- Remaining detailed-design work packets

---

# 19. Final handoff package after reconnaissance

The package handed into detailed design should contain:

## 19.1 Master reconnaissance report

- Institution coverage
- Source-family distribution
- Job-volume distribution
- NYC job distribution
- Mandate-probe results
- Field-availability results
- Completeness grades
- Browser dependence
- Coverage gaps

## 19.2 Machine-readable datasets

```text
institutions.jsonl
career_sources.jsonl
source_observations.jsonl
source_family_catalog.json
fixture_manifest.jsonl
known_gaps.jsonl
```

## 19.3 Validated schemas

```text
institution.schema.json
career_source.schema.json
source_observation.schema.json
fixture_manifest.schema.json
error.schema.json
```

## 19.4 Fixture corpus

Grouped logically by source family and source, but without requiring a prescriptive repository structure in the design.

## 19.5 Source-family mini-specifications

For every source family selected for implementation:

```text
Detection
Enumeration
Pagination
Detail retrieval
Field mapping
Completeness proof
Failure handling
Fixtures
Tests
Known anomalies
```

## 19.6 Architecture Decision Records

Each ADR should include:

```text
Question
Context
Alternatives
Evidence
Decision
Consequences
Reconsideration trigger
```

## 19.7 Agent implementation backlog

Each implementation packet should state:

- Objective
- Governing source-family spec
- Inputs and outputs
- Invariants
- Fixtures
- Required tests
- Failure behavior
- Acceptance commands
- Explicit non-goals

---

# 20. Acceptance criteria for this reconnaissance plan

The plan is successfully executed when:

## Institution and source coverage

- All 30 Wave 1 institutions have a recorded status.
- Every institution has an official-domain determination.
- Every institution has either:
  - A resolved career source
  - A manual-only classification
  - A blocked classification
  - A no-public-source observation
  - A documented unresolved gap
- Shared employment boards are identified and not double-counted.

## Acquisition

- Every resolved source has a documented preferred acquisition method.
- Every Playwright-discovered endpoint has been tested for HTTPX2 replay.
- Browser production use is justified source by source.
- No source depends on bypassing access controls.

## Completeness

- Every resolved source has a completeness grade.
- Every Grade A source reconciles advertised and enumerated counts.
- Every Grade B source has explicit repeatable terminal-pagination evidence.
- No failed or partial run is represented as a valid zero-job result.
- Every unexplained count discrepancy is recorded.

## Fixtures

- Every recurring source family has a representative fixture set.
- Every fixture has a manifest and content hash.
- Every fixture can be consumed offline.
- No fixture contains credentials or sensitive browser state.

## Data structure

- The observation and fixture records validate under Pydantic and independent Draft 2020-12 JSON Schema validation.
- The field-availability classification is complete for every resolved source.
- Raw locations and complete descriptions are preserved.
- Multiple locations are not collapsed into one string.

## Search validation

- Local mandate and geography probes can be run against every fully enumerated source.
- Results report:
  - Total jobs
  - NYC_CORE jobs
  - Mandate matches
  - NYC_CORE plus mandate matches
- The query can be changed without re-retrieving jobs.

## Architecture decisions

- Every preliminary ADR has been accepted, rejected, or deferred with evidence.
- The selected release-one adapter portfolio has explicit institution and job-volume coverage.
- Remaining manual and unresolved coverage gaps are visible.

---

# 21. Immediate first execution batch

The first concrete reconnaissance run should proceed in this order:

1. Finalize the Pydantic observation models.
2. Generate Draft 2020-12 JSON Schemas.
3. Validate the schemas with `jsonschema`.
4. Establish URL, IDNA, and PSL normalization policy.
5. Implement the reconnaissance-only HTTPX2 fetch harness.
6. Implement raw artifact and manifest capture.
7. Implement robots and sitemap inspection.
8. Implement selectolax HTML inspection.
9. Implement lxml sitemap/XML inspection.
10. Implement extruct JSON-LD inspection.
11. Configure a reproducible Playwright Chromium environment.
12. Process:
    - JPMorganChase
    - Citi
    - Goldman Sachs
    - Morgan Stanley
13. Review and adjust only the observation schema—not the production architecture—based on missing evidence fields.
14. Process the remaining eight anchor institutions.
15. Run the first architecture spikes.
16. Freeze the reconnaissance schema version before completing the remaining Wave 1 institutions.

The output of that sequence will provide the empirical basis for the next design layer: the canonical source contracts, acquisition state machines, source-family adapter specifications, completeness logic, fixture-backed test suites, and ordered implementation packets suitable for a programming agent.