# Repository Overrides for AVM Reference Skill

Scope: Use these overrides when applying the `azure-verified-modules` skill in this repository. These rules take precedence over generic AVM guidance and the `terraform-module-expert` agent defaults where they differ.

## Priority & Alignment
- Treat the **current implementation and repository patterns as the source of truth**. Only use AVM patterns for inspiration, not enforcement, when they conflict with existing code.
- Resolve conflicts in this order: 1) Current module implementation, 2) This override file, 3) Skill guidance, 4) AVM references.
- When using the `terraform-module-expert` agent, **skip provider/version lookups and terraform required blocks** per the rules below.

## Required Repository-Specific Rules
- **Skip terraform requirements and provider version blocks** in outputs/examples for this skill.
- **Do not create README.md or examples** when producing override snippets; keep guidance minimal and inline.
- **No tags guidance**: do not add or mention tags unless the existing module already defines them; avoid introducing new tag requirements in overrides.
- **Resource naming:** never use `this`; use the resource type or existing module naming pattern instead.
- **Access control scope:** `access.tf` must be created and only grant assignments to user-managed identities and Azure AD groups—no other principal types.
- **Managed identity reference:** ensure the main `main.tf` includes the user-managed identity reference consistent with other modules in the repo.
- **Outputs:** by default expose only the main resource, e.g.:
	```hcl
	output "ai_searches" {
		value = azurerm_search_service.ais_accounts
	}
	```
- **Private endpoints:** keep `private-endpoint.tf` under the `accounts` folder and use the minimal pattern below (omit DNS unless required):
	```hcl
	resource "azurerm_private_endpoint" "endpoint" {
		for_each            = var.public_access_enabled ? {} : { for k, v in var.ai_search_settings : k => v if v.sku_name != "free" }
		name                = "pe-${var.project_shortname}-ais-${each.key}-${var.env_shortname}"
		location            = var.region
		resource_group_name = var.project_data.pe_resource_group_name
		subnet_id           = var.project_data.subnet_id

		private_service_connection {
			name                           = "psc-${var.project_shortname}-ais-${each.key}-${var.env_shortname}"
			private_connection_resource_id = azurerm_search_service.ais_accounts[each.key].id
			is_manual_connection           = false
			subresource_names              = [each.value.ip_configuration.subresource_name]
		}

		ip_configuration {
			name               = each.value.ip_configuration.name
			private_ip_address = each.value.ip_configuration.private_ip_address
			subresource_name   = each.value.ip_configuration.subresource_name
			member_name        = each.value.ip_configuration.member_name
		}
	}
	```

## Consistency Checks
- Before adding new guidance, verify it does not contradict these overrides or the current module layouts.
- If an existing module diverges from AVM best practices but follows repository patterns, **preserve the current pattern** unless explicitly asked to refactor.
