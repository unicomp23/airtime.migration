# Rollback Procedures: cantina.com → airtime.com Migration

## Overview
This document provides detailed rollback procedures for each phase of the migration. Each phase has specific rollback strategies designed to minimize downtime and restore service functionality quickly.

## General Rollback Principles

### 🚨 Emergency Contact Information
- **Migration Lead**: [CONTACT INFO]
- **Infrastructure Team**: [CONTACT INFO] 
- **On-Call Engineer**: [CONTACT INFO]
- **Domain Administrator**: [CONTACT INFO]

### 🔄 Rollback Decision Criteria
Execute rollback if any of the following occur:
- Service error rate > 1% above baseline
- Authentication failure rate > 0.5%
- Service discovery failures
- Critical service unavailability > 2 minutes
- Customer-impacting issues reported

---

## Phase 0: Pre-Migration Setup
**Risk Level**: LOW

### Rollback Strategy
- **Time Required**: < 30 minutes
- **Impact**: None (no production changes)

### Rollback Steps
1. **DNS Cleanup**:
   ```bash
   # Remove cantina.com DNS records if needed
   # No impact on existing services
   ```
2. **Certificate Cleanup**: Delete unused SSL certificates
3. **Documentation**: Archive preparation documentation

### Success Criteria
- No lingering cantina.com infrastructure
- No costs from unused resources

---

## Phase 1: DNS Bridge Setup
**Risk Level**: LOW

### Rollback Strategy
- **Time Required**: 5-15 minutes
- **Impact**: None (services remain on airtime.com)

### Rollback Steps
1. **Remove CNAME Records**:
   ```bash
   # DNS Management Console or CLI
   # Remove these CNAME records:
   zookeeper-1.stage.cantina.com
   zookeeper-2.stage.cantina.com
   zookeeper-3.stage.cantina.com
   zookeeper-4.stage.cantina.com
   zookeeper-5.stage.cantina.com
   zookeeper-1.eng.cantina.com
   zookeeper-2.eng.cantina.com
   zookeeper-3.eng.cantina.com
   zookeeper-4.eng.cantina.com
   zookeeper-5.eng.cantina.com
   platform.cantina.com
   merced.cantina.com
   yosemite.cantina.com
   ```

2. **Verify DNS Resolution**:
   ```bash
   # Verify cantina.com no longer resolves
   nslookup platform.cantina.com
   # Should return NXDOMAIN
   ```

### Success Criteria
- cantina.com domains no longer resolve
- airtime.com services unaffected
- No customer impact

---

## Phase 2: Engineering Environment Migration
**Risk Level**: MEDIUM

### Rollback Strategy
- **Time Required**: 10-20 minutes
- **Impact**: Engineering environment only

### Rollback Steps

#### 1. Revert ZooKeeper Configuration
```bash
# File: /workspace.eureka/eureka/vandenberg-config.json
# Revert changes using git
cd /path/to/workspace.eureka
git checkout HEAD~1 eureka/vandenberg-config.json

# Or manual revert:
# Change back to:
# "zookeeper-1.eng.airtime.com:2181"
# "zookeeper-2.eng.airtime.com:2181"
# "zookeeper-3.eng.airtime.com:2181"
# "zookeeper-4.eng.airtime.com:2181" 
# "zookeeper-5.eng.airtime.com:2181"
```

#### 2. Revert Eureka Service
```bash
# Redeploy previous version
kubectl rollout undo deployment/eureka-service -n engineering
# Or using your deployment method
./deploy-eureka.sh --environment=engineering --version=previous
```

#### 3. Revert Token Services
```bash
# File: /jim.tokens/merced/merced/libs/base/constants.mjs
# Revert EXTERNAL_URL back to:
EXTERNAL_URL = 'https://merced.airtime.com:443'

# Redeploy token services
./deploy-merced.sh --environment=engineering --rollback
./deploy-yosemite.sh --environment=engineering --rollback
```

#### 4. Revert Automation Scripts
```bash
# File: /jim.tokens/merced/automation-test.sh
MERCED_HOST="merced-${NODE_NAME}.eng.airtime.com"

# File: /jim.tokens/yosemite/automation-test.sh  
YOSEMITE_HOST="yosemite-${NODE_NAME}.eng.airtime.com"
```

### Validation Steps
```bash
# Verify engineering environment
curl -k https://merced.eng.airtime.com/health
curl -k https://platform.eng.airtime.com/health

# Check service discovery
curl -k https://eureka.eng.airtime.com/status

# Run integration tests
./run-integration-tests.sh --environment=engineering
```

### Success Criteria
- Engineering services accessible via eng.airtime.com
- Service discovery functional
- Integration tests passing
- No impact on staging/production

---

## Phase 3: Staging Environment Migration
**Risk Level**: MEDIUM

### Rollback Strategy
- **Time Required**: 10-20 minutes
- **Impact**: Staging environment only

### Rollback Steps

#### 1. Revert Staging ZooKeeper Configuration
```bash
# File: /workspace.eureka/eureka/config.json
cd /path/to/workspace.eureka
git checkout HEAD~1 eureka/config.json

# Manual revert:
# Change back to:
# "zookeeper-1.stage.airtime.com:2181"
# "zookeeper-2.stage.airtime.com:2181"
# "zookeeper-3.stage.airtime.com:2181"
# "zookeeper-4.stage.airtime.com:2181"
# "zookeeper-5.stage.airtime.com:2181"
```

#### 2. Revert Integration Test Configs
```bash
# File: /workspace.eureka/eureka/server-allocator/integration_tests/configs/cpu_allocator_test/config.json
# Change back to:
# "zookeeper-1.stage.airtime.com:2181"
```

#### 3. Redeploy Staging Services
```bash
# Rollback staging deployments
kubectl rollout undo deployment/eureka-service -n staging
kubectl rollout undo deployment/platform-service -n staging
./deploy-services.sh --environment=staging --rollback
```

### Validation Steps
```bash
# Verify staging services
curl -k https://platform.stage.airtime.com/health
curl -k https://merced.stage.airtime.com/health

# Run staging tests
./run-staging-tests.sh
```

### Success Criteria
- Staging services accessible via stage.airtime.com
- Integration tests passing
- No production impact

---

## Phase 4: Production Platform Services Migration
**Risk Level**: HIGH ⚠️

### 🚨 CRITICAL ROLLBACK PROCEDURE 🚨
- **Time Required**: 2-10 minutes
- **Impact**: Production authentication systems

### Emergency Rollback Steps

#### 1. Immediate Platform Service Rollback
```bash
# EMERGENCY: Revert platform service immediately
kubectl rollout undo deployment/platform-service -n production --to-revision=1

# Or if using blue/green deployment
kubectl patch service platform-service -p '{"spec":{"selector":{"version":"blue"}}}'

# Verify immediately
curl -k https://platform.airtime.com/health
```

#### 2. Token Services Emergency Rollback
```bash
# File: /jim.tokens/merced/merced/libs/base/constants.mjs
# IMMEDIATELY change back to:
EXTERNAL_URL = 'https://merced.airtime.com:443'

# Emergency redeploy (use fastest method)
kubectl set image deployment/merced-service merced=merced:rollback-version
kubectl set image deployment/yosemite-service yosemite=yosemite:rollback-version

# Verify token generation
curl -k https://merced.airtime.com/generate-test-token
```

#### 3. Platform Domain Configuration Rollback
```bash
# File: /jim.tokens/yosemite/yosemite/libs/config/app-config.mjs
# IMMEDIATELY change back to:
DOMAIN_NAME: 'platform.airtime.com:443'

# Hot reload if possible, otherwise redeploy
kubectl rollout restart deployment/yosemite-service
```

### Critical Monitoring During Rollback
```bash
# Monitor these metrics in real-time:
# 1. Authentication success rate
# 2. Token generation success rate  
# 3. Service discovery health
# 4. Overall error rates

# Dashboard URLs (replace with actual):
# https://monitoring.company.com/auth-dashboard
# https://monitoring.company.com/service-health
```

### Success Criteria (Must achieve within 5 minutes)
- Authentication success rate > 99%
- Platform service responding on airtime.com
- Token services generating valid tokens
- Service discovery operational
- Error rates back to baseline

---

## Phase 5: Application Services Migration
**Risk Level**: LOW

### Rollback Strategy
- **Time Required**: 5-15 minutes
- **Impact**: Development/testing tools only

### Rollback Steps
```bash
# Revert integration test files
cd /workspace.eureka/eureka/server-allocator/integration_tests/lib/
git checkout HEAD~1 eureka.js

# Revert development scripts
cd /jim.tokens/yosemite/automation-test/bin/
git checkout HEAD~1 run-automation-local.sh

# Revert documentation
git checkout HEAD~1 README.md
```

---

## Phase 6: DNS Cleanup & Final Transition
**Risk Level**: MEDIUM

### 🚨 EMERGENCY ROLLBACK (Coordinate with Domain Owner)

### Rollback Strategy
- **Time Required**: 30-60 minutes (DNS propagation)
- **Impact**: Potential service interruption

### Emergency Rollback Steps

#### 1. Immediate DNS Restoration
```bash
# EMERGENCY: Re-create CNAME records immediately
# DNS Management Console - ADD THESE BACK:
zookeeper-1.eng.cantina.com    CNAME   zookeeper-1.eng.airtime.com
zookeeper-2.eng.cantina.com    CNAME   zookeeper-2.eng.airtime.com
zookeeper-3.eng.cantina.com    CNAME   zookeeper-3.eng.airtime.com
zookeeper-4.eng.cantina.com    CNAME   zookeeper-4.eng.airtime.com
zookeeper-5.eng.cantina.com    CNAME   zookeeper-5.eng.airtime.com
platform.cantina.com           CNAME   platform.airtime.com
merced.cantina.com             CNAME   merced.airtime.com
yosemite.cantina.com           CNAME   yosemite.airtime.com
```

#### 2. Coordinate with airtime.com Domain Owner
```bash
# Contact airtime.com domain owner IMMEDIATELY
# Request temporary access or DNS pointing
# This may be the most critical step if Phase 6 fails
```

#### 3. Full System Rollback
```bash
# If DNS bridge cannot be restored, full rollback required:
# 1. Rollback all Phase 4 changes (production)
# 2. Rollback all Phase 3 changes (staging)  
# 3. Rollback all Phase 2 changes (engineering)
# 4. Re-establish CNAME bridges
```

### Success Criteria
- All services accessible via some domain
- Authentication working
- Service discovery operational
- Customer access restored

---

## Rollback Communication Plan

### Internal Communications
1. **Immediate**: Slack/Teams emergency channel
2. **Within 5 minutes**: Email to stakeholders
3. **Within 15 minutes**: Incident report started
4. **Within 30 minutes**: Customer communication (if needed)

### Customer Communications
```
Subject: Service Restoration Complete

We experienced a brief service interruption between [TIME] and [TIME] 
due to a planned infrastructure update. All services have been restored 
and are operating normally.

We apologize for any inconvenience.
```

### Post-Rollback Actions
1. **Incident Analysis**: Document what went wrong
2. **Rollback Review**: Analyze rollback effectiveness  
3. **Plan Updates**: Update migration plan based on learnings
4. **Team Debrief**: Schedule team retrospective
5. **Stakeholder Update**: Provide detailed timeline and next steps

---

## Rollback Testing & Validation

### Pre-Migration Rollback Tests
- [ ] Test rollback procedures in staging
- [ ] Verify DNS restoration times
- [ ] Practice emergency communication
- [ ] Validate monitoring alerts work

### Rollback Success Metrics
- **Time to Detection**: < 2 minutes
- **Time to Decision**: < 3 minutes
- **Time to Restore**: < 10 minutes (Phases 1-3), < 5 minutes (Phase 4)
- **Customer Impact**: Minimize to < 15 minutes total

Remember: **It's better to rollback quickly than to debug in production during a failed migration.**