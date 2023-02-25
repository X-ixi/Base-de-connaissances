
Glossaire :
- **IDS** : Intrusion Detection System - génère une alerte en cas d'intrusion
- **IPS** : Intrusion Prevention System - bloque l'intrusion
- **HIDS** : Host IDS - supervision au niveau d'une machine (Wazuh)
- **NIDS** : Network IDS -  supervision au niveau du réseau (Suricata)
- **SIEM** : Security Information and Event Management


Les IDS lèvent de nombreuses alertes qui ne sont pas toutes pertinentes car il y a beaucoup de faux positifs, il y a des tentatives échouées et surtout une seule intrusion génèrent plusieurs alertes.
On ajoute à ça qu'un SI est souvent supervisé par plusieurs IDS et de plusieurs types (HIDS et NIDS), et c'est impossible de traiter toutes ces alertes à la main.

D'où l'intérêt du SIEM qui va se charger du traitement des alertes :
1. Normalisation
2. Enrichissement et vérification
3. Agrégation
4. Identification de scénarios
5. Priorisation et visualisation


Outil de SIEM open source : suite **ELK** (Elastic search, Log stash et Kibana) qui se sert dans un puit de logs, et à laquelle on peut ajouter Suricata (SELKS)