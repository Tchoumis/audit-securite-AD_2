
# Active Directory : Post-compromission et durcissement

## 1. Contexte général et objectifs

Ce projet s’inscrit dans un audit de sécurité Active Directory en contexte post-compromission, visant à évaluer l’impact réel d’un attaquant interne disposant d’un accès initial limité (poste utilisateur standard).

Contrairement à une approche basée sur la recherche de vulnérabilités logicielles (CVE), l’audit s’est concentré sur des attaques réalistes, exploitant des faiblesses de configuration, des mécanismes hérités et des pratiques d’administration à risque.

Les scénarios étudiés incluent notamment :

- SMB Relay
- NTLM Relay
- Pass-the-Hash
- Capture de credentials via LLMNR / NBT-NS

**Objectifs du projet :**

- Reproduire une chaîne d’attaque réaliste en environnement Active Directory
- Évaluer l’impact métier et sécurité d’une compromission interne
- Déployer des mesures de durcissement ciblées et pragmatiques
- Valider leur efficacité par des tests de non-régression
- Garantir la continuité des accès légitimes
- Intégrer une logique SOC / détection via la journalisation

## 2. Périmètre et environnement audité

- Domaine Active Directory (Windows Server 2019)
- Postes clients Windows
- Serveur Linux exposé (WordPress)
- Réseau interne sans segmentation stricte initiale

**Outils utilisés :**

- Metasploit, Impacket (psexec)
- Responder
- smbclient
- OpenVAS / GVM
- Analyse des journaux Windows (approche SIEM théorique)

## 3. Synthèse des risques identifiés

Les travaux ont mis en évidence que :

- Une compromission initiale limitée permettait un mouvement latéral rapide
- L’absence de signature SMB permettait des attaques Relay sans exploit
- La permissivité de NTLM facilitait le Pass-the-Hash
- Les mécanismes LLMNR / NBT-NS exposaient des credentials NTLM
- Les risques provenaient majoritairement de configurations par défaut et de choix historiques, non d’un défaut logiciel

### Impacts potentiels

- Compromission de comptes à privilèges
- Accès aux partages administratifs (C$, ADMIN$)
- Exécution de commandes à distance
- Prise de contrôle progressive du domaine Active Directory
- Capacité de persistance et de rebond interne

**Conclusion:**  
Le risque principal réside dans la gouvernance AD et les mécanismes d’authentification hérités, plus que dans les vulnérabilités logicielles.


## 4. Kill Chain de l’attaque (avant durcissement)

Accès utilisateur interne
        ↓
Reconnaissance réseau (T1018)
        ↓
LLMNR / NBT-NS Poisoning (T1557.001)
        ↓
Capture hash NTLM (T1003)
        ↓
NTLM Relay vers SMB (T1557 / T1021.002)
        ↓
Accès aux partages administratifs
        ↓
Exécution distante PsExec (T1569.002)
        ↓
Élévation de privilèges
        ↓
Compromission Active Directory


**Aucune CVE exploitée**  
**Uniquement des faiblesses de configuration + sessions admin actives**

## 5. Mesures de durcissement mises en œuvre

Les mesures ont été choisies pour casser la chaîne d’attaque à plusieurs niveaux, sans dégrader l’exploitation :

- Activation de la signature SMB
- Restriction progressive puis désactivation de NTLM
- Désactivation de LLMNR et NBT-NS
- Filtrage et segmentation des ports SMB
- Renforcement de la journalisation et supervision

Ces actions s’inscrivent dans une stratégie progressive, compatible avec des environnements existants.

## 6. Vérification post-correction et non-régression

Des tests de validation ont été réalisés afin de confirmer :
- Le blocage des vecteurs d’attaque
- Le maintien des accès légitimes

### Résultats clés

- Les modules Metasploit / Impacket échouent systématiquement
- Les accès non autorisés aux partages administratifs sont refusés
- Les accès administratifs légitimes restent fonctionnels
- Les événements de sécurité sont correctement journalisés

**Les attaques SMB / NTLM Relay sont neutralisées sans rupture opérationnelle**
## 7. Conclusion générale

L’audit confirme que les attaques Active Directory les plus critiques reposent rarement sur des vulnérabilités logicielles, mais sur des mauvaises configurations et mécanismes hérités.

Les mesures mises en œuvre permettent :
- De réduire significativement le risque de compromission interne
- De bloquer les attaques Relay et Pass-the-Hash
- De renforcer la posture globale de sécurité sans dégrader l’exploitation
- D’améliorer la détection et la supervision

Ce projet illustre une démarche complète, opérationnelle et réaliste, combinant attaque, défense, validation et détection.  
Il constitue une base solide pour une stratégie de sécurisation Active Directory à long terme, orientée SOC.

#  Synthèse

| Vulnérabilité / Technique | Impact                  | Criticité | MITRE ATT&CK | Mesure clé       |
| ------------------------- | ----------------------- | --------- | ------------ | ---------------- |
| SMB Relay                 | Compromission serveur   | Haute     | T1557.001    | SMB Signing      |
| NTLM Relay                | Élévation de privilèges | Haute     | T1557        | Restriction NTLM |
| Pass-the-Hash             | Mouvement latéral       | Haute     | T1550.002    | Forcer Kerberos  |
| LLMNR / NBT-NS            | Capture de hashes       | Haute     | T1040        | Désactivation    |
| Ports SMB exposés         | Attaques opportunistes  | Haute     | T1021.002    | Filtrage réseau  |
| Logs non surveillés       | Attaque indétectable    | Critique  | T1070        | SIEM et alerting |

