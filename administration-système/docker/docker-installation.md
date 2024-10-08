---
title: Docker installation
author :
    name : Mathieu BESSON
    linkedin : mathieubesson
---

# Installation de Docker sur Ubuntu

## Prérequis

- La distribution linux ubuntu

## Nécéssaires avant installation  

Mise à jour des repositories apt
```shell
sudo apt update && sudo apt upgrade
```

Installation des dépendances 
```shell
sudo apt -y install lsb-release gnupg apt-transport-https ca-certificates curl software-properties-common
```

Téléchargement des clés de certifications 
```shell
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker.gpg
```

Définition du repository officiel de Docker
```shell
sudo add-apt-repository "deb [arch=$(dpkg --print-architecture)] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

Mise à jour de la liste des paquets apt disponibles 
```shell
sudo apt update
```

## Installation 

Installation de docker et de docker-compose 
```shell
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose
```

Activation de docker au boot de la machine
```shell
sudo systemctl start docker && sudo systemctl enable docker
```

Vérification l'installation (et de la version) de docker 
```shell
docker -v
```

## Utiliser docker avec un autre utilisateur que root 

Pour utiliser docker en temps que non-root 

!!! info
    Remplacer `{new-user}` par le nom d'utilisateur souhaité


```shell
sudo newgrp docker
sudo usermod -aG docker {new-user}
```

```shell
docker --help
```