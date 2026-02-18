# V.6 Impact sécurité

**Gravité : critique**

Cette phase démontre qu’un attaquant disposant d’identifiants valides et de privilèges administrateur locaux peut compromettre totalement l’infrastructure Active Directory, sans exploitation de vulnérabilité logicielle.

## V.6.1 Impacts techniques

- Exécution de commandes à distance sur les serveurs ;
- Création de comptes administrateurs locaux persistants ;
- Vol de mots de passe et hashs NTLM en mémoire ou depuis SAM ;
- Mouvement latéral rapide dans le domaine ;
- Compromission globale du périmètre Active Directory.

## V.6.2 Impacts organisationnels et métiers

- Perte de confidentialité des données sensibles ;
- Risque d’interruption de service ou indisponibilité partielle ;
- Accès non autorisé aux ressources critiques ;
- Difficulté de détection sans supervision centralisée (SIEM / EDR) ;
- Non‑conformité potentielle aux exigences réglementaires et normes de sécurité (RGPD, ISO 27001).

## V.6.3 Faiblesses exploitées

|Faiblesse identifiée|Conséquence directe|
|---|---|
|Réutilisation d’identifiants|Pass-the-Hash / Pass-the-Password|
|Comptes administrateurs locaux exposés|Prise de contrôle complète des serveurs|
|Absence de Credential Guard|Vol de credentials en mémoire (LSASS)|
|NTLM autorisé sur le réseau|Exploitation PsExec / NTLM Relay|
|Mauvaise segmentation réseau|Propagation latérale rapide|

> Le niveau de risque est évalué comme critique, compte tenu de l’impact élevé et de la facilité d’exploitation à partir de simples privilèges administrateur locaux.

## V.6.4 Conclusion

Même un accès administrateur local limité permet à un attaquant de :

- Obtenir un contrôle interactif complet sur les serveurs ;
- Exécuter des commandes avec privilèges élevés ;
- Mettre en place des mécanismes de persistance locaux et durables ;
- Maintenir un accès même après la révocation des identifiants compromis.

**Leçon clé :**

Sans gouvernance stricte des privilèges locaux, supervision des actions administratives et contrôle du protocole NTLM, une compromission ponctuelle peut rapidement évoluer en accès permanent, conduisant à la compromission totale du domaine Active Directory.

> Cette phase illustre qu’un défaut de gestion des comptes administrateurs locaux constitue un risque systémique pour l’ensemble de l’infrastructure.