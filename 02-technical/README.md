# Technical Documentation

This directory contains all technical documentation, architecture decisions, API specifications, and deployment guides for Mobius One.

## Contents

### Architecture Decision Records (ADR)
- **[ADR Template](adr/template.md)** - Template for new architecture decisions
- **[001: Integration Strategy](adr/001-integration-strategy.md)** - Merge.dev partnership decision
- **[002: AI Model Routing](adr/002-ai-model-routing.md)** - LiteLLM implementation
- **[003: Tech Stack Decisions](adr/003-tech-stack-decisions.md)** - Core technology choices

### API Specifications
- **[Mobius API v1](api-specs/mobius-api-v1.yaml)** - OpenAPI specification
- **[Merge Integration Guide](api-specs/merge-integration.md)** - Merge.dev integration patterns
- **[LiteLLM Routing](api-specs/litellm-routing.md)** - AI model routing configuration

### Deployment Guides
- **[Kubernetes Setup](deployment/kubernetes/)** - K8s manifests and guides
- **[Docker Compose](deployment/docker-compose.yml)** - Local development setup
- **[Production Setup](deployment/production-setup.md)** - Production deployment guide
- **[Monitoring](deployment/monitoring.md)** - Observability and monitoring setup

### Security Documentation
- **[Security Model](security/security-model.md)** - Overall security architecture
- **[Compliance Checklist](security/compliance-checklist.md)** - SOC 2, GDPR requirements
- **[Incident Response](security/incident-response.md)** - Security incident procedures

### Data Architecture
- **[Data Architecture](data-architecture.md)** - Data flow and storage design

## Documentation Standards

- Use clear, concise language
- Include code examples where relevant
- Keep diagrams up to date (use Mermaid when possible)
- Version all API specifications
- Document decisions in ADR format

## For Developers

See [CLAUDE.md](../CLAUDE.md) for development standards and coding guidelines.

---
*Last updated: June 24, 2025*