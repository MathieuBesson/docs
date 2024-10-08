---
title: Reverse proxy Apache
author :
    name : Mathieu BESSON
    linkedin : mathieubesson
---

# Création d'un reverse proxy apache

## Prérequis

-   La distribution linux ubuntu
-   Apache2 installé sur votre machine [Documentation d'installation](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-22-04)


## Configuration du reverse proxy apache

Activation des modules apache nécéssaires

```shell
sudo a2enmod ssl proxy proxy_http proxy_balancer
```

!!! info
    Pour les commandes à venir, il et nécéssaire de remplacer `{domain-name.fr}` par votre nom de domaine


Création de la configuration apache du reverse proxy vers http://localhost:8000/

```shell
vi /etc/apache2/sites-available/{domain-name.fr}.conf
```

Avec la configuration suivante :

=== "/etc/apache2/sites-available/{domain-name.fr}.conf"
```apacheconf
<VirtualHost *:80>
    # Le contact de l'admin
	ServerAdmin contact@{domain-name.fr}

    # Le nom de domaine associé au projet
    ServerName {domain-name.fr}

    # Configuration du reverse proxy
    ProxyPreserveHost On
    <Proxy *>
        Order allow,deny
        Allow from all
    </Proxy>
    ProxyPass / http://localhost:8000/
    ProxyPassReverse / http://localhost:8000/

    # Le chemin vers les logs apache
    ErrorLog ${APACHE_LOG_DIR}/{domain-name.fr}-error.log
    CustomLog ${APACHE_LOG_DIR}/{domain-name.fr}-access.log combined
</VirtualHost>
```

Tester la configuration, pour éviter d'éventuelles erreurs :

```shell
sudo apachectl configtest
```

Activer la configuration

```shell
sudo a2ensite {domain-name.fr}.conf
```

Redemmarer apache

```shell
sudo systemctl restart apache2
```

-   Vérifier le virtualhost sur [http://{domain-name.fr}](http://{domain-name.fr})
