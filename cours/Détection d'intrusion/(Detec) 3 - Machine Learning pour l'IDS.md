``` toc

```

# 1. Apprentissage automatique

Beaucoup de hype sur le ML car : 
- Facile de stocker de grande quantité de données
- Grosse puissance de calcul avec les GPU
- Avancée théorique

Inclusions : IA > ML > Deep Learning

Le ML se déroule en 2 phases :
1. Apprentissage
2. Inférence

"Garbage in, garbage out" -> sortie du ML jamais meilleure que la qualité des données

Comment détecter une intrusion avec du ML :
- **Détection par règle/signature** -> nécessite des exemples labellisés (supervisé)
  On peut aussi extraire des règles à partir du modèle appris.
- **Détection d'anomalies** -> exemples de traffic normal (semi-supervisé)
  Cela permet de modéliser le traffic "légal"/légitime.
- **Détection de valeurs aberrantes**
  On cherche les entrées très distinctes des autres.


# 2. Quelques modèles de ML

- k-NN : k plus proches voisins
- Arbre de décision : ensemble de règle de décision (si "condition" alors "nœud")
- Random Forest : apprentissage d'arbres de décision sur des sous-ensembles de données puis vote majoritaire
- Réseaux de neurones


# 3. Quelques approches

PAYL : fréquence d'apparition des octets dans un flux http (si elle diffère de la normale, le flux est suspicieux)

N-grams : groupe de N appels systèmes observés à la suite
ex : la séquence "open, read, mmap, mmap" est transformé en les 2-gram suivants : "open, read", "read, mmap" et "mmap, mmap".
On peut alors modéliser les comportements avec un ensemble de N-grams.
MAIS rien n'empêche un attaquant de faire des appels systèmes inutiles pour brouiller les pistes.


# 4. Critiques et limitations

De bonnes performances pour les attaques à gros volumes : DDoS et scan de ports, mais plus de difficultés pour les attaques plus fines : XSS, ...

Limites classiques du ML : besoin de données réalistes mais saines, trop de faux positifs car jeu de données très déséquilibré (très peu d'attaques), ...
