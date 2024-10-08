---
title: Crypter transfère seveurs
author :
    name : Mathieu BESSON
    linkedin : mathieubesson
---

# Crypter un transfert de fichier entre 2 serveurs 

## Prérequis

- Pour la suite des explications, il est nécéssaire de remplacer les variables suivantes celles des machines en cours : 
    - {ip-server-1} et {ip-server-2} par les adresses IP des serveurs
    - {user-server-1} et {user-server-2} par les utilisateurs utilisés pour le transfert
    - {name-key-server-2} par le nom de la clé GPG utilisé pour chiffrer le fichier
- 2 serveurs sous linux (serveur 1 et serveur 2) avec un accès SSH ouvert avec les utilisteurs {user-server-1} et {user-server-2}


Installation de GPG
```shell
sudo apt-get update && sudo apt-get install gnupg -y
```

## Clés SSH

Génération des clés SSH sur le serveur d'envoi (serveur 1)
```shell
ssh-keygen -t ed25519
```

Copie de la clé public sur le serveur de réception (Automatisation de la connexion SSH par clé)
```shell
ssh-copy-id {user-serveur-2}@{ip-serveur-2}
```

## Clé GPG

Dans l'ordre des choses :

- Le serveur 1 qui va chiffrer le fichier à transférer avec la clé public du serveur 2
- Le serveur 2 qui va déchiffrer le fichier reçu avec sa clé privée

Il est donc nécéssaire de créer une paire de clés sur le serveur 2 :
```shell
gpg --batch --passphrase '' --quick-gen-key {name-key-server-2} default default never
```

Récupération de la clé public générée : 
```shell
gpg --output ~/{name-key-server-2}.key --armor --export {name-key-server-2}
```

Import de la clé public du serveur 2 sur le serveur 1 :
```shell
# Sur le SERVEUR 2
cat ~/{name-key-server-2}.key
# Copier le contenu de la clé
# Sur le SERVEUR 1
vi ~/{name-key-server-2}.key
# Coller le contenu de la clé dans le fichier
# Import de la clé public
gpg --import {name-key-server-2}.key
# On authentifie la clé importé 
gpg --sign-key {name-key-server-2}
```

## Chiffrement et transfert du fichier

Création du dossier et du fichier
```shell
mkdir /tmp/folder
echo "Hello world" > /tmp/folder/file.txt
```

Création d'un hash du fichier pour valider son intégrité
```shell
sha256sum /tmp/folder/file.txt > /tmp/folder/file.txt.hash
```

Compression du dossier
```shell
cd /tmp/
tar -zcvf folder.tar.gz folder
```

Chiffrement du fichier
```shell
gpg --encrypt --sign --armor -r {name-key-server-2} folder.tar.gz
```

Transfert du fichier vers le serveur 2
```shell
scp /tmp/folder.tar.gz.asc {user-server-2}@{ip-server-2}:/tmp
```

## Reception du fichier

Déchiffrement du fichier sur le serveur 2
```shell
gpg --decrypt /tmp/folder.tar.gz.asc > /tmp/folder.tar.gz
```

Décompression du dossier
```shell
tar -xvf folder.tar.gz
```

Vérifier que les sorties de ces 2 commandes donne le même résultat pour attester de la non-modification
```shell
sha256sum /tmp/folder/file.txt | awk '{print $1}'
cat /tmp/folder/file.txt.hash | awk '{print $1}'
```

Le fichier est maintenant disponible sur le serveur 2. 

Nous sommes sur que le fichier n'as pas été modifier lors du transfert et que le transfert à été chiffrer d'une part par le protocole SSH et d'autre part par GPG.

