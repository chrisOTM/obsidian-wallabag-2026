# AGENTS.md

## Scope
- Applies to `/home/chris/Python/Projekte/2026/Obisian_Wallabag_Plugin/obsidian-wallabagger-master`.
- This repository is an Obsidian plugin written in TypeScript.
- Main source entrypoint: `src/main.ts`.
- Bundled output expected by Obsidian: `main.js`.

## Repository Layout
- `src/main.ts`: plugin bootstrap, settings load, auth state, command registration.
- `src/command/`: user-triggered command implementations.
- `src/settings/`: settings model and settings tab UI.
- `src/wallabag/`: Wallabag auth and API calls.
- `src/note/`: note templates and template filling.
- `manifest.json`: Obsidian plugin manifest.
- `versions.json`: supported Obsidian versions by plugin release.
- `esbuild.config.mjs`: esbuild bundling and watch config.
- `version-bump.mjs`: release metadata update script.

## Setup
- Install dependencies with `npm install`.
- Prefer `npm run ...` over calling tools directly; TypeScript and ESLint are local devDependencies.
- The repo currently has `package-lock.json`, so use `npm`, not `pnpm` or `yarn`.

## Build Commands
- Install deps: `npm install`
- Watch build: `npm run dev`
- Production build: `npm run build`
- Lint: `npm run lint`
- Version bump for release prep: `npm run version`

## Command Details
- `npm run dev`: runs `node esbuild.config.mjs` in watch mode.
- `npm run build`: runs `tsc -noEmit -skipLibCheck && node esbuild.config.mjs production`.
- `npm run lint`: runs `eslint ./src`.
- `npm run version`: updates `manifest.json` and `versions.json`, then stages them with git.

## Test Status
- There is no configured test framework in `package.json`.
- No `test` script exists.
- No Jest, Vitest, Mocha, or Playwright config files were found.
- No dedicated `*.test.*` or `*.spec.*` files were found in this repository snapshot.

## Single-Test Guidance
- A single-test command does not exist yet because no test runner is configured.
- Do not invent test commands for this repo.
- If asked to run one test, state clearly that single-test execution is unsupported today.
- Use `npm run lint` and `npm run build` as the current validation baseline.
- If a test runner is added later, update this file with:
- Full suite command.
- Single-file command.
- Single-test-name command.

## Recommended Validation Flow
- After code changes, run `npm run lint`.
- Then run `npm run build`.
- For behavior changes, also manually verify in Obsidian.

## Manual Obsidian Validation
- Build with `npm run build`.
- Copy `main.js` and `manifest.json` into the target vault plugin directory.
- Reload the plugin in Obsidian by disabling and re-enabling it.
- Verify authentication flow.
- Verify article sync.
- Verify cache reset.
- Verify delete-and-remove-from-cache behavior.
- If PDF export or annotations mode changed, validate those flows explicitly.

## Toolchain Facts
- Language: TypeScript.
- Module resolution uses `baseUrl: "src"`.
- Bundler: esbuild.
- Build target: `es2020` in esbuild, `es2018` in TypeScript.
- Source maps are inline in dev builds and disabled in production builds.
- ESLint uses `eslint:recommended` and `plugin:@typescript-eslint/recommended`.
- Prettier is installed, but no format script or project Prettier config was found.

## Rules Files Check
- No `.cursor/rules/` directory was found.
- No `.cursorrules` file was found.
- No `.github/copilot-instructions.md` file was found.
- There are no additional Cursor or Copilot instructions to merge into this guide.

## Import Guidelines
- Follow the existing import style already used in `src/`.
- Prefer `baseUrl` imports for internal modules, for example `command/...`, `note/...`, `wallabag/...`, and `main`.
- Use relative imports for nearby files in the same area, for example `./WallabagSettings`.
- Keep one import statement per module path.
- Group Obsidian imports in one statement, for example `import { Notice, Plugin } from 'obsidian';`.
- Keep import ordering consistent with the surrounding file instead of reordering unrelated imports.

## Formatting Guidelines
- Use 2-space indentation.
- Use Unix line endings.
- Use single quotes.
- End statements with semicolons.
- Remove trailing whitespace.
- Always use braces for control statements; ESLint enforces `curly`.
- Use strict equality; ESLint enforces `eqeqeq`.
- Match the surrounding file style before making broad cleanup edits.

## TypeScript Guidelines
- `noImplicitAny`, `strictNullChecks`, `noImplicitReturns`, `noImplicitThis`, and `noUnusedLocals` are enabled.
- Do not introduce implicit `any`.
- Prefer explicit interfaces and types for structured data such as settings, tokens, articles, and API responses.
- Add explicit return types on exported functions and on async APIs when useful for clarity.
- Reuse existing interfaces before creating near-duplicate types.
- Keep types narrow and local when they are not reused elsewhere.
- The codebase currently tolerates some existing `any`; avoid expanding that pattern.

## Naming Guidelines
- Classes: `PascalCase`.
- Interfaces and type aliases: `PascalCase`.
- Functions and methods: `camelCase`.
- Local variables and properties: `camelCase`.
- Constants: `UPPER_SNAKE_CASE` only for true constants, such as `DEFAULT_SETTINGS`.
- Setting keys are camelCase, for example `syncOnStartup`, `archiveAfterSync`, `pdfFolder`, and `syncContentMode`.
- Filenames are generally `PascalCase.ts` for classes and feature modules.

## Architecture Guidelines
- Keep plugin orchestration in `src/main.ts`.
- Keep Obsidian command implementations in `src/command/` as `Command` objects/classes.
- Keep settings shape and settings UI in `src/settings/`.
- Keep Wallabag auth, token refresh, and HTTP requests in `src/wallabag/`.
- Keep note rendering and template substitution in `src/note/`.
- Prefer small direct methods over adding extra abstraction layers.
- Preserve persisted filenames such as `.synced` and `.__wallabag_token__` unless a migration is intentional.

## Error Handling Guidelines
- Use `Notice` for user-visible failures and status messages.
- Prefer actionable messages that tell the user what to do next.
- Keep plugin state consistent before notifying the user about a recoverable failure.
- Do not swallow errors silently.
- Existing code uses `console.log` in some failure paths; if improving that area, use clearer logging and meaningful thrown errors.
- Avoid throwing empty `Error('')` messages in new code.

## Async And Persistence Guidelines
- Use `async`/`await` or existing promise style consistently within the touched file.
- Await vault writes and network operations when sequencing matters.
- Do not introduce parallel vault writes unless the behavior is clearly safe.
- Settings persist through `loadData()` and `saveData()`.
- Token state is stored in `.__wallabag_token__` under the plugin directory.
- Synced article IDs are stored in `.synced` under the plugin directory.

## Obsidian-Specific Guidelines
- Use `normalizePath` when building vault paths.
- Use `PluginSettingTab` and `Setting` for settings UI.
- Respect current command IDs and user-facing labels unless a rename is intentional.
- Keep generated artifacts compatible with Obsidian's expected `main.js` and `manifest.json` layout.
- Follow the existing pattern of reloading settings UI after changes when the visible form depends on current state.

## Change Strategy For Agents
- Make the smallest correct change.
- Preserve current behavior unless the task explicitly changes behavior.
- Avoid opportunistic refactors during targeted fixes.
- If you change settings, commands, or persisted data, verify downstream behavior manually.
- Update this file when build, lint, test, or workflow expectations change.

## Practical Defaults
- Start with `npm install` if dependencies are missing.
- Validate with `npm run lint` and `npm run build` after code changes.
- If asked to run a single test, explain that the repository currently has no test runner.
- If you add tests or new tooling, document the exact commands here immediately.
