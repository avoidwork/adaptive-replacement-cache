# AGENTS.md

Rules and principles for agents working on **this** project.

---

## 1. Core Rules

### 1.1 Forbidden Patterns

The following are **strictly prohibited**:

- Hardcoded secrets, API keys, or credentials.
- Using `console.log()` in production code (use a proper logger).
- `try/catch` swallowing errors without re-throwing or handling.
- `eval()`, `Function()` at any level.
- `*` imports from external packages that don't support tree-shaking.

### 1.2 Security Rules

- Never store plaintext secrets. Use environment variables via `process.env`.
- Validate and sanitize all user input.
- No exposed credentials in source control.

---

## 2. Project Overview

**adaptive-replacement-cache** - A lightweight Adaptive Replacement Cache (ARC) implementation for Node.js and browsers.

- **Runtime**: Node.js with ES Modules (`"type": "module"`)
- **Bundler**: Rollup for CJS and ESM builds
- **Linting/Formatting**: Oxlint and Oxfmt
- **Tests**: Node.js built-in test runner via `tinybench`/`node:test`
- **Publishing**: npm (package: `adaptive-replacement-cache`)

### 2.0 Project Layout

```
adaptive-replacement-cache/
├── src/
│   └── arc.js          # ARC class + arc() factory function
├── dist/               # Built output (generated)
│   ├── adaptive-replacement-cache.cjs    # CommonJS
│   └── adaptive-replacement-cache.js     # ES Module
├── tests/unit/         # Unit tests
├── docs/
│   ├── ARCHITECTURE.md  # Detailed ARC algorithm docs
│   └── API.md          # Full API reference
├── package.json
├── rollup.config.js
├── AGENTS.md
└── README.md
```

### 2.1 Quick Commands

| Command           | Purpose                         |
|-------------------|---------------------------------|
| `npm install`     | Install dependencies            |
| `npm run build`   | Build CJS + ESM bundles         |
| `npm run lint`    | Run Oxlint + Oxfmt check        |
| `npm run fix`     | Auto-fix lint + formatting      |
| `npm test`        | Run linting then unit tests     |
| `npm run coverage`| Run tests with coverage report  |
| `npm run changelog`| Generate changelog             |

---

## 3. JavaScript/Node.js Conventions

### 3.1 Language & Tooling

- **JavaScript** (ESM), target latest stable Node.js
- **Package manager**: `npm`
- **Linter/Formatter**: Oxlint (`npm run lint`) + Oxfmt (`npm run fix`)
- **Bundler**: Rollup (dual CJS + ESM output)

### 3.2 Style

- 2 spaces for indentation, no tabs.
- Use `const` and `let`; avoid `var`.
- Use named exports (`export const`, `export function`) — default exports should only be used for the class itself (`ARC`).
- All public methods and the `ARC` class must have JSDoc comments.
- JSDoc follows Google style (or standard): `@param`, `@returns`, `@example`.

### 3.3 Error Handling

- Use `throw new Error()` or custom error classes for failure cases.
- Validate inputs at public method boundaries (e.g., `set()` should validate that the value is not `null`/`undefined` if that's a constraint).
- Do not swallow errors silently — log or re-throw.

### 3.4 Testing

- Tests are in `tests/unit/`.
- Each public method (`get`, `set`, `delete`, `has`, `clear`, `keys`, `values`, `entries`, `forEach`, `toJSON`) must have test coverage.
- Test filenames should follow the pattern for what they test (e.g., `arc-get.test.js`).

---

## 4. Architecture

### 4.1 ARC Class

The `ARC` class implements the Adaptive Replacement Cache algorithm:

- **Constructor**: `new ARC(size)` — creates cache with given maximum size; `p` starts at 0.
- **get(key)**: Retrieve value by key. If found in `t1`, move to `t2`.
- **set(key, value)**: Store value. Handles ghost hits (B1/B2) to adjust `p`.
- **delete(key)**: Remove key from `cache`, `t1`, `t2`, `b1`, `b2`.
- **has(key)**: Check if key exists in `cache`.
- **clear()**: Clear all five data structures (`cache`, `t1`, `t2`, `b1`, `b2`).

### 4.2 Internal Data Structures

The implementation uses **5 Maps**:

| Map  | Purpose | Contents |
|------|---------|----------|
| `cache` | Main storage | All key-value pairs |
| `t1` | Recently accessed (L1, transient) | Keys accessed once |
| `t2` | Frequently accessed (L2, stable) | Keys accessed multiple times |
| `b1` | Ghost list for T1 | Keys evicted from T1 (key only) |
| `b2` | Ghost list for T2 | Keys evicted from T2 (key only) |

**`p`** — the adaptive boundary controlling the target size of `t1`. `t2` target size = `maxSize - p`.

**Combined directory:**
```
[ B1 <-[ T1 <-> T2 ]-> B2 ]
```

### 4.3 Ghost Hit Adjustments

| Scenario | Action on `p` | Eviction |
|----------|---------------|----------|
| **B1 ghost hit** | `p += floor(|b2| / |b1|)` (increase T1) | Evict `t2` → `b2` |
| **B2 ghost hit** | `p -= floor(|b1| / |b2|)` (decrease T1) | Evict `t1` → `b1` |
| **Cache miss** | No change | Evict based on `p` and ghost list sizes |

Ghost lists act as "scorecards" tracking recent evictions. A ghost hit means the entry was recently evicted, so the algorithm adjusts `p` to favor that access pattern.

### 4.4 State Transitions

| Current State | Action | New State |
|---------------|--------|-----------|
| Not in cache | `set` | Enters `t1` |
| `t1` | `get` | Promoted to `t2` |
| `t2` | `get` | Refreshed in `t2` |
| `t1` | eviction | Added to `b1` |
| `t2` | eviction | Added to `b2` |
| `b1` | `set` (ghost hit) | Added to `t1` |
| `b2` | `set` (ghost hit) | Added to `t2` |

### 4.5 Performance

| Operation | Complexity |
|-----------|------------|
| `get` | O(1) |
| `set` | O(n) worst-case (evictions) |
| `delete` | O(1) |
| `has` | O(1) |
| `clear` | O(1) |

---

## 5. Git Conventions

### 5.1 Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add toJSON serialization to ARC
fix: correct B2 ghost hit p adjustment
docs: update ARCHITECTURE.md with state transitions
test: add unit tests for delete method
chore: update rollup config for ESM
```

### 5.2 Branching

- Default branch is `main`.
- Feature branches: `feat/<short-desc>` or `fix/<short-desc>`.
- Never commit directly to `main`. Create a feature branch and open a PR.

### 5.2.1 Agent Workflow

When auditing or modifying AGENTS.md (or any file):
1. Create a feature branch: `git checkout -b docs/<short-desc>` (or `feat/`, `fix/`).
2. Make changes and commit on the feature branch.
3. Push the feature branch and open a PR with `gh pr create --base main`.
4. Never commit or push directly to `main` or `master`.

### 5.3 Code Review

- All changes require at least one reviewer.
- PR descriptions must reference related issues or design docs.

### 5.4 Pull Request Templates

If a `.github/PULL_REQUEST_TEMPLATE.md` file exists, it MUST be used when creating PRs. Fill out every section — do not leave any section blank. If a section does not apply, write `N/A` rather than skipping it.

---

## 6. Operational Rules

### 6.1 Build Process

1. Rollup reads `./src/arc.js` as entry point.
2. Outputs two bundles to `dist/`:
   - `adaptive-replacement-cache.cjs` — CommonJS with named exports
   - `adaptive-replacement-cache.js` — ES Module with named exports
3. Both include a generated banner with copyright info.

### 6.2 Publish Workflow

When ready to publish:
1. Bump version in `package.json`.
2. Run `npm run changelog` to update changelog.
3. Commit changes.
4. Create release on GitHub.

---

## 7. Session Learnings

### 7.1 Delete Breaks ARC Balance

The `delete()` method currently performs naive removal from all five maps (`cache`, `t1`, `t2`, `b1`, `b2`) without adjusting `p`. This can create an imbalance:

- Deleting from B1 loses ghost history that T1 depends on for balancing.
- B2 may still contain ghosts from T2 evictions.
- The `p` boundary no longer has accurate history information.

This is documented as a known limitation. Any fix should restore ghost history (e.g., move entries from B2 to T2) or compensate with a `p` adjustment.

### 7.2 README is the Source of Truth for Public API

The `README.md` has the canonical public API surface. When adding new methods, update both `docs/ARCHITECTURE.md` and `README.md`.

---

## 8. Checklist Before Submitting a PR

- [ ] Tests pass locally (`npm test`)
- [ ] Linting and formatting pass (`npm run lint` and `npm run fix`)
- [ ] Build succeeds (`npm run build`)
- [ ] New/updated public APIs have JSDoc comments
- [ ] README.md updated if public API changed
- [ ] `docs/ARCHITECTURE.md` updated if internal design changed
- [ ] No hardcoded secrets or credentials introduced
- [ ] PR template filled out (every section or `N/A`)
