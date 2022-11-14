```toc

```

# 1. Intégrité symétrique, MAC

Vieille solution naïve : le destinataire s’aperçoit que le message a été modifié car il est sémantiquement incorrect (il a été modifié à l’aveugle). Cela s’applique uniquement au message avec une structure claire (texte en français, …) et aux attaquants incapable de déchiffrer le message.

Meilleure solution : utiliser un **Message Authentication Code** (MAC)

Un MAC est l'analogue d'un CRC dans le domaine de la détection d'erreur : tag calculé à partir du message, envoyé avec le message et qui permet au destinataire de vérifier l’intégrité du message. Cependant, le CRC est utilisé contre les perturbations accidentelles et aléatoires du canal, pas contre un attaquant intelligent.
→ Pour éviter qu’un attaquant puisse modifier le tag en concordance avec le message modifié, il faut que le tag soit **calculé à partir de la clé secrète**, c’est ce qu’on appelle un MAC.

Soit M message de longueur quelconque et K la clé secrète, le tag de **longueur fixe** T est calculé comme suit : `T = MAC(M, K)`.

Transmission intègre :
1. Alice **envoie (M, T)** à Bob
2. Bob calcule T' = MAC(M, K)
	1. Si T' = T alors le message est intègre (très très probablement)
	2. Sinon il ne l'est pas



# 2. Sécurité d’un MAC

L’attaquant connaissant des messages M et leur MAC T, peut-il contrefaire un MAC pour un nouveau message ?

Plusieurs contextes possibles : 
- **Messages connus** (KMA - Known Message Attack): nombre limité et non choisi de messages et de MAC
- **Message choisis** (CMA - Chosen Message Attack) : accès à un oracle de MAC

Plusieurs défi possibles : 
- **Contrefaçon existentielle** : trouver un couple (M,T)
- **Contrefaçon sélective** : trouver T pour M imposé
- **Contrefaçon universelle** : trouver T pour tout M

Un algorithme de MAC est sûr s’il n’existe aucune attaque meilleure que la brute force, qui à une complexité :
- 2^(taille de K) pour la contrefaçon universelle (revient à retrouver la clé)
- 2^(taille de T) pour la contrefaçon sélective ou existentielle



# 3. Fonction de hachage

## 3.1. Définition et sécurité
Une fonction de hachage est une fonction qui prend en entrée un variable de taille quelconque et renvoie une **empreinte** ou **haché** (hash en anglais) **de taille fixe** avec les propriétés suivantes :
- Rapide à calculer
- A **sens unique** (i.e. très difficile à inverser) = propriété de **pré-image** résistance : étant donné y, il est difficile de trouver x tel que y = H(x)
- Propriété de **seconde pré-image** résistance : étant donné x0, il est difficile de trouver x != x0 tel que H(x) = H(x0)
- Résistante à la **collision** : difficile de trouver x1!=x2 tel que H(x1)=H(x2)

Les complexité des attaques génériques sont (h = taille du hash) :
- Pré-image : O(2^h)
- Seconde pré-image : O(2^h)
- **Collision** : O(**sqrt(2^h)**) => paradoxe des anniversaire
=> h = 256 pour assurer une complexité d’attaque générique de 2¹²⁸.

Analogie avec les dates d’anniversaire : il est facile de connaître la date d’anniversaire d’une personne mais à partir d’une date d’anniversaire, il est difficile de trouver une personne dont c’est la date d’anniversaire. Cependant, le paradoxe des anniversaires fait qu'il n'est pas rare de trouver 2 personnes nés le même jour !

## 3.2. Construction de Merkle-Damgard
Itération d'une primitive f appelée fonction de compression. Soit b constante définie à l'avance et h la taille du hash souhaitée :
1. Ajout de padding sur M pour que sa longueur soit multiple de b
2. Découpage de M en blocs de b bits : `M[1], ..., M[L]`
3. Soit H0 constante initiale
4. Pour i de 1 à L, calcul de `H[i] = f(M[i], H[i-1])`
5. Renvoie de `H[L]`

Cette technique est notamment utilisée par les algorithmes MD4, MD5, SHA-0, SHA-1 et toutes les variantes de SHA-2 (256 bits, 512 bits, ...).

Toutes les variantes de l'algorithme SHA-3 utilisent une autre méthode itérative appelée construction éponge.



# 4. Construction de MAC

## 4.1. Avec une fonction de hachage
Soit H une fonction de hachage de taille de bloc b et de taille de hash h, soit K une clé secrète tel que h < b et len(K) < b.

Toutes les méthodes incluent un pré-traitement qui consiste à :
- Padder la clé K en Kpad de longueur b
- Padder le message M en Mpad de longueur un multiple de b

La méthode **HMAC** est le standard depuis des années et consiste à :
1. Calculer `y = H(Kpad ⊕ ipad || Mpad )`  (|| signifie concaténer)
2. Padder y en ypad de longueur b
3. Calculer `T = H(Kpad ⊕ opad || y)
4. Renvoyer T
ipad = 0x3636....36 et opad = 0x5c5c...5c (chacun de taille b bits)

Une autre bonne méthode est la méthode de l'enveloppe et consiste à calculer :
`T = H(Kpad || Mpad || Kpad)`


## 4.2. Avec une primitive bloc
Principe : utiliser un mode opératoire chaîné d'une primitive bloc (cf [[(Crypto) 2 - Chiffrement symétrique#3. Chiffrement par bloc]]) pour créer le hash. 

Mode CBCMAC : mode ressemblant fortement au CBC de l’AES.
La construction de MAC avec une primitive bloc est faisable mais moins adapté qu’avec une fonction de hachage.



# 5. Intégrité et confidentialité

Bien souvent, on a besoin d’assurer l’intégrité ET la confidentialité.
Il faut alors choisir **deux clés secrètes différentes** pour le chiffrement symétrique et l’algorithme de MAC !

On peut choisir d’appliquer le chiffrement, noté SymEnc, avant ou après le MAC et d'appliquer le MAC sur le texte clair ou sur le chiffré. Ces choix modifient les performances d'intégrité et de confidentialité :
- **Encrypt-then-MAC** : Envoyer C = SymEnc(M) et T = MAC(C)
	→ meilleure solution
	-> renforce également la confidentialité : **IND-CCA** si SymEnc est IND-CPA, cf [[(Crypto) 2 - Chiffrement symétrique#4.2. Indistinguabilité "left-right"]]
	
- **MAC-then-Encrypt** : Envoyer C = SymEnc(M, MAC(M))
	-> en général moins bon que Encrypt-then-MAC. 
	
- **Encrypt-and-MAC** : Envoyer C = SymEnc(M) et T = MAC(M)
	→ peut ne pas assurer la confidentialité car le MAC n’assure pas la confidentialité (ne pas utiliser ce mode combiné)
