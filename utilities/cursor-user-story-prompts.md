# Cursor Prompts: User Stories for Dev Teams
> BDD · Microservices · Event-Driven · Data Pipelines · SAFe
> Practical templates — skip sections that don't apply to your team.

---

## 🚦 Which prompt do I use?

| Situation | Use |
|---|---|
| New feature, not sure where it fits | **Foundational** |
| Building/changing a REST or gRPC endpoint | **Microservice API** |
| Producing or consuming a Kafka/SQS/EventBridge event | **Event-Driven** |
| Building/changing a pipeline, dbt model, or stream job | **Data Pipeline** |
| Need a `.feature` file for automated tests | **BDD Feature File** |
| Auth, PII, audit, access control | **Security** |
| Spike, tech debt, infra work | **Enabler / Spike** |
| Story is too big for one sprint | **Story Split** |
| Reviewing someone's ACs in PR/refinement | **AC + INVEST Review** |

---

## 🧱 1. FOUNDATIONAL PROMPT

```
You are a senior BA/dev working in a SAFe Agile team. Write a practical user story
a developer can pick up in the next sprint — not a textbook example.

Requirement: [PASTE HERE]
Team context: [e.g., payments service, owns Postgres + Kafka producer]

Output:
- Title (short, action-oriented)
- As a / I want / So that
- Acceptance Criteria (3–5 Gherkin scenarios, focus on what QA will actually test)
- Out of scope (1–3 bullets — explicitly call out what this story does NOT cover)
- Dependencies (other teams, services, or tickets)
- Open questions for refinement

Rules:
- Do NOT invent requirements not implied by the input — list them as open questions instead.
- Skip NFRs unless the requirement implies them.
- No story points — team estimates in refinement.
```

**Few-shot example:**
> Input: *"Users should be able to reset their password via email."*
> Output:
> - **Title**: Self-service password reset via email
> - **As a** registered user, **I want** to reset my password through an email link, **so that** I can regain access without contacting support.
> - **AC1**: Given a registered email, when I request reset, then a one-time link valid for 30 min is sent.
> - **AC2**: Given an expired/used link, when I open it, then I see "link expired" and can request a new one.
> - **AC3**: Given an unregistered email, when I request reset, then I see the same success message (no account enumeration).
> - **Out of scope**: SMS reset, security questions, admin-initiated reset.
> - **Open questions**: Reuse existing SendGrid template? Rate limit per IP?

---

## 🔬 2. MICROSERVICE API PROMPT

```
Write a user story for a microservice API change.

Service: [name]
Endpoint: [e.g., POST /v1/orders]
Change type: [new | modify | deprecate]
Callers: [known consumers, or "unknown — internal gateway only"]

Output:
- Story (As a / I want / So that — actor is usually a consuming service or client app)
- Gherkin AC for: happy path, validation failure (400), auth failure (401/403),
  downstream timeout/failure behavior
- Request/response shape (just the fields, not full OpenAPI)
- Backward compatibility note (breaking? versioned? deprecation timeline?)
- DoD: contract test updated, OpenAPI spec updated, logged + traced

Skip these unless explicitly relevant:
- Specific p99 latency numbers (team SLOs usually apply by default)
- Saga/compensating transactions (only mention if multi-service write)
- Rate limits (only if endpoint is public or known hot path)
```

---

## ⚡ 3. EVENT-DRIVEN STORY PROMPT

```
Write a user story for producing or consuming an event.

Broker: [Kafka | SQS | EventBridge | Pub/Sub]
Direction: [produce | consume | both]
Event name: [e.g., OrderPlaced, PaymentFailed]
Topic/Queue: [name if known]

Output:
- Story format
- Gherkin AC for:
  - Happy path
  - Duplicate delivery (consumer must be idempotent — most brokers are at-least-once)
  - Poison message → DLQ after N retries
  - Schema field added (backward compatible) — consumer keeps working
- Event payload shape (key fields only, not full Avro/JSON Schema)
- DoD: schema registered (if using registry), consumer is idempotent,
  DLQ alerting wired up

Be realistic:
- Do NOT claim "exactly-once". Kafka has effectively-once via idempotent
  producers + idempotent consumers. Most teams just do at-least-once + idempotency.
- Skip "event replay" AC unless the team actually has replay tooling.
- Skip "out-of-order" AC unless ordering matters for this event.
```

---

## 📊 4. DATA PIPELINE STORY PROMPT

```
Write a user story for a data pipeline change.

Pipeline type: [batch | streaming | CDC]
Tool: [dbt | Airflow | Spark | Flink | etc.]
Source → Destination: [e.g., orders_raw → orders_daily_agg]
Trigger: [schedule | event | manual]

Output:
- Story (actor is usually a downstream analyst, ML user, or business team)
- Gherkin AC for:
  - Pipeline runs on schedule/trigger and writes expected output
  - Late-arriving data is handled (batch: included in next run | streaming: watermark)
  - Schema change in source → pipeline fails loudly or adapts (state which)
  - Backfill: can re-run for a past date range without duplicates
- Data contract (input columns, output columns, key transformations in 1-2 lines)
- DoD: dbt tests / data quality checks added, Airflow alert on failure,
  runbook entry, lineage visible in catalog (if your org has one)

Skip unless relevant:
- PII masking (only if PII columns present)
- SLA numbers (use team default; only call out if business demands stricter)
- Reconciliation reports (only for financial/regulated data)
```

---

## 🥒 5. BDD FEATURE FILE PROMPT

```
Convert this user story into a Gherkin .feature file ready for
[Cucumber | Behave | SpecFlow | pytest-bdd].

Story: [PASTE]

Output a single .feature file with:
- Feature: <title> + 1-line description
- Background (only if 2+ scenarios share setup)
- 3–6 Scenarios covering happy path + key edge cases
- Use Scenario Outline with Examples table when testing same logic with different inputs
- Step phrasing should be reusable (avoid baking specific UI text into Given/When/Then
  unless the test is specifically a UI test)

Don't:
- Don't generate step definition code unless asked
- Don't add scenarios the story doesn't require — keep it tight
```

---

## 🔒 6. SECURITY / COMPLIANCE PROMPT

```
Write a security-focused user story.

Feature area: [auth | data access | audit logging | PII handling | secrets]
Data classification: [Public | Internal | Confidential | Restricted]
Regulation in scope: [SOC2 | GDPR | HIPAA | PCI | none — just internal policy]

Output:
- Story
- Gherkin AC for: authorized path works, unauthorized is rejected with right
  status code, attempt is logged
- What gets logged (who, what, when — not the sensitive payload itself)
- DoD: SAST clean, secrets not in code, dependency scan clean,
  threat reviewed with security team if Restricted data

Keep it practical:
- Don't require formal STRIDE analysis unless your org actually does it
- Don't require pen-test sign-off per story — that's a release-level gate
```

---

## 🌐 7. ENABLER / SPIKE PROMPT

```
Write an Enabler story for SAFe PI Planning or sprint backlog.

Type: [Spike (research) | Architecture | Infra | Tech Debt]
Topic: [what you're investigating or building]
Why now: [unblocks what?]

Output:
- Title and type
- Goal in 1 sentence
- Time-box (days, not story points — spikes should be capped)
- Done when: list of concrete outcomes (decision made, doc written, PoC running, benchmark numbers captured)
- What we will NOT do in this spike (prevent scope creep)
- Follow-up stories likely to come out of this

Reality check:
- Spikes >5 days usually need to be split
- Spike output is a decision/doc, not production code
```

---

## 🔁 8. STORY SPLIT PROMPT

```
This story is too big for one sprint. Split it.

[PASTE STORY]

Try these splits in order, use whichever fits:
1. Workflow steps (e.g., create → review → approve as 3 stories)
2. Happy path first, edge cases later
3. One interface/channel at a time (API first, then UI; or web first, then mobile)
4. Data variations (one entity type first, then others)
5. CRUD operations (read-only first, then writes)

Output 2–4 child stories. For each:
- Title + 1-line summary
- Which slice this represents
- Independently shippable? (yes/no — if no, flag it)
- MVP or follow-up?

Don't split into stories smaller than ~1 day of work — that's task-level.
```

---

## ✅ 9. AC + INVEST REVIEW PROMPT

```
Review this story before refinement/PR.

[PASTE STORY + AC]

Check:

INVEST on the story:
- Independent — can ship without other unmerged stories? (flag if not)
- Negotiable — leaves room for dev judgment, or over-specified?
- Valuable — clear user/business value stated?
- Estimable — enough info to estimate, or too many unknowns?
- Small — fits in a sprint?
- Testable — can QA write a test from this?

AC quality:
- Gherkin syntax valid?
- Each AC independently testable?
- Vague words ("fast", "should", "valid", "properly") — flag and suggest replacement
- Missing obvious negative case?
- Any AC that's actually a task disguised as AC? (e.g., "update the docs")

Output: pass/fail per check + specific fixes. Don't rewrite the whole story —
just point out what to change.
```

---

## 🧩 PERSONA LIBRARY (paste into any prompt)

```
- End user / Customer
- Internal user (ops, support, admin)
- Consuming service (when actor is another microservice)
- Data consumer (analyst, ML engineer, dashboard)
- On-call engineer / SRE
- Compliance / Audit reviewer
```

---

## 🏷️ STORY METADATA (lightweight)

```
| Field              | Value |
|--------------------|-------|
| Epic / Feature     |       |
| Team owner        |       |
| Linked services   |       |
| Topic / API / Table |     |
| Feature flag      |       |
| Observability     | logs / metrics / traces added? |
| Compliance flag   | PII / PCI / none |
```

---

## ⚠️ Things to push back on in refinement

- ACs written as implementation steps ("call the X API")
- Stories with no "why" — value statement missing
- "As a developer, I want..." — usually an enabler, not a user story
- Hidden scope: "and also...", "while we're at it..."
- Performance/security ACs with no numbers or threshold
- ACs the team has no way to test

---

*Drop this into `.cursor/rules/` or reference as `@prompts/stories.md` in Cursor Chat.*
