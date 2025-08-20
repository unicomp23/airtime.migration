# Infrastructure as Code - Detailed Migration Guide

## Complete IaC Migration Requirements

This document provides detailed Infrastructure as Code migration steps for all components not yet fully covered.

---

## 🚨 **Critical IaC Components Requiring Migration**

### **1. Kubernetes/EKS Configurations**

#### **Expected Files & Locations**
```yaml
# Kubernetes manifests likely locations:
k8s/
├── namespaces/
│   ├── production.yaml
│   ├── staging.yaml
│   └── engineering.yaml
├── deployments/
│   ├── platform-deployment.yaml
│   ├── merced-deployment.yaml
│   └── yosemite-deployment.yaml
├── services/
│   └── *-service.yaml
├── ingress/
│   └── *-ingress.yaml
└── configmaps/
    └── *-configmap.yaml
```

#### **Migration Tasks**
```bash
# Search for airtime.com in K8s manifests
kubectl get ingress -A -o yaml | grep airtime.com
kubectl get configmap -A -o yaml | grep airtime.com
kubectl get service -A -o yaml | grep airtime.com

# Update Ingress rules
kubectl edit ingress platform-ingress -n production
# Change: host: platform.airtime.com
# To: host: platform.cantina.com

# Update ConfigMaps
kubectl edit configmap app-config -n production
# Update all airtime.com references
```

#### **Helm Charts (if used)**
```yaml
# values.yaml updates needed:
ingress:
  hosts:
    - platform.cantina.com  # was platform.airtime.com
  tls:
    - secretName: cantina-tls
      hosts:
        - platform.cantina.com

config:
  apiUrl: https://api.cantina.com
  authUrl: https://auth.cantina.com
```

---

### **2. AWS CloudFormation Templates**

#### **Expected Templates**
```yaml
# CloudFormation templates requiring updates:
cloudformation/
├── network-stack.yaml     # VPC, subnets, security groups
├── dns-stack.yaml         # Route53 configurations
├── certificate-stack.yaml # ACM certificates
├── alb-stack.yaml        # Application Load Balancers
├── rds-stack.yaml        # Database configurations
└── s3-stack.yaml         # S3 bucket policies
```

#### **Migration Commands**
```bash
# List existing stacks
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE

# Update stack with new parameters
aws cloudformation update-stack \
  --stack-name production-dns \
  --template-body file://dns-stack-cantina.yaml \
  --parameters ParameterKey=DomainName,ParameterValue=cantina.com

# Create change set for review
aws cloudformation create-change-set \
  --stack-name production-alb \
  --change-set-name migrate-to-cantina \
  --template-body file://alb-stack-cantina.yaml
```

---

### **3. Container Orchestration**

#### **Docker Compose Configurations**
```yaml
# docker-compose.yml updates:
services:
  platform:
    environment:
      - API_HOST=platform.cantina.com  # was platform.airtime.com
      - AUTH_URL=https://auth.cantina.com
      - CORS_ORIGINS=https://*.cantina.com
    labels:
      - "traefik.http.routers.platform.rule=Host(`platform.cantina.com`)"
```

#### **ECS Task Definitions**
```json
{
  "family": "platform-service",
  "containerDefinitions": [{
    "environment": [
      {"name": "API_DOMAIN", "value": "api.cantina.com"},
      {"name": "AUTH_DOMAIN", "value": "auth.cantina.com"}
    ]
  }]
}
```

---

### **4. Service Mesh Configurations**

#### **Istio/Consul Service Mesh**
```yaml
# VirtualService updates:
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: platform-vs
spec:
  hosts:
  - platform.cantina.com  # was platform.airtime.com
  gateways:
  - platform-gateway
  
# Consul service definitions:
{
  "service": {
    "name": "platform",
    "tags": ["cantina.com"],
    "meta": {
      "domain": "platform.cantina.com"
    }
  }
}
```

---

### **5. Secrets Management**

#### **AWS Secrets Manager**
```bash
# Update secrets with new domain references
aws secretsmanager update-secret \
  --secret-id production/api/config \
  --secret-string '{"domain":"api.cantina.com","auth":"auth.cantina.com"}'

# Rotate secrets after domain change
aws secretsmanager rotate-secret \
  --secret-id production/ssl/certificates \
  --rotation-lambda-arn arn:aws:lambda:region:account:function:rotate-ssl
```

#### **HashiCorp Vault**
```bash
# Update Vault secrets
vault kv put secret/production/config \
  api_domain=api.cantina.com \
  auth_domain=auth.cantina.com \
  cors_origins="https://*.cantina.com"

# Update PKI certificates
vault write pki_int/issue/cantina-com \
  common_name=platform.cantina.com \
  alt_names="*.cantina.com"
```

---

### **6. CI/CD Pipeline Configurations**

#### **GitHub Actions Workflows**
```yaml
# .github/workflows/deploy.yml
env:
  API_DOMAIN: api.cantina.com  # was api.airtime.com
  DEPLOY_HOST: deploy.cantina.com
  
jobs:
  deploy:
    steps:
      - name: Deploy to Production
        run: |
          ssh deploy@deploy.cantina.com "cd /app && ./deploy.sh"
```

#### **GitLab CI/CD**
```yaml
# .gitlab-ci.yml
variables:
  PRODUCTION_URL: "https://platform.cantina.com"
  STAGING_URL: "https://platform.stage.cantina.com"
  
deploy:production:
  environment:
    name: production
    url: https://platform.cantina.com
```

#### **Jenkins Pipeline (Jenkinsfile)**
```groovy
pipeline {
  environment {
    PROD_URL = 'https://platform.cantina.com'
    API_ENDPOINT = 'https://api.cantina.com'
  }
  stages {
    stage('Deploy') {
      steps {
        sh 'ansible-playbook -i inventory/production deploy.yml --extra-vars "domain=cantina.com"'
      }
    }
  }
}
```

---

### **7. CDN and Edge Configurations**

#### **CloudFront Distribution**
```json
{
  "Origins": [{
    "DomainName": "origin.cantina.com",
    "OriginPath": "",
    "CustomOriginConfig": {
      "OriginProtocolPolicy": "https-only"
    }
  }],
  "Aliases": {
    "Items": ["cdn.cantina.com", "assets.cantina.com"]
  },
  "ViewerCertificate": {
    "ACMCertificateArn": "arn:aws:acm:us-east-1:123456789:certificate/cantina-cert",
    "SSLSupportMethod": "sni-only"
  }
}
```

#### **Cloudflare Workers**
```javascript
// Cloudflare Worker script
addEventListener('fetch', event => {
  const url = new URL(event.request.url);
  // Redirect old domain to new
  if (url.hostname === 'api.airtime.com') {
    url.hostname = 'api.cantina.com';
    return Response.redirect(url.toString(), 301);
  }
});
```

---

### **8. Database Migration Scripts**

#### **Database Connection Strings**
```sql
-- Update application database
UPDATE configurations 
SET value = REPLACE(value, 'airtime.com', 'cantina.com')
WHERE key LIKE '%_url%' OR key LIKE '%_domain%';

-- Update user-facing URLs
UPDATE users 
SET webhook_url = REPLACE(webhook_url, 'airtime.com', 'cantina.com')
WHERE webhook_url LIKE '%airtime.com%';

-- Update stored configurations
UPDATE settings
SET config_json = REPLACE(config_json::text, 'airtime.com', 'cantina.com')::jsonb
WHERE config_json::text LIKE '%airtime.com%';
```

#### **Database Migration Tools**
```bash
# Liquibase changeset
<changeSet id="migrate-to-cantina" author="migration">
  <sql>
    UPDATE system_config 
    SET value = 'cantina.com' 
    WHERE key = 'base_domain';
  </sql>
  <rollback>
    UPDATE system_config 
    SET value = 'airtime.com' 
    WHERE key = 'base_domain';
  </rollback>
</changeSet>

# Flyway migration
-- V2__migrate_to_cantina.sql
UPDATE configurations SET domain = 'cantina.com' WHERE domain = 'airtime.com';
```

---

### **9. Monitoring and Observability**

#### **Prometheus Configuration**
```yaml
# prometheus.yml
global:
  external_labels:
    domain: 'cantina.com'

scrape_configs:
  - job_name: 'platform'
    static_configs:
      - targets: ['platform.cantina.com:9090']
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance
        replacement: 'platform.cantina.com'
```

#### **Grafana Dashboards**
```json
{
  "dashboard": {
    "panels": [{
      "targets": [{
        "expr": "rate(http_requests_total{domain=\"cantina.com\"}[5m])"
      }]
    }],
    "tags": ["cantina", "production"]
  }
}
```

#### **DataDog Configuration**
```yaml
# datadog.yaml
hostname: platform.cantina.com
tags:
  - env:production
  - domain:cantina.com
logs_config:
  container_collect_all: true
  container_exclude:
    name:
      - airtime-legacy
```

---

### **10. Backup and Disaster Recovery**

#### **Backup Scripts**
```bash
#!/bin/bash
# backup.sh updates
BACKUP_DOMAIN="backup.cantina.com"
S3_BUCKET="s3://cantina-backups"
RESTORE_ENDPOINT="https://dr.cantina.com/restore"

# AWS Backup Plans
aws backup update-backup-plan \
  --backup-plan-id $PLAN_ID \
  --backup-plan '{
    "BackupPlanName": "cantina-production",
    "Rules": [{
      "RuleName": "DailyBackup",
      "TargetBackupVaultName": "cantina-vault"
    }]
  }'
```

---

## 📋 **IaC Migration Checklist by Tool**

### **Terraform**
- [ ] Route53 hosted zones
- [ ] ACM certificates
- [ ] ALB/NLB target groups
- [ ] CloudFront distributions
- [ ] S3 bucket policies
- [ ] RDS connection strings
- [ ] ElastiCache endpoints
- [ ] Lambda environment variables
- [ ] API Gateway custom domains

### **Ansible**
- [ ] Inventory host definitions
- [ ] Group variables
- [ ] Host variables
- [ ] Playbook variables
- [ ] Role defaults
- [ ] Template files (.j2)
- [ ] Vault encrypted files

### **Kubernetes**
- [ ] Ingress rules
- [ ] ConfigMaps
- [ ] Secrets
- [ ] Service definitions
- [ ] NetworkPolicies
- [ ] PersistentVolumes

### **Docker**
- [ ] Dockerfile ENV variables
- [ ] docker-compose.yml
- [ ] .env files
- [ ] Container labels
- [ ] Registry URLs

### **CI/CD**
- [ ] Jenkins jobs
- [ ] GitHub Actions
- [ ] GitLab CI
- [ ] CircleCI config
- [ ] Build scripts
- [ ] Deployment targets

---

## 🔄 **Rollback Procedures for IaC**

### **Terraform Rollback**
```bash
# Before making changes
terraform state pull > terraform.state.backup
terraform plan -out=rollback.plan

# To rollback
terraform state push terraform.state.backup
# OR
terraform apply rollback.plan
```

### **Kubernetes Rollback**
```bash
# Before changes
kubectl get all -A -o yaml > k8s-backup.yaml

# To rollback
kubectl rollout undo deployment/platform -n production
kubectl apply -f k8s-backup.yaml
```

### **CloudFormation Rollback**
```bash
# Automatic rollback on failure
aws cloudformation update-stack \
  --stack-name production \
  --on-failure DO_ROLLBACK

# Manual rollback
aws cloudformation cancel-update-stack \
  --stack-name production
```

---

## 🎯 **Success Metrics for IaC Migration**

- **Zero downtime** during infrastructure updates
- **All services accessible** via cantina.com domains
- **Monitoring and alerting** functional with new domains
- **Backup and DR** procedures tested with new infrastructure
- **CI/CD pipelines** deploying to correct endpoints
- **No orphaned resources** referencing airtime.com

---

## 📅 **IaC Migration Timeline**

| Week | Tasks | Risk |
|------|-------|------|
| Week 1 | Audit all IaC repositories | Low |
| Week 2 | Update Terraform configurations | Medium |
| Week 3 | Update Kubernetes manifests | Medium |
| Week 4 | Update CI/CD pipelines | Low |
| Week 5 | Test rollback procedures | High |
| Week 6 | Production cutover | High |

**Total IaC Migration Time**: 6 weeks (can run parallel with application migration)