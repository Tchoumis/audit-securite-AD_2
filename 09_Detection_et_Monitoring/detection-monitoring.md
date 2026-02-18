
# IX Journalisation et surveillance de sécurité

## IX.1 Introduction

Ce chapitre décrit les mesures de journalisation et de surveillance permettant de détecter rapidement toute activité suspecte ou malveillante sur un environnement Active Directory, y compris les attaques post-compromission détaillées dans les chapitres V à VIII (Kerberoasting, GPP, mouvements latéraux, WordPress interne).

La journalisation centralisée et la corrélation des événements constituent un élément clé de la défense en profondeur, permettant de limiter l’impact d’une compromission même lorsque les mécanismes de prévention ont échoué.

## IX.2 Objectif

L’objectif principal est d’assurer la détection rapide d’activités suspectes sur :

- Les comptes utilisateurs et les administrateurs locaux ou Domain Admins
- Les contrôleurs de domaine et services critiques Active Directory
- Les postes utilisateurs et serveurs ciblés par des mouvements latéraux

La corrélation permet également d’analyser les incidents, de déclencher des alertes automatiques via un SIEM et d’identifier les patterns de compromission.

## IX.3 Journaux de sécurité essentiels

|Event ID|Description|Détection / Utilité|Criticité|
|---|---|---|---|
|**1102**|Effacement du journal de sécurité|Indicateur fort de compromission avancée ou tentative de suppression de traces|Haute|
|**4719**|Modification de la politique d’audit|Tentative de réduire la visibilité ou masquer des actions malveillantes|Haute|
|**4672**|Privilèges spéciaux assignés|Usage anormal de comptes à privilèges élevés (Administrateurs, Domain Admins)|Haute|
|**4769**|Demande de ticket Kerberos TGS|Détection Kerberoasting via volumes anormaux de TGS|Haute|
|**4738**|Modification d’un compte utilisateur|Changement de mot de passe, ajout de SPN ou préparation Kerberoasting|Haute|
|**4720**|Création de compte utilisateur|Création de comptes persistants ou backdoors|Moyenne|
|**4732**|Ajout à un groupe d’administrateurs locaux|Élévation de privilèges locale non autorisée|Moyenne|
|**4648**|Authentification explicite (autre compte)|Détection Pass-the-Hash / Pass-the-Ticket et usages anormaux de comptes|Moyenne|
|**4688**|Création de processus|Exécution d’outils suspects (psexec, mimikatz, PowerShell offensif)|Moyenne|
|**4776**|Authentification NTLM sur contrôleur de domaine|Détection d’authentifications NTLM, Pass-the-Hash, relais NTLM|Moyenne|
|**4625**|Connexion échouée|Indicateur de brute force, password spraying ou erreurs d’authentification|Moyenne|
|**4624**|Connexion réussie|Détection d’accès légitimes vs suspects (horaires inhabituels, machines non prévues)|Faible|
|**7045**|Création d’un service|Persistance ou backdoor système (souvent via psexec)|Faible|

**Corrélations clés :**

- **4769 + 4738** → détection avancée Kerberoasting
- **4720 + 4732 + 4624** → création de compte + élévation + connexion
- **4648 + 4672** depuis postes utilisateurs → usage anormal de comptes à privilèges

> **Note :** La volumétrie et le contexte sont essentiels pour limiter les faux positifs (ex. pic anormal de 4769 ou fréquence inhabituelle de 4648).

## IX.4 Mesures complémentaires

- **Audit NTLM** : journalisation de toutes les authentifications NTLM, y compris relais SMB/HTTP, pour détecter Pass-the-Hash et mouvements latéraux.

- **Déploiement de Sysmon** : visibilité avancée sur les processus, connexions réseau et création de fichiers, utile pour suivre les activités post-compromission (Kerberoasting, GPP, scripts malveillants).

- **Centralisation SIEM** : Splunk, ELK, Microsoft Sentinel pour corrélation et alertes automatiques.

- **Analyse comportementale** : identifier anomalies par rapport aux habitudes de connexion (horaires, machines, IP externes).

- **Audit régulier et vérification continue** : réaliser périodiquement des tests de bruteforce contrôlés et l’examen de nouvelles vulnérabilités ou configurations exposées.


> Ces mesures doivent être déployées de manière homogène sur tous les postes utilisateurs, serveurs et contrôleurs de domaine.

## IX.5 Synthèse des mesures de mitigation et leur impact

|Mesure de sécurité|Menace bloquée / Détection|
|---|---|
|**Signature SMB**|Bloque attaques NTLM Relay via SMB|
|**Désactivation LLMNR / NBT-NS**|Réduit capture de hashes NTLM sur réseau local|
|**Restriction de NTLM**|Bloque authentification faible et Pass-the-Hash internes|
|**Filtrage réseau SMB**|Empêche accès SMB non autorisé entre segments réseau|
|**SMB over QUIC**|Protège flux SMB sur Internet contre interception|
|**Journalisation + SIEM**|Détection et corrélation d’attaques internes et externes|
|**Sysmon + PowerShell Logging**|Détection avancée de mouvements latéraux et exécution de scripts malveillants|
|**Audit GPO et ACL sensibles**|Détecte modifications suspectes sur objets AD|
|**Détection comportementale**|Identification d’attaques inconnues ou zero-day|
|**Fail2Ban / protection brute-force**|Alerte sur tentatives de connexion échouées et réduction de la surface d’attaque|

## IX.6 Bonnes pratiques

1. **Centraliser les logs** : éviter les journaux dispersés pour ne pas perdre d’informations critiques.
2. **Corréler les événements** : par exemple, création de compte + ajout à administrateurs + connexion = alerte immédiate.
3. **Détecter les mouvements latéraux** : surveiller authentifications sur machines inhabituelles ou via outils légitimes (psexec, PowerShell).
4. **Réagir rapidement** : plan d’incident pour isoler poste compromis et bloquer comptes affectés.
5. **Audit régulier et vérification continue** : tester périodiquement la résistance aux attaques (bruteforce, tests de Kerberoasting/GPP) pour s’assurer qu’aucune nouvelle vulnérabilité n’est introduite.
6. **Lien avec vecteurs précédemment audités** : monitorer les actions liées aux comptes de service, Kerberoasting, GPP ou WordPress interne pour corréler incidents et anomalies.

> Même en l’absence de vulnérabilité exploitable, un attaquant peut progresser de manière silencieuse. Une journalisation centralisée et une corrélation efficace sont essentielles pour détecter une compromission avancée avant tout impact métier.