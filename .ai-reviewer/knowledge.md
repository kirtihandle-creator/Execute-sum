# Execute-sum reviewer notes

## Architecture
This is a Vite + React 18 + TypeScript single-page application, with strict type-checking and no emitted TypeScript artifacts (`package.json`, `tsconfig.json`). `src/App.tsx` acts as a large composition/root module, importing domain-oriented components from `src/components`, editors from `src/editors`, views from `src/views`, and charts from `src/charts`. The sampled chart components are self-contained presentational widgets using local state and deterministic generated data rather than an external data layer.

## Conventions
- Use React function components with named TypeScript interfaces and explicit exported component functions; chart files such as `src/charts/AccountChart.tsx` define `AccountPoint`, `AccountChartMode`, and `AccountChartProps`.
- Component files are grouped by responsibility and domain: reusable cards/forms/tables live under `src/components`, editing workflows under `src/editors`, alternate read-only layouts under `src/views`, and visualization widgets under `src/charts` (`src/App.tsx` imports according to this structure).
- Domain names are propagated consistently through symbols, CSS hooks, and filenames: `AccountChart`, `AccountPoint`, `generateAccountSeries`, and `className="account-chart"` in `src/charts/AccountChart.tsx`; the same pattern appears in `AlbumChart.tsx`, `AlertChart.tsx`, and the other sampled charts.
- Optional component inputs use defaults in the parameter list, e.g. `title = "Account Chart"`, `width = 320`, and `height = 160` in `src/charts/AccountChart.tsx`.
- Chart data is represented as `{ label: string; value: number }[]`; visualization mode is a narrow union (`"bar" | "line"`), and `useMemo` is used for derived series, maxima, and SVG paths.
- Generated demo series use a deterministic local seed and the same linear-congruential calculation across chart files, for example `generateAlbumSeries` in `src/charts/AlbumChart.tsx`. Avoid replacing this with nondeterministic `Math.random()` without an explicit product reason.
- Code must satisfy strict TypeScript settings, including `noUnusedLocals`, `noUnusedParameters`, and `isolatedModules` (`tsconfig.json`). The build runs `tsc --noEmit` before `vite build` (`package.json`).

## Intentional non-standard choices
- Chart implementations are highly repetitive and domain-specific rather than factored into one generic chart component (`src/charts/AccountChart.tsx`, `AlbumChart.tsx`, `AlertChart.tsx`, etc.); preserve domain-specific exports and markup unless consolidating is explicitly requested.
- Chart modules provide both named and default exports, e.g. `export function AccountChart(...)` and `export default AccountChart` in `src/charts/AccountChart.tsx`.
- Demo charts intentionally use deterministic generated data and inline SVG primitives (`src/charts/AccountChart.tsx`), with no charting dependency or API integration visible in the sampled files.

## Watch out for
- Do not introduce unused imports, locals, or parameters; strict compiler settings make these build failures (`tsconfig.json`).
- Preserve stable list keys based on point labels (`key={p.label}` in the chart files), and ensure new generated or supplied data maintains unique labels.
- When changing chart geometry, account for the fixed `PADDING = 24`, `innerW`/`innerH`, and the `max` floor of `1`; invalid dimensions or empty/negative data can produce malformed SVG output (`src/charts/AccountChart.tsx`).
- Keep interactive SVG behavior intact: hover state drives both bar/circle highlighting and tooltip text, while the mode button toggles between bar and line rendering.
- Avoid casually adding runtime dependencies or changing module/build configuration; the repository currently uses only React, React DOM, TypeScript, and Vite (`package.json`).