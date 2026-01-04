``` toc

```

# 1. Introduction

Un protocole est un enchaînement ordonné d'opérations. Ces opérations font parties d'algorithmes cryptographique.

Pour utiliser un algorithme de manière sécurisé, il faut un protocole sécurisé. Même si les algorithmes sont sûrs, le protocole peut ne pas être sécurisé.

## 1.1. Exemple : protocole de Diffie-Hellman
Soit p nombre premier et alpha générateur de Zp* = {1, ..., p-1}
Alice choisit a, calcule alpha^a (mod p) et l'envoie à Bob
Bob choisit b, calcule alpha^b (mod p) et l'envoie à Alice
-> On a alors un secret partagé alpha^(ab) (mod p)
cf [[(Crypto) 3 - Chiffrement asymétrique#4.3. Protocole de Diffie-Hellman]]

L'algo est sécurisé mais le protocole est vulnérable aux attaques Man In The Middle.
-> Pour s'en protéger, il faut signer les messages

## 1.2. Caractéristiques
Les protocoles cryptographique doivent avoir les propriétés suivantes :
- **Complétude** : si le protocole se déroule normalement, la sécurité est assurée
- **Robustesse** : il n'existe pas de stratégie d'attaque permettant au protocole de s'exécuter sans que la sécurité soit garanti 

## 1.3. Validation
Deux façons de valider un protocole :
1. Utiliser les standards et bonnes pratiques => c'est le sujet de ce cours
2. Prouver la complétude et la robustesse


# 2. Protocoles académiques

Notations :
```
A => B : data               -> A envoie "data" à B
A => B : {Nb}Kpriv_A        -> A envoie Nb signé avec sa clé privée, à B
A => B : {Nb}Kpub_B         -> A envoie Nb chiffré avec la clé publique de B, à B
```

## 2.1. Définitions
Protocole de communication : échange de données sur un canal de communication, à priori non sécurisé.
-> Hypothèse : l'attaquant a accès à tout ce qui passe sur le réseau

Propriétés communément souhaitées :
- Authentification
- Confidentialité
- Intégrité
- Unicité : l'attaquant ne peut pas rejouer un message


## 2.2. Protocole d'authentification
Objectifs : l'entité est bien celle qu'elle prétend être, et elle est en ligne (authentification récente)

Protocoles en 2 phases :
1. **Identification** : l'entité dit qui elle est
2. **Authentification** : l'entité prouve qui elle à dit être

Différent moyens d'authentification (voir [[(Crypto) 6 - Authentification]]).

### 2.2.1. Authentification faible
Identification avec le login et authentification avec un mot de passe
-> Protocole statique (messages identiques), donc vulnérable aux attaques de rejeu (même si les messages sont chiffrés, car chiffré toujours de la même façon)

Pour éviter le rejeu, il faut encapsuler l'authentification dans un canal de communication sécurisée (cf plus loin).

### 2.2.2. Challenge / Response
Identification puis envoie du challenge (différent à chaque réalisation du protocole) et attente de la réponse 
-> Protocole dynamique
ex : Bob envoie un nombre aléatoire à Alice qui le chiffre avec une clé secrète partagée entre Alice et Bob, ou qui le signe avec sa clé privée.

### 2.2.3. Authentification mutuelle
Si Bob s'authentifie auprès d'Alice puis Alice s'authentifie auprès de Bob, cela ne fonctionne pas !
Scénario :
1. Eve demande à Alice de s'authentifier avec le challenge Ne, Alice renvoie {Ne}Kab
2. Alice demande à Eve de s'authentifier et envoie le challenge Na, Eve ne répond pas tout de suite
3. Eve demande à nouveau à Alice de s'authentifier et lui envoie le challenge Na, Alice renvoie {Na}Kab
4. Eve répond à la première demande d'authentification (étape 2) avec le nombre {Na}Kab et se fait passer pour Bob

Il faut que les authentifications des 2 parties soient simultanées.

Protocole symétrique :
```
A => B : Alice, Na
B => A : {Na, Nb} Kab
A => B : {Nb} Kab
```

Protocole asymétrique :
```
A => B : Alice, Na
B => A : {Na, Nb} Kpriv_B
A => B : {Nb} Kpriv_A
```

Pour avoir une authentification forte, il faut combiner 2 éléments parmi les 3 suivants :
- Quelque chose que l'utilisateur **connait** (mot de passe, code PIN, ...)
- Quelque chose que l'utilisateur **possède** (téléphone, clé USB, ...)
- Quelque chose que l'utilisateur **est** (biométrie)


## 2.3. Protocole de partage de clé
Objectifs : après l'exécution du protocole, Alice et Bob ont un secret partagé.

Principalement 2 approches :
- Cryptographie symétrique avec une tierce partie (serveur de clé) 
- Cryptographie asymétrique avec une infrastructure de gestion de clés publiques (PKI)


### 2.3.1. Protocole basique
Hypothèse : Alice partage un secret avec le serveur `Kas` et Bob partage aussi un secret avec le serveur `Kbs`.
```
A => S : A, B                   -> A demande à S une clé pour échanger avec B
S => A : {Kab, {Kab}Kbs}Kas
A => B : {Kab}Kbs
```

Ce protocole est vulnérable à une attaque par rejeu : 
1. Eve se fait passer pour Alice et demande au serveur à communiquer avec Eve 
2. Idem avec Bob
3. Eve intercepte la demande d'Alice au serveur et renvoie la réponse de l'étape 1
4. Eve intercepte le message d'Alice à Bob et obtient Kae
5. Idem pour Kbe
6. MITM parfait avec Kae et Kbe
```
E(A) => S : A, E                -> E se fait passer pour A
S => E : {Kae, {Kae}Kes}Kas

E(B) => S : B, E
S => E : {Kbe, {Kbe}Kes}Kbs

A => E(S) : A, B                 -> E se fait passer pour S
E(S) => A : {Kae, {Kae}Kes}Kas
A => E(B) : {Kae}Kes             -> E intercepte le message pour B et obtient Kae

...                              -> idem pour Kbe puis MITM
```


### 2.3.2. Protocole avec identification
Contre-mesure : identification de A et B
```
A => S : A, B
S => A : {B, Kab, {Kab, A}Kbs}Kas
A => B : {Kab, A}Kbs
```

Problème : on ne sait pas de quand date cet échange
Scénario d'attaque : 
1. Eve réussi à récupérer un message de demande d'Alice au serveur et à récupérer Kab (hypothèse forte mais pas irréaliste)
2. Lorsque Alice demande une clé au serveur, Eve intercepte la demande et lui renvoie le message du serveur qu'elle connait de l'étape 1 (vieille clé)
3. Eve connait le Kab que Alice va utiliser avec Bob


### 2.3.3. Protocole de Needham-Scroeder
Contre-mesure : horodatage (pour éviter les problèmes de synchronisation, utilisation d'un compteur ou d'un aléas)
```
A => S : A, B, Na
S => A : {Na, B, Kab, {Kab, A}Kbs}Kas
A => B : {Kab, A}Kbs
B => A : {Nb}Kab                   -> permet d'authentifier A directement après
A => B : {Nb-1}Kab
```

Cela protège d'une attaque de type MITM, mais cela n'empêche pas Eve de se faire passer pour Alice auprès de Bob, ou inversement.

L'ajout d'un aléas assure à Alice qu'elle parle bien au serveur mais ne donne aucune information à Bob (la clé peut avoir été générée il y a longtemps et Eve peut impersonner Alice).

Eve peut aussi se faire passer pour Bob car Bob n'est pas authentifié dans le protocole. A la fin du protocole, Eve envoie un aléas à Alice (Eve ne connait pas Kab), qui pense que cet aléas est {Nb}Kab et va renvoyer {Nb-1}Kab. Eve ne connaissant pas Kab, elle ne pourra pas continuer d'échanger avec Alice mais Alice pense néanmoins que le protocole s'est bien passé.


On ajoute 2 éléments au protocole :
- Un horodatage (timestamp T)
- Une authentification de Bob

Ce protocole est très proche de **Kerberos** : récupération d'un ticket auprès du serveur puis utilisation de ce ticket auprès d'un service (Bob dans notre cas).
```
A => S : A, B, Na
S => A : {Na, B, Kab, {Na, Tab, Kab, A}Kbs}Kas
A => B : {Na, Tab, Kab, A}Kbs
B => A : {Na-1, Nb}Kab                   -> authentifie B
A => B : {Nb-1}Kab                       -> authentifie A
```



## Protocole d'échange de clé asymétrique (PKI)

```
A => S : A, B
S => A : {Kpub_B, B}Kpriv_S
A => B : {Kpub_A, A}Kpriv_S, {{Kab}Kpriv_A}Kpub_B
```
Kab est signé par Alice et chiffré avec la clé publique de Bob.

Attaque possible : Bob utilise la clé Kab pour se faire passer pour Alice auprès de Charlie


Contre mesure : identification
...
horodatage


## Protocole d'accord de clé
Avant, Alice générait la clé partagée, maintenant c'est Alice et Bob qui génère la clé

Diffie-Hellman est un exemple de protocole d'accord de clé


slide 56 : "A=>E: {Nb}*E*pub"
Eve se fait passer pour Alice auprès de Bob



# 3. Protocole de la vraie vie

## 3.1. PAP : PPP Authentication Protocol
PPP : point-to-point
PAP consiste à faire une authentification faible avec login/mot de passe.

Les conditions d'utilisation de ce protocole nécessite une connexion point à point où l'interception est impossible.
-> C'est possible si les messages sont encapsulés dans un canal de communication sécurisé.


## 3.2. CHAP : Challenge Handshake Authentication Protocole
Authentification avec Challenge / Response

-> ne peut pas servir d'authentification mutelle, comme vu plus haut


PAP / CHAP vs RADIUS (Remote Authentication Dial In User Service)
historique : 
- CS accès sur le modem (équipement pas cher avec peu de comptes), tout le monde se connecte sur le même compte -> accès PAP
- On remplace le modem par un interrupteur pouvant lié 2 positions : serveur d'authentification et réseau de CS -> RADIUS : il faut se connecter au server d'auth pour que celui ci envoie la commande a l'interrupteur de donner l'accès au réseau (le serveur d'auth utilise toujours PAP)

Maintenant : utiliser pour le wifi, etc. Ce principe de radius peu encapsuler d'autres protocoles (PAP, CHAP, EAP, ...) et joue seulement le role d'interrupteur


## Protocole ISO
Authentification unilatéral (ex : détecteur de mouvement envoyer un heartbeat à la centrale)
...
-> algo solides


## GSM
BTS : base station (équivalent de l'interrupteur du radius)
AUC : authentication center

Le flux entre BTS et AUC est en clair dans la majorité des cas.
"Mobile ID" est en réalité l'ID de la SIM.

Protocole utilisé pour les téléphone mobiles.
Dans la carte SIM, il y a un ID et une clé secrète 
- Pour y avoir accès, l'utilisateur doit rentrer son code PIN = authentification de l'utilisateur
   -> fait au démarrage
- La SIM s'authentifie auprès de l'AUC :
	- Le mobile envoie l'ID de la SIM au BTS, qui la fait passer à l'AUC
	- L'AUC en déduit la clé partagé Ki, il prépare également un challenge (RAND), dont la résultat est SRES, et renvoie tout ça au BTS
	- Le BTS envoie le challenge au mobile
	- La carte SIM calcule la réponse au challenge et la renvoie à la BTS
	- La suite des échanges est chiffré avec Ki
  -> fait régulièrement

L'implémentation de l'auth de la SIM dépend de chaque fournisseur (**algo propriétaire**), mais tous utilise le même chiffrement avec la clé Ki

Note : les appels sont chiffrés comme vu précédemment mais pas les SMS

Et pour le roaming à l'étranger ? Lors de la 1ère connexion, l'AUC en france envoie pleins de triplets (RAND, SRES, Ki) au relais avec qui il a des accords à l'étranger.


## Carte à puce
Fonctionne presque comme GSM : 
- SIM -> carte bancaire
- Mobile -> TPE
- BTS & AUC -> Banque GIE (ou VISA, etc)

Besoin du code PIN, sauf sans contact et exception des péages

Déroulement de l'auth :
- Valeur d'auth présent sur la carte qui correspond à  la signature RSA de certaines information de la carte (signé avec la clé privée de GIE)
  -> la clé publique de GIE est présente dans les TPE
  - Code PIN
  - Info de transaction envoyé de la carte vers le TPE + MAC de ces infos calculé avec la clé de la carte (16 valeurs hexa présent sur les factures de CB)
    -> envoie (IDcarte, MAC(Info_transaction, Kcarte))
    -> le TPE n'est pas capable de vérifier le MAC, il doit l'envoyer à la banque qui se charge de vérifier (parfois, pas d'accès internet donc le TPE effectue la transaction)

Aujourd'hui, les cartes sont plus puissantes et possèdent une clé privée qui calcule dynamiquement une valeur d'auth (pour eviter les attaques qui récupèrait une VA et créait une fausse carte et faisait des transaction avec de faux MAC sur des TPE non connecté à internet).



# 4. SSL/TLS

Contexte : protocole créé pour faire du commerce électronique. Les propriétés visées étaient donc : confidentialité, intégrité et authenticité d'une transaction. Le client veut que son identité ne soit pas connu des autres clients et que le commerçant soit authentifié alors que le commerçant souhaite simplement que le moyen de paiement soit valide.

SSL : Secure Socket Layer
TLS :Transport Layer Securité
Historique : SSL v1, v2, v3 (1995) puis TLS 1, 1.1, 1.2 et 1.3 en 2018. 

Le but était de sécuriser HTTP mais comme SSL est implémenté au niveau des sockets, il est utilisable sur tous les protocoles TCP (HTTP, FTP, SMTP, ...).
-> Mise en place d'un canal de communication sécurisé (i.e. transforme la liaison en liaison point à point non-interceptable).

Propriété de sécurité de SSL / TLS :
- Confidentialité : chiffrement avec une clé de session
- Intégrité : MAC avec une clé de session
- Authenticité : authentification du serveur (le client n'est pas authentifié mais ne peut pas changer au cours de la session)
- "Libre commerce" : certificat pour les serveurs (facile pour un commerçant d'obtenir un certificat)

Déroulement en 2 phases :
1. Initialisation : négociation des algo crypto utilisé (AES, SHA-256, RSA, ...), authentification du serveur et génération des clés de session
2. Communication : échange sécurisé avec les clés de session


## 1ère phase
1ère phase très simplifié
```
C => S : Nc
S => C : Ns, Cert{Kpub_S}
C => S : {PMS}Kpub_S
```
Nc sert à identifier le client : le serveur ne veut pas connaître l'identité du client mais simplement s'assurer qu'il parle toujours au même client.
Cert{Kpub_S} : certificat de la clé publique du serveur
PMS (PreMaster Secret)
-> Les clés utilisées dans la 2ème phase dépendent de Nc, Ns et PMS

### Schéma complet
```
1) C => S : SCL, Nc
2) S => C : CLL, Ns, Cert{Kpub_S}
3) C => S : {PMS}Kpub_S
4) C => S : MAC(MS, [1,2,3])
5) D => C : MAC(MS, [1,2,3,4])
```
SCL (Supported Ciphers List) : liste des algo supportés par le client
CCL (Chosen Ciphers List) : liste des algo choisis par le serveur pour la 2ème phase
MS (Master Secret) : clé générée à partir de PMS, Nc et Ns 

Les étapes 4 et 5 permettent de s'assurer de l'intégrité de l'échange : si un attaquant intercepte un message et le modifie, l’émetteur s'en rendra compte en voyant le MAC envoyé par son destinataire.

La génération des clés se fait à l'aide de 2 fonctions comme suit :
![[generation de cles.png]]

Distinctions entre SSL et TLS :
- Différence sur les fonctions précédentes fct1 et fct2 : SSL utilise SHA-1 et MD5 tandis que TLS utilise des "Pseudo Random Fonction" issues de HMAC MD5 et HMAC SHA-1.
- Différence sur le calcul des MAC (étape 4 et 5)

Plutôt que le client soit le seul à générer la clé PMS, on peut utiliser Diffie-Hellman pour construire le secret partagé.

Identification du serveur avec son certificat puis authentification sous forme de "Challenge/Response" avec l'étape 3 en challenge et l'étape 5 en response (seul le serveur peut déchiffrer PMS et donc calculer MS et le MAC).

Toute la sécurité du protocole repose sur la robustesse du certificat publique du serveur, ou de Diffie-Hellman.


## 2ème phase
Chiffrement des échanges
...
message chiffré || MAC(message chiffré)
-> permet de vérifier l'intégrité avant de déchiffrer le message (limite le DoS car vérifier la signature va plus vite que déchiffrer le message)


Signer avant de chiffrer ou l'inverse ?
En toute rigueur il faut signé la lettre avant de la mettre dans l'enveloppe pour assurer qu'on a pris connaissance de son contenu, mais en pratique on chiffre avant de signer (dans la majorité des cas)
cf [[(Crypto) 4 - Intégrité symétrique et hachage#5. Intégrité et confidentialité]]

Et le rejeu ?
On ajoute un numéro de séquence (n° initial partagé dans le matériel crypto généré par la MS, puis incrémenté à chaque échange)
-> besoin de synchronisation et de recevoir les messages dans le bon ordre (donc ça ne peut fonctionner que sur **TCP** - UDP ne garantit pas l'ordre ni la non perte de messages)

calcul des MAC un peu différent pour SSL et TLS


## Options avancées
### Authentification du client
```
1) C => S : SCL, Nc
2) S => C : CLL, Ns, Cert{Kpub_S}**
3) C => S : Cert{Kpub_C}, {PMS}Kpub_S        -> envoie du certificat client
4) C => S : {Fct(MS, [1,2,3])}Kpriv_C        -> signature du client
5) C => S : MAC(MS, [1,2,3, 4])
6) D => C : MAC(MS, [1,2,3,4, 5])
```


### Ré-ouverture d'une session SSL
Ajout d'un identifiant de session renvoyé par le serveur à l'initialisation.
Le client envoie cet identifiant au serveur pour ré-ouvrir la session : cela permet de se passer de l'étape 3.


### Bi-clé de session
Pour assurer une confidentialité plus importante : utilisation d'une bi-clé serveur différente pour chaque session.
Utilisation d'une clé très forte (Spriv, Spub) et d'une clé faible (S'priv, S'pub)

Objectif : que les militaire puisse déchiffrer l'étape 3 car la clé utilisée est faible, pour autant on ne peut pas impersoner le serveur car la clé faible S'pub est signé par la clé forte Spriv.


### Re-négociation
Re-négocier la connexion SSL pour se connecter à un autre service qui nécessite plus (ou moins) de sécurité
-> étape 1 à 5 identique mais encapsulé dans le canal de communication déjà en place (i.e. chiffré avec les clé Esc et Ecs) = permet de négocier un nouveau matériel cryptographique qui sera utilisé par la suite


### Heartbeat de session SSL
Message de heartbeat pour maintenir une connexion même si elle n'est pas utilisé
```
C => S : {PING || MAC(...)}Ecs
S => C : {PONG || MAC(...)}Esc
```
Il y a un timer dans les connexions TCP et il faut des messages régulièrement pour renouveler le timer.


## Failles
Un protocole peut-être non sûr même si les algorithmes utilisés sont sûrs.
-> Erreurs de spécification ou d'implémentation des protocoles

### Attaque au niveau crypto
Lorsqu'une faille est découverte dans un algorithme crypto, les protocoles l'utilisant sont aussi vulnérable.

Ex : attaque par oracle sur le padding en mode CBC
L'attaquant voit les chiffré (C1, C2, C3) de (M1, M2, M3) et connait tous les octets de M2 sauf le dernier. Si on donne (C1, X, C3) à l'oracle, il sera en mesure de dire si le padding obtenu à la fin du déchiffrement est correct ou non (0x01 pour un padding de 1, 0x02 0x02 pour un padding de 2, etc).
...
Possible théoriquement mais il faut un oracle de déchiffrement et connaître tous les octets d'un bloc et ça permet de récupérer la valeur d'un seul octet
-> exploitation forte de cette faille dans le cas de SSL

Comment connaître les valeurs de tous les octets sauf 1 d'un bloc ?
Position de proxy (MITM) et ajout d'un "requestBody" dans la requête -> l'attaquant connait ce body et se débrouille pour en faire un bloc à un octet près

Oracle ?
Le serveur

=> Solution : changer le padding utilisé, ou chiffré le dernier bloc 2 fois


### Problème lié à la bi-clé de session
Factorisation de S'pub possible car elle est choisi faible volontairement
Cependant, cette attaque nécessite du temps pour factoriser la clé (RSA 512bits par exemple).

Certains sites utilisés toujours la bi-clé dans le cas d'export et on peut demander à communiquer avec ces services en mode export.

idem avec Diffie-Hellman dans le cas du mode export (cela impose l'utilisation de paramètre faible pour DH - 512 bits).


### Heartbeat attack
L'implémentation OpenSSL en C utilisait la fonction `strcpy` -> buffer overflow
Le client faitsait un PING avec comme message "A" (par exemple) et indiquait que son message faisant 2000 caractère, le serveur renvoyait alors le message PONG avec "A" et 2000 caractères présent dans la pile
-> fuite d'info sensible du serveur (matériel crypto, ...)


Recommandation : privilégier TLS 1.2 et 1.3 (et tolérer TLS 1.1 et 1.0)

