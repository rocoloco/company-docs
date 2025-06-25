# Mobius One Company Documentation

> The AI Interface Layer for Enterprise Systems

## Quick Navigation

### 📋 Strategy & Planning
- [Business Plan](01-strategy/business-plan.md) - Complete business strategy
- [Product Requirements](01-strategy/prd.md) - Product roadmap and features
- [Technical Architecture](01-strategy/technical-architecture.md) - System design

### 🔧 Technical Documentation
- [Architecture Decisions](02-technical/adr/) - Technical decision history
- [API Specifications](02-technical/api-specs/) - API documentation
- [Deployment Guides](02-technical/deployment/) - Infrastructure setup

### 🚀 Operations
- [Hiring Plan](03-operations/hiring/hiring-plan.md) - Team building strategy
- [OKRs](03-operations/metrics/okrs.md) - Company objectives
- [Go-to-Market](03-operations/go-to-market/) - Sales and marketing

### 💰 Fundraising
- [Investor Updates](05-fundraising/investor-updates/) - Monthly updates
- [Pitch Materials](05-fundraising/pitch-materials/) - Presentation materials

## Document Standards

- All documents in Markdown format
- Use relative links between documents
- Include last updated date in each document
- Follow naming convention: lowercase-with-hyphens.md

## For Developers

This repository is optimized for Claude Code integration with development context:

```bash
# Claude Code will automatically read CLAUDE.md for development standards
claude-code --context ./company-docs "Review PRD for technical inconsistencies"
claude-code --context ./mobius-api "Implement user authentication with TDD"
claude-code --context ./ "Update investor deck with latest metrics"
```

**Development Context**: See [CLAUDE.md](CLAUDE.md) for complete development standards including TDD, TypeScript guidelines, and performance requirements.

---
*Last updated: June 24, 2025*
*Maintained by: Mobius One Team*