
# Découverte
2 ports sont ouverts : 22 et 80
## Serveur web
Avec un dirb sur le service http on repère 2 répertoires intéressants :
- /admin
- /etc : contient un dossier squid avec :
	- passwd : contient `music_archive:$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.`
	- squid.conf : contient les lignes suivants :
```
auth_param basic program /usr/lib64/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Squid Basic Authentication
auth_param basic credentialsttl 2 hours
acl auth_users proxy_auth REQUIRED
http_access allow auth_users
```

Sur la page /admin :
- On trouve une note d'un admin disant qu'il a mal configuré un proxy squid
- On télécharge également un fichier `archive.tar`

## Fichier archive
L'archive contient toute une arborescence de fichier, qui correspond à un backup créé avec Borg
On installe borgbackup.
Il faut un mot de passe pour décoder l'archive -> on utilise john avec rockyou sur le hash trouvé dans le fichier passwd.
Le mot de passe est : `squidward`

Une fois décompressé, on obtient une arborescence complète de fichier (desktop, document, ...).
Dans une note on trouve le identifiants suivants : `alex:S3cretP@s3`

# SSH
On se connecte avec les identifiants précédents et on obtient le flag utilisateur.

## Priv esc
```bash
alex@ubuntu:~$ sudo -l
Matching Defaults entries for alex on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alex may run the following commands on ubuntu:
    (ALL : ALL) NOPASSWD: /etc/mp3backups/backup.sh
```

On peut exécuter le script backup.sh en tant que root !
On affiche le contenu de ce script avec `cat` : ce script sert à compresser des fichiers et il offre la possibilité de passer une commande en argument qui sera exécuter à la fin du script (merci chatGPT pour l'explication).

On exécute `sudo /etc/mp3backups/backup.sh -c "cat /root/root.txt`
Et on a le flag !


