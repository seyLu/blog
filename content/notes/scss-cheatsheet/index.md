---
title: SCSS Cheatsheet
description: "SCSS stuff I learned from Josh W Comeau"
date: "2023-09-29"
categories: ["Notes"]
tags: ["scss"]
image: Shima_Rin_Beginning_CSS.jpg
---

## NPM Package: sass

### Installation

```bash
npm i -D sass
```

### Package.json script

```json
"scripts": {
    "watch": "sass --watch --style=compressed <scss_input_dir>:<css_output_dir>"
}
```

## Naming Convention: BEM

<!-- prettier-ignore-start -->
```html
<header class="header">                                       <!-- Block -->
  <div class="header__logo-box">                              <!-- Element -->
    <img class="header__logo" ...>                            <!-- Element -->
  </div>

  <div class="header__text-box">                              <!-- Element -->
    <h1 class="heading-primary">                              <!-- Block -->
      <span class="heading-primary--main">...</span>          <!-- Modifier -->
      <span class="heading-primary--sub">...</span>           <!-- Modifier -->
    </h1>

    <a href="#" class="btn btn--white btn--animated">...</a>
  </div>
</header>
```
<!-- prettier-ignore-end -->

### `base/_typography.scss`

```scss
.heading-primary {
    ...

    &--main {
        font-size: 6rem;
        ...
    }

    &--sub {
        font-size: 2rem;
        ...
    }
}
```

## Architecture: 7-1 pattern

```bash
scss
 |-- abstracts                   // global tools/helpers
    |-- _functions.scss
    |-- _mixins.scss
    |-- _variables.scss
 |-- base                        // boilerplate
    |-- _animations.scss
    |-- _base.scss               // styles reset/normalize
    |-- _typography.scss
    |-- _utilities.scss
 |-- components                  // widgets/small modules
    |-- _button.scss
    |-- _carousel.scss
    |-- _composition.scss
    |-- _dropdown.scss
    ...
 |-- layout                      // macro/partials
    |-- _navigation.scss
    |-- _grid.scss
    |-- _header.scss
		|-- _footer.scss
    |-- _sidebar.scss
		...
 |-- pages                       // page-specific styles
    |-- _contact.scss
    |-- _home.scss
    ...
 |-- themes                      // for large projects/dark theme etc.
 |-- vendors                     // external libraries/frameworks - bootstrap, etc.
```

### `main.min.scss`

```scss
@use "abstracts";
@use "base";
@use "components";
@use "layout";
@use "pages";
@use "themes"; // if exists
@use "vendors"; // if exists
```

### Forwarding partials

#### `abstracts/_index.scss`

```scss
@forward "functions";
@forward "mixins";
@forward "variables";
```

## Styles reset/normalize

### `base/_base.scss`

```scss
*,
*::after,
*::before {
    margin: 0;
    padding: 0;
    box-sizing: inherit;
}

html {
    font-size: 62.5%; // defines what 1rem is (62.5% of 16px = 10px)
}

body {
    box-sizing: border-box; // makes elem dimensions include padding and border
}
```

### `abstracts/_variables.scss`

```scss
// COLOR
$color-primary: #55c57a;
$color-primary-light: #7ed56f;
$color-primary-dark: #28b485;

// FONT
$default-font-size: 1.6rem;

// GRID
$grid-width: 114rem;
$gutter-vertical: 8rem;
$gutter-horizontal: 6rem;
```

## Grid: float

### layout/\_grid.scss

```scss
@use "../abstracts" as *;

.row {
    max-width: $grid-width;
    margin: 0 auto;

    &:not(:last-child) {
        margin-bottom: $gutter-vertical;
    }

    @include clearfix;

    [class^="col-"] {
        float: left;

        &:not(:last-child) {
            margin-right: $gutter-horizontal;
        }
    }

    .col-1-of-2 {
        width: calc((100% - #{$gutter-horizontal}) / 2);
    }

    .col-1-of-3 {
        width: calc((100% - #{$gutter-horizontal} * 2) / 3);
    }

    .col-1-of-4 {
        width: calc((100% - #{$gutter-horizontal} * 3) / 4);
    }

    .col-2-of-3 {
        width: calc(
            #{$gutter-horizontal} + 2 * ((100% - #{$gutter-horizontal} * 2) / 3)
        );
    }

    .col-2-of-4 {
        width: calc(
            #{$gutter-horizontal} + 2 * ((100% - #{$gutter-horizontal} * 3) / 4)
        );
    }

    .col-3-of-4 {
        width: calc(
            2 * #{$gutter-horizontal} + 3 *
                ((100% - #{$gutter-horizontal} * 3) / 4)
        );
    }
}
```

### Clearfix hack

#### `abstracts/_clearfix.scss`

```scss
@mixin clearfix {
    &::after {
        content: "";
        display: table;
        clear: both;
    }
}
```
