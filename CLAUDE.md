# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Strudel is a live coding environment for creating music patterns on the web. It's a JavaScript/TypeScript port of TidalCycles (Haskell), bringing pattern-based live coding to browsers. The project combines a pattern DSL (mini-notation), a core pattern engine, audio synthesis (Web Audio API), and a web-based REPL.

**Repository:** https://codeberg.org/uzu/strudel (NOT GitHub - project moved to Codeberg for ethical reasons)
**License:** AGPL-3.0-or-later

## Development Commands

### Setup
```bash
pnpm i                    # Install all dependencies (requires pnpm, not npm/yarn)
```

### Development Server
```bash
pnpm dev                  # Start REPL dev server at http://localhost:4321
pnpm start                # Alias for pnpm dev
```

### Testing
```bash
pnpm test                 # Run all tests with Vitest
pnpm test-ui              # Run tests with Vitest UI
pnpm snapshot             # Regenerate test snapshots (use after changing pattern functions)
```

### Code Quality
```bash
pnpm check                # Run format-check, lint, and test (CI check)
pnpm format-check         # Check code formatting with Prettier
pnpm codeformat           # Format all files with Prettier
pnpm lint                 # Run ESLint
```

### Building
```bash
pnpm build                # Build website for production
pnpm preview              # Preview production build
pnpm tauri dev            # Run Tauri desktop app in development
pnpm tauri build          # Build Tauri desktop app
```

### Other Commands
```bash
pnpm osc                  # Start OSC server (packages/osc)
pnpm jsdoc-json           # Generate JSDoc JSON (required before tests/builds)
```

### Running Single Tests
For individual package tests:
```bash
cd packages/<package-name>
pnpm test                 # Run tests for this package only
```

## Architecture

### Monorepo Structure

This is a **pnpm workspace monorepo** using **Lerna** for versioning. Key directories:

- `packages/` - 30+ independently versioned packages (the core of Strudel)
- `website/` - Astro-based website with the REPL and documentation
- `src-tauri/` - Rust source for desktop app (Tauri)
- `examples/` - Example projects
- `test/` - Shared test utilities
- `tools/` - Development tools (e.g., database patching)

### Key Packages

**Core Engine:**
- `@strudel/core` - Pattern engine (Pattern class, Hap events, time representation, scheduler)
- `@strudel/mini` - Mini-notation parser (Krill parser for TidalCycles-style string syntax)
- `@strudel/tidal` - Tidal-specific functions and compatibility
- `@strudel/transpiler` - JS → Strudel code transpiler (makes invalid JS semantically valid)

**Audio Output:**
- `@strudel/webaudio` - Web Audio API bindings
- `@strudel/superdough` - Main audio synthesis engine (samples, synths, effects via AudioWorklet)
- `@strudel/sampler` - Sample loading and management
- `@strudel/soundfonts` - SoundFont support

**Musical Theory:**
- `@strudel/tonal` - Music theory helpers (scales, chords, using tonal.js)
- `@strudel/xen` - Xenharmonic/microtonal support

**I/O & Integration:**
- `@strudel/midi` - MIDI input/output
- `@strudel/osc` - OSC (Open Sound Control) protocol
- `@strudel/serial` - Serial port communication
- `@strudel/mqtt` - MQTT protocol
- `@strudel/csound` - Csound integration
- `@strudel/hydra` - Hydra visuals integration

**UI Components:**
- `@strudel/repl` - REPL as a Web Component
- `@strudel/codemirror` - CodeMirror editor setup with Strudel syntax
- `@strudel/draw` - Canvas drawing for pattern visualization
- `@strudel/web` - Web-specific utilities

**Other:**
- `@strudel/embed` - Embeddable pattern player
- `@strudel/gamepad` - Gamepad controller support
- `@strudel/motion` - Motion sensor integration
- `@strudel/desktopbridge` - Bridge between web and Tauri desktop app

### Pattern Flow

1. **User Input** (mini-notation string or JS code) →
2. **Transpiler** (@strudel/transpiler) → Valid JS code →
3. **Mini Parser** (@strudel/mini) → Pattern AST →
4. **Core Pattern** (@strudel/core) → Hap events (time + value) →
5. **Scheduler** (cyclist.mjs in core) queries pattern for time spans →
6. **Audio Output** (@strudel/webaudio + @strudel/superdough) → Web Audio API

### Publishing Workflow

**IMPORTANT:** Always use `pnpm` for publishing (NOT npm). The packages use `publishConfig` to override entry points.

```bash
npm login
npx lerna version --no-private   # Update versions interactively
pnpm --filter "./packages/**" publish --dry-run   # Test publish
pnpm --filter "./packages/**" publish --access public   # Actually publish
```

For single package: Update version in package.json, then `pnpm publish` from package directory.

## Code Style

- **Formatter:** Prettier (120 char line width, 2 spaces, single quotes, trailing commas)
- **Linter:** ESLint (flat config in eslint.config.mjs)
- **File Extensions:** `.mjs` for JavaScript modules, `.ts/.tsx` for TypeScript
- **Testing:** Vitest with snapshot testing for pattern functions
- Use `pnpm codeformat` before committing

## Important Patterns & Conventions

### Pattern Function Testing

Pattern functions use **snapshot testing**. When adding/modifying pattern functions:
1. Write tests that query the pattern
2. Run `pnpm snapshot` to generate/update snapshots
3. Review snapshot diffs carefully

### Version Tagging

Patterns are versioned with `// @version x.y` comments to handle breaking changes. When releasing a version with breaking changes, database patterns must be tagged with their creation version to preserve behavior.

### Audio Worklet Development

- Worklet code is in `packages/superdough/worklets.mjs`
- Built with `vite-plugin-bundle-audioworklet`
- Globals like `currentTime`, `sampleRate` are provided by AudioWorkletGlobalScope

### JSDoc Documentation

- Run `pnpm jsdoc-json` before tests/builds to generate API documentation
- Check undocumented exports with `pnpm report-undocumented`

## Testing Considerations

- Tests use Vitest with `isolate: false` for performance
- `vitest.setup.mjs` resets RNG state between tests to avoid bleed
- Test files: `packages/*/test/*.test.mjs`
- Exclude pattern: See vitest.config.mjs

## Website (REPL) Development

The website is an **Astro** project with React components:
- Located in `website/`
- Dev server: `pnpm dev` from root (or `cd website && npm run dev`)
- Uses all Strudel packages as workspace dependencies
- REPL components in `website/src/repl/`
- Documentation in `website/src/content/`

## CI/CD

- **CI:** Forgejo workflows in `.forgejo/workflows/`
  - `test.yml` - Runs format-check, lint, and tests on push/PR
- **Local CI testing:** Use `gitlab-ci-local` before pushing (per user instructions)
- **NEVER commit/push without permission** (per user instructions)

## Special Files

- `doc.json` - Generated JSDoc documentation (committed to repo)
- `undocumented.json` - List of undocumented exports
- `warm.js` - Warming script (purpose TBD)
- `index.mjs` - Barrel export for finding undocumented exports

## Common Gotchas

1. **Always run `pnpm jsdoc-json` before tests/builds** - It's a pretest/prebuild hook
2. **Use pnpm, not npm** - Workspace dependencies won't resolve correctly with npm
3. **Don't import from `dist/`** - In development, packages use source files directly
4. **Pattern DSL != Regular JS** - The transpiler makes string literals and other "invalid" JS work as patterns
5. **Samples require setup** - Default samples from dough-samples repo, loaded via @strudel/sampler

## LLM/AI Policy

This project has a strict policy on LLM-generated code:
- Detail any LLM usage in PRs
- Wholly LLM-generated code is NOT accepted
- Do not add LLM features to Strudel
- This is a handmade, human project

## Community

- Discord: #strudel channels on Uzulang Discord (https://discord.com/invite/HGEdXmRkzT)
- Forum: https://club.tidalcycles.org/
- Mastodon: @strudel@social.toplap.org
