---
id: pacman
aliases: []
tags: []
---

# Pacman

Pacman is the [[package manager]] for the [[Archlinux]]

## Install a package

```shell
sudo pacman -S $(package)
```

## Remove a package

```shell
sudo pacman -Rncs $(package)
```

## User repositories

[[Archlinux]] User repositories also named [[AUR]] are manage by applications
like `yay`{.verbatim} or `paru`{.verbatim} and the options are slightly the same
as `pacman`{.verbatim}
