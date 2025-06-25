# Mobius One Monitoring & Observability Guide

*Last updated: June 24, 2025*

## Overview

This guide covers the comprehensive monitoring strategy for Mobius One, including metrics collection, log aggregation, distributed tracing, alerting, and incident response procedures. Our monitoring philosophy follows the "Three Pillars of Observability": Metrics, Logs, and Traces.

## Monitoring Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Applications                             │
├──────────────┬──────────────┬──────────────┬───────────────┤
│  Mobius API  │  Mobius AI   │ Integration  │   Frontend    │
│  (Node.js)   │  (Python)    │  Service     │   (React)     │
└──────┬───────┴──────┬───────┴──────┬───────┴───────┬───────┘
       │              │              │               │
       ▼              ▼              ▼               ▼
┌─────────────────────────────────────────────────────────────┐
│                    Telemetry Collection                      │
├──────────────┬──────────────┬──────────────┬───────────────┤
│ Prometheus   │  Fluent Bit  │    Jaeger    │   DataDog     │
│  (Metrics)   │   (Logs)     │   (Traces)   │  (APM)        │
└──────┬───────┴──────┬───────┴──────┬───────┴───────┬───────┘
       │              │              │               │
       ▼              ▼              ▼               ▼
┌─────────────────────────────────────────────────────────────┐
│                    Storage & Analysis                        │
├──────────────┬──────────────┬──────────────┬───────────────┤
│  Prometheus  │Elasticsearch │   Jaeger     │   DataDog     │
│   Server     │   Cluster    │   Backend    │   Cloud       │
└──────┬───────┴──────┬───────┴──────┬───────┴───────┬───────┘
       │              │              │               │
       ▼              ▼              ▼               ▼
┌─────────────────────────────────────────────────────────────┐
│                    Visualization                             │
├──────────────┬──────────────┬──────────────┬───────────────┤
│   Grafana    │    Kibana    │  Jaeger UI   │ DataDog UI    │
└──────────────┴──────────────┴──────────────┴───────────────┘
```

## Metrics Collection

### 1. Application Metrics

#### Node.js Services (mobius-api, integration-service)
```typescript
// src/metrics/index.ts
import { Registry, Counter, Histogram, Gauge, Summary } from 'prom-client';

export const registry = new Registry();

// Business metrics
export const queryCounter = new Counter({
  name: 'mobius_queries_total',
  help: 'Total number of queries processed',
  labelNames: ['status', 'query_type', 'user_tier'],
  registers: [registry]
});

export const queryDuration = new Histogram({
  name: 'mobius_query_duration_seconds',
  help: 'Query processing duration',
  labelNames: ['query_type', 'integration'],
  buckets: [0.1, 0.5, 1, 2, 5, 10],
  registers: [registry]
});

export const activeConnections = new Gauge({
  name: 'mobius_active_connections',
  help: 'Number of active WebSocket connections',
  registers: [registry]
});

export const integrationHealth = new Gauge({
  name: 'mobius_integration_health',
  help: 'Health status of integrations (1=healthy, 0=unhealthy)',
  labelNames: ['integration', 'customer_id'],
  registers: [registry]
});

// Track revenue-impacting metrics
export const revenueImpactedQueries = new Summary({
  name: 'mobius_revenue_impacted_amount',
  help: 'Dollar amount of revenue impacted by queries',
  labelNames: ['query_type', 'action_taken'],
  percentiles: [0.5, 0.9, 0.95, 0.99],
  registers: [registry]
});

// Middleware to track HTTP metrics
export const httpMetricsMiddleware = (req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequestDuration.labels(req.method, req.route?.path || 'unknown', res.statusCode.toString()).observe(duration);
    httpRequestsTotal.labels(req.method, req.route?.path || 'unknown', res.statusCode.toString()).inc();
  });
  
  next();
};
```

#### Python Services (mobius-ai)
```python
# src/metrics.py
from prometheus_client import Counter, Histogram, Gauge, Summary
import time
from functools import wraps

# AI/ML specific metrics
model_inference_counter = Counter(
    'mobius_model_inference_total',
    'Total number of model inferences',
    ['model', 'provider', 'status']
)

model_inference_duration = Histogram(
    'mobius_model_inference_duration_seconds',
    'Model inference duration',
    ['model', 'provider'],
    buckets=[0.1, 0.5, 1, 2, 5, 10, 30]
)

model_token_usage = Summary(
    'mobius_model_token_usage',
    'Token usage per request',
    ['model', 'provider']
)

model_cost_tracker = Summary(
    'mobius_model_cost_dollars',
    'Cost per model inference in dollars',
    ['model', 'provider', 'customer_tier']
)

query_confidence = Histogram(
    'mobius_query_confidence_score',
    'Confidence scores for query responses',
    ['query_type'],
    buckets=[0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 0.95, 0.99]
)

# Decorator for tracking model performance
def track_model_metrics(model_name, provider):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            start_time = time.time()
            try:
                result = await func(*args, **kwargs)
                model_inference_counter.labels(model_name, provider, 'success').inc()
                
                # Track token usage if available
                if hasattr(result, 'usage'):
                    model_token_usage.labels(model_name, provider).observe(result.usage.total_tokens)
                    
                    # Calculate cost
                    cost = calculate_cost(model_name, result.usage)
                    model_cost_tracker.labels(model_name, provider, kwargs.get('customer_tier', 'standard')).observe(cost)
                
                return result
            except Exception as e:
                model_inference_counter.labels(model_name, provider, 'error').inc()
                raise
            finally:
                duration = time.time() - start_time
                model_inference_duration.labels(model_name, provider).observe(duration)
        
        return wrapper
    return decorator
```

### 2. Infrastructure Metrics

#### Kubernetes Metrics
```yaml
# kubernetes/monitoring/kube-state-metrics.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kube-state-metrics
  namespace: monitoring
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kube-state-metrics
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kube-state-metrics
  template:
    metadata:
      labels:
        app: kube-state-metrics
    spec:
      serviceAccountName: kube-state-metrics
      containers:
      - name: kube-state-metrics
        image: quay.io/coreos/kube-state-metrics:v2.9.2
        ports:
        - containerPort: 8080
          name: http-metrics
        - containerPort: 8081
          name: telemetry
```

#### Node Exporter for System Metrics
```yaml
# kubernetes/monitoring/node-exporter-daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      hostNetwork: true
      hostPID: true
      containers:
      - name: node-exporter
        image: prom/node-exporter:v1.6.0
        args:
        - --path.procfs=/host/proc
        - --path.sysfs=/host/sys
        - --path.rootfs=/host/root
        ports:
        - containerPort: 9100
          hostPort: 9100
          name: metrics
        volumeMounts:
        - name: proc
          mountPath: /host/proc
          readOnly: true
        - name: sys
          mountPath: /host/sys
          readOnly: true
        - name: root
          mountPath: /host/root
          readOnly: true
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
      - name: root
        hostPath:
          path: /
```

### 3. Database Metrics

#### PostgreSQL Monitoring
```yaml
# kubernetes/monitoring/postgres-exporter.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-exporter
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres-exporter
  template:
    metadata:
      labels:
        app: postgres-exporter
    spec:
      containers:
      - name: postgres-exporter
        image: prometheuscommunity/postgres-exporter:v0.13.2
        env:
        - name: DATA_SOURCE_NAME
          valueFrom:
            secretKeyRef:
              name: postgres-exporter-secret
              key: datasource
        ports:
        - containerPort: 9187
          name: metrics
```

Custom PostgreSQL queries for business metrics:
```sql
-- queries.yaml for postgres_exporter
mobius_active_customers:
  query: |
    SELECT COUNT(DISTINCT customer_id) as active_customers
    FROM queries
    WHERE created_at > NOW() - INTERVAL '24 hours'
  metrics:
    - active_customers:
        usage: "GAUGE"
        description: "Number of active customers in last 24 hours"

mobius_query_complexity:
  query: |
    SELECT 
      query_type,
      COUNT(*) as count,
      AVG(execution_time_ms) as avg_time,
      MAX(execution_time_ms) as max_time
    FROM queries
    WHERE created_at > NOW() - INTERVAL '1 hour'
    GROUP BY query_type
  metrics:
    - count:
        usage: "GAUGE"
        description: "Query count by type"
    - avg_time:
        usage: "GAUGE"
        description: "Average query execution time"
    - max_time:
        usage: "GAUGE"
        description: "Maximum query execution time"
```

## Log Aggregation

### 1. Application Logging

#### Structured Logging Configuration
```typescript
// src/utils/logger.ts
import winston from 'winston';
import { ElasticsearchTransport } from 'winston-elasticsearch';

const esTransportOpts = {
  level: 'info',
  clientOpts: {
    node: process.env.ELASTICSEARCH_URL || 'http://elasticsearch:9200',
    auth: {
      username: process.env.ELASTICSEARCH_USERNAME,
      password: process.env.ELASTICSEARCH_PASSWORD
    }
  },
  index: 'mobius-logs',
  dataStream: true
};

export const logger = winston.createLogger({
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: {
    service: process.env.SERVICE_NAME || 'mobius-api',
    environment: process.env.NODE_ENV || 'development',
    version: process.env.APP_VERSION || 'unknown'
  },
  transports: [
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple()
      )
    }),
    new ElasticsearchTransport(esTransportOpts)
  ]
});

// Request context middleware
export const requestLogger = (req, res, next) => {
  const requestId = req.headers['x-request-id'] || generateRequestId();
  req.requestId = requestId;
  
  // Log request
  logger.info('Request received', {
    requestId,
    method: req.method,
    path: req.path,
    ip: req.ip,
    userAgent: req.headers['user-agent']
  });
  
  // Log response
  const originalSend = res.send;
  res.send = function(data) {
    logger.info('Request completed', {
      requestId,
      statusCode: res.statusCode,
      responseTime: Date.now() - req.startTime
    });
    originalSend.call(this, data);
  };
  
  next();
};

// Business event logging
export const logBusinessEvent = (event: string, metadata: any) => {
  logger.info('Business event', {
    event,
    ...metadata,
    timestamp: new Date().toISOString()
  });
};

// Usage example
logBusinessEvent('query_executed', {
  userId: user.id,
  queryType: 'revenue_analysis',
  integrations: ['salesforce', 'netsuite'],
  resultCount: 42,
  executionTime: 1234
});
```

### 2. Log Collection with Fluent Bit

```yaml
# kubernetes/logging/fluent-bit-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
  namespace: logging
data:
  fluent-bit.conf: |
    [SERVICE]
        Daemon Off
        Flush 5
        Log_Level info
        Parsers_File parsers.conf
        Parsers_File custom_parsers.conf
        HTTP_Server On
        HTTP_Listen 0.0.0.0
        HTTP_Port 2020
        Health_Check On

    [INPUT]
        Name tail
        Path /var/log/containers/*_mobius-production_*.log
        Parser docker
        Tag kube.*
        Refresh_Interval 5
        Mem_Buf_Limit 50MB
        Skip_Long_Lines On
        Exclude_Path /var/log/containers/*_kube-system_*.log,/var/log/containers/*_kube-public_*.log

    [FILTER]
        Name kubernetes
        Match kube.*
        Kube_URL https://kubernetes.default.svc:443
        Kube_CA_File /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        Kube_Token_File /var/run/secrets/kubernetes.io/serviceaccount/token
        Kube_Tag_Prefix kube.var.log.containers.
        Merge_Log On
        Keep_Log Off
        K8S-Logging.Parser On
        K8S-Logging.Exclude On

    [FILTER]
        Name nest
        Match kube.*
        Operation lift
        Nested_under log

    [FILTER]
        Name modify
        Match kube.*
        Add cluster_name production
        Add environment production

    [OUTPUT]
        Name es
        Match kube.*
        Host elasticsearch.logging.svc.cluster.local
        Port 9200
        Logstash_Format On
        Logstash_Prefix mobius
        Logstash_DateFormat %Y.%m.%d
        Type _doc
        Retry_Limit 5
        Buffer_Size 5MB

    [OUTPUT]
        Name datadog
        Match kube.*
        Host http-intake.logs.datadoghq.com
        TLS on
        compress gzip
        apikey ${DD_API_KEY}
        dd_service mobius
        dd_source kubernetes
        dd_tags env:production

  custom_parsers.conf: |
    [PARSER]
        Name json_parser
        Format json
        Time_Key timestamp
        Time_Format %Y-%m-%dT%H:%M:%S.%LZ

    [PARSER]
        Name mobius_app_parser
        Format regex
        Regex ^(?<timestamp>[^ ]+) (?<level>[^ ]+) \[(?<service>[^\]]+)\] (?<message>.*)$
        Time_Key timestamp
        Time_Format %Y-%m-%dT%H:%M:%S.%LZ
```

### 3. Log Analysis Queries

#### Kibana Saved Searches
```json
// Error Analysis Dashboard
{
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "@timestamp": {
              "gte": "now-1h"
            }
          }
        },
        {
          "term": {
            "level": "error"
          }
        }
      ]
    }
  },
  "aggs": {
    "errors_by_service": {
      "terms": {
        "field": "service.keyword",
        "size": 10
      }
    },
    "error_timeline": {
      "date_histogram": {
        "field": "@timestamp",
        "interval": "5m"
      }
    }
  }
}

// Query Performance Analysis
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "event": "query_executed"
          }
        }
      ]
    }
  },
  "aggs": {
    "avg_execution_time": {
      "avg": {
        "field": "executionTime"
      }
    },
    "slow_queries": {
      "top_hits": {
        "sort": [
          {
            "executionTime": {
              "order": "desc"
            }
          }
        ],
        "size": 10
      }
    }
  }
}
```

## Distributed Tracing

### 1. OpenTelemetry Integration

```typescript
// src/tracing/index.ts
import { NodeSDK } from '@opentelemetry/sdk-node';
import { Resource } from '@opentelemetry/resources';
import { SemanticResourceAttributes } from '@opentelemetry/semantic-conventions';
import { JaegerExporter } from '@opentelemetry/exporter-jaeger';
import { BatchSpanProcessor } from '@opentelemetry/sdk-trace-base';
import { registerInstrumentations } from '@opentelemetry/instrumentation';
import { HttpInstrumentation } from '@opentelemetry/instrumentation-http';
import { ExpressInstrumentation } from '@opentelemetry/instrumentation-express';
import { RedisInstrumentation } from '@opentelemetry/instrumentation-redis-4';

const jaegerExporter = new JaegerExporter({
  endpoint: process.env.JAEGER_ENDPOINT || 'http://jaeger-collector:14268/api/traces',
});

const sdk = new NodeSDK({
  resource: new Resource({
    [SemanticResourceAttributes.SERVICE_NAME]: process.env.SERVICE_NAME || 'mobius-api',
    [SemanticResourceAttributes.SERVICE_VERSION]: process.env.APP_VERSION || '1.0.0',
    environment: process.env.NODE_ENV || 'development',
  }),
  spanProcessor: new BatchSpanProcessor(jaegerExporter),
  instrumentations: [
    new HttpInstrumentation({
      requestHook: (span, request) => {
        span.setAttributes({
          'http.request.body': JSON.stringify(request.body),
          'http.user_agent': request.headers['user-agent'],
        });
      },
    }),
    new ExpressInstrumentation(),
    new RedisInstrumentation(),
  ],
});

sdk.start();

// Custom span creation for business logic
import { trace, context, SpanStatusCode } from '@opentelemetry/api';

const tracer = trace.getTracer('mobius-api');

export async function processQuery(query: Query): Promise<QueryResult> {
  const span = tracer.startSpan('process_query', {
    attributes: {
      'query.type': query.type,
      'query.user_id': query.userId,
      'query.text': query.text,
    },
  });

  return context.with(trace.setSpan(context.active(), span), async () => {
    try {
      // Classify query
      const classificationSpan = tracer.startSpan('classify_query');
      const classification = await classifyQuery(query);
      classificationSpan.setAttributes({
        'classification.result': classification.type,
        'classification.confidence': classification.confidence,
      });
      classificationSpan.end();

      // Route to integrations
      const integrationSpan = tracer.startSpan('fetch_integration_data');
      const data = await fetchIntegrationData(classification);
      integrationSpan.setAttributes({
        'integrations.count': data.sources.length,
        'integrations.names': data.sources.join(','),
      });
      integrationSpan.end();

      // Generate response
      const responseSpan = tracer.startSpan('generate_response');
      const result = await generateResponse(data);
      responseSpan.end();

      span.setStatus({ code: SpanStatusCode.OK });
      return result;
    } catch (error) {
      span.recordException(error);
      span.setStatus({
        code: SpanStatusCode.ERROR,
        message: error.message,
      });
      throw error;
    } finally {
      span.end();
    }
  });
}
```

### 2. Trace Analysis Queries

```python
# scripts/trace_analysis.py
from jaeger_client import Config
import requests
import pandas as pd

def analyze_slow_traces(service_name, operation_name, lookback_hours=24):
    """Analyze slow traces to identify bottlenecks"""
    
    jaeger_url = "http://jaeger-query:16686"
    
    # Fetch traces
    params = {
        'service': service_name,
        'operation': operation_name,
        'lookback': f'{lookback_hours}h',
        'limit': 1000
    }
    
    response = requests.get(f"{jaeger_url}/api/traces", params=params)
    traces = response.json()['data']
    
    # Analyze trace durations
    durations = []
    for trace in traces:
        duration = trace['spans'][0]['duration'] / 1000  # Convert to ms
        
        # Find slowest span
        slowest_span = max(trace['spans'], key=lambda s: s['duration'])
        
        durations.append({
            'trace_id': trace['traceID'],
            'total_duration_ms': duration,
            'slowest_operation': slowest_span['operationName'],
            'slowest_duration_ms': slowest_span['duration'] / 1000,
            'timestamp': trace['spans'][0]['startTime']
        })
    
    df = pd.DataFrame(durations)
    
    # Calculate percentiles
    print(f"Duration percentiles for {operation_name}:")
    print(df['total_duration_ms'].describe(percentiles=[0.5, 0.9, 0.95, 0.99]))
    
    # Identify bottlenecks
    bottlenecks = df.groupby('slowest_operation')['slowest_duration_ms'].agg(['count', 'mean', 'max'])
    print("\nBottleneck operations:")
    print(bottlenecks.sort_values('mean', ascending=False).head(10))
    
    return df
```

## Alerting

### 1. Alert Rules

```yaml
# kubernetes/monitoring/alert-rules.yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: mobius-alerts
  namespace: monitoring
spec:
  groups:
  - name: mobius.performance
    interval: 30s
    rules:
    # Response time alerts
    - alert: HighQueryResponseTime
      expr: |
        histogram_quantile(0.95, 
          sum(rate(mobius_query_duration_seconds_bucket[5m])) 
          by (le, query_type)
        ) > 2
      for: 5m
      labels:
        severity: warning
        team: backend
      annotations:
        summary: "High query response time for {{ $labels.query_type }}"
        description: "95th percentile response time is {{ $value }}s (threshold: 2s)"
        runbook_url: https://wiki.mobius-one.com/runbooks/high-response-time
    
    # Error rate alerts
    - alert: HighErrorRate
      expr: |
        (
          sum(rate(mobius_queries_total{status="error"}[5m]))
          /
          sum(rate(mobius_queries_total[5m]))
        ) > 0.05
      for: 5m
      labels:
        severity: critical
        team: backend
      annotations:
        summary: "High error rate in queries"
        description: "Error rate is {{ $value | humanizePercentage }} (threshold: 5%)"
        
    # Integration health alerts
    - alert: IntegrationDown
      expr: mobius_integration_health == 0
      for: 5m
      labels:
        severity: critical
        team: integrations
      annotations:
        summary: "Integration {{ $labels.integration }} is down"
        description: "Integration {{ $labels.integration }} for customer {{ $labels.customer_id }} has been unhealthy for 5 minutes"
    
    # Model performance alerts
    - alert: LowQueryConfidence
      expr: |
        histogram_quantile(0.5, 
          sum(rate(mobius_query_confidence_score_bucket[15m])) 
          by (le, query_type)
        ) < 0.7
      for: 15m
      labels:
        severity: warning
        team: ml
      annotations:
        summary: "Low confidence scores for {{ $labels.query_type }}"
        description: "Median confidence score is {{ $value }} (threshold: 0.7)"
    
    # Cost alerts
    - alert: HighModelCosts
      expr: |
        sum(rate(mobius_model_cost_dollars[1h])) * 3600 * 24 > 1000
      for: 30m
      labels:
        severity: warning
        team: finance
      annotations:
        summary: "High AI model costs detected"
        description: "Projected daily cost is ${{ $value | humanize }} (threshold: $1000)"

  - name: mobius.infrastructure
    interval: 30s
    rules:
    # Pod health
    - alert: PodCrashLooping
      expr: |
        rate(kube_pod_container_status_restarts_total{namespace="mobius-production"}[15m]) > 0
      for: 5m
      labels:
        severity: critical
        team: platform
      annotations:
        summary: "Pod {{ $labels.pod }} is crash looping"
        description: "Pod {{ $labels.pod }} in namespace {{ $labels.namespace }} has restarted {{ $value }} times in the last 15 minutes"
    
    # Resource utilization
    - alert: HighMemoryUsage
      expr: |
        (
          container_memory_working_set_bytes{namespace="mobius-production"}
          / 
          container_spec_memory_limit_bytes{namespace="mobius-production"}
        ) > 0.9
      for: 5m
      labels:
        severity: warning
        team: platform
      annotations:
        summary: "High memory usage in {{ $labels.pod }}"
        description: "Memory usage is {{ $value | humanizePercentage }} of limit"
    
    # Database alerts
    - alert: DatabaseConnectionPoolExhausted
      expr: |
        pg_stat_database_numbackends{datname="mobius_production"} 
        / 
        pg_settings_max_connections > 0.8
      for: 5m
      labels:
        severity: critical
        team: database
      annotations:
        summary: "Database connection pool near exhaustion"
        description: "{{ $value | humanizePercentage }} of max connections are in use"
    
    - alert: DatabaseSlowQueries
      expr: |
        rate(pg_stat_statements_total_time_seconds[5m]) > 10
      for: 5m
      labels:
        severity: warning
        team: database
      annotations:
        summary: "Database experiencing slow queries"
        description: "Query time is averaging {{ $value }}s"

  - name: mobius.business
    interval: 5m
    rules:
    # Business metrics
    - alert: NoQueriesProcessed
      expr: |
        sum(increase(mobius_queries_total[5m])) == 0
      for: 10m
      labels:
        severity: critical
        team: oncall
      annotations:
        summary: "No queries processed in last 10 minutes"
        description: "The system has not processed any queries for 10 minutes"
    
    - alert: CustomerChurn
      expr: |
        (
          mobius_active_customers - mobius_active_customers offset 7d
        ) / mobius_active_customers offset 7d < -0.1
      for: 1h
      labels:
        severity: warning
        team: customer-success
      annotations:
        summary: "Significant customer churn detected"
        description: "Active customers decreased by {{ $value | humanizePercentage }} compared to last week"
```

### 2. Alert Routing

```yaml
# kubernetes/monitoring/alertmanager-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: alertmanager-config
  namespace: monitoring
data:
  alertmanager.yml: |
    global:
      resolve_timeout: 5m
      slack_api_url: 'YOUR_SLACK_WEBHOOK_URL'
      pagerduty_url: 'https://events.pagerduty.com/v2/enqueue'

    route:
      group_by: ['alertname', 'cluster', 'service']
      group_wait: 10s
      group_interval: 10s
      repeat_interval: 12h
      receiver: 'default'
      routes:
      - match:
          severity: critical
        receiver: pagerduty
        continue: true
      - match:
          severity: warning
        receiver: slack-warnings
      - match:
          team: ml
        receiver: slack-ml
      - match:
          team: database
        receiver: slack-database

    receivers:
    - name: 'default'
      slack_configs:
      - channel: '#alerts'
        title: 'Mobius Alert'
        text: '{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'

    - name: 'pagerduty'
      pagerduty_configs:
      - service_key: 'YOUR_PAGERDUTY_SERVICE_KEY'
        description: '{{ .GroupLabels.alertname }}: {{ .CommonAnnotations.summary }}'
        details:
          firing: '{{ template "pagerduty.default.firing" . }}'
          resolved: '{{ template "pagerduty.default.resolved" . }}'

    - name: 'slack-warnings'
      slack_configs:
      - channel: '#alerts-warning'
        color: 'warning'
        title: 'Warning: {{ .GroupLabels.alertname }}'
        text: '{{ .CommonAnnotations.description }}'
        actions:
        - type: button
          text: 'Runbook'
          url: '{{ .CommonAnnotations.runbook_url }}'

    - name: 'slack-ml'
      slack_configs:
      - channel: '#ml-alerts'
        title: 'ML Alert: {{ .GroupLabels.alertname }}'

    - name: 'slack-database'
      slack_configs:
      - channel: '#database-alerts'
        title: 'Database Alert: {{ .GroupLabels.alertname }}'

    inhibit_rules:
    - source_match:
        severity: 'critical'
      target_match:
        severity: 'warning'
      equal: ['alertname', 'dev', 'instance']
```

## Dashboards

### 1. Executive Dashboard

```json
// grafana/dashboards/executive-dashboard.json
{
  "dashboard": {
    "title": "Mobius One Executive Dashboard",
    "panels": [
      {
        "title": "Active Customers (24h)",
        "targets": [{
          "expr": "mobius_active_customers"
        }],
        "type": "stat"
      },
      {
        "title": "Query Volume",
        "targets": [{
          "expr": "sum(rate(mobius_queries_total[5m])) * 60"
        }],
        "type": "graph"
      },
      {
        "title": "Revenue Impact",
        "targets": [{
          "expr": "sum(mobius_revenue_impacted_amount) by (query_type)"
        }],
        "type": "piechart"
      },
      {
        "title": "System Health",
        "targets": [{
          "expr": "1 - (sum(rate(mobius_queries_total{status=\"error\"}[5m])) / sum(rate(mobius_queries_total[5m])))"
        }],
        "type": "gauge",
        "options": {
          "thresholds": {
            "steps": [
              {"value": 0, "color": "red"},
              {"value": 0.95, "color": "yellow"},
              {"value": 0.99, "color": "green"}
            ]
          }
        }
      },
      {
        "title": "AI Model Costs (Daily)",
        "targets": [{
          "expr": "sum(rate(mobius_model_cost_dollars[24h])) * 86400"
        }],
        "type": "stat",
        "options": {
          "unit": "currencyUSD"
        }
      }
    ]
  }
}
```

### 2. Operations Dashboard

```yaml
# grafana/dashboards/operations-dashboard.yaml
panels:
  - title: "Query Latency Heatmap"
    type: heatmap
    query: |
      sum(rate(mobius_query_duration_seconds_bucket[5m])) by (le)
    
  - title: "Top Slow Queries"
    type: table
    query: |
      topk(10, 
        histogram_quantile(0.95,
          sum(rate(mobius_query_duration_seconds_bucket[5m])) by (query_type, le)
        )
      )
      
  - title: "Integration Health Matrix"
    type: table
    query: |
      mobius_integration_health
    format: 
      - {integration: "Integration", customer_id: "Customer", value: "Status"}
      
  - title: "Error Rate by Service"
    type: graph
    queries:
      - expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
          / 
          sum(rate(http_requests_total[5m])) by (service)
        legend: "{{ service }}"
        
  - title: "Resource Utilization"
    type: graph
    queries:
      - expr: |
          100 * (
            container_memory_working_set_bytes{namespace="mobius-production"} 
            / 
            container_spec_memory_limit_bytes
          )
        legend: "Memory: {{ pod }}"
      - expr: |
          rate(container_cpu_usage_seconds_total{namespace="mobius-production"}[5m]) * 100
        legend: "CPU: {{ pod }}"
```

### 3. ML Performance Dashboard

```python
# scripts/generate_ml_dashboard.py
import json

ml_dashboard = {
    "dashboard": {
        "title": "ML Model Performance",
        "panels": [
            {
                "title": "Model Inference Latency",
                "type": "graph",
                "targets": [
                    {
                        "expr": 'histogram_quantile(0.95, sum(rate(mobius_model_inference_duration_seconds_bucket[5m])) by (model, le))',
                        "legendFormat": "p95 {{ model }}"
                    },
                    {
                        "expr": 'histogram_quantile(0.50, sum(rate(mobius_model_inference_duration_seconds_bucket[5m])) by (model, le))',
                        "legendFormat": "p50 {{ model }}"
                    }
                ]
            },
            {
                "title": "Token Usage by Model",
                "type": "bargraph",
                "targets": [{
                    "expr": 'sum(rate(mobius_model_token_usage_sum[1h])) by (model)'
                }]
            },
            {
                "title": "Query Confidence Distribution",
                "type": "heatmap",
                "targets": [{
                    "expr": 'sum(rate(mobius_query_confidence_score_bucket[5m])) by (le)'
                }]
            },
            {
                "title": "Model Error Rate",
                "type": "graph",
                "targets": [{
                    "expr": 'sum(rate(mobius_model_inference_total{status="error"}[5m])) by (model) / sum(rate(mobius_model_inference_total[5m])) by (model)',
                    "legendFormat": "{{ model }}"
                }]
            },
            {
                "title": "Cost per Model Provider",
                "type": "piechart",
                "targets": [{
                    "expr": 'sum(increase(mobius_model_cost_dollars_sum[24h])) by (provider)'
                }]
            }
        ]
    }
}

with open('ml-performance-dashboard.json', 'w') as f:
    json.dump(ml_dashboard, f, indent=2)
```

## Incident Response

### 1. Runbooks

```markdown
# runbooks/high-response-time.md

## High Query Response Time Runbook

### Alert: HighQueryResponseTime

**Severity**: Warning/Critical

### Description
The 95th percentile response time for queries has exceeded the threshold of 2 seconds.

### Impact
- Users experience slow query responses
- Potential timeout errors
- Degraded user experience
- Possible revenue impact

### Diagnosis Steps

1. **Check current response times**
   ```bash
   kubectl exec -n monitoring prometheus-0 -- promtool query instant \
     'histogram_quantile(0.95, sum(rate(mobius_query_duration_seconds_bucket[5m])) by (le, query_type))'
   ```

2. **Identify slow query types**
   ```bash
   kubectl logs -n mobius-production -l app=mobius-api --tail=100 | grep "SLOW_QUERY"
   ```

3. **Check resource utilization**
   ```bash
   kubectl top pods -n mobius-production
   kubectl top nodes
   ```

4. **Review database performance**
   ```sql
   -- Connect to PostgreSQL
   SELECT query, total_time, mean_time, calls
   FROM pg_stat_statements
   WHERE mean_time > 1000
   ORDER BY mean_time DESC
   LIMIT 10;
   ```

5. **Check integration latencies**
   ```bash
   kubectl exec -n monitoring prometheus-0 -- promtool query instant \
     'histogram_quantile(0.95, sum(rate(integration_request_duration_seconds_bucket[5m])) by (le, integration))'
   ```

### Mitigation Steps

1. **Immediate Actions**
   - Scale up affected services:
     ```bash
     kubectl scale deployment mobius-api --replicas=10 -n mobius-production
     ```
   - Enable query result caching if disabled
   - Increase cache TTL temporarily

2. **Database Optimization**
   - Run ANALYZE on slow tables
   - Check for missing indexes
   - Review connection pool settings

3. **Integration Optimization**
   - Enable request batching
   - Increase integration service replicas
   - Check rate limits

4. **Long-term Solutions**
   - Optimize slow queries
   - Implement query complexity limits
   - Add more caching layers
   - Consider database read replicas

### Escalation
- After 30 minutes: Page on-call engineer
- After 1 hour: Escalate to engineering lead
- After 2 hours: Involve CTO

### Related Alerts
- HighErrorRate
- DatabaseSlowQueries
- IntegrationTimeout
```

### 2. Incident Response Automation

```typescript
// src/incident-response/auto-remediation.ts
import { Alert } from './types';
import { KubernetesClient } from './k8s-client';
import { SlackClient } from './slack-client';
import { MetricsClient } from './metrics-client';

export class AutoRemediation {
  constructor(
    private k8s: KubernetesClient,
    private slack: SlackClient,
    private metrics: MetricsClient
  ) {}

  async handleAlert(alert: Alert): Promise<void> {
    switch (alert.labels.alertname) {
      case 'HighQueryResponseTime':
        await this.handleHighResponseTime(alert);
        break;
      case 'HighMemoryUsage':
        await this.handleHighMemory(alert);
        break;
      case 'IntegrationDown':
        await this.handleIntegrationDown(alert);
        break;
      default:
        await this.defaultHandler(alert);
    }
  }

  private async handleHighResponseTime(alert: Alert): Promise<void> {
    const deployment = 'mobius-api';
    const namespace = 'mobius-production';
    
    // Check current replicas
    const currentReplicas = await this.k8s.getReplicas(deployment, namespace);
    
    // Check if we're already at max scale
    if (currentReplicas >= 20) {
      await this.slack.send({
        channel: '#incidents',
        text: `⚠️ High response time alert but already at max scale (${currentReplicas} replicas). Manual intervention required.`,
        severity: 'critical'
      });
      return;
    }
    
    // Auto-scale
    const newReplicas = Math.min(currentReplicas + 5, 20);
    await this.k8s.scale(deployment, namespace, newReplicas);
    
    // Clear cache to force fresh data
    await this.k8s.exec('redis-master-0', 'redis', ['redis-cli', 'FLUSHDB']);
    
    // Notify
    await this.slack.send({
      channel: '#incidents',
      text: `🔧 Auto-scaled ${deployment} from ${currentReplicas} to ${newReplicas} replicas due to high response time`,
      severity: 'warning'
    });
    
    // Schedule scale-down
    setTimeout(async () => {
      const currentMetric = await this.metrics.getResponseTime();
      if (currentMetric < 1.5) {
        await this.k8s.scale(deployment, namespace, currentReplicas);
        await this.slack.send({
          channel: '#incidents',
          text: `✅ Scaled ${deployment} back to ${currentReplicas} replicas. Response time normalized.`
        });
      }
    }, 30 * 60 * 1000); // 30 minutes
  }

  private async handleHighMemory(alert: Alert): Promise<void> {
    const pod = alert.labels.pod;
    const namespace = alert.labels.namespace;
    
    // Get memory usage
    const memoryUsage = await this.k8s.getMemoryUsage(pod, namespace);
    
    // Force garbage collection (if Node.js)
    if (pod.includes('api') || pod.includes('integration')) {
      await this.k8s.exec(pod, namespace, ['kill', '-USR2', '1']);
    }
    
    // If still high after 5 minutes, restart
    setTimeout(async () => {
      const newUsage = await this.k8s.getMemoryUsage(pod, namespace);
      if (newUsage > 0.9) {
        await this.k8s.deletePod(pod, namespace);
        await this.slack.send({
          channel: '#incidents',
          text: `🔄 Restarted pod ${pod} due to persistent high memory usage`,
          severity: 'warning'
        });
      }
    }, 5 * 60 * 1000);
  }

  private async handleIntegrationDown(alert: Alert): Promise<void> {
    const integration = alert.labels.integration;
    const customerId = alert.labels.customer_id;
    
    // Try to refresh connection
    await this.k8s.exec('integration-service-0', 'mobius-production', [
      'curl', '-X', 'POST',
      `http://localhost:3000/internal/refresh-connection/${integration}/${customerId}`
    ]);
    
    // Check if fixed after 1 minute
    setTimeout(async () => {
      const health = await this.metrics.getIntegrationHealth(integration, customerId);
      if (health === 0) {
        // Create support ticket
        await this.createSupportTicket({
          title: `Integration ${integration} down for customer ${customerId}`,
          priority: 'high',
          description: `Automated refresh failed. Manual intervention required.`
        });
      }
    }, 60 * 1000);
  }
}
```

## Performance Monitoring

### 1. Load Testing Integration

```yaml
# k6/load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate } from 'k6/metrics';

const errorRate = new Rate('errors');

export const options = {
  stages: [
    { duration: '5m', target: 100 },   // Ramp up
    { duration: '10m', target: 100 },  // Stay at 100 users
    { duration: '5m', target: 200 },   // Ramp to 200
    { duration: '10m', target: 200 },  // Stay at 200
    { duration: '5m', target: 0 },     // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<2000'], // 95% of requests under 2s
    errors: ['rate<0.05'],             // Error rate under 5%
  },
};

export default function () {
  const queries = [
    'Which customers have overdue invoices?',
    'Show me revenue by product for last quarter',
    'What deals are likely to close this month?',
    'Calculate customer lifetime value',
  ];

  const query = queries[Math.floor(Math.random() * queries.length)];
  
  const response = http.post(
    'https://api.mobius-one.com/v1/query',
    JSON.stringify({ text: query }),
    {
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer ' + __ENV.API_TOKEN,
      },
    }
  );

  const success = check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 2000ms': (r) => r.timings.duration < 2000,
    'has answer': (r) => JSON.parse(r.body).answer !== undefined,
  });

  errorRate.add(!success);
  sleep(Math.random() * 3 + 2); // Random sleep 2-5 seconds
}
```

### 2. Continuous Performance Monitoring

```typescript
// src/monitoring/performance-monitor.ts
export class PerformanceMonitor {
  private metrics: Map<string, PerformanceMetric> = new Map();

  startOperation(name: string): OperationTimer {
    const start = process.hrtime.bigint();
    
    return {
      end: (labels?: Record<string, string>) => {
        const duration = Number(process.hrtime.bigint() - start) / 1e6; // Convert to ms
        
        this.recordMetric(name, duration, labels);
        
        // Log slow operations
        if (duration > 1000) {
          logger.warn('Slow operation detected', {
            operation: name,
            duration,
            labels
          });
        }
      }
    };
  }

  private recordMetric(name: string, duration: number, labels?: Record<string, string>) {
    // Update Prometheus metrics
    if (name.startsWith('query_')) {
      queryDuration.labels(labels?.query_type || 'unknown').observe(duration / 1000);
    }
    
    // Update internal metrics
    if (!this.metrics.has(name)) {
      this.metrics.set(name, {
        count: 0,
        total: 0,
        min: Infinity,
        max: 0,
        p50: [],
        p95: [],
        p99: []
      });
    }
    
    const metric = this.metrics.get(name)!;
    metric.count++;
    metric.total += duration;
    metric.min = Math.min(metric.min, duration);
    metric.max = Math.max(metric.max, duration);
    
    // Keep last 1000 samples for percentiles
    metric.p50.push(duration);
    if (metric.p50.length > 1000) metric.p50.shift();
  }

  getReport(): PerformanceReport {
    const report: PerformanceReport = {};
    
    for (const [name, metric] of this.metrics) {
      const sorted = [...metric.p50].sort((a, b) => a - b);
      
      report[name] = {
        count: metric.count,
        avg: metric.total / metric.count,
        min: metric.min,
        max: metric.max,
        p50: sorted[Math.floor(sorted.length * 0.5)],
        p95: sorted[Math.floor(sorted.length * 0.95)],
        p99: sorted[Math.floor(sorted.length * 0.99)]
      };
    }
    
    return report;
  }
}

// Usage
const perfMon = new PerformanceMonitor();

export async function processQueryWithMonitoring(query: Query): Promise<QueryResult> {
  const timer = perfMon.startOperation('query_processing');
  
  try {
    const classifyTimer = perfMon.startOperation('query_classification');
    const classification = await classifyQuery(query);
    classifyTimer.end({ query_type: classification.type });
    
    const integrationTimer = perfMon.startOperation('integration_fetch');
    const data = await fetchData(classification);
    integrationTimer.end({ integration_count: data.sources.length.toString() });
    
    const responseTimer = perfMon.startOperation('response_generation');
    const result = await generateResponse(data);
    responseTimer.end();
    
    return result;
  } finally {
    timer.end({ query_type: query.type });
  }
}
```

## Best Practices

### 1. Monitoring Checklist
- [ ] All services expose Prometheus metrics
- [ ] Structured logging implemented
- [ ] Distributed tracing enabled
- [ ] Critical business metrics tracked
- [ ] Alerts cover all failure modes
- [ ] Runbooks exist for all alerts
- [ ] Dashboards for all stakeholders
- [ ] Regular load testing
- [ ] Chaos engineering tests
- [ ] Regular review of metrics/alerts

### 2. Metric Naming Conventions
```
<namespace>_<subsystem>_<name>_<unit>

Examples:
- mobius_query_duration_seconds
- mobius_api_requests_total
- mobius_integration_errors_total
- mobius_model_tokens_used
```

### 3. Label Best Practices
- Keep cardinality low (<100 unique values per label)
- Use consistent label names across metrics
- Avoid user IDs or high-cardinality data
- Include: service, environment, status, type
- Avoid: timestamps, request IDs, user data

### 4. Alert Fatigue Prevention
- Only alert on user-impacting issues
- Use appropriate time windows (avoid flapping)
- Group related alerts
- Provide clear actions in runbooks
- Regular alert review and tuning
- Track alert quality metrics

---
*This monitoring guide should be reviewed monthly and updated based on incidents and operational experience.*