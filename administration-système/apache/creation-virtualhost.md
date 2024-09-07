---
title: Virtualhost apache
---

# Création d'un VirtualHost Apache

## Prérequis

- Une installation de Apache2 sur une distribution linux

## Création du virtualhost

!!! info
    Pour les commandes à venir, il est nécéssaire de remplacer `{domain-name.fr}` par votre nom de domaine

-  Créer une configuration dans /etc/apache2/sites-availables

```shell
vi /etc/apache2/sites-available/{domain-name.fr}.conf
```

Avec la configuration suivante : 


```apacheconf title="/etc/apache2/sites-available/{domain-name.fr}.conf"
<VirtualHost *:80>
    # Le contact de l'admin
	ServerAdmin contact@{domain-name.fr}
    # Le lien vers la racine du projet
	DocumentRoot  /var/www/project
    # Le nom de domaine associé au projet
	ServerName {domain-name.fr}

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
sudo systemctl reload apache2
```

- Vérifier le virtualhost sur [http://{domain-name.fr}](http://{domain-name.fr})
