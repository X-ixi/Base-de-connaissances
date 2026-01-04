``` toc

```

# 1. Big data

On parle de gros volume de données, c'est-à-dire, en exaoctets ou zettaoctets.
Exemples de cas d'usage : médecine (consultations médicales en ligne, dossier médical, ...), transport (réservation en lignes, GPS, ...), ...
![[big_data.png]]
"De plus grosses données ne sont pas toujours de meilleures données."

Les 5 V :
- *Volume* : on produit beaucoup de données que l'on doit stocker et analyser
  -> nécessite un stockage distribué, de faire des échanges de données sur le réseau et de faire des calculs distribués
- *Velocity* : on les produit à une grande fréquence et il faut les traiter en un temps fini
  -> traitement en "temps réel" et adaptabilité à une fréquence de production variable
- *Variety* : on doit traiter des données hétérogènes
  -> mélange de données structurées (SQL, page web, ...) et non structurées (image, texte, son, ...)
- *Veracity* : on doit s'assurer de la qualité des données 
  -> filtrer les données pour garder seulement les données valides
- *Value* : quelle est la valeur ajoutée de faire du big data



# 2. Calcul distribué

On a atteint la limite de la fréquence des processeurs, on ne peut pas l'augmenter sans que la puissance a dissipée en chaleur ne deviennent trop importante.
-> On augmente le nombre de cœur et on fait du parallélisme

Parallélisme de données (instructions vectorielles) et parallélisme par flot d'instructions (plusieurs cœur en parallèle), on se concentre sur le deuxième cas ici.

Plusieurs architectures possibles :
- M*émoire partagée* : une mémoire partagée entre tous les processus (pc)
- *Mémoire distribuée* : une mémoire par processus (cluster de serveur)

L'accélération des traitements grace à la parallélisation a des limites. Pour le mesurer, on définit la latence T / C, avec T le temps de traitement et C la charge (i.e. le nombre d'instruction).  On note l'accélération S = latence / latence avec parallélisation.
Dans un programme, tout ne peut pas être paralléliser, on note alors `p` le pourcentage du temps d'exécution parallélisable.

On a alors les lois suivantes :
- Loi d'Amdahl : **à charge constante** $S = \frac{1}{1 - p + p/s}$, donc même avec plus de cœurs, s augmente donc on tend vers 1/(1-p). La part non parallélisable n'est pas compressible.
  -> pour les particuliers, les pc ne dépasse jamais 10 cœurs
- Loi de Gustafson : **à temps d'exécution constant** $S = 1 -p +sp$. Il n'y a pas de limite à l'accélération dans ce cas.
  -> pour les professionnels qui mettent à disposition du temps de calcul, on trouve du 128 cœurs et plus

Dans le cas du big data on se place plutôt dans le cas de la loi de Gustafson.


Super calculateur d'aujourd'hui : mise en commun d'un très grand nombre de serveurs presque classiques (pas d'hardware très spécifique). Cela est très modulaire et scalable.
-> comme le materiel de chaque serveur ne coute pas très cher, il est susceptible de tomber en panne et il faut prévoir le système global pour **résister à ces pannes**.
=> Utilisation de framework de calcul distribué : Hadoop, Spark, ...


## Problématique du stockage
Objectif : partager un même système de fichiers via des montages simultanés sur différents noeuds

### NAS
*Network Attached storage*
Boitier branché au réseau qui contient des disques durs classiques et un OS très simple avec un système de fichier.
Protocole utilisé : NFS pour Linux et SMB pour Windows.
![[nas.png]]
C'est une solution peu coûteuse, facile à utiliser, mais lente, peu adaptée au accès concurrents et qui passe mal à l'échelle.
-> Pas adapté au calcul distribué

### SAN
*Storage Area Network*
Baie de stockage dans lesquels on branche les disques durs. Les disques durs sont branchés directement sur le réseau et il n'y a pas de système de fichiers dans le SAN
![[san.png]]
C'est très performant, très fiable mais complexe, coûteux et ne supporte pas le partage multi-sites.

### Système de fichiers distribués
Chaque machine possède des disques durs en local et c'est le système de fichiers qui est distribué entre les machines.
![[distributed_file_system.png]]
C'est performant, peu coûteux et peut se partager en multi-sites. On peut / il faut tirer parti de la localisation des données car c'est bien plus rapide d'accéder à une donnée plus proche.
-> C'est l'architecture utilisée par tous les framework, ex HDFS (Hadoop Distributed File System), googleFS, ...



# 3. Cloud computing

Le cloud a des propriétés particulièrement intéressantes pour le big data et le calcul distribué : mise à l'échelle rapide, paiement à l'usage, abstraction de l'administration des serveurs, ...



# 4. Architecture Lambda

Globalement 2 approches :
- Traitement d'un très gros volume de données qui prend du temps
- Traitement presque en temps réel de données plus petites mais produite avec une fréquence importante
-> Beaucoup de techno différentes pour faire les deux
![[architecture_lambda.png]]
La majorité de ces framework fonctionnent sur la JVM (machine virtuelle Java) et sont open source.

La tendance actuelle est de regrouper les 2 approches en traitant des batchs petits pour se rapprocher du streaming.



# 5. MapReduce

Framework initialement créé par Google pour ces besoins internes et qui repose sur un système de fichiers distribué (GoogleFS), qui a ensuite été repris et rendu open source par Yahoo en produisant Hadoop.

Deux fonctions au coeur de MapReduce :
- **map(L, f)** avec L une liste d'éléments `[x0, ..., xn]` et f une fonction qui attend un argument et rend une valeur 
  -> renvoie `[f(x0), ..., f(xn)]
- **reduce(L, f, v)** avec L liste d'éléments, f fonction attendant 2 argument et rendant une valeur et v valeur initiale
  -> renvoie `f(...f(f(v, x0), x1)..., xn)`

Hadoop implémente également des étapes split et shuffle qui sont réalisés par le framework lui-même. L'étape split divise les données à l'aide d'un ensemble de clés, on a alors xi = (clé, valeur). L'étape map modifie seulement les valeurs. Ensuite l'étape shuffle regroupe toutes les données par clés.

![[mapreduce.png]]



# 6. Spark

Le framework MapReduce est lent car il nécessite un accès au disque en lecture avant puis un accès en écriture après chaque map-reduce, et il nécessite aussi beaucoup de communication réseau durant la phase de shuffle. 
-> Inefficace pour des algorithmes qui utilise beaucoup d'étape MapReduce (machine learning, traitement de type streaming, ...)

Objectif de spark : tirer parti de la RAM des machines
-> Création d'une nouvelle structure de données : les RDD, qui resteront au maximum dans la RAM

Spark est inspiré de MapReduce mais propose beaucoup plus de types d'opérations et avec plus de souplesse (pas forcément split puis map puis shuffle puis reduce puis writeback).

Spark est codé en Scala (donc tourne dans une JVM) mais il y a aussi une API compatible avec Python.

### Spark cluster
Le Driver Program contrôle les processus qui s'exécute sur les Worker Node. Le SparkContext gère la tolérance aux fautes et relance les tâches si nécessaires.
![[spark_cluster.png]]

Les données sont mises en cache dans des structures de données : des RDD. Ensuite le code à exécuter est envoyé à chaque Worker Node qui l'exécutera sur les RDD du cache et enregistrera la sortie dans des RDD en cache.

On peut utiliser Spark dans un shell interactif, et donc dans un Jupyter Notebook.

### RDD
*Resilient Distributed Dataset* : ils représentent une collection d'objets, en particulier des paires (clé, valeur) dans le cas de programme compatible MapReduce.
Ils sont toujours mis en RAM (sauf si leur taille ne le permet pas). Les RDD peuvent être séparé en plusieurs partitions, potentiellement réparti sur plusieurs machines.

Les RDD sont immutables, ce qui permet une meilleure tolérance à la faute. Ils ne sont pas persistants et le framework se charge de supprimer les RDD quand on n'en a plus besoin (analyse du code et de l'état des différents noeuds).

### Transformations
Les transformations en Spark sont exécutée de manière paresseuses, c'est à dire exécuter au moment de l'exécution d'une *action* qui termine un traitement.

### Exemples
Exemples avec `rdd = {1, 2, 2, 3}`:
- `rdd.map(x => x+1) = {2, 3, 3, 4}`
- `rdd.map(x => x.to(3)) = {{1, 2, 3}, {2, 3}, {2, 3}, {3}}`
- `rdd.flatMap(x => x.to(3)) = {1, 2, 3, 2, 3, 2, 3, 3}`
Autres transformations : filter, distinct, sample, union, intersection, ...

Transformations sur des (clé, valeur) : reduceByKey (idem à reduce de MapReduce), groupByKey, join, ... 

Les transformations peuvent être de type "narrow" (pas d'échange sur le réseau nécessaire, ex : map) tandis que d'autres sont de type wide (ex : reduceByKey).
-> un ensemble de transformations successives de type "narrow" est appelé un stage

Une action termine le traitement (i.e. lance l'exécution de toutes les transformations annoncées avant) et donne un résultat de type "small data" que l'on peut afficher à l'écran ou enregistrer sur le disque.
ex : collect, count, countByKey