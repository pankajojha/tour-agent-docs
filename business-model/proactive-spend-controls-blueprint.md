# Proactive Spend Controls — Implementation Blueprint

## Vision
Add company travel policy once.
Travelers see their personalized limits and options automatically.
Dynamic policies adapt to market conditions without constant admin oversight.

---

## System Overview

```
Admin Portal
    |
    | define / version policy rules
    v
Policy Store (DB: company_policies, policy_rules, policy_versions)
    |
    | policy snapshot at trip creation
    v
Policy Engine (tour-agent/services/policy)
    |
    | enforced at every search/quote/booking call
    v
Specialist Agents (hotel, flight, car, restaurant/meal)
    |
    | annotated results with compliance status
    v
UI (personalized limits banner + compliant-first ordering)
```

---

## Phase 1: Static Deterministic Policy Engine

### 1.1 Policy Data Model (V003 Migration)

Add to `tour-agent/infrastructure/database/migrations/V003__policy_engine.sql`:

```sql
-- Policy owner scope
CREATE TABLE company_policies (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    company_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    scope VARCHAR(50) NOT NULL CHECK (scope IN ('company', 'department', 'role', 'traveler_tier', 'geography')),
    effective_from DATE NOT NULL,
    effective_until DATE,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    version INTEGER NOT NULL DEFAULT 1,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Individual enforceable rules
CREATE TABLE policy_rules (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    policy_id UUID NOT NULL REFERENCES company_policies(id) ON DELETE CASCADE,
    category VARCHAR(50) NOT NULL CHECK (category IN ('hotel', 'flight', 'car', 'meal', 'advance_booking', 'preferred_vendor')),
    rule_key VARCHAR(100) NOT NULL,
    rule_value JSONB NOT NULL,
    -- Examples:
    --   hotel: {"max_nightly_rate": 250, "currency": "USD", "city_tier_overrides": {"NYC": 350}}
    --   flight: {"max_cabin_class": "economy", "advance_booking_days": 14}
    --   car: {"max_car_class": "standard", "preferred_vendors": ["Hertz", "Enterprise"]}
    --   meal: {"daily_allowance": 75, "currency": "USD", "per_meal_caps": {"breakfast": 20, "lunch": 25, "dinner": 30}}
    enforcement_level VARCHAR(50) NOT NULL DEFAULT 'hard' CHECK (enforcement_level IN ('hard', 'soft', 'advisory')),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Immutable audit log of every policy evaluation
CREATE TABLE policy_evaluations (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    trip_id UUID NOT NULL REFERENCES trip_plans(id) ON DELETE RESTRICT,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    policy_id UUID REFERENCES company_policies(id),
    category VARCHAR(50) NOT NULL,
    input_snapshot JSONB NOT NULL,          -- what was evaluated
    result VARCHAR(50) NOT NULL CHECK (result IN ('compliant', 'non_compliant', 'advisory', 'override_pending', 'override_approved')),
    violations JSONB DEFAULT '[]',          -- [{rule_key, rule_value, actual_value, message}]
    allowed_alternatives JSONB DEFAULT '[]',
    policy_version INTEGER NOT NULL,
    dynamic_adjustment JSONB,               -- null if static rule applied
    correlation_id VARCHAR(255),
    evaluated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Override requests for non-compliant bookings
CREATE TABLE policy_overrides (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    evaluation_id UUID NOT NULL REFERENCES policy_evaluations(id) ON DELETE RESTRICT,
    trip_id UUID NOT NULL REFERENCES trip_plans(id) ON DELETE RESTRICT,
    requester_user_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    approver_user_id UUID REFERENCES users(id),
    reason_code VARCHAR(100) NOT NULL,
    reason_note TEXT,
    status VARCHAR(50) NOT NULL DEFAULT 'pending' CHECK (status IN ('pending', 'approved', 'denied', 'expired')),
    requested_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    resolved_at TIMESTAMP WITH TIME ZONE,
    sla_deadline TIMESTAMP WITH TIME ZONE
);
```

---

### 1.2 Policy Engine Service Contract

Add `tour-agent/services/policy/evaluate.go` (following Go service pattern from auth):

**Request:**
```json
{
  "user_id": "uuid",
  "trip_id": "uuid",
  "category": "hotel",
  "input": {
    "nightly_rate": 310,
    "currency": "USD",
    "city": "New York",
    "vendor": "Marriott",
    "check_in": "2026-05-10",
    "check_out": "2026-05-13"
  },
  "correlation_id": "abc-123"
}
```

**Response:**
```json
{
  "result": "non_compliant",
  "violations": [
    {
      "rule_key": "max_nightly_rate",
      "rule_value": 250,
      "actual_value": 310,
      "message": "Exceeds nightly hotel cap by $60 (NYC tier cap is $350, base cap is $250)"
    }
  ],
  "allowed_alternatives": [
    {"vendor": "Hilton Garden Inn", "nightly_rate": 229, "compliant": true},
    {"vendor": "Hyatt Place", "nightly_rate": 245, "compliant": true}
  ],
  "policy_version": 3,
  "override_required": false,
  "dynamic_adjustment": null
}
```

**Enforcement levels:**
- `hard` — block booking, require override
- `soft` — allow but surface warning, log violation
- `advisory` — show tip, no friction

---

### 1.3 Specialist Agent Integration

Each existing specialist agent (hotel, flight, car, budget) calls the Policy Engine before returning results.

Insert after every quote or recommendation call:

```python
# In hotel_agent.py, flight_agent.py, car_agent.py
policy_result = await call_policy_evaluate(
    user_id=context["user_id"],
    trip_id=context["trip_id"],
    category="hotel",
    input=quote_result,
    correlation_id=context["correlation_id"]
)
quote_result["policy"] = policy_result
```

Return shape always includes a `policy` block:

```json
{
  "hotel": { "name": "...", "nightly_rate": 310 },
  "policy": {
    "result": "non_compliant",
    "violations": [...],
    "allowed_alternatives": [...],
    "override_required": false
  }
}
```

---

## Phase 2: Personalized Limits + Compliant-First UX

### 2.1 Personalized Limits Endpoint

`GET /policy/limits?user_id=&trip_id=&category=hotel&city=New+York`

Response:
```json
{
  "category": "hotel",
  "city": "New York",
  "limits": {
    "max_nightly_rate": 350,
    "currency": "USD",
    "preferred_vendors": ["Marriott", "Hilton", "Hyatt"],
    "advance_booking_required_days": 7
  },
  "scope": "city_tier_override",
  "policy_version": 3
}
```

UI uses this to show a banner:
> "Your hotel limit in New York is $350/night. 3 compliant options available."

### 2.2 Compliant-First Result Ordering

Agent search results are sorted by:
1. Compliant + preferred vendor first
2. Compliant + non-preferred vendor
3. Advisory violations
4. Hard violations (shown last, flagged, override CTA)

---

## Phase 3: Dynamic Policy Layer

### 3.1 Dynamic Adjustment Rules (admin-approved only)

Extend `policy_rules.rule_value` JSONB to support dynamic clauses:

```json
{
  "max_nightly_rate": 250,
  "currency": "USD",
  "dynamic": {
    "enabled": true,
    "strategy": "market_median_uplift",
    "uplift_cap_pct": 20,
    "market_data_source": "internal_benchmark",
    "hard_ceiling": 400
  }
}
```

Dynamic adjustment logic:
- If market median for city + date window exceeds base cap × (1 + uplift_cap_pct/100), allow up to `hard_ceiling`.
- Never exceed `hard_ceiling` regardless of market data.
- Log every dynamic adjustment to `policy_evaluations.dynamic_adjustment`.

### 3.2 Feature Flag Gating

All dynamic logic behind a feature flag to allow safe rollout:

```json
{
  "flags": {
    "dynamic_policy_hotel": false,
    "dynamic_policy_flight": false,
    "dynamic_policy_car": false,
    "dynamic_policy_meals": false
  }
}
```

Start all flags `false`. Enable per-category in order: hotel → flight → car → meals.

---

## Phase 4: Audit + Insights

### 4.1 What Gets Logged Per Decision
- policy_version at evaluation time
- matched rule keys and values
- actual submitted values
- dynamic adjustment inputs and outputs (if applied)
- final result
- correlation_id linking to agent run and trip

### 4.2 Insights Queries (operations/finance)

```sql
-- Compliance rate by category this month
SELECT category, result, COUNT(*) as decisions
FROM policy_evaluations
WHERE evaluated_at >= date_trunc('month', now())
GROUP BY category, result;

-- Top policy violations by rule
SELECT violations->0->>'rule_key' AS rule, COUNT(*) as frequency
FROM policy_evaluations
WHERE result = 'non_compliant'
  AND evaluated_at >= now() - interval '30 days'
GROUP BY rule
ORDER BY frequency DESC;

-- Override approval rate
SELECT
  COUNT(*) FILTER (WHERE status = 'approved') AS approved,
  COUNT(*) FILTER (WHERE status = 'denied') AS denied,
  COUNT(*) FILTER (WHERE status = 'pending') AS pending
FROM policy_overrides
WHERE requested_at >= now() - interval '30 days';
```

---

## Rollout Plan

| Phase | What | Feature Flag | Risk |
|-------|------|-------------|------|
| 1a | DB schema + policy authoring API | always on | low |
| 1b | Hotel policy enforcement (hard + soft) | `policy_hotel_v1` | low |
| 1c | Flight policy enforcement | `policy_flight_v1` | low |
| 1d | Car + meal enforcement | `policy_car_meal_v1` | low |
| 2 | Personalized limits banner in UI | `policy_limits_ui` | low |
| 2 | Compliant-first ordering | `compliant_first_sort` | low |
| 3 | Dynamic hotel cap adjustment | `dynamic_policy_hotel` | medium |
| 3 | Dynamic flight adjustment | `dynamic_policy_flight` | medium |
| 4 | Admin insights dashboard | always on | low |

---

## Implementation Mapping (Current Repo)

| Component | Location |
|-----------|----------|
| DB schema | `tour-agent/infrastructure/database/migrations/V003__policy_engine.sql` |
| Policy evaluate service | `tour-agent/services/policy/` |
| Agent policy integration | `tour-planner-ai/tourplannerai-api/src/agents/` (hotel, flight, car, budget) |
| Policy limits endpoint | `tour-agent/services/policy/limits.go` |
| Personalized limits UI | `tour-planner-ui/src/planning/` |
| Feature flags config | `tour-agent/services/config/feature_flags.json` |
| Audit queries | `tour-agent/infrastructure/database/` |

---

## Decision Checklist

- Confirm enforcement levels per category: hotel, flight, car, meals.
- Confirm department/role scoping needed for V1 or flatten to company-wide first.
- Confirm whether dynamic policy (Phase 3) is in scope for initial delivery.
- Confirm override approval flow: self-approve, manager-approve, or auto-approve with audit?
- Confirm SLA window for override approvals.
- Confirm UI surface: inline result labels only, or separate policy summary screen?
