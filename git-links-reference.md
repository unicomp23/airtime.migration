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

- [`"disco-servers": "zookeeper-1.eng.airtime.com:2181, zookeeper-2.eng.airtime.com:2181, zookeeper-3.eng.airtime.com:2181, zookeeper-4.eng.airtime.com:2181, zookeeper-5.eng.airtime.com:2181"`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/vandenberg-config.json#L16)

**Change To**: Replace `eng.airtime.com` with `eng.cantina.com`

### Token Service External URL
**Repository**: jim.tokens/merced  
**File**: `merced/libs/base/constants.mjs`  
**Priority**: 🚨 CRITICAL - Authentication Foundation

- [`export let EXTERNAL_URL = 'https://merced.airtime.com:443';`](https://github.com/airtimemedia/merced/blob/5488b383338f22ddb65daa0e67a014b9b6ac47c8/merced/libs/base/constants.mjs#L4)

**Change To**: `EXTERNAL_URL = 'https://merced.eng.cantina.com:443'` (Phase 2) → `'https://merced.cantina.com:443'` (Phase 4)

### Platform Domain Configuration
**Repository**: jim.tokens/yosemite  
**File**: `yosemite/libs/config/app-config.mjs`  
**Priority**: 🚨 CRITICAL - Platform Gateway

- [`DOMAIN_NAME: 'platform.airtime.com:443',`](https://github.com/airtimemedia/yosemite/blob/02323c26ce2b6e00a7d0889bce9abbba38d4c767/yosemite/libs/config/app-config.mjs#L42)

**Change To**: `DOMAIN_NAME: 'platform.eng.cantina.com:443'` (Phase 2) → `'platform.cantina.com:443'` (Phase 4)

### Automation Scripts
**Repository**: jim.tokens/merced  
**File**: `automation-test.sh`  
**Priority**: ⚡ HIGH - Deployment Automation

- [`export MERCED_HOST="merced-${NODE_NAME}.eng.airtime.com"`](https://github.com/airtimemedia/merced/blob/5488b383338f22ddb65daa0e67a014b9b6ac47c8/automation-test.sh#L57)

**Repository**: jim.tokens/yosemite  
**File**: `automation-test.sh`  
**Priority**: ⚡ HIGH - Deployment Automation

- [`export YOSEMITE_HOST="yosemite-${NODE_NAME}.eng.airtime.com"`](https://github.com/airtimemedia/yosemite/blob/02323c26ce2b6e00a7d0889bce9abbba38d4c767/automation-test.sh#L57)
- [`npm config set YOSEMITE_REGRESSIONS:CONFIG_TOKEN_SERVER_HOST_PORT "merced-${NODE_NAME}.eng.airtime.com:443"`](https://github.com/airtimemedia/yosemite/blob/02323c26ce2b6e00a7d0889bce9abbba38d4c767/automation-test.sh#L64)

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

- [`"disco-servers": "zookeeper-1.stage.airtime.com:2181, zookeeper-2.stage.airtime.com:2181, zookeeper-3.stage.airtime.com:2181, zookeeper-4.stage.airtime.com:2181, zookeeper-5.stage.airtime.com:2181"`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/config.json#L15)

**Change To**: Replace `stage.airtime.com` with `stage.cantina.com`

### Integration Test Config - Staging
**Repository**: workspace.eureka/eureka  
**File**: `server-allocator/integration_tests/configs/cpu_allocator_test/config.json`  
**Priority**: ⚡ HIGH - Test Infrastructure

- [`"#disco-servers": "zookeeper-1.stage.airtime.com:2181, zookeeper-2.stage.airtime.com:2181, zookeeper-3.stage.airtime.com:2181"`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/server-allocator/integration_tests/configs/cpu_allocator_test/config.json#L11)
- [`"disco-servers": "zookeeper-1-automation1.eng.airtime.com:2181"`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/server-allocator/integration_tests/configs/cpu_allocator_test/config.json#L12)

**Change To**: Replace `airtime.com` with `cantina.com`

---

## MEDIUM PRIORITY - Phase 5: Application Services Migration

### Integration Test References
**Repository**: workspace.eureka/eureka  
**File**: `server-allocator/integration_tests/lib/eureka.js`  
**Priority**: 📋 MEDIUM - Integration Testing

- [`ipAddr + '&stream-url=tecate-legacy://tecate.airtime.com/12345678';`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/server-allocator/integration_tests/lib/eureka.js#L46)
- [`// "server": "vandenberg-i-0cc1319df496e648c-us-west-2.eng.airtime.com",`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/server-allocator/integration_tests/lib/eureka.js#L24)

**Repository**: workspace.eureka/eureka  
**File**: `server-allocator/integration_tests/tests/all-allocator-test.js`  
**Priority**: 📋 MEDIUM - Integration Testing

- [`var ZOO_NODE  = 'zookeeper-1-' + process.env['NODE_NAME'] + '.eng.airtime.com';`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/server-allocator/integration_tests/tests/all-allocator-test.js#L25)
- [`if (VANDENBERG) UUT = 'http://vandenberg-allocator-' + process.env['NODE_NAME'] + '.eng.airtime.com:8192';`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/server-allocator/integration_tests/tests/all-allocator-test.js#L116)
- [`url: UUT + '/allocate-subscribe?stream-url=tecate-legacy://tecate.airtime.com/12345678'`](https://github.com/airtimemedia/eureka/blob/9542db245df8459af9734bc215680a02659ad0c9/server-allocator/integration_tests/tests/all-allocator-test.js#L615)

**Change To**: Replace `airtime.com` with `cantina.com`

### Development Scripts
**Repository**: jim.tokens/yosemite  
**File**: `automation-test/bin/run-automation-local.sh`  
**Priority**: 📋 MEDIUM - Development Tools

- [`#YOSEMITE_TOKEN_SERVER_HOST_PORT=merced-i-123456.airtime.com:8259 \`](https://github.com/airtimemedia/yosemite/blob/02323c26ce2b6e00a7d0889bce9abbba38d4c767/automation-test/bin/run-automation-local.sh#L8)

**Change To**: `merced-i-123456.cantina.com:8259` (uncomment and update when used)

---

## LOW PRIORITY - Phase 6: Documentation & Final Cleanup

### Documentation Updates
**Repository**: jim.tokens/yosemite  
**File**: `yosemite/libs/migrate-db/README.md`  
**Priority**: 📝 LOW - Documentation

- [`\`yosemite/libs/migrate-db/quick-test.sh https://yosemite.eng.airtime.com <your dev portal token file> <test customer id>\``](https://github.com/airtimemedia/yosemite/blob/02323c26ce2b6e00a7d0889bce9abbba38d4c767/yosemite/libs/migrate-db/README.md#L116)

**Repository**: jim.tokens/yosemite  
**File**: `yosemite/libs/migrate-db/quick-test.sh`  
**Priority**: 📝 LOW - Documentation Scripts

- [`echo 'Ex: https://yosemite.eng.airtime.com /some/where/devportal.token "big customer"'`](https://github.com/airtimemedia/yosemite/blob/02323c26ce2b6e00a7d0889bce9abbba38d4c767/yosemite/libs/migrate-db/quick-test.sh#L7)

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

### Phase 2: Engineering Environment 
🚨 **CRITICAL**: 
- 1 URL - ZooKeeper Configuration (vandenberg-config.json line 16)
- 1 URL - Token Service External URL (constants.mjs line 4)
- 1 URL - Platform Domain (app-config.mjs line 42)

⚡ **HIGH**: 
- 3 URLs - Automation Scripts (automation-test.sh lines 57, 64)

📋 **MEDIUM**:
- Test configurations (various test files)

### Phase 3: Staging Environment
🚨 **CRITICAL**: 
- 1 URL - ZooKeeper Configuration (config.json line 15)

⚡ **HIGH**: 
- 2 URLs - Integration Test Config (cpu_allocator_test/config.json lines 11-12)

### Phase 4: Production Environment
🚨 **CRITICAL**: Update Phase 2 references from `.eng.` to production domains

### Phase 5: Application Services
📋 **MEDIUM**: 
- 6+ URLs - Integration Tests (eureka.js, all-allocator-test.js)
- 1 URL - Development Scripts (run-automation-local.sh line 8)

### Phase 6: Documentation
📝 **LOW**: 
- 2 URLs - Documentation (README.md line 116, quick-test.sh line 7)

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