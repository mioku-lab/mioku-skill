# Mioku Plugin Development Reference

## Scope

Use this reference when building or reviewing a plugin under `plugins/*`.

## Directory Contract

Minimum shape:

```text
plugins/<name>/
  index.ts
  package.json
```

Optional files used by current Mioku conventions. Small plugins do not need all of them:

```text
plugins/<name>/
  skills.ts
  runtime.ts
  config.md
```

## Naming

- Git repository: `mioku-plugin-<name>`
- Local directory: `plugins/<name>`
- Runtime plugin name: `<name>`
- Keep these aligned unless the task is explicitly about migration.

## Package Contract

`package.json` is the declarative source for plugin metadata.

Typical shape:

```json
{
  "name": "mioku-plugin-weather",
  "version": "1.0.0",
  "description": "天气插件",
  "main": "index.ts",
  "mioku": {
    "services": ["config", "ai"],
    "help": {
      "title": "天气",
      "description": "天气插件示例",
      "commands": [
        {
          "cmd": "/天气",
          "desc": "查询天气",
          "usage": "/天气 北京",
          "role": "member"
        }
      ]
    }
  }
}
```

Important fields:

- `mioku.services`: required system services
- `mioku.help`: help manifest consumed automatically by Mioku

Do not put help metadata anywhere else for normal plugins.

## Runtime Contract

`index.ts` should define runtime behavior only:

- event handlers
- service lookup from `ctx.services`
- config registration and config-change listeners
- scheduled jobs
- setup-time initialization
- cleanup return function

Typical shape:

```ts
import { definePlugin } from "mioki";

export default definePlugin({
  name: "demo",
  version: "1.0.0",
  description: "示例插件",
  async setup(ctx) {
    ctx.handle("message", async (event) => {
      const text = ctx.text(event).trim();
      if (text !== "/demo") return;
      await event.reply("ok");
    });

    return () => {
      ctx.logger.info("demo unloaded");
    };
  },
});
```

## Current Mioku Rules

These rules reflect the repository's current contract, not older patterns.

- `index.ts` defines runtime behavior only.
- `package.json -> mioku.services` declares required services.
- `package.json -> mioku.help` is the only place to add help content.
- `skills.ts` is the only place to add plugin AI skills/tools.
- Do not define `help` or `skill` on the plugin object anymore.
- Do not call `helpService.registerHelp(...)` from normal plugins.
- Do not call `aiService.registerSkill(...)` from normal plugins.
- Small plugins can stay in a single focused `index.ts`.
- As complexity grows, split by responsibility into small modules instead of one oversized `index.ts` or one oversized shared helper file.

## `skills.ts`

Use `skills.ts` for global AI tools exposed by the plugin.

Rules:

- default-export `AISkill[]`
- keep handlers deterministic and narrow
- pull service access from runtime context if provided by the plugin system
- do not depend on `setup()` locals directly

Example pattern from `plugins/help/skills.ts`:

- export an array of skills
- each skill contains namespaced tools
- handlers read `runtimeCtx?.ctx` and `runtimeCtx?.event`

## `runtime.ts`

Use `runtime.ts` only when `skills.ts` needs mutable state created during `setup()`.

Pattern:

1. `index.ts` computes runtime state during setup.
2. `index.ts` writes that state into `runtime.ts`.
3. `skills.ts` reads from `runtime.ts`.
4. cleanup resets the state.

This avoids capturing setup-local variables from a file that Mioku imports independently.

Important implementation note:

- do not implement `runtime.ts` as a plain module-local singleton like `const runtimeState = {}`
- Mioku/mioki may load plugin modules through separate import paths and `mioki` currently disables `jiti` module cache
- prefer the shared runtime registry in `src/core/plugin-runtime-state.ts`

## `shared.ts` and `utils.ts`

Use these files for pure reusable helpers. They are optional conventions, not mandatory filenames.

- keep mutable process state in `runtime.ts`
- keep pure rendering/parsing/business logic in `shared.ts`, `utils.ts`, or other focused helper modules

## Service Usage From Plugins

Lookup pattern:

```ts
const aiService = ctx.services?.ai as AIService | undefined;
const configService = ctx.services?.config as ConfigService | undefined;
```

Always handle missing services gracefully even when declared in `mioku.services`.

## WebUI Config Pages

If the plugin should expose editable settings in WebUI, add `config.md` at plugin root.

Frontmatter fields:

- `title`
- `description`
- `fields`

Each field key must use:

```text
<configName>.<jsonPath>
```

Examples:

- `base.enabled`
- `base.reply`
- `settings.persona.name`

Supported field types from `src/services/webui/config-page-loader.ts`:

- `text`
- `textarea`
- `number`
- `switch`
- `select`
- `multi-select`
- `secret`
- `json`

## Context and Event Notes

Useful `ctx` members from Mioku/mioki docs:

- `ctx.handle(...)`
- `ctx.text(event)`
- `ctx.match(...)`
- `ctx.segment`
- `ctx.logger`
- `ctx.cron(...)`
- `ctx.pickBot(ctx.self_id)` **USE THIS INSTEAD OF ctx.bot**
- `ctx.isOwner(event)`
- `ctx.isAdmin(event)`

Prefer `ctx.pickBot(ctx.self_id)` over `ctx.bot` when the task may run in multi-bot setups.

## Repository Quirks

- `plugins/help/package.json` currently declares services but no `mioku.help`; do not infer that as the preferred pattern.
- The current repository contract says help belongs in `package.json -> mioku.help`, even if some built-in plugins lag behind.
