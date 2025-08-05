# 🧾 Product Requirements Document (PRD)

## Product Name  
**Mobius One**

## Version  
**v1.0**

## Prepared By  
Mobius Product Team

## Date  
August 5, 2025

---

## 🎯 Purpose

Mobius One is a stream-based, AI-enhanced payment reconciliation engine designed to eliminate the pain points of batch-based and rule-bound systems. The goal is to deliver real-time, explainable reconciliation with intelligent pattern learning, 24-hour onboarding, and zero-consultant deployment — all within a single modular monolith.

---

## 🧠 Background & Motivation

### Industry Failures:
- Legacy tools like NetSuite and BlackLine require expensive multi-month implementations and suffer from brittle, rule-based matchers.
- Modern challengers like Numeric and Ledge offer AI, but lack real-time pipelines, partial payment support, and audit-grade explainability.
- Most platforms cannot handle vendor name variations, multi-invoice payments, or fee-based distortions.

### Mobius One Advantages:
- Event-driven matching engine with per-transaction processing
- Multi-stage pipeline (Exact → Fuzzy → ML) with early termination
- Partial payment + multi-invoice support
- Pattern learning per tenant and across tenants
- Sub-second match response times
- Fully auditable match logs and feedback loop integration
- Modular monolith design with microservice-readiness

---

## 🧩 Features & Requirements

### 1. Stream-Based Matching Pipeline

**Description:**  
Each transaction triggers a reconciliation flow asynchronously.

**Requirements:**
- Ingest transaction events individually
- Select invoice candidates by time window, amount range, and status
- Process through match stages with early-exit support
- Emit match records, audit logs, and optional notifications

---

### 2. Multi-Stage Matching Logic

**Pipeline Stages:**
- `ExactMatcher` (100%): amount, date, reference
- `FeeAdjustedMatcher` (95%): match after subtracting known PSP fees
- `FuzzyMatcher` (70–90%): name similarity, timing pattern
- `MLScorer` (<70%): Claude via Bedrock for semantic reasoning

**Key Feature:**  
Each stage logs match rationale (`match_factors`) and confidence.

---

### 3. Amount Matcher with Fee Intelligence

**Requirements:**
- Recognize common platform fee patterns (Stripe, PayPal, Square, wire)
- Allow configurable fee rules per tenant/PSP
- Score matches using fee-adjusted amount deltas

---

### 4. Multi-Invoice & Partial Payment Handling

**Requirements:**
- Allow 1:many and many:1 matches
- Suggest invoice groupings based on total amount
- Track and store partial payment links
- Warn on overpayment/underpayment beyond threshold

---

### 5. Pattern Learning Engine

**Functionality:**
- Extract match patterns (merchant signature, timing, fee profile)
- Learn from confirmed matches with EMA weighting
- Cache by tenant + global network (opt-in)
- Feed into candidate pre-selection

---

### 6. Feedback Loop & Self-Tuning

**Requirements:**
- Store approved/rejected/adjusted match feedback
- Reinforce pattern confidence with positive feedback
- Penalize failed match attempts in scoring memory
- Trigger fallback match retry if user corrects a bad match

---

### 7. Match Explanation Layer

**Requirements:**
- Human-readable explanation string (e.g. “Matched by amount and invoice ref”)
- Confidence score with color-coded thresholds
- Match factors (e.g. `['amount_match', 'fee_adjusted', 'vendor_signature']`)
- Suggested alternatives if confidence < 90%

---

### 8. Match Storage & Audit Logging

**Design:**
- Append-only `matches` table with denormalized transaction & invoice snapshot
- Indexed by `tenant_id`, `tx_date`, `status`, `confidence`
- JSONB column for `match_factors`, `fee_meta`, `review_notes`

---

### 9. Bulk Reconciliation Engine

**Requirements:**
- `reconcileRange(tenantId, dateRange)`
- Stream invoices and txs, pipe through same matcher logic
- Support batch override rules (e.g. “skip matches < 80% confidence”)

---

### 10. Pricing Tier Metering

**Starter:**  
- $497/mo  
- 10,000 transactions  
- $0.02 overage

**Growth:**  
- $997/mo  
- 50,000 transactions  
- $0.01 overage

**Enterprise:**  
- Custom pricing  
- Unlimited usage  
- Dedicated API, custom workflows, white-label support

---

## ⚙️ Technical Constraints

- **Architecture:** Modular monolith (Node/TypeScript)
- **Model Inference:** AWS Bedrock (Claude Sonnet)
- **Infra:** AWS-native (DynamoDB, S3, Lambda-compatible modules)
- **Data Layer:** Append-only; audit-grade; event-logged
- **Latency Targets:** <500ms median; <250ms for ExactMatcher
- **Event Backbone:** Internal EventBus abstraction (pub/sub-style interface)

---

## 📅 Timeline

| Week | Milestone |
|------|-----------|
| 1-2  | Ingestion + ExactMatcher + CandidateSelector |
| 3-4  | Fee logic, multi-invoice matcher, baseline UI |
| 5-6  | Pattern learner, feedback engine, audit layer |
| 7-8  | Bulk ops, match explainability, API surface |
| 9-12 | Usage metering, opt-in network learning, billing logic |

---

## ✅ Success Criteria

- 95%+ of transactions auto-matched with confidence >85%
- <5 minutes to reconcile 10,000+ txs in bulk
- >90% user satisfaction on explanation clarity
- Fully onboard new tenant in <24 hours

---
