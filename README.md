# Grafio

Et lille webbaseret dataanalyse-værktøj til fysikundervisning på gymnasialt niveau. Eleverne indtaster måleværdier, opretter beregnede kolonner og laver grafer med regression — alt sammen i én HTML-fil der kører lokalt i browseren.

Tænk på det som en simpel udgave af LoggerPro eller Graphical Analysis, skræddersyet til danske fysikforsøg.

## Brug

Åbn appen direkte i browseren:

**[Åbn Grafio](https://YOUR-USERNAME.github.io/REPO-NAME/)**

*(Erstat URL'en ovenfor med din egen GitHub Pages-adresse når du har sat repoet op.)*

Ingen installation, ingen login. Appen autosaver løbende i browseren, så eleverne kan komme tilbage til deres arbejde.

## Hvad appen kan

**Datatabel**
- Indtast x- og y-værdier (og flere kolonner efter behov)
- Beregnede kolonner med formler (fx `U/I`, `sin(i)`, `N - N_baggrund`)
- Indsæt data direkte fra Excel/Sheets med kopiér-paste
- Naviger med piletaster og Enter
- Slå enkelte rækker fra hvis de skal udelades fra regressionen

**Konstanter**
- Definér konstanter (fx baggrundstælletal, kalibreringsværdier) der kan bruges i formler
- Vises som diskret sidebar — fylder ikke når der ikke er konstanter

**Grafer med regression**
- Tilføj flere grafer; vælg selv x-akse, y-akse og regressionstype
- Regressionstyper: proportionalitet, lineær, andengrad, potens, eksponential, forskudt eksponential, invers (med og uden offset), afstandskvadrat (med og uden offset), kvadratrod
- Brugerdefineret regression: skriv din egen formel med navngivne parametre (Gauss-funktion, harmonisk svingning, dæmpet svingning osv.)
- Residualplot ved hver graf
- Pile-akser ved origo i klassisk fysik-stil
- Manuelle akse-grænser hvis ønsket
- "Kopiér graf"-knap der lægger grafen som billede i udklipsholderen til indsætning i rapporter

**Data-import og -eksport**
- CSV-import (fungerer med eksport fra Excel, Google Sheets, Graphical Analysis osv.)
- CSV-eksport
- Komplet projekt som JSON (til backup eller deling)
- Lokal autosave + navngivne forsøg i browseren

**Symbol-palette**
- Klik på "Ω Tegn" for at indsætte græske bogstaver (α, β, γ, … ω) og specialtegn (°, ², ³, µ, Å, π, …)

## Hosting på GitHub Pages

Sådan får du appen op at køre på en fast URL:

1. **Lav et nyt repository** på GitHub. Et oplagt navn er `grafio`.
2. **Upload `index.html` og `README.md`** til rodet af repoet (træk filerne ind via GitHub's webgrænseflade, eller brug `git push`).
3. **Aktivér GitHub Pages**: gå til repository-indstillinger → Pages → vælg branchen `main` og mappen `/ (root)` → gem.
4. Efter et par minutter er appen tilgængelig på `https://[dit-brugernavn].github.io/grafio/`.
5. Opdater URL'en i denne README og giv den til eleverne.

**Opdateringer**: ret i `index.html` på GitHub (eller push fra din computer). Eleverne ser den nye version næste gang de åbner siden.

## Bemærkninger til dataopbevaring

- **Autosave** og **navngivne forsøg** ligger i browserens localStorage og er knyttet til den specifikke browser på den specifikke enhed. Skifter eleven computer, mister hun sin autosave.
- For at flytte arbejde mellem enheder skal eleven **downloade projekt-filen** (`.json`) og indlæse den på den nye enhed.
- For aflevering kan eleven **downloade en CSV** med rådata eller **kopiere graferne** som billeder.

## Teknisk

- Én selvstændig HTML-fil med inline CSS og JavaScript
- Brug af Chart.js (CDN) til grafer
- Ingen build-proces, ingen afhængigheder ud over Chart.js
- Fungerer i alle moderne browsere (Chrome, Safari, Firefox, Edge)

## Udvikler-tip

Hvis du tester en ny version og browseren stadig viser din gamle autosave, så tilføj `?fresh=1` til URL'en. Appen starter da med en blank tilstand uden at læse autosaven.
