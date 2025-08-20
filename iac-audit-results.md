# Infrastructure as Code Audit Results

## Summary
**Date**: 2025-08-20  
**Repositories Audited**: 4  
**Total Files with airtime.com**: 110 files  
**Critical Priority**: Ansible inventory and Terraform state files

---

## 🔴 CRITICAL - Ansible Configuration Changes

### media-ansible Repository (89 files affected)

#### 1. Domain Configuration (`group_vars/all/domains.yml`)
**Impact**: CRITICAL - Affects ALL services  
**File**: `/root/repo/dev/media-ansible/group_vars/all/domains.yml`

All 56 services currently mapped to `airtime.com`:
```yaml
airtime_domains: {
    generic: airtime.com → cantina.com,
    zookeeper: airtime.com → cantina.com,
    merced: airtime.com → cantina.com,
    yosemite: airtime.com → cantina.com,
    # ... 52 more services
}
```

#### 2. ZooKeeper Cluster Configuration (`group_vars/all/zookeeper.yml`)
**Impact**: CRITICAL - Service Discovery Foundation  
**File**: `/root/repo/dev/media-ansible/group_vars/all/zookeeper.yml`

Changes required for ALL environments:
- **Engineering**: 5 ZooKeeper nodes × internal + public hosts = 10 references
- **Staging**: 5 ZooKeeper nodes × internal + public hosts = 10 references  
- **Production**: 5 ZooKeeper nodes × internal + public hosts = 10 references
- **Automation**: 3-6 ZooKeeper nodes × internal + public hosts = 6-12 references
- **Yuba clusters**: Additional 30 references across eng/stage/prod

Total ZooKeeper references: ~70+ host definitions

#### 3. Inventory Hosts (`inventory/hosts`)
**Impact**: HIGH - Deployment targets  
All host definitions need updating from airtime.com to cantina.com

#### 4. Service-Specific Roles
Files requiring updates in roles:
- `roles/zookeeper-base/tasks/main.yml`
- `roles/base/templates/airtime-yum-repo.repo.j2` → Update repo URLs
- `roles/jenkins-*/templates/airtime-yum-repo.repo.j2` → Update across all Jenkins roles
- `roles/static-services-base/files/Caddyfile` → Update proxy rules
- 30+ other role files with hardcoded domains

#### 5. Python Scripts
Critical automation scripts:
- `scripts/create_automation_environment.py`
- `scripts/deploy_group.py`
- `scripts/zk-*.py` (ZooKeeper management scripts)
- `scripts/update_support_cnames.py`
- Route53 management scripts

#### 6. Ansible Library Modules
Custom ZooKeeper modules:
- `library/zk_*.py` - 10 modules with ZK connection strings
- `library/aircore_route53.py` - DNS management

---

## 🔴 CRITICAL - Terraform Configuration Changes

### empire-terraform Repository (12 files affected)

#### 1. Root Configuration (`root.tf`)
Main Terraform configuration with domain references

#### 2. SSL Certificates (`certs_airtime_com.tf`)
**Impact**: CRITICAL - HTTPS services  
ACM certificate configurations for *.airtime.com

#### 3. ECS Task Definitions
Services requiring domain updates:
- `common/ecs_bot_core.tf`
- `common/ecs_goldie.tf`
- `common/ecs_holodeck_core.tf`
- `common/ecs_inference_task.tf`
- `common/ecs_notification_core.tf`
- `common/ecs_rey_adhoc.tf`
- `common/ecs_rey.tf`
- `common/ecs_voice_public.tf`

Environment variables and container configurations

#### 4. Load Balancer Rules (`common/alb_rules_public.tf`)
ALB listener rules with host-based routing

#### 5. CloudFront CDN (`common/cloudfront.tf`)
CDN origins and behaviors configuration

### empire-terraform-sumologic Repository (3 files affected)

#### 1. Data Sources (`common/data.tf`)
References to airtime.com infrastructure

#### 2. PagerDuty Integration (`pagerduty.tf`)
Alert routing configurations

#### 3. Root Configuration (`root.tf`)
Main Sumo Logic configuration

### devops Repository (6 files affected)

#### 1. Certificate Management (`certs/create_new_acm_cert.py`)
Script for creating ACM certificates

#### 2. CloudFormation (`cloudformation/vpc/`)
- `backend.conf` - Backend configuration
- `create-backend-vpc.py` - VPC creation script

#### 3. Deployment Scripts (`deploy/`)
- `deploy.py` - Main deployment script
- `get_ec2_instances.py` - Instance discovery
- `README.md` - Documentation with examples

---

## 📋 Migration Approach by Repository

### Phase 0: Preparation
1. **Backup all current configurations**
   ```bash
   cd /root/repo/dev
   tar -czf ansible-backup-$(date +%Y%m%d).tar.gz media-ansible/
   tar -czf terraform-backup-$(date +%Y%m%d).tar.gz empire-terraform*/
   ```

2. **Create migration branches**
   ```bash
   cd media-ansible && git checkout -b migration/airtime-to-cantina
   cd ../empire-terraform && git checkout -b migration/airtime-to-cantina
   ```

### Phase 1: Ansible Variable Updates
1. Create new domain variable file:
   ```yaml
   # group_vars/all/cantina_domains.yml
   cantina_domains: {
       generic: cantina.com,
       zookeeper: cantina.com,
       # ... all services
   }
   ```

2. Update with environment-specific overrides:
   ```yaml
   # group_vars/eng/domains.yml
   domain_suffix: eng.cantina.com
   
   # group_vars/stage/domains.yml  
   domain_suffix: stage.cantina.com
   
   # group_vars/prod/domains.yml
   domain_suffix: cantina.com
   ```

### Phase 2: Terraform State Migration
1. **Create new Route53 zones**
   ```hcl
   resource "aws_route53_zone" "cantina_com" {
     name = "cantina.com"
   }
   ```

2. **Update ACM certificates**
   ```hcl
   resource "aws_acm_certificate" "cantina_wildcard" {
     domain_name = "*.cantina.com"
     subject_alternative_names = [
       "*.eng.cantina.com",
       "*.stage.cantina.com"
     ]
   }
   ```

3. **Import existing resources**
   ```bash
   terraform import aws_route53_zone.cantina_com Z1234567890ABC
   ```

---

## 🚨 High-Risk Areas

### 1. Hardcoded Values
- **YUM Repository URLs** in multiple template files
- **Python scripts** with inline domain strings
- **Bash scripts** in roles/*/files/

### 2. Service Discovery
- **ZooKeeper connection strings** - 70+ references
- **Eureka service registry** URLs
- **Consul configurations** (if used)

### 3. External Integrations
- **Sumo Logic** collectors and sources
- **PagerDuty** integration endpoints
- **CloudFront** distribution CNAMEs
- **S3 bucket** policies with domain restrictions

### 4. Certificate Pinning
- Services that may have pinned certificates
- Mobile apps with hardcoded endpoints
- Third-party webhook receivers

---

## 📊 File Count by Service

| Service | Files | Priority |
|---------|-------|----------|
| ZooKeeper | 25+ | CRITICAL |
| Jenkins | 8 | HIGH |
| Base/AMI | 6 | HIGH |
| Bixby | 5 | MEDIUM |
| Yuba/Boca/Tahoe | 4 each | MEDIUM |
| Static Services | 1 | LOW |

---

## ⚠️ Required Actions

### Immediate (Before Migration)
1. **Audit Terraform state files**
   ```bash
   terraform state list | grep airtime
   ```

2. **Check for hardcoded IPs**
   ```bash
   grep -r "airtime\.com" --include="*.py" --include="*.sh"
   ```

3. **Review template files**
   ```bash
   find . -name "*.j2" -exec grep -l "airtime\.com" {} \;
   ```

### During Migration
1. Use Ansible `--check` mode first
2. Test with single hosts before groups
3. Maintain parallel configurations during transition
4. Use feature flags where possible

### Post-Migration
1. Clean up old CNAME records
2. Remove deprecated configuration files
3. Update documentation
4. Archive old certificates

---

## 🔄 Rollback Strategy

### Ansible Rollback
```bash
# Quick revert using git
cd /root/repo/dev/media-ansible
git checkout main -- group_vars/all/domains.yml
ansible-playbook site.yml --tags config
```

### Terraform Rollback
```bash
# Revert to previous state
terraform workspace select production
terraform apply -target=aws_route53_zone.airtime_com
```

---

## 📈 Estimated Effort

| Task | Hours | Risk |
|------|-------|------|
| Ansible variable refactoring | 8-12 | Medium |
| Terraform state migration | 16-24 | High |
| Script updates | 4-6 | Low |
| Testing & validation | 8-12 | Medium |
| Production cutover | 4-6 | High |
| **Total** | **40-60 hours** | **High** |

---

## Next Steps

1. **Clone and create feature branches** ✅ Completed
2. **Create detailed file-by-file change list** ← Current task
3. **Build automated migration scripts**
4. **Set up parallel test environment**
5. **Create rollback procedures**
6. **Schedule migration windows**

---

## Critical Questions to Answer

1. **Are there any Terraform remote state dependencies?**
2. **Which services use certificate pinning?**
3. **Are there any hardcoded IPs that resolve to airtime.com?**
4. **What third-party services have webhook URLs configured?**
5. **Are there any mobile apps or embedded devices with hardcoded endpoints?**

**Note**: This audit represents the infrastructure configuration layer. Application code changes are tracked separately in other migration documents.