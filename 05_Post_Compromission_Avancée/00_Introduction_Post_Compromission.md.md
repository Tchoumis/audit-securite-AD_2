# V Post-Compromission Avancée Active Directory

Cette phase s’inscrit dans la continuité du chapitre IV et approfondit les capacités d’un attaquant disposant déjà d’un accès administrateur local.

## V.1 Contexte

À la suite des techniques de mouvement latéral décrites au chapitre IV, un accès administrateur local a été obtenu sur un serveur membre du domaine.

Plusieurs actions de post-exploitation ont alors été menées afin d’évaluer l’impact réel de cet accès dans un environnement Active Directory.

Cette phase vise à démontrer comment un accès administrateur local peut être exploité pour atteindre un niveau de contrôle étendu sur le système, voire préparer une compromission du domaine via l’accès aux secrets d’authentification ou la compromission de comptes à privilèges élevés.

## V.2 Axes d’évaluation de la phase

Cette phase évalue notamment la capacité d’un attaquant à :
- Obtenir une exécution de code distante sur des serveurs Windows ;
- Élever les privilèges jusqu’au contexte SYSTEM et préparer une escalade vers des comptes à privilèges élevés du domaine (ex. Domain Admin) ;
- Extraire les secrets de sécurité (hashs, secrets LSA) ;
- Mettre en place des mécanismes de persistance ;
- Évaluer les limites après application de mesures de durcissement.

## V.3 Périmètre

- Serveurs Windows membres du domaine ;
- Comptes du domaine disposant de privilèges administrateur local ;
- Services SMB et authentification NTLM/Kerberos.

## V.4 Structure de la phase post-compromission

Les actions réalisées sont détaillées dans les chapitres suivants :

- **Impacket – PsExec et exécution distante**
- **PsExec via Metasploit – Session Meterpreter**
- **Extraction de secrets (SAM / LSASS / Mimikatz)**
- **Persistance locale et Active Directory**
- **Limites, échecs et enseignements post-durcissement**

Chaque sous-chapitre présente les techniques utilisées, leur impact sécurité et les recommandations associées.
