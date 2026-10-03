# AI Guidelines

## Role of AI

AI features may assist with:

- natural-language search;
- dashboard explanations;
- agreement summaries;
- payment/outstanding summaries;
- maintenance/work-order summaries;
- inventory insights;
- drafting operational notes;
- anomaly detection support;
- document extraction where later implemented.

AI is an assistant layer, not the source of truth.

---

## Authorization

Every AI request must operate within the same authorization boundary as normal ERP requests.

AI must never:

- fetch another branch's data for a normal user;
- bypass model policies;
- use unscoped database access;
- rely on frontend filtering for data security.

---

## Branch Context

AI prompts/context should carry explicit server-verified branch context.

Example conceptual context:

```text
actor_user_id
active_branch_id
authorized_branch_ids
super_admin flag
requested_scope
```

For super-admin all-branch analysis, the request must explicitly permit all-branch scope.

---

## Grounding

When answering questions based on ERP data, AI responses should preferably reference source records.

Examples:

```text
Tenant Agreement TA-2026-00121
Property P-0042
Work Order WO-00089
Purchase Order PO-00117
```

Generated explanations must not overwrite system-of-record values.

---

## Tool Use

AI actions should be exposed through narrow, permission-aware application tools such as:

```text
get_dashboard_metrics
search_properties
get_agreement
get_outstanding_payments
create_work_order_draft
```

Avoid giving an AI layer unrestricted SQL access.

Zaakiy uses server-side module skills. The request is classified against an
allowlisted skill, and that skill performs the branch-scoped, permission-aware
read. Gemini receives only the selected skill's compact verified result; it
does not receive a dashboard dump, unrestricted query access, or record text
that can act as instructions. Navigation is emitted separately from an
allowlisted route map.

The current read pipeline is:

```text
question -> IntentFrame -> ReadOrchestrator -> module skills
-> bounded application queries -> ZaakiySkillResult -> Gemini context -> SSE
```

`ZaakiySkillResult` is the internal verified-result contract. It carries
structured metrics, records, breakdowns, comparisons, warnings, source
references, approved navigation targets, follow-ups, resolved time range, and
safe execution metadata. The result is created after the skill's existing
branch and permission checks; it is not an authorization layer. Gemini may
explain the result, but it is never the source of authoritative ERP values.

One question may invoke multiple skills. Intent resolution can use prior user
messages only to understand a follow-up; authorization and branch context are
re-resolved for every request. Client conversation history is not sent to
Gemini as unrestricted context.

Zaakiy may also return a bounded `ZaakiyConversationContext` for the next
follow-up. It contains server-resolved intent, entities, ranges, filters, and
authorized result references; client-supplied context is only a claim and is
revalidated on every request. A branch change clears branch-sensitive context.
This context improves follow-up interpretation but never grants authorization
or data access.

`Property360Skill` provides a read-only, property-level operational summary
from the authorized property, current owner/tenant agreements, bounded
maintenance facts, and permission-gated installment totals. Occupancy is
calculated from the existing property-level tenant-agreement rules. Financial
fields are omitted when `accounts.view` is unavailable. The model has no
rentable components, rooms, beds, or child property hierarchy.

`Tenant360Skill` provides a bounded read-only view of a tenant-role customer,
their tenant agreements and rented properties, plus permission-gated receivable,
payment, and cheque facts. It uses the unified customer model and never mixes
owner-side relationships into the tenant view. Tenant-role and branch checks
are performed before data enters the standardized result.

`Owner360Skill` provides a bounded read-only view of an owner-role customer,
their directly owned properties, owner agreements, occupancy, portfolio
maintenance facts, and permission-gated outward payable/payment facts. It uses
the same unified customer model and does not mix tenant-side balances into the
owner view. Owner-role and branch checks are performed before data enters the
standardized result.

`Agreement360Skill` provides a normalized read-only view for one authorized
tenant or owner agreement. It includes the relevant party, directly linked
properties, lifecycle dates, and permission-gated installment/payment facts.
Tenant agreement finance is inward; owner agreement finance is outward. The
skill preserves the agreement type in results and conversation references so
follow-ups can safely route to the relevant party or property skill.

AgreementRiskSkill identifies deterministic agreement-attention conditions:
expiry within the resolved horizon, an expired agreement retaining an active
lifecycle status, outstanding or overdue balances, bounced cheques, and tenant
agreements extending beyond their linked owner-agreement coverage. It does not
produce an AI-generated risk score. Financial signals require accounts.view;
non-financial lifecycle and coverage signals remain available to authorized
users. Results are branch-scoped, bounded, sorted by explicit condition
priority, and read-only.

`CollectionsHealthSkill` is the read-only, inward-only collections view. It
uses branch-scoped tenant agreement installments and posted inward payments;
outstanding means remaining unpaid scheduled tenant installment balances, and
overdue means that remaining balance on installments past their due date.
Fixed aging buckets are current, 1-30, 31-60, 61-90, and 90+ days overdue.
Cheque summaries use the stored cheque statuses, with pending defined as
posted inward tenant cheques in `received` or `deposited` state. All financial
results require `accounts.view`; without it, no financial values, counts, or
payment/cheque records are returned. The skill is branch-scoped and read-only.

---

## Compound Queries

Compound queries are planned and executed through a bounded `QueryPlanner` and
`CompoundQueryExecutor`. They compose existing authorized skills into at most
three sections and use only validated strategies: parallel sections,
intersection by semantic reference, or bounded reference enrichment. Filters
are propagated only to capabilities that declare them, and each child skill
retains branch scoping, entity reauthorization, and permission checks.

Financial sections may be restricted while non-financial sections still
execute. Child warnings, sources, navigation, and records are deduplicated and
bounded. The executor does not generate SQL joins, infer relationships from
labels, calculate cross-domain KPIs, call Gemini for child sections, or plan
write actions. Empty intersections are valid results; unsupported enrichment
and excessive plans return structured warnings.

## Structured Chat Presentation

The Zaakiy SSE stream keeps the existing `navigation`, `token`, `done`, and
`error` events and additively emits bounded `summary`, `records`, `comparison`,
`trend`, `explanation`, `anomalies`, `sections`, `warnings`, and `suggestions`
events. These events are generated by `ZaakiyPresentationBuilder` from the
verified `ZaakiySkillResult` context before Gemini streaming begins; Gemini
prose is never parsed to recreate ERP values.

The presentation projection removes internal execution metadata, applies the
existing sensitive-data filter, and limits records, sections, warnings,
sources, and follow-ups. The Angular client renders typed neutral cards,
accessible trend tables, bounded records, warnings, and follow-up chips. It
performs no business calculations or independent data fetches, and all actions
remain read-only backend-provided navigation.

## High-Impact Actions

Require explicit user confirmation before AI triggers:

- posting payments;
- issuing/voiding receipts;
- activating agreements;
- terminating agreements;
- scrapping inventory;
- receiving large purchase orders;
- deleting/archiving important records;
- changing branch-sensitive permissions.

Backend authorization and validation still apply after confirmation.

---

## Prompt Injection

Treat ERP-entered text and uploaded documents as untrusted content.

Never let a note, document, tenant message, property description, or attachment override system instructions or authorization.

---

## Privacy

Send only required data to model providers.

Avoid unnecessary inclusion of:

- identity numbers;
- bank account numbers;
- private documents;
- unrelated customers;
- other branches.

Use redaction/minimization where possible.

---

## Audit

Log AI-triggered sensitive operations with:

```text
user
branch
AI feature/tool
target record
requested action
result
timestamp
```

Do not store private chain-of-thought.

Store concise action rationale or user-visible explanation if needed.

---

## Failure Mode

If AI is unavailable, core ERP workflows must continue to function.

Do not make:

- agreement posting;
- payment posting;
- receipt generation;
- inventory receiving;
- work-order completion

dependent on an AI provider unless the business explicitly chooses that dependency.

## Intelligent Report integration

Zaakiy may analyze the Intelligent Report through a dedicated read-only skill.
Laravel calculates and authorizes the report metrics, trend data, and leakage
findings before any evidence is sent to Gemini. Gemini may explain verified
evidence and suggest management review actions, but it must not calculate
authoritative totals, assign leakage exposure/severity, access another branch,
or mutate ERP records. If Zaakiy is unavailable, the verified report remains
usable without AI analysis.

## Identity document form assistance

Owner and tenant create forms may offer a separate identity-document extraction
helper for an Emirates ID image or PDF. This helper is not available through
Zaakiy chat and is not a write action. The backend sends the uploaded document
to the configured Gemini boundary in memory, accepts only an allowlisted JSON
field set, and discards the upload after extraction. It must never extract or
invent a phone number. Extracted values are draft form values only; the user
must review them, enter and confirm the phone number, and submit through the
normal customer endpoint. No customer is created by the extraction request.

Identity documents contain sensitive personal data. Production deployments
must present the applicable privacy notice/consent and configure the external
AI provider according to the organization's data-processing requirements.

## Renewal Intelligence

Renewal Intelligence identifies active tenant and owner agreements approaching
expiry for review. Its default window is 60 days; explicit resolved date
ranges replace it. RENEWAL_* codes are deterministic facts such as due soon,
outstanding, overdue, bounced cheque, and owner-coverage conditions. Financial
signals require accounts.view; lifecycle facts remain available without it.
The skill is branch-scoped, bounded, read-only, and does not recommend or
execute a renewal.

## Vacancy Analysis

Vacancy Analysis evaluates the active branch's active property records at
property level. Occupancy is derived from current qualifying tenant
agreements; future agreements are reported separately, and vacancy duration
uses the latest prior tenant agreement end date when available. Availability
also requires current owner-agreement coverage. Results are deterministic,
 bounded, read-only, and do not use rentable-component records.

## Maintenance Intelligence

Maintenance Intelligence is a branch-scoped, read-only work-order summary.
Open means any work-order status other than `completed` or `cancelled`, matching
the existing operational dashboard; age uses `opened_at`, falling back to
`created_at`, with fixed 0-7, 8-14, 15-30, 31-60, and 60+ day buckets. The
current schema has no authoritative work-order due/SLA field, so overdue
maintenance, repeat-category analysis, and cost intelligence are not inferred.
No maintenance score or write action is exposed.

## Metric Comparisons

Zaakiy comparisons are calculated by the backend comparison engine from two
verified values. Direction is factual only: `increase`, `decrease`, or
`unchanged`; it does not mean good or bad. Percentage deltas are omitted when
the comparison value is zero. Period metrics currently include collections,
completed maintenance, renewal candidates, and agreement attention counts.
Vacancy, maintenance backlog, and receivable snapshot comparisons are not
fabricated when historical reconstruction is unavailable; they return a
structured `HISTORICAL_SNAPSHOT_UNAVAILABLE` warning. Comparison ranges and
filters remain branch-scoped and permission-aware.
Trend Engine responses use the same stable metric IDs for bounded day, week,
month, and quarter time series. Period points are backend-calculated; snapshot
points carry an explicit `as_of` date and partial current periods are marked.
Empty event periods are zero, while unavailable historical snapshots remain
unavailable with a structured warning. Trends preserve branch, entity, and
permission checks and never forecast, detect anomalies, or judge changes.

## Metric Explanations

Metric explanations use `MetricExplanationEngine` and verified backend
decompositions. The initial additive provider explains posted inward
collections by tenant, with bounded deterministic sorting and a residual delta
when returned drivers do not cover the full change. Financial explanations
require `accounts.view` and retain branch/filter scope. Missing comparisons,
non-additive metrics, and unavailable historical state return structured
warnings. Zaakiy reports measured drivers or transitions only; it does not
infer motives, hidden causes, forecasts, or anomaly judgments.

## Deterministic Anomalies

Zaakiy anomaly detection uses `AnomalyDetectionEngine` and explicit code-defined
rules. Initial rules cover significant collection drops, zero collections in a
completed active period, vacancy increases, renewal spikes, bounced cheques,
90+ day vacancies, and old or urgent open work orders. Collection drops require
both a 25% relative decrease and an AED 10,000 absolute decrease. Comparison
rules skip partial current periods. Financial anomaly data requires
`accounts.view`; all sources remain branch-scoped and bounded. No statistical
model, forecast, anomaly score, or AI-generated severity is used.

## Capability Registry

`CapabilityRegistry` is declarative metadata for the current read-only Zaakiy
skills, stable metric IDs, supported filters, time semantics, analytical
features, permissions, references, drill-downs, and known limitations. It is
independent of users, branches, and query results. Registry metadata describes
requirements but never grants authorization, executes queries, or replaces
domain business logic. Management Briefing is registered as a composite
capability and does not own the metrics provided by its dependencies.

## Query Planner

`QueryPlanner` converts an already-resolved `IntentFrame` and conversation
context into a bounded read-only plan. It validates capability ownership,
metrics, filters, entities, time semantics, analytical feature support, and
the registry's read-only boundary. It creates metadata-only plan steps and
never queries tables, calculates domain values, authorizes users, or selects
tools through AI. Compound requests are detected but not executed; runtime
authorization and existing skill execution remain authoritative.
## Management Briefing

`ManagementBriefingSkill` composes bounded results from the existing collections, vacancy, agreement risk, renewal, maintenance, and anomaly capabilities. Its default period is today: period metrics use today and snapshot metrics are marked with the current `as_of` date. Sections are deterministic and may be `available`, `restricted`, or `empty`.

Collections remains protected by `accounts.view`; restricted financial data is omitted without leaking hidden counts. Attention items reuse existing deterministic anomaly and agreement-risk conditions, are deduplicated, severity-sorted, and limited to five. The briefing does not calculate new domain rules, produce a business score, forecast, or perform writes.
