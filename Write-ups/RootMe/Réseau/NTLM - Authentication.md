
On analyse le fichier pcap avec Wireshark et on identifie les informations suivantes :
Domain name: catcorp.local
User name: john.doe

On identifie également les requêtes suivantes :
- SMB Session setup request : NTLMSSP_NEGOTIATE
- SMB Session setup response : NTLMSSP_CHALLENGE
	- NTLM Server Challenge: 1944952f5b845db1

- SMB Session setup request
	- NTLMv2 Response: 5c336c6b69fd2cf7b64eb0bde310216201010000000000001a9790044b63da0175304c546c6f34320000000002000e0043004100540043004f005200500001000800440043003000310004001a0063006100740063006f00720070002e006c006f00630061006c000300240044004300300031002e0063006100740063006f00720070002e006c006f00630061006c0005001a0063006100740063006f00720070002e006c006f00630061006c00070008001a9790044b63da010900120063006900660073002f0044004300300031000000000000000000
	- NTProofStr: 5c336c6b69fd2cf7b64eb0bde3102162
	- -> Net-NTLMv2 hash
	- NTLMv2 Client Challenge: 75304c546c6f3432


Attention à ne pas confondre le hash NTLM stocké dans la mémoire d'une machine et le hash Net-NTLM qui transite sur le réseau pour prouver que l'utilisateur connait son mot de passe.
-> Il ne se crack pas de la même façon.


Le format attendu est par hashcat est :
Username::Realm:ServerChallenge:Net-NTLMv2:NTMLv2Response(without hash)

On crée le fichier suivant :
```ntlm_hash.txt
john.doe::catcorp.local:1944952f5b845db1:5c336c6b69fd2cf7b64eb0bde3102162:01010000000000001a9790044b63da0175304c546c6f34320000000002000e0043004100540043004f005200500001000800440043003000310004001a0063006100740063006f00720070002e006c006f00630061006c000300240044004300300031002e0063006100740063006f00720070002e006c006f00630061006c0005001a0063006100740063006f00720070002e006c006f00630061006c00070008001a9790044b63da010900120063006900660073002f0044004300300031000000000000000000
```

Puis on le crack comme suit :
```bash
hashcat -m 5600 -a 0 ntlm_hash.txt ../../wordlists/rockyou.txt
```

