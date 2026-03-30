# Repository Overrides for AVM Reference Skill

Scope: Use these overrides when applying the `azure-verified-modules` skill in this repository. These rules take precedence over generic AVM guidance and the `terraform-module-expert` agent defaults where they differ.

## Priority & Alignment
- Treat the **current implementation and repository patterns as the source of truth**. Only use AVM patterns for inspiration, not enforcement, when they conflict with existing code.
- Resolve conflicts in this order: 1) Current module implementation, 2) This override file, 3) Skill guidance, 4) AVM references.
- When using the `terraform-module-expert` agent, **skip provider/version lookups and terraform required blocks** per the rules below.

## Required Repository-Specific Rules
- **Skip terraform requirements and provider version blocks** in outputs/examples for this skill.
- **Skip tags** unless the existing module already defines them; do not add new tags by default.
- **Resource naming:** never use `this`; use the resource type or existing module naming pattern instead.
- **Outputs:** always include the primary resource output (e.g., the main resource object/id) in module outputs.
- **Private endpoints:** keep `private-endpoint.tf` with the private endpoint definition under the `accounts` folder (do not relocate/merge).
- **Access control:** always include an `access.tf` that contains the groups general assignment.

## Consistency Checks
- Before adding new guidance, verify it does not contradict these overrides or the current module layouts.
- If an existing module diverges from AVM best practices but follows repository patterns, **preserve the current pattern** unless explicitly asked to refactor.
