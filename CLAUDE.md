# Grafio – CLAUDE.md

Fysik dataanalyseværktøj til gymnasial undervisning (B-niveau primært, A-niveau funktioner inkluderet). Simpel erstatning for LoggerPro/Graphical Analysis. Hostes på GitHub Pages (`https://gunnargun63.github.io/Grafio/`).

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

Kolonne-objekter har felter: `type` ('data' eller 'calc'), `symbol`, `unit`, `formula` (kun calc), og `rawHeader` (rå brugerinput til headerfeltet — se nedenfor).

### Vigtige invarianter
- `snapshotState` / `restoreState` skal holdes synkroniserede: når et nyt graf-felt tilføjes, skal **begge** funktioner opdateres, ellers tabes feltet ved autosave/undo.
- `pushUndo()` kaldes før mutationer; den er idempotent via `lastSnapshot`-sammenligning.
- `rowContext(rowIdx)` bygger evalueringskontekst pr. række; diff-kolonner forhåndsberegnes i `_diffCache` (ryddes ved hvert `render()`).
- `diff(...)` skal stå **alene** i en formel — ikke som del af et større udtryk (fx `2*diff(s,t)` virker ikke).

### rawHeader-mekanismen
Kolonneoverskrifter kan stå i en uafsluttet tilstand under redigering (fx `"R ("` mens brugeren er ved at tilføje en enhed). `parseColHeader` er destruktiv og kasserer alt der ikke kan blive til symbol/enhed — derfor gemmer vi den rå input-streng som `col.rawHeader`. `formatColHeader` bruger `rawHeader` hvis den findes, ellers falder den tilbage til `symbol + (unit)`-rekonstruktion. Det sikrer at uafsluttede headere og særtegn (Ω, μ, osv.) overlever et `render()` uden at blive beskåret.

### Datacelle-visning vs. lagring
Datakolonneceller gemmer altid den rå input-streng i `state.rows[i].values[sym]`, så fuld præcision bevares til beregninger og CSV-eksport. Cellen *viser* dog `displayCellValue(rawStr)` — som afrunder til 4 betydende cifre hvis tallet har mere end 4 cifre, ellers viser strengen uændret. Den rå streng lagres som `data-raw`-attribut på `<input>`. Når brugeren får fokus (`cellOnFocus`), skifter feltet til den rå værdi for redigering; ved blur (`cellOnBlur`) skifter det tilbage til afrundet visning.

### Tegn-palette og fokus-håndtering
`lastFocusedInput` opdateres via en global `focusin`-listener (`trackInputFocus`). Når brugeren klikker en symbol-knap, indsætter `insertSymbol` tegnet på markørens position i det sidst-fokuserede felt. For header-input kalder `insertSymbol` direkte `updateColumnHeader(idx, newValue); render();` i stedet for at stole på at det manuelt dispatchede `change`-event fyrer `onchange`-attributten — det er ikke pålideligt på tværs af browsere.

## Pædagogisk filosofi
Generel arbejdshest — ingen forsøgsskabeloner. Eleverne tænker selv variabler og enheder igennem. Til særligt komplekse forsøg (fx Stefan-Boltzmann) laves separate specialapps.

`diff()` differentierer bevidst altid på de rå datapunkter, ikke på en eventuel regression — også når en regression er beregnet. Det er et didaktisk valg: eleven skal se den faktiske numeriske afledede med dens støj. Lav ikke om på dette uden at spørge.

Grafio starter **friskt** hver gang (ingen automatisk gendannelse af sidste session). Eleverne skal eksplicit bruge "Gem i browser" eller "Download projekt (.json)" som i andre programmer. Auto-resume kan aktiveres ved `?resume=1` i URL'en (udvikler-værktøj).

## Cache og deployment

GitHub Pages serverer index.html med en `Cache-Control`-header der kan holde gamle versioner i cache i op til 10 minutter, og CDN'en (Fastly) cacher også. For at omgå dette:

1. **Meta-tags i `<head>`** (`Cache-Control: no-cache` osv.) — får browseren til at validere mod serveren ved hver indlæsning.
2. **Synligt versions-stempel i UI'en** — opdater `id="build-version"` manuelt før hvert commit, fx `2026-05-21-a`. Bruges til at verificere at den nyeste version faktisk er deployet.
3. **URL-parameter ved deling** — del fx `https://gunnargun63.github.io/Grafio/?v=2026-05-21` med eleverne for at garantere frisk indlæsning.

## Dokumentation

`VEJLEDNING.md` indeholder elev-manualen og er embedded i `index.html` som en HTML-streng i JS-konstanten `VEJLEDNING_HTML` (placeret lige før `closeModal`-funktionen). Indholdet vises i Vejledning-modalen. Hvis `VEJLEDNING.md` opdateres, skal `VEJLEDNING_HTML`-strengen i `index.html` også opdateres manuelt.

## Kopier-graf og dark mode
"Kopiér"-knappen ved en graf eksporterer som PNG med hvid baggrund (så grafen kan indsættes pænt i Word/Docs). I dark mode skifter `copyChart` midlertidigt `data-theme` på `<html>` til light, re-renderer alle grafer med light-farver, kopierer billedet, og skifter tilbage. Det giver et kortvarigt flicker, men det kopierede billede har sort tekst på hvid baggrund.

## Kendte begrænsninger

Pan/zoom på touchpad har let synlig rystelse ved langsomme scroll på VISSE datasæt. Det ser ud til at være regressionstype-afhængigt: andengrad-fit ryster typisk ikke, mens proportionalitet og eksponentiel gør. Datasætstørrelse, beregnede vs. rå kolonner, akse-grænser, og synlighed af plugins er udelukket som årsager.

Følgende optimeringsforsøg har IKKE afhjulpet det:
- requestAnimationFrame-synkronisering
- target-smoothing med lerp
- eliminering af render() under gestus

Det er en accepteret begrænsning, ikke en bug. Tag ikke en ny optimerings-runde før der er kommet (a) ny indsigt i hvorfor regressionstype påvirker det, eller (b) konkret elev-feedback der ændrer prioritet.

## Ideer til fremtidige opgaver

Kopier kolonner til clipboard for nem eksport til andre programmer (mest sandsynligt som per-kolonne knap der kopierer x+aktuel kolonne).
