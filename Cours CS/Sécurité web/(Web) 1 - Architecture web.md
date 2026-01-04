``` toc

```

n.talon@hackuity.io


# 1. Introduction

*Avant* : application monolithe responsable de tout, aucune séparation entre présentation de la données (frontend) et la donnée elle-même (backend). Soit tout marche, soit rien ne marche... Mais devient trop complexe, difficile à passer à l'échelle.

*Maintenant* : séparation frontend/backend, permet d'avoir plusieurs frontend pour des utilisateurs différents, permet d'utiliser du Content Delivery Network.
- Frontend Single Page Application : c'est le navigateur qui s'occupe d'incorporer les données et de générer le html (Angular, React, Vue)
- Backend API : mise à disposition de la données dans un format universel (xml, json)

Mais on peut faire mieux : **microservice** (services faiblement dépendants, où chaque services correspond à une petite application).
-> Chaque microservice est facile à déployer, ils sont indépendant donc une partie peut ne pas fonctionner sans que le reste soit par terre, et meilleure gestion de la mise à l'échelle
-> Inconvénients : nécessite des communications et une bonne gestion des erreurs


## 1.2. Communication entre services

Deux types de communication possibles :
- Synchrone : les services sont en ligne au même moment, c'est plus simple à gérer
	- *Brokerless* (pas de partie tierce) : communication direct entre les services, besoin de connaître les adresses des autres services
	- *Brokered* : communication toujours faite via la partie tierce, plus de dépendance entre les services
- Asynchrone : les services n'ont pas besoin d'être en ligne au même moment, c'est plus résilient
	- *Queue based* : utilisation d'une file d'attente, présence de plusieurs consommateurs/producteurs
	- *Stream* : flux en temps réel, si un consommateur crashe, il peut reprendre au timestamp où il s'est arrêté

Autre paradigme : *event sourcing*
Enregistrement de tous les évènements et lorsqu'on veut reconstruire une donnée, il faut rejouer ces évènements.


## 1.3. Communication haut niveau

### 1.3.1. REST
Representation State Transfer (REST)
Système de communication client-serveur : interface sans état (chaque requête contient toute les info nécessaires), uniforme et standardisé
-> Utilise de HTTP 
		- méthode : get, post avec body, put, delete, ...
		- code : 2xx (succès), 3xx (redirection), 4xx (erreur côté client), 5xx (erreur côté serveur)

### 1.3.2. GraphQL
Système de communication client-serveur permettant de demander exactement la donnée souhaitée. Un seul endpoint est exposé, le client lui envoie une requête avec un json détaillant les éléments qu'ils souhaitent récupérés et sous quel format il les veut, et le serveur fait le reste.
Meilleure granularité des données récupérées mais n'utilise pas les codes d'erreurs et ne permet pas le cache


# 2. Faire fonctionner le service

## 2.1. Container Docker
Virtualisation plus légère que des VMs. Permet de créer un conteneur fonctionnant sur toutes les plateformes.

## 2.2. Orchestration
Permet le passage à l'échelle, les mises à jour et de s'assurer qu'un certain nombre de conteneur sont up.
-> Kubernetes

## 2.3. Autres éléments
- API Gateway
- Load balancer
- Serverless : on passe le code à un fournisseur de cloud
