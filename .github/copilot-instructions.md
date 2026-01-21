# Victoria 3 MOD "Daoyu Cheat" - AI Coding Agent Instructions

## Project Overview
This is a Victoria 3 game modification (MOD) that adds cheating/admin features to Paradox Interactive's Victoria 3 (v1.12). The MOD uses Paradox's proprietary scripting language with event-driven architecture.

## Architecture Patterns

### 1. **MOD File Structure & Namespacing**
- **Events** (`events/daoyu_*.txt`): Game events triggered by player actions
  - All events use `namespace = daoyu` prefix
  - Event IDs follow pattern: `daoyu.N` (e.g., `daoyu.1`, `daoyu.2`)
  - Example: [daoyu_events.txt](events/daoyu_events.txt#L1) - Main event definitions

- **Scripted Effects** (`common/scripted_effects/daoyu_*.txt`): Reusable action scripts
  - Parameterized effects use `$VARIABLE$` syntax (e.g., `$MULTIPLIER$`, `$REGION$`)
  - Example: [daoyu_effects.txt](common/scripted_effects/daoyu_effects.txt#L1-L50) - Pagination effects using state variables

- **Scripted Triggers** (`common/scripted_triggers/daoyu_*.txt`): Reusable condition scripts
  - Complex logic built from `or`/`and` blocks and `limit` conditions
  - Example: [daoyu_triggers.txt](common/scripted_triggers/daoyu_triggers.txt#L1-L50) - Pagination triggers

- **Modifiers** (`common/modifiers/daoyu_modifiers.txt`): Country/state stat modifiers
  - **Critical**: Define both positive and negative variants (e.g., `daoyu_country_legitimacy_base_add` + `daoyu_negative_country_legitimacy_base_add`)
  - Each modifier requires `icon` and numerical effects (see [common/modifiers](common/modifiers/daoyu_modifiers.txt))
  - Missing negative variants cause crashes when effects try to apply them

### 2. **Localization Strategy**
- Dual language support: Chinese (simplified) and English
- Files: [localization/english/](localization/english/daoyu_l_english.yml) and [localization/simp_chinese/](localization/simp_chinese/daoyu_l_simp_chinese.yml)
- **Key pattern**: Modifier descriptions reference vanilla game keys with `$` placeholders (e.g., `$state_working_adult_ratio_add$`)

### 3. **Effect Parameterization Pattern**
Effects accept parameters passed during invocation:
```
daoyu_add_or_remove_single_modifiers = {
    if = { limit = { has_global_variable = daoyu_remove_single_country_modifiers }
        if = { limit = { has_modifier = daoyu_$BUFFVAR$ } remove_modifier = daoyu_$BUFFVAR$ }
    }
}
# Invoked as: daoyu_add_or_remove_single_modifiers = { BUFFVAR = country_legitimacy_base_add MULTIPLIER = $MULTIPLIER$ }
```

## Critical Validation Rules (Game v1.12)

### Object Reference Validation
Before referencing game objects, verify they exist in vanilla v1.12:
- **Building types**: `building_gold_mine` ✓ but `building_gold_fields` ✗ (typo!)
- **Production methods**: `pm_steam_rail_transport` ✓ but `pm_trade_center_bureaucrat_ownership` ✗ (doesn't exist in v1.12)
- **Discrimination traits**: `anglophone`, `lusophone`, `hispanophone` ✗ (not valid in v1.12)
- **State regions**: Must have strategic region definition (e.g., `STATE_GAMBIA` crashes if strategic region missing)

**Error Detection**: Check [FIX_REPORT.md](FIX_REPORT.md) and `docs/crashes/logs/game.log` for patterns:
- `Invalid database object 'X'` - Reference doesn't exist
- `Invalid Production Method: X` - Production method not in v1.12
- `State Region is missing a strategic region` - Map definition incomplete

### Common Mistakes & Fixes
| Issue | Pattern | Fix |
|-------|---------|-----|
| Missing modifier variant | Only `daoyu_X` exists, no `daoyu_negative_X` | Add negative version to [common/modifiers/daoyu_modifiers.txt](common/modifiers/daoyu_modifiers.txt) |
| Invalid building type | `uilding_furniture_manufacturies` (typo) | Comment out or fix typo; verify in vanilla |
| Syntax errors | `remove_company = company_type:$COM$` | Replace with valid v1.12 syntax; check official wiki |
| Strategic region conflicts | `STATE_SENEGAL` hub in wrong state | Disable or fix region definitions |

## Debugging Workflow

1. **Check game crash logs**:
   ```
   docs/crashes/logs/game.log          # Primary log
   docs/crashes/logs/error.log         # Detailed errors
   docs/crashes/exception.txt          # Crash details
   ```

2. **Common error messages**:
   - `Script system error!` → Syntax or reference error (check context line)
   - `ACCESS_VIOLATION` → Invalid region/building definition conflict
   - `Event not found! EventID: daoyu.NNN` → Missing event definition

3. **Search strategy**: Use line numbers from logs to locate problematic scripts:
   ```
   Error location: events/daoyu_events.txt:1276
   → Search grep for "is_building_type" around line 1276
   ```

## Key Files Reference

- **MOD Configuration**: [descriptor.mod](descriptor.mod) (if exists) - Version compatibility
- **Main Events**: [events/daoyu_events.txt](events/daoyu_events.txt) - 14,359 lines, primary event logic
- **Effects Library**: [common/scripted_effects/daoyu_effects.txt](common/scripted_effects/daoyu_effects.txt) - 2,205 lines, core reusable actions
- **Triggers Library**: [common/scripted_triggers/daoyu_triggers.txt](common/scripted_triggers/daoyu_triggers.txt) - Pagination, modifier checks
- **Modifiers**: [common/modifiers/daoyu_modifiers.txt](common/modifiers/daoyu_modifiers.txt) - 222 lines, stat definitions
- **Building Effects**: [common/scripted_effects/daoyu_building_effect.txt](common/scripted_effects/daoyu_building_effect.txt) - Building-specific actions
- **Crash Analysis**: [FIX_REPORT.md](FIX_REPORT.md) - Troubleshooting guide

## Naming Conventions

- **Custom identifiers**: Prefix with `daoyu_` (e.g., `daoyu_on`, `daoyu_target_country`)
- **Negative modifiers**: Append `_negative_` (e.g., `daoyu_negative_country_legitimacy_base_add`)
- **Effect parameters**: Capitalized with `$` delimiters (e.g., `$BUFFVAR$`, `$REGION$`, `$MULTIPLIER$`)
- **Variables**: Lowercase with underscores (e.g., `daoyu_page_01`, `daoyu_count`)

## Testing & Validation

- **Before committing**: Verify game loads without crashes → Check logs for script errors → Test affected features in-game
- **Reference check**: All vanilla object references (modifiers, buildings, production methods) must exist in Victoria 3 v1.12
- **Localization**: Both English and Chinese keys must be present in [localization/](localization/) files
