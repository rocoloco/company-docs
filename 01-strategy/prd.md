# **Mobius One: Autonomous Payment Reconciliation**

## **Product Requirements Document v2.2 – Full Version with Mobius Core + AWS Integration**

---

## 🚀 Executive Summary

### Vision
Transform manual payment reconciliation from a 6-hour weekly nightmare into a 10-minute autonomous process, freeing finance teams to focus on strategic growth initiatives.

### Mission
Eliminate Excel-based reconciliation for mid-market SaaS companies through AI-powered transaction matching that connects banks directly to accounting systems.

### Core Value Proposition
**“Your finance team saves 6+ hours weekly while achieving 95%+ reconciliation accuracy.”**

---

## 🧱 Strategic Moat: Mobius Core

### What is Mobius Core?
Mobius Core is the proprietary AI engine that powers intelligent reconciliation for every Mobius customer. It combines deterministic heuristics, pattern-based learning, and GenAI scoring to improve over time.

**Key Functions:**
- Normalization and standardization of transaction data
- Multi-pass matching (exact, fuzzy, pattern)
- Confidence scoring and explanation generation
- Learning feedback loop that adapts by tenant
- Vertical-specific personalization

**Moat Strategy:**
- Embedded intelligence = user lock-in
- Proprietary signal = defensibility
- Feedback loop = compounding precision
- Seamless UI = Jobs-level delight

---

## 🔍 Target Customer Profile

- **Primary:** Finance managers at $5–15M ARR SaaS companies  
- **Pain Point:** 6+ hours weekly lost to manual reconciliation  
- **Buying Authority:** $400–600/month  
- **Success Metric:** Save time, avoid errors, increase insight

---

## 📈 Market Opportunity

- **TAM:** $1.2B reconciliation automation  
- **Addressable Market:** 50,000+ SaaS firms  
- **Adoption Lag:** 90% still using Excel for reconciliation

---

## 🏗️ Architecture Overview

### High-Level System Diagram

Bank Feed (Plaid)  ───┐
│
Normalizer (Mobius Core)
↓
Accounting Feed ─────┘
↓
Matcher (Exact → Fuzzy → AI-Assisted)
↓
Feedback Engine (Learning Loop)
↓
MatchingResult[] → Exception UI

---

## 🧠 Mobius Core Engine Modules

Located in `src/core/`

| Module             | Purpose                                               |
|--------------------|-------------------------------------------------------|
| `normalizer.ts`     | Cleans and standardizes Plaid and QuickBooks data     |
| `matcher.ts`        | Heuristic + AI scoring engine (multi-pass matching)   |
| `feedbackEngine.ts` | Updates behavior per tenant based on feedback         |
| `vendors.ts`        | Regex/lookup normalization of messy vendor strings    |
| `types.ts`          | Shared contracts for matching, transactions, results  |

### Shared Type Definition
```ts
interface MatchingResult {
  confidence: number;
  bank_transaction: BankTransaction;
  accounting_entry: BankTransaction;
  match_factors: string[];
  requires_review: boolean;
  explanation?: string;
}


⸻

🔧 AWS Integration

Service	Purpose
Bedrock	Claude/Nova models for scoring & explanations
S3 Vectors	Store embeddings for vendors/descriptions
OpenSearch	Fast similarity lookup for fuzzy matching
SageMaker	Train classifiers per tenant/industry (v2)
AgentCore	(Future) Agents for exception handling


⸻

🔄 Matching Logic (Mobius Core v1)

Match Tiers
	•	Exact Match: amount + date + vendor = 95%+
	•	Fuzzy Match: ± $0.05, ±2 days = 80–90%
	•	Pattern Match: user-approved in last 30 days = 75–85%
	•	Claude/Nova LLM scoring: add semantic matching and explanation text

Learning Feedback Loop
	•	User corrections influence confidence weights
	•	Preferences stored per tenant
	•	Feedback piped to feedbackEngine.ts

⸻

🧪 Claude Code Scaffolding (Dev Strategy)

Initial Directory to Scaffold:

src/core/
├── normalizer.ts
├── matcher.ts
├── vendors.ts
├── feedbackEngine.ts
├── types.ts

Core Issue for Claude Code:

/issue
title: Scaffold Mobius Core Engine v1 – Matching, Normalization, and Feedback Modules
description: |
  Build structure for the matching engine and normalization logic for transaction reconciliation.

  Must include:
  - normalizeTransaction()
  - matchTransactions()
  - learnFromFeedback()
  - MatchingResult interface
  - known vendor cleanup logic

labels: [ClaudeCode, core, matching, MVP]


⸻

💻 Core Features & User Experience

Phase 1: Foundation
	•	30-min onboarding (bank + books)
	•	Initial Mobius Core engine with matching + normalization
	•	Exception handling UI v1
	•	Confidence scoring and review flow

Phase 2: Intelligence
	•	Claude/Nova match suggestions
	•	Feedback learning
	•	S3 Vector + OpenSearch for fuzzy matches
	•	Real-time analytics dashboard

Phase 3: Scale
	•	Multi-tenant model training with SageMaker
	•	AgentCore workflow automation
	•	Mobile exception handling

⸻

💰 Pricing Model
	•	Plan: $497/month flat
	•	Target ROI: 20x from time saved
	•	Zero setup fees or per-transaction pricing

⸻

📊 Success Metrics

Metric	Target
Setup time	<30 minutes
Match accuracy	>95%
Time saved/week	6 hours → 10 minutes
Retention	>95% monthly
Feedback impact	Accuracy improves weekly
Claude confidence use	>50% of matches assisted


⸻

📅 Development Timeline

Phase 1 (Weeks 1–8)
	•	✅ Core engine scaffolding (Mobius Core v1)
	•	✅ QuickBooks + Plaid integrations
	•	🚧 Heuristic matcher + feedback loop
	•	🚧 Exception handling UI

Phase 2 (Weeks 9–16)
	•	Claude API scoring (Bedrock)
	•	Match explanation generator
	•	Fuzzy vector search + dashboard

Phase 3 (Weeks 17–24)
	•	Feedback loop tuned per tenant
	•	SageMaker classifiers per industry
	•	Agent-based automation (evaluate)

⸻

🔐 Risk & Mitigation

Risk	Mitigation
AI inaccuracy	Claude scoring layered over heuristics
Bad data input	Normalizer + vendor rules
Long onboarding	Zero-config setup
Resistance to automation	Immediate value delivery in week 1


⸻

🧠 Future Product Evolution
	•	Year 1: Fast, accurate reconciliation
	•	Year 2: Forecasting + working capital AI
	•	Year 3: Fully autonomous finance operations

⸻

Mobius Core isn’t just an AI engine—it’s the moat, the muscle, and the magic.
