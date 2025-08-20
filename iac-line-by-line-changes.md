# Infrastructure as Code - Line by Line Changes Required

## Repository Commit References
- **media-ansible**: `fb5552ca` (2025-08-20)
- **empire-terraform**: `9b905df` (2025-08-20)
- **empire-terraform-sumologic**: `75c9ff6` (2025-08-20)
- **devops**: `30aa19b` (2025-08-20)

---

## 🔴 CRITICAL - Terraform Changes

### empire-terraform Repository

#### **root.tf** - Environment Configurations
**File**: `/root/repo/dev/empire-terraform/root.tf`

**Staging Environment**:
- Line 120: `yosemite_base_url = "https://yosemite.stage.airtime.com"` → `"https://yosemite.stage.cantina.com"`
- Line 265: `hal_9000_text_to_speech_url = "https://boca.stage.airtime.com"` → `"https://boca.stage.cantina.com"`

**Production Environment**:
- Line 521: `yosemite_base_url = "https://yosemite.prod.airtime.com"` → `"https://yosemite.prod.cantina.com"`
- Line 666: `hal_9000_text_to_speech_url = "https://boca.prod.airtime.com"` → `"https://boca.prod.cantina.com"`

#### **certs_airtime_com.tf** - SSL Certificates
**File**: `/root/repo/dev/empire-terraform/certs_airtime_com.tf`

- Line 2: Comment update required
- Line 4: `domain_name = "*.airtime.com"` → Create new resource for `"*.cantina.com"`
- Line 6: `"airtime.com",` → Add `"cantina.com",` to subject_alternative_names

**Migration Strategy**: Create parallel certificate resource first, then cutover

#### **common/ecs_*.tf** - ECS Task Definitions
Each ECS task definition file needs environment variable updates:

1. `common/ecs_bot_core.tf`
2. `common/ecs_goldie.tf`
3. `common/ecs_holodeck_core.tf`
4. `common/ecs_inference_task.tf`
5. `common/ecs_notification_core.tf`
6. `common/ecs_rey_adhoc.tf`
7. `common/ecs_rey.tf`
8. `common/ecs_voice_public.tf`

**Pattern to search/replace**:
- API endpoints: `https://api.airtime.com` → `https://api.cantina.com`
- Auth endpoints: `https://auth.airtime.com` → `https://auth.cantina.com`

#### **common/alb_rules_public.tf** - Load Balancer Rules
Host-based routing rules need updates for:
- Host headers matching `*.airtime.com`
- Redirect rules from airtime.com to cantina.com

#### **common/cloudfront.tf** - CDN Configuration
- Update origin domain names
- Update CNAME aliases
- Update cache behaviors

---

## 🔴 CRITICAL - Ansible Changes

### media-ansible Repository

#### **group_vars/all/domains.yml**
**File**: `/root/repo/dev/media-ansible/group_vars/all/domains.yml`
**Lines**: 2-57 (all service definitions)

```yaml
# Line 2-57: Update all domain mappings
airtime_domains: {
    generic: cantina.com,          # Line 3
    abrt-server: cantina.com,      # Line 4
    coredump-server: cantina.com,  # Line 5
    bixby: cantina.com,            # Line 6
    # ... continue for all 56 services
}
```

#### **group_vars/all/zookeeper.yml**
**File**: `/root/repo/dev/media-ansible/group_vars/all/zookeeper.yml`

**Engineering Environment** (Lines 16-54):
- Line 21: `"zookeeper-1{{ zookeeper_extension }}-internal.eng.airtime.com"` → `.eng.cantina.com"`
- Line 22: `"zookeeper-1.eng.airtime.com"` → `"zookeeper-1.eng.cantina.com"`
- Lines 28-29, 35-36, 42-43, 49-50: Similar updates for nodes 2-5

**Staging Environment** (Lines 56-94):
- Line 61: `"zookeeper-1{{ zookeeper_extension }}-internal.stage.airtime.com"` → `.stage.cantina.com"`
- Line 62: `"zookeeper-1.stage.airtime.com"` → `"zookeeper-1.stage.cantina.com"`
- Lines 68-69, 75-76, 82-83, 89-90: Similar updates for nodes 2-5

**Production Environment** (Lines 96-134):
- Line 101: `"zookeeper-1{{ zookeeper_extension }}-internal.prod.airtime.com"` → `.prod.cantina.com"`
- Line 102: `"zookeeper-1.prod.airtime.com"` → `"zookeeper-1.prod.cantina.com"`
- Lines 108-109, 115-116, 122-123, 129-130: Similar updates for nodes 2-5

**Automation Environment** (Lines 136-183):
- Lines 141, 142, 148, 149, 155, 156: Update all `.eng.airtime.com` references
- Lines 164-182: Update version_hosts references

**Yuba Clusters** (Lines 186-307):
Similar pattern for yuba-specific ZooKeeper clusters

#### **inventory/hosts**
**File**: `/root/repo/dev/media-ansible/inventory/hosts`

Sample updates (first 20 lines shown):
- Line 2: `osx-worker2.eng.airtime.com` → `osx-worker2.eng.cantina.com`
- Line 3: `osx-worker10.eng.airtime.com` → `osx-worker10.eng.cantina.com`
- Line 4: `osx-worker11.eng.airtime.com` → `osx-worker11.eng.cantina.com`
- Line 5: `osx-worker20.eng.airtime.com` → `osx-worker20.eng.cantina.com`
- Line 8: `osx-worker100.eng.airtime.com` → `osx-worker100.eng.cantina.com`
- Line 9: `osx-worker101.eng.airtime.com` → `osx-worker101.eng.cantina.com`

**Total**: Approximately 100+ host entries need updating

#### **roles/base/templates/airtime-yum-repo.repo.j2**
YUM repository configuration template:
- Update repository baseurl from `http://yum.airtime.com/...` to internal repository URL

#### **roles/jenkins-*/templates/airtime-yum-repo.repo.j2**
Multiple Jenkins role templates:
1. `roles/jenkins-worker-base-ami/templates/airtime-yum-repo.repo.j2`
2. `roles/jenkins-worker-base-al2023-ami/templates/airtime-yum-repo.repo.j2`
3. `roles/jenkins-graviton-worker-base-ami/templates/airtime-yum-repo.repo.j2`
4. `roles/jenkins-master-base/templates/airtime-yum-repo.repo.j2`

#### **scripts/*.py** - Python Automation Scripts
Key scripts requiring updates:

1. **scripts/create_automation_environment.py**
   - ZooKeeper connection strings
   - Service endpoints

2. **scripts/deploy_group.py**
   - Deployment target hosts
   - Service discovery endpoints

3. **scripts/update_support_cnames.py**
   - CNAME record management
   - Route53 zone IDs

4. **scripts/zk-*.py** (Multiple ZooKeeper scripts)
   - `zk-smoketest.py`: Connection strings
   - `zk-load.py`: Test endpoints
   - `zk-latency.py`: Performance test targets

#### **library/zk_*.py** - Ansible Custom Modules
10 ZooKeeper management modules:
- `library/zk_add_znode.py`
- `library/zk_admin.py`
- `library/zk_delete_znode.py`
- `library/zk_disabled_znode.py`
- `library/zk_ephemeral.py`
- `library/zk_get_children.py`
- `library/zk_get_desired_version.py`
- `library/zk_get_paused.py`
- `library/zk_get_znode.py`
- `library/zk_move_znode.py`
- `library/zk_update_znode.py`

Each contains default ZooKeeper connection strings that need updating.

#### **library/aircore_route53.py**
Route53 management module:
- Update hosted zone references
- Update record creation logic

---

## 📊 Change Summary by File Type

| File Type | Files | Lines to Change | Risk |
|-----------|-------|-----------------|------|
| Terraform Variables | 4 | ~8 | HIGH |
| Terraform Resources | 3 | ~15 | CRITICAL |
| Ansible Variables | 3 | ~200+ | CRITICAL |
| Ansible Inventory | 1 | ~100+ | HIGH |
| Python Scripts | 25+ | ~500+ | MEDIUM |
| Jinja2 Templates | 8+ | ~50+ | MEDIUM |
| YAML Configs | 10+ | ~300+ | HIGH |

---

## 🔧 Automated Migration Script Example

```bash
#!/bin/bash
# migrate-iac.sh - Automated IaC migration script

# Terraform changes
cd /root/repo/dev/empire-terraform
sed -i 's/airtime\.com/cantina.com/g' root.tf
sed -i 's/airtime\.com/cantina.com/g' common/*.tf

# Ansible changes
cd /root/repo/dev/media-ansible
sed -i 's/airtime\.com/cantina.com/g' group_vars/all/domains.yml
sed -i 's/\.eng\.airtime\.com/.eng.cantina.com/g' group_vars/all/zookeeper.yml
sed -i 's/\.stage\.airtime\.com/.stage.cantina.com/g' group_vars/all/zookeeper.yml
sed -i 's/\.prod\.airtime\.com/.prod.cantina.com/g' group_vars/all/zookeeper.yml

# Inventory updates
sed -i 's/\.airtime\.com/.cantina.com/g' inventory/hosts

# Python script updates
find scripts/ -name "*.py" -exec sed -i 's/airtime\.com/cantina.com/g' {} \;
find library/ -name "*.py" -exec sed -i 's/airtime\.com/cantina.com/g' {} \;
```

---

## ⚠️ Pre-Migration Validation Commands

```bash
# Count total references before migration
grep -r "airtime\.com" . | wc -l

# Backup critical files
tar -czf pre-migration-backup.tar.gz \
  group_vars/ \
  inventory/ \
  *.tf \
  common/*.tf

# Dry run Ansible changes
ansible-playbook site.yml --check --diff

# Terraform plan
terraform plan -out=migration.plan
```

---

## 🔄 Rollback Commands

```bash
# Ansible rollback
cd /root/repo/dev/media-ansible
git checkout HEAD -- group_vars/ inventory/ roles/ scripts/ library/

# Terraform rollback
cd /root/repo/dev/empire-terraform
git checkout HEAD -- *.tf common/*.tf
terraform apply -auto-approve previous.plan
```

---

## 📝 Notes

1. **Line numbers are approximate** - Files may have changed since analysis
2. **Test in staging first** - Never apply directly to production
3. **Use version control** - All changes should be in feature branches
4. **Coordinate with teams** - Some services may have external dependencies
5. **Monitor during migration** - Watch for failed health checks

**Total Estimated Lines to Change**: 1,000+  
**Estimated Migration Time**: 40-60 hours  
**Risk Level**: HIGH - Requires careful coordination