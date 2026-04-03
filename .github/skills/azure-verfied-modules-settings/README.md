# azure-verfied-modules-settings

Skill scaffold to add/update environment-specific settings for resources following Azure Verified Module patterns.

## Structure
- `SKILL.md` — manifest and high-level usage
- `referances/azure-verfied-modules-settings-new-resource` — focused reference for adding a new instance of an existing module/resource

## Purpose
Provide a repeatable workflow to place config updates in the correct `settings/<env>/` folders (core/global/service), reuse shared assets (RGs, identities, KV), follow AVM-inspired security defaults, and verify private endpoint IP allocations before changes.

## Related Skill
See `.github/skills/azure-verified-modules/` for AVM patterns and references.
