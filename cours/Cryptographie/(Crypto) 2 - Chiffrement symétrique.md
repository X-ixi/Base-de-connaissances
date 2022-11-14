```toc

```

Hypothèse : Alice et Bob se sont rencontrés pour s’échanger une clé secrète et en conserve chacun une copie.

Le chiffrement symétrique permet la **confidentialité**.


# 1. Techniques anciennes

## 1.1. Chiffrement par substitution

### 1.1.1. Substitution mono-alphabétique
**Chiffrement de César** : décalage des lettres de l’alphabet)
26 clés possibles donc la sécurité du chiffrement de César reposait surtout sur le secret du procédé.

**Permutation quelconque** de l’alphabet
26! = 2⁸⁸ clés possibles donc attaques par brute force impossible mais les **attaques fréquentielles** sont faciles à mener.

### 1.1.2. Substitution poly-alphabétique
**Chiffrement de Vigenère** : addition du clair à un mot clé répété
Correspond à n chiffrements de César entrelacés (n étant la longueur de la clé). Les attaques fréquentielles sont toujours possible.

Pour éviter les attaques fréquentielles, il faudrait une clé aléatoire et de même longueur que le message, car la combinaison d’une chaîne de caractères qui a du sens avec une chaîne de caractères aléatoires est une chaîne de caractères aléatoires.
  

## 1.2. Chiffrement par transposition
On écrit le message en ligne et on le lit en colonne. La longueur de la clé définit le nombre de colonnes. De plus, on lit les colonnes dans l’ordre des numéros de la clés.
→ les symboles du texte clair sont réarrangés mais pas remplacés

Ce chiffrement est plus dur à attaquer que par le chiffrement par substitution mais c'est toujours possible.



# 2. Chiffrement par flot

## 2.1. Approche de Shannon
**Chiffrement parfait** : le texte chiffré ne donne aucune information sur le texte clair. La distribution de probabilités du clair sachant le chiffré est égale à la distribution a priori.

**Chiffrement de Vernam** (On Time Pad, masque jetable) : la clé est aléatoire et de même longueur que le clair et elle est additionné au clair pour obtenir le chiffré (ici on parle de clé et d’addition binaire mais cela fonctionne aussi avec des caractères).

⇒ En pratique c'est difficile à mettre en place car la clé doit être aussi longue que le message et n'être utilisé qu’une seule fois.

Anecdote : cette technique a été utilisée pour le téléphone rouge pendant la guerre froide. Ils s’accordaient ponctuellement sur une clé très longue et l’utilisaient petit à petit par morceau par la suite.


## 2.2. Générateur de pseudo aléa (GPA)
Un GPA produit des suites **déterministes** mais **indiscernables de suites aléatoires** à partir d’un **germe** ('seed' en anglais) de taille faible.

→ Alice et Bob n’ont plus qu’à s’échanger le germe qu’ils vont ensuite utiliser pour générer une **suite chiffrante** de même taille que le clair (on se ramène au chiffrement de Vernam).

C’est un **chiffrement par flot** car les opérations de chiffrement et de déchiffrement se font un bit à la fois et ne nécessite pas la connaissance de la suite du message.


**/!\\** La suite chiffrante ne doit être utilisé qu’une seule fois, par conséquent, il faut ajouter un **marquant** aléatoire spécifique au message, qui sera utilisé pour initialisé le GPA.
On a finalement : `suite chiffrante Z = GPA(clé K, marquant IV)`.
Le marquant est envoyé avec le message chiffré pour que le destinataire qui possède la clé puisse initialisé correctement le GPA.

Conséquence intéressante : si on chiffre plusieurs fois le même texte clair, on obtient des chiffrés différents (on parle d’algorithme randomisé).


## 2.3 Registres linéaires et fonction booléenne

### 2.3.1. LSFR
Prenons un exemple de GPA : le LSFR (Linear Feedback Shift Register)
![[LSFR.png]]
Soit `Xt = (x(0,t), x(1,t), ..., x(L-1,t))` l'état interne du LSFR.

A chaque étape, on calcule la combinaison linéaire des x0 à x(L-1) avec les coefficient a0 à a(L-1) puis on décale tous les éléments vers la gauche et on place le résultat de la CL dans la case x(L-1). On obtient alors le nouvel état interne `X(t+1) = (x(1,t), ..., x(L-1,t), CL)`.
Les coefficients a0 à a(L-1) sont fournis par le constructeur du LSFR (et souvent connus) et l’utilisateur fournit les x0 à x(L-1).

On définit le polynôme de rétroaction : `P(T) = a0 + a1*T + ... + a(L-1)*T^(L-1) + T^L`. 
Il regroupe formellement les coefficients a0, …., a(L-1), mais n’a pas vocation à être utilisé pour calculer quoique ce soit.

Les suites engendrées satisfont une récurrence linéaire, sont de période inférieure ou égale à 2^(L)-1 et surtout ont de bonnes propriétés statistiques.
→ on peut choisir a0, …, a(L-1) pour que la période soit égale à 2^L -1 et que les suites **ressemblent à des suites aléatoires**, si on les observe sur une taille très inférieur à la période.

L’état interne x0, …, x(L-1) est initialisé avec la clé et le marquant, et on utilise un LSFR pour générer une suite aléatoire dont on utilisera une partie très inférieure à la période.

### 2.3.2. Fonction booléenne
Une fonction booléenne transforme un vecteur de bits en un autre vecteur de bits. Elle correspond à un polynôme de degré d. Par exemple : `F([x1, x2, x3]) = x1*x2 + x1*x3`.

Le LSFR seul, de part sa linéarité, n'est pas assez sûr (voir section suivante). Pour palier à ça, on ajoute une fonction booléenne à la sortie du LSFR. On peut aussi utiliser plusieurs LSFR en entrée de la fonction booléenne. 


## 2.4. Sécurité d’un GPA
Si je connais un clair et un chiffré alors je connais la suite chiffrante utilisée car `chiffré = clair ⊕ suite chiffrante`. Le contexte "clair connu" ou "clair choisi" est équivalent ici à "suite chiffrante connu".
Est-ce possible de **retrouver la clé** à partir de la connaissance de la suite chiffrante ?

Notons L la longueur de la clé :
- La brute force a une complexité de 2^L
- Une **attaque algébrique** est possible : la suite chiffrante z0, …z(L-1) s’exprime de façon linéaire en fonction des k0, … k(L-1) ; il suffit alors de résoudre un système d’équation à L inconnus.
  → complexité L³

Si L=128 (taille normale d’une clé) :
- Brute force : 2¹²⁸
- Attaque algébrique : 128³ = 2²¹ → facile à casser

C’est pourquoi on ajoute une **fonction booléenne** non linéaire à la sortie du LSFR. On peut également utiliser plusieurs LSFR en entrée de la fonction booléenne.
→ la complexité de l’attaque algébrique devient L^(3d), avec L=128 et d=4, on obtient 2⁸⁴ !


Autres défis portant sur la faiblesse du caractère aléatoire :
- Attaque par **prolongement** : connaissant une séquence (IV, z0, …, z(N-1)), deviner le prochain bit zN
- Attaque en **distingueur** : étant donné une suite (z0, …, zN), deviner si elle est parfaitement aléatoire ou issue du GPA
→ dans les deux cas, la probabilité de succès ne doit pas être significativement éloigné de 1/2



# 3. Chiffrement par bloc
Contrairement au chiffrement par flot, il faut attendre d’avoir le contenu de l’intégralité d’un bloc avant de pouvoir le chiffrer/déchiffrer. Un algorithme de chiffrement par bloc s'appuie sur une primitive bloc et un mode d'opération.
La taille standard d’un bloc, notée b, est de 128 bits.

## 3.1. Primitive bloc
Une primitive bloc est une famille de permutation paramétré par la clé.

Le processus est itératif avec l’application à plusieurs reprise d’une fonction de tour. Cette fonction de tour prend en paramètre une sous-clé dérivée de la clé initiale. Ces sous-clés sont différentes à chaque tour.

Le nombre de tours dépend de la taille de la clé (10 tour pour une clé de 128 bits, 12 pour une de 192 et 14 pour une de 256 - valeurs choisies par les auteurs de l’AES).

## 3.2. Ingrédients d’une primitive bloc - AES
L’**AES** (Advanced Encryption Standard) est un chiffrement par bloc. L’AES est issu d’une compétition ouverte en 1998-2000 avec comme cahier des charges d’être résistant aux attaques connues.

**AES-128**
Taille de bloc = taille de clé = 128 bits
Un bloc est sous la forme d’un tableau de 4x4 octets (i.e. 128 bits).

A chaque tour, les opérations suivantes sont réalisées dans cet ordre :
- **Subbyte** : substitution octet par octet selon un tableau connu (non linéaire)
- **ShiftRow** : rotations circulaires - décalage de 0 colonne sur ligne 1, de 1 sur ligne 2, de 2 sur ligne 3 et de 3 sur ligne 4 (linéaire)
- **MixColumn** : transformation des colonnes par produit matrice-vecteur avec une matrice connue (linéaire)
- **AddRountKey** : ajout des 16 octets de sous-clés du tour

Opérations possibles sur des octets :
- Addition : addition modulo 2 (1+1=0)
- Multiplication : multiplication des polynômes équivalents aux octets (01001000 = X⁶ + X³) puis modulo ce polynôme : *X⁸+X⁴+X³+X+1*
→ on peut alors faire du calcul matriciel, … (spécification détaillée dans  le document « fips 197 » : https://nvlpubs.nist.gov/nistpubs/fips/nist.fips.197.pdf)

Voici l'enchaînement des tours :
- Un tour initial constitué uniquement de l’addition de la clé
- 9 tours avec les 4 opérations présentées précdemment avec 9 sous-clés
- Un tour final avec substitution, rotation circulaire et addition de la dernière sous-clé

Animation sur youtube : taper « aes animation zabala »
  

## 3.3. Sécurité d’une primitive bloc
Est-il difficile de trouver la clé à clair choisi ?
On voudrait que ce soit aussi difficile qu’une attaque exhaustive sur la clé.

2 attaques statistiques du chiffrement par bloc :
- **Attaque différentielle** : on calcule la différence entre deux clairs et la différence entre les chiffrés correspondants. Pour une même différence entre clairs, on obtient des différences entre chiffrés différents. Si la répartition des différences entre chiffrés n’est pas uniformément réparti, on peut exploiter ce biais statistique pour retrouver la clé du dernier tour et finalement la clé.
- **Attaque linéaire** : attaque se basant sur une corrélation linaire entre certains bits de clair, de clé et de chiffré.


<u>Définition de sécurité d’une primitive bloc</u> : la primitive est indistinguable d'une **permutation aléatoire**. 
Cela implique les propriétés suivantes :
- Les images des éléments sont indépendantes les unes des autres
- Les informations connues sur des couples (clairs, chiffrés) avec des clés différentes n’apportent pas d’information
- La primitive assure **diffusion** (chaque bit d’entrée influence tous les bits de sortie) et **confusion** (chaque bit de sortie est une fonction non linéaire des bits d’entrées).

La diffusion n’est complète qu’au bout de 4 tours de l’algorithme. Il y a 10 tours pour contrer les attaques différentielles.


## 3.4. Mode d’opération
Un mode d'opération est une sur-couche algorithmique faisant appel à la primitive bloc comme sous-routine.

Pour traiter des messages de tailles quelconques, on découpe le clair en blocs et on utilise la primitive bloc pour traiter chacun des blocs. 
Selon les modes, il est nécessaire que tous les blocs soient complets. Dans ce cas, on utilise du padding sur le dernier bloc. Ce **padding** doit être non-ambigu (i.e. bijectif) et doit être appliqué à tous les blocs. 
Exemple : écrire un « 1 » puis compléter éventuellement avec des « 0 » jusqu’au prochain multiple de la taille du bloc. Il faut le faire même si le dernier bloc est complet.

### 3.4.1. Electronic Code Book - ECB
Le mode opératoire le plus simple est le **mode dictionnaire** **ECB** pendant lequel on applique l’algorithme à chaque bloc. Ce mode nécessite des blocs complets.
![[pingouin_ecb_cbc.png]]

### 3.4.2. Cipher Bloc Chaining - CBC
![[cbc.png]]
Le **Cipher Bloc Chaining** est un autre mode opératoire dans lequel une variable auxiliaire IV choisie aléatoirement est transmise comme premier bloc. Ensuite chaque bloc de clair est additionné avec le bloc de chiffré précédent avant d’y appliquer la primitive.

Ce mode nécessite de chiffrer les blocs dans l’ordre et n’est donc pas parallélisable.

### 3.4.3 Counter - CTR
![[ctr.png]]
Le **mode compteur** **CTR** n’applique pas l’algorithme aux blocs de clair directement. Une variable auxiliaire IV est incrémenté de 1 puis est chiffré par l’algorithme puis additionné au premier bloc de clair. Cette variable IV est incrémenté puis chiffré puis additionné au deuxième bloc de clair, etc. La variable IV est transmise en clair comme premier bloc. Ce mode opératoire ne nécessite pas que les blocs soient complets. Le déchiffrement ne nécessite pas d’inverser l’étape de chiffrement.

→ mode **randomisé**, **parallélisable** et ne nécessitant **pas d’inverser l’AES**

*Note* : d'autres modes d'opération existe, comme le Cipher Feed Back ou le Output Feed Back.



# 4. Sécurité d’un chiffrement symétrique

## 4.1 Rappel : niveau de sécurité et sécurité sémantique
Définitions dans le cours [[(Crypto) 1 - Introduction#2. Attaquant, modèle de sécurité]].

Niveau de sécurité d’un algorithme = complexité de la meilleure attaque sur l’algorithme
→ on veut que la meilleure attaque ne soit pas meilleure que l’attaque exhaustive

Sécurité sémantique : ne pas pouvoir retrouver la clé à clair connu, à chiffré connu, ne pas pouvoir deviner d’information partielle, …
→ on regroupe tous ces objectifs en un seul : **l’indistinguabilité gauche-droite**


## 4.2. Indistinguabilité "left-right"
Scénario : l’algorithme à tester est connu, il est instancié avec une clé inconnue et placer dans une boite. Cette boite reçoit 2 textes clairs, en choisit un aléatoirement et renvoi le chiffré du clair choisi.
→ *Peut-on deviner quel est le clair qui a été choisi ?*

Remarques : le joueur peut choisir autant d’entrées qu’il le souhaite.

### 4.2.1. Avantage de l'attaquant
On définit l'**avantage** de l'attaquant comme suit :
```
Adv(A)  = 2*(P(A devine bien) - 1/2)
		= P(A choisit right | valeur=right) - P(A choisit right | valeur=left)
```
L'avantage mesure l'efficacité de l'attaquant et s'exprime en fonction des paramètres de l'algorithme et du nombre de requêtes faites par l'attaquant.

Si `Adv(A) <= 2⁻²⁰`, i.e. `P(A devine bien) <= 1/2 + 1/2⁻²¹`, alors le schéma de chiffrement est dit IND-CPA sûr.

**IND-CPA** : Indistinguishability under Chosen Plaintext Attack (scénario précédent).
**IND-CCA** : Indistinguishability under Chosen Cipher Attack (scénario plus avantageux pour l'attaquant qui a en plus accès à un oracle de déchiffrement).
IND-CCA => IND-CPA


### 4.2.2 Exemples
Un chiffrement déterministe (ECB par exemple) ne passera pas ce test : premier choix de clair (P1, P1) puis deuxième choix (P1, P2), dans le deuxième choix on sera en mesure de distinguer la sortie.

Les modes opératoires randomisés (CTR, CBC, …) ont un avantage qui dépend du nombre de requêtes q et de la taille du bloc b : `O(q²/2^b)`.
→ tant que q < 2^(b/2 - 10), l’algorithme est IND-CPA sûr.

Il ne faut pas utiliser ces algorithmes pour chiffrer plus de 2⁵⁴ blocs de 128 bits avec la même clé.

Aucun de ces algorithmes n'est IND-CCA sûr.