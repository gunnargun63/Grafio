# Grafio – CLAUDE.md

Fysik dataanalyseværktøj til gymnasial undervisning (B-niveau primært, A-niveau funktioner inkluderet). Simpel erstatning for LoggerPro/Graphical Analysis. Skal hostes på GitHub Pages.

## Faste rammer — må ikke brydes

- **Én fil:** Al kode bor i `index.html`. Ingen separate `.js`- eller `.css`-filer, ingen bundler, intet build-step.
- **Kun Chart.js fra CDN:** `https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js`. Ingen andre eksterne biblioteker.
- **UI på dansk:** Alle knapper, labels, fejlbeskeder og hjælpetekster skal være på dansk.
- **Syntakstjek efter hver ændring:** Kør `node --check index.html` og ret eventuelle fejl, før du rapporterer opgaven som løst.

## Arkitektur

### State
`state`-objektet holder: `constants`, `columns`, `rows`, `graphs`, `showStats`, `_graphIdCounter`.

Graf-objekter har felter: `id`, `xSym`, `ySym`, `regType`, `customExpr`, `customParams`, `xMin/xMax/yMin/yMax`, `showIntegral`, `integralFrom/To`, `showTangent`, `tangentX`, `showFitRange`, `fitFrom/To`, `connectPoints`.

### Vigtige invarianter
- `snapshotState` / `restoreState` skal holdes synkroniserede: når et nyt graf-felt tilføjes, skal **begge** funktioner opdateres, ellers tabes feltet ved autosave/undo.
- `pushUndo()` kaldes før mutationer; den er idempotent via `lastSnapshot`-sammenligning.
- `rowContext(rowIdx)` bygger evalueringskontekst pr. række; diff-kolonner forhåndsberegnes i `_diffCache` (ryddes ved hvert `render()`).
- `diff(...)` skal stå **alene** i en formel — ikke som del af et større udtryk (fx `2*diff(s,t)` virker ikke).

## Pædagogisk filosofi
Generel arbejdshest — ingen forsøgsskabeloner. Eleverne tænker selv variabler og enheder igennem. Til særligt komplekse forsøg (fx Stefan-Boltzmann) laves separate specialapps.
