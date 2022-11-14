``` toc

```

Alice veut s’authentifier auprès de Bob.
Le protocole doit échouer si Alice est malhonnête (i.e. n’est pas Alice) ou que Bob est malhonnête (il simule l’authentification d’Alice auprès de quelqu’un d’autre).

**Principe** : Alice possède un secret notoirement attaché à son identité et prouve à Bob qu’elle le possède sans le révéler.


# 1. Mot de passe 
L'authentification par mot de passe est une **authentification faible**.
Elle suppose que le vérificateur soit **honnête** : système, serveur de confiance.

On ne stocke pas les mots de passe en clair mais les hash des mots de passe (cf [[(Crypto) 4 - Intégrité symétrique et hachage#3. Fonction de hachage]]). Alice entre son mot de passe, Bob calcule le hash et vérifie la conformité avec la base de données.

Attaques possibles : par dictionnaire (hors ligne) ou par rejeu si la saisie du mot de passe est mal protégée.


## 1.1. One Time Password (OTP)
t : constante définissant le nombre d’authentification (connue de Alice et Bob)
w : secret (connu de Alice uniquement)
H : fonction de hachage (connue de Alice et Bob)

Le premier mot de passe utilisé par Alice est : `w0 = H^t (w)` (H appliqué t fois)
Le serveur conserve w0
Le deuxième mot de passe est : `w1 = H^(t-1) (w)`
Le serveur vérifie si `H(w1) = w0` et conserve w1
...
A la i-ème authentification, wi = H^(t-i) (w) et le serveur vérifie H(wi) = w(i-1)

Le serveur n’a qu’une valeur à garder en mémoire qu’il met à jour à chaque itération : w0, w1, w2, etc.

Un attaquant cherchant à se connecter doit trouver w(i+1), ce qui est impossible même en connaissant wi car H n’est pas inversible.


# 2. Challenge / réponse
L'authentification par challenge / réponse est un **authentification forte**.
Bob envoie un challenge à Alice qui doit le résoudre pour s’authentifier.

On se base sur le **chiffrement asymétrique** (cf [[(Crypto) 2 - Chiffrement symétrique]]) :
1. Bob envoie un message chiffré avec la clé publique d'Alice
2. Alice le déchiffre avec sa clé privée et lui renvoie le message déchiffré
3. Bob vérifie le message déchiffré.

On peut aussi se baser sur la **signature asymétrique** (cf [[(Crypto) 5 - Signature]]) :
1. Bob envoie un message aléatoire à Alice
2. Alice le signe avec sa clé privée et lui renvoie la signature
3. Bob connaît la clé publique pour vérifier.

Cette méthode est résistante car même si Bob est malhonnête et veut usurper l’identité d’Alice, il ne le pourra pas car il ne connaît pas la clé privée d’Alice.


# 3. Zero knowledge
Un observateur extérieur ne doit apprendre aucune information sur le secret. Il n'y a **aucune usure du secret**.
En théorie, à force d’observer des authentifications réussies (message et signature), un attaquant pourrait réussir à usurper l’identité.
→ en pratique les clés privés sont renouvelés assez souvent pour que ce soit calculatoirement impossible.

Principe : le protocole se déroule en **plusieurs itérations**, Alice **n'a pas besoin d'utiliser son secret à toutes les itérations**, Bob est convaincu ) la fin du protocole MAIS il ne peut pas transférer cette conviction et aucun spectateur ne peut-être convaincu.


## 3.1. Allégorie de la caverne
Une itération du protocole se déroule comme suit :
1. Alice choisit au hasard d'aller à gauche ou à droite
2. Bob entre quand Alice est hors de vue
3. Bob choisit au hasard un couloir et demande à Alice de sortir par ce couloir
	1. Soit Alice est dans ce couloir et sort
	2. Soit Alice est dans l'autre couloir et ouvre la porte avec sa clé (la séparation entre A et B sur le schéma est une porte et non une cloison) puis sort par le bon couloir
![[zero_knowledge_auth.png]]

Alice à la clé et peut ouvrir la porte donc elle peut sortir dans tous les cas.
Un usurpateur n’a qu’une chance sur deux de pouvoir sortir par le bon côté.
→ On répète l’opération 128 pour assurer le niveau de sécurité de 2¹²⁸

Seul Bob peut être convaincu de l’authentification et aucun autre spectateur du protocole ne peut l'être car ils ne peuvent pas savoir si Bob n’est pas complice de la prétendue Alice et n'aurait pas dissimuler toutes les fois ou elle est sortie du mauvais côté.


## 3.2. Exemple concret
Alice possède n = pq un entier RSA (n public, p et q privé), s secret et v public tel que :  
**s² = v (mod n)**
→ connaissant v et n, il est très difficile de retrouver s, mais connaissant p et q c’est facile

1. Alice choisit r aléatoire et transmet `x = r² (mod n)` à Bob
2. Bob choisit aléatoire un bit b et le transmet à Alice
3. Alice calcule `y = r / s^b (mod n)` et le transmet à Bob
4. Bob vérifie `y² * v^b = x (mod n)`

b : bit challenge valant 0 ou 1 → dans un cas sur 2 le secret s n’est pas utilisé (c’est ce qui permet d’assurer la non divulgation d’information sur le secret).