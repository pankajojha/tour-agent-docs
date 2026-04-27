# TourPlanner Confirmed Architecture And Delivery Guardrails

This document is the working reference for system architecture, data flow, security boundaries, and collaboration rules confirmed during design discussions.

## Scope

This README governs:

- UI flow and API layering
- Agentic AI integration boundaries
- Sensitive data handling and logging policy
- Async webhook handling and idempotency
- Shared-trip ownership and edit permissions
- Immutable history, retention, and audit behavior

## System Components

- UI: `tour-planner-ui`
- Core API services and DB access: `tour-agent/services`
- Agentic AI gateway/runtime services: `tour-planner-ai/tourplannerai-api`
- Primary schema baseline: `tour-agent/infrastructure/database/migrations/V001__initial_schema.sql`

## Authoritative Flow

**Primary Flow (User/Trip Data):**
1. User actions start in `tour-planner-ui`.
2. UI calls `tour-agent/services` (entry point for all client traffic).
3. Services sanitize and tokenize sensitive data before any AI call.
4. Services call the Agentic AI gateway/runtime in `tour-planner-ai/tourplannerai-api`.
5. AgentCore and Strands process redacted payloads only.
6. AI response returns to services with required AI metadata.
7. Services rehydrate as needed from token vault mappings and persist output in DB.
8. Services emit results to UI or callback pipelines.

**Agentic Data Flow (Agent State/Execution Context):**
1. Agentic runtime and services in `tour-planner-ai/tourplannerai-api` directly access agentic-scoped DB tables.
2. Direct access uses restricted IAM credentials (agentic tables only).
3. RLS ensures agent→trip ownership boundaries.
4. Agentic outputs (reasoning, state, context) are persisted as immutable agentic records.
5. Correlation IDs link agentic records to user/trip records for audit and reproducibility.

## Hard Boundary Rules

**Data Access Patterns:**
- **Agentic entries** (agent state, execution context, reasoning logs): direct access via Agentic Service to agentic-scoped DB tables.
- **User/trip data** (bookings, preferences, itineraries): all reads/writes via `tour-agent/services` only (gateway-mediated).
- Agentic Service assumes restricted IAM role with permissions scoped to agentic tables only.
- Row-level security (RLS) enforces agent→trip ownership associations in agentic tables.

**Data Handling:**
- AI layer receives IDs and tokenized values, not raw PII or PCI.
- Services (both gateway and agentic) are responsible for tokenization/detokenization and final response stitching.
- Sensitive data vault lookups remain trip-scoped and application-controlled.

## API Versioning And Rollout

- Services own API version management.
- Initial rollout model is synchronous.
- UI-impacting service changes can roll immediately when safe.
- Version bumps are strongly expected when contract or DB impacts exist.
- If no DB change and no breaking contract impact, UI plus services can ship without forced version increment.

## Identity, Auth, And Traceability

- Services propagate JWT-based identity.
- Services validate JWT claims before downstream execution.
- `user_id` and `trip_id` are passed through internal orchestration paths.
- Correlation ID is mandatory end-to-end:
	- UI
	- API services
	- AI gateway/runtime
	- webhook callbacks
	- persistence/audit records

## Sensitive Data Strategy

- Deterministic tokenization is required:
	- Same input maps to same token.
- Sensitive token vaulting is managed in `tour-agent/services`.
- Reverse lookup is application-controlled and trip-scoped.
- Prompt-safe logging is mandatory:
	- No pre-redacted payloads in logs, traces, or metrics.

## Async Orchestration And Webhooks

- Orchestration mode: async with webhooks.
- Webhook callbacks must be HMAC-signed.
- Replay protection is required with nonce tracking.
- Signature freshness window: 15 minutes.
- Duplicate callback behavior:
	- Signal duplicate explicitly (HTTP 409) using `correlation_id + run_id` idempotency key.
- Validation failure behavior:
	- Distinct failure messages or codes are allowed for signature, timestamp, and nonce failures.

## AI Response Metadata Requirements

Every AI response persisted by services must carry metadata sufficient for reproducibility:

- prompt version
- model version
- agent version
- run identifiers and correlation IDs

Services must persist these fields alongside immutable AI output records.

## Persistence Model For AI Outputs

- AI outputs are persisted as immutable records for legal/history usage.
- No hard-delete path for immutable records.
- Current retention policy for immutable records: 3 years.
- Schema for AI-specific storage will be introduced incrementally beyond `V001__initial_schema.sql`.

## Shared Trip Ownership And Edit Rules

### Category restriction policy

Owner manages the overall trip but category write rules are explicit:

- Restricted from owner modification:
	- Flight
	- Cruise
	- Hotel
	- Car
	- Shore excursions or third-party booked activities
- Owner-modifiable categories:
	- Budget
	- Planning/Notes

### Contributor ownership policy

- Each contributor's owned section is displayed with ownership labels.
- Contributors can edit their own data even when shared.
- Trip owner has override rights in Budget and Planning/Notes:
	- update
	- modify
	- delete contributor entries

### Deletion and restore policy

- Soft delete only. No hard delete.
- Deleted entries are hidden by default.
- UI should expose a `Show deleted` toggle.
- Only trip owner can restore deleted entries.

## Audit And Version History

Every edit, delete, and restore action must create an immutable version/audit record with:

- who
- when
- reason
- previous value
- new value

This applies to user-managed collaborative sections and to service-mediated AI output lifecycle events.

## Database Roadmap Notes

Current baseline file:

- `tour-agent/infrastructure/database/migrations/V001__initial_schema.sql`

This baseline does not yet include AI-run tables. Future migrations should introduce entities for:

- AI run registry
- immutable AI outputs
- callback/idempotency tracking
- token vault mapping metadata
- collaboration section versions and soft-delete state

## Implementation Principle

All future feature work must preserve this sequence:

1. sanitize and tokenize in services
2. invoke AI with redacted payloads only
3. capture metadata-rich immutable outputs
4. enforce scope-aware access and soft-delete semantics
5. maintain end-to-end correlation and auditable history