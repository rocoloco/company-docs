# Product Requirements Document (PRD)

## Mobius: The Revenue Recovery Platform

**Version:** 3.0  
**Date:** August 2025  
**Status:** Pivot to Revenue Recovery Positioning

-----

## Executive Summary

Mobius is **the revenue recovery platform** that finds the 3-5% of revenue businesses are bleeding through financial blind spots. We don’t just reconcile payments—we discover lost money hiding in merchant fees, failed payments, untracked refunds, and multi-channel chaos. For the 55,000 companies trapped between QuickBooks and enterprise solutions, we turn financial blindness into financial intelligence.

### The Brutal Truth

Every business processing payments across multiple channels is losing money they don’t even know about:

- **Hidden fees** eating 2-3% extra on Amazon
- **Failed payments** killing 5-9% of SaaS revenue
- **Return fraud** costing retailers $101B annually
- **Duplicate charges** happening daily across channels
- **Processing errors** compounding silently for months

**We find it. Automatically. In real-time.**

### Adjusted Timeline (August - December 2025)

- **Phase 1 (Aug-Sept):** Launch Revenue Recovery Engine for Stripe
- **Phase 2 (Oct):** Amazon Fee Finder - “We found $50K in hidden fees”
- **Phase 3 (Nov):** Shopify Leak Detector - Multi-gateway revenue recovery
- **Phase 4 (Dec):** Full Revenue Intelligence Platform

### Critical Achievements Needed by Year-End

- **$10M+ in recovered revenue** for customers (trackable metric)
- **500+ paying customers** finding real money
- **$350K MRR** from revenue share + subscriptions
- **Series A story:** “We find money others miss”

### The New Value Proposition

**Before:** “We reconcile your payments accurately”  
**Now:** “We find the money you’re losing”

**Before:** “95% reconciliation accuracy”  
**Now:** “Average customer recovers $47,000 in year one”

**Before:** “Multi-channel payment reconciliation”  
**Now:** “See every dollar across every channel—and get back what’s yours”

-----

## The Real Problem We’re Solving

### It’s Not About Reconciliation—It’s About Blindness

Businesses aren’t looking for better reconciliation. They’re hemorrhaging money and don’t even know it. Our research proves:

- **84% of companies** still use Excel for reconciliation, creating blind spots
- **3-5% of revenue** disappears into these blind spots
- **5-7 full days monthly** spent searching for discrepancies—after the money is already gone
- **$162 billion** in improper payments by the U.S. government alone shows the scale

### The Money Is Already Lost—They Just Don’t Know It

**E-commerce Reality:**

- Amazon sellers lose 2-3% to new fee structures they don’t understand
- 15+ different FBA fee types hiding in settlement reports
- Returns processed wrong, inventory value miscalculated
- “Found $50K in Amazon fee errors” - actual customer quote

**SaaS Bleeding:**

- 5-9% MRR loss from failed payment recovery
- Involuntary churn from card failures never attempted again
- Prorations calculated wrong, trials converting at wrong amounts
- Dunning processes failing silently

**Retail Leakage:**

- $743 billion in returns (14.5% of sales) with $101B in fraud
- Multi-location discrepancies compound daily
- Gift card breakage untracked, store credit vanishing
- POS systems not talking to accounting

### Our Unique Insight

**Everyone else:** “Let us reconcile your payments faster”  
**Mobius:** “Let us show you the money you didn’t know you lost”

The difference:

- Others match transactions to invoices
- We identify patterns that indicate lost revenue
- Others show you what happened
- We show you what SHOULD have happened

-----

## Solution: The Revenue Recovery Platform

### Core Discovery Mechanisms

#### 1. Fee Anomaly Detection

```typescript
// Not just calculating fees—finding overcharges
class FeeAuditor {
  async findLostRevenue(transactions: Transaction[]): Promise<RecoveredMoney> {
    const discoveries = {
      amazonOvercharges: this.auditAmazonFees(),     // "You were charged long-term storage on items that sold"
      stripeInternational: this.findWrongRates(),     // "You're paying international rates on domestic cards"
      shopifyDouble: this.findDuplicateCharges(),     // "Same transaction charged by two gateways"
      hidden: this.findUndisclosedFees()              // "Fees not in your original agreement"
    };
    
    return {
      totalFound: sum(discoveries),
      actionable: this.generateClaimDocuments(),      // Ready-to-send recovery claims
      prevented: this.setupAlerts()                   // Stop future losses
    };
  }
}
```

#### 2. Failed Payment Recovery

```typescript
// Every failed payment is lost revenue
class RevenueRecovery {
  async recoverFailedPayments(): Promise<Recovery> {
    return {
      // Immediate wins
      retryOptimization: this.smartRetrySchedule(),   // Retry at optimal times
      cardUpdater: this.updateExpiredCards(),         // Proactively update before failure
      dungeonRescue: this.intelligentDunning(),       // Personalized recovery sequences
      
      // Long-term prevention
      declineIntelligence: this.predictFailures(),    // Stop failures before they happen
      routingOptimization: this.smartRouting()        // Route to highest success processor
    };
  }
}
```

#### 3. Return & Refund Intelligence

```typescript
// Returns hide massive fraud and errors
class ReturnAuditor {
  async auditReturns(): Promise<Findings> {
    return {
      fraudulent: this.detectReturnFraud(),           // "Serial returner detected"
      policy: this.findPolicyViolations(),            // "Refunded outside return window"
      doubleRefunds: this.findDuplicateRefunds(),     // "Customer refunded twice"
      inventory: this.reconcileReturnedInventory()    // "Items marked returned but not received"
    };
  }
}
```

### The Money Discovery Dashboard

Instead of “reconciliation status,” show:

```typescript
interface MoneyDashboard {
  // The number they care about
  moneyFound: {
    thisMonth: "$12,847",
    yearToDate: "$47,283",
    projected: "$94,000"
  },
  
  // Where we found it
  discoveries: [
    { source: "Amazon FBA fees", amount: "$3,200", status: "Claim filed" },
    { source: "Duplicate Stripe charges", amount: "$1,100", status: "Refunded" },
    { source: "Failed payment recovery", amount: "$8,547", status: "Recovered" }
  ],
  
  // What's at risk
  warnings: [
    "23 transactions showing fraud patterns",
    "$2,100 in fees outside contracted rates",
    "142 failed payments not attempted"
  ],
  
  // Competitive intelligence
  benchmark: "You're losing 2.3% more than similar businesses"
}
```

-----

## Go-to-Market: The Money Discovery Campaign

### August: The Wake-Up Call

**“You’re Losing 3-5% of Revenue and Don’t Even Know It”**

Launch with a Revenue Leak Calculator:

1. Enter your payment volume
1. Select your channels (Stripe, Amazon, Shopify)
1. See estimated annual loss
1. “Find My Lost Revenue” CTA

Content strategy:

- “We Found $1.2M for Red Wing Shoes” (case study)
- “The Hidden Cost of Multi-Channel Payments” (research report)
- “5 Places Money Hides in Your Payment Stack” (viral LinkedIn post)

### September: The Proof Campaign

**“Real Companies. Real Money Found.”**

Customer discovery stories:

- “How We Found $50K in Amazon Fees for [Customer]”
- “SaaS Company Recovers 7% of MRR with Smart Dunning”
- “Retailer Discovers $30K Monthly Return Fraud”

Free tools:

- Amazon FBA Fee Auditor (upload settlement report, see overcharges)
- Stripe Fee Analyzer (connect via OAuth, instant audit)
- Failed Payment Calculator (show recoverable revenue)

### October: Amazon Seller Domination

**“Every Amazon Seller Is Being Overcharged”**

Amazon-specific campaign:

- Seller forum takeover with fee audit results
- “Amazon Sent Us a $50K Check” case study
- Free Chrome extension showing fees in real-time
- Partnership with Amazon seller tools

### November: Black Friday Revenue Recovery

**“Don’t Let Black Friday Profits Disappear”**

Real-time monitoring during peak:

- Live fraud detection dashboard
- Instant fee anomaly alerts
- Failed payment recovery in real-time
- “Save Your Black Friday” messaging

### December: The Series A Story

**“The Fastest Growing Revenue Recovery Platform”**

Metrics that matter:

- “$10M+ recovered for customers”
- “Average customer ROI: 847%”
- “Payback period: 1.3 months”
- “Every dollar we find is pure profit”

-----

## Pricing: Aligned with Value Created

### Pure Performance Model (Premium)

**“We Only Win When You Win”**

- **$497 base** + **15% of recovered revenue**
- No limit on recovery amount
- Includes all channels and features
- Perfect for: Companies wanting zero risk

### Subscription + Success (Standard)

**“Predictable Cost, Predictable Recovery”**

- **$997/month** + **5% of recovered revenue**
- First $20K recovered included
- Up to 3 channels
- Perfect for: Growing companies with steady volume

### Enterprise Intelligence (Custom)

**“Your Revenue Protection Department”**

- Custom pricing based on volume
- Dedicated recovery specialist
- API access and white-label options
- Perfect for: $50M+ revenue companies

### Why This Pricing Works

- Aligned incentives (we find more = we earn more)
- Immediate ROI (customers see money in month 1)
- Trust builder (we’re confident we’ll find money)
- Natural expansion (more channels = more recovery)

-----

## Success Metrics (What Actually Matters)

### Vanity Metrics We’re Abandoning

❌ Reconciliation accuracy percentage  
❌ Number of transactions processed  
❌ Time to reconcile  
❌ Matching confidence scores

### Real Metrics That Drive Growth

✅ **Revenue Recovered per Customer**

- Target: $50K average Year 1
- Drives: Customer success stories
- Proves: Actual value delivery

✅ **Recovery ROI**

- Target: 10x customer investment
- Drives: Easy sales conversations
- Proves: No-brainer value prop

✅ **Time to First Dollar**

- Target: <7 days from signup
- Drives: Immediate “aha” moments
- Proves: Instant value

✅ **Viral Coefficient**

- Target: Each customer refers 2+ others
- Drives: “You have to see what they found”
- Proves: Product-market fit

✅ **Revenue Share Percentage**

- Target: 40% of revenue from success fees
- Drives: Aligned growth model
- Proves: We deliver real money

-----

## Technical Approach: Finding Money, Not Matching Transactions

### The Discovery Engine Architecture

```typescript
class RevenueDiscoveryEngine {
  // Pattern recognition across all customers
  async learnFromNetwork(): Promise<Patterns> {
    return {
      feePatterns: this.aggregateFeeAnomalies(),      // "This fee is 2x industry average"
      fraudSignals: this.collectFraudPatterns(),      // "This looks like return fraud"
      failurePatterns: this.analyzePaymentFailures(), // "Cards from this bank always fail"
      benchmarks: this.calculateIndustryNorms()       // "You're paying more than peers"
    };
  }
  
  // Proactive discovery
  async huntForMoney(customer: Customer): Promise<Opportunities> {
    const opportunities = [];
    
    // Check every transaction for fee correctness
    opportunities.push(...this.auditAllFees(customer));
    
    // Find all failed payments worth recovering
    opportunities.push(...this.identifyRecoverablePayments(customer));
    
    // Detect fraud and errors in returns
    opportunities.push(...this.auditReturns(customer));
    
    // Compare to industry benchmarks
    opportunities.push(...this.findBenchmarkGaps(customer));
    
    return this.prioritizeByValue(opportunities);
  }
  
  // Automated recovery
  async recoverMoney(opportunity: Opportunity): Promise<Result> {
    switch(opportunity.type) {
      case 'excessive_fee':
        return this.fileClaimWithProvider(opportunity);
      case 'failed_payment':
        return this.executeSmartRetry(opportunity);
      case 'fraud_return':
        return this.disputeWithEvidence(opportunity);
      case 'duplicate_charge':
        return this.requestRefund(opportunity);
    }
  }
}
```

### Intelligence Layer (Not Just AI Buzzwords)

```typescript
class CollectiveIntelligence {
  // Learn from every customer to help all customers
  patterns = {
    // Amazon-specific
    amazonFeeErrors: [
      "Long-term storage on sold items",
      "Incorrect category fees",
      "Double-charged removal fees",
      "Wrong dimensional weight"
    ],
    
    // Stripe-specific  
    stripeOvercharges: [
      "International fees on domestic cards",
      "Incorrect non-profit rates",
      "Platform fees on direct charges",
      "Radar fees when disabled"
    ],
    
    // Shopify-specific
    shopifyLeaks: [
      "Double-processing across gateways",
      "Incorrect POS vs online rates",
      "Third-party app duplicate charges",
      "Currency conversion markups"
    ]
  };
  
  // Network effect: Every customer makes us smarter
  async improveFromDiscovery(discovery: Discovery): Promise<void> {
    this.patterns[discovery.channel].push(discovery.pattern);
    this.notifyNetworkOfNewPattern(discovery);
    this.retroactivelyCheckAllCustomers(discovery.pattern);
  }
}
```

-----

## Customer Experience: The “Holy Shit” Moment

### Day 1: Immediate Discovery

**Onboarding Goal: Find money in first session**

1. Connect payment channel (OAuth, 2 minutes)
1. Start scanning last 90 days
1. **Show first discovery within 60 seconds**
1. “We found $3,247 in overcharges. Want to recover it?”
1. One click to start recovery

### Week 1: The First Check

**Success Goal: Money in their bank**

- Automated claims filed with providers
- Smart retry campaigns launched
- Daily updates: “Recovered another $428 today”
- **First recovery hits their bank account**
- Email: “Your first $3,247 has been recovered!”

### Month 1: The Compound Effect

**Growth Goal: They can’t imagine life without us**

- Weekly recovery report: “Found another $2,100 this week”
- Benchmark report: “You’re now losing 2% less than peers”
- Prevention alerts: “Blocked $500 in duplicate charges”
- **Total recovered exceeds 3 months of subscription**

### The Referral Moment

**Viral Goal: They tell everyone**

The moment they tell others:

- “They found $50K I didn’t know I was losing”
- “It paid for itself in 3 days”
- “I thought my books were perfect—they weren’t”
- “My Amazon fees were 30% too high for 2 years”

-----

## Competitive Positioning: We Find Money, Others Find Matches

### The Positioning Matrix

|Feature          |Mobius          |Ledge          |BlackLine        |Stripe Native |
|-----------------|----------------|---------------|-----------------|--------------|
|**Core Focus**   |Find lost money |Match payments |Compliance       |Basic matching|
|**Value Prop**   |Revenue recovery|Accuracy       |Audit ready      |Free          |
|**Pricing**      |% of recovery   |$1,500/month   |$6,400/month     |Free-$99      |
|**Time to Value**|Day 1           |2 weeks        |4.5 months       |Instant       |
|**ROI**          |10x guaranteed  |Time savings   |Compliance       |None          |
|**Customer Says**|“Found $50K!”   |“It’s accurate”|“We’re compliant”|“It’s free”   |

### Our Unique Advantages

**1. Network Intelligence**

- Every customer helps find patterns for all customers
- Collective intelligence about fees, fraud, failures
- “Wisdom of the crowd” for payment recovery

**2. Aligned Incentives**

- We only succeed when customers recover money
- No money found = we work harder
- Success fees ensure we maximize recovery

**3. Immediate Value**

- Find money in first session
- No training, no consultants, no setup
- ROI in days, not months

**4. Compound Recovery**

- Find historical errors (up to 5 years back)
- Prevent future losses (real-time monitoring)
- Optimize payment stack (routing, retry logic)

-----

## The Path to Series A

### The Story We’ll Tell

> “Mobius has discovered that every business is losing 3-5% of revenue through payment blind spots. In just 4 months, we’ve recovered $10M+ for 500 customers, with an average ROI of 847%. We’re not a reconciliation tool—we’re a revenue recovery platform with a network effect. Every new customer makes us better at finding money for all customers.”

### Metrics That Get VCs Excited

**Customer Economics:**

- CAC: $500
- Year 1 Recovery per Customer: $50,000
- Our Revenue per Customer: $7,500 (15% of recovery)
- LTV:CAC Ratio: 15:1
- Payback Period: 20 days

**Growth Metrics:**

- Revenue recovered growing 100% MoM
- Viral coefficient: 2.3 (every customer brings 2.3 more)
- NRR: 165% (customers add channels as they see results)
- Zero churn on customers who recover >$10K

**Defensibility:**

- Network effect: More customers = better pattern detection
- Embedded workflow: Become their revenue protection system
- Success alignment: Switching means losing money

### The $1B Vision

**Year 1:** Payment recovery ($10M recovered)
**Year 2:** Full stack optimization ($100M recovered)
**Year 3:** Predictive prevention ($500M protected)
**Year 5:** The financial immune system for business ($2B protected)

Every business needs this. Most don’t know it yet.

-----

## Risk Mitigation

### Risk: “What if we can’t find money for everyone?”

**Mitigation:**

- Free audit before charging
- Money-back guarantee if we don’t find 3x subscription
- Focus initial sales on high-probability segments (multi-channel, high volume)

### Risk: “Payment providers block our recovery efforts”

**Mitigation:**

- Partner approach: We help them reduce disputes
- Customer advocacy: Customers demand their money back
- Legal backing: Clear documentation of overcharges

### Risk: “Reconciliation accuracy still matters”

**Mitigation:**

- We do both: Find money AND reconcile accurately
- Position reconciliation as hygiene, recovery as growth
- Show that bad reconciliation = lost money

-----

## Conclusion

We’re not building a reconciliation tool. We’re building a revenue recovery platform that finds the 3-5% of revenue every business is losing and doesn’t know about.

The market doesn’t want better reconciliation—they want their money back.

**Let’s stop helping businesses match transactions.**
**Let’s start helping them find their missing money.**

Every business is bleeding revenue through payment blind spots.
We’re the cure.

**Execute this plan and we’re not a $100M company—we’re a $1B company.**

Because every business loses money.
And we find it.

-----

*“Revenue is vanity, profit is sanity, but cash is reality.”*
*We deliver reality.*

*Document maintained by: Product & Engineering*  
*Last updated: August 16, 2025*  
*Next review: August 19, 2025 (weekly during pivot)*
