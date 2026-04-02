# Usage Examples

## Example 1: Add a feature to existing project

```
> /workflow-feature Add bulk shipment endpoint that accepts up to 100 shipments

Phase 1: Analysis
─────────────────
I've read the codebase. ShipmentController exists, Wolverine is used for async.

Understanding Summary:
- What: POST /api/v1/shipments/bulk accepting up to 100 shipments
- Why: Clients need to create multiple shipments in one API call
- Who: API consumers (e-commerce platforms)
- Scope: New endpoint + service + validation
- Not included: UI changes, carrier-specific batching
- Constraints: Rate limiting, max 100 per request

> Does this accurately reflect your intent?

You: approve

Phase 2: Architecture
─────────────────────
No new macro architecture decisions needed.
Existing stack applies: Wolverine for async, PostgreSQL, existing carrier adapters.

You: skip

Phase 3: Design
───────────────
Domain Map: Shipment subdomain (Major), Carrier (Minor), Organization (None)

New entity: BulkShipmentRequest (id, status, items_count, ...)
New endpoint: POST /api/v1/shipments/bulk
  Request: { shipments: [...] }
  Response: { success: true, data: { request_id, results: [...] } }

You: approve

Phase 4: Decomposition
──────────────────────
Epic: Bulk Shipment

Story 1: BulkShipment entity + migration     | Wave A
Story 2: BulkShipmentService                  | Wave B ← (Story 1)
Story 3: BulkShipmentController + validation  | Wave B ← (Story 1)
Story 4: Wolverine async handler              | Wave C ← (Story 2, 3)

You: approve

Phase 5: Validation ......... PASS
Phase 6: Implementation ..... 4 stories across 3 waves
Phase 7: Review ............. SHIP (0 critical, 0 high)
Phase 8: Documentation ...... 1 endpoint documented
Phase 9: Deploy ............. PR created, archived

✓ Feature complete.
```

---

## Example 2: New project from scratch

```
> /workflow-full Build a webhook relay service that receives carrier webhooks
  and forwards them to customer endpoints with retry logic

Phase 1: Analysis
─────────────────
This is a greenfield project. Let me research the domain...
[spawns researcher agent for stack/patterns/pitfalls]

Approaches:
A: Minimal — Express + PostgreSQL + simple retry queue
B: Event-driven — Fastify + Redis Streams + dead letter queue
C: Serverless — AWS Lambda + SQS + DynamoDB

My recommendation: B — best balance of reliability and simplicity.

You: go with B

Phase 2: Architecture
─────────────────────
ADR-1: Use Fastify over Express (faster, schema validation built-in)
ADR-2: Use Redis Streams for webhook queue (reliable, ordered, consumer groups)
ADR-3: Use PostgreSQL for webhook logs and customer configs

You: approve

Phase 3-9: [continues through all phases, no skipping]
```

---

## Example 3: Run a single phase

```
> /analyze Refactor the credential encryption to use AES-256-GCM instead of AES-256-CBC

[runs only Phase 1 — analysis, research, questions, approaches]
[does NOT auto-continue to Phase 2]

Analysis saved to .planning/active/01-analysis/.
Chosen approach: In-place migration with backward compatibility.

> approve

To continue: /architect
```

---

## Example 4: Resume interrupted workflow

```
> /workflow-feature continue

Reading STATE.md...
Feature: Bulk Shipment Endpoint
Last completed: Phase 4 (Decomposition)
Resuming from: Phase 5 (Validation)

Phase 5: Validation
───────────────────
[continues from where you left off]
```
