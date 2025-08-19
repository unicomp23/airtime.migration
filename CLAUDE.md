# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

This repository contains a **comprehensive zero-downtime migration strategy** to transition all services from `airtime.com` to `cantina.com`. The airtime.com domain has been sold and requires complete migration with no service interruption.

## Repository Architecture

### Document Hierarchy
- **Strategic Level**: `domain-migration-plan.md`, `incremental-rollout-plan.md` - High-level strategy and phased approach
- **Execution Level**: `step-by-step-guide.md`, `migration-checklist.md` - Detailed implementation instructions
- **Technical Reference**: `git-links-reference.md` - Exact GitHub URLs with line numbers for code changes
- **Infrastructure**: `infrastructure-migration.md` - IaC repositories (Terraform, Ansible) requirements
- **Safety**: `rollback-procedures.md` - Emergency rollback procedures for each phase
- **Visual**: `phase-*.svg` - Service dependency diagrams for each migration phase

### Critical Infrastructure Repositories (Not in this repo)
The migration requires updates to these infrastructure repositories:
- `airtimemedia/media-ansible` - Ansible playbooks and inventory
- `airtimemedia/empire-terraform` - AWS infrastructure via Terraform
- `airtimemedia/empire-terraform-sumologic` - Monitoring configuration
- `yoinc/devops` - DevOps scripts and tools

### Migration Flow
The migration follows a 6-phase approach:
1. **Phase 0**: Infrastructure setup (DNS, SSL) - no service impact
2. **Phase 1**: DNS bridge creation (CNAME records) - both domains work
3. **Phase 2**: Engineering environment migration - isolated environment
4. **Phase 3**: Staging environment migration - pre-production validation
5. **Phase 4**: Production core services (**CRITICAL PHASE**) - ZooKeeper, authentication
6. **Phase 5-6**: Application services and final cleanup

### Critical Service Architecture
```
ZooKeeper Clusters (Service Discovery) ← Eureka Service Discovery ← Application Services
Platform Gateway (Authentication) ← Token Services (Merced/Yosemite) ← External Integrations
```

## Key Commands

### Infrastructure as Code Commands
```bash
# Clone infrastructure repositories
git clone git@github.com:airtimemedia/media-ansible.git
git clone git@github.com:airtimemedia/empire-terraform.git

# Search for airtime.com in Terraform
grep -r "airtime\.com" . --include="*.tf" --include="*.tfvars"
terraform state list | grep airtime

# Search for airtime.com in Ansible
grep -r "airtime\.com" . --include="*.yml" --include="*.yaml"
find inventory/ -type f -exec grep -l "airtime\.com" {} \;

# Terraform migration commands
terraform plan -out=migration.plan
terraform apply migration.plan
terraform state backup

# Ansible testing
ansible-playbook -i inventory --check site.yml
```

### Application Code Analysis
```bash
# Search for airtime.com references in service repositories
grep -r "airtime\.com" /path/to/repository/

# Get latest commit hashes for git URL generation
git log -1 --format="%H"
```

### Migration Execution
Follow the phase-by-phase approach in `step-by-step-guide.md`:
```bash
# Example Phase 2 commands (Engineering Environment)
# Update ZooKeeper configuration
cp vandenberg-config.json vandenberg-config.json.backup
# Edit: zookeeper-{1-5}.eng.airtime.com → zookeeper-{1-5}.eng.cantina.com

# Deploy updates
./deploy-eureka.sh --environment=engineering
kubectl get pods -n engineering | grep eureka
```

### Validation Commands
```bash
# Test service health after each phase
curl -k https://platform.cantina.com/health
curl -k https://merced.cantina.com/health
curl -k https://yosemite.cantina.com/health

# Test ZooKeeper connectivity
telnet zookeeper-1.eng.cantina.com 2181
```

### Emergency Rollback
```bash
# Phase-specific rollback (example from Phase 4)
cp app-config.mjs.production.backup app-config.mjs
kubectl rollout undo deployment/platform-service -n production
```

## Development Workflow

### When Adding New Migration Documentation
1. Follow the established naming pattern: `phase-X-diagram.svg`, `migration-*.md`
2. Update the main `README.md` table of contents
3. Cross-reference with related phase documentation
4. Include rollback procedures for any new operational changes

### Critical Files That Affect Service Operations
From `git-links-reference.md`, focus on these high-priority files:
- **ZooKeeper configs**: `vandenberg-config.json`, `config.json`
- **Token services**: `constants.mjs`, `app-config.mjs` 
- **Automation**: `automation-test.sh` scripts

### Phase 4 (Production) Special Handling
Phase 4 is the **highest risk phase** requiring:
- Real-time monitoring during deployment
- 5-minute emergency rollback capability
- Authentication success rate >99%
- Service discovery cannot fail

## Repository Structure Context

### Migration Phases Map to Files
- **Phase 0-1**: Infrastructure setup → `domain-migration-plan.md`
- **Phase 2-3**: Environment migration → `step-by-step-guide.md` sections
- **Phase 4**: Critical production → `rollback-procedures.md` emergency sections
- **Phase 5-6**: Cleanup → `migration-checklist.md` final tasks

### Documentation Dependencies
- `git-links-reference.md` contains 28+ specific GitHub URLs requiring updates
- `github-org-repos.md` provides repository inventory across 4 organizations
- SVG diagrams show service dependencies and must be updated if architecture changes

## Working with This Migration

### Before Making Changes
1. Review the appropriate phase diagram to understand service dependencies
2. Check rollback procedures for the affected phase
3. Verify changes align with the incremental rollout strategy

### When Updating Existing Documents
- Maintain the phase-based organization
- Update cross-references in `README.md` if new files are added
- Preserve the risk assessment and timeline information

### Emergency Situations
- **Service failures**: Reference `rollback-procedures.md` immediately
- **Phase 4 issues**: Follow <5 minute emergency rollback procedures
- **DNS problems**: Contact domain administrator per emergency contacts section

This is a **one-time critical migration project** with specific start and end dates. The documentation serves as both execution guide and historical record of the migration approach.