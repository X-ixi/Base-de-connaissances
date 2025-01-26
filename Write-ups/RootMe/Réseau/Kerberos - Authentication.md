
On analyse le fichier pcap avec Wireshark et on identifie les informations suivantes :
Username : **william.dupond**
Realm: **CATCORP.LOCAL**

  
On identifie également les requêtes suivantes :
- SMB Session setup request :
	- SMB Session setup response : ERROR, more processing required (= besoin de l'auth kerberos)
  
- AS-REQ
	- AS-REP fail car absence de la pré-auth. Besoin d'utiliser la pré-auth suivante :
		- PA-ETYPE-INFO 2 => pré-auth en chiffrant le timestamp (PA-ENC-TIMESTAMP)
		- etype : eTYPE-AES256-CTS-HMAC-SHA1-96 (**18**)
		- salt : CATCORP.LOCALwilliam.dupond


- AS-REQ, avec pre-auth cette fois :
	- PA-ENC-TIMESTAMP
	- cipher : **fc8bbe22b2c967b222ed73dd7616ea71b2ae0c1b0c3688bfff7fecffdebd4054471350cb6e36d3b55ba3420be6c0210b2d978d3f51d1eb4f**

- AS-REP : on récupère le TGT de l'utilisateur

Ensuite, il utilise son TGT pour obtenir une TGS et se connecte au service SMB.


On va brute force l'élément de pré-authentification avec hashcat.

On crée le fichier 
```kerberos_pre_auth_hash.txt
$krb5pa$18$william.dupond$CATCORP.LOCAL$fc8bbe22b2c967b222ed73dd7616ea71b2ae0c1b0c3688bfff7fecffdebd4054471350cb6e36d3b55ba3420be6c0210b2d978d3f51d1eb4f
```

Puis on le crack avec la commande suivante :
```bash
hashcat -m 19900 -a 0 kerberos_pre_auth_hash.txt ../../wordlists/rockyou.txt
```