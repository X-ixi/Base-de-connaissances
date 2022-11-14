```toc

```

# 1. Principe

Analogue d’une signature papier : on s’attend alors aux 3 services suivants :
- **Authentification** du signataire
- **Intégrité** des données (personne ne modifie le document après la signature)
- **Non-répudiation** du signataire (le signataire ne peut pas argumenter que ce n’est pas sa signature et une tierce partie l’obligera à honorer sa signature – police, ...)
=> la signature n’assure pas de confidentialité

Une seule personne peut signer et tout le monde peut vérifier la signature.
C'est l'inverse du chiffrement asymétrique où tout le monde peut chiffrer mais une seule personne peut déchiffrer (cf [[(Crypto) 3 - Chiffrement asymétrique]]).
→ **Cryptographie asymétrique** : signature avec la clé privée et vérification avec la clé publique


Comme pour le MAC (cf [[(Crypto) 4 - Intégrité symétrique et hachage]]), Alice envoie le couple (message, signature) à Bob. MAIS contrairement à l’intégrité symétrique, Bob ne recalcule pas la signature d’Alice (il ne peut pas le faire !), et il la vérifie avec la clé publique d’Alice. Il est alors assuré des 3 propriétés d'authentification, d'intégrité et de non-répudiation.

<u>Remarques</u> :
- La sécurité des signatures reposent sur la difficulté des primitives mathématiques sous-jacentes : factorisation RSA, logarithme discret d’Elgamal, ...
- La signature doit être **de taille fixe** même si le message à signer est de taille variable
  -> On commence par **haché le message** avant de calculer la **signature sur le hash**.
- Dans le cas de l'utilisation d'un chiffrement asymétrique du message ET d'une signature, les paramètres de la signature doivent être différents de ceux utilisés pour le chiffrement du message. Dans ce cas, on signe le clair, chiffre le message et signe le chiffré.



# 2. Exemples

Dans la suite, H est une **fonction de hachage publique et commune** : le destinataire déchiffre la signature avec la clé publique, puis hache le message avec H et compare les deux.

## 2.1. Signature RSA
Rappel du chiffrement RSA (cf [[(Crypto) 3 - Chiffrement asymétrique#2.1.1 Chiffrement RSA]]) :
c = m^e (mod n)
m = c^d (mod n)
e est la clé publique et **d la clé privée**

Signature RSA :
**sig = H(m)^d (mod n)**
**H(m) = sig^e (mod n)**


## 2.2. Signature et logarithme discret

### 2.2.1. Signature Elgamal
Cette signature se base sur le chiffrement Elgamal, cf [[(Crypto) 3 - Chiffrement asymétrique#2.3.3. Chiffrement Elgamal dans F*(p)]].
La formule exacte est compliquée et n'apporte pas grand chose.
C’est une **signature randomisée** (comme le chiffrement Elgamal).

### 2.2.2. Signature DSA
Légère variante de Elgamal

### 2.2.3 Signature ECDSA
Analogue mais avec un groupe cyclique d’une courbe elliptique.
cf [[(Crypto) 3 - Chiffrement asymétrique#2.4. Logarithme discret sur courbe elliptique]]



# 3. Sécurité des signatures

Idem modèle de sécurité des MAC, cf [[(Crypto) 4 - Intégrité symétrique et hachage#2. Sécurité d’un MAC]] :
- Contextes : attaque à messages connues ou attaque à messages choisis.
- Défis : contrefaçon universelle, sélective ou existentielle.

Un algorithme de signature est considéré comme sûr s'il n'existe aucune attaque meilleure que les attaques génériques, dont la complexité dépend du défi et est :
 - **Inférieur à 2^(taille de la clé)** pour recouvrer la clé privée (résolution du problème mathématique sous-jacent)
   -> contrefaçon universelle 
- **2^(taille de la signature)** pour recouvrer la signature
  -> contrefaçon sélective
- **2^(taille du hash)** pour trouver une seconde pré-image à la fonction de hachage
  -> contrefaçon sélective
- **2^(taille du hash / 2)** pour trouver une collision de la fonction de hachage
  -> contrefaçon existentielle voire sélective

Ainsi, l'attaquant peut attaquer la fonction de hachage directement plutôt que d'essayer de casser la signature. Par conséquent, il faut que la fonction de hachage soit aussi sûr que la fonction de signature.

Ex :  m      - H →      H(m)      - RSA →      H(m)^d (mod n)
Pour assurer un niveau de sécurité de 2¹²⁸, on choisit une taille de hash de 256 bits et une taille de clé pour le RSA de 3072 bits.


## 3.1. Exemple d'attaque sur la fonction de hachage 
L’attaquant s’arrange pour que 2 messages donnent le même hash : un message facile à faire signer par le signataire et le message cible sur lequel on aimerait obtenir une signature.

Comme les hash sont les mêmes, les signatures le seront aussi ! On fait signer le message facile à faire signer et on utilise la signature sur le message cible.

Hypothèse de cette attaque : réussir à trouver une collision entre deux messages qui ont du sens, ce qui paraît très difficile mais il y a une astuce :
- Choix du message cible
- Création d’une collection de 2^(h/2) messages à partir du message cible : on choisit h/2 emplacements dans le message cible et on y ajoute (ou non) des caractères non imprimables
- Choix du message facile à faire signer
- Création d’une collection de 2^(h/2) messages  à partir de ce message
=> Statistiquement un élément de la collection cible a le même hash qu’un élément de l’autre collection

Si h=256 alors la collision est impossible à trouver en pratique.