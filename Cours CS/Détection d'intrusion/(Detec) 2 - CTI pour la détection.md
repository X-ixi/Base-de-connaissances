``` toc

```
Georges Brossert - CTO SEKOIA.IO (CTI et XDR)

# 1. Définition CTI

CTI : *Cyber Threat Intelligence*
=> renseignement d'intérêt cyber

Le renseignement ('threat intelligence' en anglais) vise à connaître et maîtriser les menaces. Le renseignement désigne plutôt les processus mis en place que l'information collectée.

Une menace doit avoir :
- Des capacités de nuire
- Une intention hostile
- Une opportunité

TTP : *Technic, Tactic and Procedure*
Si des renseignements sont récupérés sur les TTP, l'attaquant doit changer complètement d'attaque (outils, ...) pour ne pas être détecté.


Différents niveaux de renseignement :
- Long-terme :
	- Stratégique (groupe de menace)
	- Tactique (TTP)
- Court-terme :
	- Opérationnel
	- Technique (artefact permettant la détection)


Déroulement du renseignement :
1. Des chercheurs produisent des rapports sur une attaque ou un groupe d'attaquants
2. Des analystes extraient les renseignements stratégiques, tactiques, opérationnels et techniques de ces rapports
3. Diffusion de ces informations :
	- Bulletins courts de texte
	- Rapports détaillés
	- Rapports techniques dans un format compréhensible par des machines (STIX par exemple)


# 2. Production de renseignement
 
 Cycle de renseignement :
 1. Orientation : déclencheur du cycle : un incident, une alerte de l'ANSSI un secteur d'activité, une demande des décideurs, ...
 2. Collecte de données : mise en place de capteurs techniques (honeypot, télémétrie des antivirus et firewall, ...) et humains (OSINT, ...)
 3. Traitement des données : structuration et enrichissement
 4. Analyse : contextualisation et interprétation
 5. Diffusion

*Virus Total* : plateforme gratuite d'analyse de fichiers (utilise près de 70 antivirus et des sandboxes pour tester les fichiers transmis).


# 3. Analyser une intrusion
Cyber kill chain :
1. Reconnaissance
2. Préparation (infrastructure, malware, ...)
3. Livraison (mail de phishing, clé USB, ...)
4. Exploitation (exécution de code, élévation de privilèges, mouvement latéral, ...)
5. Installation du malware
6. Connexion au C2
7. Action sur objectifs

cf [[(Pentest) 1 - Test d'intrusion#2. Étapes d'un test d'intrusion]]


## Protocole de diffusion 
TLP (Traffic Light Protocol) :
- WHITE : pas de limite
- GREEN : limité à une communauté (pas d'impact sur l’émetteur si divulgué)
- AMBER : limité à une liste de diffusion défini (ex : tous membres d'une entreprise)
- RED : limité à une liste nominative de personnes

Chatham house rule :
Lors d'une discussion, les participants peuvent repartir avec l'information échangée et l'utiliser par la suite mais ne doivent pas divulguer la source de l'information.

