```toc

```

# 1. Introduction
**Cryptologie** : technique algorithmique et mathématiques permettant d’assurer la **confidentialité**, l’**intégrité** et l’**authentification**.
→ n’assure pas la disponibilité (un message peut-être détruit)

Cryptologie = cryptographie (construit les outils) + cryptanalyse (met à l’épreuves les outils)

**Algorithme cryptographique** : fonction paramétrée par des clés secrètes, composé de primitives. Une **primitive** a des propriétés spécifiques mais ne permet d'assurer la fonction cryptographique à elle seule : il faut en assembler plusieurs correctement pour ça.

La spécification d'un algorithme passe par une description algébrique avec des équations polynomiales binaires : **chiffré C = Fonction(clair P, clé K)**, avec :
- C : ciffer text
- P : plain text
- K : key

En cryptologie, on utilise des **math discrètes** et on manipule uniquement des 0 et des 1 (voire des entiers)
⊕ : addition binaire (1+1=0)

La robustesse d’un algorithme cryptographique ne doit pas reposer sur la confidentialité de son fonctionnement.
→ **spécification des algorithme très souvent publique**, voire standard (sauf militaire ou similaire)


Cryptographie symétrique = à clé secrète
→ nécessite que Alice et Bob se rencontrent avant de communiquer (« pre-shared secret »)

Cryptographie asymétrique = à clé publique
→ clé de chiffrement et de déchiffrement différente

  

# 2. Attaquant, modèle de sécurité

## 2.1. Approche intuitive
Un schéma est **sûr** lorsqu’un attaquant est incapable de deviner tout type d’information qu’il ne connaît pas : tout ou partie du clair, de la clé, etc. On parle de **sécurité sémantique**.

→ moindre gain d’information = le schéma n’est pas sûr


## 2.2. Modèle d’attaquant
**Principe de Kerckhoff** : la sécurité d’un algorithme ne saurait résider dans son caractère secret.

Un **attaquant** est caractérisé par sa connaissance de la spécification de l’algorithme, ses capacités (calcul, mémoire, action) et un scénario (contexte d’apprentissage et défi).

Complexité algorithmique : limite atteignable 2⁶⁴, minimum inatteignable 2⁸⁰, raisonnablement inatteignable dans les 30 ans à venir **2¹²⁸**.

**Contextes** pour l’attaquant du moins au plus favorable : 
- chiffré connu : observation seule des chiffrés
- clair connu : observation de couple (clair, chiffré)
- clair choisi : oracle de chiffrement (on obtient le chiffré des clairs choisis)
- chiffré choisi : oracle de déchiffrement

**Défi** par ordre décroissant de difficulté : trouver la clé (=cryptanalyse), une partie de la clé, déchiffrer un chiffré particulier, trouver une info sur un clair particulier, ...


## 2.3. Modèles de sécurité
Sécurité inconditionnelle : l’attaquant ne peut pas résoudre le défi quelle que soient ses capacités. Elle n'existe qu'en théorie mais en pratique le processus de chiffrement devient beaucoup trop lourd.

**Sécurité calculatoire** : il y a toujours des attaques de complexité (temps, mémoire) fini, qui sont fonction des paramètres de l'algorithme (taille de la clé, ...). Il suffit alors que cette complexité sont supérieures aux capacités technologiques.

**Niveau de sécurité** : complexité de la meilleure attaque connue.

Deux déclinaison de la notion de sécurité :
- **Sécurité concrète** : le niveau de sécurité est supérieur aux capacités technologiques
- **Sécurité intrinsèque** : le niveau de sécurité est celui de la brute force (aucune attaque intelligente n’est meilleure que la brute force).

En général, on prend un attaquant fort : contexte le plus avantageux possible et défi le plus facile. Ce n'est pas forcément réaliste mais « qui peut le plus, peut le moins ».
Cependant, un algorithme qui ne résiste pas à un attaquant fort n'en est pas pour autant faible !