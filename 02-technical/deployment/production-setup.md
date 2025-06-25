# Mobius One Production Setup Guide

*Last updated: June 24, 2025*

## Overview

This guide covers the complete production deployment of Mobius One on Kubernetes, including infrastructure setup, security configurations, monitoring, and operational procedures.

## Prerequisites

### Required Tools
- `kubectl` >= 1.27
- `helm` >= 3.12
- `terraform` >= 1.5
- AWS CLI or Azure CLI configured
- Docker >= 20.10

### Required Access
- Cloud provider account (AWS/Azure)
- Domain name for application
- SSL certificates (or use cert-manager)
- Container registry access

## Infrastructure Setup

### 1. Cloud Provider Setup

#### AWS Setup
```bash
# Create VPC and networking
cd terraform/environments/production
terraform init
terraform plan -out=plan.tfplan
terraform apply plan.tfplan

# Expected resources:
# - VPC with public/private subnets
# - EKS cluster with 3 node groups
# - RDS PostgreSQL instance
# - ElastiCache Redis cluster
# - S3 buckets for assets
# - ALB for ingress
```

#### Azure Setup
```bash
# Create resource group and networking
cd terraform/environments/production
terraform init
terraform plan -var="cloud_provider=azure" -out=plan.tfplan
terraform apply plan.tfplan

# Expected resources:
# - VNet with subnets
# - AKS cluster with 3 node pools
# - Azure Database for PostgreSQL
# - Azure Cache for Redis
# - Storage accounts
# - Application Gateway
```

### 2. Kubernetes Cluster Setup

```bash
# Connect to cluster
aws eks update-kubeconfig --name mobius-production --region us-east-1
# or
az aks get-credentials --resource-group mobius-prod --name mobius-production

# Verify connection
kubectl get nodes
```

### 3. Install Core Components

```bash
# Install ingress controller
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --values kubernetes/core/ingress-values.yaml

# Install cert-manager for SSL
helm repo add jetstack https://charts.jetstack.io
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set installCRDs=true

# Install monitoring stack
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --values kubernetes/monitoring/prometheus-values.yaml
```

## Application Deployment

### 1. Create Namespaces and Secrets

```bash
# Create application namespace
kubectl create namespace mobius-production

# Create image pull secret
kubectl create secret docker-registry regcred \
  --docker-server=<registry-url> \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email> \
  --namespace=mobius-production

# Create application secrets
kubectl create secret generic mobius-secrets \
  --from-literal=database-url='postgresql://user:pass@host/db' \
  --from-literal=redis-url='redis://redis-host:6379' \
  --from-literal=openai-api-key='sk-...' \
  --from-literal=anthropic-api-key='sk-ant-...' \
  --from-literal=merge-api-key='...' \
  --namespace=mobius-production
```

### 2. Deploy KrakenD API Gateway

```yaml
# kubernetes/deployments/krakend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: krakend
  namespace: mobius-production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: krakend
  template:
    metadata:
      labels:
        app: krakend
    spec:
      containers:
      - name: krakend
        image: devopsfaith/krakend:2.3
        ports:
        - containerPort: 8080
        env:
        - name: FC_ENABLE
          value: "1"
        - name: FC_SETTINGS
          value: "/etc/krakend/settings"
        volumeMounts:
        - name: config
          mountPath: /etc/krakend
        resources:
          requests:
            memory: "256Mi"
            cpu: "500m"
          limits:
            memory: "512Mi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /__health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /__health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
      volumes:
      - name: config
        configMap:
          name: krakend-config
```

### 3. Deploy Core Services

```bash
# Deploy API service
kubectl apply -f kubernetes/deployments/mobius-api-deployment.yaml
kubectl apply -f kubernetes/services/mobius-api-service.yaml

# Deploy AI service
kubectl apply -f kubernetes/deployments/mobius-ai-deployment.yaml
kubectl apply -f kubernetes/services/mobius-ai-service.yaml

# Deploy frontend
kubectl apply -f kubernetes/deployments/mobius-frontend-deployment.yaml
kubectl apply -f kubernetes/services/mobius-frontend-service.yaml

# Deploy integration service
kubectl apply -f kubernetes/deployments/integration-service-deployment.yaml
kubectl apply -f kubernetes/services/integration-service.yaml
```

### 4. Configure Autoscaling

```yaml
# kubernetes/autoscaling/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: mobius-api-hpa
  namespace: mobius-production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: mobius-api
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 100
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 25
        periodSeconds: 60
```

## Security Configuration

### 1. Network Policies

```yaml
# kubernetes/security/network-policies.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-network-policy
  namespace: mobius-production
spec:
  podSelector:
    matchLabels:
      app: mobius-api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: krakend
    ports:
    - protocol: TCP
      port: 3000
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: postgres
    ports:
    - protocol: TCP
      port: 5432
  - to:
    - podSelector:
        matchLabels:
          app: redis
    ports:
    - protocol: TCP
      port: 6379
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 443  # For external APIs
```

### 2. Pod Security Policies

```yaml
# kubernetes/security/pod-security.yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: mobius-api-pdb
  namespace: mobius-production
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: mobius-api
---
apiVersion: v1
kind: SecurityContext
metadata:
  name: secure-pod
spec:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 2000
  seccompProfile:
    type: RuntimeDefault
  capabilities:
    drop:
    - ALL
  readOnlyRootFilesystem: true
```

### 3. RBAC Configuration

```yaml
# kubernetes/security/rbac.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: mobius-production
  name: mobius-api-role
rules:
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: mobius-api-rolebinding
  namespace: mobius-production
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: mobius-api-role
subjects:
- kind: ServiceAccount
  name: mobius-api
  namespace: mobius-production
```

## Database Setup

### 1. PostgreSQL Configuration

```sql
-- Create database and schema
CREATE DATABASE mobius_production;
CREATE SCHEMA IF NOT EXISTS mobius;

-- Create read-only user for analytics
CREATE USER mobius_read WITH PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE mobius_production TO mobius_read;
GRANT USAGE ON SCHEMA mobius TO mobius_read;
GRANT SELECT ON ALL TABLES IN SCHEMA mobius TO mobius_read;

-- Performance tuning
ALTER SYSTEM SET shared_buffers = '4GB';
ALTER SYSTEM SET effective_cache_size = '12GB';
ALTER SYSTEM SET maintenance_work_mem = '1GB';
ALTER SYSTEM SET checkpoint_completion_target = 0.9;
ALTER SYSTEM SET wal_buffers = '16MB';
ALTER SYSTEM SET default_statistics_target = 100;
ALTER SYSTEM SET random_page_cost = 1.1;
```

### 2. Run Migrations

```bash
# From mobius-api directory
npm run migrate:production

# Verify migrations
npm run migrate:status
```

## Monitoring Setup

### 1. Application Metrics

```yaml
# kubernetes/monitoring/service-monitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: mobius-api-metrics
  namespace: mobius-production
spec:
  selector:
    matchLabels:
      app: mobius-api
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
```

### 2. Alerts Configuration

```yaml
# kubernetes/monitoring/alerts.yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: mobius-alerts
  namespace: monitoring
spec:
  groups:
  - name: mobius.rules
    interval: 30s
    rules:
    - alert: HighResponseTime
      expr: http_request_duration_seconds{quantile="0.95"} > 2
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High response time on {{ $labels.instance }}"
        description: "95th percentile response time is above 2 seconds"
    
    - alert: HighErrorRate
      expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "High error rate on {{ $labels.instance }}"
        description: "Error rate is above 5%"
    
    - alert: PodCrashLooping
      expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Pod {{ $labels.pod }} is crash looping"
```

### 3. Logging Configuration

```yaml
# kubernetes/logging/fluent-bit-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
  namespace: logging
data:
  fluent-bit.conf: |
    [SERVICE]
        Flush         5
        Log_Level     info
        Daemon        off

    [INPUT]
        Name              tail
        Path              /var/log/containers/*_mobius-production_*.log
        Parser            docker
        Tag               kube.*
        Refresh_Interval  5

    [FILTER]
        Name                kubernetes
        Match               kube.*
        Kube_URL            https://kubernetes.default.svc:443
        Kube_CA_File        /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        Kube_Token_File     /var/run/secrets/kubernetes.io/serviceaccount/token

    [OUTPUT]
        Name  es
        Match *
        Host  elasticsearch.logging.svc.cluster.local
        Port  9200
        Logstash_Format On
        Logstash_Prefix mobius
        Retry_Limit False
```

## Load Balancing and SSL

### 1. Ingress Configuration

```yaml
# kubernetes/ingress/production-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mobius-ingress
  namespace: mobius-production
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
spec:
  tls:
  - hosts:
    - api.mobius-one.com
    - app.mobius-one.com
    secretName: mobius-tls
  rules:
  - host: api.mobius-one.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: krakend
            port:
              number: 8080
  - host: app.mobius-one.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: mobius-frontend
            port:
              number: 80
```

### 2. SSL Certificate Setup

```yaml
# kubernetes/certificates/cluster-issuer.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: devops@mobius-one.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
```

## Backup and Disaster Recovery

### 1. Database Backups

```bash
# Automated backup CronJob
kubectl apply -f - <<EOF
apiVersion: batch/v1
kind: CronJob
metadata:
  name: postgres-backup
  namespace: mobius-production
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: postgres-backup
            image: postgres:15
            env:
            - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  name: mobius-secrets
                  key: database-password
            command:
            - /bin/bash
            - -c
            - |
              DATE=$(date +%Y%m%d_%H%M%S)
              pg_dump -h $DB_HOST -U $DB_USER -d $DB_NAME | gzip > /backup/mobius_$DATE.sql.gz
              aws s3 cp /backup/mobius_$DATE.sql.gz s3://mobius-backups/postgres/
              # Keep only last 30 days
              find /backup -name "*.sql.gz" -mtime +30 -delete
            volumeMounts:
            - name: backup
              mountPath: /backup
          volumes:
          - name: backup
            persistentVolumeClaim:
              claimName: backup-pvc
          restartPolicy: OnFailure
EOF
```

### 2. Disaster Recovery Plan

```bash
# Create DR namespace
kubectl create namespace mobius-dr

# Replicate secrets
kubectl get secret mobius-secrets -n mobius-production -o yaml | \
  sed 's/namespace: mobius-production/namespace: mobius-dr/' | \
  kubectl apply -f -

# Database restoration procedure
kubectl run -it --rm postgres-restore \
  --image=postgres:15 \
  --env="PGPASSWORD=$DB_PASSWORD" \
  --namespace=mobius-dr \
  -- bash -c "
    aws s3 cp s3://mobius-backups/postgres/mobius_20250624_020000.sql.gz - | \
    gunzip | \
    psql -h $DB_HOST -U $DB_USER -d $DB_NAME
  "
```

## Health Checks and Monitoring

### 1. Synthetic Monitoring

```yaml
# kubernetes/monitoring/synthetic-check.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: api-health-check
  namespace: monitoring
spec:
  schedule: "*/5 * * * *"  # Every 5 minutes
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: health-check
            image: curlimages/curl:latest
            command:
            - /bin/sh
            - -c
            - |
              response=$(curl -s -o /dev/null -w "%{http_code}" https://api.mobius-one.com/health)
              if [ $response -eq 200 ]; then
                echo "API is healthy"
              else
                echo "API health check failed with status $response"
                curl -X POST $SLACK_WEBHOOK -d '{"text":"API health check failed!"}'
              fi
          restartPolicy: OnFailure
```

### 2. Dashboard Access

```bash
# Port-forward to access Grafana
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80

# Access dashboards:
# - Kubernetes cluster overview
# - Application metrics (custom dashboards)
# - Database performance
# - API gateway statistics
```

## Rollout Procedures

### 1. Blue-Green Deployment

```bash
# Deploy new version to green environment
kubectl apply -f kubernetes/deployments/mobius-api-green.yaml

# Verify green deployment
kubectl get pods -l app=mobius-api,version=green

# Switch traffic to green
kubectl patch service mobius-api -p '{"spec":{"selector":{"version":"green"}}}'

# Monitor for issues (5 minutes)
# If issues, rollback:
kubectl patch service mobius-api -p '{"spec":{"selector":{"version":"blue"}}}'

# If successful, update blue deployment
kubectl apply -f kubernetes/deployments/mobius-api-blue.yaml
```

### 2. Canary Deployment

```yaml
# kubernetes/deployments/canary-deployment.yaml
apiVersion: v1
kind: Service
metadata:
  name: mobius-api-canary
spec:
  selector:
    app: mobius-api
    version: canary
  ports:
  - port: 3000
---
# Update ingress to split traffic
# 95% to stable, 5% to canary
```

## Maintenance Procedures

### 1. Cluster Upgrades

```bash
# Check available updates
az aks get-upgrades --resource-group mobius-prod --name mobius-production

# Upgrade control plane
az aks upgrade --resource-group mobius-prod --name mobius-production --kubernetes-version 1.28.0

# Upgrade node pools (rolling update)
az aks nodepool upgrade \
  --resource-group mobius-prod \
  --cluster-name mobius-production \
  --name nodepool1 \
  --kubernetes-version 1.28.0
```

### 2. Certificate Renewal

```bash
# Check certificate expiration
kubectl get certificate -n mobius-production

# Force renewal if needed
kubectl delete certificate mobius-tls -n mobius-production
# cert-manager will automatically recreate
```

## Troubleshooting

### Common Issues

1. **High Response Times**
   ```bash
   # Check pod resources
   kubectl top pods -n mobius-production
   
   # Check for throttling
   kubectl describe pod <pod-name> -n mobius-production
   
   # Scale if needed
   kubectl scale deployment mobius-api --replicas=10 -n mobius-production
   ```

2. **Database Connection Issues**
   ```bash
   # Check connection pool
   kubectl logs -n mobius-production deployment/mobius-api | grep "connection"
   
   # Verify secrets
   kubectl get secret mobius-secrets -n mobius-production -o yaml
   ```

3. **Memory Leaks**
   ```bash
   # Get heap dump
   kubectl exec -n mobius-production <pod-name> -- kill -USR2 1
   
   # Download heap dump
   kubectl cp mobius-production/<pod-name>:/tmp/heapdump.hprof ./heapdump.hprof
   ```

## Production Checklist

- [ ] All services deployed and healthy
- [ ] SSL certificates configured and valid
- [ ] Monitoring and alerts configured
- [ ] Backups scheduled and tested
- [ ] Network policies applied
- [ ] Resource limits set
- [ ] Autoscaling configured
- [ ] Load testing completed
- [ ] Disaster recovery tested
- [ ] Documentation updated
- [ ] Team trained on procedures

## Support Contacts

- **On-Call**: Use PagerDuty rotation
- **Escalation**: CTO, VP Engineering
- **Cloud Support**: AWS/Azure support tickets
- **Security Issues**: security@mobius-one.com

---
*This document should be reviewed and updated with each major deployment or infrastructure change.*