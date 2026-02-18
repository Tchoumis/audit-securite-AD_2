# V.7 Limites, échecs et enseignements post‑compromission

Ce chapitre présente les limites rencontrées lors des tentatives d’exploitation après la mise en place des mesures de durcissement : activation de la signature SMB, restriction NTLM, désactivation LLMNR/NBT-NS et déploiement de Credential Guard.

L’objectif est de démontrer l’efficacité des mesures de mitigation et de distinguer les accès légitimes des tentatives d’exploitation malveillantes.

## V.7.1 PsExec impossible après activation de la signature SMB

Après l’activation obligatoire de la signature SMB sur les serveurs et postes, les tentatives d’exécution distante via PsExec (Metasploit et Impacket) ont échoué.

L’accès aux partages administratifs (ADMIN$, C$) est refusé, même avec des identifiants valides de comptes administrateur locaux.

**Analyse :**  
La signature SMB empêche toute modification ou relais du flux NTLM, rendant les attaques SMB Relay inopérantes.

**Leçon sécurité :**  
La signature SMB est une mesure critique pour bloquer les attaques de mouvement latéral basées sur NTLM.

## V.7.2 Échec des attaques NTLM Relay après restriction NTLM

Suite à la restriction du protocole NTLM au niveau du domaine, les attaques de type NTLM Relay via Responder ou ntlmrelayx ne fonctionnent plus.

Les tentatives d’authentification relayée sont rejetées par le contrôleur de domaine et les serveurs membres.

**Analyse :**  
Le forçage de Kerberos et la restriction NTLM suppriment le principal vecteur d’attaque utilisé pour le relais d’authentification.

**Leçon sécurité :**  
La suppression progressive de NTLM réduit fortement les risques de compromission latérale dans Active Directory.

## V.7.3 Accès aux secrets LSASS bloqué par Credential Guard

Lorsque Credential Guard est activé, les tentatives d’extraction de secrets via Mimikatz ou Kiwi échouent ou retournent des informations incomplètes.

**Analyse :**  
Credential Guard isole LSASS dans un environnement sécurisé, empêchant l’accès direct aux identifiants en mémoire, même pour un compte administrateur local compromis.

**Leçon sécurité :**  
La protection de la mémoire est essentielle pour limiter les attaques de post‑compromission.

## V.7.4 Limitations liées à l’absence de privilèges administrateur

Les comptes utilisateurs standards ne permettent pas :

- l’exécution distante via SMB ;
- l’accès aux partages administratifs ;
- l’extraction de secrets locaux ou du domaine.

**Analyse :**  
Les attaques de post-compromission reposent fortement sur la présence de privilèges administrateur locaux ou de sessions élevées actives.

**Leçon sécurité :**  
La gestion stricte des privilèges et la séparation des rôles réduisent significativement l’impact d’une compromission initiale.

## V.7.5 Synthèse des protections validées

|Mesure de sécurité|Vecteur bloqué|
|---|---|
|Signature SMB|SMB Relay / PsExec relayé|
|Restriction NTLM|NTLM Relay / Pass-the-Hash|
|Credential Guard|Dump LSASS / Mimikatz|
|Séparation des privilèges|Post-compromission utilisateur|

> Ces protections montrent que la majorité des vecteurs exploités lors de la phase de post-compromission reposaient sur des défauts de configuration plutôt que sur des vulnérabilités logicielles.

## V.7.6 Conclusion

Les tests post-correction confirment que les mesures de durcissement bloquent efficacement les vecteurs d’attaque initialement exploités.

La distinction entre accès légitimes et tentatives d’exploitation est claire, confirmant l’efficacité des recommandations appliquées.

> Pour l’entreprise ICMAC, l’implémentation stricte des bonnes pratiques de configuration suffit à neutraliser les principaux vecteurs d’attaque post-compromission.

Cette phase démontre qu’une gouvernance correcte des comptes administrateurs locaux et la sécurisation des flux d’authentification constituent des mesures clés pour réduire le risque systémique sur l’ensemble du domaine Active Directory.