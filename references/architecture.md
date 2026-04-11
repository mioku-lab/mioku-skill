# Mioku Architecture Reference

## Scope

Use this reference when the task touches framework startup, plugin/service discovery, or automatic help/skill registration.

## Mental Model

Mioku sits on top of `mioki`.

- `mioki` handles bot connections, events, and plugin execution.
- Mioku adds:
  - plugin metadata discovery
  - service discovery and loading
  - automatic plugin help registration from `package.json -> mioku.help`
  - automatic plugin AI skill registration from `skills.ts`

## Startup Flow

Derived from `src/index.ts`, `plugins/boot/index.ts`, `src/core/plugin-manager.ts`, `src/core/service-manager.ts`, and `src/core/plugin-artifact-registry.ts`.

1. `app.ts` starts Mioku.
2. `src/index.ts` changes to the selected cwd and logs startup.
3. `pluginManager.discoverPlugins()` scans `plugins/*/package.json`.
4. `serviceManager.discoverServices()` scans `src/services/*/package.json`.
5. Mioku collects required services from all plugin metadata and warns if any are missing.
6. `mioki.start(...)` runs.
7. The `boot` plugin loads first with `priority: -Infinity`.
8. `boot` calls `serviceManager.loadAllServices(ctx)`.
9. Each loaded service exposes `service.api` on `ctx.services.<serviceName>`.
10. `boot` calls `registerPluginArtifacts(ctx)`.
11. `registerPluginArtifacts`:
   - registers help manifests from `package.json -> mioku.help`
   - imports `skills.ts` or `skills.js` from enabled plugins
   - registers exported `AISkill` objects into the `ai` service
12. Normal plugins continue running under `mioki`.

## Discovery Rules

### Plugins

- Source directory: `plugins/`
- Discovery condition: subdirectory contains `package.json`
- Metadata name: directory name, not npm package name
- Required-service collection comes from `packageJson.mioku.services`

### Services

- Source directory: `src/services/`
- Discovery condition: subdirectory contains `package.json`
- Load entry: `index.ts`, fallback `index.js`
- Exposed runtime API: `ctx.services.<directoryName> = service.api`

## Plugin Artifact Auto-Registration

`src/core/plugin-artifact-registry.ts` is the key bridge.

- Help is loaded from plugin metadata only.
- Skills are loaded from `skills.ts` or `skills.js`.
- Accepted exports for skills:
  - `default`
  - named `skills`
  - module object itself if it matches
- Valid skill shape is checked loosely:
  - object with string `name`
  - array `tools`

Implication:

- normal plugins should not manually register help or AI skills
- framework changes belong in Mioku core, not in individual plugin runtime setup

## Layer Boundaries

### Core

Files under `src/core/` manage discovery, metadata, loading, and framework-level types.

### Services

Files under `src/services/*` provide reusable capabilities via `ctx.services`.

### Plugins

Files under `plugins/*` implement user-facing behavior, commands, handlers, and optional AI tools.

## Current Repository Notes

- Root package version is `0.5.9`.
- Root workspaces are `plugins/*` and `src/services/*`.
- Default enabled plugins in root `package.json` are `boot`, `help`, and `chat`.
- Non-legacy services present in the repository are `ai`, `config`, `help`, `screenshot`, and `webui`.
- Legacy Minecraft code exists in both `plugins/minecraft` and `src/services/minecraft`; ignore it unless explicitly requested.
