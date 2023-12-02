---
title: "How I made this blog site"
description: "title"
date: "2023-11-29"
categories: ["Project"]
tags: ["blog", "hugo", "htmx", "typescript", "scss"]
image: Kitagawa_Marin_Holding_Nature_of_Code.png
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

Add in a `custom.html`:

```bash
layouts
 |- partials
     |- head
         |- custom.html
```

and throw in the generated html stuff there:

```html
<link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png" />
<link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png" />
<link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png" />
<link rel="manifest" href="/site.webmanifest" />
<link rel="mask-icon" href="/safari-pinned-tab.svg" color="#5bbad5" />
<meta name="msapplication-TileColor" content="#ffc40d" />
<meta name="theme-color" content="#ff0000" />
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

After a bit of configuration, now it's time to make the blog mine.

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

### Style Tweaks

And to match the Vercel Theme, I replaced the box-shadow with outline and reduced the border-radius. I also went and changed the styles of TOC. ([view source code](https://github.com/seyLu/blog/blob/main/assets/scss/custom.scss))

As for the horizontal scroll bar in code blocks, I found the default to be awfully close to the code so I went and moved it down a bit:

<!-- prettier-ignore-start -->
```scss
.article-content .chroma .lntable > tbody > tr > td:last-child > pre {
  padding-bottom: calc(var(--card-padding) / 2);
}

.highlight {
  padding-bottom: calc(var(--card-padding) / 2) !important;
}
```
<!-- prettier-ignore-end -->

Another subtle change I've made, is the grayscale effect on images and videos on darkmode:

<!-- prettier-ignore-start -->
```scss
@function native-grayscale($grayscale) {
  @return #{'grayscale(#{$grayscale})'};
}

@function native-opacity($opacity) {
  @return #{'opacity(#{$opacity})'};
}

[data-scheme='dark'] {
  img,
  video {
    --image-grayscale: 20%;
    --image-opacity: 90%;
    filter: native-grayscale(var(--image-grayscale))
      native-opacity(var(--image-opacity));
  }
}
```
<!-- prettier-ignore-end -->

This was more work than necessary, simply for the fact that grayscale & opacity in scss is different from css. Thus, requiring this workaround to tell scss to use the native css functions instead.

With the default theme being dark mode:

<!-- prettier-ignore-start -->
```yaml
colorScheme:
  toggle: true
  default: dark
```
<!-- prettier-ignore-end -->

I also had to configure giscus to default to dark mode. To do this, just copy `giscus.html` from theme to the root dir:

```bash
layouts
 |- partials
     |- comments
         |- providers
             |- giscus.html
```

and change `data-theme` to default to dark mode:

<!-- prettier-ignore-start -->
```html
{{- with .Site.Params.comments.giscus -}}
  <script
    ...
    data-theme="{{- default `dark` .darkTheme -}}"
```
<!-- prettier-ignore-end -->

## The SPA Experience

By this point, I've already tried HTMX on two other ongoing projects. I've had the HTMX experience and genuinely enjoyed using it, so I figured, why not.

Though, I did faced some issues with boosted elements and swapping. I had to use the HTMX extension, [idiomorph](https://github.com/bigskysoftware/idiomorph), and everything somehow worked as expected.

Plop in htmx and idiomorph as a js dep:

```bash
static
 |- js
     |- htmx.min.js
     |- idiomorph-ext.min.js
```

And no need to defer, just plug it in:

```html
<script src="/js/htmx.min.js"></script>
<script src="/js/idiomorph-ext.min.js"></script>
```

### Boosting Should Be Nerfed

The way boosting works is that, normal elements get the superpower to do ajax calls (links, forms, etc.). And it is inherited, meaning if the parent element is boosted, all the children elements also gets boosted.

To take advantage of this behaviour, copy the base layout from theme to root:

<!-- prettier-ignore-start -->
```html
layouts
 |- _default
     |- baseof.html
```
<!-- prettier-ignore-end -->

And turning a `Hugo-generated static site` into a `SPA-like experience` is as simple as:

<!-- prettier-ignore-start -->
```html
<body hx-ext="morph">
  <div
    hx-boost="true"
    hx-swap="morph:innerHTML"
```
<!-- prettier-ignore-end -->

But we're still not done. We still need to reinitialize components on HTMX swap. First, copy `main.ts` from theme:

```bash
assets
 |- ts
     |- main.ts
```

then initiliaze the Stack component after HTMX swap:

<!-- prettier-ignore-start -->
```ts
window.addEventListener('htmx:afterSwap', () => {
    setTimeout(() => {
        console.clear();
        window.scrollTo(0, 0);
        Stack.init();
        window.Stack = Stack;
        window.createElement = createElement;
    }, 0);
});
```
<!-- prettier-ignore-end -->

There's also the issue with the code block copy button, where it keeps adding more buttons the more HTMX swap happens. So we also have to address that and remove previously initialized copy button:

<!-- prettier-ignore-start -->
```ts
highlights.forEach((highlight) => {
    highlight.querySelectorAll('button').forEach((button) => {
        button.remove();
    });
});
```
<!-- prettier-ignore-end -->

### Fixing The Search Widget

An issue raised by boosting is that it expects other js scripts to be already loaded when needed. The previous implementation, did not need to load the js required for the search widget to work on load, not unless you go to the search page itself.

To fix this, copy the search widget in theme:

```bash
layouts
 |- partials
     |- widget
         |- search.html
```

then copy the loading of scripts from the search page in theme to the search widget in root:

<!-- prettier-ignore-start -->
```html
<script>
  window.searchResultTitleTemplate = "{{ T `search.resultTitle` }}"
</script> {{- $opts := dict "minify" hugo.IsProduction "JSXFactory" "createElement" -}}
{{- $searchScript := resources.Get "ts/search.tsx" | js.Build $opts -}}
<script
  type="text/javascript"
  src="{{ $searchScript.RelPermalink }}"
  defer
></script>
```
<!-- prettier-ignore-end -->

We also need to reinitalize the search component on HTMX swap. Copy the `search.tsx` from theme:

```bash
assets
 |- ts
     |- search.tsx
```

You don't have to do this, but I encapsulated the search component initialization, the same way as `main.ts`:

<!-- prettier-ignore-start -->
```ts
let StackSearch = {
    init: () => {
        const searchForm = document.querySelector(
                '.search-form'
            ) as HTMLFormElement,
            searchInput = searchForm.querySelector('input') as HTMLInputElement,
            searchResultList = document.querySelector(
                '.search-result--list'
            ) as HTMLDivElement,
            searchResultTitle = document.querySelector(
                '.search-result--title'
            ) as HTMLHeadingElement;

        new Search({
            form: searchForm,
            input: searchInput,
            list: searchResultList,
            resultTitle: searchResultTitle,
            resultTitleTemplate: window.searchResultTitleTemplate,
        });
    },
};
```
<!-- prettier-ignore-end -->

then used that to reiniatialize the search component:

<!-- prettier-ignore-start -->
```ts
window.addEventListener('load', () => {
    setTimeout(() => {
        StackSearch.init();
    }, 0);
});

window.addEventListener('htmx:afterSwap', () => {
    setTimeout(() => {
        StackSearch.init();
    }, 0);
});
```
<!-- prettier-ignore-end -->

We want the dynamically created element to be boosted. But on every time that this element is recreated (on page load & HTMX swap), it loses its HTMX properties. So we'll have to reapply that:

<!-- prettier-ignore-start -->
```diff
 private async doSearch(keywords: string[]) {
    const startTime = performance.now();
    const results = await this.searchKeywords(keywords);
    this.clear();

    if (results) {
        for (const item of results) {
            this.list.append(Search.render(item));
        }

+       htmx.process(this.list);

...

public static render(item: pageData) {
    return (
+       <article hx-boost="true">
```
<!-- prettier-ignore-end -->

By this point, if you go to the console tab, there will be a lot of errors. The search component expects that you're already using the search feature -- even though, what we want to do is to load the script, that's it.

To silence these errors, we just need to wrap a couple statements, to execute if the data they're expecting exists:

```diff
private async searchKeywords(keywords: string[]) {
    const rawData = await this.getData();

+   if (rawData) { ... }

...

private async doSearch(keywords: string[]) {
        const startTime = performance.now();

        const results = await this.searchKeywords(keywords);
        this.clear();

+       if (results) { ... }

...

public async getData() {
        if (!this.data) {
            /// Not fetched yet
            const jsonURL = this.form.dataset.json;

+           if (jsonURL) { ... }
```

### Page Navigation & Transition

Boosting reloads the whole page, which we don't really want, especially if we add transitions. We don't want to apply the transition to the whole page, just the parts that are changing.

Thankfully, this is possible in HTMX via [hx-indicator](https://htmx.org/attributes/hx-indicator/) attribute. For example, the left sidebar doesn't really change but the main content and the right sidebar does change.

When HTMX is doing its magic, it adds [classes depending on the operation it's currently doing](https://htmx.org/docs/#request-operations). And we can take advantage of this and apply css transitions:

<!-- prettier-ignore-start -->
```scss
.htmx-request #right,
.htmx-request #main {
  opacity: 0;
  transition: opacity 200ms ease-in;
}

.htmx-swapping #right,
.htmx-swapping #main {
  opacity: 0;
}

.htmx-settling #right,
.htmx-settling #main {
  opacity: 1;
  transition: opacity 200ms ease-in;
}
```
<!-- prettier-ignore-end -->

As you've noticed, we're targetting the right sidebar and the main content.

Now we just need to add the HTMX attributes for the main content:

```bash
layouts
 |- _default
     |- baseof.html

```

<br>

<!-- prettier-ignore-start -->
```html
<main
  hx-swap="morph:innerHTML swap:10ms settle:200ms"
  hx-indicator="#main"
  id="main"
```
<!-- prettier-ignore-end -->

We're adding a bit of delay on swap events, just enough time for the page animtation transition to kick in.

Similarly, do the same thing to the right sidebar (copied from theme):

```bash
layouts
 |- partials
     |- sidebar
         |- right.html
```

<br>

<!-- prettier-ignore-start -->
```html
<aside
  hx-swap="morph:innerHTML swap:10ms settle:200ms"
  hx-indicator="#right"
  id="right"
```
<!-- prettier-ignore-end -->

There's also the left sidebar, which contains the menu used for the main navigation. If we leave it as is, every link clicked, the whole page will reload. So we also have to address that:

```bash
layouts
 |- partials
     |- sidebar
         |- left.html
```

<br>

<!-- prettier-ignore-start -->
```html
<ol
  hx-swap="morph:innerHTML swap:10ms settle:200ms"
  hx-indicator="#main"
  ...
>
  ...
  <li id="dark-mode-toggle" hx-disable>...</li>
</ol>
```
<!-- prettier-ignore-end -->

And for elements where it doesn't make sense to be boosted, like the dark mode toggle element, we just add an `hx-disable` attribute.

### Obligatory Progress Bar

This is a classic for a SPA-like experience. We don't need to overcomplicate things, just something simple and that works.

First, add the progress bar element:

<!-- prettier-ignore-start -->
```bash
layouts
 |- _default
     |- baseof.html
```
<!-- prettier-ignore-end -->

<br>

<!-- prettier-ignore-start -->
```html
<body hx-ext="morph">
  <div id="progress-bar"></div>
```
<!-- prettier-ignore-end -->

Add some styles:

```bash
assets
 |- scss
     |- custom.scss
```

<br>

<!-- prettier-ignore-start -->
```scss
#progress-bar {
  background-color: #3498db;
  width: 0;
  height: 0.25rem;
  position: fixed;
  top: 0;
  left: 0;
  z-index: 9999;
  transition: width 0.3s ease-out;
}
```
<!-- prettier-ignore-end -->

And some scripts to expand or increase the width of the progress bar:

```bash
assets
 |- ts
     |- main.ts
```

<br>

<!-- prettier-ignore-start -->
```ts
document.body.addEventListener('htmx:configRequest', function (event) {
    // Show the progress bar before the request is made
    document.getElementById('progress-bar').style.width = '0%';
});

document.body.addEventListener('htmx:afterSwap', function (event) {
    // Hide the progress bar after the request is complete
    document.getElementById('progress-bar').style.width = '100%';

    setTimeout(function () {
        document.getElementById('progress-bar').style.opacity = '0';
    }, 300);
});
```
<!-- prettier-ignore-end -->

Hmmm. I think that's about it. I mean, if I forgot something, feel free to [raise an issue](https://github.com/seyLu/blog/issues), [start a discussion](https://github.com/seyLu/blog/discussions), or comment down below. GTG I need to sleep.
