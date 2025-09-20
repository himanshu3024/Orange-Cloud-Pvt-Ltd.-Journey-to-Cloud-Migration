# Azure Migration Lab: Orange Cloud Pvt Ltd
*A practical exploration of enterprise cloud transformation*

```
╔════════════════════════════════════════════════════════════════════════════╗
║                                                                            ║
║   "The best way to predict the future is to build it."                    ║
║                                                    - Peter Drucker        ║
║                                                                            ║
║   This is my journey from localhost to cloud host.                        ║
║                                                                            ║
╚════════════════════════════════════════════════════════════════════════════╝
```

---

## The Story

I'm a cloud computing student. While my classmates were studying cloud architecture from textbooks, I decided to build one.

This isn't a tutorial. It's not a guide. It's documentation of how I migrated a fictional company's infrastructure from physical servers to Azure cloud services. Every error message, every successful sync, every late-night troubleshooting session taught me something textbooks couldn't.

**Organization:** Orange Cloud Pvt Ltd (fictional)  
**Employees:** 25  
**Challenge:** Complete infrastructure migration  
**Timeline:** 4 weeks  
**Budget:** Student constraints  
**Experience Level:** None. That was the point.

---

## Architecture Evolution

### Before: Traditional Infrastructure
```
┌──────────────────────────────────────────────────────────┐
│                     ON-PREMISES                          │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  [Domain Controller]                                     │
│       ↓                                                  │
│  orange.local                                            │
│       ├── HR (3 users)                                   │
│       ├── Finance (4 users)                              │
│       ├── IT (6 users)                                   │
│       ├── Sales (7 users)                                │
│       └── Marketing (5 users)                            │
│                                                          │
│  [File Server]                                           │
│       └── 5 departmental shares                          │
│           └── NTFS permissions                           │
│                                                          │
│  [Web Server]                                            │
│       └── IIS intranet portal                            │
│                                                          │
│  Single point of failure. No remote access.              │
│  Recovery time: 72 hours.                                │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### After: Cloud Architecture
```
┌──────────────────────────────────────────────────────────┐
│                      AZURE CLOUD                         │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  [Azure Active Directory]                                │
│       ↓ Synchronized                                     │
│  25 cloud identities                                     │
│       └── Seamless SSO                                   │
│                                                          │
│  [Azure Files]                                           │
│       └── 5 cloud shares                                 │
│           └── RBAC + preserved permissions               │
│                                                          │
│  [Azure Virtual Machine]                                 │
│       └── Migrated IIS application                       │
│           └── Public endpoint                            │
│                                                          │
│  Globally accessible. Automatic backups.                 │
│  Recovery time: 4 hours.                                 │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Technical Implementation

### Phase 1: Foundation [Week 1]

Built the entire on-premises environment from scratch in VMware. This wasn't about clicking through wizards - it was about understanding why enterprises structure their systems this way.

**Key Decisions:**
- Windows Server 2025 (latest evaluation)
- Single forest, single domain design
- Organizational Units mirroring real corporate structure
- Security groups for role-based access

**What Went Wrong:**
- First domain promotion failed - DNS misconfiguration
- Learned: Always configure static IP before DCPROMO
- Time lost: 3 hours. Lesson value: Priceless

**Evidence Captured:**
- Active Directory structure
- User creation automation script
- Permission matrix documentation

### Phase 2: Identity Bridge [Week 2]

Connected on-premises AD to Azure AD. This is where theory met reality.

**Technical Components:**
- Azure AD tenant provisioning
- Azure AD Connect installation
- Password Hash Synchronization (chose over Pass-through for simplicity)
- UPN suffix alignment

**The 2 AM Moment:**
- Sync wasn't working. No errors, just... nothing
- Discovery: Default UPN suffix didn't match verified domain
- Solution: PowerShell script to update all 25 users
- Feeling when it finally synced: Indescribable

```powershell
# The script that saved my project at 2 AM
$LocalUsers = Get-ADUser -Filter * -SearchBase "OU=OrangeCloud,DC=orange,DC=local"
foreach ($User in $LocalUsers) {
    Set-ADUser -Identity $User -UserPrincipalName "$($User.SamAccountName)@orangecloud.com"
}
```

**Evidence Captured:**
- Azure AD Connect configuration
- Successful sync logs
- User objects in Azure portal

### Phase 3: Data Migration [Week 3]

Moving files isn't hard. Maintaining permissions, ensuring zero data loss, and keeping department isolation? That's where learning happens.

**Migration Strategy:**
- Azure File Sync for hybrid scenario
- Maintained NTFS ACLs through migration
- Created storage account with LRS redundancy (cost optimization)
- Implemented private endpoints for security

**Reality Check:**
- Azure File Sync agent installation: 30 minutes
- Getting it to actually sync: 6 hours
- Problem: Firewall rules, proxy settings, TLS versions
- Solution: Systematic troubleshooting, lots of documentation reading

**Application Migration:**
- Lifted and shifted IIS to Azure VM
- Chose B2s instance (student budget reality)
- Configured NSG rules for HTTP/HTTPS only
- Learned about Azure Bastion the hard way (after exposing RDP briefly)

**Evidence Captured:**
- File Sync health dashboard
- Storage account metrics
- VM deployment confirmation
- Network topology diagram

### Phase 4: Validation [Week 4]

Testing isn't glamorous. But it's where you prove the migration worked.

**Test Scenarios Executed:**
```
[✓] All 25 users can authenticate to Azure AD
[✓] Department files accessible only to correct groups
[✓] No orphaned SIDs in permissions
[✓] IIS application responds on public endpoint
[✓] Disaster recovery runbook validated
[✓] Cost analysis under $40/month
```

**Performance Metrics:**
- Authentication time: On-prem (instant) → Azure (1.2 seconds)
- File access: LAN (10ms) → Cloud (45ms)
- Acceptable for business use case

**Evidence Captured:**
- Test execution logs
- Performance baselines
- Cost analysis breakdown
- Final architecture diagram

---

## Honest Learnings

### What Textbooks Don't Tell You

**DNS is everything**  
Spent 6 hours debugging. The problem? A single missing DNS forwarder entry. In production, this would've been caught by change management. In my lab, it was a masterclass in troubleshooting.

**Permissions are harder than they look**  
NTFS to Azure RBAC isn't 1:1. Had to create custom mapping logic. Documented every decision for future reference.

**The cloud isn't magical**  
It's someone else's computer with better redundancy. Understanding this removes the mystique and reveals the engineering.

**Free tiers have limits**  
Hit Azure AD Connect sync limits with my testing. Learned to be strategic about sync cycles.

### Technical Skills Acquired

```yaml
Infrastructure:
  Hypervisor: VMware configuration, resource allocation
  Windows Server: AD DS, DNS, DHCP, File Services, IIS
  Networking: Subnets, routing, firewall rules
  
Cloud Platform:
  Identity: Azure AD, hybrid authentication, conditional access basics
  Storage: Blob, Files, access tiers, lifecycle management
  Compute: VM sizing, availability sets, managed disks
  Networking: VNets, NSGs, private endpoints, DNS zones
  
Automation:
  PowerShell: AD cmdlets, Azure modules, bulk operations
  Azure CLI: Resource deployment, query operations
  ARM Templates: Basic infrastructure as code
  
Professional:
  Documentation: Technical writing, diagram creation
  Troubleshooting: Systematic approach, log analysis
  Project Management: Phase planning, risk assessment
  Cost Management: Optimization, budget tracking
```

---

## Reproduction Instructions

### Prerequisites

**Hardware**
- x64 processor with virtualization support
- 16GB RAM (8GB minimum, expect performance impact)
- 100GB available storage
- Stable internet connection

**Software**
- VMware Workstation 17 Player (free)
- Windows Server 2025 Evaluation
- Azure subscription (student/free tier)

**Knowledge**
- Basic Windows Server navigation
- Command line familiarity
- Patience for troubleshooting

### Repository Structure

```
orange-cloud-migration/
│
├── /documentation
│   ├── 00-prerequisites.md
│   ├── 01-infrastructure-setup.md
│   ├── 02-identity-configuration.md
│   ├── 03-migration-execution.md
│   ├── 04-validation-procedures.md
│   └── 05-lessons-learned.md
│
├── /scripts
│   ├── New-ADTestEnvironment.ps1
│   ├── Sync-AzureIdentity.ps1
│   ├── Test-MigrationSuccess.ps1
│   └── Remove-LabEnvironment.ps1
│
├── /evidence
│   ├── phase1-screenshots/
│   ├── phase2-screenshots/
│   ├── phase3-screenshots/
│   ├── phase4-screenshots/
│   └── architecture-diagrams/
│
├── /configs
│   ├── users.csv
│   ├── departments.json
│   └── permissions-matrix.xlsx
│
└── MIGRATION-REPORT.pdf
```

### Quick Start

```bash
git clone https://github.com/yourusername/orange-cloud-migration.git
cd orange-cloud-migration

# Review prerequisites
cat documentation/00-prerequisites.md

# Begin infrastructure setup
powershell -ExecutionPolicy Bypass -File scripts/New-ADTestEnvironment.ps1
```

---

## Project Impact Analysis

### Quantifiable Outcomes

| Metric | Traditional | Cloud | Improvement |
|--------|------------|-------|-------------|
| Recovery Time | 72 hours | 4 hours | 94.4% reduction |
| Availability | ~95% | 99.9% | 4.9% increase |
| Remote Access | None | Full | Enabled |
| Scaling Time | Weeks | Minutes | ~99% reduction |
| Hardware Cost | $15,000/year | $0 | Eliminated |
| Operational Cost | Variable | $456/year | Predictable |

### Business Translation

This project demonstrates understanding of:
- Risk mitigation through redundancy
- Capital to operational expense transition
- Business continuity planning
- Change management considerations
- User experience preservation

---

## Critical Evidence for Portfolio

### Must-Capture Screenshots

**Phase 1: Foundation**
1. Active Directory Users and Computers - Full OU structure
2. PowerShell ISE - User creation script execution
3. File Explorer - Share permissions configuration
4. IIS Manager - Default website configuration

**Phase 2: Identity**
1. Azure AD Connect - Configuration wizard completion
2. Azure Portal - Synchronized users view
3. Event Viewer - Successful sync event
4. PowerShell - UPN update script

**Phase 3: Migration**
1. Azure Portal - Resource group overview
2. Storage Account - File shares created
3. File Sync - Sync group health
4. Virtual Machine - Deployment success

**Phase 4: Validation**
1. Test Results - Spreadsheet with all validations
2. Azure Monitor - Performance metrics
3. Cost Analysis - Monthly breakdown
4. Architecture - Final state diagram

---

## For Recruiters

### Why This Matters

I didn't wait for an internship to learn enterprise IT. I created my own.

**Initiative Demonstrated:**
- Self-directed learning path
- Problem-solving without supervision
- Documentation discipline
- Cost consciousness
- Business perspective

**Technical Competence:**
- Full stack infrastructure knowledge
- Cloud platform proficiency
- Automation capabilities
- Security awareness
- Troubleshooting methodology

**Professional Skills:**
- Project planning and execution
- Technical documentation
- Risk assessment
- Resource optimization
- Continuous learning mindset

### The Difference

Most students know theory. I know what happens when DNS breaks at 2 AM.

---

## Current Status & Next Steps

**Project Status:** Complete and documented

**Currently Exploring:**
- Terraform for infrastructure as code
- Kubernetes deployment on AKS
- Azure DevOps pipeline integration

**Certification Path:**
- Preparing for AZ-104 (Azure Administrator)
- Target: Q4 2025

**Seeking:**
- Cloud engineering internship
- Junior DevOps position
- Infrastructure automation role

---

## Contact

**GitHub:** [himanshu3024](https://github.com/himanshu3024)
**LinkedIn:** [Profile](https://www.linkedin.com/in/himanshu-gandhi-891204160/)  
**Email:** [Mail](mailto:gandhi111000@hotmail.com)

---

## License

MIT License - Educational Project

This project was created for learning purposes. Use it to build your own understanding.

---

```
═══════════════════════════════════════════════════════════════════════
"In the depth of winter, I finally learned that there was in me 
an invincible summer." - Albert Camus

This project was my winter. Azure was my summer.
═══════════════════════════════════════════════════════════════════════
```
