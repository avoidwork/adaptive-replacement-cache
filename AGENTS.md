# AGENTS.md

## Project Overview

**tiny-arc** - A minimal ARC (Adaptive Replacement Cache) implementation in JavaScript.

## Project Structure

```
/home/jason/projects/tiny-arc/
├── src/
│   └── arc.js          # ARC class implementation
├── dist/               # Built output (generated)
│   ├── tiny-arc.cjs    # CommonJS build
│   └── tiny-arc.js     # ES Module build
├── tests/unit/         # Unit tests directory
├── node_modules/       # Dependencies (gitignored)
├── .gitignore
├── package.json
├── rollup.config.js    # Rollup bundler configuration
├── AGENTS.md           # This file
└── README.md           # (exists, see RTK.md references)
```

## Key Technologies

- **Runtime**: Node.js with ES Modules (`"type": "module"`)
- **Bundler**: Rollup for creating CJS and ESM builds
- **Linting/Formatting**: Oxlint and Oxfmt
- **Git Hooks**: Husky
- **Changelog**: Auto-changelog

## Commands

| Command | Description |
|---------|-------------|
| `npm run build` | Build the project using Rollup |
| `npm run rollup` | Run Rollup bundler |
| `npm run lint` | Run Oxlint linter and Oxfmt formatter check |
| `npm run fix` | Run Oxlint auto-fix and Oxfmt formatting |
| `npm run test` | Run linting then Node.js tests |
| `npm run coverage` | Run tests with coverage reporting |
| `npm run changelog` | Generate changelog with auto-changelog |

## Architecture

### ARC Class

The `ARC` class implements the Adaptive Replacement Cache algorithm:

- **Constructor**: `new ARC(size)` - Creates cache with given maximum size
- **get(key)**: Retrieve value, updates access patterns
- **set(key, value)**: Store value, maintains cache bounds
- **delete(key)**: Remove key from all internal lists
- **has(key)**: Check if key exists
- **adjust()**: Internal method to balance p1/p2 and t1/t2 lists

### Internal Lists

- **P1, P2**: Recently accessed item lists
- **T1, T2**: Frequently accessed item lists
- The algorithm adaptively balances between these based on access patterns

## Build Process

1. Rollup reads `./src/arc.js` as entry point
2. Outputs two bundles to `dist/`:
   - `tiny-arc.cjs` - CommonJS with named exports
   - `tiny-arc.js` - ES Module with named exports
3. Both include generated banner with copyright info

## Development Workflow

1. Create feature branch from `init`
2. Implement changes in `src/arc.js`
3. Run tests: `npm run test`
4. Fix formatting: `npm run fix`
5. Build: `npm run build`
6. Commit changes with descriptive message
7. Push branch and create pull request

## Branch Strategy

- **init**: Current development branch
- **master**: Production (default, but using 'init' currently)

## Git Hooks

Husky is configured via `"prepare": "husky"` in package.json to run on npm install.

## Publishing

When ready to publish:
1. Bump version in package.json
2. Run `npm run changelog` to update changelog
3. Commit changes
4. Create release on GitHub

## Important Notes

- `node_modules` and `dist` are gitignored
- Package type is ESM (`"type": "module"`)
- Exports field in package.json defines dual CJS/ESM support
- Main entry points: `./dist/tiny-arc.cjs` (require) and `./dist/tiny-arc.js` (import)
