# Design Sync Notes

## Build commands (pnpm monorepo with pnpm hoisted deps)

```bash
# 1. Build the package
pnpm -F "@settgast/ui..." build

# 2. Rebuild ds-bundle (react+react-dom live in packages/ui/node_modules via pnpm symlinks)
node .ds-sync/package-build.mjs \
  --config .design-sync/config.json \
  --node-modules packages/ui/node_modules \
  --entry packages/ui/dist/index.js \
  --out ./ds-bundle \
  --storybook-static .design-sync/sb-reference

# 3. Validate
node .ds-sync/package-validate.mjs ./ds-bundle

# 4. Compare (all or scoped)
node .ds-sync/storybook/compare.mjs \
  --out ./ds-bundle \
  --storybook-static .design-sync/sb-reference \
  [--components Button,StatCard,...]
```

## CSS module scoping

tsup/esbuild does not scope CSS module class names — it uses identity maps and emits plain
`.button`, `.primary` etc. across ALL components. Multiple components sharing a class name
(e.g. Button and Pagination both have `.button`) cause cascade collisions.

Fix in `packages/ui/tsup.config.ts` `onSuccess`: prefix every class name with the module
name (`.button` → `.Button__button`), rename the selectors in the CSS per-section, and
patch the JS class map. Do NOT revert to identity maps.

## Known non-blocking warnings

- `[RENDER_THIN] Sparkline` — needs `.design-sync/previews/Sparkline.tsx` (pure SVG, no text)
- `[REFERENCE_STALE?]` — appears when bundle changes but storybook reference didn't; safe to
  ignore when only the build tooling (tsup/CSS scoping) changed, not the component source

## Re-sync risks

- **Sparkline `[RENDER_THIN]`** — still present; always will be unless `.design-sync/previews/Sparkline.tsx`
  is authored with a wrapper that adds visible text. Compare graded it `match` from images (SVG lines
  render correctly); the warn is understood and non-blocking.
- **CSS module scoping (tsup.config.ts `onSuccess`)** — any new component whose `.module.css` introduces
  a class name used elsewhere (e.g. `.wrapper`, `.container`) will be safe because per-module prefixing
  is automatic. Only risk: if `tsup.config.ts` is reverted to the old identity-map approach, cascade
  collisions return. Keep the `onSuccess` block.
- **Table Watchlist story** — graded `match`; story data is a fixed `const` array (not seeded). Preview
  captured values match storybook exactly (BTC/USD $67,420, ETH/USD $3,210, SOL/USD $142, AAPL $189.42).
- **pnpm hoisting** — `react` and `react-dom` live in `packages/ui/node_modules` (pnpm symlinks).
  If the project is re-installed with a different node_modules layout, the ds-bundle build command's
  `--node-modules packages/ui/node_modules` flag may need updating.
- **Google Fonts** — Inter and JetBrains Mono are imported from `fonts.googleapis.com` in `tokens.css`.
  Compare grading assumed network access. If the runner is sandboxed, font fallbacks will appear but
  grades will still be structurally correct.
