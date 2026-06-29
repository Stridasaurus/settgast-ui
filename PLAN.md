# @settgast/ui — Publishing & Consolidation Plan

## Goal

Make `settgast-ui` the canonical, published source of truth for the `@settgast/ui`
component library. Publish it to npm (public registry). Make `dynamica` depend on
the published version instead of its own local copy. This consolidates design into
one place so all future projects just install the package.

---

## Repo layout

```
settgast-ui/               ← THIS repo — canonical source
  packages/ui/             ← the publishable package (@settgast/ui@0.1.0)

dynamica/                  ← consumer app (separate repo)
  packages/ui/             ← duplicate of @settgast/ui — to be DELETED after publish
  apps/dynamica/           ← Vite/React 19 app, currently uses workspace:*
```

---

## What has already been done

All changes below were applied manually in a prior session. They are saved to disk
but **not committed**.

### settgast-ui/packages/ui/package.json

- Added `"files": ["dist"]` — restricts the npm tarball to built output only
- Fixed `"./tokens.css"` export: was `./src/tokens/tokens.css` (src wouldn't ship),
  now `./dist/tokens.css`
- Added `"./styles.css": "./dist/index.css"` — the full component stylesheet export
  that consumers must import for styled components
- Added `"publishConfig": { "access": "public" }` — prevents needing `--access public`
  on every publish command
- Updated devDependencies React → `^19.0.0`, `@types/react` → `^19.0.0` (matches dynamica)
- Added `"changeset"`, `"version-packages"`, `"release"` scripts for Changesets workflow

### settgast-ui/packages/ui/tsup.config.ts

- Added `copyFileSync` to the `fs` import
- Added `copyFileSync('src/tokens/tokens.css', 'dist/tokens.css')` at the end of
  `onSuccess` so `dist/tokens.css` is emitted alongside `dist/index.css`

### settgast-ui/package.json (root)

- Added `"@changesets/cli": "^2.27.0"` to `devDependencies`

### dynamica/packages/ui/package.json

- Synced to exactly match the canonical settgast-ui version above

### dynamica/apps/dynamica/src/main.tsx

- Changed `import '@settgast/ui/styles'` → `import '@settgast/ui/styles.css'`
  (the export key was renamed from `./styles` to `./styles.css`)

---

## What still needs to be done

Work through these in order. Each section is a discrete commit or step.

### Step 1 — Install dependencies (settgast-ui root)

```bash
cd settgast-ui
pnpm install
```

Picks up `@changesets/cli` added to root devDependencies.

### Step 2 — Initialize Changesets

```bash
pnpm changeset init
```

Then edit `.changeset/config.json` — set `"access": "public"`:

```json
{
  "changelog": "@changesets/cli/changelog",
  "commit": false,
  "linked": [],
  "access": "public",
  "baseBranch": "master",
  "updateInternalDependencies": "patch",
  "ignore": []
}
```

### Step 3 — Verify the build

```bash
cd packages/ui
pnpm build
```

Confirm `dist/` contains:
- `index.js`
- `index.d.ts`
- `index.css`  ← tokens + all scoped component styles
- `tokens.css` ← standalone token file

### Step 4 — Tarball test (validate before any real publish)

```bash
cd packages/ui
npm pack
tar tzf settgast-ui-0.1.0.tgz | sort
```

Must see `package/dist/index.css` and `package/dist/tokens.css` in the listing.
Then do a local install test:

```bash
mkdir /tmp/ui-test && cd /tmp/ui-test
pnpm init
pnpm add /path/to/settgast-ui/packages/ui/settgast-ui-0.1.0.tgz
node -e "const p = require('./node_modules/@settgast/ui/package.json'); console.log(p.exports)"
```

Both `./styles.css` and `./tokens.css` must appear in the output.

### Step 5 — Claim the npm scope (MANUAL — user action required)

The `@settgast` npm scope must be owned before any publish attempt or it will 403.

Go to **npmjs.com/org/create** and create a free org named `settgast`.
(Free orgs can publish unlimited public packages.)

Then authenticate:
```bash
npm login
npm whoami   # should print settgast or show you as a member
```

### Step 6 — Cut the first release

```bash
cd settgast-ui
pnpm changeset          # describe the 0.1.0 release, pick "minor"
pnpm changeset version  # bumps package.json + generates CHANGELOG.md
pnpm release            # runs: pnpm build && changeset publish
```

`changeset publish` calls `npm publish --access public` (covered by `publishConfig`).

### Step 7 — Commit and tag (settgast-ui)

```bash
git add -A
git commit -m "feat: initial publish @settgast/ui@0.1.0"
git tag @settgast/ui@0.1.0
git push && git push --tags
```

### Step 8 — Swap dynamica off the workspace dependency

In `dynamica/apps/dynamica/package.json`, change:
```json
"@settgast/ui": "workspace:*"
```
to:
```json
"@settgast/ui": "^0.1.0"
```

### Step 9 — Remove the local packages/ui copy from dynamica

```bash
rm -rf dynamica/packages/ui
```

Update `dynamica/pnpm-workspace.yaml` — remove `packages/*` if ui was the only package:
```yaml
packages:
  - 'apps/dynamica'
```

Then reinstall:
```bash
cd dynamica
pnpm install
```

Verify the app still builds:
```bash
pnpm build
```

### Step 10 — Commit dynamica

```bash
cd dynamica
git add -A
git commit -m "chore: consume published @settgast/ui instead of local workspace copy"
```

---

## Future: local dev workflow (link mode)

When making library changes and testing them in dynamica before cutting a release:

```bash
# Terminal 1 — in settgast-ui/packages/ui
pnpm dev           # tsup --watch, rebuilds on every save

# dynamica/apps/dynamica/package.json — temporarily set:
"@settgast/ui": "link:../../path/to/settgast-ui/packages/ui"
```

Add this to `dynamica/package.json` to prevent duplicate React:
```json
"pnpm": {
  "overrides": {
    "react": "$react",
    "react-dom": "$react-dom"
  }
}
```

Revert the `link:` back to `^x.y.z` before committing.

---

## Consumer import contract (for docs/README)

```js
import { Button, Card, Badge } from '@settgast/ui';   // components + types
import '@settgast/ui/styles.css';                      // all styles (tokens + components)
// OR
import '@settgast/ui/tokens.css';                      // CSS custom properties only
```

---

## Key decisions already made

| Decision | Choice | Reason |
|----------|--------|--------|
| Registry | npm public | Zero consumer auth; GitHub Packages requires a PAT even for public packages |
| Versioning | Changesets | Automated changelog, standard in monorepos |
| CSS export name | `./styles.css` | Explicit file extension, Node exports convention |
| Scope | `@settgast` | Must create npm org at npmjs.com/org/create first |
