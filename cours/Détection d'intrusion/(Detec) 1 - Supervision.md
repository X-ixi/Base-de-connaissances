``` toc

```

# 1. Comprendre la menace

Motivations des attaquants : 
- Lucrative : cybermercenaires
- Politique : hacktivistes, cyberterroristes
- Militaire
- Ludique : adolescent désoeuvré
- Technique : hacker
- Emotive : employé mécontent

Deux types d'attaques :
- Non ciblées : approche automatisée, peu évolué et peu discret
   ex : stealer, ransomware, ...
- Ciblées : attaquants structurés (équipe, moyen technique), avec un objectif précis
   ex : APT : Advanced Persistent Thread

## 1.1. Connaître l'infrastructure d'attaque
![[1_infra_attaque.png]]

**Serveur C2** : serveur de Command & Control. 
Infra interne : mise en place sur le long terme, pendant plusieurs mois éventuellement
-> ex : permet de télécharger la charge utile
Infra externe : serveur éphémère pour rendre difficile les enquêtes
-> ex : DNS temporaire qui résout l'adresse du C&C


## 1.2. Déroulement d'une attaque
Exemple du déroulement de l'attaque de TV5 Monde (diffusion de propagande islamique)
1. Phase d'exploration : service exposé sur internet (RDP, VPN, ...)
2. Phase de compromission : scan du réseau interne, compromission d'une machine, d'un compte de l'AD, ...
3. Phase de collecte : accès aux documents internes de la cible, aux mails, récupération d'identifiants et mot de passe
4. Phase de sabotage : altération de configuration, de documents, du site internet, ...



# 2. La réponse à incident

Les grandes étapes de la réponse à incident sont :
1. Détection
2. Confinement : empêcher l'attaquant de se propager davantage
3. Eradication : suppression des accès de l'attaquant sur le système
4. Remise en état : reconstruction et sécurisation du SI

Dans 60% des attaques, des données sont volées dans les premières heures.
Plus de 50% des attaques ne sont pas découvertes avant plusieurs mois. 

<u>Note</u> : temps de détection moyen d'une attaque de 220 jours, d'où le besoin de conserver des logs pendant un an.

## 2.1. Confinement de l'incident
Objectifs : limiter la propagation sur le SI et les capacité de contrôle d'un attaquant (connexion au serveur C2 par exemple).

Pour cela, il est possible de mettre en quarantaine une partie du réseau. Si une seule machine est infectée, elle peut être isolée du réseau puis éteinte, ou isoler puis surveiller de près pour obtenir des informations sur les attaquants.

## 2.2. Analyse de l'incident
Dans un premier temps, il faut collecter un maximum d'information : copie des disques, traces réseau, évènements remontés au SIEM.

Les questions à se poser :
- Quel est le vecteur d'infection initiale ?
- Quels sont les fichiers accédés, créés ou supprimés ?
- Quelles sont les communication réseau interne/externe de l'attaquant ? Quelles sont leur destination et objectifs (DOS, spam, exfiltration de données) ?
- Quel est le périmètre de compromission ?

## 2.3. Eradication
L'objectif est de nettoyer les machines infectées. Pour ce faire, il faut supprimer les comptes frauduleux, désactiver les comptes usurpés / changer leur mot de passe, supprimer les ports dérobées et corriger les vulnérabilités exploitées.

## 2.4. Remise en état
Réalisation d'un plan d'actions pour faire fonctionner à nouveau le SI le plus rapidement possible : réinstallation des postes/serveurs, déploiement d'un AD sain, déploiement de meilleures mesures de sécurité (VLANs, filtrage, ...), collecte et suivi de logs, ...

## 2.5. Contre attaque
Plusieurs niveaux de contre attaque sont possibles :
![[1_pyramid_of_pain.png]]



# 3. Organisation équipe SOC / CERT

Différent profils :
- Analyste de niveau 1 : analyse en temps-réel, peu expérimenté, trie les alertes
- Analyste de niveau 2 : analyse des alertes relevé par les analystes de niveau 1
- Analystes plus spécialisés : recherche de schéma et de tendance dans les logs, analyse de la menace, met en place le plan d'action de réponse à incident, analyste forensique

## 3.1. Fiches réflexe
Fiches utilisées par les analyste pour savoir comment réagir face à une catégorie d'incident (DDoS, ransomware, phishing, ...).
Chaque étape de la réponse à incident y est décrite : détection, confinement, éradication et remise en état.

Exemples de fiches : https://github.com/certsocietegenerale/IRM

## 3.2. Entraînement des équipes
Objectif : former les équipes, tester l'efficacité et la réaction et valider l'architecture de défense.

Plusieurs approches :
- Outil de simulation d'attaquant automatique
- Red team en conditions réelles
- Simulation d'un groupe d'attaquant : utilisation des mêmes outils et méthodes que le groupe simulé (https://attack.mitre.org/resources/adversary-emulation-plans/)



# 4. Analyse forensique

## 4.1. Analyse de logs
Parmi les postes qui sont infectés (ou suspectés d'être infectés), on recherche des IOC (indices de compromission : hash de malware connu, ...) et des patterns de comportement malveillant.

Outil classique de mouvement latéral : mimikatz, PsExec, wmic, WinRM
-> détection des logs laissés par ces outils

## 4.2. Analyse de support de stockage
Les disques durs sont copiés sans altérer le disque initial (boitier physique permettant le WriteProtect). Ensuite, identification et accès au partition du disque. S'il y a des partitions connues, on parcourt les fichiers à l'aide du système de fichiers, sinon, on les reconstruit à partir des données brutes. On identifie également les fichiers cachés et supprimés.

Des fichiers peuvent être cachés à de nombreux endroits : dans la section boot d'une partition non bootable, dans des espaces non alloué de partition, ...

Il faut également rechercher des techniques de persistance, par exemple dans l'UEFI, le grub, ...

## 4.3. Analyse de mémoires
Certaines activités suspectes ont lieu intégralement en mémoire volatile (RAM). Pour que l'analyse forensique soit complète, on dump la mémoire volatile pour l'analyser. 
Pendant l'analyse, on identifie les processus en cours d'exécution, les connexions au réseau, les zones mémoire et DLL suspectes.



