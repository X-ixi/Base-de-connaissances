
# Quick fix
```bash
xrandr --dpi 282 --fb 8960x5040 \
	--output DP-1 --pos 0x0 --panning 5120x2880+0+0 --scale 2x2 \
	--output eDP-1 --mode 3840x2160 --pos 640x2880
```
___

# Explications

`xrandr` est un outil installé par défaut sur linux permettant de gérer l'affichage : la disposition des écrans, leurs résolutions, ....

Très bon blog pour comprendre : https://blog.summercat.com/configuring-mixed-dpi-monitors-with-xrandr.html

# Problèmes
Mon setup :
- Un pc avec un écran de 15 pouces UltraHD : 3840x2160
- Un écran secondaire de 28 pouces 4k : 3840x2160

Les problèmes associés :
- Par défaut, la résolution de mon écran secondaire est mis à 2560x1440 et sa vraie résolution n'est pas disponible
- Il faut configurer un zoom de 200% sur l'écran de mon pc et de 100% sur l'écran secondaire pour que tout soit a une taille confortable, et il n'est pas possible de définir 2 zoom différent par défaut.

# Solution
## Découverte de xrandr
On commence par lister les écrans identifiés par `xrandr` :
```bash
xrandr --listmonitors
Monitors: 2
 0: +*eDP-1 3840/346x2160/194+0+0  eDP-1
 1: +DP-1 2560/621x1440/341+3840+0  DP-1
```
eDP-1 est le nom de l'écran du pc et il fait physiquement 346mmx194mm et logiquement 3840x2160 pixels.

La commande suivante fonctionne (mais ne résout aucun problème) et place l'écran secondaire à droite de l'écran du pc
```bash
xrandr --dpi 282 --fb 6400x3600 --output eDP-1 --mode 3840x2160 --output DP-1 --pos 3840x0 --panning 2560x1440+3840+0
```
Détails de la commande :
- `dpi` : dot per inch (ou pixel per inch)
- `fb` : configure un écran global qui est la fusion de tous les écrans : 3840x2160 + 2560x1440
- `output` : sélection un écran sur lequel les options suivantes seront appliquées
- `mode` : sélectionne un mode existant
- `pos` : position de l'écran
- `panning` : configure la détection du curseur pour passer d'un écran à l'autre

## Zoom
L'objectif et de configurer un zoom de 200% sur l'écran du pc et 100% sur l'écran secondaire.
Une solution est de multiplier par 2 le nombre de pixel de l'écran secondaire et de mettre une échelle x2.
On a alors un écran secondaire de 5120x2880
```bash
xrandr --dpi 282 --fb 8960x5040 \
	--output eDP-1 --mode 3840x2160 \
	--output DP-1 --pos 3840x0 --panning 5120x2880+3840+0 --scale 2x2
```

## Placement
Objectif : garder la configuration de zoom précédente mais placer l'écran secondaire au dessus et non à droite de l'écran du pc.
On commence par configurer l'écran secondaire en 0x0, puis on place l'écran du pc en bas de l'écran secondaire à 0x2880, et si on veut le centrer en bas, on le décale vers la droite à 640x2880 (car (5120-3840)/2 = 640) :
```bash
xrandr --dpi 282 --fb 8960x5040 \
	--output DP-1 --pos 0x0 --panning 5120x2880+0+0 --scale 2x2 \
	--output eDP-1 --mode 3840x2160 --pos 640x2880
```

## Résolution d'écran
Objectif : changer la résolution de l'écran secondaire sr 3840x2160 en gardant tous les réglages précédents...

```bash
xrandr --dpi 282 --fb 11520x6480 \
	--output DP-1 --pos 0x0 --panning 7680x4320+0+0 --scale 2x2 \
	--output eDP-1 --mode 3840x2160 --pos 0x4320
```

CA NE FONCTIONNE PAS !!
Problème lié au DPI ?
