---
title: Deban IP fail2ban
---

# Deban une IP de fail2ban

## Prérequis

- La distribution linux ubuntu avec fail2ban installé et configuré

## Récupérer l'IP banni sur le serveur

```shell
sudo fail2ban-client status sshd
```

output
```
Status for the jail: sshd
|- Filter
|  |- Currently failed:	0
|  |- Total failed:	0
|  `- File list:	/var/log/auth.log
`- Actions
   |- Currently banned:	0
   |- Total banned:	0
   `- Banned IP list: x.x.x.x
```

## Débannir l'IP

```shell
sudo fail2ban-client set sshd unbanip x.x.x.x
```