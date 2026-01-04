
# Théorie du DNS rebinding
L'attaquant dispose d'un nom de domaine attaquant.fr qu'il a enregistré sur un serveur DNS qu'il contrôle. Puis :
- Il fait du phishing sur un utilisateur qui clique sur le lien attaquant.fr
- Le PC de la victime fait une requête au DNS de l'attaquant et récupère l'IP du serveur de l'attaquant (avec un TTL=0 pour éviter que cette réponse ne soit mise en cache)
- Le navigateur de la victime fait une requête au serveur contrôlé par l'attaquant et reçoit un site web avec du code javascript
- Le code javascript fait une nouvelle requête au domaine attaquant.fr, mais cette fois le serveur DNS répond avec une IP interne à l'entreprise de la victime.

Cela permet de contourner la SOP (Same Origin Policy) car la requête faite par le script reçu du domaine attaquant.fr est faite sur le même domaine.

=> Conséquence : accès à des données internes non exposées sur internet.

Dans le cas de ce challenge, le navigateur de la victime est remplacé par le site web qui visite les url qu'on lui donne. Et on ne peut pas lui donner des scripts javascript car ils seront exécuté sur notre navigateur...


# Challenge
On a le code source d'un site web en Flask, que l'on peut simplifier comme suit (sans le html, css, try/except, etc) :

```Python
import re, html, ipaddress, socket, requests, random, string, flask, sys    
from urllib.parse import urlparse 

FLAG = ""+open(".passwd").readlines()[0]+""    
AUTHORIZED_IPS = ['127.0.0.1', '::1', '::ffff:127.0.0.1']
AUTH_TOKEN = ''.join(random.choice(string.ascii_letters + string.digits) for i in range(42))

random.seed(FLAG)    
app = flask.Flask(__name__, static_url_path='/static')


### Super secure checks
def valid_ip(ip):
	try:
		result = ipaddress.ip_address(ip)
		return result.is_global # Not a LAN address
	except Exception as e:
		return False

def valid_fqdn(fqdn):
	return valid_ip(socket.gethostbyname(fqdn))

def check_url(url):    
	#check = valid_fqdn(urlparse(url).netloc.split(':')[0])
	check = valid_fqdn(urlparse(url).hostname)
	if check == False:
		return 'Erreur "wow-so-clever.jpg"'
	return None

def get_url(url, recursion):
	r = requests.get(url, allow_redirects=False, timeout=5, headers={'rm-token': AUTH_TOKEN})
	if 'location' in r.headers:
		if recursion > 1:
			return 'Erreur "too_far.jpg"'
		url = r.headers['location']
		check = check_url(url)
		if check is not None:
			return check
		return get_url(url, recursion + 1)
	return r.text


# Internal route, only for local administration!
@app.route('/admin')
def admin():
    if flask.request.remote_addr not in AUTHORIZED_IPS or 'rm-token' not in flask.request.headers or flask.request.headers['rm-token'] != AUTH_TOKEN:
		return 'Erreur "magicword_jurassic.jpg"'

	return "Admin page with FLAG"


# Main application    
@app.route('/grab')
def grab():    
	url = flask.request.args.get('url', '')
	if not url.startswith('http://') and not url.startswith('https://'):
		url = 'http://' + url
	check = check_url(url)
	if check is not None:
		return check
	res = get_url(url, 0)
	return res

@app.route('/')    
def index():    
	return '''
		<!DOCTYPE html>
			<html>
				<head>
					...
				</head>
				<body>
					<div>
						<h1>Mega super URL grabber</h1>
						<h6>Please be aware that I'm a nice tool, I don't grab pages that forbid me to frame them!</h3>
						<h6>Also keep out of my <a href="/admin">/admin</a> page (it's only accessible from localhost anyway...)</h6>

						<div>
							<input name="searchie">
							<div>
								<button onclick="grab()">graby-grabo?</button>
							</div>
						</div>
						
						<iframe name='framie' srcdoc="<html>Try me I'm famous</html>"></iframe>
					</div>

					<script>
						var grab = function () {
							fetch('/grab?url=' + this.document.getElementsByName('searchie')[0].value)
								.then((r) => r.text())
								.then((r) => {
							this.document.getElementsByName('framie')[0].setAttribute('srcdoc',r);
							})
						};
					</script>
				</body>
			</html>
		'''
```

La page principale a une fonctionnalité assez simple : l'utilisateur donne un URL et la page inclut le résultat d'une requếte à cet URL dans un iframe.

On souhaite visiter l'URL `/admin`, mais il n'est accessible que depuis locahost (127.0.0.1, ...).

# Analyse du code
## Requête
Lorsqu'on rentre un URL :
- Une requête à `/grab?url=<URL>`est faite
- Si l'URL est valide (pas 127.0.0.1), alors la fonction `get_url(<URL>, 0)` est appelée
- Une requête est faite à l'URL :
	- S'il n'a pas de header `location`, la réponse est renvoyée au client
	- Sinon :
		- `new_url = response.headers['location']`
		- Si le NEW_URL est valide, alors la fonction `get_url(<NEW_URL>, 1)` est appelée
		- S'il n'a pas de header `location`, la réponse est renvoyée au client, sinon c'est une erreur (pas d'autre récursion)

## URL valide
La fonction `check_url` peut s'expliquer comme suit :
- `urlparse(url).hostname` : extrait le nom de domaine de l'URL
- `socket.gethostbyname(fqdn)` : traduit le nom de domaine en adresse IP
- `ipaddress.ip_address(ip).is_global` : vérifie si l'adresse IP est sur une plage d'adressage non privée

## Page admin
Pour accéder à la page admin, il faut remplir 2 conditions :
- `flask.request.remote_addr not in AUTHORIZED_IPS` : que la requête proviennent d'un IP autorisé (à savoir 127.0.0.1)
- `'rm-token' not in flask.request.headers or flask.request.headers['rm-token'] != AUTH_TOKEN` : qu'un header 'rm-token' soit présent avec la bonne valeur


# Faille
Le code va traduire le nom de domaine en IP pour vérifier que ce n'est pas 127.0.0.1 (`check_url`), puis il va faire une requête au nom de domaine (`get_url`), et pas à l'IP traduite précédemment !
=> Si l'attaquant contrôle le DNS, alors il peut :
- Répondre à la première résolution par un IP public qu'il possède (`check_url`)
- Répondre à la deuxième résolution par 127.0.0.1 (`get_url`)


# Tentatives
location : root-meURL/admin ?

```bash
echo -e "HTTP/1.1 302 Found\r\nLocation: http://challenge01.root-me.org:54022/admin\r\n\r\n" | nc -l -p 8000

# In another terminal
ngrok http 8000

# Then give the graby-grabo the ngrok url
```

