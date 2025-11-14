# Repository Guidelines

## Project Structure & Module Organization
This repo is a Yarn/Lerna monorepo. Runtime packages live under `packages/`, e.g., `engine-server` (indexing pipeline), `plugin-core`/`dendron-plugin-views` (VS Code), `dendron-cli` (binary), and `nextjs-template` (site export). Docs, specs, and assets live in `docs/`, automation scripts in `bootstrap/` and `shell/`, and reproducible workspaces in `dev/` and `test-workspace/`. Keep tests beside sources inside each package (`src/...` + `__tests__`) to retain ownership.
一律用繁體中文回答

## Build, Test, and Development Commands
`yarn setup` bootstraps dependencies, builds every package, and ensures the CLI is executable. During daily work favor `yarn bootstrap:init` or `yarn bootstrap:build:fast` to rebuild only touched scopes. `yarn lint`, `yarn format`, and `yarn lerna:typecheck` are the minimum pre-push gates. Run `yarn test` for the full Jest suite, `yarn test:cli` for CLI-only projects, and `yarn --cwd packages/nextjs-template run test` for template smoke checks.

## Coding Style & Naming Conventions
All code is TypeScript or modern ES modules. Prettier forces 2-space indentation and double quotes (`prettier.config.js`); never reflow manually. ESLint extends Airbnb plus Jest/React plugins, so resolve warnings instead of suppressing. Use `PascalCase` for React components and classes, `camelCase` for helpers, and kebab-case file names (e.g., `note-index-builder.ts`). Shared schemas and constants belong in `packages/common-*` so downstream packages stay in sync.

## Testing Guidelines
Jest is the single runner. Name files `*.spec.ts` or `*.test.ts`, co-locating them with sources. Favor higher-level tests built with helpers from `packages/common-test-utils` and `packages/engine-test-utils`; add unit tests only where behavior is tightly scoped. For CLI feature work, refresh snapshots via `yarn test:cli -u` and keep fixture updates intentional. PRs must state what was tested and call out any skipped coverage.

## Commit & Pull Request Guidelines
Commits follow Conventional Commits (`feat(engine-server): ...`, `fix(plugin-core): ...`) so `standard-version` can generate changelogs. Reference GitHub issues with “Fixes #123” in the body. Every PR should paste the appropriate checklist linked in `pull_request_template.md`, include repro steps or screenshots for UX work, and highlight manual test output. Do not request review until `yarn lint` and `yarn test` both pass.

## Environment & Tooling Notes
Stick to the Node version pinned in `package.json`, use Yarn 1.x, and rely on Lerna for per-package scripts (`npx lerna run build --scope @dendronhq/dendron-cli`). Husky hooks (`lint-staged`, `hooks:pre-commit`, `hooks:pre-push`) run automatically; use `yarn stash:unstaged` if you need the hooks to bypass local edits. `node bootstrap/scripts/buildAll.js` mirrors CI dependency resolution, and `yarn gen:data` regenerates config schemas consumed by the plugin.
