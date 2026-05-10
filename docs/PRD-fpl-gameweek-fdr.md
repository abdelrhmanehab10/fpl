# PRD: Official FPL Gameweek FDR Import and Custom Scoring

## Problem Statement

FPL managers need a reliable fixture difficulty view that is based on official Fantasy Premier League fixture data but adapted to this app's own ranking logic. The app currently has no FPL domain data model, importer, scoring rules, or user-facing FDR table. Users cannot compare teams across gameweeks, inspect blanks or doubles, or understand why a fixture is considered easy or difficult.

## Solution

Build a server-side FPL fixture import and custom FDR scoring pipeline. The app will fetch official FPL teams, gameweeks, and fixtures, persist normalized snapshots in Convex, compute one custom FDR record per team per gameweek, and expose a simple user-facing FDR table. The custom scoring system will be deterministic, rules-based, versioned, and explainable. Official FPL difficulty fields will be preserved separately from custom scores so users and developers can compare the official baseline with the app's model.

## User Stories

1. As an FPL manager, I want to see fixture difficulty by team and gameweek, so that I can plan transfers and rotations.
2. As an FPL manager, I want difficulty values to come from official FPL fixture data, so that the table reflects the real fixture schedule.
3. As an FPL manager, I want the app to show custom difficulty rather than only official FPL FDR, so that the table reflects the app's preferred scoring logic.
4. As an FPL manager, I want each team-gameweek to have one comparable difficulty value, so that I can scan gameweeks quickly.
5. As an FPL manager, I want double gameweeks to show that there is more than one fixture, so that I do not confuse them with normal gameweeks.
6. As an FPL manager, I want blank gameweeks to be explicitly marked, so that I do not interpret them as easy fixtures.
7. As an FPL manager, I want fixture difficulty to distinguish home and away matches, so that venue context affects the result.
8. As an FPL manager, I want custom FDR cells to be color-coded, so that easy and hard runs are visible at a glance.
9. As an FPL manager, I want difficulty bands in addition to numeric scores, so that I can understand scores without memorizing thresholds.
10. As an FPL manager, I want numeric custom scores to be available, so that I can sort, compare, or inspect close calls.
11. As an FPL manager, I want the official FPL difficulty to remain available, so that I can compare official and custom difficulty.
12. As an FPL manager, I want double gameweek difficulty to be averaged rather than summed, so that a double does not look harder only because it has two fixtures.
13. As an FPL manager, I want fixture counts to be visible or available, so that I can identify single, double, and blank gameweeks.
14. As an FPL manager, I want a cell's score to have explanation factors, so that I can understand why the app assigned that difficulty.
15. As an FPL manager, I want season support in the data model, so that the app can support current and historical seasons.
16. As an FPL manager, I want the first UI to default to the current imported season, so that I do not need to choose a season when only one exists.
17. As an FPL manager, I want official team names and short names stored with the data, so that the table can render labels without extra API calls.
18. As an FPL manager, I want official gameweek labels stored with the data, so that fixture data remains understandable after import.
19. As an app maintainer, I want official FPL IDs to be canonical, so that imports and joins are reliable.
20. As an app maintainer, I want imports to be idempotent, so that rerunning an import updates data instead of creating duplicates.
21. As an app maintainer, I want derived FDR records to be recomputed safely, so that changes to fixture data or scoring config do not leave stale rows.
22. As an app maintainer, I want scoring configurations to be versioned, so that future scoring changes do not silently rewrite historical meaning.
23. As an app maintainer, I want each computed record to include a scoring version, so that outputs can be traced to the rules that produced them.
24. As an app maintainer, I want the scorer to be a pure module, so that scoring behavior can be tested without Convex or network calls.
25. As an app maintainer, I want importer normalization to be tested, so that official API shape changes or mapping mistakes are caught early.
26. As an app maintainer, I want official fixture difficulty fields preserved exactly, so that custom score differences can be debugged.
27. As an app maintainer, I want manual developer/admin import for v1, so that the data pipeline can be proven before scheduling automation.
28. As an app maintainer, I want the recomputation workflow separated from raw import, so that scoring can be rerun without refetching official data.
29. As an app maintainer, I want a simple user-facing table in v1, so that the imported and computed data can be validated visually.
30. As an app maintainer, I want opportunity scoring to remain out of scope, so that this PRD stays focused on fixture difficulty rather than fantasy upside.
31. As an app maintainer, I want the model to support future scheduled imports, so that automation can be added without reworking the core pipeline.
32. As an app maintainer, I want the model to support future scoring weights, so that recent form or team strength can be added later.

## Implementation Decisions

- Persist official FPL fixture, team, and event snapshots in Convex rather than fetching and calculating directly in the frontend.
- Use official FPL IDs as canonical identifiers for teams, events/gameweeks, and fixtures.
- Store display labels such as team names, team short names, and gameweek labels alongside official IDs.
- Preserve official fixture difficulty fields separately from custom FDR outputs.
- Model custom FDR as one record per season, team, gameweek, and scoring version.
- Allow each team-gameweek FDR record to contain zero, one, or multiple fixture breakdowns.
- Represent blank gameweeks explicitly with a blank flag or blank band rather than a low difficulty score.
- Represent double gameweeks with fixture count and individual fixture breakdowns.
- Aggregate double gameweek difficulty by averaging fixture difficulty in v1.
- Keep opportunity scoring for double gameweeks out of scope for this PRD.
- Implement custom FDR as a deterministic, rules-based scoring module.
- Return both a numeric custom score and a normalized difficulty band.
- Derive difficulty bands from configured thresholds, with the numeric score as the source of truth.
- Include explanation factors with computed results so scores can be inspected.
- Version scoring configuration and stamp computed records with the scoring version.
- Build a manual developer/admin import path for v1.
- Make imports idempotent by official fixture ID and season.
- Make derived FDR recomputation idempotent by season, team, gameweek, and scoring version.
- Separate raw official-data import from FDR recomputation.
- Add season support in schema and query APIs.
- Let the v1 UI default to the current imported season when only one season exists.
- Build a minimal FDR table with teams as rows and gameweeks as columns.
- Make official/custom comparison data available to the UI.
- Defer scheduled import automation until the manual importer and recomputation flow are stable.
- Defer an admin import UI until the command-based import path is proven.

## Testing Decisions

- Good tests should verify external behavior and stable contracts, not implementation details.
- Prioritize pure scoring tests because the scorer is the deepest module and carries the highest domain risk.
- Test blank gameweek behavior to ensure blanks are not treated as easy fixtures.
- Test single home fixture scoring.
- Test single away fixture scoring.
- Test double gameweek aggregation by average difficulty.
- Test fixture count and fixture breakdown output for double gameweeks.
- Test official difficulty preservation during normalization.
- Test scoring-version stamping on computed records.
- Test difficulty-band threshold behavior.
- Test importer normalization from official FPL fixture/team/event payloads into internal shapes.
- Test idempotent import behavior at the module/workflow level where practical.
- Add lighter query or UI tests only for behavior not covered by scoring and normalization tests.
- Use existing Vitest setup for pure TypeScript tests.
- Avoid testing Convex implementation details directly when a pure normalization or scoring module can be tested instead.

## Out of Scope

- Machine learning or statistical prediction.
- Recent form, expected goals, betting odds, player availability, or injury inputs.
- Opportunity scoring for double gameweeks.
- Automated scheduled imports.
- Admin import UI.
- Advanced filtering, transfer planning, or squad-specific recommendations.
- Historical backfills beyond the imported season needed for v1.
- Replacing official FPL data with third-party fixture sources.

## Further Notes

The first implementation should focus on a small number of deep, testable modules: official-data normalization, deterministic FDR scoring, Convex persistence/querying, recomputation, and a minimal table UI. The scoring model should remain transparent and easy to tune rather than pretending to be a prediction engine.
