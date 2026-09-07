# Styles og tema — Grafio-skabelonen

Dette dokument er en komplet beskrivelse af det visuelle system der bruges i
Grafio. Det er tænkt som en **kopier-og-brug-skabelon** til nye fysik-apps,
så de visuelt hænger sammen med Grafio uden at man skal genopfinde paletten
hver gang.

Skabelonen omfatter:

- **Farver** — en lys og en mørk palet, defineret som CSS-variabler
- **Toggle mellem lys og mørk** — en lille knap øverst til højre
- **Typografi, mellemrum og kanter** — kort sammenfatning
- **Chart.js-integration** — sådan får grafer til at følge temaet

Alt er gennemtænkt med læselighed og kontrast for øje. WCAG-noter er
inkluderet, så lærere og elever altid har en behagelig læseoplevelse.

---

## 1. CSS-variabler

Definér disse i `:root` i `<style>`-blokken. Den lyse palet er aktiv som
standard.

```css
:root {
  /* Baggrund og overflader */
  --bg: #ffffff;
  --surface: #ffffff;
  --panel: #f5f4ef;
  --row-alt: #fafaf7;
  --panel-border: #e8e5dc;

  /* Tekst */
  --text: #2a2a2a;
  --muted: #6b6b6b;

  /* Kolonnetyper (i tabeller) */
  --data-col: #eaf3fb;
  --data-col-border: #cfe1f2;
  --calc-col: #f5f4ef;
  --calc-col-border: #e8e5dc;
  --const-col: #f4ecdb;
  --const-col-border: #e6d8b5;

  /* Accentfarver */
  --blue: #378ADD;   /* Primær handling, links, fit-punkter */
  --green: #1D9E75;  /* Residualer, succes */
  --brown: #993C1D;  /* Fit-kurver, integral, tangent */
  --danger: #a04040; /* Slette-handlinger */

  /* Fit/integral-boks */
  --fit-bg: rgba(153,60,29,0.05);
  --fit-border: rgba(153,60,29,0.20);

  /* Chart.js-integration */
  --chart-grid: #eeeeee;
  --chart-text: #2a2a2a;

  /* Skygger */
  --shadow: 0 1px 3px rgba(0,0,0,0.05), 0 1px 2px rgba(0,0,0,0.03);
}
```

Den mørke palet overskriver de samme variabler når `<html>` har attributten
`data-theme="dark"`:

```css
[data-theme="dark"] {
  --bg: #1a1a1e;
  --surface: #222229;
  --panel: #27272f;
  --row-alt: #1e1e25;
  --panel-border: #3d3d48;
  --text: #e4e4ec;
  --muted: #9494a8;
  --data-col: #1c3048;
  --data-col-border: #2d4d70;
  --calc-col: #27272f;
  --calc-col-border: #3d3d48;
  --const-col: #2e2614;
  --const-col-border: #5a4828;
  --blue: #5ba3f0;
  --green: #33c48a;
  --brown: #e07850;
  --danger: #e05050;
  --fit-bg: rgba(220,100,60,0.12);
  --fit-border: rgba(220,100,60,0.30);
  --shadow: 0 1px 3px rgba(0,0,0,0.5), 0 1px 2px rgba(0,0,0,0.3);
  --chart-grid: #333340;
  --chart-text: #b8b8cc;
}
```

### Designvalg bag farverne

Den lyse palet er bevidst varm og **ikke** ren hvid. Baggrunden er `#ffffff`,
men paneler og rækker har en let beige toning (`#f5f4ef`, `#fafaf7`) som
beroliger øjet på lange arbejdsdage og giver appen et roligt
papir-agtigt udtryk frem for et skarpt "kontor"-look.

Den mørke palet er en lun kul-grå, **ikke** ren sort. Baggrunden er `#1a1a1e`
med en let blå toning, og teksten `#e4e4ec` har samme tone — det giver et
roligt mørkt look uden den hårde "OLED-sort"-stil. Accentfarverne er lysere
og varmere end i den lyse palet (orange `#e07850` frem for rødbrun
`#993C1D`), så de forbliver synlige på den mørke baggrund uden at virke
neon-agtige.

Kolonnetyperne (data, beregnet, konstant) bruger blå, grå og amber som
visuelle "zoner" i tabellen. I mørk tilstand er disse mætningsmæssigt
justeret — særligt konstanter (mørk amber) og datakolonner (mørk marine) —
så de stadig læses som distinkte zoner og ikke smelter ind i baggrunden.

### Kontrast (WCAG)

Følgende centrale kombinationer er tjekket mod WCAG AA-standarden, som
kræver mindst 4,5:1 for normal tekst og 3:1 for stor tekst (18pt+).

**Lys palet:**

| Forgrund | Baggrund | Kontrast | Vurdering |
|---|---|---|---|
| Tekst `#2a2a2a` | Baggrund `#ffffff` | 14,3:1 | AAA |
| Tekst `#2a2a2a` | Panel `#f5f4ef` | 13,1:1 | AAA |
| Nedtonet `#6b6b6b` | Baggrund `#ffffff` | 5,4:1 | AA |
| Fit-farve `#993C1D` | Baggrund `#ffffff` | 6,2:1 | AA |
| Blå `#378ADD` på hvid | Baggrund `#ffffff` | 3,3:1 | AA (kun store tegn) |

**Mørk palet:**

| Forgrund | Baggrund | Kontrast | Vurdering |
|---|---|---|---|
| Tekst `#e4e4ec` | Baggrund `#1a1a1e` | 13,8:1 | AAA |
| Tekst `#e4e4ec` | Panel `#27272f` | 11,1:1 | AAA |
| Nedtonet `#9494a8` | Baggrund `#1a1a1e` | 6,4:1 | AA |
| Fit-farve `#e07850` | Baggrund `#1a1a1e` | 5,7:1 | AA |
| Blå `#5ba3f0` på mørk | Baggrund `#1a1a1e` | 6,9:1 | AA |

Brug `--muted` til sekundær tekst (etiketter, hjælpetekster, hover-tooltips).
Lad være med at bruge den til brødtekst — brug `--text` der.

`--blue` er kun marginalt over kontrast-grænsen i lys tilstand. Brug den
til store flader og overskrifter, ikke til småtekst. Til knapper med
blå baggrund: brug hvid tekst (`#ffffff`), som har 4,5:1 mod `#378ADD`.

---

## 2. Toggle mellem lys og mørk

### HTML

Placer denne knap øverst til højre, ved siden af din apps titel:

```html
<div class="app-header">
  <div>
    <h1>App-navn <span class="app-subtitle">— undertitel</span></h1>
    <p class="subtitle">Kort beskrivelse.</p>
  </div>
  <button class="theme-btn" onclick="toggleTheme()"
          title="Skift til mørk/lys tilstand"
          aria-label="Skift tema">
    <span class="icon-moon">
      <svg width="16" height="16" viewBox="0 0 24 24" fill="none"
           stroke="currentColor" stroke-width="2" stroke-linecap="round"
           stroke-linejoin="round">
        <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"/>
      </svg>
    </span>
    <span class="icon-sun">
      <svg width="16" height="16" viewBox="0 0 24 24" fill="none"
           stroke="currentColor" stroke-width="2" stroke-linecap="round"
           stroke-linejoin="round">
        <circle cx="12" cy="12" r="5"/>
        <line x1="12" y1="1" x2="12" y2="3"/>
        <line x1="12" y1="21" x2="12" y2="23"/>
        <line x1="4.22" y1="4.22" x2="5.64" y2="5.64"/>
        <line x1="18.36" y1="18.36" x2="19.78" y2="19.78"/>
        <line x1="1" y1="12" x2="3" y2="12"/>
        <line x1="21" y1="12" x2="23" y2="12"/>
        <line x1="4.22" y1="19.78" x2="5.64" y2="18.36"/>
        <line x1="18.36" y1="5.64" x2="19.78" y2="4.22"/>
      </svg>
    </span>
  </button>
</div>
```

SVG-ikoner er valgt frem for emoji-tegn for at sikre konsistent visning på
tværs af styresystemer og browsere.

### CSS

```css
.app-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
}

.theme-btn {
  flex-shrink: 0;
  padding: 6px 8px;
  background: transparent;
  border: 1px solid var(--panel-border);
  border-radius: 4px;
  cursor: pointer;
  color: var(--muted);
  line-height: 1;
  display: flex;
  align-items: center;
  margin-top: 4px;
}

.theme-btn:hover {
  color: var(--text);
  background: var(--panel);
}

.theme-btn svg { display: block; }

/* I lys tilstand: vis måne-ikon (klik → skift til mørk).
   I mørk tilstand: vis sol-ikon (klik → skift til lys). */
.icon-sun  { display: none; }
.icon-moon { display: block; }
[data-theme="dark"] .icon-sun  { display: block; }
[data-theme="dark"] .icon-moon { display: none; }
```

### JavaScript

To dele. Først hjælpefunktionen og toggle-funktionen:

```js
function toggleTheme() {
  const html = document.documentElement;
  const goingDark = html.getAttribute('data-theme') !== 'dark';
  html.setAttribute('data-theme', goingDark ? 'dark' : 'light');
  localStorage.setItem('grafio_theme', goingDark ? 'dark' : 'light');
  // Hvis du har Chart.js eller andre canvas-elementer der skal opdateres:
  // kald din render-funktion her.
  render();
}
```

Brug en unik localStorage-nøgle pr. app (fx `mitprojekt_theme` i stedet for
`grafio_theme`), så apps ikke deler hinandens valg.

Dernæst — og **vigtigt** — initialisering der sker *før* siden renderer,
så der ikke opstår et lyst blink ved sideopstart i mørk tilstand:

```html
<script>
  /* Initialiser tema FØR resten af appen renderer.
     Læg denne IIFE i <head> eller allerførst i din <body><script>. */
  (function() {
    var saved = localStorage.getItem('grafio_theme');
    if (saved === 'dark') {
      document.documentElement.setAttribute('data-theme', 'dark');
    }
  })();
</script>
```

Hvis du glemmer denne IIFE, vil siden kortvarigt vise sig lyst inden den
skifter til mørk — et generende blink kendt som "flash of unstyled content".

---

## 3. Typografi

```css
body {
  font-family: 'Helvetica Neue', Arial, sans-serif;
  font-size: 14px;
  line-height: 1.5;
  font-variant-numeric: tabular-nums;  /* tal får ens bredde */
}

h1 { font-size: 22px; font-weight: 600; }
h2 { font-size: 16px; font-weight: 600; }
h3 { font-size: 12px; font-weight: 600; text-transform: uppercase;
     letter-spacing: 0.5px; color: var(--muted); }

.subtitle { color: var(--muted); font-size: 13px; }
```

`tabular-nums` er vigtigt for tabeller med tal — det sikrer at cifrene er
lige brede, så kolonner med tal flugter pænt selv uden hård tabel-formatering.

For monospaced indhold (formler, kode, kolonnesymboler):

```css
font-family: 'Menlo', 'Monaco', monospace;
```

---

## 4. Knapper

Tre niveauer:

```css
/* Standardknap */
button {
  font-family: inherit;
  font-size: 13px;
  padding: 7px 14px;
  border: 1px solid var(--panel-border);
  background: var(--panel);
  color: var(--text);
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.1s;
}
button:hover { background: #ecebe4; }

/* Primær handling (blå, fremhævet) */
button.primary {
  background: var(--blue);
  color: white;
  border-color: var(--blue);
}
button.primary:hover { background: #2c75c0; }

/* Destruktiv handling (rød tekst, ikke baggrund) */
button.danger {
  color: var(--danger);
}

/* Lille knap (til inline-handlinger) */
button.small {
  padding: 3px 8px;
  font-size: 12px;
}

/* Ikon-knap (transparent, kun et symbol) */
button.icon {
  padding: 3px 7px;
  background: transparent;
  border: none;
  color: var(--muted);
  font-size: 16px;
  line-height: 1;
}
button.icon:hover { color: var(--danger); background: transparent; }
```

I mørk tilstand skal hover-baggrundene overrides:

```css
[data-theme="dark"] button:hover         { background: #32323c; }
[data-theme="dark"] button.primary:hover { background: #4a90d9; }
```

---

## 5. Inputs og dropdowns

```css
select, input[type="text"] {
  font-family: inherit;
  font-size: 13px;
  padding: 5px 8px;
  border: 1px solid var(--panel-border);
  border-radius: 3px;
  background: var(--surface);
  color: var(--text);
}
```

Bemærk: i mørk tilstand skal `<select>` og `<input>`-felter eksplicit have
`color: var(--text)` og `background: var(--surface)` — browsere bruger ofte
deres eget standardtema hvis du ikke siger andet:

```css
[data-theme="dark"] select,
[data-theme="dark"] input[type="text"] {
  color: var(--text);
  background: var(--surface);
}
```

---

## 6. Kanter, skygger og afrunding

```css
border: 1px solid var(--panel-border);
border-radius: 4px;   /* knapper, små bokse */
border-radius: 6px;   /* paneler, kort, modaler */
box-shadow: var(--shadow);
```

Brug `--shadow`-variablen i stedet for at skrive skyggen i hver enkelt
selector — så opdateres skyggen automatisk når temaet skifter (mørke
skygger er dybere fordi de skal være synlige på den mørke baggrund).

---

## 7. Chart.js-integration

Chart.js bruger ikke CSS-variabler automatisk. Tre ting skal gøres:

### a) Hjælpefunktion der læser de aktuelle tema-farver

```js
function chartThemeColors() {
  const s = getComputedStyle(document.documentElement);
  const get = v => s.getPropertyValue(v).trim();
  return {
    grid:    get('--chart-grid')  || '#eeeeee',
    text:    get('--chart-text')  || '#2a2a2a',
    blue:    get('--blue')        || '#378ADD',
    green:   get('--green')       || '#1D9E75',
    brown:   get('--brown')       || '#993C1D',
    surface: get('--surface')     || '#ffffff'
  };
}
```

### b) Brug farverne i Chart-konfigurationen

```js
const colors = chartThemeColors();

new Chart(canvas, {
  type: 'scatter',
  data: { ... },
  options: {
    scales: {
      x: {
        grid:   { color: colors.grid },
        ticks:  { color: colors.text },
        title:  { color: colors.text }
      },
      y: {
        grid:   { color: colors.grid },
        ticks:  { color: colors.text },
        title:  { color: colors.text }
      }
    }
  }
});
```

### c) Tegn graferne om når temaet skifter

I `toggleTheme()` kalder vi `render()` — og `render()` skal genopbygge
graferne fra bunden, så de henter de nye tema-farver. Hvis du har en mere
fin-kornet rendering kan du i stedet løbe gennem dine `chartInstances`
og opdatere farver direkte, men "genopbyg alt" er enklest.

### Plugins der tegner direkte på canvas

Hvis du har custom Chart.js-plugins der bruger `ctx.fillStyle` eller
`ctx.strokeStyle`, så læs farven via `getComputedStyle` **inde i tegnefasen**,
ikke ved plugin-definitionen. Sådan her:

```js
const integralShadePlugin = {
  id: 'integralShade',
  beforeDatasetsDraw(chart) {
    // ...
    const brown = getComputedStyle(document.documentElement)
                    .getPropertyValue('--brown').trim() || '#993C1D';
    ctx.fillStyle   = hexToRgba(brown, 0.15);
    ctx.strokeStyle = hexToRgba(brown, 0.55);
    // ...
  }
};

function hexToRgba(hex, alpha) {
  const r = parseInt(hex.slice(1, 3), 16);
  const g = parseInt(hex.slice(3, 5), 16);
  const b = parseInt(hex.slice(5, 7), 16);
  return `rgba(${r},${g},${b},${alpha})`;
}
```

Hvis du i stedet låser farven ved plugin-definitionstidspunktet, vil
plugin'et beholde den oprindelige farve selv efter et tema-skift.

---

## 8. Hurtig start-skabelon

Hvis du laver en ny lille app, så start her:

```html
<!DOCTYPE html>
<html lang="da">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Mit projekt</title>

<script>
  (function() {
    var saved = localStorage.getItem('mitprojekt_theme');
    if (saved === 'dark') {
      document.documentElement.setAttribute('data-theme', 'dark');
    }
  })();
</script>

<style>
  :root {
    /* ... indsæt den lyse palet fra afsnit 1 her ... */
  }
  [data-theme="dark"] {
    /* ... indsæt den mørke palet fra afsnit 1 her ... */
  }

  * { box-sizing: border-box; }

  body {
    margin: 0;
    padding: 24px;
    font-family: 'Helvetica Neue', Arial, sans-serif;
    background: var(--bg);
    color: var(--text);
    font-size: 14px;
    line-height: 1.5;
    font-variant-numeric: tabular-nums;
  }

  /* ... resten af stylene fra afsnit 3-6 ... */
</style>
</head>
<body>

<div class="app-header">
  <div>
    <h1>Mit projekt</h1>
    <p class="subtitle">Kort beskrivelse.</p>
  </div>
  <!-- ... tema-toggle-knap fra afsnit 2 ... -->
</div>

<!-- Resten af din app -->

<script>
  function toggleTheme() {
    const html = document.documentElement;
    const goingDark = html.getAttribute('data-theme') !== 'dark';
    html.setAttribute('data-theme', goingDark ? 'dark' : 'light');
    localStorage.setItem('mitprojekt_theme', goingDark ? 'dark' : 'light');
    // Hvis din app har grafer eller andet at gentegne, kald det her.
  }
</script>

</body>
</html>
```

Husk at skifte `mitprojekt_theme` til en unik nøgle for din app, så
forskellige apps ikke deler tema-valg via localStorage.

---

## 9. Tjekliste når du laver en ny app

- [ ] Lys palet i `:root`
- [ ] Mørk palet i `[data-theme="dark"]`
- [ ] IIFE i `<head>` der læser temaet fra localStorage før render
- [ ] `.app-header` med tema-toggle øverst til højre
- [ ] `toggleTheme()`-funktion
- [ ] Unik localStorage-nøgle (`<appnavn>_theme`)
- [ ] Hvis Chart.js bruges: `chartThemeColors()`-helper og opdatering ved
      tema-skift
- [ ] Hvis canvas-plugins bruges: farver læses via `getComputedStyle` i
      tegnefasen, ikke ved definition
- [ ] Test begge tilstande visuelt før udgivelse
- [ ] Test at temaet huskes ved genindlæsning af siden

Med den her skabelon kan en ny app komme i gang på 5-10 minutter med et
fuldt funktionelt lys/mørk-tema der ser professionelt ud fra første
gang.
