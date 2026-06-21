---
id: Adding existing file to gitignore
aliases: []
tags:
  - linux
areas:
  - "[[linux]]"
  - "[[Git]]"
---

# Adding existing file to .gitignore

To ignore a file already existant in the repository you have to first remove the
file from the cache

```shell
git rm --cached FILENAME
```
