# airtime.com → cantina.com Migration Checklist

## Pre-Migration Setup

### DNS & Infrastructure
- [ ] Purchase/configure cantina.com domain
- [ ] Set up DNS zones for cantina.com
- [ ] Create subdomains:
  - [ ] eng.cantina.com
  - [ ] stage.cantina.com  
  - [ ] platform.cantina.com
  - [ ] merced.cantina.com
  - [ ] yosemite.cantina.com
- [ ] Generate SSL certificates for all subdomains
- [ ] Plan ZooKeeper migration strategy
- [ ] Set up temporary DNS CNAMEs (if needed)

## Critical Files - MUST UPDATE

### ZooKeeper Configuration
- [ ] `/root/repo/dev/workspace.eureka/eureka/config.json`
  - Replace: `zookeeper-{1,2,3,4,5}.stage.airtime.com:2181`
  - With: `zookeeper-{1,2,3,4,5}.stage.cantina.com:2181`
- [ ] `/root/repo/dev/workspace.eureka/eureka/vandenberg-config.json`  
  - Replace: `zookeeper-{1,2,3,4,5}.eng.airtime.com:2181`
  - With: `zookeeper-{1,2,3,4,5}.eng.cantina.com:2181`

### Token Service Constants
- [ ] `/root/repo/dev/jim.tokens/merced/merced/libs/base/constants.mjs`
  - Replace: `EXTERNAL_URL = 'https://merced.airtime.com:443'`
  - With: `EXTERNAL_URL = 'https://merced.cantina.com:443'`

### Default Configuration
- [ ] `/root/repo/dev/jim.tokens/yosemite/yosemite/libs/config/app-config.mjs`
  - Replace: `DOMAIN_NAME: 'platform.airtime.com:443'`
  - With: `DOMAIN_NAME: 'platform.cantina.com:443'`

### Automation Scripts
- [ ] `/root/repo/dev/jim.tokens/merced/automation-test.sh`
  - Replace: `MERCED_HOST="merced-${NODE_NAME}.eng.airtime.com"`
  - With: `MERCED_HOST="merced-${NODE_NAME}.eng.cantina.com"`
- [ ] `/root/repo/dev/jim.tokens/yosemite/automation-test.sh`
  - Replace: `YOSEMITE_HOST="yosemite-${NODE_NAME}.eng.airtime.com"`
  - With: `YOSEMITE_HOST="yosemite-${NODE_NAME}.eng.cantina.com"`
  - Replace: `merced-${NODE_NAME}.eng.airtime.com:443`
  - With: `merced-${NODE_NAME}.eng.cantina.com:443`

## Test Configuration Files

### Integration Test Config
- [ ] `/root/repo/dev/workspace.eureka/eureka/server-allocator/integration_tests/configs/cpu_allocator_test/config.json`
  - Replace: `zookeeper-1.stage.airtime.com:2181`
  - With: `zookeeper-1.stage.cantina.com:2181`

### Test Files
- [ ] `/root/repo/dev/jim.tokens/merced/merced/test/test-app-config.mjs`
  - Replace: `token-i-12345.us-west-2.eng.airtime.com`
  - With: `token-i-12345.us-west-2.eng.cantina.com`
- [ ] `/root/repo/dev/jim.tokens/yosemite/yosemite/test/config/test-app-config.mjs`
  - Replace airtime.com references with cantina.com
- [ ] `/root/repo/dev/workspace.eureka/eureka/server-allocator/integration_tests/lib/eureka.js`
  - Replace: `tecate-legacy://tecate.airtime.com/12345678`
  - With: `tecate-legacy://tecate.cantina.com/12345678`
  - Replace: `vandenberg-i-0cc1319df496e648c-us-west-2.eng.airtime.com`
  - With: `vandenberg-i-0cc1319df496e648c-us-west-2.eng.cantina.com`

### Development Scripts
- [ ] `/root/repo/dev/jim.tokens/yosemite/automation-test/bin/run-automation-local.sh`
  - Replace: `merced-i-123456.airtime.com:8259`
  - With: `merced-i-123456.cantina.com:8259`

## Documentation Updates

- [ ] `/root/repo/dev/jim.tokens/yosemite/yosemite/libs/migrate-db/README.md`
  - Update airtime.com references to cantina.com
- [ ] `/root/repo/dev/jim.tokens/yosemite/yosemite/libs/migrate-db/quick-test.sh`
  - Replace: `https://yosemite.eng.airtime.com`
  - With: `https://yosemite.eng.cantina.com`

## Integration Test Files

### Eureka Integration Tests
- [ ] `/root/repo/dev/workspace.eureka/eureka/server-allocator/integration_tests/tests/all-allocator-test.js`
- [ ] `/root/repo/dev/workspace.eureka/eureka/server-allocator/integration_tests/tests/developer-pages-test.js`
- [ ] `/root/repo/dev/workspace.eureka/eureka/server-allocator/integration_tests/tests/servers-endpoint-test.js`
- [ ] `/root/repo/dev/workspace.eureka/eureka/server-allocator/tests/test-server-allocator.js`

## Deployment & Testing

### Pre-Deployment Testing
- [ ] Run all unit tests with updated configurations
- [ ] Test ZooKeeper connectivity with new endpoints
- [ ] Validate token generation with new domain
- [ ] Test automation scripts in staging environment

### Staging Deployment
- [ ] Deploy to engineering environment first
- [ ] Run integration tests
- [ ] Validate service discovery
- [ ] Test authentication flows
- [ ] Monitor service health

### Production Deployment
- [ ] Deploy to staging environment
- [ ] Full production deployment
- [ ] Monitor all services
- [ ] Validate external integrations
- [ ] Check SSL certificate validity

## Post-Migration Verification

### Service Health Checks
- [ ] All services responding on cantina.com domains
- [ ] ZooKeeper clusters healthy
- [ ] Token service issuing valid tokens
- [ ] Authentication flows working
- [ ] Inter-service communication functional

### Integration Validation
- [ ] Third-party services connecting properly
- [ ] Monitoring and alerting operational
- [ ] Log aggregation working
- [ ] Backup systems functional

### Performance Validation
- [ ] Response times within acceptable ranges
- [ ] No increase in error rates
- [ ] Database connections stable
- [ ] Cache systems operational

## Cleanup Tasks

### Remove Temporary Configurations
- [ ] Remove DNS CNAMEs (if used)
- [ ] Clean up old automation scripts
- [ ] Remove temporary SSL certificates
- [ ] Update monitoring configurations

### Documentation Updates
- [ ] Update runbooks with new domains
- [ ] Update deployment guides
- [ ] Update troubleshooting docs
- [ ] Update onboarding materials

## Rollback Procedures

### Emergency Rollback (< 1 hour)
- [ ] Rollback plan documented and tested
- [ ] DNS rollback procedures ready
- [ ] Application rollback scripts prepared
- [ ] Team contact information updated

### Communication Plan
- [ ] Stakeholder notification list
- [ ] Status page updates
- [ ] Customer communication plan
- [ ] Internal team notifications

## Sign-off

- [ ] Infrastructure team approval
- [ ] Security team review
- [ ] Product team approval  
- [ ] Executive sign-off
- [ ] Migration date confirmed

---

**Migration Lead**: _________________ **Date**: _________

**Infrastructure Lead**: _____________ **Date**: _________

**Security Review**: ________________ **Date**: _________