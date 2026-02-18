# X.2 Mesures de mitigation mises en œuvre suite à l’attaque SMB/NTLM

## X.2.1 Introduction

Ce chapitre présente les mesures de durcissement mises en œuvre suite à l’exploitation SMB/NTLM identifiée lors de l’audit. L’objectif est de réduire les risques de compromission interne et de limiter le mouvement latéral dans l’infrastructure Active Directory.

Ces mesures ciblent spécifiquement :

- Attaques SMB Relay, Pass-the-Hash et NTLM Relay
- Renforcement de la sécurité des communications SMB
- Limitation des vecteurs liés aux comptes et services Windows


## X.2.2 Schéma global de mitigation

**Utilisateur compromis**  
    ↓  
**Tentative NTLM / Pass-the-Hash**
    ↓  
• **Signature SMB activée** → Bloque la propagation NTLM Relay  
• **Restriction NTLM** → Empêche la réutilisation d’identifiants  
• **LLMNR / NBT-NS désactivés** → Empêche la capture réseau de hashes

> Ce schéma sert de fil conducteur pour la mise en œuvre détaillée des mesures.

## X.2.3 Tableau des mesures SOC

|Mesure|Objectif|Impact sécurité|Criticité|Suivi / Volumétrie SIEM|Indicateurs / KPI SOC|
|---|---|---|---|---|---|
|**SMB Signing**|Intégrité et authentification SMB|Bloque NTLM Relay|Haute|Journaliser toutes les connexions SMB (succès/échec) ; alertes sur connexions SMB non signées|% de connexions SMB signées ; alertes SMB non signées par serveur|
|**LLMNR désactivé**|Empêcher capture de hash|Bloque Responder|Haute|Corréler tentatives LLMNR ; alertes sur trafic suspect|Nombre de requêtes LLMNR détectées après désactivation (devrait être proche de 0)|
|**NTLM restreint**|Forcer Kerberos|Réduit Pass-the-Hash|Haute|Monitorer tentatives NTLM bloquées ; déclencher alertes sur tentatives répétées|Tentatives NTLM bloquées par hôte/utilisateur ; pics anormaux|
|**NBT-NS désactivé**|Réduire vecteurs NTLM|Limite mouvements latéraux|Moyenne|Observer tentatives NetBIOS ; alertes sur machines non conformes|Tentatives NBT-NS détectées ; alertes sur machines avec NetBIOS encore actif|
|**Ports SMB restreints**|Limiter exposition réseau|Réduit scans et mouvements latéraux|Haute|SIEM : alertes sur ports 445/139 non autorisés ; corrélation avec segments réseau non autorisés|Tentatives de connexion SMB depuis segments interdits ; alertes firewall|
|**SMB over QUIC (Win2022+)**|SMB distant sécurisé|Protège SMB Internet|Moyenne|Monitorer trafic SMB distant via QUIC ; alertes sur anomalies ou échecs|Taux de connexions SMB via QUIC réussies/échouées ; tentatives d’accès non autorisées|
|**Journalisation proactive**|Détection rapide des attaques|Alertes sur activités suspectes|Haute|Centralisation logs SMB/NTLM ; corrélation SIEM ; alertes temps réel|Nombre d’alertes SMB/NTLM ; détection de tentatives de relai ou de hash cracking|
|**Tests internes réguliers**|Validation des mesures de mitigation|Identification de nouvelles faiblesses|Moyenne|Audit périodique, bruteforce contrôlé, simulation attaque SMB/NTLM|Rapport de vulnérabilités internes ; temps moyen de remédiation|


## X.2.4 Lien vers découverte réseau (X.1)

La désactivation de la découverte réseau complète le durcissement SMB/NTLM :

- **Pourquoi** : Un attaquant qui peut identifier les hôtes via découverte réseau peut toujours cibler SMB/NTLM malgré les restrictions.

- **Action recommandée** : appliquer les mesures X.1.2 (désactivation découverte réseau sur tous les serveurs et postes critiques) en parallèle avec X.2.

- **Effet attendu** : réduction significative des vecteurs de reconnaissance internes et limitation des attaques opportunistes.

## X.2.5 Détails techniques des mesures

### X.2.5.1 Activation de la signature SMB

- **Chemin** : secpol.msc → Stratégies locales → Options de sécurité
- **Paramètres** : Serveur = toujours activé ; Client = activé si serveur accepte
- **Impact sécurité** : Bloque NTLM Relay, renforce échanges SMB internes, protection partielle contre Responder/ntlmrelayx.

### X.2.5.2 Désactivation de LLMNR via GPO

- **Chemin** : Configuration ordinateur → Modèles d’administration → Réseau → Client DNS → Désactiver LLMNR
- **État** : Désactivé sur tout le domaine
- **Impact sécurité** : Empêche la capture de hashes NTLM via résolution de noms.

### X.2.5.3 Désactivation de NBT-NS

- **Chemin** : Carte réseau → Propriétés IPv4 → Avancé → WINS → Désactiver NetBIOS sur TCP/IP
- **Impact sécurité** : Réduit vecteurs NTLM et mouvements latéraux.

### X.2.5.4 Restriction / désactivation NTLM

- **Chemin** : Paramètres Windows → Stratégies locales → Options de sécurité → Sécurité réseau : restreindre NTLM
- **Exemple** : Deny all accounts to use NTLM
- **Impact sécurité** : Supprime NTLM Relay, réduit Pass-the-Hash, limite compromission latérale.

### X.2.5.5 Limitation des ports SMB

- Bloquer 445/139 depuis Internet
- Segmenter réseau (VLAN / sous-réseaux)
- Restreindre SMB aux hôtes de confiance
- **Impact sécurité** : Réduit attaques opportunistes et mouvements latéraux.

### X.2.5.6 SMB over QUIC (Windows Server 2022+)

- SMB encapsulé dans HTTPS via QUIC (UDP 443)
- Chiffrement et signature obligatoires
- **Impact sécurité** : Protège SMB distant contre NTLM Relay et MITM.

### X.2.5.7 Journalisation proactive et tests internes

- **Journalisation** : centralisation des logs SMB/NTLM dans le SIEM, corrélation et alertes temps réel sur tentatives suspectes

- **Tests réguliers** : audits périodiques internes simulant attaque SMB/NTLM pour valider la résistance des mesures et identifier tout vecteur non couvert