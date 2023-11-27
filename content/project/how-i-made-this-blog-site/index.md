---
title: "How I made this blog site"
description: "title"
date: "2023-11-27"
categories: ["Project"]
tags: ["blog", "hugo", "htmx", "typescript", "scss"]
image: Kitagawa_Marin_Holding_Nature_of_Code.png
draft: true
---

Source code can be found in [blog](https://github.com/seyLu/blog) repo.

## Basic Setup

For this blog site, I used this hugo theme [stack by Jimmy Cai](https://stack.jimmycai.com/). Plus, it has great documentation on how to customize the theme.

### Theme Installation

I installed the theme via git submodule:

```bash
git submodule add https://github.com/CaiJimmy/hugo-theme-stack.git themes/hugo-theme-stack
```

and configured hugo to use the theme via `hugo.yaml`:

```yaml
theme: hugo-theme-stack
```

### Splitting Config

You don't have to split the config. But I figured, since the default config is too long, why not split it up. ([read more at Hugo docs](https://gohugo.io/getting-started/configuration/#configuration-directory))

```bash
config
 |- _default
     |- config.yaml
     |- menu.yaml
     |- params.yaml
```

### Social Icons

Social Icons can be configured at `menu.yaml`. And for icons not included in the theme, go to [tablericons](https://tablericons.com/) and save the svg at:

```bash
assets
 |- icons
     |- <new_icon>.svg
```

### Avatar & Favicon

To change the avatar, place your avatar image at:

```bash
assets
 |- img
     |- seyLu.png
```

Then configure at `params.yaml`:

<!-- prettier-ignore-start -->
```yaml
sidebar:
  subtitle: pachi pachi pachi pachi
  avatar:
    enabled: true
    local: true
    src: img/seyLu.png
```
<!-- prettier-ignore-end -->

I am too lazy, so I used the same image for the favicon. To generate favicons that work on most devices, go to [realfavicongenerator](https://realfavicongenerator.net/), and place the generated files at:

```bash
static
 |- ...
```

then configure at `params.yaml`:

```yaml
favicon: /favicon.ico
```

### Github Comments

For comments, I went with [giscus](https://giscus.app/), which uses Github discussions to store comments.

Github Discussions is disabled by default, so make sure to enable it first in your repo settings.
![Github Discussions](gh-discussions.png)

You can then find the giscus config on `params.yaml`:

<!-- prettier-ignore-start -->
```yaml
comments:
  enabled: true
  provider: giscus

  giscus:
    repo: seyLu/blog
    repoID: <repo_id>
    category: General
    categoryID: <category_id>
    mapping: pathName
    reactionsEnabled: 1
    emitMetadata: 0
    lightTheme: light
    darkTheme: transparent_dark
```
<!-- prettier-ignore-end -->

And to get the `repoID` and `categoryID`, I used the [Github CLI](https://cli.github.com/) with this GraphQL query:

```bash
gh api graphql -f query='
{
  repository(owner: "seyLu", name: "blog") {
    id # RepositoryID
    name
    discussionCategories(first: 10) {
      nodes {
        id # CategoryID
        name
      }
    }
  }
}'
```

## Spicing Things up

### Custom Font

I really wanted to try out the new [Geist font by Vercel](https://vercel.com/font). It was just released a couple weeks ago as of me writing this blog, and I've heard it's very similar to [Inter font](https://fonts.google.com/specimen/Inter) but better.

For the code font, I went with [JetBrains Mono](https://www.jetbrains.com/lp/mono/) -- I mean there's really no competition.

To customize the font, download the fonts, then put them in:

```bash
static
 |- fonts
     |- geist
         |- GeistVariableVF.ttf
         |- GeistVariableVF.woff2
     |- jetbrains_mono
         |- JetBrainsMono.ttf
```

I went with variable fonts because it is easier to setup and maintain. But since it's relatively new, it might have some compatibility issues with older browsers -- but this is my blog, so who cares about that.

To use the fonts, first create a custom scss file:

```bash
assets
 |- scss
     |- custom.scss
```

then, add this mixin to simplify loading the fonts:

<!-- prettier-ignore-start -->
```scss
@mixin font($font-family, $font-file) {
  @font-face {
    font-family: $font-family;
    src: local(''), url($font-file + '.otf') format('otf'),
      url($font-file + '.woff2') format('woff2'),
      url($font-file + '.ttf') format('truetype');
    font-dispay: swap;
  }
}

@include font('Geist', '/fonts/geist/GeistVariableVF');
@include font('JetBrains Mono', '/fonts/jetbrains_mono/JetBrainsMono');
```
<!-- prettier-ignore-end -->

then copy the the part where the font is loaded in the theme and configure the custom font to be loaded first:

<!-- prettier-ignore-start -->
```scss
:root {
  --sys-font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Droid Sans',
    'Helvetica Neue';
  --zh-font-family: 'PingFang SC', 'Hiragino Sans GB', 'Droid Sans Fallback',
    'Microsoft YaHei';

  --base-font-family: 'Geist', 'Lato', var(--sys-font-family),
    var(--zh-font-family), sans-serif;
  --code-font-family: 'JetBrains Mono', Menlo, Monaco, Consolas, 'Courier New',
    var(--zh-font-family), monospace;
}
```
<!-- prettier-ignore-end -->

### Vercel Theme

Since I've used Geist font, might as well fully commit and follow the [color scheme of Vercel](https://vercel.com/design/color).
