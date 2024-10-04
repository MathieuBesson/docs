---
title: Installation Bitwarden Docker
---

# Installation de bitwarden avec Docker

## Prérequis

-   La distribution linux ubuntu
-   Apache2 installé sur votre machine [Documentation d'installation](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-22-04)
-   Docker installé sur votre machine : [Documentation d'installation](../docker/docker-installation.md)

## Mise en place

Création du dossier de l'application

```shell
sudo mkdir -p /var/www/bitwarden && sudo chown -R www-data:www-data /var/www/bitwarden
```

Création du docker-compose.yml
```shell
sudo vi /var/www/bitwarden/docker-compose.yml
```

Avec le contenu suivant :

=== "/var/www/bitwarden/docker-compose.yml"
```yaml
version: "3"

services:
    bitwarden:
        image: bitwardenrs/server
        container_name: bitwarden
        restart: always
        ports:
            - "127.0.0.1:8000:80"
        volumes:
            - ./bw-data:/data
        environment:
            WEBSOCKET_ENABLED: "true"
            SIGNUPS_ALLOWED: "true"
```

Remise de www-data comme propriétaire du fichier

```shell
sudo chown www-data:www-data /var/www/bitwarden/docker-compose.yml
```

## Installation de Bitwarden

Lancement du conteneur de bitwarden

```shell
cd /var/www/bitwarden && docker-compose up -d
```

## Configuration du reverse proxy apache

Création d'un reverse proxy Apache sur {domain-name.fr} :

-   [Création d'un reverse proxy Apache](../apache/creation-reverse-proxy.md)

## Mise en place du certificat SSL

Il est aussi important de configurer un certificat SSL sur {domain-name.fr} :

-   [Documentation de création d'un certificat SSL avec Apache](../apache/configuration-ssl.md)

## Mise à jour de Bitwarden

Stop et suppression du conteneur
```bash
docker stop bitwarden
docker rm bitwarden
```

Récupération de la dernière version 
```bash
docker pull vaultwarden/server:latest
```

Redémarrage du conteneur
```bash
cd /var/www/bitwarden && docker-compose up -d
```
