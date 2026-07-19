# cong-skills

Claude Code plugin marketplace. Each plugin groups skills by domain.

## Plugins

| Plugin | Skills | Description |
|---|---|---|
| [plan-skills](plan-skills) | `grill-me`, `to-plan` | Interview/stress-test a plan, convert a spec into a numbered plan file. |

## Adding a plugin

1. `mkdir -p <name>/.claude-plugin <name>/skills`
2. Add `<name>/.claude-plugin/plugin.json`:
   ```json
   {
     "name": "<name>",
     "version": "1.0.0",
     "description": "..."
   }
   ```
3. Add skills under `<name>/skills/<skill-name>/SKILL.md`.
4. Register in `.claude-plugin/marketplace.json`:
   ```json
   {
     "name": "<name>",
     "source": "./<name>",
     "description": "..."
   }
   ```

Naming convention: `<domain>-skills` (e.g. `database-skills`, `monitor-skills`).
