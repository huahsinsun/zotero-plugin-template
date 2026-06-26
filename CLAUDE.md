# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm start              # Serve with auto hot-reload (development)
npm run build          # Build + type-check (tsc --noEmit)
npm test               # Run tests via zotero-plugin scaffold
npm run lint:check     # Prettier + ESLint check
npm run lint:fix       # Prettier + ESLint auto-fix
npm run release        # Release to GitHub
```

**Environment setup**: Copy `.env.example` to `.env` and set `ZOTERO_PLUGIN_ZOTERO_BIN_PATH` and `ZOTERO_PLUGIN_PROFILE_PATH`. Use a dedicated dev profile created via `zotero.exe -p`.

## Architecture

This is a **Zotero 7 plugin template** — Zotero plugins are Firefox addons that run in a sandboxed JavaScript environment. The scaffold tool (`zotero-plugin-scaffold`) uses esbuild to bundle TypeScript source into a single JS file that executes inside Zotero's plugin sandbox.

### Plugin identity

Plugin metadata lives in `package.json` → `config`:
- `addonName` — display name
- `addonID` — unique plugin ID (e.g. `addontemplate@euclpts.com`)
- `addonRef` — short namespace used for chrome URIs, filenames, locale prefixes
- `addonInstance` — key on the global `Zotero` object where the addon instance lives
- `prefsPrefix` — Zotero preferences key prefix

These are injected at build time into `addon/` files via placeholder substitution (`__addonRef__`, `__addonName__`, etc.).

### Bootstrap & lifecycle (`addon/bootstrap.js`)

The Firefox addon entry point. On startup it:
1. Registers chrome content (so `chrome://<addonRef>/content/...` URIs resolve)
2. Creates a sandbox context with `_globalThis` as root, then loads the bundled script
3. Calls hooks in sequence: `onStartup` → `onMainWindowLoad` (per window) → `onShutdown`
4. On shutdown, calls `onShutdown` (unless the whole app is shutting down)

### TypeScript source (`src/`)

**Entry flow**: `src/index.ts` → instantiates `Addon` (from `addon.ts`) → stores it on `_globalThis.addon` and `Zotero[addonInstance]`.

**`src/addon.ts`** — The `Addon` class holds:
- `data` — plugin state (alive flag, config, env, ztoolkit instance, locale, prefs, dialog)
- `hooks` — lifecycle hook implementations
- `api` — public API object for other plugins

**`src/hooks.ts`** — THE central file. All Zotero lifecycle hooks and event callbacks are wired here. The pattern is pure dispatch: each hook switches on an event/type string and delegates to factory methods. Per the comment: *"hooks only do dispatch. Don't add code that does real jobs in hooks."*

**`src/modules/examples.ts`** — Example factory classes (`BasicExampleFactory`, `KeyExampleFactory`, `UIExampleFactory`, `PromptExampleFactory`, `HelperExampleFactory`) demonstrating the `zotero-plugin-toolkit` API surface. Each method is decorated with `@example` which wraps it in try/catch logging. This is the primary reference for how to use toolkit APIs.

**`src/modules/preferenceScript.ts`** — Preferences window logic, including a `VirtualizedTable` demo. Called from the preferences XHTML's `onpaneload`.

**Key utilities**:
- `utils/ztoolkit.ts` — Creates and configures the `ZoteroToolkit` singleton. Sets log prefix, plugin ID, icon URI. Contains a commented-out `MyToolkit` class showing how to tree-shake by importing only needed toolkit modules.
- `utils/locale.ts` — Fluent localization helpers. `initLocale()` loads `.ftl` files; `getString(key)` resolves a fluent message ID (auto-prefixed with `addonRef`); `getLocaleID(key)` returns the prefixed ID for use with Zotero APIs.
- `utils/prefs.ts` — Typed wrappers around `Zotero.Prefs.get/set/clear` that auto-prefix keys.
- `utils/window.ts` — `isWindowAlive(win)` checks for dead wrappers and closed windows.

### Static assets (`addon/`)

- `addon/manifest.json` — Firefox addon manifest with placeholders
- `addon/content/` — icons, `zoteroPane.css`, `preferences.xhtml`
- `addon/locale/{en-US,zh-CN}/` — Fluent `.ftl` files (`addon.ftl`, `mainWindow.ftl`, `preferences.ftl`)

### Type system (`typings/`)

- `global.d.ts` — Declares global variables: `addon`, `ztoolkit`, `rootURI`, `__env__`
- `i10n.d.ts` — **Auto-generated** `FluentMessageId` union type from `.ftl` keys. Regenerate when adding/removing locale strings.
- `prefs.d.ts` — **Auto-generated** `PluginPrefsMap` interface. Regenerate when changing preference keys in `addon/content/preferences.xhtml`.

### Build pipeline

`zotero-plugin.config.ts` defines:
- esbuild bundles `src/index.ts` → `.scaffold/build/addon/content/scripts/<addonRef>.js` targeting Firefox 115
- `__env__` is defined as `process.env.NODE_ENV` at build time
- `test.waitForPlugin` ensures tests wait until `addon.data.initialized` is true
- Assets from `addon/` are copied to the build output

### Testing

Tests use Mocha + Chai and run inside Zotero's environment via `zotero-plugin test`. The scaffold starts Zotero with the dev profile, loads the plugin, waits for initialization, then runs test files. Tests have access to all Zotero globals (`Zotero`, `addon`, `ztoolkit`).

## Agent skills

### Issue tracker

GitHub Issues，通过 `gh` CLI 操作，仓库为 `windingwind/zotero-plugin-template`。PR 不作为需求入口。详见 `docs/agents/issue-tracker.md`。

### Triage labels

全部使用规范名称: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`。详见 `docs/agents/triage-labels.md`。

### Domain docs

单上下文布局: 一个 `CONTEXT.md` + `docs/adr/` 位于仓库根目录。详见 `docs/agents/domain.md`。
