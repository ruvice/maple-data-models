# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

`@ruvice/my-maple-models` — a dependency-free TypeScript package of shared domain models, raw Nexon Open API response types, and per-server (KMS/SEA/TMS) localisation tables for MapleStory character data. No runtime logic beyond lookup tables and thin dispatch helpers. Published to npm and consumed by two sibling repos.

## Commands

- Build: `npm run build` (runs `tsc`, emits `dist/` with declarations) — this is also the only typecheck available; there is no separate lint or test script/framework.
- Clean: `npm run clean`
- Publish flow (from README.md): `npm run build` → `npm version patch|minor|major` → `npm publish` (public access, per `publishConfig`).

## Architecture

- Two parallel type hierarchies: `src/models/api/*` is the raw Nexon Open API response shape (string-typed fields matching the JSON verbatim), `src/models/maple/*` is the cleaned/typed domain model (enums, numbers) that consumers actually work with. `Character`/`BasicCharacterInfo`/`ItemEquipInfo`/`SymbolInfo`/`StatInfo` in `models/maple/mapleModels.ts` are the final assembled shapes.
- Two separate localisation systems that sound similar but run in opposite directions:
  - `models/api/apiResponseParser.ts` — raw string → enum, per server (`ItemEquipmentSlotMappingSEA/KMS/TMS`, `PotentialMappingSEA/KMS/TMS`, `CharacterClassMappingSEA/KMS/TMS`, `StatMappingSEA/KMS/TMS`), plus `toMapleServer()`. Used when parsing an incoming Nexon API response.
  - `utils/constants.ts` + `utils/localisations.ts` — enum → display string, per server (`CharacterClassLocalisationSEA/KMS/TMS`, `ITEM_EQUIP_STAT_OPTIONS_*`, `SYMBOL_OPTIONS_*`, `STAT_*`), exposed via `getLocalisedCharacterClass()`, `getLocalisedItemEquipOptions()`, `getLocalisedSymbolEquipOptions()`, `getLocalisedStatName()`. Used when rendering UI text.
- `src/index.ts` is the public barrel export. **`utils/constants.ts` is not re-exported** — only the `utils/localisations.ts` dispatch functions are public. A new table added to `constants.ts` isn't reachable by consumers until something re-exports it (or a new dispatch function is added to `localisations.ts`).
- `MapleServer` (KMS/SEA/TMS) is the discriminant threaded through nearly every parsing/localisation function in this package.
- The package builds as ESM (`module: "ESNext"` in tsconfig) — consumers need to be ESM-aware.

## Known inconsistencies

- `package-lock.json` has previously drifted out of sync with `package.json`'s version after a manual `npm version` bump — run `npm install` right after bumping to keep them aligned.
- `tsconfig.json` targets `include: ["src"]`, but several large reference/scratch JSON files sit directly under `src/` (`dualblade.json`, `len equip.json`, `sample response.json`, etc.) alongside real source — not part of the published package (`files: ["dist"]`), just dev-time reference data kept for building the localisation tables.

## Related repositories

This repo publishes the shared package consumed by two sibling repos checked out alongside this one:

- **`../my-maple-proxy`** (GitHub `ruvice/my-maple-proxy`) — Vercel edge-function proxy that parses raw Nexon responses using this package's `api/*` types and `apiResponseParser.ts` mappings.
- **`../my-maple`** (GitHub `ruvice/my-maple`) — React/Twitch-extension frontend that renders the `maple/*` domain types and uses `utils/localisations.ts` for display strings.

Changing a type or mapping table here does not automatically update either consumer — publishing and propagating is a separate, deliberate step. Use the `publish-shared-model` skill (`.claude/skills/publish-shared-model/`) for the full bump → publish → propagate checklist rather than improvising it.

## Agent skills

### Issue tracker

Issues and specs live as markdown files under `.scratch/<feature-slug>/`. See `docs/agents/issue-tracker.md`.

### Triage labels

Default five canonical labels (`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`). See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: `CONTEXT.md` + `docs/adr/` at the repo root. See `docs/agents/domain.md`.
