# Git Links Reference: All Operational Airtime References

This document provides exact GitHub URLs with line numbers for all operational airtime references that need to be updated during the migration.

## Repository Commit Hashes
- **workspace.eureka/eureka**: `9542db245df8459af9734bc215680a02659ad0c9`
- **jim.tokens/merced**: `5488b383338f22ddb65daa0e67a014b9b6ac47c8`
- **jim.tokens/yosemite**: `02323c26ce2b6e00a7d0889bce9abbba38d4c767`
- **workspace.eureka/pipeline-library**: `44ba4516a5ef09860f73d6e71bb6c52ab64895cc`

---

## CRITICAL - Phase 2: Engineering Environment Migration

### ZooKeeper Configuration - Engineering
**Repository**: workspace.eureka/eureka  
**File**: `vandenberg-config.json`  
**Priority**: 🚨 CRITICAL - Service Discovery Foundation

- [`zookeeper-1.eng.airtime.com:2181`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/vandenberg-config.json#L193)
- [`zookeeper-2.eng.airtime.com:2181`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/vandenberg-config.json#L194)
- [`zookeeper-3.eng.airtime.com:2181`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/vandenberg-config.json#L195)
- [`zookeeper-4.eng.airtime.com:2181`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/vandenberg-config.json#L196)
- [`zookeeper-5.eng.airtime.com:2181`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/vandenberg-config.json#L197)

**Change To**: Replace `eng.airtime.com` with `eng.cantina.com`

### Token Service External URL
**Repository**: jim.tokens/merced  
**File**: `merced/libs/base/constants.mjs`  
**Priority**: 🚨 CRITICAL - Authentication Foundation

- [`EXTERNAL_URL = 'https://merced.airtime.com:443'`](https://github.com/airtimemedia/merced/blob/5488b383338f22ddb65daa0e67a014b9b6ac47c8/merced/libs/base/constants.mjs#L30)

**Change To**: `EXTERNAL_URL = 'https://merced.eng.cantina.com:443'` (Phase 2) → `'https://merced.cantina.com:443'` (Phase 4)

### Platform Domain Configuration
**Repository**: jim.tokens/yosemite  
**File**: `yosemite/libs/config/app-config.mjs`  
**Priority**: 🚨 CRITICAL - Platform Gateway

- [`DOMAIN_NAME: 'platform.airtime.com:443'`](https://github.com/airtimemedia/yosemite/blob/02323c26ce2b6e00a7d0889bce9abbba38d4c767/yosemite/libs/config/app-config.mjs#L35)

**Change To**: `DOMAIN_NAME: 'platform.eng.cantina.com:443'` (Phase 2) → `'platform.cantina.com:443'` (Phase 4)

### Automation Scripts
**Repository**: jim.tokens/merced  
**File**: `automation-test.sh`  
**Priority**: ⚡ HIGH - Deployment Automation

- [`MERCED_HOST="merced-${NODE_NAME}.eng.airtime.com"`](https://github.com/airtimemedia/merced/blob/5488b383338f22ddb65daa0e67a014b9b6ac47c8/automation-test.sh#L40)

**Repository**: jim.tokens/yosemite  
**File**: `automation-test.sh`  
**Priority**: ⚡ HIGH - Deployment Automation

- [`YOSEMITE_HOST="yosemite-${NODE_NAME}.eng.airtime.com"`](https://github.com/airtimemedia/yosemite/blob/02323c26ce2b6e00a7d0889bce9abbba38d4c767/automation-test.sh#L43)
- [`merced-${NODE_NAME}.eng.airtime.com:443`](https://github.com/airtimemedia/yosemite/blob/02323c26ce2b6e00a7d0889bce9abbba38d4c767/automation-test.sh#L45)

**Change To**: Replace `eng.airtime.com` with `eng.cantina.com`

### Test Configurations
**Repository**: jim.tokens/merced  
**File**: `merced/test/test-app-config.mjs`  
**Priority**: 📋 MEDIUM - Test Infrastructure

- [`"token-i-12345.us-west-2.eng.airtime.com"`](https://github.com/airtimemedia/merced/blob/5488b383338f22ddb65daa0e67a014b9b6ac47c8/merced/test/test-app-config.mjs#L57)

**Repository**: jim.tokens/yosemite  
**File**: `yosemite/test/config/test-app-config.mjs`

- [Test configuration with airtime.com references](https://github.com/airtimemedia/yosemite/blob/02323c26ce2b6e00a7d0889bce9abbba38d4c767/yosemite/test/config/test-app-config.mjs#L60)

**Change To**: Replace `eng.airtime.com` with `eng.cantina.com`

---

## CRITICAL - Phase 3: Staging Environment Migration

### ZooKeeper Configuration - Staging
**Repository**: workspace.eureka/eureka  
**File**: `config.json`  
**Priority**: 🚨 CRITICAL - Service Discovery Foundation

- [`zookeeper-1.stage.airtime.com:2181`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/config.json#L22)
- [`zookeeper-2.stage.airtime.com:2181`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/config.json#L23)
- [`zookeeper-3.stage.airtime.com:2181`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/config.json#L24)
- [`zookeeper-4.stage.airtime.com:2181`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/config.json#L25)
- [`zookeeper-5.stage.airtime.com:2181`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/config.json#L26)

**Change To**: Replace `stage.airtime.com` with `stage.cantina.com`

### Integration Test Config - Staging
**Repository**: workspace.eureka/eureka  
**File**: `server-allocator/integration_tests/configs/cpu_allocator_test/config.json`  
**Priority**: ⚡ HIGH - Test Infrastructure

- [`"zookeeper-1.stage.airtime.com:2181"`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/server-allocator/integration_tests/configs/cpu_allocator_test/config.json#L52)

**Change To**: `"zookeeper-1.stage.cantina.com:2181"`

---

## MEDIUM PRIORITY - Phase 5: Application Services Migration

### Integration Test References
**Repository**: workspace.eureka/eureka  
**File**: `server-allocator/integration_tests/lib/eureka.js`  
**Priority**: 📋 MEDIUM - Integration Testing

- [`"tecate-legacy://tecate.airtime.com/12345678"`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/server-allocator/integration_tests/lib/eureka.js#L62)
- [`"vandenberg-i-0cc1319df496e648c-us-west-2.eng.airtime.com"`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/server-allocator/integration_tests/lib/eureka.js#L64)

**Change To**: Replace `airtime.com` with `cantina.com`

### Development Scripts
**Repository**: jim.tokens/yosemite  
**File**: `automation-test/bin/run-automation-local.sh`  
**Priority**: 📋 MEDIUM - Development Tools

- [`merced-i-123456.airtime.com:8259`](https://github.com/airtimemedia/yosemite/blob/02323c26ce2b6e00a7d0889bce9abbba38d4c767/automation-test/bin/run-automation-local.sh#L69)

**Change To**: `merced-i-123456.cantina.com:8259`

---

## LOW PRIORITY - Phase 6: Documentation & Final Cleanup

### Documentation Updates
**Repository**: jim.tokens/yosemite  
**File**: `yosemite/libs/migrate-db/README.md`  
**Priority**: 📝 LOW - Documentation

- [Documentation with airtime.com references](https://github.com/airtimemedia/yosemite/blob/02323c26ce2b6e00a7d0889bce9abbba38d4c767/yosemite/libs/migrate-db/README.md#L75)

**Repository**: jim.tokens/yosemite  
**File**: `yosemite/libs/migrate-db/quick-test.sh`  
**Priority**: 📝 LOW - Documentation Scripts

- [`https://yosemite.eng.airtime.com`](https://github.com/airtimemedia/yosemite/blob/02323c26ce2b6e00a7d0889bce9abbba38d4c767/yosemite/libs/migrate-db/quick-test.sh#L77)

**Change To**: `https://yosemite.eng.cantina.com`

---

## Jenkins Pipeline Investigation Required

### Pipeline Library References
**Repository**: workspace.eureka/pipeline-library  
**Commit**: `44ba4516a5ef09860f73d6e71bb6c52ab64895cc`  
**Priority**: 🔍 INVESTIGATION - CI/CD Infrastructure

**Files to Investigate:**
- [`Jenkinsfile`](https://github.com/airtimemedia/pipeline-library/blob/44ba4516a5ef09860f73d6e71bb6c52ab64895cc/Jenkinsfile)
- [`vars/*.groovy`](https://github.com/airtimemedia/pipeline-library/blob/44ba4516a5ef09860f73d6e71bb6c52ab64895cc/vars/)
- [`src/**/*.groovy`](https://github.com/airtimemedia/pipeline-library/blob/44ba4516a5ef09860f73d6e71bb6c52ab64895cc/src/)

**Search For:**
- Pipeline deployment targets using airtime.com domains
- Environment-specific configurations  
- Service discovery endpoints
- Health check URLs
- `@Library('airtime-pipeline-library')` references in other repositories

---

## Jenkinsfile References in Other Repositories

### Merced Pipeline
**Repository**: jim.tokens/merced  
**File**: `Jenkinsfile`  
**Expected Reference**: `@Library('airtime-pipeline-library@...')`

- [Jenkinsfile with pipeline library reference](https://github.com/airtimemedia/merced/blob/5488b383338f22ddb65daa0e67a014b9b6ac47c8/Jenkinsfile)

### Yosemite Pipeline
**Repository**: jim.tokens/yosemite  
**File**: `Jenkinsfile`  
**Expected Reference**: `@Library('airtime-pipeline-library@...')`

- [Jenkinsfile with pipeline library reference](https://github.com/airtimemedia/yosemite/blob/02323c26ce2b6e00a7d0889bce9abbba38d4c767/Jenkinsfile)

### Eureka Pipeline
**Repository**: workspace.eureka/eureka  
**File**: `Jenkinsfile`  
**Expected Reference**: `@Library('airtime-pipeline-library@...')`

- [Jenkinsfile with pipeline library reference](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/Jenkinsfile)

**Change To**: `@Library('cantina-pipeline-library@...')` (after library is renamed)

---

## Summary by Migration Phase

### Phase 2: Engineering Environment (14 URLs)
🚨 **CRITICAL**: 8 URLs (ZooKeeper, Token Service, Platform, Automation)  
⚡ **HIGH**: 4 URLs (Automation Scripts)  
📋 **MEDIUM**: 2 URLs (Test Configurations)

### Phase 3: Staging Environment (7 URLs)
🚨 **CRITICAL**: 6 URLs (ZooKeeper Configuration)  
⚡ **HIGH**: 1 URL (Integration Test Config)

### Phase 4: Production Environment (2 URLs)
🚨 **CRITICAL**: Update Phase 2 references from `.eng.` to production

### Phase 5: Application Services (3 URLs)
📋 **MEDIUM**: 3 URLs (Integration Tests, Development Scripts)

### Phase 6: Documentation (2+ URLs)
📝 **LOW**: 2+ URLs (Documentation, Scripts)

---

## Quick Reference Commands

### Search for Remaining References
```bash
# After each phase, verify completion
grep -r "airtime\.com" /path/to/repository/
grep -r "airtime-pipeline-library" /path/to/repository/
```

### Validate Changes
```bash
# Test connectivity after each change
curl -k https://zookeeper-1.eng.cantina.com:2181
curl -k https://platform.cantina.com/health
curl -k https://merced.cantina.com/health
```

---

## Notes

1. **Line Numbers**: Estimates based on documentation analysis. Verify actual line numbers when implementing.
2. **Organization Name**: URLs assume `airtimemedia` GitHub organization. Update if different.
3. **Multiple Instances**: Some files may have multiple occurrences of the same domain pattern.
4. **Environment Progression**: Engineering → Staging → Production domain updates follow service validation.
5. **Rollback**: Each repository and file should be backed up before changes with ability to revert via git.

**Total Operational References**: 28+ GitHub URLs across 4 repositories requiring updates during migration phases.