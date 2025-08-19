# Incremental Migration Plan: airtime.com → cantina.com

## Overview
This document provides a safe, incremental migration strategy with zero-downtime rollout phases. Each step includes dependency analysis, rollback procedures, and validation checkpoints.

## Migration Phases

### Phase 0: Pre-Migration Setup (1-2 weeks)
**Goal**: Prepare infrastructure without impacting existing services

#### Tasks:
- [ ] Register cantina.com domain
- [ ] Set up DNS zones and nameservers
- [ ] Generate SSL certificates for all subdomains
- [ ] Create monitoring for both domains
- [ ] Document current service baseline metrics

#### Validation:
- [ ] DNS resolution working for cantina.com
- [ ] SSL certificates valid
- [ ] No impact on existing airtime.com services

#### Rollback:
- Low risk - no changes to existing services

---

### Phase 1: DNS Bridge Setup (2-3 days)
**Goal**: Create dual-domain accessibility using DNS CNAMEs

#### Strategy:
Use DNS CNAME records to make services accessible via both domains simultaneously.

#### Tasks:
- [ ] Create CNAME records:
  - `zookeeper-1.stage.cantina.com` → `zookeeper-1.stage.airtime.com`
  - `zookeeper-2.stage.cantina.com` → `zookeeper-2.stage.airtime.com`
  - `zookeeper-{3,4,5}.stage.cantina.com` → corresponding airtime.com hosts
  - Repeat for `eng.cantina.com` subdomains
  - `platform.cantina.com` → `platform.airtime.com`
  - `merced.cantina.com` → `merced.airtime.com`
  - `yosemite.cantina.com` → `yosemite.airtime.com`

#### Validation:
- [ ] All services respond on both domains
- [ ] SSL certificates work for cantina.com endpoints
- [ ] Service discovery functional via both domains
- [ ] Token generation works with both domains

#### Rollback:
- [ ] Remove CNAME records
- [ ] Services continue on airtime.com unchanged

---

### Phase 2: Engineering Environment Migration (3-4 days)
**Goal**: Migrate engineering environment to use cantina.com natively

#### Step 2a: ZooKeeper Configuration Update
**Files to Update:**
- [ ] `/workspace.eureka/eureka/vandenberg-config.json`
  ```json
  // Change from:
  "zookeeper-{1,2,3,4,5}.eng.airtime.com:2181"
  // Change to:
  "zookeeper-{1,2,3,4,5}.eng.cantina.com:2181"
  ```

#### Step 2b: Eureka Service Discovery
- [ ] Update Eureka configurations to use cantina.com ZooKeeper endpoints
- [ ] Deploy updated Eureka service
- [ ] Validate service registration working

#### Step 2c: Token Services - Engineering
- [ ] Update Merced constants:
  ```javascript
  // /jim.tokens/merced/merced/libs/base/constants.mjs
  EXTERNAL_URL = 'https://merced.eng.cantina.com:443'
  ```
- [ ] Update automation scripts:
  ```bash
  # /jim.tokens/merced/automation-test.sh
  MERCED_HOST="merced-${NODE_NAME}.eng.cantina.com"
  ```
- [ ] Update Yosemite configurations similarly

#### Validation:
- [ ] Engineering services accessible via cantina.com
- [ ] Service discovery working
- [ ] Authentication flows functional
- [ ] Integration tests passing
- [ ] No impact on staging/production

#### Rollback:
- [ ] Revert configuration files to airtime.com
- [ ] Redeploy previous versions
- [ ] Engineering falls back to CNAME resolution

---

### Phase 3: Staging Environment Migration (3-4 days)
**Goal**: Migrate staging environment following engineering success

#### Step 3a: Staging ZooKeeper Update
- [ ] `/workspace.eureka/eureka/config.json`
  ```json
  "zookeeper-{1,2,3,4,5}.stage.cantina.com:2181"
  ```
- [ ] `/workspace.eureka/eureka/server-allocator/integration_tests/configs/cpu_allocator_test/config.json`

#### Step 3b: Staging Services Update
- [ ] Update platform domain references
- [ ] Update automation testing endpoints
- [ ] Deploy to staging environment

#### Validation:
- [ ] Staging environment fully functional on cantina.com
- [ ] Integration tests pass
- [ ] Performance metrics within normal ranges
- [ ] Cross-service communication working

#### Rollback:
- [ ] Revert staging configurations
- [ ] Staging falls back to CNAME resolution
- [ ] No production impact

---

### Phase 4: Production Platform Services (2-3 days)
**Goal**: Migrate core platform infrastructure

#### Step 4a: Platform Service Domain Update
- [ ] Update platform.cantina.com endpoint configurations
- [ ] Deploy platform service updates
- [ ] Validate authentication gateway working

#### Step 4b: Primary Token Services
- [ ] Deploy Merced with cantina.com configuration
- [ ] Deploy Yosemite with cantina.com configuration
- [ ] Validate token generation and validation

#### Validation:
- [ ] Production platform accessible via cantina.com
- [ ] Authentication flows operational
- [ ] Token services issuing valid tokens
- [ ] All dependent services connecting successfully

#### Progressive Rollback:
- [ ] Individual service rollback capability
- [ ] Platform service can revert independently
- [ ] Token services can revert independently

---

### Phase 5: Application Services Migration (1-2 days)
**Goal**: Migrate remaining application services and automation

#### Tasks:
- [ ] Update Tecate service references
- [ ] Update integration test configurations
- [ ] Update development automation scripts
- [ ] Update documentation references

#### Validation:
- [ ] All services operational on cantina.com
- [ ] Automation pipelines functional
- [ ] Development workflows working

#### Rollback:
- [ ] Service-by-service rollback
- [ ] Non-critical services can roll back independently

---

### Phase 6: DNS Cutover & Cleanup (1 day)
**Goal**: Remove CNAME bridges and make cantina.com native

#### Step 6a: DNS Transition
- [ ] Update DNS to point airtime.com CNAMEs to cantina.com (temporary)
- [ ] Monitor for any legacy references
- [ ] Validate all traffic routing correctly

#### Step 6b: CNAME Cleanup
- [ ] Remove cantina.com → airtime.com CNAME records
- [ ] Update any remaining hardcoded airtime.com references
- [ ] Final validation of all services

#### Validation:
- [ ] All services native on cantina.com
- [ ] No dependencies on airtime.com infrastructure
- [ ] Performance and reliability maintained

#### Final Rollback (Emergency Only):
- [ ] Requires coordination with airtime.com domain owner
- [ ] May require temporary service interruption

---

## Risk Mitigation Strategies

### During Each Phase:
1. **Monitoring**: Continuous monitoring of error rates, response times
2. **Canary Deployment**: Test changes in isolation before full rollout  
3. **Communication**: Real-time status updates to stakeholders
4. **Quick Rollback**: Keep previous configurations readily available

### Phase Dependencies:
- Each phase builds on the previous one
- Later phases can be rolled back independently
- Early phases (DNS, ZooKeeper) require careful coordination

### Success Metrics:
- **Error Rate**: < 0.1% increase during migration
- **Response Time**: < 10% increase during migration  
- **Availability**: > 99.9% uptime maintained
- **Service Discovery**: 100% service registration success

## Timeline Summary

| Phase | Duration | Parallel Possible | Risk Level |
|-------|----------|-------------------|------------|
| Phase 0 | 1-2 weeks | Yes | Low |
| Phase 1 | 2-3 days | Yes | Low |
| Phase 2 | 3-4 days | No | Medium |
| Phase 3 | 3-4 days | No | Medium |
| Phase 4 | 2-3 days | Partial | High |
| Phase 5 | 1-2 days | Yes | Low |
| Phase 6 | 1 day | No | Medium |

**Total Duration**: 3-5 weeks with proper validation at each step.