``` toc

```

# 1. Modélisation d'un calcul réparti

## 1.1. Introduction
Il n'y a pas de définition précise mais il faut au moins 3 critères : plusieurs processeurs, un réseau de communication inter-connectant les processeurs et des objectifs communs nécessitant une coopération entre processus.

Il y a une grand variété de systèmes répartis (taille, puissance de calcul, répartition, ...).

Des grandes classes de systèmes :
- Communication par **échange de messages** ou par **mémoire partagée**
- **Défaillances** possibles au niveau des **calculateurs** ou du **réseau**
- Système **synchrone** ou **asynchrone**

Des systèmes sont dit synchrones s'ils partagent une horloge commune et effectuent des tâches bornées temporellement.


## 1.2. Evènements partiellement ordonnés
Plusieurs types d'évènements :
- Interne
- De communication :
	- Reception
	- Emission

On peut associer à chaque processus une séquence d'évènements. On a alors des ordres partiels. Cependant pour créer un ordre global de tous les évènements, il faut une horloge commune !

Propriété de causalité entre 2 évènements et lien avec leurs horloges : *(e -> f) <=> (h(e) < h(f))*.


## 1.3. Horloges logiques
L'utilisation d'une horloge logique va permettre de capturer fidèlement la relation de causalité précédente.

### 1.3.1. Solution 1 : horloge de Lamport
Chaque processus $P_i$ gère sa propre horloge logique $h_i$.
Chaque évènement $e_i^x$ est estampillé $h_i(e_i^x)$ au moment ou il se produit.
Si $e_i^x$ est un évènement d'envoi de message alors $h_i(e_i^x)$ est transporté par le message m dans le champ noté *m.h*.

On met à jour $h_i$ comme suit :
- Si $e_i^x$ est un évènement interne ou d'émission : $h_i = h_i + 1$
- Si $e_i^x$ est un évènement de reception : $h_i = max(h_i, m.h) + 1$

On a la relation suivante *(e -> f) => (h(e) < h(f))*, cependant la réciproque est fausse ! 
=> Cette horloge ne permet pas de traduire la causalité entre évènements.


### 1.3.2. Solution 2 : horloges vectorielles
Utilisation d'un vecteur au lieu d'un entier comme horloge. Ce mécanisme a un coût plus élevé en mémoire, en temps de calcul et en taille des messages, mais il permet d'identifier les évènements concurrents. 

Il permet également de traduire la causalité des évènements : *(e -> f) <=> (h(e) < h(f))*.

Deux vecteurs sont comparables si toutes leurs composantes prises les unes après les autres sont ordonnées de la même façon : $V1 \ge V2 \Leftrightarrow \forall i, V1[i] \ge V2[i]$.
-> Si 2 vecteurs sont **incomparables** alors ils correspondent à des **évènements concurrents**.


### 1.3.3. Solution 3 : horloge globale
Une solution pour avoir un ordre global est d'avoir une horloge globale.
Mais il n'est pas toujours intéressant d'avoir un ordre global : certains évènements sont totalement décorrélés, pour d'autres on n'a pas besoin de connaître l'ordre (la reception d'un message arrive toujours après son émission), ...
=> on recherche des caractéristiques d'ordres moins fortes en pratique



# 2. Synchronisation et coordination

On étudie ici le cas d'une exclusion mutuelle dans un système réparti. Il existe d'autres problème de synchronisation que l'on ne détaillera pas ici (sémaphores, ...).

**Session critique** : suite d'instructions qui peut produire des résultats incohérents lorsque la suite est exécutée simultanément par plusieurs processus.

3 états cyclique : non-demandeur, demandeur et utilisateur de la session critique.

Propriétés à garantir :
- **Sûreté** : un processus au plus est dans l'état utilisateur
- **Vivacité** : un processus dans l'état demandeur finit par devenir utilisateur en un temps fini


## 2.1. Protocole à permissions
Exemple simple :
Pour entrer en section critique, il faut obtenir l'accord de tous les autres.

Lorsque 2 processus demandent l'accès à la section critique en même temps, il faut être capable de les prioriser (et tous les processus doivent les prioriser de la même façon).
-> utilisation d'une horloge de Lamport (qui augmente à chaque fois que le processus accède à la section critique) : on favorise la plus petit horloge qui correspond au processus le plus en retard.


## 2.2. Protocole à jeton
Un processus ne peut accéder à la section critique que lorsqu'il a le jeton et le jeton est unique.

### 2.2.1. Protocole de l'anneau
Les processus sont reliés de sorte à former un anneau. Le jeton tourne le long de l'anneau : si un processus n'a pas besoin du jeton, il le fait passer au processus suivant, etc.

Ce protocole fonctionne très bien lorsqu'il y a peu de processus et qu'ils utilisent tous régulièrement la section critique.

### 2.2.2. Protocole de Naimi-Tréhel
Gestion d'une file d'attente dynamique ou chaque demandeur pointe vers le demandeur suivant. Le processus ayant le jeton est en tête de la file d'attente.
En période calme où aucun processus n'est demandeur, le dernier processus ayant le jeton le garde et le libérera dès qu'un processus arrivera dans la file d'attente.

Comment s'insérer dans la file ?
Le dernier demandeur (fin de file) est la racine d'un arbre des derniers demandeurs connus et chaque processus à un pointeur vers un processus père dans cet arbre (le père d'un processus dans l'arbre est le dernier demandeur DD de son point de vu). Lorsqu'un processus devient demandeur, il arrive en feuille de l'arbre et dit à son père qu'il est le dernier demandeur, son père fait passer l'information à son père et actualise son pointeur de DD. L'information se propage ainsi et modifie l'arbre des DD.
Si 2 processus deviennent demandeur en même temps, l'information de leur demande se propage dans l'arbre et le premier à arriver à la racine se place dans la file d'attente (le second se placera ensuite en DD).

Ce protocole demande peu d'échange de messages pour devenir demandeur.


# 3. Points de contrôle et retour arrière

L'idée de cette partie est de permettre un retour en arrière en cas de défaillance.
Pour éviter de recommencer de zero en cas de défaillance, on peut créer des sauvegardes régulièrement (c'est valable pour tous les systèmes, répartis ou non) et repartir de celles-ci si besoin.

Un point de contrôle global est un ensemble de points de contrôle locaux, un par processus.

Lors de la création d'un point de contrôle global, les points de contrôle locaux ne sont pas forcément parfaitement synchronisés et on peut avoir des :
- Message manquant : message émis et non reçu
  -> On peut s'en sortir en les identifiant et les rejouant  
- Message orphelin : message reçu et non émis
  -> Point de contrôle incohérent que l'on ne pourra pas utilisé, il faudra faire un retour au point de contrôle précédent sur le processus qui a reçu le message orphelin aussi.

C'est difficile de définir quand faire des points de contrôle pour éviter qu'ils soient soumis a des retours arrière à cause de messages orphelins et donc inutiles.
-> on force l'ajout de points de contrôle supplémentaires pour s'assurer que tous les points de contrôle soit utilisable en cas de défaillance = points de contrôle forcés.

Protocole de Chandy-Lamport : les points de contrôle sont faits par vague. Tous les messages sont marqués avec la vague à laquelle ils appartiennent lorsqu'un processus fait un point de contrôle, il envoie un message spécifique aux autres indiquant le changement de vague et le nombre de messages qu'il a émis à la vague précédente, les autres processus vont vérifier/attendre d'avoir reçu tous les messages de la vague précédente et faire un point de contrôle eux aussi.
-> Évite tous les messages orphelins et permet d'identifier facilement les messages manquants pour pouvoir les rejouer.


