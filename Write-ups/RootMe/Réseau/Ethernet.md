On a les trame ethernet suivante et il faut deviner les valeurs des `?` :
```
>>> INGRESS >>>
        0x0000:  0050 569e 7bf9 0050 569e 7bfb 8100 0185  
        0x0010:  86dd 6000 0000 0040 3a40 2002 c000 0203  
        0x0020:  0000 0000 0000 0000 7331 2002 c000 0203  
        0x0030:  0000 0000 0000 0000 dead 8000 0af0 0792  
        0x0040:  0001 146d a451 0000 0000 d020 0300 0000  
        0x0050:  0000 2d4d 452e 4f52 4720 524f 4f54 2d4d  
        0x0060:  452e 4f52 4720 524f 4f54 2d4d 452e 4f52  
        0x0070:  4720 524f 4f54 2d4d 452e


>>> INGRESS >>>
        0x0000:  0050 569e 7bf7 0050 569e 7bf9 8100 0186  
        0x0010:  86dd 6000 0000 0040 3a40 2002 c000 0203  
        0x0020:  0000 0000 0000 0000 b00b 2002 c000 0203  
        0x0030:  0000 0000 0000 0000 fada 8000 0af0 0792  
        0x0040:  0001 146d a451 0000 0000 d020 0300 0000  
        0x0050:  0000 2d4d 452e 4f52 4720 524f 4f54 2d4d  
        0x0060:  452e 4f52 4720 524f 4f54 2d4d 452e 4f52  
        0x0070:  4720 524f 4f54 2d4d 452e


>>> INGRESS >>>
        0x0000:  0050 569e 7bfe 0050 569e 7bf7 8100 0186  
        0x0010:  86dd 6000 0000 0040 3a40 2002 c000 0203  
        0x0020:  0000 0000 0000 0000 7331 2002 c000 0203  
        0x0030:  0000 0000 0000 0000 b00b 8000 c760 0795  
        0x0040:  0001 906d a451 0000 0000 8fac 0b00 0000  
        0x0050:  0000 2d4d 452e 4f52 4720 524f 4f54 2d4d  
        0x0060:  452e 4f52 4720 524f 4f54 2d4d 452e 4f52  
        0x0070:  4720 524f 4f54 2d4d 452e                 
                
<<< EGRESS <<<
        0x0000:  0050 569e 7b?? 0050 569e 7b?? ???? 0186  
        0x0010:  86dd 6000 0000 0040 ??40 2002 c000 0203  
        0x0020:  0000 0000 0000 0000 ???? 2002 c000 0203  
        0x0030:  0000 0000 0000 0000 ???? ??00 09f0 0792 
        0x0040:  0001 146d a451 0000 0000 d020 0300 0000  
        0x0050:  0000 2d4d 452e 4f52 4720 524f 4f54 2d4d  
        0x0060:  452e 4f52 4720 524f 4f54 2d4d 452e 4f52  
        0x0070:  4720 524f 4f54 2d4d 452e 
```

Explication octet par octet :
```
>>> INGRESS >>>
        0x0000:  0050 569e 7bf9 0050 569e 7bfb 8100 0185
			    [Dest MAC     ][Source MAC   ][1  ][VLAN id]
        0x0010:  86dd 6000 0000 0040 3a40 2002 c000 0203
			    [IPv6][IPv6 header ][2][H][Source address .. 
        0x0020:  0000 0000 0000 0000 7331 2002 c000 0203  
		        ...                     ] [Dest adress ... 
        0x0030:  0000 0000 0000 0000 dead 8000 0af0 0792
		        ...                     ] [ICMPv6 header ...
        0x0040:  0001 146d a451 0000 0000 d020 0300 0000
		        ...                     
        0x0050:  0000 2d4d 452e 4f52 4720 524f 4f54 2d4d
		        ... ] [payload ...
        0x0060:  452e 4f52 4720 524f 4f54 2d4d 452e 4f52  
		        ...
        0x0070:  4720 524f 4f54 2d4d 452e
		        ...                      ]

1 : EtherType
2 : Next header (0x3a : IPv6-ICMP)
```


On analyse les requêtes comme suit :
- INGRESS 1 :
	- **Destination MAC** : `00:50:56:9e:7b:f9`
	- **Source MAC** : `00:50:56:9e:7b:fb`
	- **ID VLAN** : `0x0185` (VLAN ID = 389)
	- **Adresse IPv6 Source** : `2002:c000:0203::7331`
	- **Adresse IPv6 Destination** : `2002:c000:0203::dead`
	- **Type** : `0x80` (Echo Request)
	- **Payload** : `ROOT-ME.ORG`
- INGRESS 2 :
	- **Destination MAC** : `00:50:56:9e:7b:f7`
	- **Source MAC** : `00:50:56:9e:7b:f9`
	- **ID VLAN** : `0x0186` (VLAN 390)
	- **Adresse IPv6 Source** : `2002:c000:0203::b00b`
	- **Adresse IPv6 Destination** : `2002:c000:0203::fada`
	- **Type** : `0x80` (Echo Request)
	- **Payload** : `ROOT-ME.ORG`
- INGRESS 3 :
	- **Destination MAC** : `00:50:56:9e:7b:fe`
	- **Source MAC** : `00:50:56:9e:7b:f7`
	- **VLAN ID** : `0x0186` (VLAN 390)
	- **Adresse Source** : `2002:c000:0203::7331`
	- **Adresse Destination** : `2002:c000:0203::b00b`
	- **Type** : `0x80` (Echo Request)
	- **Payload** : `ROOT-ME.ORG`
- EGRESS :
	- **Destination MAC** : `00:50:56:9e:7b:??`
	- **Source MAC** : `00:50:56:9e:7b:??`
	- **VLAN ID** : `0x0186` (VLAN 390)
	- **Adresse Source** : `2002:c000:0203::????`
	- **Adresse Destination** : `2002:c000:0203::????`
	- **Type** : `0x??`
	- **Payload** : `ROOT-ME.ORG`

La requête EGRESS sera certainement un Echo Reply. De plus on sait qu'il s'agit du VLAN 390, donc on regarde en particulier les requêtes INGRESS 2 et 3.

Si on teste de créer une réponse au ping de la requête INGRESS 2, ça fonctionne.
(Les solutions proposées utilisent `text2pcap` pour ouvrir les requêtes avec Wireshark et ça permet de se rendre compte que seule INGRESS 2 a un checksum valide.)

Réponse :
- f9
- f7
- 8100 (utilisation d'un tag VLAN)
- 3a (IPv6-ICMP)
- fada
- b00b
- 81 (Echo Reply)
