# Mioku Service Development Reference

## Scope

Use this reference when building or reviewing a service under `src/services/*`.

## Directory Contract

Minimum shape:

```text
src/services/<name>/
  index.ts
  package.json
```

Optional supporting files:

```text
src/services/<name>/
  types.ts
  utils.ts
  README.md
```

The repository also contains service-specific extras in some built-in services; match existing local patterns.

## Naming

- Git repository: `mioku-service-<name>`
- Local directory: `src/services/<name>`
- Runtime access path in plugins: `ctx.services?.<name>`

## Service Contract

Mioku services export a plain `MiokuService` object.

```ts
interface MiokuService {
  name: string;
  version: string;
  description?: string;
  init(): Promise<void>;
  api: Record<string, any>;
  dispose?(): Promise<void>;
}
```

Implementation rules:

- keep the exported `name` aligned with the directory name
- assign the public API to `service.api`
- initialize resources in `init()`
- clean up long-lived resources in optional `dispose()`

## Discovery and Loading

Derived from `src/core/service-manager.ts`.

- Mioku scans `src/services/*`
- a subdirectory is considered a service if it has `package.json`
- load entry is `index.ts`, fallback `index.js`
- after `init()`, Mioku injects `service.api` into `ctx.services[metadata.name]`
- load order is the directory iteration order returned by the filesystem; do not depend on a custom service order unless you control the framework

## Consumption From Plugins

Plugin side:

1. declare the service name in `package.json -> mioku.services`
2. read the API from `ctx.services?.<name>`
3. cast to the service API type when one exists

Example:

```ts
import type { ScreenshotService } from "../../src/services/screenshot/types";

const screenshotService = ctx.services?.screenshot as
  | ScreenshotService
  | undefined;
```

## Change Management

When changing a service API:

- update or add exported types
- update all plugin call sites
- check docs/examples that show import paths
- run `bun run build`

## Repository Quirk

The config service type file is currently named `src/services/config/tpyes.ts`. Use the real filename when editing imports in this repository unless you are explicitly fixing that typo.
