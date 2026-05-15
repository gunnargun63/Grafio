# Grafio – CLAUDE.md

Fysik dataanalyseværktøj til gymnasial undervisning (B-niveau primært, A-niveau funktioner inkluderet). Simpel erstatning for LoggerPro/Graphical Analysis. Skal hostes på GitHub Pages.

## Faste rammer — må ikke brydes

- **Én fil:** Al kode bor i `index.html`. Ingen separate `.js`- eller `.css`-filer, ingen bundler, intet build-step.
- **Kun Chart.js fra CDN:** `https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js`. Ingen andre eksterne biblioteker.
- **UI på dansk:** Alle knapper, labels, fejlbeskeder og hjælpetekster skal være på dansk.
- **Syntakstjek efter hver ændring:** Udtræk JavaScript fra `<script>`-tags og tjek syntaks med:
  ```
  node -e "const h=require('fs').readFileSync('index.html','utf8');(h.match(/<script>([\s\S]*?)<\/script>/g)||[]).forEach((b,i)=>{try{new Function(b.replace(/<\/?script>/g,''))}catch(e){console.error('Blok',i+':',e.message);process.exit(1)}});console.log('Syntaks OK')"
  ```
  Ret eventuelle fejl inden du rapporterer opgaven som løst.

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

`diff()` differentierer bevidst altid på de rå datapunkter, ikke på en eventuel regression — også når en regression er beregnet. Det er et didaktisk valg: eleven skal se den faktiske numeriske afledede med dens støj. Lav ikke om på dette uden at spørge.

## Kendte begrænsninger

Pan/zoom på touchpad har let synlig rystelse ved meget langsomme scroll. Det er en accepteret begrænsning, ikke en bug. Flere optimeringsforsøg (render-eliminering, requestAnimationFrame-synkronisering, target-smoothing med lerp) har IKKE afhjulpet det — det er sandsynligvis en grundlæggende konsekvens af diskrete browser-events kombineret med en HTML-canvas-baseret graf. Tag ikke runde 4 før der er kommet ny indsigt eller elev-feedback der ændrer prioritet.
