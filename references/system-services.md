# Mioku System Services Reference

## Scope

Use this reference when a plugin or framework task needs built-in non-Minecraft service APIs.

Current built-in non-legacy services in this repository:

- `ai`
- `config`
- `help`
- `screenshot`
- `webui`

Ignore `src/services/minecraft` unless the user explicitly asks for it.

## `ai`

Primary files:

- `src/services/ai/index.ts`
- `src/services/ai/types.ts`
- `docs/reference/ai-service.md`

What it provides:

- create and manage named AI instances
- set/get default AI instance
- register and query plugin AI skills
- register a chat runtime
- perform raw completion with optional executable tool loops

Key APIs:

- `create(...)`
- `get(name)`
- `setDefault(name)`
- `getDefault()`
- `registerSkill(skill)`
- `getSkill(skillName)`
- `getAllSkills()`
- `getTool(toolName)`
- `registerChatRuntime(runtime)`
- `getChatRuntime()`

AI instance APIs:

- `generateText(...)`
- `generateMultimodal(...)`
- `generateWithTools(...)`
- `complete(...)`

Important notes:

- default model fallback comes from `plugins/chat/configs/base.ts`
- plugin skills are auto-registered from `skills.ts`
- use `complete({ executableTools })` when the model should decide whether to call tools
- `chat-runtime` is a higher-level integration registered by the `chat` plugin, not by the core service itself

## `config`

Primary files:

- `src/services/config/index.ts`
- `src/services/config/tpyes.ts`
- `docs/reference/config-service.md`

What it provides:

- plugin config registration
- merged defaults for existing config files
- config reads and updates
- file watching and hot-reload callbacks

Key APIs:

- `registerConfig(pluginName, configName, initialConfig)`
- `updateConfig(pluginName, configName, updates)`
- `getConfig(pluginName, configName)`
- `getPluginConfigs(pluginName)`
- `onConfigChange(pluginName, configName, callback)`

Behavior notes:

- config files live under `config/<pluginName>/<configName>.json`
- register merges defaults into existing config
- callbacks are debounced

## `help`

Primary files:

- `src/services/help/index.ts`
- `src/services/help/types.ts`
- `docs/reference/help-service.md`

What it provides:

- store help manifests
- retrieve one plugin help block or the full registry

Key APIs:

- `registerHelp(pluginName, help)`
- `getHelp(pluginName)`
- `getAllHelp()`
- `unregisterHelp(pluginName)`

Important note:

- framework code auto-registers help from `package.json -> mioku.help`
- normal plugins should not call this service directly to self-register help

## `screenshot`

Primary files:

- `src/services/screenshot/index.ts`
- `src/services/screenshot/types.ts`
- `docs/reference/screenshot-service.md`

What it provides:

- render HTML to image with Puppeteer
- capture screenshots from URLs
- clean up old temporary screenshots

Key APIs:

- `screenshot(html, options?)`
- `screenshotFromUrl(url, options?)`
- `cleanupTemp(olderThanMs?)`

Behavior notes:

- temp output lives under `temp/screenshots`
- generated HTML injects Tailwind via CDN
- `screenshot(...)` writes a temporary HTML file then opens it with Puppeteer
- `screenshotFromUrl(...)` honors `waitTime`
- browser executable resolution differs by platform

## `webui`

Primary files:

- `src/services/webui/index.ts`
- `src/services/webui/types.ts`
- `src/services/webui/config-page-loader.ts`
- `docs/guide/webui.md`
- `docs/reference/config-page.md`

What it provides:

- WebUI HTTP server and WebSocket log stream
- auth gate for `/api/*`
- management routes for configs, AI, database, plugin config pages, memes, and plugin/service management
- parsing of plugin `config.md` manifests for WebUI editing

Public API exposed on `ctx.services?.webui`:

- `getSettings()`

Important notes:

- most plugin-facing WebUI integration happens through `config.md`, not direct `webui` service calls
- `config.md` is parsed with `gray-matter`
- field keys must follow `<configName>.<jsonPath>`
- current config-page field types are `text`, `textarea`, `number`, `switch`, `select`, `multi-select`, `secret`, and `json`

## Choosing a Service

- Use `ai` for model calls, tool loops, or chat runtime integration.
- Use `config` for persistent plugin settings and hot reload.
- Use `help` only when changing framework internals or querying help registry from another component.
- Use `screenshot` when rendering cards, menus, or shareable images.
- Use `webui` when the task involves the management UI or plugin `config.md` support.
