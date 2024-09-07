---
title: Backup/Restaure BDD PostgreSQL
---

# Backupp et restauration d'une base de donnée PostgreSQL

## Prérequis

-   2 serveurs sous linux ({serveur-1} et {serveur-2}) avec un accès SSH ouvert avec les utilisteurs {user-server-1} et {user-server-2}
-   Une base de donnée PostgreSQL {bdd_name}

## Backup de la base de donées sur le serveur 1

### Sur le serveur 1

Backup de la base de donnée

```shell
cd /tmp && sudo -u postgres pg_dump {bdd_name} | gzip -9 > {bdd_name}_backup.sql.gz
```

## Restauration de la BDD sur le serveur 2

### Sur le serveur 2

Récupération du backup

```shell
scp {user-server-1}@{serveur-1}:/tmp/{bdd_name}_backup.sql.gz /tmp
```

Dézip du fichier de backup

```shell
gunzip {bdd_name}_backup.sql.gz
sudo chown postgres:postgres {bdd_name}_backup.sql
```

Connexion à postgres

```shell
sudo -iu postgres
psql
```

Création de la base

```sql
DROP DATABASE "{bdd_name}";
CREATE DATABASE "{bdd_name}" OWNER "postgres" ENCODING 'UTF8' LC_COLLATE = 'fr_FR.UTF-8' LC_CTYPE = 'fr_FR.UTF-8' template=template0;
exit
```

Exécution du backup sur la base

```shell
cd /tmp/
psql -U postgres -b {bdd_name} -f {bdd_name}_backup.sql
```

## Commandes utiles

Terminer toutes les connexions à la base de donnée à restaurer

```sql
REVOKE CONNECT ON DATABASE {bdd_name} FROM public; 
```

Vérifier que toutes les connexions à la base de données sont closes

```sql
SELECT pg_terminate_backend(pg_stat_activity.pid)
FROM pg_stat_activity
WHERE pg_stat_activity.datname = '{bdd_name}';
```

