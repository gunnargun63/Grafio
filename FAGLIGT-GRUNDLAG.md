# Grafio — Dokumentation

En dokumentation til lærere af det matematiske og numeriske grundlag for
Grafio. Formålet er at lærere kan bedømme om værktøjet er solidt nok til
deres undervisning.

Denne manual antager fysikfaglig og matematisk baggrund på universitetsniveau.
Den er ikke en brugsanvisning til appen.

---

## 1. Formål og pædagogisk udgangspunkt

Grafio er bygget som en **generel arbejdshest** til dataanalyse i fysik på
gymnasieniveau. Designvalgene er trukket i denne retning:

- Eleverne bygger selv tabellen og navngiver kolonner og enheder.
- Beregnede kolonner skrives som matematiske udtryk, ikke valgt fra menuer.
- Regressionsformler vises altid eksplicit med deres parametre.
- Tallene rapporteres med et fast antal betydende cifre (4), ikke med
  usikkerhedsestimater.
- `diff()`-funktionen bruger altid de rå datapunkter, aldrig en fitkurve,
  så eleven ser den faktiske numeriske afledede med dens støj.

Programmet er bevidst skrevet i ét enkelt HTML-dokument uden eksterne
afhængigheder bortset fra Chart.js (til visualisering). Det betyder det
kan udleveres som én fil og virker i enhver moderne browser, også offline.

---

## 2. Datamodel

### Kolonner

Tre typer:

- **Datakolonner** — indtastede måleværdier. Hver række kan have en værdi
  eller være tom; tomme værdier giver `NaN` i beregninger.
- **Beregnede kolonner** — en formel der evalueres pr. række. Kan referere
  til andre datakolonner, beregnede kolonner, og konstanter ved deres
  symbol.
- **Konstanter** (sidebar) — navngivne værdier som `g = 9.82`, der kan
  bruges i alle formler.

Hver række har en `included`-flag (checkbox). Ekskluderede rækker tegnes
nedtonet på grafer og indgår ikke i regression, statistik, eller
numerisk differentiation.

### Formelevaluering

Formler følger almindelig matematisk operatorprioritet.

Understøttede operatorer:
- `+ - * /`
- `^` (potens)
- Unær `-`

Understøttede funktioner:
- Trigonometri i **grader**: `sin(x)`, `cos(x)`, `tan(x)`, `asin(x)`, `acos(x)`, `atan(x)`
- `ln(x)` (naturlig logaritme), `log(x)` (10-tal), `exp(x)`
- `sqrt(x)`, `abs(x)`

Trigonometriens grader-konvention er valgt fordi det egner sig bedere til forsøg på fysik B.
Hvis der er behov for radianer i en specifik formel, kan eleven selv konvertere.

---

## 3. Regression — fælles principper

Regressionerne bygger på **mindste kvadraters metode**. For modeller der
fittes direkte i de oprindelige variable (proportionalitet, lineær,
andengrad, og ikke-lineære fits via Nelder-Mead) minimeres summen af
kvadrerede residualer i $y$-retningen:

$$
\text{RSS} = \sum_{i=1}^{n} \bigl(y_i - \hat{y}_i\bigr)^2
$$

For modeller der lineariseres ved en transformation (potens, eksponential,
invers, kvadratrod og potens-heltal) findes parametrene ved mindste
kvadrater i det **transformerede** rum, fx ved at minimere
$\sum_i (\ln y_i - \ln \hat{y}_i)^2$ for potens- og eksponentialfit.
Det er en bevidst forenkling: lukket-form-løsning frem for iterativ
optimering. Konsekvensen er at fit'et giver lidt anderledes parametre
end et "rigtigt" ulineært fit i $y$-rummet ville, særligt hvis datasættet
har stort dynamisk omfang. For typisk fysik-data på gymnasieniveau er
forskellen sjældent stor nok til at have betydning.

**RMSE rapporteres altid i det oprindelige $y$-rum**, også for de
transformerede modeller — altså i samme enheder som målingerne. Det er
gjort for at tallet er sammenligneligt på tværs af regressionstyper og
let at fortolke for eleverne. Se afsnit 6 for detaljer.

For ikke-lineære modeller (eksponential med offset, invers med offset,
brugerdefineret) bruges Nelder-Mead simplex-optimering — se afsnit 5.

### Transformationsstrategi

For at undgå iterativ optimering hvor det ikke er nødvendigt, lineariserer
Grafio følgende modeller før den lineære fit:

| Model | Transformation | Fittes som |
|---|---|---|
| $y = a \cdot x^b$ (potens, fri eksponent) | $\ln y = \ln a + b \ln x$ | Lineær på $\ln x$, $\ln y$ |
| $y = a \cdot e^{bx}$ (eksponential) | $\ln y = \ln a + bx$ | Lineær på $x$, $\ln y$ |
| $y = a/x$ | $u = 1/x$ | Proportionalitet $y = a \cdot u$ |
| $y = a/x^2$ | $u = 1/x^2$ | Proportionalitet $y = a \cdot u$ |
| $y = a\sqrt{x}$ | $u = \sqrt{x}$ | Proportionalitet $y = a \cdot u$ |
| $y = a \cdot x^n$ (heltal $n$, fast) | $u = x^n$ | Proportionalitet $y = a \cdot u$ |

Modeller med en konstant tilføjet (eksponential med offset, $1/(x+b)$ osv.)
kan ikke lineariseres direkte og fittes derfor med iterativ optimering.

### Krav til datapunkter

Forskellige transformationer kræver forskellige forudsætninger:

- **Eksponentialfit** ($y = a \cdot e^{bx}$): $y > 0$ for alle inkluderede
  punkter. Punkter med $y \leq 0$ udelades automatisk fra fit'et.
- **Potensfit med fri eksponent** ($y = a \cdot x^b$): både $x > 0$ og
  $y > 0$. Punkter der ikke opfylder begge betingelser udelades.
- **Eksponential med offset**: ingen forudsætning, men der laves et
  grid-search over offset-værdien $c$ — se afsnit 5.
- **Inverse** ($y = a/x$, $y = a/x^2$): $x \neq 0$. Punkter med $x = 0$
  udelades.
- **Kvadratrod**: $x \geq 0$ for alle punkter. Hvis nogen punkter har
  $x < 0$, afvises hele fit'et.
- **Potens (heltal) med $n < 0$**: $x \neq 0$. Punkter med $x = 0$ udelades.

Når punkter udelades pga. kravene, vises antal gyldige punkter i
resultatboksen sammen med RMSE. Hvis kravene betyder at færre end 2
punkter er gyldige, kan regressionen ikke beregnes, og eleven får en
fejlbesked.

---

## 4. De enkelte regressionstyper

### Proportionalitet: $y = ax$

Lukket løsning. Med vægten på at fit'et går gennem origo:

$$
a = \frac{\sum x_i y_i}{\sum x_i^2}
$$

Antal frie parametre: $k = 1$.

### Lineær: $y = ax + b$

Standard mindste kvadraters lineær regression:

$$
a = \frac{n\sum x_i y_i - \sum x_i \sum y_i}{n \sum x_i^2 - (\sum x_i)^2}, \qquad
b = \frac{\sum y_i - a\sum x_i}{n}
$$

Antal frie parametre: $k = 2$.

### Andengrad: $y = ax^2 + bx + c$

Lukket løsning via normalligningerne for polynomiel regression af grad 2
(3×3-system løses med Cramers regel). Antal frie parametre: $k = 3$.

Højere polynomielle grader er bevidst udeladt for at undgå at eleverne
"over-fitter" lavt-støj-data med polynomier af høj grad, hvilket sjældent
giver fysikalsk mening på gymnasieniveau.

### Eksponential: $y = a \cdot e^{bx}$

Lineariseret via $\ln y = \ln a + bx$. Kræver $y > 0$. Antal frie
parametre: $k = 2$.

### Eksponential med offset: $y = a \cdot e^{bx} + c$

Ikke-lineær. Strategi:

1. Grid-search over $c$: 60 værdier i intervallet $[\min(y) - 2(\max-\min),\ \min(y))$,
   plus et tilsvarende interval over $\max(y)$ for tilfælde med $a < 0$
2. For hvert $c$: lineariser de resterende parametre via $\ln(y - c)$ og
   lineær fit
3. Vælg det bedste $c$, og forfin alle tre parametre samtidigt med
   Nelder-Mead

Antal frie parametre: $k = 3$.

Dette er den standardmodel for Newtons afkølingslov og lignende
asymptotisk tendens.

### Potens (fri): $y = a \cdot x^b$

Lineariseret via $\ln y = \ln a + b \ln x$. Kræver $x > 0$ og $y > 0$.
Antal frie parametre: $k = 2$.

### Potens (heltal): $y = a \cdot x^n$

Eleven vælger $n \in \{-5, ..., -1, 1, ..., 5\}$. Med $n$ fast bliver
modellen lineær i $a$: substitution $u = x^n$ giver $y = a \cdot u$, og
proportionalitetsfit kan bruges.

Krav: hvis $n < 0$ skal $x \neq 0$ for alle punkter; rækker med $x = 0$
udelades. For $n > 0$ er der ingen krav (også negative $x$ er tilladt,
fordi heltallige eksponenter altid er definerede).

Antal frie parametre: $k = 1$.

Denne regressionstype er særligt nyttig når fysikken forudsiger en bestemt
potenslov (afstandskadrat $\sim 1/r^2$, stedfunktion i frit fald, Stefan-Boltzmanns lov),
så eleven kan teste den specifikke model snarere end at lade en fri
eksponent absorbere systematiske fejl.

### Invers: $y = a/x$ og $y = a/x^2$

Substitution $u = 1/x$ eller $u = 1/x^2$, derefter proportionalitetsfit.
Krav: $x \neq 0$. Antal frie parametre: $k = 1$.

### Invers med offset: $y = a/(x+b)$ og $y = a/(x+b)^2$

Ikke-lineære. Strategi: grid-search over $b$, lineær fit af $a$ for hvert
$b$, derefter golden-section-lignende forfining af $b$. Antal frie
parametre: $k = 2$.

### Kvadratrod: $y = a\sqrt{x}$

Substitution $u = \sqrt{x}$, derefter proportionalitetsfit. Krav:
$x \geq 0$ for alle punkter. Antal frie parametre: $k = 1$.

### Brugerdefineret

Eleven skriver en formel som `a*exp(-((x-b)/c)^2)` med startgæt for
parametrene $\{a, b, c\}$. Modellen fittes via Nelder-Mead simplex
direkte på RSS.

For ikke-trivielle modeller (Gauss-funktion, Lorentz, harmonisk svingning) er
startgættet afgørende for konvergens. Nelder-Mead er robust men kan
finde lokale minima hvis startgættet er meget forkert. I lærer­manualen
er det værd at instruere elever i at gætte $a, b, c$ omtrent rigtigt
før de starter fit'et.

---

## 5. Nelder-Mead simplex

Til ikke-lineære regressionsproblemer bruges Nelder-Mead-algoritmen
(opkaldt efter dens opfindere fra 1965). Det er en gradient-fri
optimeringsmetode, hvilket betyder den ikke kræver analytiske afledede
af modelfunktionen — kun evaluering. Det gør den særligt velegnet til
brugerdefinerede modeller hvor afledte ville være besværlige at få ud.

Implementationen følger den klassiske Nelder-Mead med standardkoefficienter:

| Operation | Parameter | Værdi |
|---|---|---|
| Refleksion | $\alpha$ | 1.0 |
| Ekspansion | $\gamma$ | 2.0 |
| Kontraktion | $\rho$ | 0.5 |
| Krympning | $\sigma$ | 0.5 |

Konvergens når intervallet af RSS-værdier i simplex'en bliver mindre end
$10^{-10}$ (eller $10^{-12}$ for brugerdefineret), eller når 2000 iterationer
er opnået.

Algoritmen er kendt for at konvergere langsommere end gradient-baserede
metoder, og den har ikke samme konvergensgarantier som specialiserede
least-squares-metoder som Levenberg-Marquardt. Men den er robust mod
numerisk støj og dårligt betingede problemer, og den kræver kun at
modelfunktionen kan evalueres — ingen analytiske afledede. Det gør den
til et rimeligt og praktisk valg for et undervisnings­værktøj hvor
brugerne kan definere vilkårlige modeller og hvor pæne startgæt ikke
kan forventes.

---

## 6. RMSE

Grafio rapporterer **Root Mean Square Error** for hver regression:

$$
\text{RMSE} = \sqrt{\frac{1}{n}\sum_{i=1}^{n}(y_i - \hat{y}_i)^2}
$$

Bemærk delingen med $n$, **ikke** $n-k$ (hvor $k$ er antal parametre).

### Begrundelse for valget af $n$

RMSE i sin oprindelige definition er præcis "kvadratroden af gennemsnittet
af kvadrerede fejl" — altså med $n$ i nævneren. Det er det navnet siger.

Visse statistiske værktøjer (R, gnuplot, Excels `LINEST`) rapporterer
i stedet *Residual Standard Error* (RSE) eller *Standard Error of the
Regression*, som er:

$$
s = \sqrt{\frac{1}{n-k}\sum_{i=1}^{n}(y_i - \hat{y}_i)^2}
$$

RSE er den sædvanlige estimator for residualernes (og dermed støjens)
standardafvigelse under antagelse af at modellen er korrekt og at
residualerne har samme varians. Den er statistisk mere meningsfuld
hvis man vil bruge tallet til at konstruere konfidensintervaller for
parametrene.

For gymnasiebrug er **RMSE** den mere intuitive størrelse: den siger
direkte "gennemsnitlig afvigelse i kvadrat-forstand", og eleven kan
sammenligne den med y-værdiernes typiske størrelsesorden for at vurdere
om fit'et er rimeligt. Den passer også til hvordan Logger Pro og GeoGebra
rapporterer, hvilket er de andre værktøjer eleverne sandsynligvis møder.

For at konvertere mellem dem:

$$
s = \text{RMSE} \cdot \sqrt{\frac{n}{n-k}}
$$

For $n \gg k$ er forskellen lille; for $n$ tæt på $k$ bliver $s$ betydeligt
større end RMSE.

### Hvad RMSE *ikke* fortæller

RMSE alene siger ikke om en fit er god — den skal sammenholdes med
y-værdiernes spredning. Et fit med $\text{RMSE} = 0.1$ er fremragende hvis
$y$ varierer fra 0 til 100, men dårligt hvis $y$ kun varierer fra 0 til
0.2. Læreren kan introducere en normaliseret størrelse som
$\text{RMSE} / (y_{\max} - y_{\min})$ til eleverne hvis de skal vurdere
fit-kvalitet uafhængigt af enhederne.

---

## 7. Statistik pr. kolonne

Når "Vis statistik" er slået til, vises tre rækker under hver kolonne:

### Middelværdi

$$
\bar{x} = \frac{1}{n}\sum_{i=1}^{n} x_i
$$

Beregnet kun over inkluderede rækker med endelige værdier.

### Spredning (empirisk standardafvigelse)

$$
s = \sqrt{\frac{1}{n-1}\sum_{i=1}^{n}(x_i - \bar{x})^2}
$$

Bemærk **Bessels korrektion** med $n-1$ — dette er den empiriske
standardafvigelse, ikke populations-standardafvigelsen. Det er det
rigtige valg når data er et stikprøveudtag (hvilket næsten altid er
tilfældet i fysikforsøg). Kræver $n \geq 2$.

### Statistisk usikkerhed på middelværdien

$$
\text{SEM} = \frac{s}{\sqrt{n}}
$$

Dette er *Standard Error of the Mean* — usikkerheden på middelværdien
selv, ikke på de enkelte målinger.

**Forudsætningen for at SEM kan tolkes som usikkerheden på middel­værdien**
er at målingerne er uafhængige gentagelser af samme størrelse, og at
eventuelle systematiske fejl er negligible. Hvis instrumentet har en
fast bias, hvis målingerne påvirker hinanden, eller hvis der er drift
under måleserien, fanger SEM kun den tilfældige spredning — ikke den
faktiske usikkerhed på resultatet. Det er en pointe der ofte misforstås
af gymnasieelever, som let tror at $s/\sqrt{n}$ er "hele usikkerheden".

Bemærk forskellen i nævneren mellem spredningen ($n-1$) og RMSE ($n$):
de to størrelser har forskellige formål, og forskellen er bevidst.
Spredningen er en estimator af populations­spredningen og bruger derfor
Bessels korrektion. RMSE er beskrivende statistik for kvaliteten af
en model-fit og bruger den oprindelige definition.

---

## 8. Numerisk differentiation

Grafio understøtter numerisk differentiation på to måder:

### 8a. `diff()`-funktion i en beregnet kolonne

En kolonne med formlen `diff(y, x)` beregner d$y$/d$x$ for hver række
ved **central forskel**:

$$
\left(\frac{dy}{dx}\right)_i \approx \frac{y_{i+1} - y_{i-1}}{x_{i+1} - x_{i-1}}
$$

Bemærk:

- For det første og sidste punkt bruges henholdsvis fremad- og baglæns
  forskel:
  $$\left(\frac{dy}{dx}\right)_0 \approx \frac{y_1 - y_0}{x_1 - x_0}$$
- Ekskluderede rækker springes over: hvis række $i-1$ er ekskluderet,
  bruges $i-2$ som forrige nabo. Det betyder at en enkelt ekskluderet
  måling ikke ødelægger den afledede ved nabopunkter.
- Hvis $x_{i+1} = x_{i-1}$ (deling med nul) eller en af værdierne er
  `NaN`, gives `NaN` for den række.

**Vigtig pædagogisk pointe:** `diff()` regner altid på de rå datapunkter,
**aldrig** på en eventuel regressionsmodel. Dette er et bevidst
didaktisk valg: eleven ser den faktiske numeriske afledede med dens
støj, hvilket illustrerer at numerisk differentiation forstærker måle­fejl.

`diff()` af en beregnet kolonne (fx `diff(T_kor, t)` hvor `T_kor` er
en formel-kolonne) understøttes også, med korrekt håndtering af
cirkulære referencer.

### 8b. Tangent på en graf

Knappen "Vis tangent" tegner en kort tangentlinje ved et brugervalgt
$x_0$. Hvis en regression er valgt, bruges numerisk differentiation af
fit-funktionen:

$$
f'(x_0) \approx \frac{f(x_0 + h) - f(x_0 - h)}{2h},
\qquad h = 10^{-5} \cdot \max(|x_0|, 1)
$$

Hvis ingen regression er valgt, bruges central forskel på de to
nærmeste datapunkter omkring $x_0$.

Tangentlinjens hældning vises med korrekte enheder, fx $\Omega/\text{m}$
hvis $y$ er modstand og $x$ er længde.

---

## 9. Numerisk integration

Knappen "Vis integral" tegner et integral-område på grafen og rapporterer
to tal:

### 9a. Integral af data

Trapezreglen anvendt på den **inkluderede** datasekvens i intervallet
$[a, b]$:

$$
\int_a^b y\, dx \approx \sum_{i=0}^{m-1} \frac{(y_i + y_{i+1})}{2}(x_{i+1} - x_i)
$$

Endepunkterne $a$ og $b$ håndteres ved lineær interpolation mellem
nabopunkter: hvis brugeren vælger $a$ mellem to datapunkter, indsættes
et virtuelt punkt $(a, y_{\text{interp}}(a))$ som første trapez-grænse,
og tilsvarende for $b$.

Trapezreglen er valgt frem for Simpsons regel for data fordi
sidstnævnte forudsætter jævnt fordelte $x$-værdier, hvilket reelle
måleserier ikke altid har.

### 9b. Integral af fit-funktion

Hvis en regression er valgt, beregnes også integralet af fit-kurven
med **Simpsons regel**:

$$
\int_a^b f(x)\, dx \approx \frac{h}{3}\left[f(x_0) + 4f(x_1) + 2f(x_2) + 4f(x_3) + ... + f(x_n)\right]
$$

med $n = 1000$ underintervaller og $h = (b-a)/n$.

Simpsons regel er valgt for fit-funktioner fordi vi har en analytisk
funktion at evaluere på vilkårlige punkter, så $n$ kan vælges højt og
fordelingen er garanteret jævn.

Begge tal vises samtidigt, så eleven kan sammenligne dem og se hvordan
modellen "udjævner" datapunkterne i integralet.

---

## 10. Visualisering — relevante detaljer

### Akseskalering

Grafer beregner som default en "pæn" akseskala der inkluderer alle
datapunkter plus et margen på 5% (x-akse) eller 8% (y-akse). Brugeren
kan overskrive med eksplicitte min/max-værdier.

### Pan og zoom

Musen (touchpad eller scroll-hjul) zoomer omkring markørens position.
Træk panner. Hver zoom/pan-gestus tæller som ét undo-trin.

### Ekstrapolation

Hvis brugeren udvider akseintervallet ud over data-området, tegnes
fit-kurven *uden for data* som en stiplet og lysere version, så det
visuelt fremgår at modellen ikke er valideret der.

### Residualplot

Til hver graf vises et separat residualplot, $y_i - \hat{y}_i$ som
funktion af $x_i$. Når fit-interval er aktivt, vises kun residualer
for de punkter regressionen faktisk hviler på, så residualplottet
ikke domineres af punkter uden for fit-området.

---

## 11. Hvad Grafio *ikke* gør

For ærlighed: Grafio er en undervisnings-app, ikke et komplet
statistik­værktøj. Følgende er **ikke** implementeret:

- **Usikkerheder på regressionsparametre.** Der rapporteres kun
  punktestimater, ikke standardfejl eller konfidensintervaller på
  parametrene. Hvis eleverne skal vurdere usikkerhed på fx en hældning,
  må det ske ved en særskilt usikkerhedsanalyse — fx ved at gentage
  måleserien flere gange og se hvordan hældningen varierer, ved manuel
  vurdering af "plausible fits" gennem datapunkterne, eller ved at
  bruge et egentligt statistikværktøj. Det er bevidst, fordi formel
  standardfejl-estimering kræver antagelser (om normalitet og uafhængighed
  af residualer) som elever sjældent har værktøj til at vurdere.
- **Vægtede regressioner.** Alle datapunkter vægter ens. Hvis nogle
  målinger har større usikkerhed end andre, kan eleven ekskludere
  dem manuelt, men der er ingen mulighed for at angive måleusikkerhed
  som vægte.
- **Goodness-of-fit-tests.** Der rapporteres ikke $R^2$, $\chi^2$,
  eller p-værdier. Eleven kan vurdere fit-kvaliteten visuelt
  (residualplot) eller via RMSE i forhold til y-spredningen.
- **Outlier-detektion.** Eleven må selv kigge på residualplottet og
  beslutte hvilke punkter der eventuelt skal ekskluderes.
- **Bootstrap eller Monte Carlo.** Ingen statistisk inferens udover
  punktestimaterne.

Disse fravalg er bevidste. De ville flytte appen fra "værktøj eleven
forstår fuldt" til "værktøj eleven bruger som en sort kasse". Hvis et
forsøg kræver formel statistisk inferens, er Grafio ikke det rigtige
værktøj — der ville R, Python+SciPy, eller en dedikeret usikkerheds­app
være bedre.

---

## Bilag: oversigt over regressionstyper

| Navn | Model | $k$ | Lukket løsning | Krav |
|---|---|---:|---|---|
| Proportionalitet | $y = ax$ | 1 | Ja | — |
| Lineær | $y = ax + b$ | 2 | Ja | — |
| Andengrad | $y = ax^2 + bx + c$ | 3 | Ja | — |
| Potens | $y = a \cdot x^b$ | 2 | Via $\ln$ | $x > 0$, $y > 0$ |
| Potens (heltal) | $y = a \cdot x^n$ | 1 | Via $u = x^n$ | $x \neq 0$ hvis $n < 0$ |
| Eksponential | $y = a \cdot e^{bx}$ | 2 | Via $\ln$ | $y > 0$ |
| Eksp. med offset | $y = a \cdot e^{bx} + c$ | 3 | Nej (NM) | — |
| Invers | $y = a/x$ | 1 | Via $u = 1/x$ | $x \neq 0$ |
| Invers (kvadrat) | $y = a/x^2$ | 1 | Via $u = 1/x^2$ | $x \neq 0$ |
| Invers offset | $y = a/(x+b)$ | 2 | Nej | — |
| Invers² offset | $y = a/(x+b)^2$ | 2 | Nej | — |
| Kvadratrod | $y = a\sqrt{x}$ | 1 | Via $u = \sqrt{x}$ | $x \geq 0$ |
| Brugerdefineret | Eleven definerer | varierer | Nej (NM) | Afhænger af model |

$k$ = antal frie parametre. "Lukket løsning" angiver om der findes et
analytisk udtryk for løsningen (evt. efter en variabel-transformation),
eller om der bruges iterativ optimering (Nelder-Mead, NM).

---

*Dette dokument beskriver Grafios faglige fundament som det står ved
udgivelsen. Hvis du finder uoverensstemmelser mellem dokumentet og
appens reelle opførsel, så betragt appen som autoritativ og send en
note tilbage til udvikleren.*
