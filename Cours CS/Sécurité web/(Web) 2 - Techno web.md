``` toc

```

# 1. Javascript

Interprété nativement par le navigateur web et permet de rendre une page interactive. Permet également de déporter l'exécution du code du serveur vers le client.

Rien à voir avec Java qui exécute du byteCode dans une machine virtuelle.

Node.js a re-dynamiser le language, notamment côté serveur.

ECMAScript est une spécification du language Javascript. Tous les navigateurs n'implémente pas toutes les fonctionnalités.

Spécificité du langage :
- Déclaration de variable : *var* pour les variables globales, *let* et *const* pour les locales
- Déclaration de fonction : `function Machin(a,b) {return a+b}` ou fonction fléchée `(a,b) => return a+b`.
- Supporte les classes, l'héritage, ...
- Possibilité de créer et d'utiliser des modules avec les mots clés export/import
- Les promesses (type *Promise*) très utiles pour effectuer des actions asynchrones


# 2. Flask
Flask est un framework back-end python open-source. Plus léger et simple à prendre en main que django.
Flask inclut les fonctionnalités http par défaut mais permet surtout d'ajouter plein d'extensions.

Utilisation du module Jinja2 pour faire des templates de pages HTML avec des paramètres modifaibles

Utilisation du module sql-alchemy comme ORM

Utilisation du module flask-login pour gérer les pages protégées et non protégées


# 3. React
Historiquement : les pages étaient entièrement générée par le serveur (Server Side Rendering).
Maintenant on peut aussi faire générer la page par le client avec javascript (Client Side Rendering), mais on va quand même utiliser du SSR pour garder de bonnes performances (SEO).

Catégorie de test de sécurité sur du code :
- Statique
- Dynamique
- Sofware Code Analysis : analyse si la librairie est maintenu, a des vulnérabilité connu, ...(ex : snyk advisor)

JSX : extension de javascript permettant de faire des composants HTML

React permet de faire des composants réutilisables.
Un composant peut avoir des données de 2 façons :
- En les recevant du composant parent
- Avec un état interne propre au composant (fonction *useState*)

## XSS
XSS (Cross Site Scripting): injection de code javascript par un attaquant
Le XSS peut-être stocké (permanent car stocké dans la bdd) ou réfléchi (temporaire car dans l'url)

React se prémunit des injections malveillante d'utilisateur en assainissant les entrées utilisateur. Si dans le code `<div> {username} </div>`, on met `username=<script> ...</script>`, le script ne sera pas exécuté mais la string sera affiché.
-> Il faut se méfier des librairies qui peuvent forcer l'injection d'HTML.

On peut insérer du javascript de 3 façons :
- Avec une balise `<script>`
- Avec un handler : attribut `onClick` d'un bouton, `onError` avec une image avec un mauvaise source, etc.
- Avec un lien : au lieu de mettre `href="https://..."` ou `href="mailto:..."` on peut aussi mettre `href="javascript:..."` et le code js sera exécuté lors d'un clic.

Librairie pour assainir les entrées utilisateurs : *DOMPurify*

## Content Security Policy
Header qui permet de configurer d'où on peut récupérer des ressources (images, styles, iframe, police, script,...).



# 4. Angular

## 4.1. Typescript
Javascript avec du typage. Le typage c'est bien : permet de faire des interface, d'éviter des erreurs, de générer la documentation facilement.
Lors de la compilation, transformation en javascript.

## 4.2. Angular
Framework frontend utilisant typescript, et basé sur *npm* pour gérer les dépendances.
Lors du déploiement, utilise node JS pour servir les fichiers.
Angular fonctionne avec des composants composé de 3 fichiers : ts, html et css, et de service composé d'un fichier ts.
Les services sont stockés et exécutés côté client !
La structure de fichiers des composants et des services peut être générée automatiquement :
```
ng generate component cmpt-name
ng generate service srvce-name
```

Comme pour React, on fait du Client Side Rendering donc l'URL est modifié par Angular mais il n'y a pas de nouvelle requête au serveur quand on change de page.
