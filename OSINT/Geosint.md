```toc

```

# La base
Google maps
Google Earth

# Geoportail
https://www.geoportail.gouv.fr
Site de l'état regroupant plein de fond de carte : IGN, vue satellite, plan cadastrale, relief, ...
Contient aussi de nombreux outils : calcul de distance, vue de tous les endroits accessible à moins de 5km / 10min en voiture (isochrone / isodistance), ...

# Overpass turbo
https://overpass-turbo.eu/
Outil très pratique pour faire des requêtes sur les données de OpenStreetMap et afficher tous les cinémas à moins de 300m d'un Mc Donalds par exemple.

Documentation utile car la syntaxe est un peu pénible : https://osm-queries.ldodds.com/tutorial/index.html.

```c
[out:json][timeout:25];
// gather results
(
  nwr["amenity"="fast_food"]["brand"="McDonald's"]({{bbox}}) -> .mcdo;
  nwr["amenity"="cinema"](around.mcdo:1300) -> .cinema;
);

// print results
.cinema out geom;
```

Tips :
- Les attributs sont ceux de OpenStreetMap, donc se référer à leur documentation selon les objets recherchés (parc, rivière, muraille historique, route limitée à 30km/h, ...)
- Attention à ne pas trop restreindre une recherche sans le faire exprès. Exemple : `  nwr["shop"="supermarket"]["brand"="Franprix"]({{bbox}});` donnera moins de résultats que `nwr["brand"="Franprix"]({{bbox}});`
`
# What 3 words (W3W)
Chaque carré de 3m par 3m peut être identifié par une combinaison de 3 mots, utile pour transmettre des coordonnées via un code : https://what3words.com.