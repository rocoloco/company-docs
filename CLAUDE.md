# CLAUDE.md - Mobius One Development Context

*This file provides Claude Code with essential development practices, technical preferences, and coding standards for Mobius One.*

---

## **Context Loading Verification**

**FIRST INTERACTION ONLY**: When starting a new session, confirm context by stating:
"🚀 Mobius One dev context active: TDD + TypeScript strict + <500ms targets + Revenue ops domain"

This verifies you understand our core development principles and business domain.

**CONTEXT QUALITY CHECKS**: 
Your responses should demonstrate understanding of:
- ✅ Test-first development (mention tests in implementation suggestions)
- ✅ TypeScript strict patterns (no `any`, proper typing)
- ✅ Performance awareness (mention caching, timeouts, response times)
- ✅ Business domain knowledge (revenue, CFO users, enterprise needs)

**DO NOT** repeat confirmation for subsequent tasks in the same conversation.

**CONTEXT DRIFT INDICATORS**: If you find yourself suggesting `any` types, ignoring tests, or providing generic solutions without business context, re-read this entire file.

---

## **Development Philosophy**

### **Core Principles**
- **Test-Driven Development (TDD)** - Write tests first, then implementation
- **Clean Architecture** - Separation of concerns, dependency injection
- **Type Safety** - Leverage TypeScript strictly, no `any` types
- **Performance First** - Sub-second response times are non-negotiable
- **Enterprise Ready** - Production-grade code from day one

### **Quality Standards**
- **Code Coverage**: Minimum 80%, target 90%
- **Performance**: <500ms API responses, <2s complex queries
- **Security**: All inputs validated, no SQL injection vectors
- **Documentation**: Self-documenting code with business context
- **Testability**: All code must be unit testable

*Remember: Always start with tests (TDD)*

---

## **Technical Stack & Preferences**

### **Primary Languages**
- **TypeScript**: All frontend and Node.js backend code
- **Python**: AI/ML services only (LiteLLM, WrenAI, Splink)
- **Go**: KrakenD configuration only (don't write custom Go)
- **SQL**: PostgreSQL for queries, Redis for caching

### **Framework Preferences**
```typescript
// Frontend: React + TypeScript + assistant-ui
import { useAssistant } from '@assistant-ui/react';

// Backend: Express or Fastify with TypeScript
import { FastifyInstance } from 'fastify';

// Testing: Jest + Supertest for API testing
import { describe, it, expect, beforeEach } from '@jest/globals';

// Database: Prisma ORM for type safety
import { PrismaClient } from '@prisma/client';
```

*Remember: TypeScript strict mode, no any types*

---

## **Test-Driven Development Standards**

### **TDD Workflow**
1. **Red**: Write failing test first
2. **Green**: Write minimal code to pass
3. **Refactor**: Clean up while keeping tests green
4. **Repeat**: For every feature, bug fix, refactor

### **Testing Hierarchy**
```typescript
// 1. Unit Tests (70% of tests)
describe('RevenueAnalyticsService', () => {
  it('should calculate DSO correctly for given invoices', () => {
    const service = new RevenueAnalyticsService();
    const invoices = [mockInvoice1, mockInvoice2];
    
    const dso = service.calculateDSO(invoices);
    
    expect(dso).toBe(45.5);
  });
});

// 2. Integration Tests (20% of tests)
describe('Salesforce Integration', () => {
  it('should fetch and transform opportunity data', async () => {
    const integration = new SalesforceIntegration(mockConfig);
    
    const opportunities = await integration.getOpportunities();
    
    expect(opportunities).toHaveLength(3);
    expect(opportunities[0]).toMatchSchema(OpportunitySchema);
  });
});

// 3. E2E Tests (10% of tests)
describe('Revenue Query Workflow', () => {
  it('should answer "deals closed not invoiced" query', async () => {
    const response = await request(app)
      .post('/api/queries')
      .send({ text: 'deals closed not invoiced' });
    
    expect(response.status).toBe(200);
    expect(response.body.confidence).toBeGreaterThan(0.8);
  });
});
```

### **Test Organization**
```
src/
├── services/
│   ├── revenueAnalytics.service.ts
│   └── __tests__/
│       ├── revenueAnalytics.service.test.ts
│       └── revenueAnalytics.integration.test.ts
├── controllers/
│   ├── query.controller.ts
│   └── __tests__/
│       └── query.controller.test.ts
└── __tests__/
    ├── e2e/
    │   └── revenue-queries.e2e.test.ts
    └── fixtures/
        └── mockData.ts
```

---

## **Code Quality Standards**

### **TypeScript Configuration**
```typescript
// tsconfig.json requirements
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true
  }
}

// No any types allowed
const badExample: any = getData(); // ❌
const goodExample: ApiResponse<Customer> = getData(); // ✅

// Prefer type unions over any
type QueryStatus = 'pending' | 'processing' | 'completed' | 'failed'; // ✅
```

### **Error Handling Patterns**
```typescript
// Use Result pattern for operations that can fail
type Result<T, E = Error> = { success: true; data: T } | { success: false; error: E };

class QueryService {
  async processQuery(query: string): Promise<Result<QueryResponse, QueryError>> {
    try {
      const result = await this.executeQuery(query);
      return { success: true, data: result };
    } catch (error) {
      return { 
        success: false, 
        error: new QueryError('Failed to process query', { cause: error })
      };
    }
  }
}

// Always handle errors explicitly
const result = await queryService.processQuery(userInput);
if (!result.success) {
  logger.error('Query failed', { error: result.error, query: userInput });
  return res.status(500).json({ error: result.error.message });
}
```

*Remember: <500ms response times required*

### **Performance Requirements**
```typescript
// All async operations must have timeouts
const response = await Promise.race([
  fetchFromSalesforce(query),
  new Promise((_, reject) => 
    setTimeout(() => reject(new Error('Timeout')), 5000)
  )
]);

// Cache expensive operations
@Cacheable({ ttl: 300 }) // 5 minutes
async getCustomerData(customerId: string): Promise<Customer> {
  return await this.mergeApi.fetchCustomer(customerId);
}

// Monitor performance in tests
it('should respond within 500ms', async () => {
  const start = Date.now();
  
  await request(app).get('/api/revenue/summary');
  
  expect(Date.now() - start).toBeLessThan(500);
});
```

---

## **Architecture Patterns**

### **Clean Architecture Layers**
```typescript
// Domain Layer (business logic)
export class Revenue {
  constructor(
    private readonly amount: number,
    private readonly date: Date,
    private readonly customerId: string
  ) {}
  
  isOverdue(currentDate: Date): boolean {
    const daysDiff = differenceInDays(currentDate, this.date);
    return daysDiff > 30;
  }
}

// Application Layer (use cases)
export class CalculateDSOUseCase {
  constructor(
    private readonly invoiceRepo: InvoiceRepository,
    private readonly paymentRepo: PaymentRepository
  ) {}
  
  async execute(params: DSOParams): Promise<DSOResult> {
    // Use case implementation
  }
}

// Infrastructure Layer (external dependencies)
export class SalesforceInvoiceRepository implements InvoiceRepository {
  constructor(private readonly mergeClient: MergeClient) {}
  
  async findByCustomer(customerId: string): Promise<Invoice[]> {
    // Implementation
  }
}
```

### **Dependency Injection**
```typescript
// Use dependency injection for testability
export class QueryController {
  constructor(
    private readonly queryService: QueryService,
    private readonly validator: QueryValidator,
    private readonly logger: Logger
  ) {}
  
  async handleQuery(req: Request, res: Response): Promise<void> {
    const validationResult = await this.validator.validate(req.body);
    if (!validationResult.isValid) {
      return res.status(400).json({ errors: validationResult.errors });
    }
    
    // Process query...
  }
}

// Easy to mock in tests
const mockQueryService = {
  processQuery: jest.fn().mockResolvedValue(mockResponse)
};
const controller = new QueryController(mockQueryService, mockValidator, mockLogger);
```

*Remember: Enterprise customers (CFOs) are the users*

### **API Design Standards**
```typescript
// RESTful endpoints with proper HTTP methods
GET    /api/revenue/summary           // Get revenue summary
GET    /api/revenue/leakage          // Get revenue leakage data
POST   /api/queries                  // Process natural language query
GET    /api/customers/:id/invoices   // Get customer invoices

// Request/Response schemas
interface QueryRequest {
  text: string;
  context?: {
    dateRange?: DateRange;
    filters?: QueryFilters;
  };
}

interface QueryResponse {
  answer: string;
  confidence: number; // 0-1
  sources: DataSource[];
  executionTime: number; // milliseconds
  suggestedActions?: Action[];
}

// OpenAPI specification required
/**
 * @swagger
 * /api/queries:
 *   post:
 *     summary: Process natural language query
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             $ref: '#/components/schemas/QueryRequest'
 */
```

---

## **Business Context Integration**

### **Target Users & Use Cases**
- **Primary Users**: CFOs at mid-market companies (Sarah persona)
- **Secondary Users**: RevOps managers (Mike persona)
- **Core Use Cases**: 
  - "Which customers closed deals but haven't paid invoices?"
  - Revenue leakage detection
  - Pipeline accuracy analysis
  - DSO trends and aging reports

### **Enterprise Requirements**
- **Performance**: <500ms simple queries, <2s complex queries
- **Security**: SOC 2 compliance, no data storage (pass-through only)
- **Reliability**: 99.9% uptime, enterprise SLAs
- **Integrations**: Salesforce, NetSuite, HubSpot via Merge.dev

*Remember: Revenue operations is the business domain*

---

## **Development Workflow**

### **Git Standards**
```bash
# Conventional commits required
feat(auth): add enterprise SSO integration
fix(query): resolve timeout issues for complex queries
test(revenue): add DSO calculation unit tests
docs(api): update OpenAPI specification

# Branch naming
feature/enterprise-sso-integration
bugfix/query-timeout-resolution
hotfix/critical-auth-vulnerability
```

### **Code Review Checklist**
- [ ] **Tests written first** (TDD followed)
- [ ] **All tests passing** with >80% coverage
- [ ] **TypeScript strict mode** compliance
- [ ] **Performance requirements** met (<500ms APIs)
- [ ] **Error handling** implemented properly
- [ ] **Security considerations** addressed
- [ ] **Documentation** updated (API specs, README)
- [ ] **No console.log** statements in production code

### **CI/CD Pipeline Requirements**
```yaml
# Required pipeline steps
- Lint (ESLint + Prettier)
- Type Check (TypeScript compiler)
- Unit Tests (Jest with coverage)
- Integration Tests (Supertest)
- Security Scan (npm audit + Snyk)
- Performance Tests (Artillery/k6)
- Build (TypeScript compilation)
- Deploy (staging → production)
```

---

## **Common Patterns & Preferences**

### **File Naming & Structure**
```typescript
// Use domain-driven structure
src/
├── revenue/
│   ├── revenue.entity.ts          // Domain entities
│   ├── revenue.repository.ts      // Repository interface
│   ├── revenue.service.ts         // Business logic
│   ├── revenue.controller.ts      // HTTP handlers
│   ├── revenue.dto.ts            // Data transfer objects
│   └── __tests__/                // Colocated tests
├── shared/
│   ├── types/                    // Shared TypeScript types
│   ├── utils/                    // Pure utility functions
│   └── middleware/               // Express middleware
└── infrastructure/
    ├── database/                 // Prisma models
    ├── external/                 // Third-party integrations
    └── config/                   // Configuration
```

### **Function Design Principles**
```typescript
// Pure functions preferred (easier to test)
export function calculateDSO(invoices: Invoice[]): number {
  if (invoices.length === 0) return 0;
  
  const totalAmount = invoices.reduce((sum, inv) => sum + inv.amount, 0);
  const totalDays = invoices.reduce((sum, inv) => sum + inv.daysOutstanding, 0);
  
  return totalDays / invoices.length;
}

// Small, focused functions
export class InvoiceService {
  // Each method does one thing well
  async findOverdueInvoices(customerId: string): Promise<Invoice[]> { }
  async calculateAging(invoices: Invoice[]): Promise<AgingReport> { }
  async sendPaymentReminder(invoiceId: string): Promise<void> { }
}

// Avoid long parameter lists
interface PaymentReminderOptions {
  templateId: string;
  recipientEmail: string;
  urgencyLevel: 'low' | 'medium' | 'high';
  includeStatement: boolean;
}

async sendPaymentReminder(options: PaymentReminderOptions): Promise<void> {
  // Implementation
}
```

### **Testing Utilities**
```typescript
// Test data builders for complex objects
export class InvoiceBuilder {
  private invoice: Partial<Invoice> = {};
  
  withAmount(amount: number): this {
    this.invoice.amount = amount;
    return this;
  }
  
  overdue(days: number): this {
    this.invoice.dueDate = subDays(new Date(), days);
    return this;
  }
  
  build(): Invoice {
    return {
      id: randomUUID(),
      amount: 1000,
      dueDate: new Date(),
      ...this.invoice
    } as Invoice;
  }
}

// Usage in tests
const overdueInvoice = new InvoiceBuilder()
  .withAmount(5000)
  .overdue(45)
  .build();
```

---

## **Performance & Monitoring**

### **Performance Testing**
```typescript
// Load test critical endpoints
describe('Query Performance', () => {
  it('should handle 100 concurrent queries', async () => {
    const promises = Array(100).fill(0).map(() => 
      request(app).post('/api/queries').send(mockQuery)
    );
    
    const responses = await Promise.all(promises);
    
    responses.forEach(response => {
      expect(response.status).toBe(200);
      expect(response.body.executionTime).toBeLessThan(500);
    });
  });
});

// Memory leak detection
it('should not leak memory during query processing', async () => {
  const initialMemory = process.memoryUsage().heapUsed;
  
  // Process 1000 queries
  for (let i = 0; i < 1000; i++) {
    await queryService.processQuery(mockQuery);
  }
  
  global.gc?.(); // Force garbage collection
  const finalMemory = process.memoryUsage().heapUsed;
  
  expect(finalMemory - initialMemory).toBeLessThan(50 * 1024 * 1024); // 50MB threshold
});
```

### **Monitoring & Observability**
```typescript
// Structured logging with context
import { Logger } from 'winston';

export class QueryService {
  constructor(private readonly logger: Logger) {}
  
  async processQuery(query: string, userId: string): Promise<QueryResponse> {
    const correlationId = randomUUID();
    const startTime = Date.now();
    
    this.logger.info('Query processing started', {
      correlationId,
      userId,
      queryLength: query.length,
      timestamp: new Date().toISOString()
    });
    
    try {
      const result = await this.executeQuery(query);
      
      this.logger.info('Query processing completed', {
        correlationId,
        executionTime: Date.now() - startTime,
        confidence: result.confidence,
        sourceCount: result.sources.length
      });
      
      return result;
    } catch (error) {
      this.logger.error('Query processing failed', {
        correlationId,
        error: error.message,
        stack: error.stack,
        executionTime: Date.now() - startTime
      });
      throw error;
    }
  }
}
```

---

## **Security Standards**

### **Input Validation**
```typescript
// Use Zod for runtime type checking
import { z } from 'zod';

const QueryRequestSchema = z.object({
  text: z.string().min(1).max(1000),
  context: z.object({
    dateRange: z.object({
      start: z.string().datetime(),
      end: z.string().datetime()
    }).optional(),
    filters: z.array(z.string()).optional()
  }).optional()
});

export async function validateQueryRequest(req: Request): Promise<QueryRequest> {
  try {
    return QueryRequestSchema.parse(req.body);
  } catch (error) {
    throw new ValidationError('Invalid query request', { cause: error });
  }
}
```

### **Authentication & Authorization**
```typescript
// JWT validation middleware
export function requireAuth(req: AuthenticatedRequest, res: Response, next: NextFunction): void {
  const token = req.headers.authorization?.replace('Bearer ', '');
  
  if (!token) {
    return res.status(401).json({ error: 'Authentication required' });
  }
  
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET!);
    req.user = payload as UserPayload;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
}

// Role-based access control
export function requireRole(role: UserRole) {
  return (req: AuthenticatedRequest, res: Response, next: NextFunction): void => {
    if (!req.user || req.user.role !== role) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    next();
  };
}
```

---

## **Session Context Management**

### **Context Retention Guidelines**
- **New sessions**: Confirm understanding once with verification message
- **Long sessions (10+ interactions)**: Self-check that you're still following TDD, TypeScript strict mode, performance requirements
- **Context quality indicators**: Always mention tests, proper typing, performance considerations, business domain

### **Context Validation Questions**
Before suggesting any implementation, verify you understand:
1. Should I write tests first? (Answer: Yes, always TDD)
2. What's the performance target? (Answer: <500ms for APIs, <2s for complex queries)
3. What TypeScript strictness? (Answer: Strict mode, no `any` types)
4. What's the business domain? (Answer: Revenue operations, Salesforce/NetSuite integration)
5. What's the primary user? (Answer: CFOs and RevOps managers)

---

## **Common Pitfalls to Avoid**

### **Code Smells to Watch For**
- **Large functions** (>20 lines) - break them down
- **Deep nesting** (>3 levels) - use early returns
- **Magic numbers** - use named constants
- **Mutable global state** - use dependency injection
- **Catching and ignoring errors** - always handle properly

### **Performance Anti-Patterns**
- **N+1 queries** - use proper joins or batching
- **Synchronous file operations** - always use async
- **Memory leaks** - properly clean up event listeners
- **Blocking the event loop** - avoid heavy CPU operations

### **Testing Anti-Patterns**
- **Testing implementation details** - test behavior, not internals
- **Flaky tests** - ensure deterministic test data
- **Slow tests** - mock external dependencies
- **No edge case testing** - test error conditions

### **Context Loss Indicators**
- Suggesting `any` types in TypeScript
- Not mentioning tests in implementation plans
- Ignoring performance requirements
- Providing generic solutions without business context
- Violating clean architecture principles

---

*Last updated: June 24, 2025*
*This file should be updated whenever development standards change*
*Context should be validated at the start of each new session*