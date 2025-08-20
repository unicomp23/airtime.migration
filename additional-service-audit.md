# Additional Service Repository Audit

## Critical Service Repositories Requiring Analysis

Based on the GitHub organization inventory, the following **active service repositories** were not included in the initial migration analysis but likely contain airtime.com references:

---

## 🔴 **HIGH PRIORITY - Active Service Repositories**

### **airtimemedia Organization (2025-08-19 - Active)**

#### **Core Backend Services**
1. **`airtimemedia/backend`** - Core backend services
   - **Priority**: 🔴 CRITICAL
   - **Expected References**: Database connections, API endpoints, service URLs
   - **Search Commands**:
     ```bash
     grep -r "airtime\.com" . --include="*.js" --include="*.ts" --include="*.json"
     grep -r "airtime\.com" . --include="*.env*" --include="*.config*"
     ```

2. **`airtimemedia/deathstar`** - Airtime's galactic superweapon API for the empire
   - **Priority**: 🔴 CRITICAL  
   - **Expected References**: API endpoints, authentication URLs, service discovery
   - **Likely Files**: Configuration files, environment variables, OpenAPI specs

3. **`airtimemedia/yoda`** - Cantina WebApp! The one and only!
   - **Priority**: 🔴 CRITICAL
   - **Expected References**: Frontend API calls, authentication endpoints
   - **Likely Files**: JavaScript/TypeScript configs, environment files

#### **Media & Communication Services**
4. **`airtimemedia/bigsur`** - Javascript library for interfacing with the media backend
   - **Priority**: ⚡ HIGH
   - **Expected References**: Media API endpoints, WebRTC signaling servers
   - **Likely Files**: JavaScript SDK configurations

5. **`airtimemedia/yuba`** - Speech to Text Service
   - **Priority**: ⚡ HIGH
   - **Expected References**: API endpoints, webhook URLs
   
6. **`airtimemedia/boca`** - TTS (Text-to-Speech) Service
   - **Priority**: ⚡ HIGH
   - **Expected References**: API endpoints, callback URLs

7. **`airtimemedia/tahoe`** - Voice management service
   - **Priority**: ⚡ HIGH
   - **Expected References**: Voice API endpoints, media servers

8. **`airtimemedia/render`** - Graphics rendering engine
   - **Priority**: ⚡ HIGH
   - **Expected References**: Asset URLs, rendering API endpoints

#### **Analytics & Monitoring**
9. **`airtimemedia/darthvader`** - Analytics tracking service
   - **Priority**: ⚡ HIGH
   - **Expected References**: Tracking endpoints, metrics collection URLs

10. **`airtimemedia/ventana-master`** - Host state management across regions
    - **Priority**: ⚡ HIGH
    - **Expected References**: Regional endpoint configurations, health check URLs

#### **Client Libraries**
11. **`airtimemedia/backend-client-js`** - JavaScript client library
    - **Priority**: ⚡ HIGH
    - **Expected References**: API base URLs, authentication endpoints

12. **`airtimemedia/R2D2`** - New Android client
    - **Priority**: 📋 MEDIUM
    - **Expected References**: API endpoints in Android configs

---

## ⚡ **MEDIUM PRIORITY - Build & Tooling**

### **yoinc Organization**

13. **`yoinc/bixby`** - Recently updated (2025-08-19)
    - **Priority**: 📋 MEDIUM
    - **Expected References**: Service endpoints, configuration files

14. **`yoinc/tecate`** - Recently updated (2025-08-18)
    - **Priority**: 📋 MEDIUM
    - **Expected References**: Service discovery, health check endpoints

15. **`yoinc/build`** - Airtime build tools and scripts
    - **Priority**: 📋 MEDIUM
    - **Expected References**: Deployment targets, artifact repositories

16. **`yoinc/certs`** - SSL Certificates
    - **Priority**: ⚡ HIGH
    - **Expected References**: Certificate domains, renewal scripts

### **vline Organization**
17. **`vline/vline-webrtc`** - Recently updated (2025-08-19)
    - **Priority**: 📋 MEDIUM
    - **Expected References**: STUN/TURN servers, signaling endpoints

---

## 🔍 **Search Strategy for Each Repository**

### **1. Clone and Search Pattern**
```bash
# For each repository:
git clone git@github.com:airtimemedia/[REPO_NAME].git
cd [REPO_NAME]

# Search for airtime.com references
grep -r "airtime\.com" . \
  --include="*.js" --include="*.ts" --include="*.json" \
  --include="*.yml" --include="*.yaml" --include="*.env*" \
  --include="*.config*" --include="*.properties" \
  > ../[REPO_NAME]-airtime-refs.txt

# Count references found  
grep -c "airtime\.com" ../[REPO_NAME]-airtime-refs.txt
```

### **2. File Types to Prioritize**
- **Configuration Files**: `*.json`, `*.yml`, `*.yaml`, `*.config.*`
- **Environment Files**: `.env*`, `config.js`, `settings.js`
- **Source Code**: `*.js`, `*.ts`, `*.py`, `*.go`, `*.java`
- **Build Files**: `package.json`, `Dockerfile`, `*.gradle`
- **Documentation**: `README.md`, `*.md` (may contain example URLs)

### **3. Expected Reference Types**
- **API Base URLs**: `https://api.airtime.com`, `https://backend.airtime.com`
- **Authentication URLs**: `https://auth.airtime.com`, `https://login.airtime.com`
- **WebSocket URLs**: `wss://ws.airtime.com`, `wss://realtime.airtime.com`
- **CDN URLs**: `https://cdn.airtime.com`, `https://assets.airtime.com`
- **Webhook URLs**: `https://webhooks.airtime.com`
- **Documentation URLs**: `https://docs.airtime.com`

---

## 📊 **Expected Impact Assessment**

### **Critical Services (Require immediate attention)**
| Repository | Service Type | Expected References | Migration Phase |
|------------|--------------|-------------------|-----------------|
| backend | Core API | Database, auth, APIs | Phase 2-4 |
| deathstar | Main API | Service endpoints | Phase 4 |
| yoda | Web Frontend | API calls, assets | Phase 4 |
| bigsur | Media SDK | Media endpoints | Phase 2-4 |

### **Supporting Services (Important but not blocking)**
| Repository | Service Type | Expected References | Migration Phase |
|------------|--------------|-------------------|-----------------|
| yuba, boca, tahoe | Media Services | API endpoints | Phase 5 |
| darthvader | Analytics | Tracking URLs | Phase 5 |
| backend-client-js | Client Library | SDK configurations | Phase 5 |

---

## ⚠️ **Critical Findings to Look For**

### **1. Database Connection Strings**
```javascript
// Look for patterns like:
const DB_HOST = "db.airtime.com";
mongodb://user:pass@mongo.airtime.com/database
postgres://user:pass@postgres.airtime.com/database
```

### **2. Service Discovery Configurations**
```yaml
# Look for service registries:
eureka:
  client:
    serviceUrl:
      defaultZone: http://eureka.airtime.com:8761/eureka/
```

### **3. External Integration Callbacks**
```javascript
// Webhook configurations:
const WEBHOOK_BASE = "https://webhooks.airtime.com";
const CALLBACK_URL = "https://api.airtime.com/callbacks";
```

### **4. Frontend API Configurations**
```javascript
// Frontend environment configs:
const API_BASE_URL = "https://api.airtime.com/v1";
const WS_ENDPOINT = "wss://realtime.airtime.com";
```

---

## 📋 **Action Items**

### **Immediate (Next Steps)**
- [ ] Clone and audit the 4 critical repositories (backend, deathstar, yoda, bigsur)
- [ ] Generate search results for each repository
- [ ] Update `git-links-reference.md` with new findings
- [ ] Add new references to migration checklists

### **Short Term**
- [ ] Audit medium priority repositories
- [ ] Update phase diagrams if new service dependencies found
- [ ] Adjust migration timeline based on additional complexity

### **Long Term**  
- [ ] Coordinate with repository owners for access if needed
- [ ] Validate findings with development teams
- [ ] Update rollback procedures for newly identified services

---

## 🎯 **Success Criteria**

- **Complete inventory** of all airtime.com references across all active repositories
- **Updated migration plan** reflecting all service dependencies  
- **Comprehensive timeline** accounting for all services requiring updates
- **Zero missed references** that could cause post-migration issues

**Note**: This audit is essential for ensuring the migration captures ALL airtime.com dependencies, not just the initially identified service configuration files.