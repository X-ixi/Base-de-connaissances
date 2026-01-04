``` toc

```

# 1. Briques de base
Les **cookies** c'est important... Ils ont une durée de vie limitée et un périmètre limité pour qu'ils ne soient pas envoyés à tous les sites.

*Same Origin Policy* : interdit à un site de demander des informations à un autre site depuis le contexte d'exécution du navigateur (sauf si autorisé explicitement avec CORS).

Authentification :
- Basic : on envoie `<username>:<password>` en base 64
- Digest
- ...

# 2. BurpSuite
Version pro : 450 euros par an par utilisateur

BurpSuite agit comme un proxy entre le navigateur et le serveur web, ce qui permet de modifier les requêtes à envoyer.

Fonctionnalités :
- SiteMap : dresse l'arborescence des URL
- Scope : limite l'affichage aux domaines sélectionnés
- Intercept : place le proxy en mode bloquant qui arrête toutes les requêtes et attend une action de l'utilisateur
- Logger : historique de l'ensemble des requêtes
- Repeater : répéter une requête en la modifiant légèrement
- Intruder : envoie automatique de requête pour faire du bruteforce
  -> Limiter à 1 requête par seconde en version gratuite


# 3. OWASP Top 10

## A07: Identification and authentication failures
Exemples :
- Bruteforce possible sans blocage (pas de limite de tentatives ou de CAPTCHA)
- Mauvaise implémentation de MFA
- Utilisation de mots de passe faibles autorisées par le site

## A01 : Broken Access Control
Exemples : 
- Contournement du contrôle d'accès en modifiant URL ou paramètre HTTP
- Utilisation de méthode non protégé (PUT au lieu de POST, ...)

## A03 : Injection
Vulnérabilité du à un manque de filtrage des données utilisateur
Exemples : 
- Injection SQL
- Injection de code Javascript (XSS)
- Injection de commande
- Injection de template

### Injection SQL
**UNION based** Injection SQL : on ferme la première requête (avec une quote par exemple), puis on utilise le mot clé UNION et on complète avec une autre requête. 
Attention, il faut que les résultats des 2 requêtes aient le même nombre de colonnes. Pour trouver le nombre de colonnes présentes dans la requête initial, on peut utiliser ORDER BY n, en incrémentant n (le n° de la colonne) jusqu'à avoir une erreur.
**ERROR based** : on exploite le fait que certaines fonctions exécute l'entrée utilisateur et renvoie la valeur de retour dans la réponse
-> Permet de dump une grande quantité de données d'un coup
**BLIND based** : on envoie des requêtes qui selon une condition, met un sleep de quelques secondes ou non. Cela permet de deviner un caractère après l'autre.
-> Ok pour dump des hash ou des emails mais pas une bdd entière

Procédure d'exploitation :
- Manipulation manuelle avec Burp Repeater jusqu'à trouver une injection possible
- Automatisation du dump : utilisation de *SQLMap*

SQLMap peut tout faire : découverte du site, tests de toutes les routes, ... = gros porc
=> A ne pas utiliser dès le début ! D'abord trouver une faille.

Les données que l'on souhaite extraire :
- @@version
- user()
- current_user()
- database()
- @@hostname
- le nom des tables ou des colonnes
- les données des tables

A la fin de requête SQL, on veut mettre un symbole de terminaison pour être sur que la partie injectée ne se trouvent pas dans une requête plus complexe avec une partie à sa droite : 
- `/*` : commentaire multiligne
- `#` : commentaire monoligne
- `--` : opérateur stop

Contournement :
- Si les espaces ne sont pas autorisées, on peut utiliser : `/**/`, `%09` (tabulation), `%0a` (new line)
- Si on ne peut pas utiliser `=`, on peut utiliser `1 < 2`, ...

### Injection XSS
Injection de code javascript qui sera exécuté par le navigateur de la victime.
Cela peut permettre de voler les cookies, les jetons, de faire une redirection, ...

Plusieurs types de XSS :
- *Stored* : injection dans la base de données et attaque sur tous les utilisateurs qui charge la page
- *Reflected* : l'attaquant envoie un lien malicieux à la victime

Payload type :
- `<script>`
- `<img src=x onerror=code/>`
- `<svg/onload=code/>` : pratique car pas de caractère espace dedans
- ...
On peut utiliser JQuery comme alternative au javascript pur : souvent plus court et moins détecté par les pare-feu applicatif WAF.

Mesures préventives :
- WAF
- *Content Security Policy* (CSP) : définit quelles peuvent être les sources de chaque type de ressource (html, police, code javascript, ...)
- Cookie : 
	- flag *HttpOnly* : cookie accessible seulement par le navigateur pour être envoyé dans des requêtes HTTP, donc pas accessible à javascript
	- flag *Secure* : transmission possible du cookie seulement en HTTPS
	- flag *SameSite* : restreint à quels sites le cookie peut être transmis
- SOP : Same Origin Policy - restriction des sites auxquels un site peut accéder
  -> CORS pour ajouter des exceptions


## Server Side Request Forgery
Un serveur a une fonctionnalité qui lui demande de faire une requête à un autre serveur : image, API, ...
L'attaquant qui exploite une SSRF peut détourner ce comportement pour faire une requête à une autre ressource que celle légitimement souhaitée : scan interne, ...

-> Requête OPTIONS pour connaître les méthodes autorisées