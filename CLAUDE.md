# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
一律用繁體中文回答

## Status
Dendron is currently in **maintenance mode** - active development has ceased. This is a reference guide for understanding the codebase and working within it.

## 1. What is Dendron?

Dendron is an **open-source, local-first, markdown-based, note-taking tool** for personal knowledge management (PKM). It's built specifically for developers and integrates natively with IDEs like VS Code and VSCodium.

Key Design Principles:
- Developer Centric: Efficient, extensible, keyboard-focused
- Gradual Structure: Works at any scale (1 to 10k+ notes)
- Flexible & Consistent: Automatic consistency maintenance
- Just Plaintext: Git-compatible, editor-agnostic

Core Features:
- Hierarchical note lookup system
- Note references and embedding
- Markdown extensions (mermaid, KaTeX, wiki links)
- Multiple vault support
- Pod system (import/export to Obsidian, Notion, Airtable, etc.)
- Backlinks and graph views
- Journal and scratch notes
- Templates and schemas

## 2. Monorepo Structure

Dendron uses Yarn Workspaces with TypeScript. 15 main packages organized by function:

### Core Engine Packages
- engine-server: Core note management, hierarchy, file operations
- unified: Markdown processing (AST manipulation, remark/rehype)
- api-server: Express REST API server
- pods-core: Import/export to external formats

### Common/Shared Packages
- common-all: Isomorphic types, config, utilities (foundational)
- common-server: Logging, file I/O, git operations
- common-frontend: React/Redux primitives
- common-test-utils: Testing fixtures

### Application Packages
- plugin-core: VSCode Extension (100+ commands)
- dendron-cli: Command-line interface
- dendron-viz: D3.js graph visualization
- dendron-plugin-views: React webviews for VSCode
- nextjs-template: Next.js website generator

## 3. Package Dependencies

Dependency Flow:
common-all (foundational)
  ↓
common-server, unified
  ↓
engine-server
  ↓
api-server, pods-core, engine-test-utils
  ↓
dendron-viz, dendron-cli, common-frontend
  ↓
dendron-plugin-views
  ↓
plugin-core (highest consumer)

Key Rules:
1. common-all is foundational (used by almost all)
2. engine-server is central (core engine)
3. plugin-core is highest consumer
4. api-server runs as separate process
5. dendron-cli talks directly to engine-server
6. unified handles markdown AST work
7. pods-core depends on engine-server

## 4. Key Architectural Patterns

### The Engine Pattern
engine-server provides:
- DendronEngineV3: Main engine (indexing, hierarchy)
- DendronEngineV2: Legacy implementation
- EngineClient: Remote HTTP client

Core methods:
- .init(): Initialize workspace
- .getNote(id): Retrieve single note
- .queryNotes(opts): Search
- .updateNote(note): Persist
- .renameNote(): Move with backlinks
- .deleteNote(): Remove from hierarchy

### Unified.js Pipeline (Markdown)
remark-parse (markdown → AST)
  ↓ [remark plugins]
remark-rehype (AST → HTML)
  ↓ [rehype plugins]
rehype-stringify (HTML output)

### Pod System (Import/Export)
Pluggable architecture:
- Pod Interface: JSON config
- PodExecutor: Import/export logic
- Built-in: Obsidian, Notion, Airtable, Google, CSV, Email

### Vault System
Multiple knowledge bases:
- Self-contained vaults
- Multi-vault workspaces
- Migration/conversion
- Snapshot/restore

## 5. Technology Stack

Languages & Tools:
- TypeScript 4.6
- Yarn Workspaces
- tsc + Webpack

Markdown & AST:
- unified (9.1.0)
- remark (12.0.1)
- rehype, mdast, Gray-matter

Server & API:
- Express.js (4.17.1)
- Prisma (4.1.1), SQLite3 (5.1.2)
- Pino (6.3.2) logging

Frontend & UI:
- React (17.0.2), Redux/Toolkit
- Ant Design (4.21.4)
- D3.js (7.6.1)
- Cytoscape (3.19.0)

Development:
- Jest (28.1.0) testing
- ESLint, Prettier

Validation:
- Zod (3.19.1)
- AJV

Integrations:
- Dropbox, Airtable, Notion, Google APIs
- GitHub GraphQL, Sentry, Segment

## 6. Build & Compilation

Build Order (dependency-based):
1. common-all
2. common-server, unified
3. engine-server
4. api-server, pods-core, engine-test-utils
5. dendron-viz, dendron-cli, common-frontend
6. dendron-plugin-views
7. plugin-core

Commands:
yarn setup                    # Full setup
yarn bootstrap:bootstrap      # Install + metadata
yarn bootstrap:build          # Build all
yarn bootstrap:build:fast     # Critical only

Compilation:
1. Clean: rimraf lib
2. Compile: tsc -p tsconfig.build.json
3. Special:
   - engine-server: buildPrismaClient
   - dendron-viz: copyNonTSFiles
   - plugin-core: webpack build

## 7. Development Workflow

Setup:
cd dendron/
yarn setup
yarn bootstrap:build:fast

Engine work:
cd packages/engine-server/
yarn watch

Plugin work:
cd packages/plugin-core/
yarn watch
yarn test

CLI work:
cd packages/dendron-cli/
yarn watch

Testing:
yarn test              # All tests
yarn test:cli          # CLI tests
yarn ci:test:plugin    # VSCode tests

Linting:
yarn lint              # ESLint
yarn format            # Prettier
yarn lerna:typecheck   # Type check

## 8. Important Files

Config:
- /package.json
- /.eslintrc.js
- /babel.config.js
- /bootstrap/

Key Directories:
engine-server/src/:
- DendronEngineV3.ts (main)
- store/, drivers/, migrations/

plugin-core/src/:
- _extension.ts (main)
- commands/ (100+ handlers)
- features/, components/

pods-core/src/:
- pods/ (individual implementations)

unified/src/:
- index.ts (pipeline)
- utils/

## 9. Code Patterns

Error Handling: neverthrow Result types
Dependency Injection: tsyringe
Validation: Zod schemas
Note Hierarchy: Dot-separated IDs (dendron.concepts.naming)

## 10. Testing

yarn test                          # All
yarn test:cli                      # CLI only
npm test -- --testPathPattern=...  # Specific
yarn test:cli:update-snapshots     # Snapshots

## 11. Release & Publishing

yarn release                       # Bump version
yarn build:patch:local            # Patch
yarn deploy:vscode                # VS Code
yarn deploy:ovsx                  # Open VSX

## 12. Common Tasks

Add a command:
1. Define in plugin-core/src/commands/
2. Register in package.json
3. Call via vscode.commands.executeCommand()

Add a Pod:
1. Create in pods-core/src/pods/yourFormat/
2. Implement interface
3. Register in factory

Extend markdown:
1. Add to unified/ or engine-server/
2. Update tests

Add CLI command:
1. Create in dendron-cli/src/commands/
2. Register in yargs
3. Test with yarn dendron

Modify storage:
1. Change Prisma schema
2. Create migration
3. Run yarn buildPrismaClient

---

Dendron v0.124.0 (Maintenance Mode)
https://github.com/dendronhq/dendron
