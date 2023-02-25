``` toc

```

# 1. Sûreté de fonctionnement

La sûreté de fonctionnement (SF) est un besoin pour les systèmes répartis (dépendance critique entre des calculateurs non fiables).
Un SR est un moyen de mettre en oeuvre la SF grâce à la redondance qu'il apporte.

La SF doit faire face aux fautes provenant de la conception, du développement et d'exécution.

Definitions :
- Faute : cause (humaine de mauvaise conception par ex, et pas issue de phénomènes naturels)
- Erreur : conséquence observable au niveau du système
- Défaillance : comportement non conforme


## 1.1. Tolérance aux fautes
Eviter les fautes est indispensables mais coûteux, par conséquent il faut être capable de les tolérer.
-> redondance : duplication des composants, des traitements et/ou des données

La redondance permet de mettre en place des techniques de récupération (error recovery) et de compensation (error masking).
Il faut bien **identifier les fautes à tolérer** pour concevoir la redondance appropriée.


## 1.2. Réplication active
Contrairement au réplication passive ou un seul élément est utilisé et les redondances ne sont utilisées que en cas de défaillance (copie de secours), les réplications actives sont toutes utilisées en même temps et exécute le même traitement.

Hypothèses :
- Diffusion fiable des requêtes : une requête est reçue par tous les destinataires ou par aucun.
- Les requêtes sont reçues dans le même ordre par toutes les copies
- Copies avec des comportements déterministes, on peut renvoyer la réponse de n'importe quelle copie
  -> sinon on peut prendre la réponse majoritaire



# 2. Pannes franches

Ah bah ça marche pas...

