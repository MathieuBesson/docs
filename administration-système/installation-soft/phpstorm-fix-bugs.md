---
title: Résolution bug PhpStorm
author :
    name : Mathieu BESSON
    linkedin : mathieubesson
links:
  - title: "Bug officiel : `Process x is still running`"
    url: https://youtrack.jetbrains.com/issue/IDEA-331715/Unable-to-start-Intellij-Exception-Process-4230-is-still-running-but-this-pid-does-no-exists
---

# Résoudre les bugs de PhpStorm 

!!! danger "Exemples d'erreur"

    Exception: Process x is still running

### Erreur complète : 

```shell
Start Failed
Cannot connect to already running IDE instance.
Exception: Process 4,085 is still running
```

### Solution

```shell
rm -f /home/{your-home}/.config/JetBrains/{your-PhpStormVersion}/.lock
```

## Erreur : Freeze PhpStorm

### Solution

```shell
kill -9 $(pgrep -f phpstorm)
```
