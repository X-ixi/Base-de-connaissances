Input : trame pcap contenant des requêtes pour récupérer des fichiers en FTP, en SMB et en NFS.
Objectif : reconstruire ces fichiers et récupérer 3 flags.

# FTP
Il suffit de :
- Sélectionner un paquet `FTP-DATA` du fichier que l'on souhaite exporter
- Clic droit, `Follow > TCP Stream`
- Afficher les données en `Raw` 
- Cliquer sur `Save as` et entrer le nom du fichier et de l'extension 

# SMB
Il suffit d'aller dans `Fichier > Exporter Objets > SMB` et de choisir le fichier à exporter.

# NFS
On commence par identifier des requêtes à de nombreux fichiers, mais si on suit l'arborescence, ils sont tous dans le dossier `/home/tidusrose/NFS/Trash` et aucun de ces fichiers n'est lu.
Ensuite, l'utilisateur accède au fichier `/home/tidusrose/NFS/.hidden` et récupère son contenu via 5 requêtes NFS `READ <offset> <lengh>`.

Pour chacune de ses requêtes READ, on exporte l'attribut "data" dans un fichier binaire (export packet bytes). On remarque que le premier fragment est identifié comme étant un format mp4.
Ensuite on concatène les 5 morceaux comme suit :
```bash
cat frag1.bin frag2.bin frag3.bin frag4.bin frag5.bin > full_video.mp4
```

Puis il suffit d'ouvrir le fichier :)