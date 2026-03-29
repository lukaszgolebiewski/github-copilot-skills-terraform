# AGENTS.md – GitHub Copilot Skills for Terraform Azure Blueprint

Context and instructions for AI coding agents working with this repository.

## Repository Overview

This repo now contains a **Terraform Azure blueprint** plus GitHub Copilot agents and skills. Use it as the source of truth for the blueprint under `blueprint/`, environment settings under `settings/`, and Copilot customizations under `.github/`.

## Repository Structure (current)
```
.github/
├── agents/                         # Copilot agent definitions
├── skills/                         # Reusable skills
├── copilot-instructions.md         # Global Copilot instructions
.vscode/mcp.json                    # MCP server configuration
AGENTS.md                           # This file
README.md                           # Repo overview
blueprint/                          # Terraform blueprint source
├── bootstrap/                      # Foundational RG, storage, Key Vault, DevOps setup
├── core/                           # Subscription/core resources (RGs, KV, Log Analytics, identities)
├── storage/                        # Storage account workloads
├── adf/                            # Azure Data Factory
├── batch-account/                  # Batch accounts + schedules
├── linux-web-app/                  # App Service (Linux) workloads
├── ms-foundry/                     # Foundry-related IaC
├── modules/                        # Reusable modules (adf, storage, app services, batch, kv, la, search, foundry, appinsights)
├── helpers/, local-pipeline/, ml-studio/, templates/  # Supporting pieces
settings/                           # tfsettings per environment (dev/snd/test/prod) plus README
```

## Using the Blueprint
- Run Terraform from the appropriate `blueprint/<workload>` folder; import and tfplan files are present for reference.
- Per-workload `import.tfsettings-*.tf` files define tfsettings inputs; environment overlays live in `settings/<env>/`.
- Keep state remote, encrypted, and locked (Azure storage + blob lease).

## Naming & Tagging (apply to all resources)
Each and every module contain its own naming convention for specific resources.
Examples: rg-pada-iadwh-cross-selling-snd, id-pada-iadwh-cross-selling-snd-lwa, bapadaiadwhadfsnd

## Security Requirements
1) Prefer managed identity; 2) OIDC/federated creds for CI/CD; 3) Service principal w/ cert; 4) SP w/ secret (last resort).
- No hardcoded secrets; use Key Vault data sources.
- Enforce encryption at rest + TLS 1.2+, private endpoints for PaaS.

## Terraform Practices
- `terraform init` → `terraform validate` → `terraform fmt -recursive` → `terraform plan -out=tfplan` → `terraform apply tfplan`.
- Pin provider versions (`~>`), follow HashiCorp style guide.
- Use Azure Verified Modules as reference patterns; ensure modules include examples, variables with descriptions, outputs, and README.

## Agents (in .github/agents)
- `terraform-coordinator` (routing), `terraform-module-expert`, `terraform-security`, `azure-architecture-reviewer`, `terraform-provider-upgrade`.

## Skills (in .github/skills)
- `azure-architecture-review`, `azure-verified-modules`, `github-actions-terraform`, `terraform-provider-upgrade`, `terraform-security-scan`.

## MCP Tooling
- HashiCorp Terraform MCP: `search_modules`, `get_module_details`, `search_providers`, `get_provider_details`.
- Azure MCP: `azureterraformbestpractices get` **before any Azure TF code**, `get_bestpractices` (general/azurefunctions/static-web-app/coding-agent), `azure_resources` for ARG queries.

## Common Tasks
- Add/modify resources in the relevant `blueprint/<workload>/` folder; include required tags and naming.
- For new modules, place code under `blueprint/modules/<module>/` and include an `examples/` subdir if expanded.
- Use environment tfsettings from `settings/<env>/` when planning/applying per environment.

### Naming Conventions

**Resources follow this pattern:**

Examples:
- `rg-pada-iadwh-adf-snd` (Resource Group)
- `sapadaiadwhadfadlssnd` (Storage Account)
- `kv-pada-iadwh-adf-snd` (Key Vault)
- `lwa-pada-iadwh-cross-selling-snd` (Web App)

### Required Tags
No tags required yes

## Security Requirements

### Authentication
1. **Managed Identity** - Preferred for Azure-hosted workloads
2. **OIDC/Federated Credentials** - Required for CI/CD pipelines
3. **Service Principal** - Only when above options unavailable

### Secrets Management
- NEVER hardcode credentials in Terraform files
- Use Azure Key Vault for all secrets
- Reference secrets via data sources:
```hcl
data "azurerm_key_vault_secret" "example" {
  name         = "secret-name"
  key_vault_id = data.azurerm_key_vault.main.id
}
```

### Encryption
- Enable encryption at rest for all storage
- Require TLS 1.2+ for all connections
- Use private endpoints for PaaS services

## Terraform Best Practices

### State Management
- Remote state stored in Azure Storage Account
- State encryption enabled
- State locking via Azure Blob lease

### Provider Configuration
```hcl
terraform {
  required_version = ">= 1.5.0"
  
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
  
  backend "azurerm" {
    # Backend config in backend.tfvars
  }
}
```

### Module Usage
- Prefer Azure Verified Modules (AVM) as reference patterns, not as direct dependencies
- When creating custom modules, follow the proper structure:
  - Module code in root of module directory
  - Examples in `examples/` subdirectory within the module
  - Each example must include:
    - Complete working Terraform configuration
    - `terraform.tfvars.example` with sample values
    - `README.md` with usage instructions
- Always pin provider versions using pessimistic constraints (~>)
- Document all variables with descriptions and validation rules

## Workflow Commands

### Local Development
```bash
# Initialize
terraform init

# Validate
terraform validate

# Format
terraform fmt -recursive

# Plan
terraform plan -out=tfplan

# Apply (with saved plan)
terraform apply tfplan
```

## Common Tasks
To be added

## Troubleshooting
To be added

## MCP Server Integration

This repository uses the HashiCorp Terraform MCP server for enhanced tooling:

- **search_modules** - Find Terraform modules
- **get_module_details** - Get module documentation
- **search_providers** - Find provider resources
- **get_provider_details** - Get resource documentation

Configure in `.vscode/mcp.json` for VS Code integration.

## Agent Architecture

This repository includes five specialized agents:

### Terraform Coordinator
**Purpose:** Central routing agent for handoffs between specialist agents.

**Responsibilities:**
- Routes security review requests to `terraform-security`
- Routes implementation requests to `terraform-module-expert`
- Routes architecture reviews to `azure-architecture-reviewer`
- Tracks handoff state and maintains a single canonical path for review/implementation cycles
- Avoids performing specialist tasks; delegates to appropriate agents

**When to use:** Use as the central handoff point when one agent needs to invoke another (e.g., module expert requesting security review, security agent requesting implementation).

### Terraform Module Expert
**Purpose:** Discovers, evaluates, and implements Azure Terraform modules.

**Responsibilities:**
- Discover modules from Azure Verified Modules and Terraform Registry
- Evaluate modules for quality, security, and fit
- Implement modules with best practices
- Create custom modules following Azure standards and AVM patterns
- Maintain module versions and handle upgrades

**Handoffs:** Routes security review requests to `terraform-coordinator`, architecture reviews to `azure-architecture-reviewer`.

### Terraform Security
**Purpose:** Analyzes Terraform configurations for security vulnerabilities and compliance.

**Responsibilities:**
- Scan configurations for security vulnerabilities and misconfigurations
- Enforce compliance with security policies and frameworks
- Detect issues that could lead to security incidents
- Provide remediation guidance with secure code examples

**Handoffs:** Routes implementation requests to `terraform-coordinator`.

**Compliance frameworks checked:**
- Azure Security Benchmark
- CIS Azure Foundations Benchmark v2.0
- SOC 2 Type II requirements
- PCI DSS (if applicable)
- HIPAA (if applicable)

### Azure Architecture Reviewer
**Purpose:** Reviews Terraform Azure configurations against Microsoft Cloud Adoption Framework (CAF) and Azure Well-Architected Framework (WAF) before deployment.

**Responsibilities:**
- Review Terraform code for CAF compliance (network topology, naming, tagging, organization)
- Analyze Terraform configurations for WAF alignment across all five pillars:
  - Reliability (availability zones, redundancy, SLA)
  - Security (NSGs, encryption, zero trust)
  - Cost Optimization (right-sizing, shared resources)
  - Operational Excellence (monitoring, IaC, tagging)
  - Performance Efficiency (scalability, throughput)
- Provide actionable recommendations with code examples
- Generate comprehensive compliance reports with scores

**MCP Tools Used:**
1. `azureterraformbestpractices get` - ALWAYS called first
2. `mcp_azure_mcp_documentation search` - Search CAF and WAF documentation
   - Queries Cloud Adoption Framework patterns
   - Queries Well-Architected Framework for each pillar
   - Queries service-specific best practices

**When to use:** Before deploying infrastructure, during architecture reviews, for compliance audits, or when optimizing existing Terraform code.

**Example Workflow:**
```bash
# Step 1: Get Terraform best practices
azureterraformbestpractices get

# Step 2: Search CAF documentation
mcp_azure_mcp_documentation search
  query: "Cloud Adoption Framework hub spoke network topology"
  
# Step 3: Search WAF documentation (per pillar)
mcp_azure_mcp_documentation search
  query: "Well-Architected Framework reliability availability zones"
  
# Step 4: Generate report with compliance scores and recommendations
```

**Output:** Detailed compliance report with:
- CAF compliance score (0-100%)
- WAF pillar scores (0-100% each)
- Prioritized recommendations (High/Medium/Low)
- Code examples for fixes
- Links to Microsoft documentation

### Terraform Provider Upgrade
**Purpose:** Safely upgrade Terraform providers with automatic resource migration, breaking change detection, and state management.

**Responsibilities:**
- Analyze current provider versions and identify latest stable versions
- Research breaking changes using provider upgrade guides
- Automatically migrate removed resources using `moved` blocks
- Validate argument changes between old and new resources
- Check for default value changes that may affect behavior
- Apply code migrations with proper state management
- Generate comprehensive upgrade documentation

**Handoffs:** Routes security reviews to `terraform-security`, module updates to `terraform-module-expert`, complex workflows to `terraform-coordinator`.

**MCP Tools Used:**
1. `get_latest_provider_version(namespace, name)` - Get latest provider version from Terraform Registry
2. `resolveProviderDocID(...)` - Find provider documentation IDs
3. `getProviderDocs(providerDocID)` - Fetch upgrade guides, changelogs, resource documentation
4. `azureterraformbestpractices get` - Get Azure Terraform best practices

**When to use:** When upgrading Terraform provider versions (especially major versions), handling removed/deprecated resources, migrating between resource types, or performing provider maintenance.

**Key Principles:**
- **Action over documentation** - Apply code changes instead of listing manual steps
- **Moved blocks preferred** - Use `moved` blocks for automatic state migration
- **Validate arguments** - Always check official docs for argument changes
- **Check defaults** - Compare default values between old and new resources
- **Update references** - Find and update all dependent resources
- **Pipeline validation** - User validates through CI/CD, not local commands

**Example Workflow:**
```bash
# Step 1: Get latest version
get_latest_provider_version(namespace="hashicorp", name="azurerm")

# Step 2: Get upgrade guide
resolveProviderDocID(...serviceSlug="4.0-upgrade-guide", providerDataType="guides"...)
getProviderDocs(providerDocID="<id>")

# Step 3: Search for removed resources in codebase
grep -r "azurerm_sql_server" --include="*.tf"

# Step 4: Get old resource docs
resolveProviderDocID(...serviceSlug="sql_server", providerVersion="3.117.1"...)
getProviderDocs(providerDocID="<id>")

# Step 5: Get new resource docs
resolveProviderDocID(...serviceSlug="mssql_server", providerVersion="latest"...)
getProviderDocs(providerDocID="<id>")

# Step 6: Apply migrations with moved blocks
# Step 7: Update dependent resources
# Step 8: Document changes in TERRAFORM_UPGRADE_BREAKING_CHANGES.md
```

**Output:** 
- Updated Terraform files with new provider version
- Migrated resource types with `moved` blocks
- Updated argument references and attribute changes
- `TERRAFORM_UPGRADE_BREAKING_CHANGES.md` documentation with:
  - Version change summary
  - Breaking changes handled (with argument mappings)
  - Default value analysis
  - Modified files list
  - Pipeline-based next steps
  - Official documentation links
