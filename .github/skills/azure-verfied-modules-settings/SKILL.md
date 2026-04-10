---
name: azure-verfied-modules-settings
description: Scaffold for adding and updating settings for Azure Verified Module–inspired resources across environments (dev/snd/test/prod) in this repo.
metadata:
  author: github-copilot-skills-terraform
  version: "0.2.0"
  category: terraform-azure-settings
---

# Azure Verified Modules Settings Skill

Use this skill to add or update settings for resources that follow Azure Verified Modules (AVM) patterns in this repository's `settings/<env>/` folders. It guides how to:
- gather required context from existing environments
- align with AVM patterns learned via `azure-verified-modules` reference skill
- place changes in the right per-environment folders and core/global files
- verify private endpoint IP allocations before committing changes
- keep naming, tagging, identity, networking, and diagnostics consistent across environments

**Trigger phrases:** "add AVM-style settings", "update settings/<env>", "add storage settings", "add key vault settings", "private endpoint IP selection", "align with AVM patterns", "mirroring configuration".

**Do not use for:** writing module code (use module skills), CI/CD fixes (use GitHub Actions Terraform skill), or provider upgrades (use provider upgrade skill).

## When to Use
- Adding a new instance of an existing module/resource (e.g., new storage account, key vault, app insights) in an environment.
- Updating settings (e.g., SKU, networking, tags, diagnostics) for existing resources.
- Preparing per-environment configuration aligned to AVM patterns.
- Ensuring cross-environment consistency for settings already present in another env.

## Prerequisites
- Read `azure-verified-modules` skill for AVM patterns and validation approaches.
- Understand repo layout: `settings/<env>/` (dev/snd/test/prod) with per-service folders plus shared `core/` and `global/` configs.
- Follow Terraform best practices: validate → plan → apply; no hardcoded secrets; required tags; naming `{type}-{workload}-{env}-{region}-{instance}`.
- Check whether the resource already exists in another environment and mirror proven settings before diverging.

## Environment & Folder Map (settings/<env>/)
- `core/`: shared foundations. Key files possibly touched:
  - `ad-groups.tf`, `app-ins.tf`, `kv.tf`, `main.tf`, `resource-groups.tf`, `user-managed-identities.tf`, `la.tf`, `app-ids.tf`
- `global/`: project- and backend-level locals, feature toggles, devops pipelines, tool settings.
- Service folders: `acr/`, `adf/`, `ai-search/`, `batch-account/`, `bootstrap/`, `linux-web-app/`, `ml-studio/`, `ms-foundry/`, `storage/`, etc. Each holds service-specific `*.tf`, imports, variables, outputs, and plan artifacts.
- Bootstrap vs core: bootstrap = shared prerequisites (RGs, KV, MIs) reused by many services; core = shared but sometimes environment-shaped resources (Log Analytics, App Insights, AD groups). Extend, do not duplicate.
- Golden paths span layers (storage/adf/kv/mi). Always consider upstream dependencies (RG, KV, MI, subnets, diagnostics) before editing a service folder.

## Standard Workflow (fast path)
1) **Scope & inputs**: identify resource type, environment(s), RG, subnet/PE needs, identity, diagnostics, SKU, tags.

2) **Multi-layer context gathering** (CRITICAL — NEVER skip):
   - **Core layer**: Check `settings/<env>/core/` for RG, MI, KV, diagnostics (LA/App Insights), AD groups, networking
   - **Global layer**: Check `settings/<env>/global/` for project settings, feature flags, backend config
   - **ALL dependency layers**: Check EVERY related service folder:
     - `storage/` — storage accounts, containers, file shares
     - `acr/` — container registries
     - `adf/` — data integration patterns
     - `ai-search/`, `batch-account/`, `ml-studio/`, `ms-foundry/`, etc.
   - **Target service layer**: Finally check the target service folder itself
   - **WHY**: Services are configured at different levels. A web app may need RG (core), MI (core), storage (storage/), ACR image (acr/), App Insights (core). Missing ANY layer = incomplete config.

3) **Cross-env mirror**: Check the SAME resource type across ALL layers in another environment (e.g., dev → snd); note intentional differences vs. patterns to copy.

4) **Plan placements**:
  - Shared prerequisites (RG/MI/KV/App Insights/AD groups) → `settings/<env>/core/*.tf` (or bootstrap if truly global).
  - Service-specific settings → `settings/<env>/<service>/`.
  - Never duplicate RG/KV/MI; extend locals/variables instead.

5) **Networking/PE IPs**: check current subnet allocations; pick next free IP; align DNS zone links; enforce TLS 1.2+, private endpoints when supported.

6) **Security & governance**: ensure managed identity over secrets; apply required tags; lock down public access; enable diagnostics to Log Analytics/App Insights.

7) **Variables/outputs**: add/update variable defaults and outputs if new settings are consumed elsewhere; keep descriptions mandatory.

8) **Validation**: `terraform fmt -recursive` (scope to touched folder), `terraform validate`, and (when requested) env-scoped `terraform plan`.

9) **Docs**: note deviations from AVM defaults in comments or README within the service folder.

## Checklists
**Before editing (Multi-Layer Verification)**
- [ ] Identify resource type + environment(s)
- [ ] Check CORE layer: `settings/<env>/core/` (RG, MI, KV, LA, App Insights, AD groups, networking)
- [ ] Check GLOBAL layer: `settings/<env>/global/` (project settings, feature flags)
- [ ] Check ALL dependency service layers (storage, acr, adf, ai-search, batch, ml-studio, ms-foundry, etc.)
- [ ] Check TARGET service layer: `settings/<env>/<target-service>/`
- [ ] Cross-env mirror: Repeat ALL layer checks in reference environment
- [ ] Confirm subnet and PE IP availability

**While editing**
- [ ] Place shared bits in `core/` (or bootstrap), service bits in service folder
- [ ] Keep naming `{type}-{workload}-{env}-{region}-{instance}`
- [ ] Add/keep required tags on all resources
- [ ] Prefer managed identity; avoid secrets
- [ ] Enforce private access + TLS1.2 where applicable
- [ ] Wire diagnostics to Log Analytics/App Insights

**After editing**
- [ ] Update variables/outputs with descriptions + validations if needed
- [ ] Run `terraform fmt -recursive` in the touched folder
- [ ] Run `terraform validate`
- [ ] (If asked) run `terraform plan` for the environment
- [ ] Document intentional deviations from AVM defaults

## References
- AVM reference skill: `.github/skills/azure-verified-modules/`
- Repo structure: `settings/<env>/...` (dev/snd/test/prod)
- Best practices: required tags (environment, project, owner, cost-center, managed-by); managed identity preferred; private endpoints for PaaS; TLS 1.2+; diagnostics enabled.

## Sub-skill
- `referances/azure-verfied-modules-settings-new-resource`: focused guide to add new instances of existing module-based resources.
