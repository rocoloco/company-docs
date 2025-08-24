# Product Requirements Document (PRD)

## Mobius 1: Multi-Channel Payment Reconciliation Platform

**Version:** 3.0  
**Date:** November 2025  
**Status:** Pre-Launch Beta Testing

-----

## Executive Summary

SMBs waste 60 hours monthly reconciling payments from multiple channels. Mobius 1 automates this entire process, reducing reconciliation to 5 minutes. We automatically match bank deposits to payment processor payouts (Stripe, Amazon, Shopify) and sync everything to QuickBooks with fees properly separated.

### Current Reality

- **Customers:** 10 beta testers (pre-launch)
- **Product Status:** Core matching engine built and working (92% accuracy)
- **Revenue:** $0 MRR (launching January 2026)
- **Timeline:** 12 months to 150 customers and seed funding

### The Validated Problem

**PwC research documents the burden:**

- **60 hours monthly** for businesses with 1,000 transactions
- **200 hours monthly** for businesses with 10,000 transactions
- **0.8-1.8% error rate** in manual reconciliation
- **20 hours weekly** lost to multi-channel data reconciliation (Webgility)

### Solution: 5-Minute Reconciliation

Multi-channel payment reconciliation that:

- Automatically matches bank deposits to payment processor payouts
- Breaks down payouts into individual transactions
- Syncs to QuickBooks with fees properly separated
- Flags discrepancies for investigation

### Launch Timeline

- **January 2026:** Launch with Stripe only (5 customers)
- **April 2026:** Add second channel (40 customers, $12K MRR)
- **December 2026:** Full platform (150 customers, $45K MRR)

-----

## Problem Statement

### The Multi-Channel Reconciliation Nightmare

Mid-market e-commerce businesses processing payments through multiple channels face an exponential reconciliation burden:

**Current Process (60+ hours monthly):**

1. Export bank statements
1. Export Stripe payouts
1. Export Amazon settlement reports
1. Export Shopify payouts
1. Manually match deposits to payouts in Excel
1. Break down bulk payouts into individual transactions
1. Separate fees for proper categorization
1. Create QuickBooks journal entries
1. Investigate and resolve discrepancies
1. Maintain audit trail for compliance

**Why Current Solutions Fail:**

- **QuickBooks:** Can’t match deposits to payment processor payouts
- **A2X/Synder:** Single channel only, still require manual bank matching
- **BlackLine:** $6,400/month enterprise overkill
- **Manual/Excel:** Error-prone, no audit trail, doesn’t scale

### Target Customer Profile

**Primary: Shopify Plus Stores**

- **Revenue:** $3-10M annually
- **Payment Mix:** Stripe (primary) + PayPal/Amazon (secondary)
- **Transaction Volume:** 500-5,000 monthly
- **Current Solution:** Excel or partial automation
- **Pain:** “We spend 3 days every month just matching deposits”
- **Budget:** $300-1,000/month for financial tools

**Secondary: Multi-Channel E-commerce**

- **Revenue:** $5-20M annually
- **Channels:** Direct site + Amazon + retail/wholesale
- **Pain:** “Can’t reconcile across all our payment channels”
- **Trigger:** Month-end close taking 5+ days

-----

## Solution Overview

### Core Value Proposition

**“Multi-channel payment reconciliation in 5 minutes”**

Save 60 hours monthly by automating the entire reconciliation process across Stripe, Amazon, and Shopify.

### Product Architecture

#### Phase 1: Stripe Foundation (January 2026)

**Core Features:**

- Connect bank via Plaid (2-minute setup)
- Automatic Stripe payout matching
- Transaction breakdown with fee separation
- QuickBooks journal entry creation
- Discrepancy flagging for review

**User Flow:**

```
1. Connect bank account (2 min)
2. Connect Stripe (1 min)
3. Connect QuickBooks (1 min)
4. Click "Reconcile December"
5. Review matches (95% auto-matched)
6. Confirm edge cases
7. Sync to QuickBooks
Total time: 5 minutes
```

#### Phase 2: Amazon Integration (April 2026)

**Additional Features:**

- Settlement report parsing
- Reserve tracking and release matching
- FBA fee breakdown (15+ fee types)
- Multi-marketplace support (US, CA, UK)

#### Phase 3: Shopify Support (July 2026)

**Additional Features:**

- Multi-gateway reconciliation
- Shopify Payments + PayPal + others
- POS vs online separation
- Currency conversion handling

#### Phase 4: Platform Maturity (December 2026)

**Platform Features:**

- Unified dashboard across all channels
- Historical reconciliation (24 months)
- API access for Scale customers
- Custom matching rules
- Audit trail exports

-----

## Product Requirements

### Functional Requirements

#### Core Reconciliation Engine

**Matching Pipeline:**

1. **SQL Pattern Matching** - 80% of standard deposits
1. **Vector Embeddings** - Complex description matching
1. **ML Confidence Scoring** - Historical pattern learning
1. **Manual Review Queue** - Low-confidence matches

**Performance Targets:**

- 95% auto-match accuracy
- <5 minute processing for 10,000 transactions
- 99.9% uptime

#### Integration Requirements

**Payment Processors:**

- Stripe API (payouts, transactions, fees)
- Amazon SP-API (settlement reports)
- Shopify API (payouts, orders)

**Banking:**

- Plaid (read-only transaction access)
- Support for 12,000+ US banks

**Accounting:**

- QuickBooks Online (initial)
- Xero (Q3 2026)

#### User Interface

**Dashboard:**

- Reconciliation status by month
- Time saved counter
- Match confidence indicators
- Discrepancy alerts

**Reconciliation View:**

- Deposit list with match status
- Drag-and-drop manual matching
- Bulk actions for common patterns
- Transaction detail drill-down

**Settings:**

- Connection management
- QuickBooks account mapping
- Matching rule customization
- Team user management

### Non-Functional Requirements

#### Performance

- Page load: <2 seconds
- Reconciliation: <5 minutes for monthly data
- API response: <500ms p99

#### Security

- SOC 2 Type II by December 2026
- Read-only bank access via Plaid
- AES-256 encryption at rest
- No storage of card numbers or bank credentials

#### Scalability

- 100,000 transactions/month per customer
- 1,000 concurrent users
- 99.9% availability SLA

-----

## Pricing Strategy

### Simple Tier Structure

#### Starter - $297/month

- Stripe only
- Unlimited transactions
- QuickBooks sync
- Email support

**Target:** Small businesses testing automation
**Positioning:** “Try automated reconciliation”

#### Growth - $597/month *(Primary tier)*

- Stripe + one additional channel
- Unlimited transactions
- Priority support
- Historical reconciliation (12 months)

**Target:** Growing businesses with multi-channel pain
**Positioning:** “Complete reconciliation solution”

#### Scale - $997/month

- All payment channels
- Unlimited transactions
- API access
- Dedicated success manager

**Target:** Larger SMBs approaching enterprise
**Positioning:** “Enterprise features at SMB prices”

### Pricing Rationale

- **Time-based value:** 60 hours saved = $3,000 labor cost
- **Below alternatives:** BlackLine at $6,400/month
- **Fixed pricing:** Predictable costs, no transaction fees

-----

## Go-to-Market Strategy

### Launch Strategy

#### Phase 1: Stripe Focus (Q1 2026)

**Channels:**

- Google Ads: “Stripe QuickBooks reconciliation”
- Direct outreach to Shopify Plus stores
- Stripe App Marketplace listing

**Messaging:**
“Reconcile a month of Stripe in 5 minutes”

**Goals:**

- 5 paying customers
- 3 case studies
- Refine onboarding

#### Phase 2: Expansion (Q2 2026)

**Add Second Channel:**

- Launch Amazon or Shopify based on demand
- Target existing customers for upgrades

**Goals:**

- 40 total customers
- $12K MRR
- Prove multi-channel value

#### Phase 3: Scale (Q3-Q4 2026)

**Full Platform:**

- All channels live
- Referral program
- Partnership discussions

**Goals:**

- 150 customers
- $45K MRR
- Seed funding secured

### Customer Acquisition

**Primary Channel: Google Ads (60%)**

- High-intent searches
- Expected CAC: $800 → $400
- Budget: $3K/month

**Secondary: Direct Outreach (30%)**

- Shopify Plus store list
- LinkedIn to e-commerce CFOs
- 100 qualified conversations/month

**Tertiary: Marketplaces (10%)**

- Stripe Apps
- QuickBooks App Store
- Organic/referral

-----

## Success Metrics

### Product Metrics

- **Reconciliation time:** <5 minutes
- **Match accuracy:** >95%
- **Setup completion:** >80%
- **Monthly active usage:** >90%

### Business Metrics

- **CAC:** <$800 (Year 1), <$400 (Year 2)
- **Monthly churn:** <5% (Year 1), <3% (Year 2)
- **Gross margin:** >70%
- **LTV:CAC:** >3:1

### Customer Success Metrics

- **Time to first value:** <24 hours
- **Support resolution:** <4 hours
- **NPS:** >50
- **Customer time saved:** 60+ hours/month average

-----

## Risk Mitigation

### Product Risks

**Risk:** Can’t achieve 95% matching accuracy
**Mitigation:** Manual review queue, continuous learning, clear confidence scores

**Risk:** Payment processor API changes
**Mitigation:** Version monitoring, regression testing, graceful degradation

**Risk:** Plaid connectivity issues
**Mitigation:** CSV upload fallback, multiple connection methods

### Market Risks

**Risk:** QuickBooks adds native multi-channel reconciliation
**Mitigation:** Move fast, build switching costs, superior UX

**Risk:** Competitors copy approach
**Mitigation:** First-mover advantage, pattern library moat

### Execution Risks

**Risk:** Complex onboarding hurts adoption
**Mitigation:** White-glove onboarding for first 50 customers

**Risk:** Customer support overwhelm
**Mitigation:** Proactive monitoring, self-service docs, community forum

-----

## Implementation Timeline

### Q4 2025: Foundation

- Complete beta testing with 10 users
- Achieve 95% matching accuracy
- Polish Stripe integration
- Prepare launch materials

### Q1 2026: Launch

- **January:** Convert 5 beta users to paid
- **February:** Add 5 new customers
- **March:** 20 total customers, gather feedback

### Q2 2026: Expand

- **April:** Launch second channel, 40 customers
- **May:** Optimize based on learnings
- **June:** 60 customers, hire support

### Q3 2026: Scale

- **July:** Launch third channel
- **August:** 90 customers
- **September:** 115 customers, begin fundraising

### Q4 2026: Growth

- **October:** API launch
- **November:** 135 customers
- **December:** 150 customers, $45K MRR, close seed round

-----

## Team Requirements

### Current Team

- Founders (technical + business)
- Contract designer

### Q1 2026 Hires

- Customer Success Lead (part-time)
- Technical Writer

### Q2 2026 Hires

- Support Engineer
- Sales Executive

### Post-Funding Hires

- VP Sales
- 2 Senior Engineers
- Head of Customer Success
- Marketing Manager

-----

## Technical Architecture

### Matching Engine Pipeline

Our sophisticated 4-stage matching architecture ensures 95% accuracy:

1. **SQL Pattern Matching**
- Direct amount and date matching
- Known processor patterns
- Handles 80% of standard cases
1. **Vector Embeddings**
- Semantic similarity for descriptions
- Handles variations in bank descriptions
- Learns from confirmed matches
1. **ML Confidence Scoring**
- Historical pattern recognition
- Customer-specific learning
- Confidence thresholds for auto-matching
1. **Quality Assurance**
- Manual review queue for <85% confidence
- Bulk action tools for common patterns
- Continuous feedback loop

### Infrastructure

- **Backend:** Node.js/TypeScript
- **Database:** PostgreSQL with TimescaleDB
- **Queue:** Redis/Bull for async processing
- **APIs:** REST with GraphQL planned
- **Hosting:** AWS with multi-region support
- **Monitoring:** DataDog, Sentry, PagerDuty

-----

## Validation Metrics

### What We’ve Proven

- 92% matching accuracy in beta
- 8-minute average setup time
- 5-minute reconciliation achievable
- Architecture scales to 100K transactions

### What Needs Validation

- $597 price point acceptance
- 5% monthly churn achievable
- $800 CAC sustainable
- Multi-channel demand strength

-----

## Success Definition

### Minimum Success (December 2026)

- 100 paying customers
- $30K MRR
- <5% monthly churn
- Clear product-market fit signals

### Target Success (December 2026)

- 150 paying customers
- $45K MRR
- Multiple channels proven
- Seed funding secured

### Exceptional Success (December 2026)

- 200+ customers
- $60K+ MRR
- <3% monthly churn
- Multiple term sheets

-----

## Conclusion

Mobius 1 solves a validated, quantified problem that costs SMBs 60+ hours monthly. Our multi-channel reconciliation platform delivers immediate time savings with simple, predictable pricing. With a realistic growth path to 150 customers and $45K MRR by December 2026, we’re positioned to capture a significant portion of the 55,000 SMBs currently underserved by existing solutions.

**Next Steps:**

1. Complete beta testing (December 2025)
1. Launch with Stripe (January 2026)
1. Achieve product-market fit (Q1 2026)
1. Scale to 150 customers (December 2026)

-----

*“Multi-channel payment reconciliation in 5 minutes. Save 60 hours monthly.”*

**Mobius 1** | **Product Requirements Document**  
**Last Updated:** November 2025  
**Launch Date:** January 2026​​​​​​​​​​​​​​​​
