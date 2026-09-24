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
-> bounded application queries -> SkillEvidence -> Gemini context -> SSE
```

One question may invoke multiple skills. Intent resolution can use prior user
messages only to understand a follow-up; authorization and branch context are
re-resolved for every request. Client conversation history is not sent to
Gemini as unrestricted context.

---

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
