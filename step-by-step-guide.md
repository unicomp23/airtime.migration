# Step-by-Step Migration Guide: airtime.com → cantina.com

## Quick Reference

| Phase | Duration | Risk | Can Rollback | Impact |
|-------|----------|------|--------------|---------|
| Phase 0 | 1-2 weeks | LOW | ✅ Easy | None |
| Phase 1 | 2-3 days | LOW | ✅ Easy | None |
| Phase 2 | 3-4 days | MEDIUM | ✅ Independent | Engineering only |
| Phase 3 | 3-4 days | MEDIUM | ✅ Independent | Staging only |
| Phase 4 | 2-3 days | HIGH | ⚠️ Complex | Production auth |
| Phase 5 | 1-2 days | LOW | ✅ Easy | Dev tools only |
| Phase 6 | 1 day | MEDIUM | ⚠️ Coordinate | Final cutover |

## Prerequisites

### Team Assignments
- [ ] **Migration Lead**: Overall coordination and decision making
- [ ] **Infrastructure Engineer**: DNS, certificates, monitoring
- [ ] **DevOps Engineer**: Deployment automation and rollbacks
- [ ] **Platform Engineer**: Service configuration changes
- [ ] **QA Engineer**: Testing and validation at each phase

### Required Access
- [ ] DNS management console access
- [ ] SSL certificate authority access
- [ ] Production deployment permissions
- [ ] Monitoring dashboard access
- [ ] Emergency contact information

---

## Phase 0: Pre-Migration Setup (1-2 weeks)

### Week 1: Infrastructure Setup

#### Day 1-2: Infrastructure as Code Audit
```bash
# Clone infrastructure repositories FIRST
git clone git@github.com:airtimemedia/media-ansible.git
git clone git@github.com:airtimemedia/empire-terraform.git
git clone git@github.com:airtimemedia/empire-terraform-sumologic.git
git clone git@github.com:yoinc/devops.git

# Search for airtime.com references in Terraform
cd empire-terraform
grep -r "airtime\.com" . --include="*.tf" --include="*.tfvars"
terraform state list | grep airtime

# Search for airtime.com references in Ansible
cd ../media-ansible
grep -r "airtime\.com" . --include="*.yml" --include="*.yaml"
find inventory/ -type f -exec grep -l "airtime\.com" {} \;
```

#### Day 3-4: Domain and DNS Setup via Terraform
```bash
# Update Terraform configurations for cantina.com
cd empire-terraform

# Create new Route53 zone configuration
cat > dns-cantina.tf <<EOF
resource "aws_route53_zone" "cantina_com" {
  name = "cantina.com"
  tags = {
    Environment = "production"
    Migration   = "airtime-to-cantina"
  }
}
EOF

# Plan and apply DNS changes
terraform plan -out=cantina-dns.plan
terraform apply cantina-dns.plan

# Note the nameservers for domain registrar configuration
terraform output cantina_nameservers
```

#### Day 5-6: SSL Certificate Generation via Terraform/ACM
```bash
# Add ACM certificate request to Terraform
cd empire-terraform

cat > acm-cantina.tf <<EOF
resource "aws_acm_certificate" "cantina_com" {
  domain_name               = "*.cantina.com"
  subject_alternative_names = [
    "cantina.com",
    "*.eng.cantina.com",
    "*.stage.cantina.com"
  ]
  validation_method = "DNS"
  
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_route53_record" "cantina_com_validation" {
  for_each = {
    for dvo in aws_acm_certificate.cantina_com.domain_validation_options : dvo.domain_name => {
      name   = dvo.resource_record_name
      record = dvo.resource_record_value
      type   = dvo.resource_record_type
    }
  }
  
  zone_id = aws_route53_zone.cantina_com.zone_id
  name    = each.value.name
  type    = each.value.type
  records = [each.value.record]
  ttl     = 60
}
EOF

# Apply certificate configuration
terraform plan -out=cantina-acm.plan
terraform apply cantina-acm.plan

# Wait for certificate validation
aws acm wait certificate-validated --certificate-arn $(terraform output cantina_cert_arn)
```

#### Day 5-7: Monitoring Setup
```bash
# Set up dual-domain monitoring
# Add cantina.com endpoints to monitoring system
# Configure alerting for both domains
# Create migration-specific dashboards

# Example monitoring endpoints:
# https://platform.cantina.com/health
# https://merced.cantina.com/health  
# https://yosemite.cantina.com/health
```

### Week 2: Documentation and Testing

#### Day 8-10: Baseline Documentation
```bash
# Document current service metrics
# Record current configuration files
# Create baseline performance measurements

# Example metrics to capture:
# Response times, error rates, throughput
# Service discovery latency
# Authentication success rates
```

#### Day 11-14: Migration Plan Review
- [ ] Review all identified files requiring changes
- [ ] Validate rollback procedures
- [ ] Schedule migration timeline
- [ ] Communicate plan to stakeholders

### Validation Checklist
- [ ] cantina.com domain resolves
- [ ] SSL certificates valid for all subdomains
- [ ] DNS propagation complete (24-48 hours)
- [ ] Monitoring configured for both domains
- [ ] No impact on existing airtime.com services

---

## Phase 1: DNS Bridge Setup (2-3 days)

### Day 1: CNAME Record Creation via Terraform

#### Step 1: Create ZooKeeper CNAMEs in Terraform
```bash
# Update Terraform to add CNAME records
cd empire-terraform

cat > cnames-cantina.tf <<EOF
# ZooKeeper CNAME records for staging
resource "aws_route53_record" "zookeeper_stage_cnames" {
  for_each = toset(["1", "2", "3", "4", "5"])
  
  zone_id = aws_route53_zone.cantina_com.zone_id
  name    = "zookeeper-\${each.value}.stage.cantina.com"
  type    = "CNAME"
  ttl     = 300
  records = ["zookeeper-\${each.value}.stage.airtime.com"]
}

# ZooKeeper CNAME records for engineering
resource "aws_route53_record" "zookeeper_eng_cnames" {
  for_each = toset(["1", "2", "3", "4", "5"])
  
  zone_id = aws_route53_zone.cantina_com.zone_id
  name    = "zookeeper-\${each.value}.eng.cantina.com"
  type    = "CNAME"
  ttl     = 300
  records = ["zookeeper-\${each.value}.eng.airtime.com"]
}
EOF

# Apply CNAME configuration
terraform plan -out=phase1-cnames.plan
terraform apply phase1-cnames.plan
```

#### Step 2: Create Service CNAMEs in Terraform
```bash
# Add service CNAME records

# Staging Environment
zookeeper-1.stage.cantina.com    CNAME    zookeeper-1.stage.airtime.com
zookeeper-2.stage.cantina.com    CNAME    zookeeper-2.stage.airtime.com
zookeeper-3.stage.cantina.com    CNAME    zookeeper-3.stage.airtime.com
zookeeper-4.stage.cantina.com    CNAME    zookeeper-4.stage.airtime.com
zookeeper-5.stage.cantina.com    CNAME    zookeeper-5.stage.airtime.com

# Engineering Environment  
zookeeper-1.eng.cantina.com     CNAME    zookeeper-1.eng.airtime.com
zookeeper-2.eng.cantina.com     CNAME    zookeeper-2.eng.airtime.com
zookeeper-3.eng.cantina.com     CNAME    zookeeper-3.eng.airtime.com
zookeeper-4.eng.cantina.com     CNAME    zookeeper-4.eng.airtime.com
zookeeper-5.eng.cantina.com     CNAME    zookeeper-5.eng.airtime.com
```

#### Step 2: Create Service CNAMEs
```bash
# Platform and Token Services
platform.cantina.com     CNAME    platform.airtime.com
merced.cantina.com        CNAME    merced.airtime.com
yosemite.cantina.com      CNAME    yosemite.airtime.com
```

### Day 2: DNS Propagation and Testing
```bash
# Wait for DNS propagation (4-24 hours)
# Test resolution from multiple locations

# Verify CNAME resolution
nslookup platform.cantina.com
# Should return platform.airtime.com

dig platform.cantina.com CNAME
# Should show CNAME record

# Test from different geographic locations
# Use online DNS checking tools
```

### Day 3: Service Validation
```bash
# Test all services via cantina.com
curl -k https://platform.cantina.com/health
curl -k https://merced.cantina.com/health  
curl -k https://yosemite.cantina.com/health

# Test ZooKeeper connectivity
telnet zookeeper-1.stage.cantina.com 2181
telnet zookeeper-1.eng.cantina.com 2181

# Verify SSL certificates work
openssl s_client -connect platform.cantina.com:443 -servername platform.cantina.com
```

### Validation Checklist
- [ ] All CNAME records created and propagated
- [ ] Services respond identically on both domains
- [ ] SSL certificates valid for cantina.com
- [ ] No performance degradation
- [ ] Both domains work simultaneously

---

## Phase 2: Engineering Environment Migration (3-4 days)

### Day 1: ZooKeeper Configuration Update

#### Step 1: Update Eureka Configuration
```bash
# File: /workspace.eureka/eureka/vandenberg-config.json
# Create backup first
cp vandenberg-config.json vandenberg-config.json.backup

# Edit the file - change:
# FROM:
"hosts": [
  "zookeeper-1.eng.airtime.com:2181",
  "zookeeper-2.eng.airtime.com:2181", 
  "zookeeper-3.eng.airtime.com:2181",
  "zookeeper-4.eng.airtime.com:2181",
  "zookeeper-5.eng.airtime.com:2181"
]

# TO:
"hosts": [
  "zookeeper-1.eng.cantina.com:2181",
  "zookeeper-2.eng.cantina.com:2181",
  "zookeeper-3.eng.cantina.com:2181", 
  "zookeeper-4.eng.cantina.com:2181",
  "zookeeper-5.eng.cantina.com:2181"
]
```

#### Step 2: Deploy Eureka Updates
```bash
# Deploy updated Eureka configuration
# Use your deployment method:
./deploy-eureka.sh --environment=engineering

# Monitor deployment
kubectl get pods -n engineering | grep eureka
kubectl logs -f deployment/eureka-service -n engineering
```

#### Step 3: Validate Service Discovery
```bash
# Test ZooKeeper connectivity
./test-zookeeper-connection.sh --env=engineering

# Verify services can register
curl -k https://eureka.eng.cantina.com/status

# Check service catalog
curl -k https://eureka.eng.cantina.com/eureka/apps
```

### Day 2: Token Services Configuration

#### Step 1: Update Merced Constants
```bash
# File: /jim.tokens/merced/merced/libs/base/constants.mjs
# Create backup
cp constants.mjs constants.mjs.backup

# Edit the file - change:
# FROM:
export const EXTERNAL_URL = 'https://merced.airtime.com:443'

# TO:
export const EXTERNAL_URL = 'https://merced.eng.cantina.com:443'
```

#### Step 2: Update Yosemite Configuration  
```bash
# File: /jim.tokens/yosemite/yosemite/libs/config/app-config.mjs
# Create backup
cp app-config.mjs app-config.mjs.backup

# Edit the file - change:
# FROM:
DOMAIN_NAME: 'platform.airtime.com:443'

# TO: 
DOMAIN_NAME: 'platform.eng.cantina.com:443'
```

#### Step 3: Deploy Token Services
```bash
# Deploy Merced
./deploy-merced.sh --environment=engineering

# Deploy Yosemite  
./deploy-yosemite.sh --environment=engineering

# Monitor deployments
kubectl get pods -n engineering | grep -E "(merced|yosemite)"
```

### Day 3: Automation Scripts Update

#### Step 1: Update Merced Automation
```bash
# File: /jim.tokens/merced/automation-test.sh
# Create backup
cp automation-test.sh automation-test.sh.backup

# Edit the file - change:
# FROM:
MERCED_HOST="merced-${NODE_NAME}.eng.airtime.com"

# TO:
MERCED_HOST="merced-${NODE_NAME}.eng.cantina.com"
```

#### Step 2: Update Yosemite Automation
```bash
# File: /jim.tokens/yosemite/automation-test.sh  
# Create backup
cp automation-test.sh automation-test.sh.backup

# Edit the file - change multiple instances:
# FROM:
YOSEMITE_HOST="yosemite-${NODE_NAME}.eng.airtime.com"
# ... and ...
merced-${NODE_NAME}.eng.airtime.com:443

# TO:
YOSEMITE_HOST="yosemite-${NODE_NAME}.eng.cantina.com"
# ... and ...
merced-${NODE_NAME}.eng.cantina.com:443
```

### Day 4: Testing and Validation

#### Step 1: Integration Tests
```bash
# Run engineering integration tests
cd /jim.tokens/merced
./automation-test.sh

cd /jim.tokens/yosemite  
./automation-test.sh

# Check test results
# All tests should pass
```

#### Step 2: Service Health Checks
```bash
# Verify all services healthy
curl -k https://platform.eng.cantina.com/health
curl -k https://merced.eng.cantina.com/health
curl -k https://yosemite.eng.cantina.com/health

# Test token generation
curl -k -X POST https://merced.eng.cantina.com/generate-token \
  -H "Content-Type: application/json" \
  -d '{"test": true}'

# Validate service discovery
curl -k https://eureka.eng.cantina.com/eureka/apps | grep -i cantina
```

### Validation Checklist
- [ ] ZooKeeper accessible via eng.cantina.com
- [ ] Eureka service discovery working
- [ ] Token services generating valid tokens
- [ ] Automation scripts working
- [ ] Integration tests passing
- [ ] No impact on staging/production

### Rollback (if needed)
```bash
# Quick rollback commands
cp vandenberg-config.json.backup vandenberg-config.json
cp constants.mjs.backup constants.mjs
cp app-config.mjs.backup app-config.mjs
cp automation-test.sh.backup automation-test.sh
./deploy-services.sh --environment=engineering --rollback
```

---

## Phase 3: Staging Environment Migration (3-4 days)

### Day 1: Staging ZooKeeper Update

#### Step 1: Update Staging Configuration
```bash
# File: /workspace.eureka/eureka/config.json
# Create backup
cp config.json config.json.backup

# Edit the file - change:
# FROM:
"hosts": [
  "zookeeper-1.stage.airtime.com:2181",
  "zookeeper-2.stage.airtime.com:2181",
  "zookeeper-3.stage.airtime.com:2181",
  "zookeeper-4.stage.airtime.com:2181", 
  "zookeeper-5.stage.airtime.com:2181"
]

# TO:
"hosts": [
  "zookeeper-1.stage.cantina.com:2181",
  "zookeeper-2.stage.cantina.com:2181",
  "zookeeper-3.stage.cantina.com:2181",
  "zookeeper-4.stage.cantina.com:2181",
  "zookeeper-5.stage.cantina.com:2181"
]
```

#### Step 2: Update Integration Test Config
```bash
# File: /workspace.eureka/eureka/server-allocator/integration_tests/configs/cpu_allocator_test/config.json
# Create backup
cp config.json config.json.backup

# Edit the file - change:
# FROM:
"zookeeper-1.stage.airtime.com:2181"

# TO:
"zookeeper-1.stage.cantina.com:2181"
```

### Day 2: Deploy Staging Updates

#### Step 1: Deploy Eureka to Staging
```bash
# Deploy to staging environment
./deploy-eureka.sh --environment=staging

# Monitor deployment
kubectl get pods -n staging | grep eureka
kubectl logs -f deployment/eureka-service -n staging
```

#### Step 2: Validate Service Discovery
```bash
# Test staging ZooKeeper
./test-zookeeper-connection.sh --env=staging

# Verify service registration
curl -k https://eureka.stage.cantina.com/status
curl -k https://eureka.stage.cantina.com/eureka/apps
```

### Day 3: Staging Service Updates

#### Step 1: Update Platform Domain References
```bash
# Update any staging-specific platform configurations
# This may vary based on your architecture

# Example for staging platform config:
# File: /staging-configs/platform-config.json
# Update domain references to stage.cantina.com
```

#### Step 2: Update Automation Endpoints
```bash
# Update any staging automation scripts
# Update test automation endpoints
# Update CI/CD pipeline configurations for staging
```

### Day 4: Staging Validation

#### Step 1: Run Staging Tests
```bash
# Run comprehensive staging tests
./run-staging-integration-tests.sh

# Run performance tests
./run-staging-performance-tests.sh

# Verify all staging services
curl -k https://platform.stage.cantina.com/health
```

#### Step 2: Load Testing
```bash
# Run load tests to ensure performance
# Monitor staging metrics during testing
# Compare with baseline metrics
```

### Validation Checklist
- [ ] Staging ZooKeeper accessible via cantina.com
- [ ] Service discovery working in staging
- [ ] Integration tests passing
- [ ] Performance within acceptable ranges
- [ ] No production impact
- [ ] Both engineering and staging on cantina.com

---

## Phase 4: Production Platform Services Migration (2-3 days) ⚠️

### ⚠️ CRITICAL PHASE WARNING ⚠️
- **Have rollback scripts ready**
- **Monitor in real-time**
- **Emergency contacts on standby**
- **Peak traffic hours avoided**

### Day 1: Pre-Production Preparation

#### Step 1: Final Testing in Staging
```bash
# Complete final validation in staging
# Run full test suite
# Verify performance under load
# Confirm rollback procedures work
```

#### Step 2: Production Readiness Checklist
- [ ] Rollback scripts tested and ready
- [ ] Monitoring dashboards open
- [ ] Emergency contacts notified
- [ ] Change window scheduled (off-peak)
- [ ] Stakeholders informed

### Day 2: Platform Service Migration

#### Step 1: Update Platform Configuration
```bash
# File: /jim.tokens/yosemite/yosemite/libs/config/app-config.mjs  
# ⚠️ CRITICAL FILE - BACKUP FIRST
cp app-config.mjs app-config.mjs.production.backup

# Edit the file - change:
# FROM:
DOMAIN_NAME: 'platform.airtime.com:443'

# TO:
DOMAIN_NAME: 'platform.cantina.com:443'
```

#### Step 2: Deploy Platform Service
```bash
# Deploy with monitoring
./deploy-platform.sh --environment=production --monitor

# IMMEDIATELY verify deployment
curl -k https://platform.cantina.com/health

# Monitor metrics in real-time
# Watch authentication success rate
# Watch error rates
# Watch response times
```

### Day 3: Token Services Migration

#### Step 1: Update Merced Production
```bash
# File: /jim.tokens/merced/merced/libs/base/constants.mjs
# ⚠️ CRITICAL FILE - BACKUP FIRST  
cp constants.mjs constants.mjs.production.backup

# Edit the file - change:
# FROM:
export const EXTERNAL_URL = 'https://merced.airtime.com:443'

# TO:
export const EXTERNAL_URL = 'https://merced.cantina.com:443'
```

#### Step 2: Deploy Token Services
```bash
# Deploy Merced FIRST
./deploy-merced.sh --environment=production --monitor

# IMMEDIATELY test token generation
curl -k -X POST https://merced.cantina.com/generate-test-token

# Deploy Yosemite
./deploy-yosemite.sh --environment=production --monitor

# IMMEDIATELY test authentication flow
./test-auth-flow.sh --environment=production
```

#### Step 3: Real-Time Validation
```bash
# Monitor these metrics continuously:
# 1. Authentication success rate (must be > 99%)
# 2. Token generation success rate (must be > 99%)
# 3. Service discovery health (must be 100%)
# 4. Overall error rates (must not increase > 0.5%)

# If ANY metric fails, execute immediate rollback
./rollback-production.sh --phase=4
```

### Validation Checklist (Complete within 10 minutes)
- [ ] Platform service responding on cantina.com
- [ ] Token services generating valid tokens
- [ ] Authentication success rate > 99%
- [ ] Service discovery operational
- [ ] Error rates within normal ranges
- [ ] No customer-reported issues

### Emergency Rollback (if needed)
```bash
# EMERGENCY ROLLBACK SCRIPT
#!/bin/bash
echo "EMERGENCY ROLLBACK STARTING"

# Revert platform config
cp app-config.mjs.production.backup app-config.mjs

# Revert token config  
cp constants.mjs.production.backup constants.mjs

# Rollback deployments
kubectl rollout undo deployment/platform-service -n production
kubectl rollout undo deployment/merced-service -n production  
kubectl rollout undo deployment/yosemite-service -n production

echo "ROLLBACK COMPLETE - VALIDATING"
# Validate rollback worked
curl -k https://platform.airtime.com/health
```

---

## Phase 5: Application Services Migration (1-2 days)

### Day 1: Integration Test Updates

#### Step 1: Update Eureka Integration Tests
```bash
# File: /workspace.eureka/eureka/server-allocator/integration_tests/lib/eureka.js
# Create backup
cp eureka.js eureka.js.backup

# Edit file - change line 46:
# FROM:
tecate-legacy://tecate.airtime.com/12345678

# TO:
tecate-legacy://tecate.cantina.com/12345678

# Edit file - change line 24:
# FROM:
vandenberg-i-0cc1319df496e648c-us-west-2.eng.airtime.com

# TO:
vandenberg-i-0cc1319df496e648c-us-west-2.eng.cantina.com
```

#### Step 2: Update Test Configurations
```bash
# File: /jim.tokens/merced/merced/test/test-app-config.mjs
# Update test data:
# FROM:
token-i-12345.us-west-2.eng.airtime.com

# TO:
token-i-12345.us-west-2.eng.cantina.com

# File: /jim.tokens/yosemite/yosemite/test/config/test-app-config.mjs
# Update similar test references
```

### Day 2: Development Scripts and Documentation

#### Step 1: Update Development Scripts
```bash
# File: /jim.tokens/yosemite/automation-test/bin/run-automation-local.sh
# Create backup
cp run-automation-local.sh run-automation-local.sh.backup

# Edit file - change:
# FROM:
merced-i-123456.airtime.com:8259

# TO:
merced-i-123456.cantina.com:8259
```

#### Step 2: Update Documentation
```bash
# File: /jim.tokens/yosemite/yosemite/libs/migrate-db/README.md
# Update documentation references

# File: /jim.tokens/yosemite/yosemite/libs/migrate-db/quick-test.sh
# Change:
# FROM:
https://yosemite.eng.airtime.com

# TO:
https://yosemite.eng.cantina.com
```

#### Step 3: Run Tests
```bash
# Run all updated integration tests
./run-integration-tests.sh --all-environments

# Run unit tests
./run-unit-tests.sh

# Verify development scripts work
./test-development-scripts.sh
```

### Validation Checklist
- [ ] All integration tests passing
- [ ] Unit tests passing
- [ ] Development scripts functional
- [ ] Documentation updated
- [ ] No impact on production services

---

## Phase 6: DNS Cleanup & Final Transition (1 day) ⚠️

### ⚠️ FINAL CUTOVER WARNING ⚠️
- **Point of no return**
- **Coordinate with domain owner**
- **Have emergency contacts ready**

### Hour 1-2: Final Validation

#### Step 1: Complete System Health Check
```bash
# Verify ALL environments healthy
curl -k https://platform.eng.cantina.com/health
curl -k https://platform.stage.cantina.com/health  
curl -k https://platform.cantina.com/health

curl -k https://merced.eng.cantina.com/health
curl -k https://merced.stage.cantina.com/health
curl -k https://merced.cantina.com/health

# Run comprehensive test suite
./run-all-tests.sh --final-validation
```

#### Step 2: Performance Validation
```bash
# Run performance tests
./run-performance-tests.sh --all-environments

# Verify metrics within acceptable ranges
# Response times, error rates, throughput
```

### Hour 3-4: DNS CNAME Removal

#### Step 1: Remove CNAME Records
```bash
# ⚠️ POINT OF NO RETURN ⚠️
# Remove CNAME records from DNS:

# Remove these CNAMEs:
zookeeper-1.stage.cantina.com    (remove CNAME)
zookeeper-2.stage.cantina.com    (remove CNAME)
zookeeper-3.stage.cantina.com    (remove CNAME) 
zookeeper-4.stage.cantina.com    (remove CNAME)
zookeeper-5.stage.cantina.com    (remove CNAME)
zookeeper-1.eng.cantina.com      (remove CNAME)
zookeeper-2.eng.cantina.com      (remove CNAME)
zookeeper-3.eng.cantina.com      (remove CNAME)
zookeeper-4.eng.cantina.com      (remove CNAME)
zookeeper-5.eng.cantina.com      (remove CNAME)
platform.cantina.com             (remove CNAME)
merced.cantina.com                (remove CNAME)
yosemite.cantina.com              (remove CNAME)
```

#### Step 2: Immediate Validation
```bash
# ⚠️ IMMEDIATE VALIDATION REQUIRED
# Test all services still work (should be native now)
curl -k https://platform.cantina.com/health
curl -k https://merced.cantina.com/health
curl -k https://yosemite.cantina.com/health

# Test ZooKeeper connectivity
telnet zookeeper-1.eng.cantina.com 2181
telnet zookeeper-1.stage.cantina.com 2181
```

### Hour 5-6: Final Cleanup

#### Step 1: Scan for Remaining References
```bash
# Search for any remaining airtime.com references
grep -r "airtime.com" /path/to/repositories/
# Should return minimal/no results

# Update any remaining references found
```

#### Step 2: Documentation Update
```bash
# Update all documentation to reflect cantina.com
# Update deployment guides
# Update troubleshooting docs
# Update onboarding materials
```

### Hour 7-8: Migration Completion

#### Step 1: Final System Test
```bash
# Complete end-to-end system test
./run-e2e-tests.sh --production

# Verify all functionality works
# Test user authentication
# Test service-to-service communication
# Test external integrations
```

#### Step 2: Success Declaration
```bash
# If all tests pass:
echo "🎉 MIGRATION COMPLETE 🎉"
echo "All services now native on cantina.com"
echo "No dependencies on airtime.com"

# Send success notification to stakeholders
# Update status pages
# Document final success metrics
```

### Final Validation Checklist
- [ ] All services accessible ONLY via cantina.com
- [ ] No CNAME dependencies remaining
- [ ] All tests passing
- [ ] Authentication working
- [ ] Service discovery functional
- [ ] Performance within normal ranges
- [ ] No customer issues reported
- [ ] Documentation updated

---

## Post-Migration Activities

### Immediate (Day 1)
- [ ] Monitor system health for 24 hours
- [ ] Document any issues encountered
- [ ] Update monitoring baselines
- [ ] Notify stakeholders of successful completion

### Short-term (Week 1)
- [ ] Conduct migration retrospective
- [ ] Update runbooks and procedures
- [ ] Archive migration documentation
- [ ] Plan for airtime.com domain cleanup (if applicable)

### Long-term (Month 1)
- [ ] Review performance metrics
- [ ] Optimize configurations based on learnings
- [ ] Update disaster recovery procedures
- [ ] Plan for future migrations (lessons learned)

---

## Emergency Contacts & Resources

### 🚨 Emergency Contacts
- **Migration Lead**: [NAME] - [PHONE] - [EMAIL]
- **Infrastructure**: [NAME] - [PHONE] - [EMAIL]  
- **DevOps**: [NAME] - [PHONE] - [EMAIL]
- **On-Call Engineer**: [PHONE] - [SLACK]

### 📊 Monitoring Dashboards
- **System Health**: [URL]
- **Authentication Metrics**: [URL]
- **Service Discovery**: [URL]
- **Error Rates**: [URL]

### 📚 Key Resources
- **Rollback Procedures**: `/path/to/rollback-procedures.md`
- **Architecture Diagrams**: `/path/to/phase-diagrams/`
- **Configuration Backups**: `/path/to/config-backups/`
- **Test Scripts**: `/path/to/test-scripts/`

---

## Success! 🎉

If you've completed all phases successfully:

✅ **All services are now running natively on cantina.com**  
✅ **Zero dependencies on airtime.com infrastructure**  
✅ **Authentication, service discovery, and communication functional**  
✅ **Performance maintained or improved**  
✅ **Team knowledge updated and documented**

**Congratulations on a successful zero-downtime migration!**