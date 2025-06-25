# Mobius One Technical Architecture

*Last updated: June 24, 2025*

## Overview

Mobius One is designed as a modern, microservices-based platform that provides a conversational AI interface for enterprise systems. The architecture prioritizes performance, scalability, and maintainability while ensuring enterprise-grade security and reliability.

## Architecture Principles

### Core Principles
1. **Microservices Architecture**: Loosely coupled services for independent scaling
2. **API-First Design**: All functionality exposed through well-defined APIs
3. **Event-Driven Communication**: Asynchronous processing for better performance
4. **Cloud-Native**: Built for Kubernetes and containerized deployment
5. **Security by Design**: Zero-trust architecture with defense in depth

### Design Goals
- Sub-second query response times
- Linear horizontal scalability
- 99.9% uptime SLA
- Multi-region deployment capability
- Real-time data synchronization

## System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                          Client Layer                            │
├─────────────────┬───────────────────┬───────────────────────────┤
│   Web App       │   Mobile App      │   API Clients             │
│   (React)       │   (React Native)  │   (REST/GraphQL)          │
└─────────────────┴───────────────────┴───────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                      API Gateway (KrakenD)                       │
│   • Rate Limiting  • Authentication  • Load Balancing           │
└─────────────────────────────────────────────────────────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        ▼                       ▼                       ▼
┌───────────────┐     ┌───────────────┐     ┌───────────────┐
│  Query Service│     │   AI Service  │     │  Integration  │
│  (Node.js)    │     │   (Python)    │     │   Service     │
└───────────────┘     └───────────────┘     └───────────────┘
        │                       │                       │
        └───────────────────────┴───────────────────────┘
                                │
                    ┌───────────┴───────────┐
                    ▼                       ▼
            ┌───────────────┐     ┌───────────────┐
            │   PostgreSQL  │     │     Redis     │
            │   (Metadata)  │     │   (Caching)   │
            └───────────────┘     └───────────────┘
```

### Component Architecture

#### 1. Client Layer
- **Web Application**: React + TypeScript with assistant-ui
- **Mobile Application**: React Native (future)
- **API Clients**: REST and GraphQL endpoints

#### 2. API Gateway Layer
- **KrakenD**: High-performance API gateway
  - Request routing and load balancing
  - Rate limiting and throttling
  - Authentication and authorization
  - Request/response transformation
  - Circuit breaking

#### 3. Core Services

##### Query Service (mobius-api)
- **Technology**: Node.js + TypeScript + Express/Fastify
- **Responsibilities**:
  - Natural language query processing
  - Query plan optimization
  - Result aggregation and formatting
  - Session management
  - Response caching

##### AI Service (mobius-ai)
- **Technology**: Python + FastAPI
- **Components**:
  - **LiteLLM Router**: Intelligent model selection
  - **Semantic Router**: Query classification
  - **WrenAI Integration**: SQL generation
  - **Confidence Scorer**: Result validation
- **Responsibilities**:
  - NLP processing
  - Intent classification
  - Entity extraction
  - Model management

##### Integration Service
- **Technology**: Node.js + TypeScript
- **Responsibilities**:
  - Merge.dev API orchestration
  - Custom connector management
  - Data synchronization
  - Webhook processing
  - Error handling and retries

#### 4. Data Layer
- **PostgreSQL**: 
  - User data and preferences
  - Query history and analytics
  - Integration configurations
  - System metadata
- **Redis**:
  - Query result caching
  - Session storage
  - Rate limiting counters
  - Pub/sub for real-time updates

### Data Flow

#### Query Processing Flow
```
1. User Query → Web App
2. Web App → API Gateway (auth, rate limit)
3. API Gateway → Query Service
4. Query Service → AI Service (classify intent)
5. AI Service → Query Service (routing plan)
6. Query Service → Integration Service (fetch data)
7. Integration Service → External APIs (parallel)
8. External APIs → Integration Service (results)
9. Integration Service → Query Service (aggregate)
10. Query Service → Redis (cache)
11. Query Service → Web App (formatted response)
```

#### Real-time Sync Flow
```
1. Webhook → API Gateway
2. API Gateway → Integration Service
3. Integration Service → Validate & Transform
4. Integration Service → PostgreSQL (update)
5. Integration Service → Redis (invalidate cache)
6. Integration Service → WebSocket (notify clients)
```

## Microservices Design

### Service Boundaries
Each service is designed around a specific business capability:

1. **Query Service**: Query orchestration and response
2. **AI Service**: Machine learning and NLP
3. **Integration Service**: External system connectivity
4. **Auth Service**: Authentication and authorization
5. **Analytics Service**: Usage tracking and insights
6. **Notification Service**: Alerts and notifications

### Service Communication
- **Synchronous**: REST/gRPC for request-response
- **Asynchronous**: RabbitMQ/Kafka for events
- **Real-time**: WebSocket for live updates

### Service Discovery
- Kubernetes DNS for internal service discovery
- Consul for advanced service mesh (future)

## Security Architecture

### Authentication & Authorization
- **OAuth 2.0/OIDC**: For user authentication
- **JWT Tokens**: For API authorization
- **RBAC**: Role-based access control
- **API Keys**: For programmatic access

### Data Security
- **Encryption at Rest**: AES-256 for database
- **Encryption in Transit**: TLS 1.3 minimum
- **Key Management**: AWS KMS/Azure Key Vault
- **Data Masking**: PII redaction in logs

### Network Security
- **Zero Trust**: No implicit trust
- **Network Policies**: Kubernetes network isolation
- **WAF**: Web application firewall
- **DDoS Protection**: CloudFlare/AWS Shield

## Scalability Design

### Horizontal Scaling
- **Stateless Services**: All services designed stateless
- **Auto-scaling**: Based on CPU/memory/queue depth
- **Load Balancing**: Round-robin with health checks
- **Database Sharding**: By tenant ID (future)

### Caching Strategy
```
L1 Cache: Application Memory (30s TTL)
L2 Cache: Redis (5m TTL)
L3 Cache: CDN (static assets)
```

### Performance Optimization
- **Query Result Caching**: Redis with smart invalidation
- **Connection Pooling**: Database and HTTP connections
- **Batch Processing**: Aggregate multiple requests
- **Lazy Loading**: On-demand data fetching

## Infrastructure

### Kubernetes Architecture
```yaml
Namespaces:
- mobius-production
- mobius-staging
- mobius-development

Core Deployments:
- api-gateway (3 replicas)
- query-service (5 replicas)
- ai-service (3 replicas)
- integration-service (5 replicas)

Supporting Services:
- postgresql (RDS)
- redis (ElastiCache)
- rabbitmq (Amazon MQ)
```

### Multi-Region Strategy
- **Primary Region**: US-East-1
- **Secondary Region**: EU-West-1
- **Database Replication**: Cross-region read replicas
- **CDN**: Global edge locations

### Monitoring & Observability
- **Metrics**: Prometheus + Grafana
- **Logging**: ELK Stack (Elasticsearch, Logstash, Kibana)
- **Tracing**: Jaeger for distributed tracing
- **APM**: DataDog/New Relic

## Development Practices

### CI/CD Pipeline
```
1. Code Commit → GitHub
2. GitHub Actions → Run Tests
3. Build Docker Images
4. Push to Registry
5. Deploy to Staging
6. Run E2E Tests
7. Deploy to Production (manual approval)
8. Smoke Tests
9. Monitoring Alerts
```

### Testing Strategy
- **Unit Tests**: 80% minimum coverage
- **Integration Tests**: Service boundaries
- **Contract Tests**: API contracts
- **E2E Tests**: Critical user journeys
- **Load Tests**: Performance validation

### Code Quality
- **Linting**: ESLint for TypeScript, Black for Python
- **Type Checking**: TypeScript strict mode
- **Code Review**: Required for all PRs
- **Security Scanning**: Snyk/SonarQube

## Disaster Recovery

### Backup Strategy
- **Database**: Daily automated backups, 30-day retention
- **File Storage**: S3 with versioning
- **Configuration**: Git-based config management

### Recovery Objectives
- **RTO**: 1 hour (Recovery Time Objective)
- **RPO**: 15 minutes (Recovery Point Objective)
- **Failover**: Automated for critical services

### Incident Response
1. Automated alerting
2. On-call rotation
3. Runbook automation
4. Post-mortem process

## Technology Stack Summary

### Languages
- **TypeScript**: Frontend and Node.js backend
- **Python**: AI/ML services
- **Go**: KrakenD configuration only
- **SQL**: Database queries

### Frameworks
- **Frontend**: React + assistant-ui
- **Backend**: Express/Fastify (Node.js), FastAPI (Python)
- **Testing**: Jest, Pytest
- **ORM**: Prisma (Node.js), SQLAlchemy (Python)

### Infrastructure
- **Container**: Docker
- **Orchestration**: Kubernetes
- **Cloud**: AWS/Azure (multi-cloud)
- **CI/CD**: GitHub Actions
- **Monitoring**: Prometheus + Grafana

### Third-Party Services
- **Integrations**: Merge.dev
- **LLM**: OpenAI/Anthropic via LiteLLM
- **Analytics**: Segment/Mixpanel
- **Error Tracking**: Sentry

## Future Considerations

### Short Term (6 months)
- GraphQL API support
- Advanced caching strategies
- Service mesh implementation
- Enhanced monitoring

### Medium Term (12 months)
- Multi-tenant database architecture
- Event sourcing for audit trail
- Edge computing for global performance
- Advanced ML model deployment

### Long Term (24 months)
- Blockchain for audit integrity
- Quantum-resistant encryption
- Self-healing infrastructure
- Predictive scaling

## Appendices

### A. API Design Guidelines
- RESTful principles
- Consistent naming conventions
- Versioning strategy
- Error handling standards

### B. Database Schema
- User and organization models
- Query history structure
- Integration configuration
- Caching strategies

### C. Security Checklist
- OWASP Top 10 compliance
- Regular penetration testing
- Security training requirements
- Incident response procedures

---
*This architecture document is maintained by the engineering team and updated with each major technical decision.*