# Revision Authoring Guidelines

Standards for authoring feature revisions — the functional-level change specifications that sit between features and stories in the SPDM hierarchy.

## Overview

A **revision** is the functional analysis artifact in SPDM. It sits between a Feature (strategic capability) and Stories (component-level spec-update tasks):

```
Feature → Revision → Stories → Spec Updates → Code Generation
```

A revision answers the question: **what needs to change, for whom, under what circumstances, and to what end?** It does NOT answer which component to modify, which spec file to update, or how to implement anything. Those decisions are resolved at the Story layer.

**Schema:** [revision-schema.json](../schemas/revision-schema.json)

**Location:** `solution-definition/analysis/revisions/{revision-id}.yaml`

**ID pattern:** `{feature-id}-rev-{N}` (e.g., `user-registration-rev-1`)

**File wrapper:** none — flat top-level properties (no wrapping object key)

The revision is the primary artefact that a functional analyst owns. When a revision is complete, a development team or AI agent can generate unambiguous stories from it without any further analyst involvement. This is the bar. If a developer reading the revision would still need to ask the analyst a question, the revision is not done.

## Contents

- [What Makes a Complete Revision](#what-makes-a-complete-revision) — completeness checklist and the anti-pattern
- [Field-by-Field Reference](#field-by-field-reference) — every schema field and its authoring guidance
- [Functional-Changes as the Structured Container](#functional-changes-as-the-structured-container) — the prefix catalog
- Per-area sections: [Description](#description), [Rationale](#rationale), [Terminology and Glossary Alignment](#terminology-and-glossary-alignment), [Actor and Authorization Rules](#actor-and-authorization-rules), [Trigger](#trigger), [Happy Path](#happy-path), [Alternative Flows](#alternative-flows), [Exception and Error Flows](#exception-and-error-flows), [Business Rules](#business-rules), [Validation Rules](#validation-rules), [Required Data](#required-data), [System Behavior](#system-behavior), [UI and Interaction Expectations](#ui-and-interaction-expectations), [Status and State Changes](#status-and-state-changes), [Integration Expectations](#integration-expectations), [Audit, Logging, and Traceability Needs](#audit-logging-and-traceability-needs), [Notifications and Communication Behavior](#notifications-and-communication-behavior), [Acceptance Criteria](#acceptance-criteria)
- [Status Lifecycle](#status-lifecycle) — the revision's own `status` field
- [Revision Dependencies](#revision-dependencies) — when to use the `dependencies` array
- [Complete YAML Example](#complete-yaml-example) — one fully worked revision
- [Related Guidelines](#related-guidelines)

---

## What Makes a Complete Revision

A revision is eligible for `ready-for-story-creation` only when a development team can derive unambiguous, independently executable stories from it — without asking the analyst a single clarifying question.

### Completeness Checklist

A complete revision covers all of the following analytical areas. Every area must contain specific, actionable content — not placeholders or one-liners. The parenthetical after each area names its **container** — the schema field or `functional-changes` [prefix](#functional-changes-as-the-structured-container) where the content lives.

- **Description** (`description` field) — multi-sentence narrative covering context, affected capability, and expected outcome
- **Rationale** (`rationale` field) — business driver, user pain, regulatory need, or technical debt that justifies this revision
- **Terminology and glossary alignment** (`description`/`notes`, and the solution glossary) — business terms match the canonical glossary, load-bearing terms are defined, no ambiguous or colliding terms, and one concept is named the same way throughout
- **Actors and authorization** (`description`/`notes`, or `BUSINESS RULE BR-N:`) — who initiates the flow, what roles/permissions are required, data-access constraints
- **Trigger** (`TRIGGER:`) — event, action, or condition that causes this functionality to execute
- **Happy path** (`HAPPY PATH:`) — numbered success flow: who acts, what the system does, what data moves, what the user perceives
- **Alternative flows** (`ALTERNATIVE:`) — variations that still succeed but diverge from the happy path
- **Exception and error flows** (`EXCEPTION:`) — what happens when things go wrong: validation failures, service unreachable, unauthorized
- **Business rules** (`BUSINESS RULE BR-N:`) — named invariants governing when and how the system may act
- **Validation rules** (`VALIDATION:`) — field-level and cross-field input constraints with their consequences, including bounds on any operation acting on a set/batch (maximum selection or batch size)
- **Required data** (`SYSTEM BEHAVIOR:`, or `description` when foundational) — input data, fetched data, and external data with source and purpose
- **System behavior** (`SYSTEM BEHAVIOR:`) — internal computations, transformations, state transitions, persistence order
- **UI and interaction expectations** (`UI:`) — affected screens, layout intent, interactive elements, feedback patterns (when UI is involved)
- **Status and state changes** (`STATE CHANGE:`) — entity transitions triggered by this revision with side-effects
- **Integration expectations** (`INTEGRATION:`) — external systems touched, direction, protocol, sync/async, failure intent
- **Audit, logging, and traceability** (`AUDIT:`) — what must be recorded for compliance, debugging, and observability
- **Notifications** (`NOTIFICATION:`) — what communications the system sends as a result, to whom, and via which channel
- **Acceptance criteria** (`acceptance-criteria` field) — testable, observable, unambiguous statements proving the revision is complete

### Anti-Pattern: Incomplete Revision

```yaml
# ❌ WRONG — vague, lacks analysis, cannot drive story generation
revision-id: order-management-rev-1
feature-id: order-management
revision-number: 1
title: Add order cancellation
description: Allow users to cancel orders.
acceptance-criteria:
  - Order can be cancelled.
status: draft
created-at: "2026-01-15T10:00:00Z"
```

The ✓ RIGHT counterpart replaces every vague line with specific, classified analysis: a multi-sentence `description`, a measurable `rationale`, and `functional-changes` items labeled `TRIGGER:`, `HAPPY PATH:`, `ALTERNATIVE:`, `EXCEPTION:`, `BUSINESS RULE BR-N:`, `VALIDATION:`, `SYSTEM BEHAVIOR:`, `STATE CHANGE:`, `INTEGRATION:`, `NOTIFICATION:`, `AUDIT:`, and `UI:`, plus observable `acceptance-criteria` and `status: ready-for-story-creation`. See the [Complete YAML Example](#complete-yaml-example) at the end of this guideline for the full worked revision.

---

## Field-by-Field Reference

Every field defined in `revision-schema.json`:

| Field | Required | Type / Constraints | Authoring Guidance |
|-------|----------|--------------------|--------------------|
| `revision-id` | Yes | string — pattern `{feature-id}-rev-{N}` | Set once at creation. Never change after the revision is shared. Example: `user-registration-rev-2` |
| `feature-id` | Yes | string | Must match the `feature-id` of the parent feature exactly. |
| `revision-number` | Yes | integer ≥ 1 | Sequential within the feature. First revision is `1`. Increment by 1 for each subsequent revision. |
| `title` | No | string | Short (≤ 80 characters), action-oriented title that names the specific change. "Customer-initiated order cancellation with refund eligibility check" is better than "Cancel orders". |
| `description` | Yes | string | Primary narrative. Must be multi-sentence. See [Description](#description) section. |
| `rationale` | No | string | Why this revision exists now. See [Rationale](#rationale) section. |
| `functional-changes` | No | array of `{ change-description: string }` | Enumerated discrete changes. Use labeled prefixes: TRIGGER, HAPPY PATH, ALTERNATIVE, EXCEPTION, BUSINESS RULE, VALIDATION, SYSTEM BEHAVIOR, STATE CHANGE, INTEGRATION, NOTIFICATION, AUDIT, UI. See [functional-changes usage](#functional-changes-as-the-structured-container) below. |
| `acceptance-criteria` | No | array of strings | Testable, observable statements. See [Acceptance Criteria](#acceptance-criteria) section. Must be non-empty before moving to `ready-for-story-creation`. |
| `dependencies` | No | array of revision-id strings | List revisions that must be completed before this one. Pattern: `{feature-id}-rev-{N}`. See [Revision Dependencies](#revision-dependencies). |
| `status` | No | enum: `draft \| ready-for-story-creation \| stories-created \| in-progress \| completed \| cancelled` | Analyst manages `draft` and `ready-for-story-creation`. Other values are set by automation. See [Status Lifecycle](#status-lifecycle). |
| `created-by` | No | string | Person or system that authored the revision. Use email or username. |
| `created-at` | Yes | string (date-time) | ISO 8601 UTC timestamp. Set at creation, never modified. |
| `updated-at` | No | string (date-time) | ISO 8601 UTC timestamp. Update whenever the revision content changes. |
| `tags` | No | object (key-value string pairs) | Free-form categorization: team, sprint, priority, domain area. Example: `{ "team": "payments", "sprint": "2026-Q1-S2" }` |
| `notes` | No | string | Analyst notes, open questions, parking-lot items, decisions not yet resolved. Capture actors, UI surface absence, and open questions here. |
| `generated-by` | No | object: `{ model, generation-date }` | Populated automatically when an AI agent creates or materially contributes to the revision. Do not fill in manually. |

---

## Functional-Changes as the Structured Container

The `functional-changes` array is the primary place to enumerate every discrete analytical item: happy path steps, alternative flows, exception flows, business rules, validation rules, system behavior, state changes, integrations, notifications, and audit requirements. While `description` holds the top-level narrative, the detailed breakdown lives in `functional-changes`.

**Use labeled prefixes** so that story-generation agents and human reviewers can immediately classify each item:

| Prefix | What it contains |
|--------|-----------------|
| `TRIGGER:` | The event or action that initiates the flow |
| `HAPPY PATH:` | Primary success flow (numbered steps) |
| `ALTERNATIVE:` | A variation that diverges from the happy path but still succeeds |
| `EXCEPTION:` | A failure condition and the system's response |
| `BUSINESS RULE BR-N:` | A named invariant with its violation consequence |
| `VALIDATION:` | Field-level or cross-field input constraint |
| `SYSTEM BEHAVIOR:` | Internal processing, computations, persistence sequencing |
| `STATE CHANGE:` | Entity status/state transition with side-effects |
| `INTEGRATION:` | External system interaction with direction and failure intent |
| `NOTIFICATION:` | Communication sent as a result of this revision |
| `AUDIT:` | What must be recorded for compliance or observability |
| `UI:` | User-facing screen, layout, interactive element, and feedback behavior |

These prefixes are a **convention, not a schema constraint** — the schema only sees `change-description` as free text and does not validate the label. Consistent prefixing is what lets downstream story-generation agents and reviewers classify each item, so use the labels exactly as written above.

Every analytical area listed in this guideline must appear in at least one `functional-changes` item or in `description`/`notes`. The schema's `additionalProperties: false` constraint means you cannot add custom fields — use `functional-changes` and `notes` as the structured containers for all analytical content. The [Completeness Checklist](#completeness-checklist) annotates which container each analytical area uses.

---

## Description

The `description` field is the primary narrative — the functional analyst's own words on what this revision delivers.

It must answer:
- **What** is changing or being added?
- **For whom** — which actor or user role benefits?
- **Under what circumstances** — preconditions, context, constraints?
- **To what end** — the expected outcome and business value?

**Minimum scope:** at least three to five sentences covering context, affected capability, and expected outcome. A single sentence is never sufficient. A vague statement like "Allow users to cancel orders" provides no analytical value.

```yaml
# ✓ RIGHT — substantive description with context, capability, and outcome
description: >
  This revision introduces the ability for authenticated customers to cancel their own
  orders from the order detail screen, subject to business rules governing cancellation
  eligibility. Only orders in 'Pending' or 'Confirmed' status may be cancelled; upon
  cancellation the system releases reserved inventory, evaluates refund eligibility, and
  notifies the customer. This capability replaces a manual customer support workflow that
  currently accounts for 15% of support ticket volume.
```

See the [Complete YAML Example](#complete-yaml-example) for how this description sits within a full revision.

---

## Rationale

The `rationale` field explains why this revision exists **now** — the force that makes this change necessary or valuable at this point in time.

Rationale should reference one or more of:
- **Business driver** — strategic goal, revenue impact, competitive pressure
- **User pain** — friction, complaint, support ticket volume, usability issue
- **Regulatory or compliance need** — legal mandate, audit requirement, data residency rule
- **Technical debt** — architectural inconsistency, duplication, or fragility that must be addressed

The rationale connects this revision to the parent feature's purpose. A revision whose rationale does not relate to the feature's description indicates a potential misclassification.

```yaml
# ✓ RIGHT — specific, measurable, connects to strategy
rationale: >
  Customer support receives 15% of all inbound contacts about order cancellations
  handled manually by agents. This revision directly reduces that load, improves
  customer satisfaction NPS (target: +4 points), and fulfills the self-service
  mandate in the Q3 product strategy review. It also removes a compliance gap:
  GDPR article 17 requires that customers can withdraw from a transaction without
  contacting support.
```

---

## Terminology and Glossary Alignment

A revision must use **consistent, canonical business vocabulary**. Terminology problems are the most dangerous kind of defect because a confused term reads as correct and then propagates into every spec and line of code derived from the revision. There are two distinct concerns: alignment with the **canonical glossary** (external) and **internal naming consistency** (within the revision itself).

### Glossary fit (external)

The solution's canonical business vocabulary lives in `solution-definition/glossary.yaml` (see [glossary-schema.json](../schemas/glossary-schema.json)). For every **business** term the revision references — not just the one it is *about* — confirm it matches the glossary:

- **Use the canonical term**, not a synonym. If the glossary defines *Customer Order*, write "Customer Order" — never "customer request" or "sales order" for the same concept.
- **Define load-bearing terms that are missing.** If the revision leans on a business term the glossary does not define, add it to the glossary (canonical shape: a top-level `glossary:` list of `{ key, term, definition, tags? }`). Anchor candidate terms to the business entities in `solution.yaml`.
- **Reference, not subject, is the bar.** A revision centred on Transport Requests that mentions converting one into a *Customer Order* depends on **both** terms — define *Customer Order* too, even though it "belongs to" another feature.
- **Write durable definitions.** A glossary definition captures what the term fundamentally *is* in the business — not its current rules. Leave out statuses/state machines, internal keys and field names, conditions and preconditions, and formats/thresholds (that detail belongs in the specs, which revisions keep changing).
  - *Too specific (avoid):* "An inbound request to move goods, identified by transportRequirementId, progressing through New, Open, Booked, Closed."
  - *Durable (prefer):* "An inbound request from a customer to move goods from one location to another; the starting point of the transport lifecycle."

**Business vocabulary only.** The glossary captures domain language a stakeholder would use unprompted — entities, actors, statuses, and genuine domain concepts or actions (*Transport Request*, *Partner*, *Convert to Order*). It is **not** a data dictionary. Do **not** treat as glossary terms: field/attribute names or internal identifiers (`sendTime`, `transportRequirementId`), technical mechanisms or processes ("number range", "allocate-before-persist"), formats/encodings ("TRYYXXXXXX"), or UI/storage artifacts. **Litmus test:** *would a domain expert who has never seen the code use this word to describe the business?* If it only makes sense to an implementer, it does not belong in the glossary.

### Ambiguous or colliding terms (most serious)

Do not introduce a business term that is *similar but not identical* to an existing glossary term or `solution.yaml` entity name. If a revision says *"customer request"* while the glossary carries *Customer Order* and *Transport Request*, downstream cannot tell whether it means one of those or a third thing — and every spec built on the term is then guessing. Resolve it before the revision is complete: either it is the **same** concept → collapse to the single canonical term, or a **different** concept → name and define both distinctly. The same applies to a single revision term that could plausibly map to two existing concepts.

### Internal consistency

Within the revision, name one concept the **same way everywhere** — across `description`, `functional-changes`, and `acceptance-criteria` — and align that name with the entity names in `solution.yaml`. Synonyms drifting across fields are a real hazard for the agents and reviewers who consume the revision.

```yaml
# ❌ WRONG — the same thing is called three different names across fields
description: >
  Customers can cancel an order from the order screen.
functional-changes:
  - change-description: "HAPPY PATH: ... the system cancels the request ..."
acceptance-criteria:
  - "Given a pending job, when the customer cancels it, then ..."

# ✓ RIGHT — one canonical name ('Order') used consistently, matching the solution.yaml entity
description: >
  Customers can cancel an Order from the order detail screen.
functional-changes:
  - change-description: "HAPPY PATH: ... the system cancels the Order ..."
acceptance-criteria:
  - "Given a pending Order, when the customer cancels it, then ..."
```

---

## Actor and Authorization Rules

The revision schema has no dedicated `actors` field. Actor and authorization information **must** be captured in `description` and/or `notes`, or as `functional-changes` items prefixed `BUSINESS RULE BR-N:`.

For every revision, identify:
- **Who initiates the flow** — end-user role (Customer, Administrator, Back-office Agent), system actor (scheduled job, webhook receiver, Service Bus consumer), or external party
- **What permissions are required** — authenticated/unauthenticated, role-based, ownership-based
- **Data-access constraints** — tenant isolation, ownership (user can only access their own records), organizational scope

| Actor type | Description | Authorization requirement |
|------------|-------------|--------------------------|
| End-user (Customer) | Registered customer interacting via UI or mobile app | Must be authenticated; may only act on own data (ownership check) |
| End-user (Administrator) | Internal staff with elevated privileges | Must be authenticated with Admin role; may act across all tenants/customers |
| System actor (Scheduler) | Automated job running on a cron schedule | No user session; uses service identity; no per-user ownership checks |
| External integration | Webhook or callback from a third-party service | Validated via shared secret or OAuth token; no user context |

State explicitly whether unauthenticated access is permitted. If no UI is involved (system-to-system flow), state this in `notes`.

```yaml
# Example: capturing actor rules in functional-changes
functional-changes:
  - change-description: >
      BUSINESS RULE BR-1: Only an authenticated Customer may cancel their own order.
      The request must carry a valid session token. The API must verify that the
      order's customer-id matches the authenticated user's identity. Attempting to
      cancel another customer's order returns HTTP 403.
notes: >
  Unauthenticated access is not permitted for any operation in this revision.
  Internal admin cancellation (back-office use case) is out of scope; that is
  covered in order-management-rev-3.
```

---

## Trigger

The trigger describes **what event, action, or condition causes this revision's functionality to execute**. There is no dedicated `trigger` schema field — capture triggers in `description` or as the first `functional-changes` item labeled `TRIGGER:`.

Triggers can be:

| Trigger type | Example |
|-------------|---------|
| **User action** | Button click, form submission, navigation event |
| **System event** | Scheduled job fires, message received on Service Bus, timer elapses |
| **External callback** | Webhook from payment gateway, OAuth callback, third-party API event |
| **State transition** | Entity reaches a specific status in another process |
| **API call** | External system calls an endpoint directly |

If multiple triggers can initiate the same flow (e.g., both a user action and a system event), list each one as a separate `TRIGGER:` item in `functional-changes`.

```yaml
functional-changes:
  - change-description: >
      TRIGGER: Customer clicks 'Cancel Order' on the order detail screen for an
      order in 'Pending' or 'Confirmed' status. The button is not rendered for
      other statuses.
  - change-description: >
      TRIGGER: Customer calls the cancellation API endpoint directly with a valid
      session token and an eligible order ID. The API must enforce the same
      eligibility rules independently of the UI.
```

---

## Happy Path

The happy path is the primary success scenario — a step-by-step, numbered sequence of what happens when everything goes right. Capture it in `description` (as a numbered-list paragraph) or as a `functional-changes` item labeled `HAPPY PATH:`.

Each step must identify:
1. **Who acts** — user, system, external service
2. **What the system does** — the action taken
3. **What data moves** — inputs consumed, outputs produced
4. **What the user perceives** — UI feedback, navigation change, message displayed

```yaml
functional-changes:
  - change-description: >
      HAPPY PATH:
      (1) Customer opens their order detail screen and clicks 'Cancel Order'.
      (2) System displays a confirmation modal with an optional reason field
          (max 500 characters) and 'Confirm' / 'Back' actions.
      (3) Customer enters an optional reason and clicks 'Confirm'.
      (4) System validates that the order is still in 'Pending' or 'Confirmed' status.
      (5) System transitions order status to 'Cancelled' and records the reason.
      (6) System calls the inventory service synchronously to release reserved quantities.
      (7) System evaluates refund eligibility and stores the result on the order record.
      (8) System publishes an Order.Cancelled event to the notification topic.
      (9) System sends a cancellation confirmation email to the customer asynchronously.
      (10) UI closes the modal, navigates to the order list, and displays a success toast:
           "Your order has been cancelled."
```

---

## Alternative Flows

Alternative flows are variations that still result in success but follow a different path. Each alternative must reference the step in the happy path where it diverges.

Capture in `functional-changes` as items labeled `ALTERNATIVE:`.

```yaml
functional-changes:
  - change-description: >
      ALTERNATIVE (diverges after step 2): Customer clicks 'Back' in the confirmation
      modal. The order is unchanged. The modal closes and the customer remains on the
      order detail screen. No API call is made.
  - change-description: >
      ALTERNATIVE (diverges after step 3): Customer submits without entering a reason.
      The reason field is optional; the flow proceeds identically to the happy path
      with a null reason value stored on the order record.
```

---

## Exception and Error Flows

Exception flows describe situations where the system cannot complete the operation. For each exception, specify:
- The **trigger condition** that causes it
- What **the system does** (rolls back, returns error, logs, alerts)
- What **the user perceives** (error message intent, navigation outcome)

Distinguish **recoverable exceptions** (user can correct and retry) from **non-recoverable** ones (system error, data inconsistency).

Capture in `functional-changes` as items labeled `EXCEPTION:`.

```yaml
functional-changes:
  - change-description: >
      EXCEPTION (recoverable): The order status is no longer 'Pending' or 'Confirmed'
      when the system validates eligibility (step 4 of happy path — concurrent change).
      System returns an error: "This order can no longer be cancelled." UI refreshes
      the order detail to show the current status. No state change occurs.
  - change-description: >
      EXCEPTION (non-recoverable): The inventory service is unavailable or returns an
      error when releasing reservations (step 6). The system rolls back the status
      transition — the order remains in its original status. Customer receives an error:
      "Cancellation could not be completed. Please try again later." The failure is
      logged with correlation ID for support investigation.
  - change-description: >
      EXCEPTION (non-recoverable): The customer's session is expired or invalid when
      the cancellation request reaches the API. System returns HTTP 401. UI redirects
      to the login screen.
```

---

## Business Rules

Business rules are named invariants that govern **when and how** the system may perform the operation. Each rule must be:
- **Named** with a sequential identifier: `BR-1`, `BR-2`, etc.
- **Stated as an invariant** — a condition that must always hold true
- **Associated with a consequence** — what happens when the rule is violated (intent, not exact error code; the error code is resolved at the Story layer)

Capture in `functional-changes` as items labeled `BUSINESS RULE BR-N:`, or in `notes` when there are many rules.

**Anti-pattern:** embedding implicit rules only in the description prose without naming them explicitly.

```yaml
# ❌ WRONG — rule buried in prose, not extractable
description: >
  Users can cancel their orders but not if they are dispatched. Also only their own orders.

# ✓ RIGHT — named, invariant-style, with violation consequence
functional-changes:
  - change-description: >
      BUSINESS RULE BR-1: An order may only be cancelled by the customer who placed it.
      Violation: HTTP 403 with message indicating unauthorized action.
  - change-description: >
      BUSINESS RULE BR-2: Cancellation is only permitted when order status is 'Pending'
      or 'Confirmed'. Orders in 'Dispatched', 'Delivered', or 'Cancelled' status cannot
      be cancelled. Violation: HTTP 422 with message indicating ineligible status.
  - change-description: >
      BUSINESS RULE BR-3: Inventory reservation must be released before confirming
      cancellation to the customer. If the inventory service call fails, the cancellation
      is aborted and the order status is not changed. Violation: HTTP 503 with retry guidance.
```

---

## Validation Rules

Validation rules describe field-level and cross-field input constraints. For each rule, specify:
- Which **field(s)** it applies to
- The **constraint** (required, format, range, maximum length, uniqueness, reference check, cross-field dependency)
- The **consequence** (what the user sees; exact error codes resolved at Story layer)

Capture in `functional-changes` as items labeled `VALIDATION:`.

```yaml
functional-changes:
  - change-description: >
      VALIDATION: The cancellation reason field is optional. If provided, it must not
      exceed 500 characters. Consequence: inline validation error displayed beneath
      the field before submission; form cannot be submitted while the constraint is violated.
  - change-description: >
      VALIDATION: The order-id in the cancellation request path parameter must match
      an existing order belonging to the authenticated customer. A non-existent order
      returns HTTP 404; an order belonging to another customer returns HTTP 403.
```

### Bounded Operations and Batch Constraints

When a change acts on a **set** — "one or more selected …", a batch, a bulk action, a list processed in a loop — field-level validation is not enough. Such an operation has **three independent constraints**, and the third is the most frequently missed:

1. **Eligibility** — which items may be acted on (a precondition).
2. **Atomicity** — what happens when one item fails mid-operation (all-or-nothing, or partial-success with a defined outcome per item).
3. **Maximum size** — *how many* items may be acted on at once.

The first two do not answer the third. A bounded-eligibility, atomic batch with **no size cap is still unbounded** — an open-ended, long-running operation with partial-failure exposure. The revision must either state an explicit cap or deliberately declare "no cap" with a reason it is safe under load. **Silence is an undecided maximum, not a decided "unlimited".**

Capture these in `functional-changes` using `VALIDATION:` (size/eligibility) and `BUSINESS RULE BR-N:` (atomicity), and add matching acceptance criteria.

```yaml
# ✓ RIGHT — a bulk operation with all three constraints decided
functional-changes:
  - change-description: >
      VALIDATION: A bulk-cancel request may include at most 100 order IDs. A request
      exceeding 100 is rejected with a validation error before any order is processed.
  - change-description: >
      BUSINESS RULE BR-5: Bulk cancellation is processed per-item; if an individual
      order is ineligible, it is skipped and reported in the per-item result while the
      remaining eligible orders are still cancelled (partial success, not all-or-nothing).
acceptance-criteria:
  - "Given a bulk-cancel request with 101 order IDs, then the request is rejected and no order is cancelled."
  - "Given a bulk-cancel request where 1 of 5 orders is ineligible, then the other 4 are cancelled and the response reports the 1 skipped order."
```

---

## Required Data

Document every data item the system needs to complete the revision's functionality.

| Data item | Source | Mandatory | Purpose |
|-----------|--------|-----------|---------|
| Order record | Database (order-service) | Yes | Read current status and customer ownership for eligibility check |
| Customer identity | Authentication token (session) | Yes | Ownership enforcement — compare with order's customer-id |
| Cancellation reason | User input (optional free text) | No | Stored on the order record; included in notification email |
| Inventory reservation records | Database (inventory-service) | Yes (fetched by inventory service) | Released when cancellation is confirmed |
| Cancellation policy configuration | Configuration / database | Yes | Determines refund eligibility for this order type and status |

Capture data requirements in `functional-changes` using the `SYSTEM BEHAVIOR:` prefix, or in `description` when they are foundational to understanding the revision.

---

## System Behavior

System behavior describes **internal processing** the system performs: computations, transformations, state machine transitions, data enrichment, and async processing.

**Level of detail:** business-level descriptions of what gets computed or persisted, in what sequence critical steps must occur. Do NOT include implementation details (no ORM calls, no SQL, no framework-specific terms, no HTTP client library names).

```yaml
functional-changes:
  - change-description: >
      SYSTEM BEHAVIOR: The cancellation flow must execute steps in this order to maintain
      data integrity: (1) validate eligibility, (2) call inventory service synchronously
      to release reservations — abort if this fails, (3) transition order status to
      'Cancelled' and store reason and cancellation timestamp, (4) evaluate and store
      refund eligibility, (5) publish Order.Cancelled event, (6) queue cancellation
      email for async delivery. Steps 5 and 6 are fire-and-forget: failures are logged
      but do not affect the HTTP response.
  - change-description: >
      SYSTEM BEHAVIOR: Refund eligibility is determined by the cancellation policy:
      orders cancelled within 1 hour of placement are fully refundable; orders cancelled
      after 1 hour but before dispatch are partially refundable (minus a processing fee);
      orders in 'Confirmed' status cancelled after 24 hours are non-refundable. The result
      ('FULL', 'PARTIAL', or 'NONE') and the applicable policy version are stored on the
      order record.
```

---

## UI and Interaction Expectations

When the revision involves end-user-facing behavior, describe:
- **Screen(s) affected** — which views change and how
- **Layout intent** — wizard, modal, form, list/detail, inline edit
- **Interactive elements** — buttons, filters, contextual actions, drag handles
- **Feedback patterns** — loading state, success message, error display, empty state
- **Navigation outcome** — where the user lands after completing the flow

Capture in `description`, in `functional-changes` using the `HAPPY PATH:` context, or in `notes`.

**Note:** exact visual design (colors, typography, spacing, component library choices) is resolved at the Story/Functional-UI-Spec layer. Capture **intent and behavior**, not CSS.

If the revision has no UI surface (e.g., a purely system-to-system or background process), state this explicitly in `notes`:

```yaml
notes: >
  This revision has no UI surface. It is triggered exclusively by the inventory
  reconciliation scheduler and produces no user-visible output. All feedback is
  via Service Bus events consumed by downstream services.
```

When UI is involved, example guidance:

```yaml
functional-changes:
  - change-description: >
      UI: The 'Cancel Order' button appears on the order detail screen in the actions
      panel, visible only when order status is 'Pending' or 'Confirmed'. Clicking the
      button opens a confirmation modal (not a new page). The modal contains: a
      summary of the order being cancelled (order reference, total amount), an optional
      reason textarea, and two actions: 'Confirm Cancellation' (primary/destructive
      style) and 'Go Back' (secondary). While the API call is in flight, the 'Confirm'
      button shows a loading spinner and is disabled to prevent double-submission.
      On success: modal closes, toast notification appears ("Order cancelled"), order
      status badge updates in the list. On error: modal remains open, error message
      displayed inline within the modal.
```

---

## Status and State Changes

Describe every entity status or state transition triggered by this revision.

For each transition:

| From state | Event / action | To state | Who can trigger | Side-effects |
|------------|---------------|----------|-----------------|--------------|
| Pending | Customer confirms cancellation | Cancelled | Customer (owner) | Inventory released; Order.Cancelled event published |
| Confirmed | Customer confirms cancellation | Cancelled | Customer (owner) | Inventory released; Order.Cancelled event published |

Capture in `functional-changes` using the `STATE CHANGE:` prefix.

```yaml
functional-changes:
  - change-description: >
      STATE CHANGE: Order transitions from 'Pending' or 'Confirmed' to 'Cancelled'
      when the customer confirms the cancellation and the inventory service confirms
      reservation release. Side-effects: (1) Order.Cancelled domain event published
      to the notification topic (not fire-and-forget — failure rolls back the transition),
      (2) inventory reservation released, (3) refund eligibility evaluated and stored.
```

### Revision's Own Status Lifecycle

The revision file itself has a `status` field that the analyst manages as work progresses. See [Status Lifecycle](#status-lifecycle) for the full lifecycle description.

---

## Integration Expectations

Describe every external system, service, or component this revision interacts with.

For each integration:

| Target system | Direction | Protocol | Sync/Async | Failure handling intent |
|--------------|-----------|----------|-----------|------------------------|
| inventory-service | Outbound | REST | Synchronous (blocking) | Failure rolls back status change; HTTP 503 returned to customer |
| notification-topic | Outbound | Azure Service Bus | Asynchronous (fire-and-forget) | Failure logged; does not affect HTTP response |
| email-service | Outbound | REST / async queue | Asynchronous (fire-and-forget) | Failure logged; customer may not receive email but cancellation succeeds |

Capture in `functional-changes` using the `INTEGRATION:` prefix.

**Note:** Which specific component implements each integration and which spec files to update is resolved at the Story layer. The revision captures **intent**, not wiring.

```yaml
functional-changes:
  - change-description: >
      INTEGRATION: The inventory service must be called synchronously to release the
      reservation before confirming the cancellation. The call is blocking: if the
      inventory service returns an error or is unreachable, the order status must NOT
      be changed. Timeout: 10 seconds. On timeout, treat as failure.
  - change-description: >
      INTEGRATION: An Order.Cancelled event must be published to the domain event
      notification topic after persistence. This is NOT fire-and-forget: failure to
      publish must surface as a 500 response and the status transition is not committed.
```

---

## Audit, Logging, and Traceability Needs

Describe what the system must record for compliance, debugging, and operational visibility.

Cover:
- **Audit log entries** — what action is recorded, by whom, when, and on what entity
- **Activity records** — structured entries in the activity service (actor, action, entity type, entity ID, timestamp)
- **Correlation ID propagation** — whether the correlation ID must be threaded through async processing
- **Retention requirements** — how long records must be kept (regulatory or operational)
- **Compliance drivers** — if a specific regulation requires the log (GDPR, SOX, PCI-DSS, etc.)

Capture in `functional-changes` using the `AUDIT:` prefix.

```yaml
functional-changes:
  - change-description: >
      AUDIT: Record a cancellation event in the activity log immediately after
      the order status transitions. Required fields: actor (customer ID and session
      reference), action ('ORDER_CANCELLED'), entity type ('Order'), entity ID,
      timestamp (ISO 8601 UTC), cancellation reason (if provided), and refund
      eligibility result. Retention: 7 years per financial transaction compliance
      policy. The correlation ID from the original HTTP request must be included
      to enable end-to-end tracing from the customer action to the audit record.
```

---

## Notifications and Communication Behavior

Describe every notification or communication the system sends as a result of this revision.

For each notification:

| Recipient | Trigger condition | Channel | Content intent |
|-----------|-----------------|---------|---------------|
| Customer | Order successfully cancelled | Email (transactional) | Order reference, cancellation date, itemised order summary, refund eligibility outcome |
| Operations team | Cancellation fails due to inventory service error | Internal alert / monitoring | Error details, correlation ID, affected order ID |

Capture in `functional-changes` using the `NOTIFICATION:` prefix.

**Note:** The exact notification template, email rendering, and Service Bus message schema are resolved at the Story/spec layer. The revision captures **what must be communicated and to whom**, not the HTML template.

```yaml
functional-changes:
  - change-description: >
      NOTIFICATION: A transactional email is sent to the customer's registered email
      address after a successful cancellation. Content must convey: (1) confirmation
      that the order has been cancelled, (2) the order reference number, (3) the
      cancellation date and time, (4) an itemised summary of the cancelled order,
      (5) the refund eligibility outcome and next steps (e.g., "A full refund of
      €47.50 will be processed within 3–5 business days"). The email is sent
      asynchronously — a failure to send does not reverse the cancellation.
```

---

## Acceptance Criteria

Acceptance criteria are testable, observable, unambiguous statements that prove the revision is complete and correct.

**Quality requirements for each criterion:**
- **Observable** — verifiable through the UI, API response, database state, or log output
- **Unambiguous** — only one possible interpretation of pass/fail
- **Independent** — each criterion tests one distinct behavior
- **Free of implementation detail** — no references to function names, SQL, or class names

**Minimum coverage:**
- One criterion per distinct observable outcome of the happy path (not one per micro-step)
- One criterion per major exception flow
- One criterion per named business rule
- One criterion per key integration behavior
- One criterion per notification

**Format:** Prefer `Given / When / Then` for user-facing behavior, or declarative statements for system-level properties.

```yaml
# ❌ WRONG — not testable, vague, multiple behaviors in one criterion
acceptance-criteria:
  - "The cancellation works correctly and the user gets a confirmation."
  - "Errors are handled properly."
```

```yaml
# ✓ RIGHT — one behavior per criterion, observable, unambiguous
acceptance-criteria:
  - "Given a customer with an order in 'Pending' status, when they confirm cancellation, then the order status changes to 'Cancelled' within 2 seconds and a success toast is displayed."
  - "Given a customer with an order in 'Dispatched' status, when they view the order detail, then the 'Cancel Order' button is not rendered."
  - "Given a customer attempting to cancel another customer's order via the API with a valid session token, then the API returns HTTP 403."
  - "Given the cancellation reason exceeds 500 characters, when the customer attempts to submit, then an inline validation error is displayed and the form cannot be submitted."
  - "Given the inventory service returns a 500 error during a cancellation attempt, then the order status remains unchanged and the customer receives an error message."
  - "Given a successful cancellation, then an Order.Cancelled event is published to the notification topic within 5 seconds."
  - "Given a successful cancellation, then a cancellation confirmation email is sent to the customer's registered address within 60 seconds."
  - "Given a successful cancellation, then an audit log entry is created with the actor ID, action 'ORDER_CANCELLED', entity ID, timestamp, and correlation ID."
```

---

## Status Lifecycle

The `status` field tracks where the revision sits in the analysis and implementation pipeline. The analyst is responsible for managing the `draft` and `ready-for-story-creation` values. All other values are set by automation (story-generation tooling or project management integration).

| Status | Owner | Meaning | Conditions |
|--------|-------|---------|------------|
| `draft` | Analyst | Being written; content is incomplete or under review | Default value; set at creation |
| `ready-for-story-creation` | Analyst | All analytical sections are complete; development team can generate stories without further analyst input | **Must not be set while `acceptance-criteria` is empty.** All sections in the completeness checklist must be addressed. |
| `stories-created` | Automation | Stories have been generated for all affected components | Set by story-generation agent after successful run |
| `in-progress` | Automation / PM tooling | At least one story is actively being implemented | Set by project management integration |
| `completed` | Automation / PM tooling | All stories are in `done` status and the revision has been verified | Set when all child stories reach terminal state |
| `cancelled` | Analyst / PM | Revision is abandoned and will not be implemented | When setting `cancelled`, document the reason in `notes` |

**Rule:** A revision **must not** be moved to `ready-for-story-creation` while the `acceptance-criteria` array is empty or contains only vague statements. Acceptance criteria are the gate.

```yaml
# When cancelling a revision, always document the reason
status: cancelled
notes: >
  Cancelled 2026-03-10. The business requirement this revision addressed was
  superseded by the decision to integrate with the third-party returns management
  platform (see revision order-management-rev-5). Functionality is now covered
  by that integration.
```

---

## Revision Dependencies

When this revision logically depends on another being completed first, list the prerequisite revision IDs in the `dependencies` array.

**When to use `dependencies`:**
- The happy path of this revision relies on data, entities, or capabilities introduced by another revision
- A state machine transition in this revision can only occur if the target state was introduced by another revision
- An integration in this revision calls an endpoint that does not yet exist and will be created by another revision

**When NOT to use `dependencies`:**
- When the revisions are merely related in business purpose but technically independent
- When the sequencing is a planning/prioritisation preference rather than a technical constraint

```yaml
# ✓ RIGHT — technical dependency: this revision uses the inventory reservation
# concept introduced by order-management-rev-2
dependencies:
  - order-management-rev-2
```

For soft sequencing preferences, use `notes` instead:

```yaml
notes: >
  Ideally implemented after order-management-rev-2 (inventory reservations) is
  deployed, but technically independent — can proceed in parallel if needed.
```

---

## Complete YAML Example

A complete, realistic revision covering all analytical areas. Domain: order management.

```yaml
# $schema: point this at your solution's synced local copy of revision-schema.json.
# The relative depth depends on where schemas are synced in your project layout;
# the revision file itself lives at solution-definition/analysis/revisions/{revision-id}.yaml.

revision-id: order-management-rev-1
feature-id: order-management
revision-number: 1
title: Customer-initiated order cancellation with refund eligibility check
description: >
  This revision introduces the ability for authenticated customers to cancel their own
  orders from the order detail screen, subject to business rules governing cancellation
  eligibility. An order may only be cancelled when it is in 'Pending' or 'Confirmed'
  status; orders that have already been dispatched or delivered cannot be cancelled
  through this flow. Upon cancellation, the system must transition the order status to
  'Cancelled', release any reserved inventory synchronously, evaluate refund eligibility
  according to the cancellation policy, publish a domain event, and notify the customer
  of the outcome via email. This capability replaces a manual customer support workflow
  that currently accounts for 15% of support ticket volume.
rationale: >
  Customer support receives 15% of all inbound contacts about order cancellations
  handled manually by agents, with an average handling time of 8 minutes per ticket.
  Providing self-service cancellation reduces that load, improves customer satisfaction
  NPS (target: +4 points in Q3), and fulfills the self-service mandate in the product
  strategy review. There is also a compliance gap: GDPR article 17 requires that
  customers can withdraw from a transaction without contacting support.
functional-changes:
  - change-description: >
      TRIGGER: Customer clicks 'Cancel Order' on the order detail screen. The button
      is only rendered when the order status is 'Pending' or 'Confirmed' and the
      customer is the order owner.
  - change-description: >
      TRIGGER: Customer calls the cancellation REST endpoint directly with a valid
      session token. The API enforces eligibility rules independently of the UI.
  - change-description: >
      HAPPY PATH:
      (1) Customer opens their order detail screen and clicks 'Cancel Order'.
      (2) A confirmation modal appears with an optional reason field (max 500 chars)
          and 'Confirm Cancellation' / 'Go Back' actions.
      (3) Customer optionally enters a reason and clicks 'Confirm Cancellation'.
      (4) System validates that the customer owns the order and that the order is
          still in 'Pending' or 'Confirmed' status.
      (5) System calls the inventory service synchronously to release reserved quantities.
      (6) System transitions order status to 'Cancelled', stores reason and timestamp.
      (7) System evaluates refund eligibility against the cancellation policy and
          stores the result ('FULL', 'PARTIAL', or 'NONE') on the order record.
      (8) System publishes an Order.Cancelled domain event (blocking — failure aborts).
      (9) System queues a cancellation confirmation email (fire-and-forget).
      (10) System creates an audit log entry.
      (11) UI closes the modal, navigates to the order list, and displays a success
           toast: "Your order has been cancelled."
  - change-description: >
      ALTERNATIVE (diverges after step 2): Customer clicks 'Go Back' in the
      confirmation modal. No API call is made. The modal closes and the customer
      remains on the order detail screen. Order is unchanged.
  - change-description: >
      ALTERNATIVE (diverges after step 3): Customer submits without a reason.
      The reason field is optional; the flow proceeds identically with a null
      reason value stored on the order record.
  - change-description: >
      EXCEPTION (recoverable): At step 4, the order status is no longer 'Pending' or
      'Confirmed' (concurrent change by another process). System returns an error:
      "This order can no longer be cancelled." UI refreshes the order detail to show
      the current status. No state change occurs.
  - change-description: >
      EXCEPTION (non-recoverable): At step 5, the inventory service is unavailable
      or returns an error. The cancellation is aborted. Order status is NOT changed.
      Customer receives: "Cancellation could not be completed. Please try again later."
      The failure and correlation ID are logged for support investigation.
  - change-description: >
      EXCEPTION (non-recoverable): At step 8, the Order.Cancelled event cannot be
      published. The status transition is rolled back (order status reverts). Customer
      receives a generic error. This is non-fire-and-forget for data consistency.
  - change-description: >
      EXCEPTION: Customer's session is expired or invalid. API returns HTTP 401.
      UI redirects to the login screen.
  - change-description: >
      BUSINESS RULE BR-1: An order may only be cancelled by the customer who placed
      it. Attempting to cancel another customer's order returns HTTP 403.
  - change-description: >
      BUSINESS RULE BR-2: Cancellation is only permitted when order status is 'Pending'
      or 'Confirmed'. Orders in 'Dispatched', 'Delivered', or 'Cancelled' status cannot
      be cancelled. Violation: HTTP 422 with message indicating ineligible status.
  - change-description: >
      BUSINESS RULE BR-3: Inventory reservation must be released before the order
      status is committed as 'Cancelled'. If the inventory service call fails, the
      cancellation is aborted and the order status is unchanged.
  - change-description: >
      BUSINESS RULE BR-4: Refund eligibility is determined at the moment of cancellation:
      within 1 hour of placement → FULL refund; after 1 hour but before dispatch →
      PARTIAL (minus processing fee); after 24 hours in 'Confirmed' status → NONE.
      The applicable policy version must be stored on the order record.
  - change-description: >
      VALIDATION: The cancellation reason field is optional. If provided, it must not
      exceed 500 characters. Consequence: inline validation error displayed beneath the
      field before submission; form is blocked while the constraint is violated.
  - change-description: >
      VALIDATION: The order-id path parameter must refer to an existing order owned
      by the authenticated customer. Non-existent order: HTTP 404. Order belonging to
      another customer: HTTP 403.
  - change-description: >
      SYSTEM BEHAVIOR: Steps must execute in this order for data integrity:
      (1) eligibility validation, (2) inventory service call (abort on failure),
      (3) status transition and reason storage, (4) refund eligibility evaluation
      and storage, (5) publish Order.Cancelled event (abort on failure and revert
      status), (6) queue cancellation email (fire-and-forget). Correlation ID must
      propagate through all steps.
  - change-description: >
      STATE CHANGE: Order transitions from 'Pending' → 'Cancelled' or 'Confirmed' →
      'Cancelled'. Triggering action: customer confirms cancellation and all blocking
      steps succeed. Who can trigger: the order owner (Customer role). Side-effects:
      inventory reservation released; Order.Cancelled event published.
  - change-description: >
      INTEGRATION: Inventory service is called synchronously (blocking) to release
      reservation. If the call fails or times out (10 seconds), the cancellation is
      aborted and the order status is unchanged.
  - change-description: >
      INTEGRATION: Order.Cancelled domain event published to the Azure Service Bus
      notification topic after persistence. This is blocking (non-fire-and-forget):
      failure reverts the status transition.
  - change-description: >
      NOTIFICATION: Transactional cancellation confirmation email sent to the
      customer's registered address after successful cancellation. Content must convey:
      order reference, cancellation date/time, itemised order summary, refund
      eligibility outcome with next steps (e.g., expected refund timeline). Sent
      asynchronously — failure does not reverse the cancellation.
  - change-description: >
      AUDIT: Activity log entry created after status transition with: actor (customer
      ID + session reference), action 'ORDER_CANCELLED', entity type 'Order', entity
      ID, timestamp (ISO 8601 UTC), cancellation reason (if provided), refund
      eligibility result, and correlation ID. Retention: 7 years per financial
      compliance policy.
  - change-description: >
      UI: 'Cancel Order' button displayed in the actions panel on the order detail
      screen. Visible only when status is 'Pending' or 'Confirmed'. Clicking opens
      a confirmation modal (not a new page). Modal contains: order summary (reference
      + total), optional reason textarea (max 500 chars), 'Confirm Cancellation'
      (primary/destructive) and 'Go Back' (secondary) buttons. While API call is in
      flight: 'Confirm' button shows a loading spinner and is disabled. On success:
      modal closes, order list is shown with success toast. On error: modal remains
      open with inline error message.
acceptance-criteria:
  - "Given a customer with an order in 'Pending' status, when they confirm cancellation, then the order status changes to 'Cancelled' and a success toast is displayed."
  - "Given a customer with an order in 'Confirmed' status, when they confirm cancellation, then the order status changes to 'Cancelled' and a success toast is displayed."
  - "Given an order in 'Dispatched' status, when the customer views the order detail, then the 'Cancel Order' button is not rendered."
  - "Given an order in 'Delivered' status, when the customer views the order detail, then the 'Cancel Order' button is not rendered."
  - "Given a customer attempting to cancel another customer's order via the API, then the response is HTTP 403."
  - "Given the cancellation reason exceeds 500 characters, when the customer attempts to submit, then an inline validation error is shown and the form cannot be submitted."
  - "Given the inventory service returns a 500 error during cancellation, then the order status remains unchanged and the customer receives an error message."
  - "Given a successful cancellation, then an Order.Cancelled event is published to the notification topic."
  - "Given a successful cancellation, then the inventory reservation is released in the inventory service."
  - "Given a successful cancellation, then a cancellation confirmation email is sent to the customer's registered address within 60 seconds."
  - "Given a successful cancellation, then an audit log entry exists with actor, action 'ORDER_CANCELLED', entity ID, timestamp, and correlation ID."
  - "Given a cancellation within 1 hour of order placement, then the refund eligibility result stored on the order record is 'FULL'."
dependencies: []
status: ready-for-story-creation
tags:
  team: order-management
  domain: commerce
  sprint: "2026-Q1-S3"
created-by: bart.tenvelde@maxqlabs.be
created-at: "2026-01-15T10:00:00Z"
updated-at: "2026-01-20T14:30:00Z"
notes: >
  Actor context: only the Customer (owner) role is covered. Back-office admin
  cancellation (acting on behalf of a customer) is out of scope and will be
  addressed in order-management-rev-3. Unauthenticated access is not permitted
  for any operation in this revision.
```

---

## Related Guidelines

- [Glossary Schema](../schemas/glossary-schema.json) — Canonical shape for business-term entries referenced by the Terminology and Glossary Alignment section
