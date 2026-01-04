``` toc

```

# 1. Introduction

## 1.1. Abstraction
Un ensemble de machines veulent collaborativement résoudre un problème mais elle possède chacune une partie seulement des données du problème.

L'objectif est de définir des problèmes très simples, appelés **abstractions**, qui serviront de briques de base à des problèmes plus complexes.

Exemples d'abstraction : 
- Diffusion fiable (si j'envoie un message à tous les autres processus, alors ils le reçoivent tous)
- Diffusion causale (une message émis avant un autre est remis avant l'autre)
- Synchronisation d'horloge
- prise de snapshot (cf [[(SC&R) 1 - Systèmes répartis - observation#3. Points de contrôle et retour arrière]])
- Exclusion mutuelle
- Trouver un consensus
- Élire un leader
- ...


## 1.2. Algorithme distribué
Pour spécifier un algorithme distribué il faut définir :
- Qui sont les acteurs
- Quel est l'algorithme
- Quel est le modèle temporel : 
	- Synchrone/asynchrone
	- Modèle de défaillance (crash vs processus malveillant)
	- Communication (mémoire partagée ou par message)

Problème non résoluble : consensus asynchrone (si un seul processus crashe, le consensus peut ne jamais être atteint)


## 1.3. Algorithme de consensus
On suppose que au plus T processus peuvent crasher (sur N processus au total).

Un des processus est coordinateur et fait avancer la connaissance des autres pour qu'ils finissent par se mettre d'accord.
L'algorithme se déroule en un multiple de 3 tours :
1. Les processus indécis demande au coordonnateur la valeur
2. Le coordinateur envoie le consensus courant, les processus indécis acceptent la valeur reçue (si elle est reçue) et ne sont plus indécis
3. Le coordinateur délivre la décision courante à tous les processus
Un autre processus devient le coordinateur.

Pour contrer le problème de processus Byzantin (malveillant / aléatoire), on peut demander à chaque processus de signer ses messages.

Il faut T+1 étapes pour atteindre un consensus.