```toc

```

# 1. Principe

Différence fondamentale : 
- Passer du chiffré au clair doit être difficile
- Passer du clair au chiffré peut être facile

On utilise un coupe de clés (clé publique, clé privée) :
- **chiffrement avec la clé publique** (tout le monde peut le faire) 
- **déchiffrement avec la clé privée**

Les clés publiques sont présentes dans un annuaire de clés publiques.

Comment peut-on être sûr que partant de la clé publique on ne peut pas récupérer la clé privée ?
En théorie, il y a une relation mathématique entre les clés et on peut deviner la clé privée à partir de la clé publique. La solidité du système repose sur la **sécurité calculatoire**, elle-même assurée par la difficulté du problème mathématiques sous-jacent.



# 2. Problèmes mathématiques et schémas

## 2.1. Factorisation des entiers et RSA
Problème mathématique : connaissant `n = p*q` produit de 2 **nombres premiers**, trouver les valeurs de p et q.

Il est facile de calculer n en connaissant p et q mais il est difficile de retrouver p et q ne connaissant que n (complexité sous-exponentielle en log n, soit `>> O(polynome(log n))`).

Le record de factorisation est un nombre de 240 chiffres (795 bits) en utilisant 6000 machines pendant 1 an.
→ recommandation de l’ANSSI : utiliser p et q de taille 1536 pour obtenir *n de taille 3072 bits*.

### 2.1.1 Chiffrement RSA
- Génération de 2 nombres premiers **p** et **q** (cela prends quelques heures)
- Calcul `n = p*q`
- Choix d’un entier **e** premier avec `phi(n) = (p-1)*(q-1)`
- Calcul de **d** inverse de e modulo phi(n), i.e. `d*e = 1 (mod phi(n))`
→ clé publique **(n, e)** ; clé privée **(p, q, d)**

p : texte clair, interprété comme un entier entre 0 et n
chiffrement : **c = p^e (mod n)**
déchiffrement : **p = c^d (mod n)**

### 2.2. Sécurité du chiffrement RSA :
Retrouver d est équivalent en difficulté que de retrouver la factorisation `n = p*q` ou que de connaître `phi(n)` ou l'exposant `d`.

Problèmes :
- La primitive RSA est déterministe, par conséquent distinguable gauche droite (définition dans le cours [[(Crypto) 2 - Chiffrement symétrique#4.2. Indistinguabilité "left-right"]])
- `(chiffré de p1*p2) = (chiffré de p1) * (chiffré de p2)`
=> solution : appliquer l’OAEP ou le PKCS11 en plus du RSA.


## 2.2. Formatage OAEP
Optimal Asymetric Encryption Padding : 
1. Ajout à la suite du clair 128 bits à 0 puis 256 bits aléatoires, 
2. Puis mélange de tous les bits obtenus par des opérations linéaires
3. Puis chiffrement RSA

Cette méthode randomise le chiffré produit par l'algorithme RSA, ce qui assure la non distinguabilité gauche droite. Le RSA avec OAEP est donc **IND-CCA**.
Elle permet également de vérifier que le chiffré reçu a bien été calculé avec cette méthode (il y a bien un champ de 128 bits à 0 puis 256 bits aléatoires à la fin du message).


## 2.3. Logarithme discret et Elgamal
Notation : `F(p) = Z/(pZ) = ensemble des entiers modulo p`

### 2.3.1. Problème du logarithme discret
Soit G groupe cyclique (exemple de groupe cyclique : racine n-ième de l’unité dans les complexes), g un générateur de G et y dans G. Trouver l'entier naturel L tel que **y = g^L**.

Remarques :
- si L convient alors tout `L' = L [mod card(G)]` convient aussi et il suffit de chercher L dans {0, ..., card(G)-1}
- on note `L = log(y)`


### 2.3.2. Problème du logarithme discret dans F*(p)
On note l'ensemble des entiers modulaires à p`F(p) = Z/pZ = {0, ..., p-1}` et cet ensemble privé de l'élément nul `F*(p) = {1, ..., p-1}`.

Soit p un nombre premier, g un générateur de `F*(p) = {1, ..., p-1}`, h dans `F*(p)`, trouver l'entier L tel que **h = g^L mod p**.
L est dans {0, ..., p-2}

ex : p = 19 ; g = 2 générateur de F*(19), on a F*(19) = {1, …., 18} = {2⁰, …, 2¹⁷} (mod 19)
Par exemple, h = 12, retrouver L...

Calculer h = g^L (mod p) est rapide mais retrouver L à partir de h, g et p est difficile !

Recommandations pour choisir p :
- p premier d’au moins *3072 bits*
- (p-1) a au moins un facteur premier de 256 bits

### 2.3.3. Chiffrement Elgamal dans F*(p)
- Génération d'un nombre premier **p**
- Choix d'un générateur **g** du groupe F*(p)
- Choix d'un entier **L** dans {1, ..., p-2}
- Calcul `h = g^L (mod p)`
-> clé publique **(p, g, h)**, clé privée **L** 

<u>Chiffrement</u> :
1. Choisir le secret k aléatoire dans {1, ..., p-2}
2. Soit m le message à chiffrer
3. `chiffré = (c1, c2) = (g^k mod p, m*h^k mod p)`
La variable auxiliaire k est envoyé au destinataire de façon détourné : on envoie g^k (mod p) mais pas k.

<u>Déchiffrement</u> :
`m = c2 * (c1^L)^(-1) mod p


## 2.4. Logarithme discret sur courbe elliptique

### 2.4.1. Courbes elliptiques
Une courbe elliptique est l’ensemble :
`E(F(p)) = {0(E)} U {(X,Y) dans F(p)² tel que Y² = X³ + aX + b}`, avec (a,b) dans F(p) et p nombre premier.

Construction d’une loi de composition interne :
Soit (P, Q) dans F(p)², on construit l’élément P+Q comme suit :
- On trace la droite passant par P et Q
- On note P’ le point d’intersection entre cette droite et la courbe elliptique
- P+Q est le symétrique de P’ par l’axe des abscisses

Exemple de calcul de P+Q d'une courbe elliptique dans E(R).
![[courbe-elliptique.png]]
<u>Remarques</u> : 
- E(F(p)) est un ensemble fini et card(E(F(p))) != p
- Si card(E(F(p))) est premier alors E(F(p)) est un groupe cyclique

On note l’exponentielle d’un point `LP = [L]P = P + … + P` (L fois).


### 2.4.2. Problème du logarithme discret sur courbe elliptique
Étant donné p nombre premier != 2 et 3, E(F(p)) une courbe elliptique, P et Q ∈ E(F(p)), trouver L ∈ N (s’il existe) satisfaisant `Q = [L]P`.

Il est facile de calculer `Q = [L]P`, mais il est difficile, connaissant P et Q, de retrouver L.


### 2.4.3. Chiffrement Elgamal sur courbe elliptique
- Génération d'un nombre premier **p** != 2 et 3
- Choix d'une courbe elliptique **E(F(p))** de cardinal premier, donc cyclique
- Choix de **P** dans E(F(p))  (c'est automatiquement un générateur)
- Choix d'un entier **L** dans {1, ..., card(E(F(p))-1}
- Calcul `Q = [L]P`
-> clé publique **(p, E(F(p)), card(E(F(p))), P, Q)**, clé privée **L** 

<u>Chiffrement</u> :
1. Choisir le secret k aléatoire dans {1, ..., p-2}
2. Soit m le message à chiffrer
3. `chiffré = (c1, c2) = ([k]P, m + [k]Q)`
La variable auxiliaire k est envoyé au destinataire de façon détourné : on envoie `[k]P` mais pas k.

<u>Déchiffrement</u> :
`m = c2 - [L]c1


### 2.4.4. Sécurité du chiffrement sur courbe elliptique
Le problème du log discret sur courbe elliptique (ECDL) ne possèdent pas de meilleure attaque que les attaques génériques (brute force en O(p) et BSGS en O(sqrt(p-1))), tandis que le log discret sur F(p) (DL sur F(p)) possède de meilleures attaques. Par conséquent, on peut utiliser des **clés plus petites** pour assurer le même niveau de sécurité avec les courbes elliptiques.
![[comparaison_log_discret.png]]



# 3. Sécurité d’un chiffrement asymétrique

Le **niveau de sécurité** n’est pas la taille de la clé ! C’est une fonction croissante de la taille de la clé mais il est **très inférieur à 2^(taille de la clé)**. L'attaque générique résout le problème mathématique sous jacent et non la recherche exhaustive de la clé.

De plus, tout choix de clé n’est pas bon...

Le principe d’indistinguabilité est le même que pour le chiffrement symétrique, cf [[(Crypto) 2 - Chiffrement symétrique#4.2. Indistinguabilité "left-right"]].

  

# 4. Autour du chiffrement asymétrique

## 4.1. Chiffrement hybride
**Performances** du chiffrement asymétrique :
- Permet de ne pas partager de secret au préalable
- Permet d’avoir moins de clés en cas de nombreux utilisateurs
- MAIS est **100 à 1000 fois plus lent** que son équivalent symétrique
=> Chiffrement asymétrique uniquement pour de petits messages.
=> Partager un secret chiffrer asymétriquement pour continuer en chiffrement symétrique.


## 4.2. Certification de clés publiques
Rappel : on utilise la clé publique du destinataire pour chiffrer un message qu’on lui envoie.
Si la clé publique n’est pas certifié, comment être sûr qu’elle appartient bien au destinataire ?
→ attaque de type Man In The Middle possible !

Pour éviter ce type d'attaque, il faut  **assurer l’intégrité des clés publiques**. Cela peut être fait au moyen de signature (cf [[(Crypto) 5 - Signature]]) mais cela impose tout de même une infrastructure de gestion de clés avec, notamment, un autorité de certification.


## 4.3. Protocole de Diffie-Hellman
Diffie et Hellman proposent un protocole d’échange de clé, aussi appelé négociation de clé, qui permet de s’échanger un secret commun sans se rencontrer.

Ce protocole est une bonne alternative au chiffrement hybride.


Soit G groupe cyclique tel que le logarithme discret est difficile et g générateur de G.
g est public
Alice choisit a dans {1, …, card(G)–1}, calcule `g^a` et l’envoie à Bob
Bob choisit b dans {1, …, card(G)–1}, calcule `g^b` et l’envoie à Alice
→ Alice et Bob sont capables de calculer le secret commun : `g^ab`

Peut on déterminer g^ab sachant g^a et g^b ?
La difficulté vient du problème du logarithme discret dans G.
On pourra utiliser le problème des courbes elliptiques en modifiant l’algorithme.


Une attaque par le milieu est possible si le canal n’est pas intègre et que l’attaquant peut modifier ce qui passe sur le canal. Une parade est d’authentifié les émetteurs (cf [[(Crypto) 5 - Signature]]).

Ce protocole nécessite également que les deux acteurs soient connectés en même temps pour s’échanger la clé.