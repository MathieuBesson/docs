---
title: Docker ports interdits
author :
    name : Mathieu BESSON
    linkedin : mathieubesson
---

# Interdire à Docker d'utiliser des ports bannis par ufw ou iptable 

## Prérequis

-   La distribution linux ubuntu
-   Docker installé sur votre machine : [Documentation d'installation](docker-installation.md)

## Solution 1 : Preciser un binding de port local

=== "docker-compose.yml"
```yaml
version: "3"

services:
    container:
        container_name: container_name
        image: hello-world
        ports:
            - "127.0.0.1:8000:80"
```

En utilisant `127.0.0.1:8000` on force docker à seulement éffectuer un binding local et non sur l'ip public de la machine

## Solution 2 : Ne pas autoriser docker à utiliser iptables

Ajouter l'option suivante à la fin du fichier `/etc/default/docker`

=== "/etc/default/docker"
```shell
DOCKER_OPTS="--iptables=false"
```

**OU**

Modifier le fichier `/etc/docker/daemon.json`
```json
{
    "iptables": false
}
```

Puis redémarrer le deamon docker 
```shell
sudo systemctl restart docker
```