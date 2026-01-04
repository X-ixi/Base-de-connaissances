
Dans un terminal, taper `tmux`.

Pour faire une action il faut systématiquement faire `Ctr+a` puis la touche de l'action (par défaut c'est `Ctr+b` mais je trouve ça moins pratique).

- `%` : split horizontal en 2 zone droite et gauche
- `"` : split vertical en 2 zone haut et bas
- `<-`, `->`, `^`, `v` : passer au terminal de gauche, droite, haut ou bas
- `[` : rentre en mode permet de scroll (puis `q` pour quitter)
- `q` : affiche le numéro des terminal
- `t`: affiche une horloge
- `x` : supprime le terminal sélectionné


- `:` : permet de taper des commandes entières
	- `:resize-pane -L 20` : agrandit le terminal sélectionné sur la gauche (possible de remplace L par D, U ou R -> Down, Up, Left, Right)
