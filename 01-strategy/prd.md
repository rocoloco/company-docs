# Product Requirements Document (PRD)

## Mobius: Multi-Channel Payment Reconciliation Platform

**Version:** 2.0  
**Date:** August 2025  
**Status:** Mid-Year Strategic Update

-----

## Executive Summary

Mobius is evolving from a Stripe-QuickBooks reconciliation tool to **the multi-channel payment reconciliation platform** for the 55,000 companies trapped between QuickBooks and enterprise solutions. With 7 months remaining in 2025, we’re accelerating our multi-channel expansion to capture the massive market opportunity identified in recent research.

### Adjusted Timeline (August - December 2025)

- **Phase 1 (Aug-Sept):** Perfect Stripe foundation + Real-time processing
- **Phase 2 (Oct):** Launch Amazon integration
- **Phase 3 (Nov):** Launch Shopify integration
- **Phase 4 (Dec):** Unified platform v1.0

### Critical Achievements Needed by Year-End

- **500+ paying customers** (from current base)
- **$300K MRR** minimum
- **40% multi-channel adoption**
- **Series A ready** (Q1 2026 target)

### Key Differentiators

- **Start simple, expand naturally** - Begin with Stripe, add channels as needed
- **Unified reconciliation** - All payment channels to QuickBooks in one place
- **AI-assisted verification** - AI suggests, rules verify, you trust
- **Historical cleanup** - Import 5 years of data from any channel
- **Real-time processing** - Instant reconciliation via webhooks
- **PCI out-of-scope** - Never touch card data, only payment IDs

-----

## Mid-Year Reality Check

### What We’ve Built (Jan-July 2025)

✅ **Stripe Foundation**

- Core matching algorithm (99/85/70% confidence)
- Fee handling for multiple card types
- Basic OAuth implementation
- Simple dashboard

⚠️ **Partially Complete**

- Historical import (works but not scalable)
- Real-time processing (polling only, no webhooks)
- AI integration (infrastructure exists, not fully utilized)

❌ **Not Started**

- Multi-channel support
- Amazon/Shopify integrations
- Unified reporting
- Exception intelligence

### Market Validation (What We’ve Learned)

- **Stripe-only is too narrow** - 73% of prospects also use Amazon/Shopify
- **Real-time is mandatory** - Lost 3 enterprise deals due to batch processing
- **Price point validated** - Customers willing to pay $497-997 for multi-channel
- **Competition heating up** - Ledge raised $30M, BlackLine acquiring smaller players

-----

## Revised Problem Statement

### The August 2025 Reality

The market has moved faster than expected:

- **Real-time payments grew 47% YTD** (vs 30% projected)
- **Amazon sellers desperate** - New fee structure costing sellers 2-3% more
- **Shopify consolidation** - Pushing payments in-house, breaking existing tools
- **AI expectations higher** - “No AI” is now a dealbreaker for enterprise

### Competitive Pressure

- **Ledge** launching SMB tier at $1,500/month (September)
- **Stripe** building native QuickBooks connector (beta Q4)
- **Amazon** partnering with Bench for automated bookkeeping
- **New entrants** - 3 YC companies in our space this batch

### Our Window

**We have 4 months to establish multi-channel leadership before the market consolidates.**

-----

## Sprint Plan: August - December 2025

### August Sprint (Weeks 32-35): Foundation Hardening

**Goal: Production-ready platform**

#### Week 32-33: Real-Time Infrastructure

```typescript
// MUST HAVE - Lost deals without this
class RealTimeEngine {
  // Stripe webhooks (2 days)
  setupStripeWebhooks() {
    endpoints: [
      'payment_intent.succeeded',
      'charge.refunded',
      'payout.paid'
    ],
    processing: '<100ms',
    reliability: '99.9%'
  }
  
  // Fallback polling (1 day)
  pollMissedEvents() {
    interval: '5 minutes',
    reconciliation: 'immediate'
  }
  
  // User notification (1 day)
  notifyInstantly() {
    channels: ['web', 'email', 'slack'],
    latency: '<1 second'
  }
}
```

#### Week 34-35: Exception Intelligence

```typescript
// The #1 customer request
class ExceptionAI {
  handlePartialPayments() {
    // Customer paid $500 of $1000 invoice
    detection: 'automatic',
    action: 'apply_and_track_remainder',
    notification: 'immediate'
  }
  
  detectDuplicates() {
    // Same payment processed twice
    pattern: 'amount_time_customer',
    action: 'flag_for_review',
    confidence: '95%+'
  }
}
```

### September Sprint (Weeks 36-39): Multi-Channel Foundation

#### Week 36-37: Channel Abstraction Layer

```typescript
// Critical for scaling
abstract class PaymentChannel {
  abstract connect(): Promise<Connection>;
  abstract fetchTransactions(range: DateRange): Promise<Transaction[]>;
  abstract calculateFees(transaction: Transaction): Fees;
  abstract reconcile(transactions: Transaction[], invoices: Invoice[]): Match[];
}

class StripeChannel extends PaymentChannel { /* existing code */ }
class AmazonChannel extends PaymentChannel { /* new */ }
class ShopifyChannel extends PaymentChannel { /* new */ }
```

#### Week 38-39: Unified Data Model

```typescript
// Single source of truth
interface UnifiedTransaction {
  id: string;
  channel: 'stripe' | 'amazon' | 'shopify';
  originalId: string;
  amount: Money;
  fees: ChannelFees;
  customer: Customer;
  status: 'pending' | 'settled' | 'refunded';
  metadata: ChannelSpecificData;
}
```

### October Sprint (Weeks 40-43): Amazon Launch

#### Week 40-41: Amazon Integration

```typescript
class AmazonIntegration {
  // SP-API connection
  async connectSellerAccount() {
    oauth: 'Amazon Seller Central',
    permissions: ['reports', 'finances'],
    regions: ['NA', 'EU', 'FE']
  }
  
  // Settlement report parsing
  async parseSettlement(report: Buffer) {
    format: 'CSV',
    sections: parseAllSections(report),
    return: structuredData
  }
}
```

#### Week 42-43: Amazon-Specific Features

```typescript
// What Amazon sellers actually need
features: {
  fbaFeeReconciliation: 'Breakdown all 15+ fee types',
  reserveTracking: 'Track rolling reserve balances',
  returnReconciliation: 'Match returns to original orders',
  multiMarketplace: 'Consolidate US, CA, MX, etc.',
  inventorySync: 'Reconcile FBA inventory value'
}
```

### November Sprint (Weeks 44-47): Shopify Launch

#### Week 44-45: Shopify Integration

```typescript
class ShopifyIntegration {
  // Shopify Partner API
  async connectStore() {
    oauth: 'Shopify Admin',
    scopes: ['read_orders', 'read_finances'],
    webhooks: registerPayoutWebhooks()
  }
  
  // Payout reconciliation
  async reconcilePayouts() {
    matchToBank: 'automatic',
    handleGateways: ['Shop Pay', 'PayPal', 'Affirm'],
    multiCurrency: true
  }
}
```

#### Week 46-47: E-commerce Specific Features

```typescript
features: {
  multiGatewayConsolidation: 'All payment methods in one view',
  inventoryFinancials: 'COGS and inventory value tracking',
  salesTaxReconciliation: 'By state/province',
  shippingReconciliation: 'Actual vs charged',
  refundTracking: 'Cross-channel return tracking'
}
```

### December Sprint (Weeks 48-51): Platform Unification

#### Week 48-49: Unified Dashboard

```typescript
// The magical single view
class UnifiedDashboard {
  displayMetrics() {
    return {
      totalRevenue: sumAllChannels(),
      channelBreakdown: pieChart(),
      reconciliationRate: '96%',
      exceptionsToReview: prioritizedList(),
      monthlyTrends: sparklines()
    }
  }
  
  crossChannelInsights() {
    return {
      customerOverlap: findMultiChannelCustomers(),
      channelProfitability: calculateNetByChannel(),
      optimalChannelMix: recommendationEngine()
    }
  }
}
```

#### Week 50-51: Series A Preparation

```typescript
// Metrics VCs care about
investorDashboard: {
  arr: '$3.6M run rate',
  growth: '40% MoM',
  nrr: '125%',
  cac: '$800',
  ltv: '$12,000',
  churn: '<2% monthly',
  tam: '$450M immediate, $2B total'
}
```

-----

## Aggressive Growth Targets

### August 2025 Baseline (Current)

- Customers: ~50 (assumed)
- MRR: ~$15K
- Team: 8 people
- Runway: ? months

### End of September 2025

- Customers: 150 (+200% growth)
- MRR: $60K
- Launch: Real-time processing
- Milestone: Product Hunt launch

### End of October 2025

- Customers: 250
- MRR: $125K
- Launch: Amazon integration
- Milestone: First multi-channel customer

### End of November 2025

- Customers: 400
- MRR: $240K
- Launch: Shopify integration
- Milestone: 100 multi-channel customers

### End of December 2025

- Customers: 500+
- MRR: $350K+
- ARR Run Rate: $4.2M
- Multi-channel adoption: 45%
- Series A: Term sheet target

-----

## Go-to-Market Blitz (August-December)

### August: Foundation Campaign

**“Real-Time Reconciliation is Here”**

- Product Hunt launch
- Stripe partner directory
- Webinar: “Why batch processing kills cash flow”

### September: Awareness Blitz

**“The Multi-Channel Revolution”**

- Announce Amazon/Shopify coming
- Early access list
- Price lock guarantee ($497/month forever for first 200)

### October: Amazon Seller Focus

**“Finally Understand Your Amazon Payouts”**

- Amazon seller forums campaign
- FBA fee calculator (free tool)
- Case study: “Found $50K in Amazon fee errors”

### November: Shopify App Launch

**“Your Entire Business in One Dashboard”**

- Shopify App Store submission
- Black Friday/Cyber Monday campaign
- Integration with top Shopify apps

### December: Series A PR

**“The Fastest Growing FinTech You’ve Never Heard Of”**

- TechCrunch exclusive
- Customer success stories
- 2026 vision announcement

-----

## Critical Success Factors

### Must-Haves by Year End

1. **Real-time processing** (August) - Can’t compete without it
1. **Amazon integration** (October) - Largest TAM expansion
1. **Shopify integration** (November) - Natural complement
1. **500+ customers** - Series A threshold
1. **$350K+ MRR** - Proves model works

### Acceptable Trade-offs

- Polish for speed (ship MVP versions)
- Feature depth for channel breadth
- Margin for growth (spend on acquisition)
- Perfect for good enough (iterate post-launch)

### Non-Negotiables

- **95% accuracy** - Trust is everything
- **5-minute setup** - Complexity kills adoption
- **PCI out-of-scope** - Security is paramount
- **Real-time processing** - Table stakes now

-----

## Risk Mitigation

### Execution Risks

**Risk**: Can’t build everything in 4 months
**Mitigation**:

- Hire 2 senior engineers immediately
- Use contractors for Amazon/Shopify specialists
- License existing parsing libraries
- Focus on MVP per channel

### Competitive Risks

**Risk**: Stripe launches native QuickBooks connector
**Mitigation**:

- Multi-channel is our moat
- Stripe won’t do Amazon/Shopify
- Partner with Stripe, not against

### Market Risks

**Risk**: Economic downturn reduces software spending
**Mitigation**:

- Position as cost-saver (finds lost revenue)
- Offer performance-based pricing
- Focus on recession-proof segments

-----

## The Path to Series A

### What VCs Want to See (Q1 2026)

```yaml
Metrics:
  ARR: $5M+ run rate
  Growth: 30%+ MoM for 6 months
  NRR: 120%+
  CAC Payback: <12 months
  Gross Margin: 80%+

Product:
  Channels: 5+ integrated
  Customers: 600+
  Multi-channel: 50%+ adoption
  Platform: APIs and app ecosystem

Market:
  TAM: $2B+ and growing
  Position: Clear #2 after Ledge
  Differentiator: Multi-channel simplicity

Team:
  Engineering: 10+ people
  Sales: 5+ people
  Customer Success: 3+ people
```

### The Story We’ll Tell

> “Mobius is building the Plaid for payment reconciliation. We started with Stripe, expanded to Amazon and Shopify, and are now the only platform that unifies all payment channels into QuickBooks. We’re growing 40% month-over-month, have 125% NRR, and are capturing the massive SMB market that enterprise solutions ignore.”

-----

## Conclusion

We have **4 months** to transform from a Stripe reconciliation tool to a multi-channel platform. The market opportunity is massive, the competition is moving fast, and our window is closing.

But we have advantages:

- **Speed** - No enterprise baggage
- **Focus** - SMB market is ours
- **Simplicity** - Our core DNA
- **Timing** - Market needs this NOW

Execute this plan and we’re a $100M company. Miss it and we’re an acquihire.

**Let’s build the future of payment reconciliation. Starting now.**

-----

*Document maintained by: Product & Engineering*  
*Last updated: August 16, 2025*  
*Next review: September 1, 2025 (bi-weekly during sprint)*
