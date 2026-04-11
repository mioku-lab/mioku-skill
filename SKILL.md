---
name: mioku-developer
description: Build, review, refactor, or explain Mioku plugins, services, framework integrations, and plugin AI skills against the real Mioku repository layout. Use when working on Mioku architecture, `plugins/*`, `src/services/*`, `skills.ts`, `runtime.ts`, plugin `package.json -> mioku.*`, WebUI `config.md`, or system services such as `ai`, `config`, `help`, `screenshot`, and `webui`. Exclude legacy Minecraft plugin/service work unless the user explicitly asks for it.
---

# Mioku Developer

Use this skill to make repository-accurate changes for Mioku. Treat the checked-out source as the source of truth and use the bundled references to avoid drifting back to older plugin contracts.

## Start Here

Follow this order:

1. Confirm whether the task is framework-level, plugin-level, service-level, AI-skill-level, or WebUI config-page work.
2. Inspect the target files in the repository before proposing changes.
3. Read only the relevant reference files:
   - `references/architecture.md` for startup flow, discovery, auto-registration, and layer boundaries
   - `references/plugin-development.md` for plugin package shape, `skills.ts`, `runtime.ts`, `config.md`, and help conventions
   - `references/service-development.md` for service contract, discovery, lifecycle, and plugin consumption
   - `references/system-services.md` for built-in non-Minecraft service APIs and caveats
4. Implement changes in the repository's established style.
5. Validate with `bun run build` for edits under `src/`, `plugins/`, or `src/services/`.

## Core Rules

- Prefer repository source over prose docs when they disagree.
- Treat Mioku as a layer on top of `mioki`: `mioki` handles bot/events/plugin execution, Mioku adds service discovery, plugin metadata discovery, auto help registration, and AI skill loading.
- Keep normal plugin `index.ts` focused on runtime behavior only.
- Declare required services only in `package.json -> mioku.services`.
- Declare help only in `package.json -> mioku.help`.
- Export global plugin AI skills only from `skills.ts`.
- Do not add `help` or `skill` fields onto the plugin object for normal plugins.
- Do not call `helpService.registerHelp(...)` or `aiService.registerSkill(...)` from normal plugins unless the task is explicitly changing Mioku framework internals.
- Use `runtime.ts` only as a bridge when `skills.ts` needs mutable state created during `setup()`.
- Ignore legacy Minecraft plugin/service code unless the user explicitly asks for it.

## Plugin Workflow

When creating or editing a plugin:

1. Read `package.json`, `index.ts`, and `skills.ts` if present.
2. Ensure the plugin directory name, plugin name, and package naming stay aligned.
3. Put declarative metadata in `package.json`.
4. Put event handlers, config registration, service wiring, and cleanup in `index.ts`.
5. Put reusable pure logic into `shared.ts` or `utils.ts` if the plugin is growing.
6. Add `runtime.ts` only when `skills.ts` must access setup-created state.
7. If the plugin needs WebUI editing, define `config.md` with `<configName>.<jsonPath>` keys.

## Service Workflow

When creating or editing a service:

1. Keep the service under `src/services/<name>/`.
2. Export a `MiokuService` object from `index.ts`.
3. Expose a typed API on `service.api`.
4. Initialize resources in `init()` and release them in optional `dispose()`.
5. Remember that Mioku injects service APIs into plugins through `ctx.services.<name>`.
6. Check whether plugin docs, examples, or imports need updates when service API shape changes.

## AI Skill Workflow

When the task involves plugin AI tools:

1. Read `references/plugin-development.md` and `references/system-services.md`.
2. Prefer plugin-scoped skills exported from `skills.ts` as `AISkill[]`.
3. Keep tool handlers small and predictable.
4. Pass runtime state through `runtime.ts` instead of closing over `setup()` locals.
5. Use `ai` service runtime features only when the plugin truly needs model calls or chat runtime integration.

## Validation

- Run `bun run build` after meaningful code changes in Mioku.
- If behavior changed, verify the affected startup path, message command, or service usage locally.
- Call out repository quirks that affect generated code, such as the current `src/services/config/tpyes.ts` filename typo.
