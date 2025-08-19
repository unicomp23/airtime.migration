# Infrastructure as Code (IaC) Migration Requirements

## Critical Infrastructure Repositories Missing from Analysis

This document identifies Infrastructure as Code (IaC) repositories and configurations that **MUST** be analyzed and updated as part of the airtime.com → cantina.com migration but were not accessible during the initial analysis.

---

## 🚨 HIGH PRIORITY - Infrastructure Repositories

### 1. Ansible Configuration Management
**Repository**: `airtimemedia/media-ansible`  
**Last Updated**: 2025-08-19  
**Priority**: 🔴 CRITICAL

**Expected airtime.com References**:
- Inventory host definitions
- Playbook deployment targets
- Group variables with domain configurations
- SSL certificate deployment tasks
- Service configuration templates

**Migration Tasks**:
```yaml
# Example files to update:
inventory/hosts:
  - zookeeper-1.eng.airtime.com
  - platform.airtime.com
  - merced.airtime.com

playbooks/deploy-services.yml:
  - host: "*.airtime.com"
  - domain_name: "airtime.com"

group_vars/all.yml:
  - base_domain: "airtime.com"
  - ssl_domain: "*.airtime.com"
```

---

### 2. Terraform Infrastructure
**Repository**: `airtimemedia/empire-terraform`  
**Last Updated**: 2025-08-19  
**Priority**: 🔴 CRITICAL

**Expected airtime.com References**:
- Route53 DNS zone configurations
- ACM certificate requests
- Load balancer target groups
- CloudFront distributions
- S3 bucket policies

**Migration Tasks**:
```hcl
# Example Terraform files to update:
dns.tf:
  - resource "aws_route53_zone" "airtime_com"
  - resource "aws_route53_record" "platform"

certificates.tf:
  - domain_name = "*.airtime.com"
  - subject_alternative_names = ["*.eng.airtime.com", "*.stage.airtime.com"]

load_balancers.tf:
  - host_header = ["platform.airtime.com"]
  - target_group_name = "airtime-platform"
```

---

### 3. Terraform Monitoring
**Repository**: `airtimemedia/empire-terraform-sumologic`  
**Last Updated**: 2025-08-19  
**Priority**: ⚡ HIGH

**Expected airtime.com References**:
- Log collection endpoints
- Metric namespaces
- Alert configurations
- Dashboard definitions

**Migration Tasks**:
- Update Sumo Logic collectors
- Modify alert rules with new domain patterns
- Update dashboard queries

---

### 4. DevOps Tools & Scripts
**Repository**: `yoinc/devops`  
**Last Updated**: 2025-08-14  
**Priority**: ⚡ HIGH

**Expected airtime.com References**:
- Deployment scripts
- Health check scripts
- DNS update scripts
- Certificate renewal automation

---

### 5. Jenkins CI/CD
**Repository**: `yoinc/jenkins`  
**Last Updated**: 2022-12-08 (Backup)  
**Priority**: 📋 MEDIUM

**Expected airtime.com References**:
- Job configurations
- Pipeline definitions
- Deployment targets
- Webhook URLs

---

## 📋 IaC File Patterns to Search

### Terraform Files
```bash
# Search commands for Terraform repositories:
find . -name "*.tf" -exec grep -l "airtime\.com" {} \;
find . -name "*.tfvars" -exec grep -l "airtime\.com" {} \;
find . -name "terraform.tfstate*" -exec grep -l "airtime\.com" {} \;
```

**Expected Files**:
- `variables.tf` - Domain variable definitions
- `dns.tf` - Route53 zone and record configurations
- `certificates.tf` - ACM certificate configurations
- `alb.tf` / `load_balancer.tf` - ALB/NLB configurations
- `cloudfront.tf` - CDN distributions
- `s3.tf` - Bucket policies with domain references

### Ansible Files
```bash
# Search commands for Ansible repositories:
find . -name "*.yml" -exec grep -l "airtime\.com" {} \;
find . -name "*.yaml" -exec grep -l "airtime\.com" {} \;
find . -path "*/inventory/*" -exec grep -l "airtime\.com" {} \;
find . -path "*/group_vars/*" -exec grep -l "airtime\.com" {} \;
```

**Expected Files**:
- `inventory/production` - Production host definitions
- `inventory/staging` - Staging host definitions
- `group_vars/all.yml` - Global variables
- `playbooks/site.yml` - Main playbook
- `roles/*/defaults/main.yml` - Role default variables
- `roles/*/templates/*.j2` - Configuration templates

### Kubernetes Manifests
```bash
# Search commands if K8s manifests exist:
find . -name "*.yaml" -path "*/k8s/*" -exec grep -l "airtime\.com" {} \;
kubectl get ingress -A -o yaml | grep airtime.com
kubectl get configmap -A -o yaml | grep airtime.com
```

**Expected Resources**:
- Ingress rules with airtime.com hosts
- ConfigMaps with domain configurations
- Secrets with SSL certificates
- Service definitions with external names

### Docker Configurations
```bash
# Search commands for Docker configurations:
find . -name "Dockerfile*" -exec grep -l "airtime\.com" {} \;
find . -name "docker-compose*.yml" -exec grep -l "airtime\.com" {} \;
find . -name ".env*" -exec grep -l "airtime\.com" {} \;
```

---

## 🔄 Migration Phases for IaC

### Phase 0: IaC Preparation
- [ ] Clone all infrastructure repositories
- [ ] Create branches for migration changes
- [ ] Inventory all airtime.com references
- [ ] Document current infrastructure state

### Phase 1: DNS Infrastructure (Terraform)
- [ ] Create cantina.com Route53 hosted zone
- [ ] Duplicate DNS records with cantina.com
- [ ] Set up CNAME bridges in Terraform
- [ ] Generate new ACM certificates

### Phase 2-3: Environment Updates (Ansible)
- [ ] Update inventory files for engineering
- [ ] Update inventory files for staging
- [ ] Modify playbook variables
- [ ] Update configuration templates

### Phase 4: Production Infrastructure (Critical)
- [ ] Update production Terraform configurations
- [ ] Modify production Ansible playbooks
- [ ] Update monitoring configurations
- [ ] Adjust alerting rules

### Phase 5-6: Cleanup
- [ ] Remove airtime.com from Terraform state
- [ ] Clean up old Ansible inventory entries
- [ ] Update CI/CD pipeline configurations
- [ ] Archive old configurations

---

## ⚠️ Critical IaC Considerations

### Terraform State Management
```bash
# CRITICAL: Before making changes
terraform plan -out=migration.plan
terraform state backup
terraform state list | grep airtime

# After migration
terraform state rm <airtime_resources>
```

### Ansible Inventory Backup
```bash
# Backup current inventory
cp -r inventory/ inventory.backup.$(date +%Y%m%d)

# Test new inventory
ansible-playbook -i inventory.new --check site.yml
```

### Certificate Migration
- Request new certificates for cantina.com BEFORE migration
- Maintain both certificates during transition
- Update certificate ARNs in load balancers
- Monitor certificate expiry dates

### DNS TTL Considerations
- Lower TTLs to 300 seconds before migration
- Use Terraform to manage DNS changes
- Implement gradual traffic shifting

---

## 📊 Infrastructure Components Impact Matrix

| Component | Tool | Priority | Rollback Complexity | Phase |
|-----------|------|----------|-------------------|-------|
| DNS Zones | Terraform | 🔴 CRITICAL | Easy | 1 |
| SSL Certificates | Terraform/ACM | 🔴 CRITICAL | Medium | 1 |
| Load Balancers | Terraform | 🔴 CRITICAL | Complex | 4 |
| Service Discovery | Ansible | 🔴 CRITICAL | Complex | 2-3 |
| Monitoring | Terraform | ⚡ HIGH | Easy | 5 |
| CI/CD Pipelines | Jenkins | 📋 MEDIUM | Easy | 5 |
| Documentation | Manual | 📝 LOW | N/A | 6 |

---

## 🚨 Immediate Actions Required

1. **Access Infrastructure Repositories**:
   ```bash
   git clone git@github.com:airtimemedia/media-ansible.git
   git clone git@github.com:airtimemedia/empire-terraform.git
   git clone git@github.com:airtimemedia/empire-terraform-sumologic.git
   ```

2. **Run Comprehensive Search**:
   ```bash
   # Create search script
   #!/bin/bash
   for repo in media-ansible empire-terraform empire-terraform-sumologic; do
     echo "=== Searching $repo ==="
     cd $repo
     grep -r "airtime\.com" . --include="*.tf" --include="*.yml" \
       --include="*.yaml" --include="*.tfvars" | tee ../$repo-airtime-refs.txt
     cd ..
   done
   ```

3. **Document Findings**:
   - Create inventory of all IaC airtime.com references
   - Map references to migration phases
   - Identify dependencies between components
   - Plan rollback procedures for each component

---

## 📝 Notes

- This analysis is based on repository names from `github-org-repos.md`
- Actual infrastructure code was not accessible during initial analysis
- Additional IaC tools (Chef, Puppet, CloudFormation) may exist
- Private repositories require proper authentication to access
- Some infrastructure may be managed through AWS Console (manual review needed)

**Critical**: Complete IaC audit BEFORE starting Phase 1 DNS bridge setup to ensure all infrastructure dependencies are identified.