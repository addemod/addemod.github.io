---
sidebar_position: 1
---

# SCSS-struktur och teman

:::info

Frontenden är uppsatt för att stödja mer än ett tema som standard 🎨

:::

## Lägga till ett nytt tema

För att lägga till ett nytt tema lägger man helt enkelt till det i listan med teman:
```js title="themes/themes.js"
module.exports = ['inhouse-group', 'matt-tema'];
```

När detta är gjort finns det tre saker att göra:

1. Skapa en tema-fil i scripts/-mappen, med samma namn som ditt tema: matt-tema.js. Innehållet i filen bör vara följande:

```js title="scripts/matt-tema.js"
import app from './app.js';

app('matt-tema');
```

2. Skapa en tema-fil i styles/-mappen, med samma namn som ditt tema: matt-tema.scss. Innehållet i filen bör vara följande:

```scss title="styles/matt-tema.scss"
// Entry file for matt-tema
@forward "variables/matt-tema"; // forward theme specific variables

// All components/pages etc., without any theme specific styling
@use "globals";

// Theme specific styling for each component
@use './components/footer/matt-tema.scss'; // exempel
@use './components/header/matt-tema.scss'; // exempel
```

3. Skapa en tema-fil i styles/variables/-mappen, med samma namn som ditt tema: matt-tema.scss. Innehållet i filen bör vara något snarlikt, beroende på vilka variabler ni använder i projektet:

```scss title="styles/variables/matt-tema.scss"
@use "sass:map";
@use "colors";

// Skriv över endast primary-färgen
$theme-colors: map.merge(colors.$mox-colors, (
    primary: #06835b,
));

@forward "../variables" with (
  $mox-colors: $theme-colors,
);
```

## Styling för react-komponenter

När vi skriver styling för våra react-komponenter så vill vi hålla ihop CSS:en med själva komponenten, vilket gör att vi har följande struktur för varje komponent:

- scripts
  - components
    - FooComponent
      - **foo-component.js**
      - style
        - **main.scss**
        - **inhouse-group.scss**
        - **matt-tema.scss**

I javascripten behöver vi sedan importera vår styling. Detta gör vi för att kunna splitta upp stylingen i egna chunks, för att inte ladda in onödig css som inte används på alla sidor. För att enkelt importera all stylingen för en komponent kan vi använda oss av följande kodsnutt i komponenten:

```js
import(`./style/${window.__litium.themeName}.scss`);
```

Temanamnet sätts på `window.__litium.themeName` efter `app('matt-tema')` har anropats i `scripts/matt-tema.js`. Genom att skriva importen på följande vis så skapas en css-fil per tema, för den här komponenten.

### Exempel på komponent-css

```scss title="scripts/components/<namn>/styles/main.scss"
@use 'abstracts'as *; // To access 'get-color()'

.foo {
  &__bar {
    color: get-color(primary);
  }
}
```

```scss title="scripts/components/<namn>/styles/inhouse-group.scss"
@forward 'css-vars/inhouse-group.scss'; // forward theme specific variables

@use 'main'; // include global styling for this component

// Styling for this specific theme
.foo {
  &__bar {
    background: red;
  }
}
```

```scss title="scripts/components/<namn>/styles/matt-tema.scss"
@forward 'css-vars/matt-tema.scss';// forward theme specific variables

@use 'main'; // include global styling for this component

// Styling for this specific theme
.foo {
  &__bar {
    background: green;
  }
}
```

### Styling för komponent utan något tema

Mappstrukturen är den samma som med flera teman, men importen i komponenten ser lite annorlunda ut:

```js
import './style/<namn på fil>.scss';
```

Skillnaden här är att vi inte importerar en fil beroende på en variabel, utan vi skriver specifikt vad filen heter.

### Styling utanför react-komponenter

#### Två eller flera teman
Exemplet nedan gäller för komponenten “footer”.

- En entrypoint per tema, t.ex. `styles/matt-tema.scss` och `styles/inhouse-group.scss`
- En grundfil per komponent som håller generell styling för just den komponenten, t.ex. `styles/components/footer/index.scss`
- För varje komponent bör det finnas en fil per tema, som sätter styling specifikt för det temat. (Obs. endast om det ska finnas styling specifikt för det temat, för den komponenten)
- Indexfilen för varje komponent/sida/block måste inkluderas i `styles/globals.scss`. Detta görs enklast genom att lägga till `@use 'footer';` i filen `styles/components/index.scss` om det är en komponent. Samma gäller för sidor och block.

Varje tema-specifik style måste importeras i grundfilen för varje tema (t.ex. `styles/matt-tema.scss`)

##### Exempel
```scss title="styles/components/footer/index.scss"
@use 'abstracts'as *; // To access 'get-color()'

.footer {
  background-color: get-color(primary);
}
```

```scss title="styles/components/footer/inhouse-group.scss"
.footer {
  border-color: red;
}
```

```scss title="styles/components/footer/matt-tema.scss"
.footer {
  border-color: green;
}
```

```scss title="styles/matt-tema.scss"
// Entry file for matt-tema
@forward "variables/matt-tema"; // forward theme specific variables

// All components/pages etc., without any theme specific styling
@use "globals";

// Theme specific styling for each component
@use './components/footer/matt-tema.scss'; // exempel
```

#### Inget tema

Exemplet nedan gäller för komponenten “footer”.

Indexfilen för varje komponent/sida/block måste inkluderas i globals. Detta görs enklast genom att lägga till `@use 'footer';` i filen `styles/components/index.scss` om det är en komponent. Samma gäller för sidor och block.

## Utan tema

För att köra utan något tema alls så kan man helt enkelt rensa listan i `themes/themes.js`, vilket då kommer kommer peka på `scripts/no-theme.js` som entry för javascripten, samt `styles/globals.scss` som entry för stylingen.

## Variabler

För varje typ av variabler bör det finnas en fil för respektive typ, t.ex. för färger så finns `/styles/variables/_colors.scss` där alla variabler som är relaterade till färger bör finnas.

Utöver varje specifik fil så finns huvudfilen `/styles/variables/index.scss` där alla variabler definieras igen, för att enkelt kunna skriva över dem för varje tema.

```scss title="styles/variables/index.scss"
@use "colors";
@use "grid";
@use "typography";

// Grid
$mox-breakpoints: grid.$mox-breakpoints !default;
$mox-breakpoint-classes: grid.$breakpoint-classes !default;
$mox-row-max-width: grid.$row-max-width !default;
$mox-grid-row-gutter: grid.$grid-row-gutter !default;
$mox-grid-column-gutter: grid.$grid-column-gutter !default;

// Colors
$mox-colors: colors.$mox-colors !default;

// Typography
$font-family: typography.$font-family !default;
$text-sizes: typography.$text-sizes !default;
```

### Skriva över per tema

Varje tema ska, som bekant, ha en egen variabelfil, t.ex. `styles/variables/inhouse-group.scss`. I den här filen görs alla överskrivningar. Som ett exempel så vill jag skriva över endast primary-färgen för tema “inhouse-group”:
```scss title="styles/variables/inhouse-group.scss"
@use "sass:map"; // För att komma åt "map" funktioner
@use "colors"; // Standardfärger

// Hämta in standardfärgerna, slå ihop dem med färgerna för det här temat
$theme-colors: map.merge(colors.$mox-colors, (
    primary: #06835b,
));

// Skicka vidare variablerna, men skriv över $mox-colors med $theme-colors.
@forward "../variables" with (
  $mox-colors: $theme-colors,
);
```

Ett annat exempel, där vi skriver över mer än bara färgerna:

```scss title="styles/variables/inhouse-group.scss"
@use "sass:map";
@use "colors";

// Colors
$theme-colors: map.merge(colors.$mox-colors, (
    primary: #06835b,
));

// Typography
$theme-font-family: 'Roboto', Arial, sans-serif;
$theme-text-sizes: (
  xsmall: 12px,
  small: 14px,
  medium: 16px,
  large: 18px,
  xlarge: 20px,
);

// Skicka vidare variablerna, men skriv över $mox-colors med $theme-colors.
@forward "../variables" with (
  $mox-colors: $theme-colors,
  $font-family: $theme-font-family,
  $text-sizes: $theme-text-sizes
);
```