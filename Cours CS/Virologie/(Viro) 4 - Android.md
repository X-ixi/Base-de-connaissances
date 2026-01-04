``` toc

```

# 1. Rappels

## 1.1. Application android
Android est un OS avec en cœur une instance de machine virtuelle interprétant du bytecode (Dalvik) ou exécutant le bytecode compilé (Art).
Dalvik : ré-implémentation de la machine virtuelle Java (JVM)
-> code en Java, ou code créant du bytecode Java (Kotlin par exemple)

Tache de la machine virtuelle :
- Communique avec d'autres processus (contacts, comptes, ...) via le *Binder*
- Utilise un seul thread
- Gère les évènements systèmes (reception d'un message, ...)

Une application android est composée de :
- **Activités** : applications ayant une interface utilisateur
- **Services** : tâche de fond
- Fournisseur de contenu (content provider) : partage d'information entre applications
- **Intents** : envoie un message à un composant externe sans le nommer explicitement
- Récepteur d'intents (broadcast receiver) : capable de répondre des intents
- Notifications : notifie l'utilisateur

Le **manifest** de l'application est un fichier en XML décrivant l'application et déclarant ses activités, ses services, ses intents, ...

Déploiement d'une application android
![[deploiement android.png]]
Classes.dex est soit exécuté par Dalvik, soit compilé par ART pour être exécuté comme un binaire ensuite.


## 1.2. Intents
Les intents permettent de gérer l'envoi et la réception de message afin de faire coopérer les applications. Le but est de déléguer une action à un autre composant (autre application ou autre activité de l'application courante).

Un objet Intent contient :
- Le nom du composant cible (facultatif - on ne connait pas le navigateur par défaut par exemple)
- L'action à réaliser
- Les données
- Une catégorie pour cibler un type d'application

Le manifest définit les intents que l'application va écouter et lorsqu'un intent qu'elle écoute est émis, le récepteur s'exécute. Cette méthode de reception est déprécié pour la majorité des intents car trop d'application écoutait trop d'intent et cela causait des problèmes de performances. L'autre solution est de créer le récepteur directement dans le code.

Exemples d'intent prédéfinis : screen on / off, power connected / disconnected, shutdown, boot completed, call (passer un appel), send (envoyer sms ou email), web search, ...



# 2. Système de permissions

Les intents ne nécessite pas de permissions car il délègue l'action à réaliser : l'application demande d'envoyer un sms mais c'est l'application sms qui s'en charge.

Si une application veut envoyer un sms directement, elle doit en avoir la permission. Le manifest décrit les permissions que l'application nécessite. Pour la plupart des permissions, une demande formelle (popup) est faite à l'utilisateur (ce n'est pas le cas pour l'accès à internet par exemple).

Exemples de permissions : lire les sms, accéder à la caméra, lire le stockage, envoyer un sms, passer un appel, écrire un contact, redémarrer, tuer un processus, installer une autre application, ...

Cas particulier des fichiers :
- *Internal storage* : fichiers internes de l'application auxquels l'utilisateur n'a pas accès
  -> pas de permission nécessaire
- *External storage* : fichier utilisateurs (photos, ...)
  -> permission de lecture / écriture



# 3. Retro-ingénierie
![[retro android.png]]

Les applications Android sont très facilement décompilables, car ceux sont des applications java qui tournent dans une machine virtuelle.

Outils intégrés apktools : 
- baksmali : désassemblage du fichier dex en fichier smali au format jasmin (humainement lisible)
- smali : ré-assemblage du fichier dex -> permet de modifier l'apk avant de la recompiler

![[compilation android.png]]

android.jar : classe de base d'android
certificate : certificat du développeur (la mise à jour d'une app ne peut avoir lieu que si le certificat des apk des 2 versions est le même)

Attaque par repackaging : 
1. Désassemblage
2. Etude du code source et ajout de code malveillant à l'intérieur
3. Compilation
4. Assemblage et signature

La signature de l'application malveillante n'est pas la même mais rien n'empêche de la mettre sur le store et que quelqu'un la télécharge (Android ne vérifie pas l'identité des développeurs, contrairement à Apple).


## Outils de retro
Quelques outils de retro plus évolué :
- dex2jar : convertir le fichier dex en fichier jar puis utilisation d'outils spécifiques aux fichiers jar (bytecode Java traditionnel)
- Jadx et ByteCodeViewer : outils de décompilation du dex en Java (ou smali) avec interface graphique
- Editeur de bytecode Java (JByteMod, Cafebabe Lite, Recaf) : plutot que de décompiler en Java puis modifier puis recompiler, permet de décompiler puis modifier "facilement" le bytecode.



# 4. Malware

Objectifs possibles :
- S'attaquer aux données utilisateur
- Gagner de l'argent en déclenchant des services payants
- Introduire des pub aggressives

Globalement la faille est humaine : l'utilisateur donne les permissions nécessaires au malware.

Type de malware : 
- Worm : existe peu car difficile de se répliquer d'un téléphone à l'autre
- Botnet
- Spyware : le plus courant (données bien rangées et légère, contrairement aux pc)
- Adware
- Ransomware / cryptominer

Exemples de malware : envoie de SMS à un numéro sur-taxé en boucle, exploit d'une faille de l'OS android, publicité aggressive en plein écran (attention virus, ...)

Souvent le malware reste inactif pendant quelques heures, ce qui rend l'analyse et la détection plus difficile, et l'utilisateur ne fait pas le lien. 



# 5. Sécurité

## 5.1. Signature des applications
L'application doit être signée pour être installée et chaque mise à jour doit être signée avec la même clef privée du développeur.


## 5.2. Obfuscation
De manière générale, 3 types d'obfuscation :
- De représentation : anonymiser les noms des variables, des fonctions, ...
- De contrôle : modifier les boucles et la structure du code
- De données : couper une chaîne de caractère en plusieurs morceaux, changer d'encodage, ...

*R8* : Google code shrinker pour android
-> utiliser pour réduire la taille du code et l'obfusquer : enlever code inutile, fonction inline, remplacer les enum par des constantes, ...

Toutes les classes et méthodes ne peuvent pas être renommées car certaines sont déclarées dans le manifest.

L'obfuscation est mise en place sur quasiment toutes les applications android


## 5.3. Anti retro-ingénierie
Techniques plus avancées que l'obfuscation : 
- **Malformation volontaire du bytecode** pour que les décompileurs standard échoue
- **Chargement dynamique de code** : l'analyste doit récupérer les fichiers chargés dynamiquement (présent sur le disque ou sur le réseau, potentiellement chiffré)
- **Code natif** : code déporté dans des librairies natives (fichier .so), le code est réparti à plusieurs endroits et plus dur à analyser
- **Packing** : le développeur passe son dex en entrée d'un packer qui créé une application APK packé dont le dex est le code du depacker. Ce depacker est en charge de charger le dex original lors de son exécution. Le packer ralentit le démarrage de l'application.
- **Anti debugging** : l'application détecte si elle est en mode debug (flag de debug, mesure du temps d'exécution, breakpoint déclarés, ...) et adapte son comportement

Les applications légitimes mettent rarement en place ces techniques mais les malwares oui.


## 5.4. Architecture android et sa sécurité
La sécurité est aussi renforcée dans le noyau : les applications sont isolées les unes des autres, et les accès aux ressources du téléphone sont contrôlés (permissions). 

Android utilise les utilisateurs Linux pour isoler les applications : les services systèmes démarrent avec l'UID 1000 et les applications avec l'UID 10000.
Les données applicatives appartiennent à l'utilisateur uY_aXXX (utilisateur Y et application XXX), ce qui empêche les autres utilisateurs d'y accéder.

La gestion des permissions se basent sur celle implémentée dans Linux avec les UID et les GID.


## 5.5. SELinux
SELinux (Security Enforced Linux) s'invite progressivement dans android.
Cela permet de créer une séparation de différent contextes de sécurité avec des règles exhaustives des actions autorisées.
