# Tour Planner - B2B Business Model

## Executive Summary

Tour Planner's B2B model targets corporate event coordinators, travel management companies, group organizers, cruise group operators, and enterprise hospitality partners that need structured travel coordination at scale. The B2B product is built around shareable itineraries, role-based administration, approval workflows, group booking operations, checklist management, and analytics. It includes the same AI planning intelligence used in the consumer product, including the ability to structure travel data entered as free-form text or imported from email content and Word or PDF documents. Coordinators and travelers also receive live schedule updates and more relevant recommendations as each event, departure, and itinerary milestone gets closer. Shared itinerary collaboration now supports category-level owner controls: coordinator-owned source entries remain protected from collaborator edits, while invited participants can add their own category contributions to the same trip payload for downstream planning and approvals. Revenue is driven by SaaS subscriptions, coordinator seat licenses, group-trip platform fees, vendor tooling, premium integrations, and enterprise service agreements.

Privacy and AI data protection are also part of the B2B positioning. The intended product posture is that raw PII, PCI, and other sensitive booking content are not sent directly to the AI backend. Uploaded or copied itinerary material is intended to be sanitized, obfuscated, or reduced to privacy-safe planning context before any AI-driven structuring, summarization, or recommendation workflow is executed.

### Relationship to the Overall Platform

The B2B business is the managed-travel and enterprise monetization layer of the overall Tour Planner platform. It uses the same trip, booking, collaboration, and AI infrastructure as the B2C product, but packages it for coordinators and organizations that operate at higher scale and with stricter governance requirements. In practice:

- **B2B** monetizes organizations, coordinator teams, and managed travel programs.
- **B2C** continues to serve direct travelers and self-serve trip organizers.
- Both models reinforce each other: B2C creates end-user familiarity and demand, while B2B creates higher-value contracts, operational lock-in, and larger booking volumes.

---

## Platform Overview

### Core Capabilities

The B2B platform combines travel coordination and operational management features:

- **Coordinator-Controlled Itineraries** - Centralized itinerary creation for large groups and managed travel programs
- **Free-Form and Document Intake** - Import and structure travel requests from free-form text, email, Word, and PDF sources
- **Role-Based Access Control** - Coordinator, manager, participant, finance approver, and view-only roles
- **Shareable Itineraries** - Controlled sharing across teams, departments, clients, and travelers
- **Owner-Locked Category Control** - Original owner or coordinator entries are read-only for shared participants
- **Multi-Contributor Category Inputs** - Participants can append their own category-level itinerary content without mutating owner entries
- **Approval Workflows** - Booking, budget, and itinerary changes routed through approval chains
- **Group Excursion Booking** - Aggregated booking workflows for excursions and activities
- **Trip Checklist Management** - Coordinator-managed checklists for traveler readiness, required documents, and operational tasks
- **Budget & Cost Governance** - Shared budget tracking, approvals, and optimization insights
- **AI Planning Assistant** - AgentCore-driven planning recommendations and schedule optimization
- **Live Travel Updates** - Continuous schedule updates and recommendation refreshes as trips and events approach
- **Audit Logs & Compliance** - Activity tracking for changes, approvals, and cost decisions
- **Vendor Management Tools** - Inventory, pricing, capacity, and reporting for suppliers
- **Enterprise Integrations** - Connectivity with travel, procurement, CRM, and finance systems
- **Privacy-Conscious AI Data Boundary** - Sensitive data is intended to be excluded or sanitized before AI processing

### Privacy, Compliance, and AI Guardrails

For B2B customers, privacy and control expectations are adoption requirements. The managed-travel version of Tour Planner is therefore positioned around a clear AI data boundary:

- Raw personal data such as names, email addresses, phone numbers, passport details, and other direct identifiers are intended to stay out of general AI prompts.
- Payment card data, billing details, and PCI-regulated information are intended to remain outside the AI layer entirely.
- Uploaded travel documents, copied itineraries, and booking confirmations are intended to go through sanitization and structuring steps before AI summarization or recommendation workflows.
- The AI layer is intended to operate on minimum-necessary, privacy-safe travel context such as itinerary structure, destination, timing, generalized booking metadata, and traveler counts.
- Sensitive transactional systems remain the source of truth for booking state, payment state, and identity-sensitive workflows.

This direction supports enterprise conversations around GDPR-conscious processing boundaries, privacy-by-design expectations, and ISO 27001-style operating controls. It also strengthens the platform's enterprise positioning by giving buyers a concrete answer to how AI is constrained around sensitive travel data and how collaboration permissions protect source itinerary ownership while still supporting multi-party input.

Planned production guardrails for the AI agent solution include:

- server-enforced sanitization before AI invocation
- formal sensitive-data classifiers and deny rules for prompts
- centralized audit logging for AI-bound payloads
- role-based access controls for itinerary review and sharing
- retention and deletion workflows for uploaded travel documents
- production security review of AI integrations and data flows

### Target Customers

1. **Corporate Event Coordinators** - Offsites, retreats, incentive trips, and customer events
2. **Travel Management Companies (TMCs)** - Multi-client group travel operations
3. **Cruise Group Organizers** - Charter groups, affinity groups, and hosted cruise programs
4. **Hospitality and Destination Management Partners** - Companies coordinating local activities and logistics

---

## Primary Revenue Streams

### 1. Enterprise SaaS Subscriptions

**Model:** Recurring SaaS subscriptions based on organization size and feature depth

#### Team Plan
**Price:** $299-$799/month
- 3-10 coordinator seats
- Up to 20 active trips
- Role-based itinerary management
- Owner-protected category source records
- Participant contribution entries in shared category payloads
- AI-assisted trip structuring from free-form text and imported email, Word, and PDF content
- Privacy-conscious AI handling posture with sensitive-data sanitization before planning workflows
- Shared checklist tracking for coordinators and travelers
- Shared traveler portal
- Standard reporting and exports

#### Business Plan
**Price:** $1,000-$2,500/month
- 10-50 coordinator seats
- Unlimited active trips
- AI planning assistance and schedule optimization across managed trips
- Live updates and time-sensitive recommendations as itineraries get closer
- Approval workflows and budget governance
- Stronger privacy and governance posture for AI-assisted itinerary handling
- Checklist oversight for trip readiness and task completion
- Advanced reporting
- Vendor coordination tools
- Branded traveler experience

#### Enterprise Plan
**Price:** $3,000-$10,000+/month
- Unlimited seats
- AI-assisted planning across enterprise travel programs
- SSO, audit logs, API access, and white-label options
- Custom workflows and data retention controls
- AI guardrail alignment with enterprise privacy and security expectations
- Enterprise checklist and readiness workflows
- Dedicated onboarding and success support
- Procurement and travel-system integrations

**Projected Revenue Contribution:** 35-45% of total B2B revenue

---

### 2. Group Trip Platform Fees

**Model:** Per-trip or per-participant fees for managed group travel

**Pricing Structure:**
- Small managed group: $100-$300 per trip
- Mid-size event group: $500-$2,000 per trip
- Large enterprise event: $2,500-$10,000+ per trip

**Value Drivers:**
- Centralized coordination reduces manual planning overhead
- Shared itinerary portal improves traveler communication
- Coordinator workflows reduce booking errors and missed changes

**Projected Revenue Contribution:** 20-25% of total B2B revenue

---

### 3. Coordinator Seat Licensing

**Model:** Per-seat licensing for internal planners and event operators

**Example Pricing:**
- Standard coordinator seat: $49-$99/month
- Advanced operations seat: $149-$249/month
- Finance/approver seat: $29-$59/month

**Use Cases:**
- Regional event teams
- Agency account managers
- Travel operations staff
- Finance and procurement reviewers

**Projected Revenue Contribution:** 10-15% of total B2B revenue

---

### 4. Group Booking & Excursion Commissions

**Model:** Commission on group excursion and activity bookings managed through the platform

**Commission Structure:**
- Standard excursions: 15-25%
- Premium private experiences: 10-20%
- Group-negotiated packages: 8-15% plus service fees

**Why B2B Wins Here:**
- Larger booking volumes
- Better negotiated supplier pricing
- Stronger conversion from coordinator-led demand

**Projected Revenue Contribution:** 15-20% of total B2B revenue

---

## Secondary Revenue Streams

### 5. Integration and Implementation Services

**Model:** One-time and recurring services for onboarding and systems integration

**Services:**
- Procurement and travel system integration
- SSO and identity setup
- Data migration from spreadsheets or legacy tools
- Workflow configuration and policy setup
- Team onboarding and training

**Pricing:**
- Standard onboarding: $2,500-$10,000
- Custom integration package: $10,000-$75,000+

**Projected Revenue Contribution:** 5-10% of total B2B revenue

---

### 6. Premium Analytics and Compliance Modules

**Model:** Paid add-on modules for enterprise reporting, governance, and optimization

**Modules:**
- Spend analytics and budget variance reporting
- Traveler engagement reporting
- Compliance reporting and audit exports
- Supplier performance scorecards
- AI optimization insights for cost and schedule efficiency
- AI data-boundary reporting and privacy-control visibility for enterprise customers

**Pricing:**
- Analytics add-on: $500-$1,500/month
- Compliance add-on: $1,000-$3,000/month

**Projected Revenue Contribution:** 5-10% of total B2B revenue

---

### 7. Vendor SaaS and Supply-Side Tools

**Model:** Subscription products for suppliers supporting B2B-managed travel demand

**Pricing:**
- Vendor Pro: $99-$299/month
- Vendor Enterprise: $500-$2,000/month

**Features:**
- Group inventory controls
- Capacity and contract management
- Dynamic pricing support
- Group quote workflows
- Performance and payout reporting

**Projected Revenue Contribution:** 5-10% of total B2B revenue

---

## Revenue Breakdown Summary

### Estimated Revenue Mix

| Revenue Source | % of Total | Notes |
|---|---|---|
| Enterprise SaaS | 35-45% | Core recurring revenue base |
| Group Trip Platform Fees | 20-25% | Per-trip and per-participant pricing |
| Coordinator Seat Licensing | 10-15% | Expansion revenue inside accounts |
| Group Booking Commissions | 15-20% | Excursion and activity margins |
| Integration Services | 5-10% | Onboarding and custom work |
| Analytics & Compliance Add-Ons | 5-10% | Premium enterprise modules |
| Vendor SaaS | 5-10% | Supply-side monetization |
| **Total** | **100%** | |

---

## Unit Economics

### Sample Customer: Mid-Market Corporate Retreat Program

**Scenario:**
- 1 business customer
- 15 coordinator seats
- 8 managed retreats per year
- 250 travelers annually
- One analytics add-on

**Revenue Breakdown:**

| Item | Amount | Annual Revenue to Platform |
|---|---|---|
| Business subscription | $1,500/month | $18,000 |
| 15 coordinator seats @ $79/month | $1,185/month | $14,220 |
| 8 managed retreats @ $1,000 | - | $8,000 |
| Group booking commissions | - | $20,000 |
| Analytics add-on | $750/month | $9,000 |
| **Total Annual Account Revenue** | - | **$69,220** |

**Economics:**
- **Gross margin:** 80-90%
- **Estimated CAC:** $8,000-$20,000 depending on segment
- **Payback period:** 6-12 months
- **3-year account value:** $200,000+ for retained customers

---

## Growth Projections (5-Year)

### Year-by-Year Revenue Forecast

| Year | Accounts | Avg Rev/Account | Total Revenue | Managed Traveler Volume |
|---|---|---|---|---|
| Y1 | 50 | $25,000 | $1.25M | 20,000 |
| Y2 | 150 | $35,000 | $5.25M | 75,000 |
| Y3 | 400 | $50,000 | $20M | 250,000 |
| Y4 | 900 | $70,000 | $63M | 700,000 |
| Y5 | 1,800 | $95,000 | $171M | 1.8M |

**Assumptions:**
- Expansion from mid-market to enterprise accounts over time
- Rising account revenue through seats, modules, and trip volume
- Strong net revenue retention through cross-sell and add-ons

### Roll-Up Assumptions for Combined Company Model

The B2B forecast is intended to roll into a combined company model alongside the B2C forecast. To maintain a clean consolidated model, these assumptions should apply:

- **B2B revenue includes only managed, coordinator-led, or contracted business** and excludes self-serve consumer bookings.
- **Group booking commissions generated inside B2B-managed programs should be counted in B2B only**, even when the end travelers are individuals.
- **Implementation and integration revenue is B2B-only** and should not appear in consumer model totals.
- **Vendor SaaS revenue should be assigned once** in the consolidated model based on the commercial motion that closes and owns the account.
- **Managed traveler volume should be tracked separately from B2C user counts** to avoid mixing account-based and consumer metrics.
- **Shared platform infrastructure and R&D should be modeled centrally** instead of being embedded directly in B2B segment margins.

---

## Market Opportunity

### Total Addressable Market (TAM)

**Corporate and Managed Group Travel Software:**
- Corporate retreat and offsite planning market
- Travel management and event operations software
- Group excursion and destination activity coordination
- **TAM:** $8-$12 billion annually

### Serviceable Addressable Market (SAM)

**Initial focus:**
- North American and English-speaking corporate travel buyers
- Mid-market event teams and travel agencies
- Cruise and destination group operators
- **SAM:** $2-$4 billion

### Serviceable Obtainable Market (SOM)

**Year 5 target:** $150M-$250M revenue opportunity with focused segment penetration

---

## Go-to-Market Strategy

### Phase 1: Design Partner Sales (Months 1-6)

**Target:** Event coordinators, boutique agencies, and destination management companies
- Sell into operations-heavy teams still using spreadsheets and email
- Prioritize customers with repeat group travel programs
- Offer concierge onboarding to shape enterprise workflows

**Channels:**
- Founder-led sales
- LinkedIn outbound to event and travel operations leaders
- Partnerships with destination management companies
- Industry referrals and design-partner incentives

### Phase 2: Mid-Market Expansion (Months 7-18)

**Expand to:**
- Travel management companies
- Corporate retreat and offsite planners
- Cruise group operators

**Sales Motion:**
- Case studies and ROI selling
- Partner channels with agencies and hospitality groups
- Integration-led selling for procurement and travel systems

### Phase 3: Enterprise Scale (Months 19-36)

**Enterprise focus:**
- Security, compliance, and approval workflows
- Procurement-led sales cycles
- Multi-department rollouts
- White-label and API offerings

---

## Competitive Differentiation

| Feature | Tour Planner B2B | Competitors |
|---|---|---|
| **Coordinator-Controlled Itineraries** | ✅ Built for managed travel | ❌ Consumer-first tools |
| **Role-Based Access Control** | ✅ Multi-role governance | ❌ Limited permissions |
| **Approval Workflows** | ✅ Budget and booking approvals | ❌ Manual processes |
| **Group Excursion Operations** | ✅ Native booking workflows | ❌ Not purpose-built |
| **AI Planning Assistant** | ✅ Embedded AgentCore workflows | ❌ Basic automation |
| **Budget Governance** | ✅ Shared controls and optimization | ❌ Spreadsheet-based |
| **Audit Logs & Compliance** | ✅ Enterprise-ready | ❌ Weak or absent |
| **Vendor and Supply Tools** | ✅ Two-sided platform advantage | ❌ Single-sided products |

---

## Financial Projections & Metrics

### Key Performance Indicators (KPIs)

| Metric | Year 1 | Year 3 | Year 5 |
|---|---|---|---|
| Paying Accounts | 50 | 400 | 1,800 |
| Avg Revenue Per Account | $25,000 | $50,000 | $95,000 |
| Gross Revenue Retention | 85% | 88% | 90% |
| Net Revenue Retention | 105% | 118% | 125% |
| Sales Cycle | 45 days | 60 days | 90 days |
| Gross Margin | 80-90% | 85-90% | 88-92% |

---

## Next Steps

- Validate enterprise pricing with 10-15 design partners
- Define seat model versus trip-based packaging
- Prioritize approval workflows, audit logs, and SSO in roadmap
- Build ROI case studies for corporate and agency buyers
- Prepare B2B sales deck and implementation playbook

---

*Last Updated: March 30, 2026*  
*Document Version: 1.0*
