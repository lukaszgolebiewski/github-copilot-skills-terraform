---
name: azure-verfied-modules-settings
description: Scaffold for adding and updating settings for Azure Verified Module–inspired resources across environments (dev/snd/test/prod) in this repo.
metadata:
  author: github-copilot-skills-terraform
  version: "0.1.0"
  category: terraform-azure-settings
---

# Azure Verified Modules Settings Skill

Use this skill to add or update settings for resources that follow Azure Verified Modules (AVM) patterns in this repository's `settings/<env>/` folders. It guides how to:
- gather required context from existing environments
- align with AVM patterns learned via `azure-verified-modules` reference skill
- place changes in the right per-environment folders and core/global files
- verify private endpoint IP allocations before committing changes

## When to Use
- Adding a new instance of an existing module/resource (e.g., new storage account, key vault, app insights) in an environment.
- Updating settings (e.g., SKU, networking, tags, diagnostics) for existing resources.
- Preparing per-environment configuration aligned to AVM patterns.

## Prerequisites
- Read `azure-verified-modules` skill for AVM patterns and validation approaches.
- Understand repo layout: `settings/<env>/` (dev/snd/test/prod) with per-service folders plus shared `core/` and `global/` configs.
- Follow Terraform best practices: validate → plan → apply; no hardcoded secrets; required tags; naming `{type}-{workload}-{env}-{region}-{instance}`.

## Environment & Folder Map (settings/<env>/)
- `core/`: shared foundations. Key files possibly touched:
  - `ad-groups.tf`, `app-ins.tf`, `kv.tf`, `main.tf`, `resource-groups.tf`, `user-managed-identities.tf`, `la.tf`, `app-ids.tf`
- `global/`: project- and backend-level locals, feature toggles, devops pipelines, tool settings.
- Service folders: `acr/`, `adf/`, `ai-search/`, `batch-account/`, `bootstrap/`, `linux-web-app/`, `ml-studio/`, `ms-foundry/`, `storage/`, etc. Each holds service-specific `*.tf`, imports, variables, outputs, and plan artifacts.

## Standard Workflow
1) **Scope & inputs**: identify resource type, environment(s), RG, network requirements, identity, diagnostics, tags.
2) **Discover existing state**: inspect corresponding `settings/<env>/{core,global,service}/` files; reuse RGs, identities, KV secrets where applicable.
3) **AVM pattern alignment**: apply security defaults (TLS1.2, private access, managed identity, diagnostics, tagging) referencing `azure-verified-modules` skill and references.
4) **Plan placements**:
   - Shared additions (RG/MI/KV/App Insights/ad groups) go to `settings/<env>/core/*.tf` as needed.
   - Service-specific config goes under `settings/<env>/<service>/`.
   - Avoid duplicating RG/KV/MI entries if already present; extend instead.
5) **Private endpoints/IPs**: scan existing PEs in env to pick the next free IP; if uncertain, query Azure (via MCP) or inspect subnet IP usage; double-check before committing.
6) **Validation**: ensure variables/outputs updated; run `terraform fmt` and `terraform validate` in target folder; plan per env if required.

## References
- AVM reference skill: `.github/skills/azure-verified-modules/`
- Repo structure: `settings/<env>/...` (dev/snd/test/prod)
- Best practices: required tags (environment, project, owner, cost-center, managed-by); managed identity preferred; private endpoints for PaaS.

## Sub-skill
- `referances/azure-verfied-modules-settings-new-resource`: focused guide to add new instances of existing module-based resources.
