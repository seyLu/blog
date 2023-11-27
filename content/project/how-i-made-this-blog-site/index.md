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

### Github Comments

For comments, I went with [giscus](https://giscus.app/), which uses Github discussions to store comments.

Github Discussions are disabled by default, so make sure to enable it first in your repo settings.
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
