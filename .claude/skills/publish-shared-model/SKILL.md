---
name: publish-shared-model
description: Bump, publish, and propagate a new version of @ruvice/my-maple-models to its two consumer repos (my-maple-proxy, my-maple). Use when a change to this package needs to actually reach the proxy and/or frontend, not just land in this repo.
---

# Publish shared model and propagate to consumers

`@ruvice/my-maple-models` is consumed by two sibling repos checked out alongside this one: `../my-maple-proxy` and `../my-maple`. A change here is invisible to both until it's published and bumped downstream — publishing to npm is necessary but not sufficient. Work through this checklist in order; don't stop after `npm publish` succeeds.

## 1. Build and verify locally

- `npm run build` — compiles `src/` to `dist/`. This is also effectively the only typecheck this repo has; there's no separate lint or test script.
- Check that `dist/index.d.ts` reflects the intended public API. Only what's re-exported from `src/index.ts` is reachable by consumers — if the change added something to `utils/constants.ts`, it isn't visible downstream unless it's also re-exported via `utils/localisations.ts` or `index.ts` directly.

## 2. Bump and publish

- `npm version patch|minor|major` — patch/minor for additive changes, major if an existing exported shape changes incompatibly.
- `npm publish` (public access already set via `publishConfig`).
- Run `npm install` in this repo right after bumping — `package-lock.json` has drifted out of sync with `package.json`'s version before.

## 3. Propagate to each consumer

For **both** `../my-maple-proxy` and `../my-maple`:

- Bump the `@ruvice/my-maple-models` version in that repo's `package.json`.
- `npm install`.
- `npx tsc --noEmit` — the fastest way to catch a breaking change before it reaches runtime, since neither consumer has a broader test suite.
- If the change affects a raw API response shape or a display string, spot-check the specific file that consumes it:
  - In `my-maple-proxy`: `api/utils/apiResponseParser.ts`, `api/utils/parsers/equipmentParser.ts`.
  - In `my-maple`: wherever `getLocalisedXxx()` or the affected `Character`/`Equipment`/`Stat` field is rendered.

## 4. Commit per repo

This is 3 independent git histories, not a monorepo. A typical change is: one commit here (model change + version bump), and one commit in each consumer (the dependency bump). Don't try to bundle these into a single commit or PR.

## Local iteration without publishing every time

Publishing a throwaway version for every small tweak is wasteful. Consider `npm link`, or temporarily pointing a consumer's `package.json` at `"@ruvice/my-maple-models": "file:../maple-data-models"` — just revert it to the published semver range before committing there.
