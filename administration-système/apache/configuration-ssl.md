---
title: SSL Let's Encrypt Apache2
---

# Configuration SSL avec Apache2 et Let's Encrypt

## Prérequis 

- Une installation d'apache2 fonctionnelle
- Un virtualhost apache fonctionnelle sur le port 80

!!! info

    Pour les commandes à venir, il est nécéssaire de remplacer `{domain-name.fr}` par votre nom de domaine.

## Installation de Let's Encrypt

```bash
sudo apt install certbot python3-certbot-apache
```

## Activation du modules SSL apache 

```shell
sudo a2enmod ssl rewrite
```

Redémarrage d'apache

```shell
sudo systemctl restart apache2
```

## Installation du certificat

Création du certificat en mode non intéractif

```shell
certbot certonly --noninteractive --agree-tos --apache -d {domain.com}
```

Mise en place de la conf SSL apache

```apacheconf title="/etc/apache2/sites-available/{domain.com}.conf"
<VirtualHost *:80>
        ServerName {domain-name.fr}

        ErrorLog ${APACHE_LOG_DIR}/{domain-name.fr}-error.log
        CustomLog ${APACHE_LOG_DIR}/{domain-name.fr}-access.log combined

        RewriteEngine On
        RewriteRule .* https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
</VirtualHost>

<VirtualHost *:443>
        ServerName {domain-name.fr}
        ServerAlias {domain-name.fr}

        # La configuration apache
        # ... 

        # Le chemin vers les logs apache
        ErrorLog ${APACHE_LOG_DIR}/{domain-name.fr}-error.log
        CustomLog ${APACHE_LOG_DIR}/{domain-name.fr}-access.log combined

        # SSL 
        SSLCertificateFile /etc/letsencrypt/live/{domain-name.fr}-0001/fullchain.pem
        SSLCertificateKeyFile /etc/letsencrypt/live/{domain-name.fr}-0001/privkey.pem
        Include /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>
```

Ne pas oublier de placer votre configuration à l'intérieur de celle-ci et de modifier `{domain-name.fr}` par votre nom de domaine.

Redémarrage d'apache

```shell
sudo systemctl restart apache2
```

## Renouvellement automatique du certificat

À la main sur le serveur

```shell
sudo certbot renew --dry-run && systemctl restart apache2
```

!!! info
     Une tache cron est déjà parametré pour le renouvellement des certificats par certbot dans ce fichier `/etc/cron.d/certbot`


## Suppression d'un certificat

Suppression du certificat

```shell
sudo certbot delete --cert-name {domain-name.fr}
```

Supression des liens vers le certificat dans la conf apache 

```shell
vi /etc/apache2/sites-available/{domain-name.fr}.conf
```

Redémarrage d'apache

```shell
sudo systemctl restart apache2
```
