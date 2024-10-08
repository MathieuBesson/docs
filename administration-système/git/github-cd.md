---
title: Minimaliste CI/CD Github action
author :
  name : Mathieu BESSON
  linkedin : mathieubesson
links:
  - title: SSH pour Github Actions
    url: https://github.com/appleboy/ssh-action#setting-up-ssh-key
  - title: Mise en place d'une CD pour une application nodejs avec Github Actions
    url: https://gist.github.com/danielwetan/4f4db933531db5dd1af2e69ec8d54d8a
---

# Mise en place d'une CD simple avec Github et Github Actions

## Prérequis

- Un compte github
- Un serveur sous linux

## Configuration de l'utilisateur sur le serveur 

Création de l'utilisateur github-action
```shell
sudo adduser github-action
```

Générer une paire de clé SSH  
```shell
# Connection en tant qu'user github-action
sudo su github-action
# Création de la paire de clé
ssh-keygen -t rsa
```

Créer le dossier de base du projet  
```shell
sudo mkdir -p /var/www/projet && sudo chown -R github-action:github-action /var/www/projet
```

## Configuration du repository git

Création d'un repository git avec un fichier yaml précisant les jobs à lancer à chaque push sur la branch main.

=== "./github/workflows/main.yml"
```yaml
# Workflow actions
name: Workflow CD

on:
  # Trigger push on branch
  push:
    branches: [ main ]

jobs:
  build:
    # Type of runner
    runs-on: ubuntu-latest

    steps:
    - name: Deploy using ssh
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.PRIVATE_KEY }}
        port: ${{ secrets.PORT }}
        script: cd /var/www/projet && git pull
```

Copier la clé privé de l'utilisateur `github-action` dans les clés de déploiement du projet : 

```shell
cat /home/github-action/.ssh/id_rsa
```
Dans le repository Github > Settings > Secrets and variables > Actions > Actions secrets > [New repository secret]

Ajouter 4 variables : 

```yaml
HOST : ip du serveur
PORT : port SSH du serveur
PRIVATE_KEY : clé privée de l'utilisateur github-action sur le serveur
USERNAME : github-action
```

Ajouter la clé public de l'utilisateur github-action lui permettant de pull depuis le serveur sur le repository Github
```shell
cat /home/github-action/.ssh/id_rsa.pub
```

Dans le compte Settings > SSH and GPG keys > [New SSH key]

## Mise en place de l'automatisation sur le serveur

Faire un premier git clone sur le serveur

```shell
cd /var/www/projet/
git clone git@github.com:user/projet.git .
```

À partir de maintenant chaque push entrainera le déploiement du code sur le serveur

Il est possible de retrouver les logs de chaque déploiement sur le repository dans l'onglet "Actions"