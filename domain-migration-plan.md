# airtime.com → cantina.com Migration Plan

## Executive Summary

This document outlines the complete migration strategy from airtime.com to cantina.com. The domain airtime.com has been sold, requiring immediate migration of all services, infrastructure, and code references.

## Critical Infrastructure Dependencies

### ZooKeeper Clusters (PRIORITY: CRITICAL)
- **Stage Environment**: `zookeeper-{1,2,3,4,5}.stage.airtime.com:2181`
- **Engineering Environment**: `zookeeper-{1,2,3,4,5}.eng.airtime.com:2181`
- **Action Required**: DNS migration or infrastructure reconfiguration
- **Downtime Risk**: HIGH - Service discovery failure

### Core Services (PRIORITY: CRITICAL)
- **Merced Token Service**: `https://merced.airtime.com:443`
- **Platform Endpoint**: `platform.airtime.com:443`
- **Impact**: Authentication and core platform access

## Migration Strategy

### Phase 1: Infrastructure Preparation (Pre-Migration)
1. **DNS Setup**
   - Configure cantina.com DNS zones
   - Set up subdomains: eng.cantina.com, stage.cantina.com, platform.cantina.com
   - Prepare SSL certificates for new domain

2. **ZooKeeper Migration**
   - Option A: DNS CNAME from airtime.com to cantina.com (temporary)
   - Option B: Update ZooKeeper cluster with new hostnames
   - Test service discovery with new endpoints

### Phase 2: Code Updates (Pre-Deployment)
1. **Configuration Files** (22 files identified)
   - Update all hardcoded airtime.com references
   - Modify automation scripts for deployment endpoints
   - Update test configurations

2. **Token Service Updates**
   - Update JWT token generation with new domain
   - Ensure backwards compatibility during transition

### Phase 3: Deployment & Cutover
1. **Staged Rollout**
   - Deploy to engineering environment first
   - Validate all services with new domain
   - Deploy to staging environment
   - Full production cutover

2. **Monitoring & Validation**
   - Monitor service health during migration
   - Validate authentication flows
   - Check inter-service communication

### Phase 4: Cleanup (Post-Migration)
1. **Remove Dependencies**
   - Remove temporary DNS CNAMEs
   - Clean up old automation scripts
   - Update documentation references

## Risk Assessment

### HIGH RISK
- **Service Discovery Failure**: ZooKeeper endpoints must be accessible
- **Authentication Disruption**: Token services must maintain continuity
- **Deployment Pipeline Failure**: Automation scripts must work with new endpoints

### MEDIUM RISK
- **Integration Test Failures**: Test environments may need reconfiguration
- **Third-party Service Disruption**: External integrations may need updates

### LOW RISK
- **Documentation Inconsistencies**: Can be updated post-migration
- **Development Environment Issues**: Local development configurations

## Rollback Plan

### Immediate Rollback (< 1 hour)
- Revert DNS changes to point back to airtime.com
- Rollback application deployments to previous version
- Restore ZooKeeper configurations

### Extended Rollback (1-24 hours)
- Coordinate with domain purchaser for temporary access
- Implement emergency DNS overrides
- Deploy hotfixes with airtime.com references

## Success Criteria

1. ✅ All services accessible via cantina.com domains
2. ✅ Authentication flows working correctly
3. ✅ No broken inter-service communication
4. ✅ Deployment automation functioning
5. ✅ Monitoring and alerting operational

## Timeline Estimate

- **Phase 1**: 1-2 weeks (DNS setup, certificate generation)
- **Phase 2**: 1 week (code updates, testing)
- **Phase 3**: 2-3 days (deployment, validation)
- **Phase 4**: 1 week (cleanup, documentation)

**Total Estimated Time**: 4-6 weeks

## Next Steps

1. **Immediate**: Begin DNS setup for cantina.com
2. **Week 1**: Complete infrastructure preparation
3. **Week 2-3**: Code updates and testing
4. **Week 4**: Migration execution
5. **Week 5-6**: Cleanup and optimization

## Dependencies & Blockers

- DNS control for cantina.com domain
- SSL certificate authority access
- Coordination with infrastructure team
- Testing environment availability
- Business approval for migration timeline