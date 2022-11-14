``` toc

```

# 1. Crypto quantique
Cela consiste à concevoir des algorithmes cryptographiques mettant en jeu des objets quantiques. La sécurité repose alors sur les **lois de la mécanique quantique**, et non plus seulement sur des résultats mathématiques.
C’est une **alternative à la cryptographie classique**.


# 2. Crypto post-quantique
Quel méthodes cryptographiques (AES, RSA, …?) pourra-t-on utiliser quand un ordinateur quantique existera ?
Un ordinateur quantique (de quelques milliers de q-bits) pourrait réaliser certaines opérations beaucoup plus vite qu’un ordinateur classique et certaines attaques seraient beaucoup plus efficaces.

## 2.1. Impact sur la cryptographie symétrique
Cf cours [[(Crypto) 2 - Chiffrement symétrique]].
Un ordinateur quantique permet une recherche exhaustive en O(sqrt(nb d’éléments))
→ le niveau de sécurité devient sqrt(2^(taille clé)) = 2^(taille clé / 2)
=> **il faut doubler la tailler des clés** (une version 256 bits de l’AES existe déjà)


## 2.2. Impact sur la cryptographie asymétrique
Cf cours [[(Crypto) 3 - Chiffrement asymétrique]].
Un ordinateur quantique permet de factoriser n en complexité O((log n)³), et similaire pour le logarithme discret.
→ l’ordinateur quantique cracke aussi vite que le destinataire déchiffre
=> **gros problème** (RSA de niveau 2¹²⁸, il faut n de taille 2⁴³ au lieu de 3072 bits).

D’autres problèmes mathématiques que la factorisation et le logarithme discret sont explorés : recherche de vecteurs courts dans un réseau euclidien, résolution d’équation polynomiales multivariées, …
→ des algorithmes les implémentant existent déjà mais utilisent des clés plus importantes : 500 000 bits pour une sécurité de 2¹²⁸.

Croyance : l’ordinateur quantique aura aussi du mal à résoudre des systèmes linéaires d’ordre élevé.
