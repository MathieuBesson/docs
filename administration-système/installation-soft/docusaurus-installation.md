---
title: Installation Docusaurus Docker
---

# Installation de docusaurus avec Docker

## Prérequis

-   La distribution linux ubuntu
-   Apache2 installé sur votre machine [Documentation d'installation](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-22-04)
-   Docker installé sur votre machine : [Documentation d'installation](../docker/docker-installation.md)
-   Nodejs installé sur une machine ou un conteneur Docker

## Mise en place

Création du dossier de l'application
```shell
sudo mkdir -p /var/www/docusaurus && sudo chown -R www-data:www-data /var/www/docusaurus
```

Création du Dockerfile de Docusaurus
```Dockerfile title="/var/www/docusaurus/Dockerfile"
FROM node:16-slim

RUN mkdir -p /var/www/docusaurus
WORKDIR /var/www/docusaurus
COPY . /var/www/docusaurus

VOLUME /var/www/docusaurus

RUN npx create-docusaurus@latest docusaurus classic && npm install --frozen-lockfile && npm run build

EXPOSE 8001

ENTRYPOINT npm install --frozen-lockfile && npm run serve -- --build --port 8001 --no-open
```

Création du docker-compose.yml de Docusaurus
```yaml title="/var/www/docusaurus/docker-compose.yml"
version: "3"

services:
    docusaurus:
        container_name: docusaurus
        image: docusaurus
        restart: always
        tty: true
        ports:
            - "127.0.0.1:8001:8001"
        volumes:
            - .:/var/www/docusaurus
```

Build de l'image de Docusaurus
```shell
docker build -t docusaurus:latest .
```

Lancement du conteneur 
```shell
docker-compose up -d
```

## Configuration du reverse proxy apache

Création d'un reverse proxy Apache sur {domain-name.fr} :

-   [Création d'un reverse proxy Apache](../apache/creation-reverse-proxy.md)

## Mise en place du certificat SSL

Il est aussi important de configurer un certificat SSL sur {domain-name.fr} :

-   [Documentation de création d'un certificat SSL avec Apache](../apache/configuration-ssl.md)

## Liens utiles : 

-   [Installation de Docusaurus](https://docusaurus.io/docs/installation)