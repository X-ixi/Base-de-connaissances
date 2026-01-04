``` toc

```

# 1. OpenSSH
SSH : shell sécurisé à distance (remplace telnet qui n'est pas chiffré)
SSH se base sur des certificats (cryptographie asymétrique), qui sont stocké par défaut dans *.ssh/id_rsa* et *.ssh/id_rsa.pub*.

Quelques commandes utiles
```bash
# Generate a key
ssh-keygen -t rsa

# Connect to host
ssh user@host

# Copy file
scp localfile user@host:/path/distantfile
```

SSH peut être utilisé avec une Yubikey plutôt qu'un certificat local.

Variante *cssh* : outil permettant de se connecter en ssh simultanément à plusieurs machines pour exécuter des commandes sur toutes à la fois.
-> Compliqué pour un grand parc (et moins bien que Ansible dans tous les cas)


# 2. Fabric et Ansible
## 2.1. Fabric
Librairie python pour l'exécution de commande à distance.
Permet de scripter facilement des actions simples

## 2.2. Ansible
### 2.2.1. Introduction
Ecrit en python, powershell et Ruby.
Permet d'administrer tous les OS (Windows, Linux, ...).
Ansible permet de gérer à distance : la configuration de la machine, le déploiement d'applications et l'orchestration.
Il **ne nécessite pas d'avoir un agent installé** sur la machine distance et utilisera SSH pour faire des commandes bash ou powershell.

Fonctionnement :
- **Inventaire** (fichier inventory.ini) : liste des machines à administrer
- **Playbook** (fichier playbook.yml) : recette des actions à faire. Il contient les cibles de l'inventaire et les tâches à réaliser.

Chaque tâche est décrite par un ensemble de clé-valeur dans le fichier yaml. Il faut voir la documentation de Ansible pour connaître les attributs à affecter.
Exemples de tâche : *apt* pour gérer les packages, *user* pour les utilisateurs, *git*, *copy*, ...

### 2.2.2. Stratégies
Les tâches sont exécutées dans l'ordre de leur déclaration. Elles peuvent être effectuées en parallèle sur plusieurs cibles sans synchronisation entre les tâches. On peut également ajouter des conditions pour exécuter ou non une tâche sur une machine.

Ansible permet également de gérer les erreurs : les modules build-ins l'intègre directement et pour les binaires custom, il est possible d'ajouter des conditions d'erreur en fonction de la réponse à l'exécution du binaire.
Par défaut, si une tâche échoue, le reste des tâches n'est pas effectué sur cette machine mais continue sur les autres. On peut définir d'arrêter tout le playbook sur toutes les machines si les tâches ont échoué sur plus de x% des machines.

Il y a également plusieurs stratégies possibles : tester sur un nombre limité de machines avant de lancer sur toutes les machines, faire la première tâche sur toute les machines avant de passer à la suivante, ou faire toutes les tâches sur une machine avant de passer à la suivante (ex : 100 jobs en parallèle pour 1000 machines).

Autres options intéressantes : limiter le nombre d'exécution en parallèle d'un tâche précise (évite de se DDoS), exécuter une tâche une seule fois par une machine et distribuer le résultat sur toutes les machines.

### 2.2.3. Sécurité des secrets
De nombreuses opérations à distance nécessite des secrets !
Ansible peut chiffrer les secrets utilisés avec un mot de passe utilisateur *ansible-vault*. Lors du lancement du playbook, Ansible demande à l'administrateur d'entrer son mot de passe.



# 3. Vagrant
Outil de gestion du cycle de vie des machines virtuelles (VirtualBox, HyperV, AWS, Azure, Docker ...) : provisionnement, administration, ...

Un fichier de configuration permet de :
- Créer la VM avec toutes ses caractéristiques (taille, fournisseurs, ...)
- Provisionner des logiciels : exécution d'un script Ansible, Chef, Puppet, ...
- Modifier la configuration réseau, les points de montage, ...

Vagrant s'utilise en ligne de commande : `vagrant up`, `vagrant status`, ...

Se combine parfaitement avec Ansible : on n'a pas besoin de fournir d'inventaire à Ansible car Vagrant va lui fournir.

