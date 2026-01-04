*Dernière modification : Mai 2025*
```toc

```


Notes issues du module ISO27001 Foundation de la plateforme d'e-learning fournie par Certi-trust.

# Introduction

La norme ISO27001 est un ensemble d'exigence pour la mise en place, la maintenance et l'amélioration continue d'un Système de Management de la Sécurité de l'Information (SMSI).
## SMSI
Objectifs du SMSI :
- Assurer la confidentialité, l'intégrité et la disponibilité de l'information via des processus de gestion des risques
- Donner aux parties intéressées l'assurance que les risques sont gérés de manière adéquate

Un SMSI est basé sur une approche par les risques : il vise à anticiper les menaces et à mettre en place des défenses appropriées.

## Plan-Do-Check-Act
Appliqué au SMSI, le modèle PDCA permet de renforcer la sécurité :
- **Plan** : identification des besoins et objectifs, évaluation des risques et définition des mesures de sécurité ;
- **Do** : mise en oeuvre des mesures, et formation et sensibilisation ;
- **Check** : surveillance et contrôle, audits internes et revues de direction ;
- **Act** : amélioration continue et réponse aux incidents.

## Vocabulaire
- Un **système de management** est un ensemble d'éléments d'un organisme visant à établir des objectifs et des processus permettant d'atteindre ces objectifs.
- Un **système de management intégré (SMI)** permet de mutualiser certains processus entre plusieurs système de management (sécurité, qualité, ...), ce qui permet notamment de réduire le nombre de jours d'audits.

- Les **actifs primaires** sont aussi appelés *valeur métier* et correspondent aux actifs informationnel et aux processus.
- Les **actifs secondaires** correspondent aux *biens supports* : matériel, logiciel, réseau, personnel, site et organisation qui supportent les actifs primaires.
 
- La **déclaration d'applicabilité** résume les mesures de sécurité choisies par l'organisation, justifie leur adoption ou exclusion, et est un élément clé de la norme.
- La **revue de direction** vise à garantir que la direction évalue régulièrement et systématiquement les performances du SMSI, en s'assurant qu'il reste pertinent, adéquat et efficace.

# ISO27001
## Organisation de l'ISO27001
L'ISO27001 comprend 10 chapitres. Les 3 premiers correspondent à l'introduction et la définition des termes et du périmètre du SMSI.
Les 7 suivants sont :
4. **Contexte de l'organisation** : comprendre le contexte organisationnel, les besoins et attentes des parties intéressées, et définir le périmètre du SMSI ;
5. **Leadership** : la direction doit démontrer son leadership et son engagement dans le SMSI ;
6. **Planification** : identifier les risques et opportunités, établir les objectifs de sécurité et les plans pour les atteindre ;
7. **Support**  : l'organisation doit mettre à disposition les ressources et compétences nécessaires, sensibiliser et communiquer auprès des parties intéressées ;
8. **Fonctionnement** : traiter les risques de l'information, gérer les changements et la documentation ;
9. **Évaluation des performances** : surveillance, mesures, analyse et évaluation/révision des contrôles de sécurité de l'information. Aussi, audit interne et revue de direction ;
10. **Amélioration** : traiter les recommandations des audits et des revues, et amélioration continue du SMSI.

Pour reprendre le modèle PDCA, les clauses 4 à 7 sont la phase PLAN, la 8 est la phase DO, la 9 est la phase CHECK et la 10 la phase ACT.
![[ISO27001-PDCA.png]]
L'annexe A de l'ISO27001 contient l'ensemble des mesures de sécurité de référence devant être appliquées. Cette annexe fait référence aux mesures de l'ISO27002.

## ISO27001 vs ISO27002
### ISO27001
L'ISO27001 est une norme, c'est à dire un ensemble d'exigences.
Les clauses principales doivent impérativement entre suivies (verbe "doit").
L'annexe A contient 93 mesures de sécurité réparties en 4 types de mesures : organisationnelles, liées aux personnes, physiques et technologiques.
### ISO27002
L'ISO27002 est une directive, c'est-à-dire un ensemble de bonnes pratiques.
Les clauses principales sont des recommandations destinées à être adoptées volontairement ("devrait").
Elle est composée de 93 mesures de sécurité. Elle contient également des conseils de mise en oeuvre.

## Evolution de ISO27001 2013 à 2022
La création d'une nouvelle version de l'ISO27001 en 2022 permet notamment de mieux prendre en compte les nouvelles technologies et menaces (cloud, IoT, IA, ...).
L'annexe A passe de 114 mesures réparties en 14 chapitres dans la version 2013 à 93 mesures réparties dans 4 catégories dans la version 2022.

Cette évolution permet également la convergence avec d'autres réglementations ou directives, tel que celle du NIST Cybersecurity Framework.

A partir de mai 2024, il n'est plus possible de réaliser une certification initiale ou un renouvellement selon la norme ISO27001:2013.
Pour le 31 octobre 2025, tous les certificats existants devront avoir été mis à jour vers l'ISO27001:2022.


# Clause 4 - Contexte de l'organisation

La clause 4 précise 4 points principaux :
1. *L'organisation et son contexte* : détermine les enjeux internes et externes pouvant avoir une incidence sur le SMSI ;
2. *Les besoins et attentes des parties intéressées* : détermine les parties intéressées concernées par le SMSI, leurs exigences et le traitement de ces exigences par le SMSI ;
3. *Le domaine d'application* : détermine les limites et l'applicabilité du SMSI en fonction notamment des enjeux internes et externes (4.1) et des exigences des parties intéressées (4.2) ;
4. *Le SMSI* : clause précisant que l'organisation doit mettre en oeuvre, tenir à jour et améliorer en continu un SMSI

## Clause 4.1 - Organisation et son contexte

L’organisation doit évaluer et comprendre les enjeux externes et internes qui sont pertinents pour ses objectifs et ses activités. Ces informations doivent être prises en compte pour l’établissement, l’implémentation, la maintenance et l’amélioration du SMSI de l’organisation et l’établissement des priorités.
### Enjeux externes
Définition : facteurs qui façonnent l'environnement dans lequel l'entreprise cherche à atteindre ses objectifs.
Cela peut inclure l'environnement culturel, social, politique, légal, réglementaire, financier, technologique, économique, naturel et concurrentiel, au niveau international, national, régional ou local.
Exemples : client dictant la demande, marché et concurrence imposant un positionnement stratégique, réglementations imposant des exigences, ...

### Enjeux internes
Définition : rouages internes de l'organisation, qui peuvent influencer la manière dont l'organisation gérera les risques.
Exemples : gouvernance (rôles et responsabilité), objectifs et stratégies, capacité en termes de ressources et de connaissances, relation avec les parties prenantes internes, ...

Le SMSI doit être aligné avec la culture, les processus, la structure et la stratégie de l'organisation.

## Clause 4.2 - Besoins et attentes des parties intéressées

Les parties intéressées, aussi appelées parties prenantes, sont le réseau élargi d'entités qui ont un intérêt direct ou indirect dans les activités de l'entreprise.
Exemples : consommateurs, employés, actionnaires, fournisseurs, partenaires, concurrents, gouvernement, ...

Une fois les parties intéressées identifiées, il convient de :
- Déterminer leurs besoins et leurs attentes, c'est-à-dire les exigences obligatoires et déclarées (ex :  réglementations, contrats, politiques internes, ...) mais également celles qui sont sous-entendues ;
- Déterminer leur pouvoir et intérêt afin d'identifier leur niveau d'influence sur les décisions et actions de l'entreprise ;
- Fixer les objectifs et priorités à atteindre pour réduire les risques que les besoins et attentes des parties intéressées ne soient pas satisfaits.

## Clause 4.3 - Domaine d'application du SMSI
Le domaine d'application du SMSI correspond au périmètre des informations devant être protégées. Ce périmètre n'est pas initialement défini par un lieu de stockage ou un moyen d'accès mais bien par les informations qu'il contient.

Il convient de définir le domaine d'application selon :
- Les frontières physiques
- Les frontières organisationnelles
- Les frontières du SI

Le périmètre défini doit avoir du sens et une cohérence vis-à-vis des risques pour l'entreprise. En particulier, il faut que les informations et processus à l'intérieur du périmètre soit suffisamment segmenté des informations et processus à l'extérieur pour ne pas porter à confusion.
Finalement, le domaine d'application doit être cohérent avec les exigences des parties intéressées et les objectifs du SMSI.


# Clause 5 - Leadership

**Leadership** : art de motiver un groupe de personnes pour agir vers la réalisation d’un but commun. Il ne s'agit pas seulement de diriger/ordonner mais d'inspirer !

La clause 5 précise 3 points principaux :
- *Leadership et engagement* : la direction doit faire preuve de leadership et d'engagement pour le SMSI ;
- *Politique de Sécurité de l'Information (PSI)* : la direction doit établir et communiquer une politique de sécurité de l'information ;
- *Rôles et responsabilités* : la direction doit s'assurer que les rôles et responsabilités pertinentes sont attribuées et communiquées au sein de l'organisation.

## Clause 5.1 - Leadership et engagement
Les dirigeants de l'organisation doivent prouver leur leadership et leur engagement en faveur du SMSI en :
- Établissant les politiques et les objectifs du SMSI ;
- Intégrant les exigences du SMSI aux processus métier afin que ces derniers en tiennent compte ;
- Allouant les ressources nécessaires : finances, personnel et temps ;
- Communicant sur l'importance du SMSI ;
- Vérifiant l'efficacité du SMSI (suivi des audits et contrôles, revue de direction) ;
- Promouvant l'amélioration continue ;

Le leadership ne se limite pas à la haute direction. Les autres managers, dans leurs domaines de responsabilité respectifs, doivent également faire preuve de leadership en matière de SMSI. La direction principale doit les soutenir et les guider dans cette démarche.

Les "dirigeants" au sens de l'ISO27001 sont les membres de la direction et les autres managers importants.

## Clause 5.2 - Politique de Sécurité de l'Information
La PSI :
- Est une preuve de l'engagement de la direction ;
- Sert de document officiel énonçant les intentions et objectifs de la direction. Toute action, décision ou procédure pourra être évaluée par rapport à la PSI ;
- Fournit un cadre à partir duquel des procédures et directives opérationnelles seront développées (actions concrètes) ;
- Est un outil de communication auprès des tiers ;

La PSI doit inclure les objectifs de sécurité de l'entité, elle doit inclure l'engagement de satisfaire aux exigences applicables, et elle doit inclure l'engagement d'œuvrer pour l'amélioration continue du SMSI.

Elle doit être communiquée au sein de l'organisation, et mise à disposition des parties intéressées quand c'est approprié.

## Clause 5.3 - Rôles et responsabilités
La direction doit attribuer la responsabilité et l'autorité pour :
- S'assurer que le SMSI est conforme aux exigences de l'ISO27001 ;
- Rendre compte à la direction des performances du SMSI.

La direction doit s'assurer que ces rôles sont communiquées au sein de l'organisation.
Cela est une preuve de l'engagement de la direction.

La direction peut également nommé un "sponsor" : il est le représentant de la direction et fait le lien avec les équipes opérationnelles du SMSI (RSSI, ...). Il s'assure que les objectifs du SMSI soit atteint, garantit que les ressources nécessaires soient mises à disposition, ...

La figure suivante représente une adaptation à la sécurité de l'information du principe « des trois lignes de maîtrise pour une gestion des risques et un contrôle efficaces ».
![[3 lignes de maîtrise.png]]

On a alors :
- *Un comité de gouvernance* animé par le RSSI auprès des dirigeants (DG, DAF, DSI, DRH, ...), qui permet d'assurer l'alignement de la stratégie de sécurité avec la stratégie de l'organisation. Les résultats d'avancement et des travaux d'audit interne et externe sont présentés. Ce comité peut mener la revue de direction ;
- *Un comité opérationnel* constitué des managers de différents service, qui permet d'assurer le suivi du programme de gestion des risques. Il coordonne la conduite des projets et contribue à la mise en œuvre de la stratégie définit par le comité de gouvernance de la sécurité.
Des tableaux de bord sont utilisés lors de ces comités.

Les 3 lignes de maîtrise des risques :
- Les managers : ils gèrent les risques et mettent en oeuvre les mesures correctives ;
- Les équipes de surveillance de l'efficacité du SMSI ;
- Les auditeurs internes : assurance d'une indépendance totale.

# Clause 6 - Planification

La clause 6 précise 3 points :
- *Risques et opportunités* : mettre en place un processus d'appréciation des risques de sécurité de l'information, tenant comptes des éléments de contexte (cf clause 4), et de définir un plan de traitement des risques.
- *Objectifs et plans* : établir des objectifs de sécurité clairs et mesurables. Puis planifier leur atteinte en identifiant les responsables, les ressources et les échéances. 
- *Planification des modifications* : toute modification nécessaire du SMSI doit être effectuée de manière planifiée.

## Clause 6.1 - Risques et opportunités
Cet étape doit permettre d'identifier les risques et opportunités de la situation actuelle pour l'organisation.
A prendre en compte :
- Les risques associés à la perte de confidentialité, d'intégrité et de disponibilité des processus et informations métiers ;
- Les attentes des parties intéressées ;
- Les opportunités éventuelles : innovations liées au numérique, accès à de nouveaux marchés, ... ;
Il est conseillé de formaliser une matrice SWOT (Strengths, Weaknesses, Opportunities, Threats).
Cf la méthode EBIOS RM pour l'identification, la priorisation et le traitement des risques.

## Clause 6.2 - Objectifs et plans
L'organisation définit les objectifs du SMSI en accord avec :
- Les risques et opportunités identifiés.
- La PSI
- La vision globale de l'organisation
Chaque objectif doit être mesurable.

Pour chaque objectif, planification de :
- Ce qui sera fait
- Les ressources nécessaires
- Le responsable
- L'échéance
- La méthode d'évaluation

Les objectifs permettent de définir les mesures de sécurité (annexe A) incluses ou excluses de la déclaration d'applicabilité (DDA).

## Clause 6.3 - Planification des modifications
La clause 6.3 de l'ISO 27001 exige que toute modification nécessaire du SMSI soit effectuée de manière planifiée.


# Clause 7 - Support
Cette clause revient sur 5 points cruciaux dans la mise en place du SMSI :
- *Ressources* : allocation des moyens nécessaires au SMSI (capital, technologie et personnel) ;
- *Compétences* : avoir les personnes compétentes pour accomplir les tâches liées au SMSI (identifier, former et évaluer les compétences des individus sur le SMSI) ;
- *Sensibilisation* : les employés doivent connaître les politiques, mais également comprendre pourquoi elles existent et quels sont leur rôle ;
- *Communication* : une sensibilisation efficace n'est possible que grâce à une communication claire et ciblée (en interne et en externe) ;
- *Documentation* : les processus, politiques et procédures doivent être documentés afin de d'assurer la pérennité des pratiques de sécurité (et facilite les audits).

## Clause 7.1 - Ressources
L'organisation doit s'assurer que les ressources suffisantes sont mises à disposition du SMSI. Cela inclut :
- Personne
- Finance
- Technologie de l'information
- Information documentée
- Communication (ex : soutien de la direction)
Ces ressources doivent être revues périodiquement pour s'assurer de leur cohérence.

## Clause 7.2 - Compétences
Le management des compétences nécessaires au SMSI passe par les étapes suivantes :
- Identifier les compétences nécessaires des personnes dont le travail a une incidence sur la sécurité de l'information
- S'assurer que ces personnes sont compétentes sur la base de : formation initiale ou continue, ou expérience appropriée

Le cas échéant :
- Mener des actions pour acquérir les compétences nécessaires (formation, encadrement, recrutement, sous-traitance)
- Evaluer l'efficacité des actions entreprises
- Documenter les preuves de ces compétences

## Clause 7.3 - Sensibilisation
Les personnes à sensibiliser sont les collaborateurs internes, mais peuvent également inclure des sous-traitants, fournisseurs ou partie intéressées.
La sensibilisation a 3 objectifs :
- Sensibiliser à la PSI et aux procédures associées
- Faire prendre conscience aux personnes de leurs contributions attendues au SMSI
- Souligner l'importance de la conformité aux politiques et procédures

Un plan de sensibilisation doit être élaboré : réunions de sensibilisation, lettre d'information, publication de rapport d'incident, briefing de la direction, ...

## Clause 7.4 - Communication
Un plan de communication doit définir :
- *Quoi* : définir quels sujets du SMSI nécessitent une communication (politiques, procédures, incidents, changements, ...) ;
- *Pourquoi* : définir les objectifs de la communication (sensibiliser, éduquer les collaborateurs, changer des comportements, obtenir un soutien, ...) ;
- *Avec qui* : définir le public cible de chaque communication (interne : départements, équipes techniques, direction, ..., ou externes : partenaires, fournisseurs ou clients) ;
- *Par qui* : définir les émetteurs légitimes en fonction des publics cibles
- *Quand* : définir la fréquence, le moment opportun (période de l'année), ou l'instant précis en réaction à un événement ou incidents ;
- *Comment* : définir le canal de communication le plus approprié (réunion, courriel, bulletin d'information, ...).
Si possible, définir des moyens d'évaluer les performances de ces communications (analyse d'impact, questionnaire, ...).

## Clause 7.5 - Documentation

Les documents suivants sont obligatoires à la conformité ISO27001 :
- Domaine d'application du SMSI (cf 4.3) : périmètre du SMSI
- PSI (cf 5.2)
- Processus et résultats de l'appréciation des risques (cf 6.1 et 8.2)
- Processus et résultats du traitement des risques (cf 6.1 et 8.3)
- Déclaration d'applicabilité (DDA)
- Objectifs de sécurité (cf 6.2)
- Compétences des personnes (cf 7.2) : documents identifiant les compétences nécessaires et les mesures prises pour les remplir
- *Contrôles des informations documentées* (cf 7.5) : ces contrôles garantissent que les documents nécessaires au SMSI sont correctement créés, mis à jour, approuvés, stockés, protégés, accessibles, révisés et éliminés si nécessaire
- Procédures de sécurité (cf 8.1)
- Résultats de surveillance et de mesures (cf 9.1) : indicateurs d'efficacité du SMSI
- Programme et résultat d'audit interne (cf 9.2)
- Revues de direction (cf 9.3)
- Non-conformités, actions correctives et résultats (cf 10.2)

L'étendue des informations documentées pour un SMSI dépend de chaque organisation : de sa taille, de ses processus internes, de la compétence des personnes, ...
Par conséquent, l'organisation doit définir d'elle-même les documents supplémentaires pertinentes pour son SMSI : procédures techniques détaillées, journal d'incidents de sécurité, liste des actifs, rapports d'audits internes, PAS, BIA, politique d'amélioration continue, ...

La gestion documentaire du SMSI doit faire l'objet d'une grande rigueur :
- Chaque document utilise un format commun indiquant son titre, sa version, son historique de modification, ...
- Un processus de modification, revue et approbation est suivi systématiquement.
- Chaque document est disponible pour les personnes ayant le besoin d'en connaître, avec le niveau de droits appropriés (consultation, modification).


# Clause 8 - Fonctionnement
La clause 8 précise 3 points :
- *Planification et contrôles opérationnels* : planification, mise en oeuvre et contrôle des processus définis à la clause 6 ;
- *Appréciation des risques* : analyse des risques à intervalles réguliers ou sur changements significatifs ;
- *Traitement des risques* : mise en oeuvre du plan de traitement des risques ;

Définition : « le **risque** de sécurité de l'information est exprimé en termes de combinaison des conséquences d'un événement de sécurité de l'information et de la probabilité associée d'occurrence » (ISO 27001, clause 3.9.4)

Risque = Actif x Menace x Vulnérabilité
Risque important = Actif précieux x Source de menace déterminée x Présence de vulnérabilités 

## Clause 8.1 - Planification et contrôles opérationnels
L'organisation doit planifier, mettre en oeuvre et contrôler les processus nécessaires pour satisfaire aux exigences et réaliser les actions déterminées dans l'Article 6.
Pour cela, elle doit :
- Établir des critères
- Mettre en oeuvre les contrôles conformément à ces critères

Des informations documentées doivent être conservées lors de la réalisation des processus afin de permettre leur contrôle.

## Clause 8.2 - Appréciation des risques
La norme ISO27001 n'impose pas de méthode d'analyse de risque : elle donne le "quoi faire" et non le "comment faire".
Exemple de méthode : EBIOS RM, MEHARI, OCTAVE, CRAMM, ...

L'appréciation des risques se réalise en 3 grandes étapes :
- *Identification* d'un scénario menant à la réalisation d'un événement redouté :
	- Identification des actifs à protéger
	- Identification des sources de risques (externe ou interne)
	- Identification des vulnérabilités (faiblesse exploitable par une source de risque) et des contrôles (mesures de sécurité ralentissant un attaquant) existants
- *Évaluation* des risques :
	- Gravité : impact associés (financier, opérationnel, réputationnel, ...)
	- Vraisemblance : probabilité d’occurrence
- *Priorisation* des risques selon leur gravité et vraisemblance

Des échelles doivent être définies afin d'évaluer les risques de la manière la plus objective possible.

L'appréciation des risques est un processus proactif, à répéter :
- A intervalle régulier
- Lors de changement significatif sur le SI ou sur l'écosystème de menace

## Clause 8.3 - Traitement des risques
Pour chaque risque identifié, l'organisation décide de le traiter d'une des façon suivante :
- L'*accepter*
- L'*éviter* : arrêt de l'activité associée
- Le *réduire* : ajout de mesures de sécurité, pour ensuite accepter le *risque résiduel*
- Le *transférer* : exemple, souscription d'une assurance sécurité

Cette décision se base sur des critères définis à l'avance, par exemple : gravité/vraisemblance du risque, coût de son traitement et bénéfices attendus, contraintes opérationnelles ajoutées par les mesures de traitement, ...

La décision finale est prise par le **propriétaire du risque**. C'est lui le porteur ou responsable du risque, par conséquent il doit avoir le pouvoir hiérarchique suffisant pour assurer son traitement (ex : la direction). Cette décision est formalisée.

Ces choix donnent lieu à l'élaboration d'un **plan de traitement des risques**.

Pour réduire un risque, il est conseillé de s'inspirer des 93 mesures de sécurité présentes dans l'annexe A de l'ISO27001.
4 natures de mesures selon leurs objectifs : 
- Dissuasive : décourager le passage à l'acte (sanction en cas de non-respect des politiques, affichage d'avertissement à la connexion, sensibilisation, ...) ;
- Préventive / de protection : empêcher un incident de sécurité (2FA, mise à jour, filtrage réseau, antivirus, ...)
- Défense : détecter un incident et le contrer (journalisation, IDS, SIEM, SOC, procédure de gestion d'incident, isolation réseau, ...)
- Résilience : limiter les dégâts et rétablir un fonctionnement normal (PCA/PRA, gestion de crise, retour d'expérience)

Les mesures de sécurité peuvent être classées par catégorie :
- Organisationnelles, humaines, technologiques et physiques
- Techniques, managériales, administratives et légales

## Déclaration d'Applicabilité
La DDA liste les 93 mesures (aussi appelé contrôle) de l'annexe A et précise pour chacune :
- Si elle est appliquée, avec une justification de son application
- Si elle n'est pas appliquées, avec une justification de son exclusion

Elle correspond dont à la stratégie de l'organisation pour traiter les risques et atteindre ses objectifs de sécurité.
Les auditeurs s'appuieront sur ce document pour savoir quelles mesures de sécurité contrôler.

# Clause 9 - Évaluation des performances


# Clause 10 - Amélioration continue


# Processus de certification
Pour être certifié, il faut
- Mettre en oeuvre un SMSI :
	- Créer un business case pour la certification ISO27001 (motivations suffisantes)
	- Obtenir l'aval de la direction
	- Mettre une équipe en place et, le cas échéant, recruter un consultant externe
	- Établir le périmètre du SMSI (sites, processus, informations, ...)
	- Implémenter un système de gestion documentaire
	- Implémenter un système de mesure de la performance (indicateurs)
	- Apprécier les risques et développer des mesures de sécurité
	- Établir les objectifs et la politique du SMSI (PSI)
	- Former les différentes parties prenantes
- Réaliser un audit interne et corriger les écarts
- Mener une revue de direction
- Être certifié par un organisme indépendant

Les organismes de certification sont les organismes possédant une accréditation les autorisant à auditer un SMSI puis délivrer la certification ISO27001.

Audit de certification :
- Audit initial : revue documentaire, puis vérification de la mise en oeuvre du SMSI
- Audit de surveillance : tous les ans, prend 1/3 du temps de l'audit initial
- Audit de renouvellement : tous les 3 ans, prend 2/3 du temps de l'audit initial