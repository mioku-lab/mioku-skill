# CLAUDE.md

This skill is for repository-accurate Mioku development.

## Use This Skill For

- building or reviewing code in `plugins/*`
- building or reviewing code in `src/services/*`
- changing Mioku startup, discovery, or auto-registration behavior
- adding or fixing plugin `skills.ts`
- adding or fixing plugin `runtime.ts`
- wiring plugin help through `package.json -> mioku.help`
- wiring plugin service dependencies through `package.json -> mioku.services`
- adding plugin WebUI config pages through `config.md`
- working with built-in non-Minecraft services: `ai`, `config`, `help`, `screenshot`, `webui`

Ignore legacy Minecraft plugin/service code unless the task explicitly targets it.

## Operating Rules

- Treat repository source as the source of truth.
- Do not fall back to older Mioku plugin patterns.
- Keep plugin runtime behavior in `index.ts`.
- Keep plugin AI tools in `skills.ts`.
- Use `runtime.ts` only to bridge state created in `setup()` to `skills.ts`.
- Keep plugin help in `package.json -> mioku.help`.
- Do not manually call `helpService.registerHelp(...)` or `aiService.registerSkill(...)` from normal plugins.

## Read Order

Read only what the task needs:

1. `references/architecture.md`
   Use for startup flow, discovery, service loading, and auto-registration.
2. `references/plugin-development.md`
   Use for plugin layout, package contract, `skills.ts`, `runtime.ts`, and `config.md`.
3. `references/service-development.md`
   Use for `MiokuService`, `service.api`, service lifecycle, and plugin consumption.
4. `references/system-services.md`
   Use for built-in non-Minecraft service APIs and caveats.

## Working Pattern

1. Inspect the target repository files first.
2. Match the local conventions already used in the repository.
3. Keep TypeScript changes small and typed.
4. If plugin or service behavior changes, check dependent imports and call sites.
5. Validate with `bun run build` after edits under `src/`, `plugins/`, or `src/services/`.

## Repository-Specific Notes

- Mioku sits on top of `mioki`.
- `plugins/boot` loads services first, then Mioku auto-registers help and skills.
- Plugin skills are discovered from `skills.ts` or `skills.js`.
- Service APIs are exposed to plugins through `ctx.services.<name>`.
- The current repository has a real filename typo at `src/services/config/tpyes.ts`; use the real path unless explicitly fixing it.
