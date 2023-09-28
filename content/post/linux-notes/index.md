---
title: Linux Notes
description: Hugo, the world's fastest framework for building websites
date: "2023-09-28"
description: "collection of linux notes"
categories: [
    "Notes",
]
tags: [
    "linux",
    "shell",
]
image: Sakurajima_Mai_Linux_Essentials.jpg
toc: true
---

## chmod

```bash
chmod [options] file(s)
```

Permission start with letter specifying which users should be affected by change:

```bash
u   owner user
g   owner group
o   other (neither u nor g)
a   all users
```

Followed by a change instruction:

```bas
+   set
-   clear
```

Every mode has a corresponding number:

```bash
r   read    4
w   write   2
x   execute 1
```

Shorthand for changing entire permisson follows (u, g, o) triplet with sum of mode corresponding to permission. For example:

```bash
chmod 755 sample.txt
```

<br>

```bash
means               in triplet

u+rwx               rwx
g+rx-w              r-x
o+rx-w              r-x
```
