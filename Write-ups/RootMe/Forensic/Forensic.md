
# 1 - fichier supprimé
Input : fichier usb.image
Objectif : trouver le nom du propriétaire de la clé USB

```bash
> file usb.image
DOS/MBR boot sector

> strings -n usb.image
<rdf:RDF xmlns:rdf='http://www.w3.org/1999/02/22-rdf-syntax-ns#'>
 <rdf:Description rdf:about=''
  xmlns:dc='http://purl.org/dc/elements/1.1/'>
  <dc:creator>
   <rdf:Seq>
    <rdf:li>Javier Turcot</rdf:li>
   </rdf:Seq>
  </dc:creator>
 </rdf:Description>
</rdf:RDF>
```
-> On obtient le nom du propriétaire

# 2 - capture moi ça
Input :
- Fichier keepass : Database.kdbx
- Image : Capture.png
Objectif : retrouver le mot de passe de la base keepass

La capture d'écran laisse penser qu'elle a été rognée et que le mot de passe du keepass se trouve dans la zone rognée... Est-il possible de récupérer la zone rognée ?

## Recherche à l'aveugle
Le site : https://www.nayuki.io/page/png-file-chunk-inspector permet d'inspecter les chunk de données du fichier PNG un par un, et il semble que le dernier chunk n'est pas pris en compte car il est situé après le chunk de IEND (end of image).

```bash
# On vérifie la position de la section que l'on veut supprimer (end of file)
hexdump --skip 162156 --length 12 Capture.png

# On copie le début du fichier dans un nouveau fichier
dd bs=162156 count=1 if=Capture.png of=New.png
# On copie la suite en skippant la section IEND
dd skip=162168 iflag=skip_bytes bs=1M count=1 if=Capture.png of=New.png conv=notrunc oflag=append
```

Malheureusement, c'est une impasse...

## La vulnérabilité
Finalement, en regardant la capture de plus près, on identifie l'outil utilisé qui est l'outil Snip & Sketch de capture d'écran sur Windows.
On découvre alors qu'une vulnérabilité a été découverte sur cet outil et permet de récupérer le contenu croppé ! C'est la CVE-2023-28303, aussi appelée "acropalypse".

La faille est que l'image croppée est réécrite sur l'image initiale, seulement comme l'image croppée est plus petite, la fin du fichier initial est toujours présent !

Finalement, un exploit est disponible sur ce repo git : https://github.com/frankthetank-music/Acropalypse-Multi-Tool
On réussit à récupérer une partie de l'image rognée et le mot de passe !

