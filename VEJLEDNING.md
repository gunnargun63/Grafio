# Vejledning til Grafio

Grafio er et værktøj til at analysere data fra fysikforsøg: skrive målinger
ind, beregne nye størrelser, lave grafer og finde regression. Her er en
kort gennemgang af det du oftest får brug for.

---

## 1. Indtast data

I tabellen midt på siden skriver du dine målinger.

- Klik i en celle og skriv et tal
- Tryk **piletast** (← ↑ → ↓) for at flytte til nabocellen — det er hurtigere
  end at klikke
- Tryk **Tab** eller **Enter** for at gå videre. Tryk **Enter** i sidste
  række for at oprette en ny række

### Navngiv kolonner

Klik på kolonneoverskriften og skriv fx `tid (s)` eller `U (V)`. Det i
parentes er enheden — den vises automatisk på grafer.

### Indsæt fra Excel eller LoggerPro

Markér flere kolonner i Excel og kopier (⌘+C). Klik på den første celle i
Grafio og indsæt (⌘+V). Alle målingerne kommer ind på én gang.

### Hvordan skrives tal?

- **Decimal:** både komma og punktum virker. `3.14` og `3,14` er det
  samme tal.
- **Store og små tal med eksponent:** skriv `3e3` (eller `3E3`) for
  3000. Skriv `1.5e-4` for 0,00015.
- **Undgå tusindtalsseparatorer.** Skriv `12500`, ikke `12.500` eller
  `12,500` (Grafio vil tolke det som 12,5).

---

## 2. Konstanter

Til venstre for tabellen er der en sidebar med **Konstanter**. Brug den
til faste værdier som tyngdeacceleration, lysets fart, eller en
baggrundstælling der skal trækkes fra alle målinger.

Klik **+ Tilføj konstant**, skriv et symbol (fx `g`), en værdi (fx
`9,82`) og en enhed (fx `m/s²`). Konstanten kan derefter bruges i alle
formler ved sit symbol.

---

## 3. Beregnede kolonner

Klik **+ Tilføj beregnet kolonne** for at lave en kolonne hvor værdierne
udregnes fra andre kolonner og konstanter.

Skriv et symbol (fx `R`), en enhed (fx `Ω`), og en **formel**.

### Hvilke operatorer kan jeg bruge?

- `+` `-` `*` `/` til plus, minus, gange, division
- `^` til potens, fx `t^2`
- Parenteser til at gruppere, fx `(s2 - s1)/(t2 - t1)`

### Hvilke funktioner findes der?

- `sin(x)`, `cos(x)`, `tan(x)` — vinkler er i **grader**
- `asin(x)`, `acos(x)`, `atan(x)` — returnerer grader
- `ln(x)` — naturlig logaritme
- `log(x)` — logaritme med grundtal 10
- `exp(x)` — *e* opløftet i *x*
- `sqrt(x)` — kvadratrod
- `abs(x)` — absolut værdi

### Eksempler

- Modstand fra Ohms lov: `U/I`
- Korrigeret tælletal: `N - N_b` (hvor `N_b` er en konstant)
- Stedet fra konstant acceleration: `0,5*a*t^2`
- Kinetisk energi: `0,5*m*v^2`

---

## 4. Afledet kolonne med `diff()`

Hvis du har målt sted som funktion af tid og vil have hastigheden, så
opret en beregnet kolonne med formlen `diff(s, t)`. Det betyder:
*beregn ændringen i `s` divideret med ændringen i `t` for hver række*.

Det er numerisk differentiation — Grafio kigger på naboværdier i tabellen
og laver forskellen. Det virker bedst når dine målepunkter ligger pænt i
rækkefølge.

`diff()` skal stå alene som hele formlen. Du kan ikke skrive `2*diff(s,t)`
eller bruge `diff` inde i et større udtryk.

---

## 5. Lav en graf

Klik **+ Tilføj graf** under tabellen.

Vælg hvilken kolonne der skal være på **x-aksen** og hvilken på
**y-aksen**. Du kan vælge mellem alle dine kolonner — også beregnede.

Grafen tegnes automatisk.

---

## 6. Regression — find den bedste model

Vælg en **regressionstype** fra dropdown-menuen ved grafen:

- **Proportionalitet** $y = a\cdot x$ — bruges hvor det skal gå gennem 0,0
- **Lineær** $y = a\cdot x + b$ — en almindelig ret linje
- **Andengrad** $y = a\cdot x^2 + b\cdot x + c$ — parabel
- **Potens (heltal)** $y = a\cdot x^n$ — du vælger selv heltallet *n*
- **Potens** $y = a\cdot x^b$ — *b* er en fri eksponent som programmet
  finder
- **Eksponential** $y = a\cdot e^{bx}$ — vækst eller henfald
- **Eksponential med offset** $y = a\cdot e^{bx} + c$ — fx Newtons
  afkøling
- **Invers** $y = a/x$ og **Invers²** $y = a/x^2$ — Coulomb, lyseffekt
  vs. afstand
- **Kvadratrod** $y = a\sqrt{x}$
- **Brugerdefineret** — du skriver selv en formel og startgæt på
  parametrene

Når regressionen er fundet vises **formel, parametre og RMSE** under
grafen. RMSE er et mål for hvor godt fit'et passer — *jo lavere, jo
bedre*. Et lille RMSE betyder at fit-kurven ligger tæt på dine
målepunkter. Dog skal man passe på med fortolkning af selve værdien af RMSE. Den skal ses i sammenhæng med y-aksens værdier.

---

## 7. Tangent, integral og fit-interval

Til hver graf har du tre knapper der kan slås til:

- **Vis tangent** — du skriver en x-værdi, og Grafio tegner tangentlinjen
  og viser dens hældning. Hvis du har en regression, bruges fit-kurven;
  ellers bruges dine datapunkter.
- **Vis integral** — du skriver et interval *[fra, til]*, og Grafio
  skygger arealet under kurven og viser arealets størrelse. Både integral
  af data (trapezregel) og fit (hvis regression er valgt).
- **Vis fit-interval** — du afgrænser hvilke datapunkter regressionen
  bruger. Punkter uden for intervallet tegnes nedtonede og indgår ikke i
  fit'et. Brug det hvis kun en del af dine målinger følger den model du
  fitter.

---

## 8. "Tegn afledet"-knappen

Denne knap er forskellig fra `diff()`-kolonnen. Den opretter en **ny
graf** der viser den afledede af din nuværende graf — altså hvordan
y-størrelsen ændrer sig per x-enhed.

`diff()` regner i tabellen og giver en kolonne du kan bruge videre.
"Tegn afledet" laver bare en graf direkte ud fra punkterne.

---

## 9. Tilpas grafen

- **Akse-værdier** — øverst på grafen er der felter til x og y. Skriv
  præcise grænser eller lad dem stå tomme for automatisk skala.
- **Auto-knap** — nulstiller akserne til automatisk, så datapunkterne fylder hele grafvinduet.
- **Vis origo** — sørger for at både x = 0 og y = 0 er med på grafen
- **Pan og zoom** — træk grafen med musen for at flytte den, scroll for
  at zoome (centreret omkring musen)
- **Forbind punkter** — tegner en linje mellem dine målepunkter (uden om
  regressionskurven). Nyttigt når data følger en kurve uden klar
  matematisk model.

---

## 10. Gem dit arbejde

Tre måder:

- **Gem i browser** — hurtigt og nemt. Virker kun på *denne* computer
  og *denne* browser.
- **Download projekt (.json)** — gemmer alt: tabel, formler, grafer,
  konstanter, indstillinger. Den fil kan du flytte mellem computere
  eller dele med andre.
- **Download data (.csv)** — gemmer kun datatabellen som en regneark-fil
  der kan åbnes i Excel eller LoggerPro.

For at hente noget ind igen: **Indlæs fra browser** eller **Indlæs
projekt (.json)**.

Grafio gemmer også automatisk i baggrunden — du ser tidspunktet ved
siden af knapperne. Hvis du lukker fanen og åbner den igen, er din
seneste version der.

---

## 11. Mørk tilstand

Klik på måne-ikonet øverst til højre for at skifte mellem lys og mørk
tilstand. Valget huskes næste gang du åbner Grafio.

---

## 12. Genveje

| Genvej | Hvad |
|---|---|
| Piletaster | Flyt mellem celler |
| **Tab** | Næste celle |
| **Enter** | Næste række (eller opret ny række) |
| **⌘+Z** / **Ctrl+Z** | Fortryd |
| **⌘+Shift+Z** / **Ctrl+Y** | Gendan |
| **⌘+V** | Indsæt data fra Excel m.fl. |
| Klik checkboks i en række | Slå rækken til/fra (ekskludér fra fit og statistik) |
| **Vis statistik** | Vis middelværdi, spredning og statistisk usikkerhed under hver kolonne |
