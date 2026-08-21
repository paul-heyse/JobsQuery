# Comprehensive Career-Source Inventory and Detailed Acquisition Design Specification

## Large Financial Institutions · Internal Operations and Process Optimization · New York City

**Document status:** Detailed design basis for implementation by an LLM programming agent  
**Design version:** 0.4  
**Reference date:** August 20, 2026

---

# 1. Purpose

This document defines:

1. The comprehensive array of web sources that should be reviewed to identify the target employer universe and retrieve public job postings.
2. The source hierarchy and acquisition strategy.
3. Canonical source and adapter contracts.
4. Acquisition and lifecycle state machines.
5. Source-family adapter requirements.
6. The logic for determining whether a source was enumerated completely.
7. The required fixture-backed test architecture.
8. An ordered set of implementation packets suitable for an LLM programming agent.

The target search is initially:

```text
Industry:
  Large financial institutions

Broad mandate:
  Internal operations, process optimization, operational excellence,
  process engineering, productivity, business transformation,
  operating-model redesign, workflow improvement, intelligent automation,
  and closely adjacent mandates

Geography:
  New York City
```

These criteria apply to querying and curating the collected data. They must not restrict job acquisition. The system should retrieve every public external job from each in-scope employer and filter locally afterward.

The design deliberately does not specify:

- A repository layout
- A complex role-attractiveness model
- Candidate-to-role scoring
- Interview-probability estimation
- Automated job applications
- A paid job-data API
- Automated access to authenticated job-board accounts

---

# 2. Executive architecture

The system should be organized around a durable source registry and a set of independently testable adapters.

```text
Employer-universe sources
        ↓
Employment-brand resolution
        ↓
Official career-source discovery
        ↓
Source-family detection
        ↓
Complete source enumeration
        ↓
Raw artifact preservation
        ↓
Source-specific parsing
        ↓
Canonical job normalization
        ↓
Lifecycle reconciliation
        ↓
Industry + mandate + geography queries
        ↓
Manual curation and export
```

The acquisition escalation path should be:

```text
Official structured source
        ↓ unavailable
Ordinary HTTP listing/detail pages
        ↓ insufficient
Embedded structured metadata
        ↓ insufficient for enumeration
Static scripts and page configuration
        ↓ insufficient
Playwright network reconnaissance
        ↓
Reproduce public browser request through HTTPX2
        ↓ impossible
Playwright production acquisition
        ↓ unsuitable
Manual-only or blocked source
```

The supplied web-acquisition reference pack supports this division of responsibility: begin with HTTPX2, escalate to Playwright only when browser execution is required, use selectolax for ordinary HTML, lxml for XML and XPath-heavy work, extruct for embedded metadata, and tldextract for public-suffix-aware domain decomposition.

---

# 3. Governing design principles

## 3.1 Employer-first, not query-first

The system should begin with a defined employer universe, identify every career source associated with each employer, and retrieve the complete public posting inventory from those sources.

A search such as “process improvement jobs in New York” is insufficient as an acquisition mechanism because:

- Relevant roles use inconsistent titles.
- Search interfaces may cap or rank results.
- Search engines may omit recently posted jobs.
- Results may be personalized.
- Different subsidiaries may use different career sites.
- A search result does not provide an auditable denominator.

## 3.2 Retrieve broadly and filter locally

No mandate, title, seniority, or location term should be used to limit the production crawl unless a source cannot otherwise be enumerated and a partitioned crawl is required.

The local dataset should support changing:

- Mandate terms
- NYC definitions
- Industry boundaries
- Freshness windows
- Employer inclusions
- Manual tags

without retrieving the jobs again.

## 3.3 Official sources are authoritative

The normal authority hierarchy should be:

1. Official employer job-detail page or official employer-linked career source
2. Public machine-readable interface powering that official source
3. Official employer job feed or job-alert email
4. Officially embedded `JobPosting` metadata
5. External job boards and search engines
6. Historical web archives

Third-party sources are useful for discovering misses. They should not normally override an active/closed status or description obtained directly from the employer.

## 3.4 Acquisition, parsing, and normalization are separate

An adapter should not conflate:

```text
retrieving bytes
parsing source data
normalizing fields
reconciling job lifecycle
```

This separation allows:

- Parsers to be replayed against saved fixtures.
- Network behavior to be tested independently.
- Historical snapshots to be reprocessed after parser improvements.
- Browser-backed sources to hand network payloads to the same parsing layer used by HTTP sources.
- Source-specific field interpretation to remain auditable.

## 3.5 Direct HTTP is the default

Adapters must not instantiate browsers by default.

HTTPX2 should handle:

- Ordinary HTML
- JSON
- XML
- RSS and Atom
- Public ATS endpoints
- Browser-discovered public network endpoints
- Conditional requests
- Streaming large feeds

Playwright should be used when JavaScript execution, interaction, browser state, or network observation is necessary. The browser should then be used to identify a more direct public data source wherever possible.

## 3.6 Completeness is an evidence-backed conclusion

A request returning HTTP 200 is not proof that a source was completely enumerated.

A source run should retain evidence for:

- Board scope
- Pagination or cursor traversal
- Advertised count
- Unique source-job count
- Terminal condition
- Failed listing pages
- Failed job details
- Duplicate source IDs
- Truncation indicators
- Source anomalies
- Whether the source was personalized or filtered

## 3.7 No silent zero

The system must distinguish:

```text
The employer currently has zero public jobs.
```

from:

```text
The source failed, changed, blocked the crawler, or returned an incomplete result.
```

A zero-job result is valid only after a complete source enumeration.

## 3.8 Incomplete runs cannot close jobs

A job must not be marked closed because:

- A request timed out
- An adapter failed
- A browser challenge appeared
- Pagination was incomplete
- A parser returned zero records unexpectedly
- The source changed its HTML
- A regional board was missed
- A source run was aborted

## 3.9 Raw source data is immutable

Every source observation should retain:

- Original bytes or response payload
- Final URL
- Redirect chain
- Retrieval time
- Relevant response headers
- Content type
- Content hash
- Source adapter and parser version

Normalized data may be rebuilt. Raw source artifacts should not be overwritten.

## 3.10 Source adapters cannot bypass shared policy

Adapters should receive a policy-controlled acquisition interface. They must not independently create:

- HTTP clients
- Browser processes
- Retry loops
- Robots policies
- Proxy behavior
- Unbounded concurrency

This ensures that security, retry, throttling, logging, and snapshot behavior remain consistent.

---

# 4. Comprehensive web-source inventory

The system should distinguish between sources used to define employers, sources used to retrieve jobs, and sources used only to audit coverage.

---

## 4.1 Employer-universe and entity-resolution sources

These sources answer:

> Which large financial institutions should be monitored, and which legal entities map to the same employment brand?

| Source | Primary use | Authority and limitations |
|---|---|---|
| FFIEC National Information Center | Identify large bank and financial holding companies; obtain RSSD IDs and organizational structures | Particularly useful for large holding companies and banking groups. The FFIEC large-holding-company list covers holding companies reporting more than $10 billion in assets. |
| FDIC BankFind and bulk institution data | Identify FDIC-insured banks, official websites, certificates, locations, status, and historical entities | Strong banking-universe source, but many legal institutions may map to one employment brand. |
| OCC institution lists | Identify national banks, federal savings associations, trust banks, and federal branches and agencies | Useful for U.S. and foreign banking entities operating under OCC supervision. |
| New York State Department of Financial Services | Identify institutions regulated or licensed in New York, including banks, insurers, lenders, and other financial-services organizations | Important for NYC-specific universe expansion. |
| SEC IAPD and Form ADV data | Identify investment advisers, alternative asset managers, and registered advisory entities | The SEC publishes downloadable adviser information derived from Form ADV and links to current IAPD records. |
| FINRA broker-dealer firm data and BrokerCheck | Identify broker-dealers and securities firms | FINRA’s broker-dealer firm dataset covers registered firms and can supplement bank and adviser lists. |
| NAIC company search | Identify insurance companies, subsidiaries, and legal operating names | Useful for mapping insurers and subsidiaries to employment brands. |
| GLEIF LEI data | Identify legal entities, alternate names, and corporate relationships across jurisdictions | LEIs provide unique legal-entity identifiers and publicly available entity information. |
| SEC EDGAR | Confirm public-company names, subsidiaries, corporate domains, mergers, and current filings | Useful for resolving parent relationships and employment-brand changes. |
| Official corporate websites | Confirm current brand, official domain, investor relations, operating businesses, and official careers links | Employer-controlled, but may present marketing brands rather than legal structure. |
| Annual reports, 10-Ks, 20-Fs, and proxy filings | Resolve material subsidiaries, acquisitions, business segments, and names | Strong evidence for parent/subsidiary relationships but not necessarily current career-board structure. |
| Public exchange, clearinghouse, and market-infrastructure directories | Add institutions such as exchanges, central counterparties, and depositories that are not captured well by bank datasets | Usually curated explicitly for this search universe. |
| Trade-association member lists | Recall expansion for asset managers, insurers, payments firms, and specialty lenders | Useful for discovery, not authoritative entity resolution. |
| Manually curated employer list | Include strategically important employers that fall outside regulatory datasets | Every manual addition should retain an inclusion reason and official-domain evidence. |

### Required entity model

The system should represent:

```text
Regulatory or legal entity
        ↓ many-to-one or many-to-many mapping
Employment brand
        ↓ one-to-many
Career sources
```

A shared parent career board should be crawled once even when dozens of regulated subsidiaries map to it.

---

## 4.2 Official employer-controlled web surfaces

For every employment brand, the source-discovery process should review all of the following.

| Surface | What to inspect |
|---|---|
| Corporate homepage | Official careers links, subsidiary links, alternate domains |
| Careers landing page | Board links, experienced-hire and early-career separation, country selectors |
| Job-search page | Search interface, counts, filters, pagination, scripts, network requests |
| Job-detail page | Source IDs, requisition IDs, descriptions, locations, dates, compensation, canonical and apply URLs |
| Application page | Underlying ATS identity and stable posting identifier; do not automate submission |
| Regional career sites | U.S., Americas, global, or country-specific boards |
| Subsidiary boards | Acquired companies, wealth-management brands, insurance subsidiaries, technology units |
| Early-career boards | Record separately; include or exclude through later query policy |
| Contractor boards | Record separately if publicly linked |
| `robots.txt` | Crawl directives and declared sitemaps |
| Sitemap indexes | Career, job, country, and language-specific sitemaps |
| Job-specific sitemaps | Candidate source of complete job-detail URLs |
| RSS or Atom feeds | Standardized incremental posting data |
| XML feeds | Full job inventory where exposed |
| JSON-LD | `JobPosting`, `Organization`, and related embedded entities |
| Microdata | Alternate structured `JobPosting` representation |
| Canonical links | Canonical detail URL and duplicate-page relationships |
| Embedded JavaScript configuration | Tenant IDs, site IDs, locale, public endpoint paths |
| XHR, `fetch`, and GraphQL calls | Public listing, detail, location, and facet data |
| Iframes and widgets | Underlying vendor-hosted source |
| Job-alert signup | Official email-based fallback and change-audit channel |
| Official employer email alerts | Useful when automated source access is not viable |
| Social/job-board links | External sources to use only for coverage audits |

Google explicitly supports `JobPosting` structured data on job-detail pages, making JSON-LD and Microdata important generic extraction sources. However, embedded structured data ordinarily helps parse a detail page; it does not itself prove that every job was discovered.

---

## 4.3 Public or machine-readable ATS source families

These should receive dedicated detection logic and, where valuable, dedicated adapters.

### 4.3.1 Greenhouse Job Board

**Typical signatures**

```text
boards.greenhouse.io
job-boards.greenhouse.io
boards-api.greenhouse.io
```

**Preferred source**

Greenhouse’s public Job Board API.

**Why it is valuable**

The list operation exposes all job posts for a board token and can include complete descriptions, departments, offices, posting dates, requisition IDs, locations, and metadata. The response also exposes a total count. A detail operation is available for a specific posting.

**Adapter priority**

High. It is structured, relatively uniform, and easy to test.

---

### 4.3.2 Lever Postings API

**Typical signatures**

```text
jobs.lever.co
jobs.eu.lever.co
api.lever.co
api.eu.lever.co
```

**Preferred source**

Lever’s public Postings API.

**Why it is valuable**

Lever documents public retrieval of published postings, pagination through `skip` and `limit`, individual posting retrieval, global and EU instances, workplace type, descriptions, locations, hosted URLs, apply URLs, and optional salary information. Internal and nonpublished jobs are not exposed, which is appropriate for the external-opportunity dataset.

**Adapter priority**

High.

---

### 4.3.3 Ashby public Job Postings API

**Typical signatures**

```text
jobs.ashbyhq.com
api.ashbyhq.com/posting-api
```

**Preferred source**

Ashby’s public Job Postings API.

**Why it is valuable**

The public interface returns currently published jobs and may include title, primary and secondary locations, department, team, workplace type, HTML and plain descriptions, publication time, employment type, URLs, and compensation.

**Adapter priority**

High, especially for fintechs, technology-oriented financial companies, and newer employers.

---

### 4.3.4 Workable

**Typical signatures**

```text
apply.workable.com
workable.com/boards
```

**Available sources**

- Employer-specific public career pages
- A global Workable XML feed
- Public pages and widgets

Workable’s published XML feed contains jobs that customers have elected to publish to Jobs by Workable. It includes jobs across Workable customers and requires downstream filtering by company. This feed is useful for discovery and supplementation, but it should not automatically be treated as a complete representation of every Workable employer’s externally published jobs.

**Adapter priority**

Medium.

**Important rule**

The global XML feed should normally be a supplemental source. The employer-specific career page or board must be reviewed separately.

---

### 4.3.5 Recruitee

**Typical signatures**

```text
*.recruitee.com
```

Recruitee’s developer documentation describes a Careers Site API for viewing jobs on a company’s career site and states that it does not require authorization. A more recent help-center article says that Careers Site API tokens may be required. Because the official materials are inconsistent, the adapter must probe each tenant rather than assume one universal authentication rule.

**Adapter strategy**

1. Test the hosted public career page.
2. Test documented public job endpoints without credentials.
3. Inspect page network traffic.
4. Use public API access where available.
5. Otherwise parse the public hosted site.

**Adapter priority**

Medium.

---

### 4.3.6 Personio

**Typical signatures**

```text
*.jobs.personio.de
```

Personio documents a company career-site XML feed with language-specific variants.

**Adapter strategy**

- Probe the XML feed.
- Select a primary language.
- Optionally retrieve alternate languages.
- Deduplicate language variants by source position ID.
- Preserve localized descriptions when useful.

**Adapter priority**

Medium to low for the first large-U.S.-financial-institution deployment, but inexpensive to support.

---

### 4.3.7 Teamtailor

**Typical signatures**

```text
*.teamtailor.com
custom career domains backed by Teamtailor
```

Teamtailor publishes a careers-site RSS feed containing job title, description, URL, remote status, global ID, locations, department, and role. Its default feed is limited to the first 100 jobs, and the official documentation describes `offset` and `per_page` pagination.

**Adapter priority**

Medium to low for the initial financial-institution universe, but a strong generic feed adapter.

---

### 4.3.8 SmartRecruiters

**Typical signatures**

```text
jobs.smartrecruiters.com
api.smartrecruiters.com
```

SmartRecruiters documents a Posting API with listing and detail endpoints, but its current official documentation says API-key authentication is required.

**Adapter strategy**

- Do not assume access to the documented authenticated API.
- Inspect the public employer job-search page.
- Observe any public network calls used by the hosted site.
- Use a credentialless endpoint only where direct probing confirms it is intentionally public.
- Otherwise use public page or browser-backed acquisition.

**Adapter priority**

Medium.

---

## 4.4 Enterprise ATS and candidate-experience platforms

These platforms are especially important for large financial institutions. They often require tenant-specific browser or network reconnaissance.

### Core enterprise ATS families

| Family | Typical detection patterns | Expected acquisition approach |
|---|---|---|
| Workday Recruiting | `*.myworkdayjobs.com`, branded Workday career hosts | Playwright reconnaissance followed by direct replay of public search/detail requests where possible |
| Oracle Recruiting Candidate Experience | Oracle Cloud Candidate Experience paths and branded domains | Inspect public career-site search and detail traffic |
| Oracle Taleo | `*.taleo.net`, `/careersection/` paths | Public HTML or browser-network extraction |
| SAP SuccessFactors Recruiting | SuccessFactors and branded Career Site Builder domains | Inspect job-search network calls and detail pages |
| iCIMS | `careers-*.icims.com`, `jobs-*.icims.com`, custom iCIMS career domains | Public portal HTML or network endpoint; partner XML feed is not assumed available |
| Jobvite | `jobs.jobvite.com` and branded Jobvite sites | Public listing/detail or network interface |
| Dayforce | Dayforce-hosted public career sites | Public listing/detail or network interface |
| ADP Recruiting | ADP recruiting and Workforce Now career portals | Public page and network inspection |
| Cornerstone | `*.csod.com` and branded career portals | Public page and network inspection |
| IBM Kenexa / BrassRing | BrassRing-hosted search and detail pages | Public page or browser-backed extraction |
| PageUp | PageUp People career domains and branded sites | Public listing/detail or network interface |
| UKG Recruiting | UKG/UltiPro career hosts | Public page and network inspection |
| Paylocity | Paylocity-hosted recruiting portals | Public listing/detail |
| Paycom | Paycom-hosted public application portals | Public listing/detail |
| Avature | Avature-hosted or custom-branded career experiences | Browser/network inspection and underlying apply-source detection |

Workday formally supports multiple external career sites and lets employers contextualize sites by geography and other criteria, so an employer may require more than one Workday board to achieve full external coverage.

Oracle similarly supports multiple external career sites that may be contextualized by location, organization, job category, job function, and other fields. Its public search can operate without search criteria, which is useful for complete enumeration. Oracle also warns that location-facet counts may count location occurrences rather than requisitions.

SAP Career Site Builder supports employer-controlled career sites and synchronized job-search experiences.

iCIMS offers a standardized XML job-board feed, but official documentation says consuming vendors must authenticate through OAuth and be approved. The system should therefore use the public employer portal rather than assume access to the partner feed.

---

## 4.5 Candidate-experience overlays and recruitment-marketing platforms

Large companies may place a candidate-experience layer in front of a separate ATS.

Important families include:

- Phenom
- Eightfold
- Beamery
- Radancy
- Symphony Talent / SmashFly
- Avature
- Custom employer recruitment-marketing sites

Phenom, Eightfold, Beamery, and Radancy all provide personalized or branded career-site experiences. Because personalization can change the visible result set, these sources require clean anonymous browser contexts, unfiltered searches, and evidence that the enumerated inventory is not profile-specific.

### Critical modeling rule

The system must not assume that the visible career-site platform is the system of record.

A source chain may look like:

```text
Employer corporate site
    ↓
Phenom or Eightfold career experience
    ↓
Public search endpoint
    ↓
Workday or Oracle application destination
```

The source record should therefore distinguish:

```text
presentation platform
enumeration interface
detail interface
application platform
canonical employer
```

Both the overlay’s source job ID and the underlying ATS or requisition ID should be preserved when available.

---

## 4.6 Long-tail ATS detection catalog

The source detector should recognize, or at least flag for review, host patterns associated with:

- BambooHR
- Pinpoint
- Comeet
- Breezy HR
- JazzHR / ApplyToJob
- JobScore
- ApplicantPro
- ClearCompany
- ApplicantStack
- TalentReef
- Fountain
- Workstream
- Crelate
- Zoho Recruit
- Manatal
- Hireology
- Rippling Recruiting
- Oracle NetSuite recruiting interfaces
- Proprietary employer-hosted systems

These need not all receive dedicated adapters. They should be detected and routed to:

1. A generic structured-data adapter
2. A generic static HTML adapter
3. A browser-network investigation queue
4. A manual-only classification

Implementation priority should depend on actual employer coverage.

---

## 4.7 Standards-based and generic sources

These source types should be supported independently of ATS vendor.

| Generic source | Purpose |
|---|---|
| `robots.txt` | Access policy and sitemap discovery |
| Sitemap index | Find subordinate job and career sitemaps |
| URL sitemap | Enumerate detail URLs and optional modification times |
| RSS | Incremental or complete job feed |
| Atom | Incremental or complete job feed |
| XML job feed | Structured complete inventory |
| JSON feed | Structured complete inventory |
| Schema.org `JobPosting` JSON-LD | Parse job-detail fields |
| Schema.org Microdata | Alternate detail-page structured fields |
| Canonical link | Resolve duplicate and canonical detail pages |
| Open Graph metadata | Supplemental title, employer, and canonical URLs |
| Static listing HTML | Enumerate detail URLs |
| Static detail HTML | Extract complete job description |
| Embedded script state | Discover tenant IDs, source IDs, initial result payloads |
| Public XHR or `fetch` endpoint | Structured listing and detail acquisition |
| Public GraphQL endpoint | Structured listing and detail acquisition |
| Browser DOM | Final extraction fallback |

`extruct` should be used to parse embedded structured metadata rather than writing regular expressions for JSON-LD or Microdata. It should receive the final redirected page URL as `base_url`, extract only needed syntaxes, preserve syntax-specific results, and leave entity reconciliation to a later stage.

---

## 4.8 External audit and miss-discovery sources

These sources should not normally form the production source of truth.

| Source | Use |
|---|---|
| LinkedIn | Manual review of recommendations and saved searches; import discovered jobs for official-source reconciliation |
| Google Job Search | Search-engine audit for missing employers or postings |
| Bing search | Supplemental site and posting discovery |
| Indeed | Manual or terms-compliant coverage audit |
| Glassdoor | Manual coverage audit |
| eFinancialCareers | Financial-services-specific miss discovery |
| Built In NYC | NYC technology and fintech opportunity discovery |
| ZipRecruiter | Supplemental discovery |
| Recruiter emails | Discovery of public and nonpublic opportunities |
| Employer job-alert emails | Official change and closure backstop |
| Search-engine `site:` queries | Find hidden regional boards and detail URLs |
| Common Crawl | Historical career-source and ATS-domain discovery |
| Wayback Machine | Historical source investigation and prior URL patterns |
| Professional associations | Employer-universe recall expansion |
| Referral and networking records | Nonpublic opportunity tracking outside the crawler |

LinkedIn explicitly prohibits third-party crawlers, bots, extensions, and software that scrape or automate activity on its website. LinkedIn should therefore remain a manual discovery and import channel.

Common Crawl provides an open web-crawl repository and is useful for historical domain and source discovery, but it is not sufficiently current to serve as the production job source.

The Wayback Machine can help investigate prior career URLs and source migrations, but individual captures may be incomplete and should not determine current job status.

---

# 5. Source priority for the initial target market

For large financial institutions with NYC hiring, implementation priority should be:

## Priority 1 — Enterprise career sources

- Workday
- Oracle Recruiting Candidate Experience
- Oracle Taleo
- iCIMS
- SAP SuccessFactors
- Eightfold
- Phenom
- Avature
- Proprietary large-employer sites

These are likely to determine coverage of the largest target employers.

## Priority 2 — Uniform public job interfaces

- Greenhouse
- Lever
- Ashby
- SmartRecruiters where publicly accessible
- Recruitee
- Workable
- Personio
- Teamtailor

These are less likely to dominate large-bank coverage but are comparatively easy to implement and will increase coverage of asset managers, fintechs, financial-software companies, and adjacent institutions.

## Priority 3 — Generic source handling

- Sitemap plus detail pages
- JSON-LD and Microdata
- Static HTML
- RSS and Atom
- Generic XML
- Embedded initial-state JSON

## Priority 4 — External audits

- LinkedIn manual imports
- Google and Bing
- Indeed, Glassdoor, eFinancialCareers, and Built In NYC
- Official email alerts
- Common Crawl and Wayback source investigation

---

# 6. Logical system components

The detailed implementation should contain the following logical capabilities, regardless of how code is organized.

```text
Employer Universe Manager
Employment-Brand Resolver
Career-Source Registry
Source Discovery Engine
Acquisition Gateway
Robots and Host Policy Manager
Adapter Registry
Raw Artifact Store
Source Parser Layer
Canonical Normalizer
Job Identity Resolver
Lifecycle Reconciler
Scheduler and Source Lease Manager
Source Health Evaluator
Full-Text Search and Geographic Filter
Manual Annotation Store
Coverage Auditor
Export Service
```

---

# 7. Canonical source contracts

The exact use of dataclasses, Pydantic models, SQLAlchemy models, or equivalent constructs may be decided during implementation. The conceptual contracts and invariants below should remain stable.

---

## 7.1 Employment brand

```python
EmploymentBrand:
    employer_id
    canonical_name
    aliases
    parent_employer_id
    legal_entity_ids
    official_domains
    industry_verticals
    active_status
    inclusion_reason
    universe_sources
    manual_review_state
```

### Invariants

- One employment brand may map to many legal entities.
- One legal entity may not silently map to two brands without an explicit relationship.
- A shared career source should not be crawled repeatedly for every subsidiary.
- Employer aliases must remain searchable.
- Manual overrides must retain their provenance.

---

## 7.2 Career source

```python
CareerSource:
    source_id
    employer_id

    careers_landing_url
    board_url
    canonical_host
    registrable_domain

    source_family
    presentation_platform
    application_platform

    tenant_identifier
    site_identifier
    board_scope
    regions
    locales
    employment_types

    acquisition_mode
    adapter_name
    adapter_config

    robots_policy
    check_interval
    source_status

    first_verified_at
    last_verified_at
    last_complete_run_at
    last_successful_run_at
```

### `board_scope`

A source must record what it represents:

```text
global_external
us_external
regional_external
subsidiary_external
experienced_hires
early_careers
contractor
mixed
unknown
```

### Invariants

- A source is not “resolved” merely because a URL was found.
- It must be associated with the employer by official-link or equivalent high-confidence evidence.
- Source family and presentation platform are separate from application platform.
- Region and locale scope must remain explicit.
- An unresolved scope blocks claims of employer-level completeness.

---

## 7.3 Acquisition gateway

Adapters should receive a shared gateway rather than directly creating network clients.

```python
AcquisitionGateway:
    fetch_http(request) -> HttpArtifact
    open_browser_session(source) -> BrowserSession
    check_robots(url, user_agent) -> RobotsDecision
    resolve_url(base, relative) -> NormalizedUrl
    persist_artifact(artifact) -> ArtifactReference
```

The gateway owns:

- HTTPX2 client lifecycle
- Connection pooling
- Timeouts
- Retry policy
- Redirect validation
- Per-host limits
- User agent
- TLS policy
- Response-size limits
- Robots evaluation
- Browser lifecycle
- Snapshot persistence
- Request and response logging
- Sensitive-header redaction

Adapters must not bypass it.

---

## 7.4 Raw artifact

```python
RawArtifact:
    artifact_id
    run_id
    source_id

    artifact_type
    request_method
    request_url
    request_body_hash
    safe_request_headers

    response_status
    response_headers
    final_url
    redirect_chain
    content_type

    raw_content_reference
    raw_content_hash
    decoded_content_hash

    retrieved_at
    acquisition_method
    browser_context_metadata
```

### Artifact types

```text
robots
sitemap
listing_html
listing_json
listing_xml
listing_rss
detail_html
detail_json
detail_xml
rendered_html
embedded_metadata
har
browser_trace
script_config
error_page
```

### Invariants

- Raw artifacts are immutable.
- Parsed DOM serialization must never replace the original bytes.
- Identical content may be content-addressed to avoid duplicate storage.
- Sensitive request headers and browser storage state must not be persisted.

selectolax repairs malformed HTML according to HTML5 parsing rules, so parsed serialization is not equivalent to original bytes. The raw response must be retained independently.

---

## 7.5 Source job reference

This is the minimum identity discovered during enumeration.

```python
SourceJobReference:
    source_id
    source_job_id
    requisition_id

    listing_url
    detail_url
    apply_url

    raw_title
    raw_locations
    raw_department
    raw_posted_at
    raw_updated_at

    listing_position
    partition_key
    source_payload_reference
```

### Invariants

- `source_job_id` must come from the source where possible.
- Title plus employer is not a sufficient source ID.
- Generated IDs must be clearly marked and based on stable source facts.
- Multiple locations must not be collapsed into one pseudo-location.

---

## 7.6 Enumeration page

```python
EnumerationPage:
    source_id
    run_id
    page_ordinal

    request_or_cursor
    returned_references
    returned_count

    advertised_total
    next_cursor
    next_url
    terminal_signal

    raw_artifact_id
    warnings
```

### Terminal signals

```text
explicit_end
empty_page
short_page
missing_next_link
null_next_cursor
advertised_total_reached
all_partitions_complete
source_declared_empty
unknown
```

---

## 7.7 Enumeration result

```python
EnumerationResult:
    source_id
    run_id

    unique_job_references
    page_count
    partition_count

    advertised_total
    enumerated_total
    duplicate_reference_count

    terminal_condition
    failed_pages
    skipped_partitions
    warnings

    scope_verified
    enumeration_status
```

---

## 7.8 Parsed source posting

```python
ParsedSourcePosting:
    source_id
    source_job_id
    requisition_id

    employer_name
    title
    department
    job_family
    employment_type

    date_posted
    date_modified
    valid_through

    locations
    remote_type
    remote_scope

    description_html
    description_text
    description_sections

    compensation_raw
    compensation_components

    detail_url
    canonical_url
    apply_url

    source_status
    source_language

    source_field_provenance
    parser_version
```

The parser should preserve both:

```text
raw source value
normalized value
```

for fields where normalization is nontrivial.

---

## 7.9 Completeness assessment

```python
CompletenessAssessment:
    source_id
    run_id

    scope_status
    pagination_status
    count_status
    identity_status
    detail_status
    parser_status
    anomaly_status

    enumeration_complete
    content_refresh_complete
    lifecycle_reconciliation_allowed
    closure_allowed

    completeness_class
    evidence
    blocking_reasons
```

---

## 7.10 Adapter protocol

```python
class CareerSourceAdapter(Protocol):

    source_family: str

    async def probe(
        self,
        source: CareerSource,
        acquisition: AcquisitionGateway,
    ) -> ProbeResult:
        ...

    async def enumerate_jobs(
        self,
        source: CareerSource,
        acquisition: AcquisitionGateway,
        prior_run: PriorRunContext | None,
    ) -> EnumerationResult:
        ...

    async def fetch_job_detail(
        self,
        source: CareerSource,
        reference: SourceJobReference,
        acquisition: AcquisitionGateway,
    ) -> DetailAcquisitionResult:
        ...

    def parse_job(
        self,
        source: CareerSource,
        reference: SourceJobReference,
        artifacts: Sequence[RawArtifact],
    ) -> ParsedSourcePosting:
        ...

    def assess_completeness(
        self,
        source: CareerSource,
        run_evidence: CrawlRunEvidence,
    ) -> CompletenessAssessment:
        ...
```

### Capability flags

Each adapter should declare:

```text
listing_contains_full_description
detail_fetch_required
supports_incremental_fetch
supports_conditional_requests
supports_multiple_locales
supports_multiple_boards
supports_advertised_count
supports_explicit_closed_status
requires_browser_discovery
requires_browser_production
supports_anonymous_access
```

---

# 8. Acquisition state machines

---

## 8.1 Career-source lifecycle

```text
UNRESOLVED
    ↓ candidate URL discovered
CANDIDATE
    ↓ employer association verified
VERIFIED
    ↓ first complete successful crawl
HEALTHY
    ↓ recoverable anomaly or partial run
DEGRADED
    ↓ later complete run
HEALTHY

CANDIDATE / VERIFIED / HEALTHY / DEGRADED
    ├── access unsuitable → BLOCKED
    ├── manual source selected → MANUAL_ONLY
    ├── source replaced → RETIRED
    └── source no longer belongs to employer → REJECTED
```

### Source-state rules

| State | Meaning |
|---|---|
| `UNRESOLVED` | No verified public career source |
| `CANDIDATE` | Possible source found but not yet validated |
| `VERIFIED` | Employer relationship and source interface established |
| `HEALTHY` | Recent complete crawl succeeded |
| `DEGRADED` | Partial, anomalous, or recently failed crawl |
| `FAILED` | Repeated terminal acquisition failure |
| `BLOCKED` | Automation is inappropriate or technically prevented |
| `MANUAL_ONLY` | Source monitored through an approved manual process |
| `RETIRED` | Replaced, discontinued, or superseded source |
| `REJECTED` | Candidate source was not actually associated with the employer |

---

## 8.2 Crawl-run state machine

```text
PLANNED
   ↓ lease acquired
LEASED
   ↓ policy and robots checked
PRECHECKED
   ↓
ENUMERATING
   ↓ all listing pages processed
ENUMERATED
   ↓ required details fetched
DETAIL_FETCH
   ↓
PARSING
   ↓
VALIDATING
   ↓
RECONCILING
   ↓
COMPLETE
```

Alternative terminal paths:

```text
PLANNED          → CANCELLED
LEASED           → ABANDONED
PRECHECKED       → BLOCKED
ENUMERATING      → PARTIAL
DETAIL_FETCH     → CONTENT_PARTIAL
PARSING          → PARSER_FAILED
VALIDATING       → INDETERMINATE
Any active state → FAILED
```

### Key distinction

A run may have:

```text
complete enumeration
+
incomplete detail refresh
```

In that case:

- Active source IDs may still be reconciled.
- Existing jobs absent from the complete enumeration may move toward closure.
- Failed job details must retain their previous parsed content.
- The run must not be represented as fully content-complete.

---

## 8.3 HTTP request state machine

```text
QUEUED
  ↓
IN_FLIGHT
  ├── successful response → SUCCEEDED
  ├── retryable failure → RETRY_WAIT
  │                           ↓
  │                       IN_FLIGHT
  ├── robots denied → POLICY_DENIED
  ├── nonretryable response → TERMINAL_FAILURE
  └── retry budget exhausted → TERMINAL_FAILURE
```

### Retryable candidates

```text
Connect timeout
DNS or connection failure
Connection reset
Read timeout
HTTP 408
HTTP 429
Selected 5xx responses
```

### Normally nonretryable

```text
Invalid URL
Robots prohibition
Unsupported scheme
Authentication requirement
Ordinary 4xx response
Unsupported content type
Response exceeding allowed size
Parser contract failure
```

HTTPX2 does not provide one universal retry policy. The system should explicitly define method safety, exception classes, status codes, backoff, jitter, `Retry-After`, and total attempt/time budgets. Its native `MockTransport` should be used for deterministic transport tests. 
---

## 8.4 Browser-escalation state machine

```text
HTTP_PROBE
    ↓ insufficient
STATIC_HTML_INSPECTION
    ↓ insufficient
SCRIPT_AND_CONFIG_INSPECTION
    ↓ insufficient
PLAYWRIGHT_NETWORK_OBSERVATION
    ↓ candidate public endpoint
HTTP_REPLAY_TEST
    ├── complete replay succeeds → PROMOTED_TO_HTTP
    ├── browser state required → BROWSER_NETWORK_ADAPTER
    └── no usable endpoint → DOM_EXTRACTION_TEST
                                  ├── viable → BROWSER_DOM_ADAPTER
                                  ├── blocked → BLOCKED
                                  └── unsuitable → MANUAL_ONLY
```

### Promotion rule

A browser-discovered endpoint should be promoted to HTTPX2 only after confirming from a clean process that:

- The same active job IDs are returned.
- The same board scope is represented.
- Pagination behaves identically.
- No browser credential or private token is required.
- Required request values are public tenant/site identifiers rather than secrets.
- The request can be made within the source’s access policy.

---

## 8.5 Job lifecycle state machine

```text
NOT_PREVIOUSLY_SEEN
        ↓ observed in complete or partial run
ACTIVE

ACTIVE
    ├── observed again → ACTIVE
    ├── absent from complete run once → POSSIBLY_CLOSED
    ├── explicit source closure → CLOSED
    └── source run incomplete → ACTIVE / UNKNOWN, never closed

POSSIBLY_CLOSED
    ├── observed again → ACTIVE
    ├── absent from required complete runs → CLOSED
    └── incomplete run → POSSIBLY_CLOSED unchanged

CLOSED
    ├── same source ID reappears → REOPENED → ACTIVE
    ├── linked new source ID → REPOSTED
    └── replacement job established → SUPERSEDED
```

The required number of complete absent observations should be configurable by source family. Two is a reasonable initial default, but the adapter may justify another value.

---

# 9. Source-family adapter specifications

Every dedicated adapter should have a source-family specification containing:

```text
Detection signatures
Tenant/site identifier discovery
Board-scope rules
Listing interface
Pagination or cursor rules
Detail interface
Source identifiers
Field mapping
Location behavior
Date behavior
Description behavior
Compensation behavior
Completeness proof
Closure behavior
Known anomalies
Required fixtures
Shared contract tests
Live smoke test
```

---

## 9.1 Greenhouse adapter

### Detection

Inspect:

```text
Greenhouse board host
Greenhouse scripts or iframe
board token in official careers link
Greenhouse API hostname
```

### Enumeration

Use the board jobs operation with full content enabled.

Expected evidence:

```text
board token
response meta total
unique post IDs
internal job IDs when present
all jobs returned or all pagination links followed
```

### Detail policy

The list response may contain enough content for normal ingestion.

Fetch individual details only when needed for:

- Pay transparency fields not present in the list
- Additional posting metadata
- Source-specific anomalies

Do not retrieve application-form questions; they do not support the job-discovery purpose.

### Identity

Primary:

```text
board token + Greenhouse post ID
```

Secondary:

```text
internal job ID
requisition ID
absolute URL
```

### Completeness

`EXACT` when:

- All pages or the full response were processed.
- Advertised total equals unique post count.
- No page failed.
- Board scope is verified.

### Required fixtures

- Empty board
- One job
- Multi-page board if applicable
- Multi-office job
- Job with no requisition ID
- Job with metadata
- Job whose description contains encoded HTML
- Job removed between runs

---

## 9.2 Lever adapter

### Detection

Identify:

```text
global or EU Lever host
site slug
hosted board link
```

### Enumeration

Use JSON mode without title, location, team, or commitment filters.

Iterate:

```text
skip = 0
limit = configured_page_size
skip += number_returned
```

Stop when:

- Returned count is less than limit, or
- An explicitly verified terminal response is reached

Detect repeated pages by hashing the ordered source IDs.

### Detail policy

The list response generally contains full posting content. Individual detail calls are optional and should not be made routinely unless a field is missing or a source inconsistency is being investigated.

### Identity

```text
site slug + posting ID
```

### Completeness

Normally `TERMINAL`, because pagination termination rather than a separately advertised total establishes completeness.

### Required fixtures

- Global site
- EU site
- Exact full page followed by empty or short page
- Duplicate page caused by bad `skip`
- Remote/hybrid job
- Job with salary fields
- Job with empty optional description sections

---

## 9.3 Ashby adapter

### Detection

Identify:

```text
Ashby hosted board
job-board name
public posting endpoint
```

### Enumeration

Use the public board endpoint and request compensation only if the field is useful and the incremental payload remains reasonable.

### Identity

Prefer:

1. Explicit public posting ID where present
2. Stable Ashby job URL path
3. Stable apply URL path

Do not generate identity from title and location unless absolutely necessary.

### Completeness

If the board endpoint returns the complete published jobs collection in one response:

```text
terminal condition = single_complete_collection
```

Repeated observations should confirm that there is no hidden pagination.

### Required fixtures

- Empty board
- Primary and secondary locations
- Remote job
- Job with compensation
- Job without compensation
- Multiple teams and departments
- Job removed between runs

---

## 9.4 Workable adapter

### Source modes

```text
employer_specific_board
global_workable_xml
public_detail_page
```

### Global-feed policy

The global XML feed should:

- Be streamed with lxml `iterparse`.
- Be considered supplemental.
- Retain company name and reference number.
- Link each result to the employer-specific source where possible.

### Completeness

The global feed can be complete as a file while still incomplete for an employer because participation depends on publication to Jobs by Workable.

Therefore distinguish:

```text
feed_file_complete
employer_inventory_authoritative
```

The latter should normally be false until the employer-specific board is verified.

---

## 9.5 Recruitee adapter

### Probe sequence

1. Retrieve public career page.
2. Test unauthenticated Careers Site API.
3. Inspect scripts and public network requests.
4. Determine whether tenant-specific credentials are required.
5. Fall back to public HTML or browser network.

### Completeness

Use:

- API collection terminal behavior when credentialless
- Public page count and pagination otherwise

### Implementation caution

Because official documentation currently conflicts on authentication, the adapter’s probe result must be persisted. A future documentation change should not silently alter existing source configuration.

---

## 9.6 Personio adapter

### Enumeration

Retrieve the configured XML language feed.

Optional alternate-language flow:

```text
primary language feed
    ↓
additional language feeds
    ↓
join on Personio position ID
```

### Completeness

A feed is complete when:

- XML is well formed.
- The root closes successfully.
- Every item is parsed.
- No stream error occurred.
- The tenant and language are verified.

### XML policy

Use lxml with:

```python
resolve_entities=False
load_dtd=False
no_network=True
huge_tree=False
```

Use `iterparse` and clear completed elements for large feeds. The lxml reference recommends this posture for untrusted XML and large streaming documents.

---

## 9.7 Teamtailor adapter

### Enumeration

Use the RSS feed and paginate with `offset` and `per_page`.

Stop when:

- Returned item count is less than `per_page`, or
- A verified empty terminal page is returned

### Identity

Use the feed’s global ID.

### Completeness

Detect:

- Repeated offset pages
- Duplicate global IDs
- Silent first-100 truncation
- Missing descriptions
- Custom-domain redirects

---

## 9.8 SmartRecruiters adapter

### Probe sequence

1. Identify company or tenant ID from public page.
2. Test public hosted search behavior.
3. Test whether the tenant’s public posting endpoint is credentialless.
4. If an API key is required, do not use the authenticated API.
5. Observe public network traffic.
6. Use public HTML or browser-backed extraction if required.

### Completeness

Do not treat the public global SmartRecruiters search portal as the employer’s authoritative board unless it reconciles with the employer-linked source.

---

## 9.9 Generic `JobPosting` adapter

### Purpose

Extract structured fields from a known job-detail URL.

### It must not own enumeration

JSON-LD on one job page cannot prove that all jobs were found.

It should be combined with:

```text
sitemap enumeration
listing-page enumeration
feed enumeration
source-specific endpoint enumeration
```

### Extraction

Use extruct with:

```text
syntaxes = JSON-LD and Microdata
base_url = final redirected page URL
uniform = false
```

Normalize:

- Single object
- Array
- `@graph`
- Multiple script blocks
- Duplicate entities
- Malformed block next to valid block

Preserve:

```text
syntax
block order
raw object
source page
resolved URLs
```

---

## 9.10 Generic sitemap adapter

### Discovery

Sources:

- `robots.txt`
- Standard sitemap locations
- HTML sitemap links
- Sitemap indexes

### Parsing

Use lxml and namespace-aware XML handling.

Use streaming parse for large indexes.

### URL classification

Classify URLs by:

```text
job detail
job listing
career landing
irrelevant
unknown
```

Classification may use:

- Path patterns
- Embedded metadata
- Page title
- Canonical links
- Source history

### Completeness

A sitemap is authoritative only when:

- It is officially associated with the employer.
- Job URLs appear to be systematically included.
- Reconciliation against the visible board shows no unexplained gap.

Otherwise it is a discovery source rather than a completeness source.

---

## 9.11 Generic static HTML adapter

### Configurable recipe

```text
listing container
job-link selector
advertised-count selector
next-page selector
detail title selector
location selector
description selector
requisition selector
date selector
apply-link selector
```

### Parser

Use `LexborHTMLParser`.

Use strict unique matching for fields where duplication signals source drift.

Use non-strict matching only when first-match behavior is intended.

### Completeness

Evidence may include:

- Advertised count reconciled
- Every pagination link traversed
- Terminal next link absent
- No repeated page fingerprints
- Unique canonical detail URLs
- All required detail pages fetched

### Drift detection

Store a source-structure fingerprint based on:

- Key selector match counts
- Script hostnames
- Form actions
- Representative tag hierarchy
- Structured-data presence

A material fingerprint change should degrade the source before jobs are closed.

---

## 9.12 Workday adapter

### Discovery

Use the official employer-linked Workday site.

Record:

```text
Workday host
tenant
site
locale
external board scope
```

An employer may maintain several external sites.

### Browser reconnaissance

Observe:

- Initial search request
- Subsequent page request
- Job-detail request
- Facet requests
- Locale/site configuration
- Total-result field
- Page-size cap

### HTTP promotion

Replay the public search request through HTTPX2.

The production adapter should prefer HTTPX2 when:

- Anonymous public requests work.
- Required tenant and site identifiers are nonsecret.
- Counts and IDs match the browser.
- Pagination can be reproduced.

### Enumeration

Use a blind, unfiltered search if supported.

Validate:

- Total count
- Offset or page arithmetic
- Unique job IDs
- Last page
- No result cap

If the endpoint caps the total query, use partitioned enumeration.

### Partition candidates

```text
country
region
job category
business unit
stable location facet
```

A partition is acceptable only if the union can be shown to cover the complete public board.

### Detail

Prefer structured detail payload.

Fall back to job-detail HTML.

### Completeness

`EXACT` when:

```text
unique job IDs = official total count
all pages retrieved
board scope verified
no cap detected
```

---

## 9.13 Oracle Recruiting Candidate Experience adapter

### Discovery

Record:

```text
career site code
locale
site context
organization/location restrictions
candidate-experience version
```

### Enumeration

Use blind search when available.

Do not sum location-facet counts as a requisition total; Oracle documents that location counts may represent occurrences rather than distinct requisitions.

### Multiple-site handling

An Oracle tenant may expose several context-restricted external sites.

Employer completeness therefore requires:

```text
union of all verified external career sites
-
confirmed duplicates
```

### Identity

Prefer:

- Oracle requisition identifier
- Public job identifier
- Stable detail URL

### Taleo

Treat Taleo as a separate source family because:

- URL patterns differ.
- Pagination and session behavior differ.
- Detail-page representations differ.
- It may coexist with newer Oracle Candidate Experience.

---

## 9.14 SAP SuccessFactors adapter

### Discovery

Determine:

- Branded site
- Career Site Builder identity
- Locale
- Site-specific search configuration
- Public search endpoint
- Detail endpoint

### Enumeration

Use anonymous, unfiltered search.

Check:

- Count semantics
- Pagination
- Language variants
- Real-time sync behavior
- Duplicate detail URLs
- Job IDs and requisition IDs

### Browser fallback

Promote public network calls to HTTPX2 where possible.

---

## 9.15 iCIMS adapter

### Source policy

Do not use the partner XML feed unless credentials have been legitimately provided for this project, which is outside the current design.

### Public acquisition

Use:

- Public career portal
- Listing pagination
- Detail pages
- Browser-observed public requests

### Identity

Typical useful fields:

```text
portal job ID
requisition ID
detail URL
application URL
```

### Completeness

Validate:

- Result count
- Search pagination
- Portal scope
- Regional portals
- Hidden caps
- Duplicate result pages

---

## 9.16 Phenom, Eightfold, Beamery, Radancy, and similar overlays

### Clean-context requirement

Every enumeration should begin with:

```text
new anonymous browser context
no uploaded resume
no candidate profile
no persisted cookies
no prior search history
fixed locale and timezone
```

### Personalization detection

Compare:

- Clean browser run A
- Clean browser run B
- Direct endpoint replay
- Optional distinct locale context

If the same blind search produces materially different job IDs, the source should be treated as personalized or unstable until explained.

### Underlying ATS relationship

Store:

```text
overlay source job ID
overlay detail URL
apply destination
underlying ATS family
underlying requisition ID
```

### Enumeration

Prefer an explicit “all jobs” or blind-search path.

Do not rely on:

- Recommended jobs
- Resume-matched results
- Personalized home-page cards
- Default location inferred from IP

---

# 10. Completeness logic

Completeness should be evaluated by dimensions rather than one opaque percentage.

---

## 10.1 Completeness classes

```text
EXACT
TERMINAL
PARTITIONED
PARTIAL
INDETERMINATE
FAILED
BLOCKED
MANUAL
```

### `EXACT`

All of the following are true:

- Source scope is known.
- Advertised total is available.
- Unique enumerated job IDs equal the advertised total.
- Pagination is exhausted.
- No listing page failed.
- No truncation signal exists.

### `TERMINAL`

No trustworthy total exists, but:

- Pagination has an explicit, repeatable terminal state.
- No page failed.
- No cursor repeated.
- The source scope is known.
- A repeated run confirms the behavior.

### `PARTITIONED`

The source cannot be enumerated through one unfiltered query, but:

- Every partition is explicit.
- Every partition is completely enumerated.
- The union is deduplicated by stable source ID.
- Partition coverage is demonstrated.
- No “other” or unknown category is omitted.

### `PARTIAL`

Some results were retrieved, but one or more pages, partitions, or identity steps failed.

### `INDETERMINATE`

The run completed technically, but the system cannot establish whether the result set represents all public jobs.

### `FAILED`

No usable enumeration was produced.

### `BLOCKED`

Access is unsuitable or intentionally blocked.

### `MANUAL`

Coverage depends on a human or email workflow.

---

## 10.2 Completeness dimensions

| Dimension | Required question |
|---|---|
| Source scope | Which employer, subsidiary, region, language, and candidate population does the board cover? |
| Pagination | Were all pages, offsets, or cursors traversed? |
| Count | Does the enumerated unique count reconcile with an authoritative total? |
| Identity | Does every posting have a stable source identity? |
| Listing coverage | Were all active source references discovered? |
| Detail coverage | Were all required detail records successfully fetched? |
| Parsing | Were all records parsed without fatal loss? |
| Personalization | Was the result set anonymous and nonpersonalized? |
| Truncation | Is there evidence of a result cap? |
| Stability | Does a repeated observation behave consistently? |

---

## 10.3 Enumeration versus content completeness

These must be separate.

```text
enumeration_complete:
  all active source job references are known

content_refresh_complete:
  all job details required for the run were successfully refreshed
```

Lifecycle reconciliation may proceed when enumeration is complete even if some detail refreshes failed.

Content updates for failed details must not replace prior good content with nulls.

---

## 10.4 Pagination safeguards

Every paginated adapter must detect:

- Repeated cursor
- Repeated page URL
- Repeated ordered job-ID fingerprint
- Nonadvancing offset
- Empty middle page
- Page count exceeding configured safety bound
- Advertised total reached prematurely
- Advertised total exceeded
- Duplicates across pages
- Sorting instability
- New jobs appearing during a long run

Where source churn during enumeration is likely, rerun the first page at the end and determine whether material changes occurred during the crawl.

---

## 10.5 Valid zero-job result

A zero-job result is valid only when:

- Source identity and scope are verified.
- HTTP or browser acquisition succeeded.
- No login, challenge, or error page was returned.
- Pagination terminated normally.
- Advertised total is zero or the source explicitly returns a complete empty collection.
- Parser health checks passed.
- Page/source fingerprints remain plausible.

If the prior complete run contained a material number of jobs, a zero result should normally require:

- A second confirmation run, or
- Manual source review

before mass closure.

---

## 10.6 Hard result caps

If a source returns at most \(N\) records and no single query can exceed that cap:

1. Identify a stable partition dimension.
2. Enumerate all partition values.
3. Retrieve each partition fully.
4. Deduplicate by source job ID.
5. Verify that every job belongs to at least one partition.
6. Include an unknown or uncategorized partition where possible.
7. Retain overlap and partition evidence.

Do not partition only by NYC because that would prevent reuse of the source inventory and make employer-level coverage impossible to audit.

---

## 10.7 Multiple career boards

Employer-level coverage is:

\[
\text{Employer Inventory}
=
\bigcup_{s \in \text{verified external sources}} \text{Jobs}(s)
\]

The system must account for:

- Regional boards
- Subsidiary boards
- Separate experienced and early-career sites
- Separate contractor sites
- Recently acquired companies
- Alternate languages
- Old and new ATSs operating during migration

A duplicate job appearing in two boards should preserve both source observations.

---

## 10.8 Closure eligibility

A source run may close or advance jobs toward closure only when:

```text
source scope verified
AND enumeration complete
AND source not anomalous
AND source identity unchanged
AND run not personalized
```

Detail-fetch completeness is not necessarily required if the complete listing itself is the authority for active IDs.

---

## 10.9 Source anomaly rules

Flag at least:

```text
Job count falls to zero
Job count falls by more than configured proportion
Job count increases implausibly
Tenant or company branding changes
Expected source identifier disappears
HTML selector counts change materially
JSON response shape changes
Known active sentinel job disappears while external sources still show it
Pagination terminates much earlier than recent history
All locations become null
All descriptions become empty
All source IDs change
```

An anomalous but technically complete run should be marked `INDETERMINATE` until reviewed or confirmed.

---

# 11. Parsing and normalization principles

## 11.1 HTML

Use selectolax Lexbor for ordinary HTML.

Leverage:

- `css`
- `css_first`
- Strict uniqueness where expected
- Efficient tag lookup
- Built-in text extraction
- Fragment parsing
- Parser-local node navigation

Parse each document once, extract primitives, and release the tree. Parser nodes must not be retained beyond the parser’s lifetime.

## 11.2 XML

Use lxml for:

- Sitemaps
- XML job feeds
- RSS or Atom where feedparser is not used
- Namespaces
- XPath
- Streaming large feeds

Use compiled XPath for repeated expressions and namespace URIs rather than document-specific prefixes.

## 11.3 Structured metadata

Use extruct for:

- JSON-LD
- Microdata
- Open Graph only when useful

Do not use `uniform` mode as the canonical source representation.

## 11.4 URLs and domains

Use:

```text
HTTPX2 URL or urllib parsing
IDNA normalization
ipaddress classification
tldextract
```

`tldextract` should use a pinned PSL policy and should not be treated as URL validation or SSRF protection. Store the full hostname, public-suffix boundary, registry suffix, and PSL policy separately.

## 11.5 Locations

Preserve:

```text
raw location
normalized city
normalized state or region
country
postal code
remote type
remote scope
normalization confidence
```

Never replace the source location with only a normalized value.

## 11.6 Descriptions

Preserve:

```text
raw HTML
normalized full text
ordered sections
ordered bullets
section headings
```

Deterministic heading normalization may map:

```text
What You’ll Do          → responsibilities
Responsibilities        → responsibilities
Required Qualifications → qualifications
Preferred Experience    → preferred_qualifications
Compensation            → compensation
Benefits                → benefits
About Us                → company
```

Unknown sections should remain available.

---

# 12. Fixture-backed test architecture

Testing should be fixture-first. Live websites are unsuitable as the only test environment.

---

## 12.1 Fixture classes

### Listing fixtures

- Empty collection
- One item
- Exact full page
- Short terminal page
- Multiple pages
- Cursor pagination
- Offset pagination
- Repeated page
- Duplicate IDs
- Advertised-count mismatch
- Truncated response
- Unexpected field
- Missing optional field
- Source error disguised as HTTP 200

### Detail fixtures

- Complete job
- Multiple locations
- Remote job
- Hybrid job
- Compensation
- No compensation
- Missing requisition ID
- Malformed description HTML
- Description with embedded scripts
- Job already closed
- Redirected detail page
- Application host different from detail host

### XML and feed fixtures

- Valid XML feed
- Namespace-heavy feed
- Large feed
- Empty feed
- Truncated XML
- Invalid character
- External entity attempt
- Multi-language duplicates
- RSS default-100 truncation
- Relative URL

### Structured-metadata fixtures

- One JSON-LD object
- Array
- `@graph`
- Multiple script blocks
- Duplicate `JobPosting`
- Malformed block next to valid block
- Microdata only
- Relative URLs
- Server HTML and rendered HTML disagreement

### Browser fixtures

- HAR with initial search
- HAR with next page
- HAR with terminal page
- HAR with job detail
- Rendered DOM fallback
- Service-worker interference
- Personalized result set
- Consent banner
- Infinite scroll
- Load-more button
- GraphQL request
- Failed browser navigation

---

## 12.2 Fixture artifacts

Each fixture should retain:

```text
source family
source version or observation date
request method and URL
safe request headers
response status and headers
raw bytes
content hash
expected parser output
expected completeness outcome
redaction status
```

Screenshots are supporting evidence only. They must not serve as parser fixtures. Playwright traces should be retained for difficult failures, while HAR and extracted network payloads should drive deterministic tests. 

---

## 12.3 Shared adapter contract tests

Every adapter must pass the following common behaviors.

### Detection tests

- Positive source-family detection
- Similar but unrelated source rejected
- Custom domain detected through scripts or network
- Ambiguous source returns review state rather than false certainty

### Enumeration tests

- Every page traversed
- Cursor advances
- Duplicate IDs deduplicated but reported
- Page loop detected
- Exact count reconciled
- Empty valid source recognized
- Empty error page rejected
- Partial result marked partial

### Parsing tests

- Required fields preserved
- Optional fields may be null
- Multiple locations preserved
- Raw values retained
- Description HTML and text produced
- Unexpected fields do not crash parser
- Fatal parse errors are observable

### Lifecycle tests

- Complete absence advances toward closure
- Partial absence does not
- Failed run does not
- Reappearing job reopens
- Duplicate observation does not create duplicate canonical job
- New source ID with same requisition is linked as probable repost

### Idempotency tests

Running the same fixture twice must not:

- Duplicate observations
- Duplicate canonical jobs
- Advance closure
- Change first-seen time
- Produce different normalized content

---

## 12.4 HTTP acquisition tests

Use HTTPX2 `MockTransport` for:

- Redirect chains
- 304 responses
- 408
- 429 with `Retry-After`
- 500 and 503
- Read timeout
- Connection failure
- Oversized body
- Unexpected content type
- Brotli or Zstandard decoding where enabled
- Private-address redirect rejection
- Duplicate query parameters
- Conditional request headers
- Stream closure

HTTPX2 clients should be long lived, use bounded pools and differentiated timeouts, and catch specific transport and status exceptions rather than treating every failure as a generic network error.

---

## 12.5 Browser tests

Use Playwright with:

- One reusable browser process
- Isolated browser contexts
- Fixed viewport, locale, and timezone
- Service workers blocked where deterministic interception requires it
- State-based waits rather than sleeps
- Network request and response capture
- HAR replay
- Trace retained on failure

Playwright’s BrowserContext is the normal state-isolation boundary, while locators and network events should be preferred over persistent element handles and arbitrary waits. 

---

## 12.6 State-machine property tests

Property-based tests should verify invariants such as:

```text
An incomplete run can never close a job.
A failed run can never reduce the active inventory.
Reordering listing pages does not alter the canonical inventory.
Processing a run twice is idempotent.
A repeated cursor always terminates with a pagination error.
A job observed after closure becomes reopened or active.
No source ID maps to two canonical jobs within the same source.
A raw artifact is never mutated after persistence.
```

---

## 12.7 Live smoke tests

Live tests should be:

- Explicitly enabled
- Low request volume
- Read only
- Independent of exact job counts
- Focused on source identity and response compatibility
- Separate from ordinary unit and integration tests

A smoke test may assert:

```text
Source responds
Expected tenant is present
At least zero valid jobs can be parsed
Pagination contract remains recognizable
No authentication challenge appeared
```

It should not assert that an employer always has exactly a particular number of jobs.

---

# 13. Key library features to leverage

## 13.1 HTTPX2

Use:

- Long-lived `AsyncClient`
- Connection limits
- Separate connect/read/write/pool timeouts
- Streaming response bodies
- Conditional request headers
- Redirect history
- Event hooks for metrics
- Narrow exception handling
- `MockTransport`
- Optional HTTP/2 after source testing
- `trust_env=False` unless proxy environment behavior is intended

Do not:

- Create a client for every request
- Disable TLS verification
- Read unbounded bodies
- Retry all failures
- Trust redirects without revalidation
- Treat URL parse success as URL safety

## 13.2 Playwright

Use:

- One browser process per worker
- Fresh BrowserContext per source investigation or isolated run
- Network request/response observation
- HAR recording and replay
- `page.content()` only when rendered HTML is needed
- Locator APIs for interactions
- Web-first waits
- Trace on failure
- Controlled locale, timezone, and service-worker policy

Do not:

- Launch a browser per page
- Use `time.sleep()`
- Persist personal browser profiles
- Retain storage state
- Use screenshots as extraction data
- Keep the browser alive for offline parsing

## 13.3 selectolax

Use:

- `LexborHTMLParser`
- CSS selectors
- Strict uniqueness checks
- Built-in text extraction
- Efficient node traversal
- Fragment parsing
- Parser-local script inspection

Do not:

- Retain nodes beyond parser lifetime
- Assume serialized HTML equals source bytes
- Parse the same document repeatedly
- Use it for XML or JavaScript execution

## 13.4 lxml

Use:

- Safe `XMLParser`
- `iterparse`
- Namespace-aware XML
- Compiled XPath
- Structured error logs
- Streaming large feeds

Do not:

- Enable external entities
- Enable network retrieval
- Read very large feeds into a full tree unnecessarily
- Parse XML with regular expressions

## 13.5 extruct

Use:

- JSON-LD extraction
- Microdata extraction only when needed
- Final page URL as `base_url`
- Syntax-specific output
- Independent normalization

Do not:

- Parse every syntax by default
- Use `uniform` mode as canonical source truth
- Trust embedded metadata more than visible official source content
- Fetch URLs found in metadata without normal URL policy

## 13.6 tldextract

Use:

- Explicit reusable extractor instance
- Pinned bundled PSL or project-owned PSL
- `top_domain_under_public_suffix`
- `registry_suffix`
- Explicit private-domain policy

Do not:

- Use naive last-two-label splitting
- Treat tldextract as URL validation
- Treat it as SSRF protection
- Change PSL policy without versioning stored domain facts

## 13.7 Supporting libraries

Recommended supporting libraries include:

| Library or protocol | Purpose |
|---|---|
| Tenacity or equivalent bounded retry wrapper | Application-level retry predicates and backoff |
| Protego or another RFC-focused robots parser | Robots Exclusion Protocol handling |
| feedparser | RSS and Atom when encountered |
| `idna` | Explicit hostname normalization |
| `ipaddress` | Public/private/reserved address classification |
| SQLite | Local operational storage |
| SQLite FTS5 | Broad-mandate full-text search |
| DuckDB | Analytical queries and Parquet processing |
| Polars | Efficient tabular transformations where useful |
| zstandard | Raw artifact compression |
| BLAKE3 | Fast content fingerprints |
| pytest | Test execution |
| pytest-asyncio | Async tests |
| Hypothesis | State-machine and normalization invariants |

The LLM programming agent may substitute equivalent libraries where compatibility or implementation evidence justifies it, but the governing behaviors should remain.

---

# 14. Ordered implementation packets

The packets below are ordered by dependency. Each packet should be independently implementable and reviewable.

---

## WP-01 — Core vocabulary and persisted source registry

### Objective

Implement the foundational records and state vocabularies for:

- Employment brands
- Legal entities
- Career sources
- Source scopes
- Source statuses
- Crawl runs
- Job lifecycle states

### Required behavior

- Stable internal identifiers
- Manual override support
- Parent/subsidiary mapping
- Multiple sources per employer
- Presentation and application platform separation
- Migration-safe persistence

### Acceptance

- A sample employer with several subsidiaries and two boards can be represented without duplication.
- A source can move through candidate, verified, healthy, degraded, and retired states.
- No job acquisition is implemented yet.

---

## WP-02 — Raw artifact store and provenance

### Objective

Persist immutable HTTP, XML, JSON, HTML, and browser artifacts.

### Required behavior

- Content hashing
- Compression
- Duplicate-content suppression
- Safe headers
- Redirect chain
- Final URL
- Retrieval timestamp
- Artifact type
- Parser/acquisition version
- Redaction

### Acceptance

- Identical content is not stored repeatedly.
- Original bytes can be replayed.
- Parsed output links back to its source artifact.
- Sensitive headers and cookies are absent.

---

## WP-03 — URL, hostname, and network-safety policy

### Objective

Create one normalization and safety layer for every acquired URL.

### Required behavior

- Scheme allowlist
- Credential rejection
- IDNA handling
- PSL decomposition
- IP classification
- Redirect revalidation
- Private and link-local address rejection
- Maximum redirects
- Maximum response size
- TLS verification
- Decompression bounds

### Acceptance

Tests cover:

- Public URL
- Private IP
- Redirect to private IP
- Unicode hostname
- Punycode hostname
- Unknown suffix
- Custom ATS subdomain
- Embedded credentials

---

## WP-04 — HTTP acquisition gateway

### Objective

Implement shared HTTPX2 acquisition.

### Required behavior

- Long-lived async clients
- Per-host concurrency
- Differentiated timeouts
- Streaming
- Conditional requests
- Retry policy
- `Retry-After`
- Request metrics
- Raw artifact persistence
- Narrow error classification

### Acceptance

MockTransport tests prove:

- Retryable and nonretryable distinctions
- Redirect safety
- 304 handling
- Partial stream failure
- Oversized body rejection
- Idempotent artifact persistence

---

## WP-05 — Robots and sitemap discovery

### Objective

Apply crawl policy and discover employer source URLs.

### Required behavior

- RFC-oriented robots parsing
- Cache robots decisions
- Extract sitemap directives
- Parse sitemap indexes
- Stream large sitemaps
- Classify likely career and job URLs

### Acceptance

- Robots allow and disallow cases pass.
- Missing and failed robots cases follow defined policy.
- Nested sitemap indexes are exhausted.
- A job sitemap can be identified from a fixture corpus.

---

## WP-06 — Adapter SDK and shared contract-test harness

### Objective

Implement the common adapter interface and reusable test suite.

### Required behavior

- Probe
- Enumerate
- Fetch detail
- Parse
- Completeness assessment
- Capability flags
- Shared acquisition gateway
- Shared test fixtures
- Consistent error vocabulary

### Acceptance

A demonstration adapter passes:

- Empty source
- One-page source
- Paginated source
- Repeated page
- Partial detail failure
- Complete run
- Incomplete run

---

## WP-07 — Employer-universe and employment-brand ingestion

### Objective

Load large financial institutions from official and curated sources.

### Initial sources

- FFIEC NIC
- FDIC
- OCC
- NYDFS
- SEC IAPD
- FINRA
- NAIC
- GLEIF
- Manual inclusions

### Required behavior

- Preserve original records
- Normalize official domains
- Generate candidate parent mappings
- Require review for ambiguous merges
- Prevent repeated crawling of shared career boards

### Acceptance

The initial large-financial-institution universe can be generated and reviewed.

---

## WP-08 — Official career-source discovery

### Objective

Resolve employer domains to candidate career sources.

### Discovery methods

- Homepage links
- Footer links
- Robots
- Sitemaps
- Conventional paths
- Redirects
- Vendor-host detection
- Embedded scripts
- JSON-LD
- Iframes and widgets

### Acceptance

For a mixed fixture set, the system produces:

```text
candidate board URL
source-family hints
evidence
confidence
scope status
```

It must not mark a source verified without employer-association evidence.

---

## WP-09 — Generic HTML, XML, feed, and metadata parsing

### Objective

Implement shared parsing primitives.

### Subcomponents

- selectolax HTML parser
- lxml XML and sitemap parser
- RSS/Atom parser
- extruct JSON-LD and Microdata parser
- Description-section splitter
- URL resolver
- Location extractor

### Acceptance

All parser fixtures produce deterministic outputs and retain raw provenance.

---

## WP-10A — Greenhouse adapter

### Acceptance

- Board token discovered
- Exact total reconciled
- Full descriptions parsed
- Offices and departments preserved
- Removed job identified only after complete runs
- All shared contract tests pass

---

## WP-10B — Lever adapter

### Acceptance

- Global and EU hosts supported
- `skip` and `limit` pagination exhausted
- Repeated pages detected
- Descriptions, workplace type, and salary fields preserved
- All shared contract tests pass

---

## WP-10C — Ashby adapter

### Acceptance

- Board name discovered
- Complete public collection retrieved
- Secondary locations preserved
- Compensation optionality handled
- All shared contract tests pass

---

## WP-11 — XML, RSS, and hosted-board adapters

### Source families

- Workable
- Personio
- Teamtailor
- Recruitee

### Acceptance

- Large XML processed in bounded memory
- Teamtailor first-100 truncation avoided
- Recruitee access mode probed per tenant
- Language variants deduplicated
- Supplemental versus authoritative source status distinguished

---

## WP-12 — Generic `JobPosting`, sitemap, and static HTML adapters

### Objective

Support custom employer sites without dedicated ATS adapters.

### Acceptance

- Job URLs enumerated from sitemap or listing
- JSON-LD and visible HTML reconciled
- Listing pagination proved
- Selector drift detected
- JSON-LD alone is never treated as board completeness proof

---

## WP-13 — Job identity, deduplication, and lifecycle reconciliation

### Objective

Implement canonical job identity and active/closed transitions.

### Required behavior

Identity hierarchy:

```text
same source ID
same employer + requisition ID
same canonical apply URL
same employer + exact normalized content fingerprint
fuzzy duplicate candidate
manual review
```

### Acceptance

- Multi-location posting remains one canonical job.
- Two similar but distinct requisitions remain separate.
- Incomplete runs never close jobs.
- Reposts remain linked to prior lifecycles.
- Reprocessing is idempotent.

---

## WP-14 — Scheduler, leases, source health, and anomaly detection

### Objective

Operate recurring crawls safely.

### Required behavior

- Due-source scheduling
- Source leases
- Crash recovery
- Per-host concurrency
- Retry rescheduling
- Run abandonment detection
- Count anomalies
- Source-state transitions
- No-silent-zero rule

### Acceptance

- Concurrent duplicate run is prevented.
- Crashed run can be resumed or safely replaced.
- Zero after high count creates an anomaly rather than closure.
- Health dashboard data is queryable.

---

## WP-15 — Playwright reconnaissance and browser acquisition framework

### Objective

Create the controlled browser-escalation layer.

### Required behavior

- Reusable browser process
- Ephemeral contexts
- Fixed environment
- Network capture
- HAR
- Trace on failure
- Service-worker controls
- Endpoint replay testing
- DOM fallback
- Browser resource limits

### Acceptance

- A JavaScript board is observed.
- Its public endpoint is replayed through HTTPX2.
- Browser-backed and HTTP-promoted outcomes are distinguishable.
- No browser storage state is persisted.

---

## WP-16 — Workday adapter

### Objective

Support Workday external career sites.

### Required behavior

- Tenant/site discovery
- Multiple-site support
- Anonymous search
- Pagination
- Count reconciliation
- Detail retrieval
- HTTP promotion where possible
- Partitioning fallback
- Source-scope verification

### Acceptance

- At least several structurally distinct Workday fixtures pass.
- Unique IDs reconcile with authoritative count.
- Result cap and cursor loop cases are detected.
- Browser use is limited to sources that require it.

---

## WP-17 — Oracle Candidate Experience and Taleo adapters

### Objective

Support both current Oracle career sites and older Taleo sites.

### Required behavior

- Career-site code and context
- Multiple external sites
- Blind search
- Correct count interpretation
- Location-facet handling
- Detail and requisition IDs
- Taleo-specific pagination

### Acceptance

- Multiple site scopes can be unioned.
- Location occurrence counts are not mistaken for requisition totals.
- Oracle and Taleo source families remain separate.

---

## WP-18 — iCIMS and SAP SuccessFactors adapters

### Objective

Support two common enterprise source families without private partner credentials.

### Required behavior

- Public portal acquisition
- Browser-network observation
- Pagination
- Detail parsing
- Custom domains
- Multi-language behavior
- Source-scope verification

### Acceptance

- No authenticated partner API is required.
- Full public portal inventory can be reconciled.
- HTML and network variants are fixture tested.

---

## WP-19 — Candidate-experience overlay adapters

### Families

- Phenom
- Eightfold
- Beamery
- Radancy
- Avature
- Symphony Talent / SmashFly

### Objective

Support personalized career layers and identify underlying ATS relationships.

### Acceptance

- Clean-context enumeration is stable.
- Personalized recommendation results are not mistaken for all jobs.
- Overlay and underlying apply-system identifiers are both stored.
- Public endpoints are promoted to HTTPX2 when possible.

---

## WP-20 — Search, geography, and export

### Objective

Make the inventory useful without building a complex classifier.

### Required behavior

- SQLite FTS5 or equivalent
- Weighted title and description terms
- Industry filter
- NYC core filter
- NYC extended-metro filter
- Unknown-location inclusion
- Active/closed filter
- Employer and date filter
- CSV
- JSONL
- Parquet

### Acceptance

A mandate dictionary can be edited and rerun without recrawling.

Results expose the terms and fields that caused each match.

---

## WP-21 — Manual curation and audit-source imports

### Objective

Ingest opportunities discovered through:

- LinkedIn
- Google
- Indeed
- eFinancialCareers
- Recruiter email
- Employer alerts

### Required behavior

- Stable external observation record
- Official-source reconciliation
- Known-miss classification
- Manual tags and notes
- No overwriting of source-derived fields

### Acceptance

A manually found role can be linked to an existing official posting or recorded as a source miss.

---

## WP-22 — Coverage audit and target-institution rollout

### Objective

Deploy against the initial large-financial-institution universe.

### Initial validation set

- JPMorganChase
- Citi
- Goldman Sachs
- Morgan Stanley
- Bank of America
- BNY
- Wells Fargo
- BlackRock
- Blackstone
- American Express
- MetLife
- New York Life
- AIG
- Prudential
- DTCC
- Intercontinental Exchange / NYSE
- Nasdaq
- Federal Reserve Bank of New York
- Selected foreign banks with substantial NYC operations

### Required outputs

```text
Employer
Official domain
Career sources
Source family
Source scope
Current job count
NYC job count
Mandate match count
Completeness class
Last complete run
Coverage exceptions
```

### Acceptance

Every target employer has one of:

```text
healthy automated source
degraded automated source
manual-only source
blocked source
no public source observed
documented unresolved gap
```

No employer is silently omitted.

---

# 15. Non-negotiable implementation invariants

The LLM programming agent may adjust implementation details, persistence choices, object names, and internal organization. It should not change the following without an explicit design decision.

1. All public jobs are retrieved before mandate filtering.
2. Official employer sources are primary.
3. Raw source artifacts are retained.
4. Acquisition and parsing remain separable.
5. Adapters use the shared acquisition gateway.
6. Direct HTTP is preferred over browser acquisition.
7. Browser-discovered public endpoints are tested for HTTP replay.
8. Source scope is explicit.
9. Completeness requires evidence.
10. HTTP success alone does not imply completeness.
11. Zero jobs require a complete validated run.
12. Incomplete or failed runs cannot close jobs.
13. Enumeration completeness and detail completeness are separate.
14. Multiple locations are preserved.
15. Presentation platform and application platform are separate.
16. Embedded metadata does not prove complete enumeration.
17. Personalized recommendations are not treated as the full job inventory.
18. Public source IDs are preferred over generated identifiers.
19. Every canonical job remains traceable to source observations and raw artifacts.
20. Automated LinkedIn scraping is outside scope.
21. No private ATS credential or application-submission interface is required.
22. Tests run primarily against saved fixtures, not live websites.
23. Every recurring adapter passes the shared contract-test suite.
24. Source drift produces degradation and review, not silent data loss.
25. Search definitions can change without recrawling.

---

# 16. Final target behavior

The implemented system should support an operation conceptually equivalent to:

```text
1. Load all large financial institutions in the active universe.
2. Identify every verified external career source.
3. Crawl every due source.
4. Enumerate every public job.
5. Preserve raw responses.
6. Normalize job identity, description, locations, and URLs.
7. Reconcile active and closed states only from complete runs.
8. Search locally for:
      - NYC
      - internal operations
      - process optimization and adjacent mandates
9. Export the broad candidate set.
10. Compare externally discovered jobs against the official inventory.
11. Report every coverage gap.
```

The resulting dataset should provide, at minimum:

```text
Canonical employer
Legal or subsidiary employer where exposed
Industry verticals
Job title
Department or job family
Employment type
All locations
Remote type and scope
Posting date
First-seen date
Last-seen date
Requisition ID
Source job ID
Compensation fields where available
Description HTML
Description text
Description sections and bullets
Official detail URL
Application URL
Presentation platform
Application platform
Source observations
Active/closed lifecycle
Mandate terms matched
Manual review status
Completeness and source-health context
```

The central measure of quality is not the sophistication of role assessment. It is whether the system can make a credible, auditable statement that it has repeatedly captured the complete public job inventory of every employer in its declared target universe—and can show exactly where that statement remains uncertain.