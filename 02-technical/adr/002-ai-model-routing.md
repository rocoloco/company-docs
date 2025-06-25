# Architecture Decision Record

## ADR-002: AI Model Routing - LiteLLM Implementation

**Status**: Accepted  
**Date**: 2025-01-20  
**Author(s)**: ML Lead, VP Engineering  
**Deciders**: CTO, ML Lead, VP Engineering, Head of Infrastructure  

## Context

Mobius One requires sophisticated natural language processing to understand user queries and generate appropriate responses. We need to:
- Support multiple LLM providers (OpenAI, Anthropic, Cohere, etc.)
- Optimize for cost vs. performance per query
- Handle provider outages gracefully
- Route different query types to appropriate models
- Maintain <2 second response times
- Control costs as we scale

Building a custom model routing solution would require significant engineering effort and ongoing maintenance as new models are released.

## Decision

We will implement LiteLLM as our AI model routing layer, providing intelligent routing across multiple LLM providers with built-in fallbacks, cost optimization, and performance monitoring.

## Considered Options

### Option 1: Single Provider (OpenAI only)
- **Description**: Use only OpenAI's APIs for all NLP needs
- **Pros**:
  - Simple implementation
  - Well-documented API
  - Good performance
  - Wide model selection
- **Cons**:
  - Single point of failure
  - No cost optimization
  - Vendor lock-in
  - Limited by OpenAI rate limits

### Option 2: Custom Multi-Provider Router
- **Description**: Build our own routing layer for multiple LLM providers
- **Pros**:
  - Complete control over routing logic
  - Custom optimization algorithms
  - Proprietary advantage
- **Cons**:
  - 3-6 months development time
  - Ongoing maintenance burden
  - Complex testing requirements
  - Delayed product launch

### Option 3: LiteLLM Open Source
- **Description**: Use LiteLLM for unified interface and routing
- **Pros**:
  - 100+ LLM providers supported
  - Built-in fallbacks and retries
  - Cost tracking and optimization
  - Load balancing across providers
  - Active open source community
  - Battle-tested in production
- **Cons**:
  - Additional abstraction layer
  - Need to understand LiteLLM internals
  - Potential latency overhead
  - Dependency on open source project

### Option 4: Proprietary Router Service
- **Description**: Use a commercial routing service like Portkey or Helicone
- **Pros**:
  - Managed service with SLA
  - Advanced analytics
  - No maintenance required
- **Cons**:
  - Additional cost layer
  - Data privacy concerns
  - Less flexibility
  - Another vendor dependency

## Consequences

### Positive
- Immediate access to 100+ LLM models
- Automatic fallback during provider outages
- 40-60% cost reduction through intelligent routing
- Built-in retry logic and error handling
- Detailed cost and performance analytics
- Future-proof as new models emerge
- Can contribute improvements back to open source

### Negative
- Additional abstraction layer (5-10ms latency)
- Need to monitor LiteLLM updates
- Learning curve for team
- Debugging complexity increases
- Open source project risks

### Neutral
- Need to implement custom routing rules
- Requires Redis for caching
- Must track LiteLLM version updates

## Implementation Notes

### Architecture Integration
```python
from litellm import Router

# Configure model routing
router = Router(
    model_list=[
        {
            "model_name": "smart-router",
            "litellm_params": {
                "model": "claude-3-opus",
                "api_key": os.getenv("ANTHROPIC_API_KEY"),
            },
            "tpm": 1000000,
            "rpm": 1000,
        },
        {
            "model_name": "smart-router",
            "litellm_params": {
                "model": "gpt-4-turbo-preview",
                "api_key": os.getenv("OPENAI_API_KEY"),
            },
            "tpm": 900000,
            "rpm": 900,
        },
        {
            "model_name": "fast-router",
            "litellm_params": {
                "model": "claude-3-haiku",
                "api_key": os.getenv("ANTHROPIC_API_KEY"),
            },
            "tpm": 2000000,
            "rpm": 2000,
        }
    ],
    fallbacks=[
        {"smart-router": ["gpt-4-turbo-preview", "claude-3-opus"]},
        {"fast-router": ["claude-3-haiku", "gpt-3.5-turbo"]}
    ],
    context_window_fallbacks=[
        {"smart-router": ["claude-3-opus-200k", "gpt-4-32k"]}
    ],
    set_verbose=True,
)

# Query classification and routing
async def route_query(query: str, context: QueryContext) -> Response:
    # Classify query complexity
    if context.is_complex or context.requires_reasoning:
        model = "smart-router"
    else:
        model = "fast-router"
    
    # Make request with automatic retries and fallbacks
    response = await router.acompletion(
        model=model,
        messages=[{"role": "user", "content": query}],
        metadata={
            "user_id": context.user_id,
            "query_type": context.query_type,
        }
    )
    
    return response
```

### Routing Strategy
1. **Simple queries** → Fast models (Haiku, GPT-3.5)
2. **Complex queries** → Smart models (Opus, GPT-4)
3. **Long context** → Extended context models
4. **Code generation** → Specialized code models
5. **Cost optimization** → Route based on user tier

### Monitoring and Optimization
```python
# Track costs and performance
@router.success_callback
async def log_success(kwargs, completion_response, start_time, end_time):
    await metrics.track({
        "model": kwargs["model"],
        "latency": end_time - start_time,
        "tokens": completion_response["usage"]["total_tokens"],
        "cost": litellm.completion_cost(completion_response),
    })

# Implement custom fallback logic
@router.fallback_callback
async def handle_fallback(kwargs, completion_response, start_time, end_time):
    logger.warning(f"Fallback triggered for {kwargs['model']}")
    await alerts.notify_on_repeated_fallbacks()
```

### Cost Controls
1. Set spending limits per user/organization
2. Route free tier users to cheaper models
3. Implement token limits based on subscription
4. Cache common query patterns
5. Monitor cost per query type

## References

- [LiteLLM Documentation](https://docs.litellm.ai)
- [LiteLLM Router Docs](https://docs.litellm.ai/docs/routing)
- [Model Pricing Analysis](../finance/llm-cost-analysis.md)
- [Performance Benchmarks](../metrics/llm-performance-benchmarks.md)

---

## Review History

| Date | Reviewer | Comments |
|------|----------|----------|
| 2025-01-20 | CTO | Approved - critical for cost control and reliability |
| 2025-01-21 | CFO | Approved - 60% cost reduction projected |
| 2025-01-22 | ML Lead | Approved - provides needed flexibility |

## Notes

- Monitor LiteLLM releases for security updates
- Consider contributing enterprise features back to project
- Evaluate custom fork if we need proprietary features
- Set up automated testing for model availability