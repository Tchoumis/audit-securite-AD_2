
### Synthèse Audit Active Directory et Sécurité Windows

## Résumé

Ce projet présente un audit de sécurité Active Directory mené dans un environnement Windows réaliste.  
Il démontre comment une compromission utilisateur initiale peut évoluer vers une compromission étendue de l’infrastructure, puis valide l’efficacité de mesures de durcissement natives Windows appliquées en remédiation.

L’approche combine :
- attaque contrôlée (Red Team),
- analyse d’impact et de risque,
- recommandations de sécurité,
- tests de non-régression post-remédiation (Blue Team).

## Arborescence du projet
<img width="439" height="590" alt="image" src="https://github.com/user-attachments/assets/a557e1ee-7a42-44bb-ad7d-fff79c8780a3" />
---
<img width="440" height="589" alt="image" src="https://github.com/user-attachments/assets/46f3e8e5-da4e-4d1b-b54f-1e4bdef8342e" />


<img width="440" height="161" alt="image" src="https://github.com/user-attachments/assets/dd6f1e75-2e41-45e7-89ba-9d8d6125a5d4" />

00_Executive_Summary
01_Architecture_et_Lab_Setup
   ├─ architecture-overview.md
   ├─ lab-setup-active-directory
   └─ lab-setup-wordpress

02_Reconnaissance_et_Enumération
   ├─ audit-ad-02-enumeration-reseau
   ├─ audit-ad-powerview-enumeration
   ├─ audit-ad-bloodhound
   └─ 02_Enumération_AD_Services_Partages

03_Capture_et_Compromission_Initiale
   ├─ audit-ad-01-llmnr-poisoning
   ├─ audit-ad-03-post-capture-ntlm
   ├─ audit-ad-04-smb-relay.md
   ├─ audit-ad-ipv6-mitm6-ntlm-relay
   └─ audit-ad-adcs-ntlm-relay-esc8

04_Exploitation_et_Post_Compromission
   ├─ 04-extraction-secrets-et-escalade-potentielle
   ├─ audit-ad-post-compromission
   ├─ Exploitation_Active_Directory_via_Metasploit
   └─ 05_Post_Compromission_AD_CME

05_Post_Compromission_Avancée
   ├─ Impacket_psexec_et_exec_distante
   ├─ Metasploit_PsExec_Meterpreter
   ├─ Secrets_dump_et_Mimikatz
   ├─ Persistance_locale_et_AD
   └─ Limites_et_Echecs

06_Exploitation_Kerberos_et_GPO
   └─ 08_Kerberoasting_GPP

07_Chaine_d_Attaque_Complete
   └─ 05-chaine-d-attaque-active-directory

08_Audits_Complémentaires
   ├─ audit-wordpress
   └─ audit-iam-oauth-openid-passback

09_Detection_et_Monitoring
   └─ detection-monitoring

10_Mitigation_et_Hardening
   ├─ smb-mitigation
   ├─ hardening-ad
   └─ recommandations

11_Scan_Automatisé
   └─ openvas-gvm

12_Validation_Post_Remediation
   └─ validation-post-remediation

## Objectif du projet

Évaluer :
- l’impact réel d’un accès utilisateur compromis,
- les risques liés à NTLM, SMB et aux privilèges excessifs,
- l’efficacité des mesures de durcissement Active Directory

## Périmètre technique

- Active Directory (contrôleur de domaine, serveurs membres)
- Authentification NTLM / Kerberos
- SMB et partages administratifs
- Comptes utilisateurs et administrateurs
- Poste attaquant (Kali Linux)

**Outils utilisés :**

- Impacket
- Metasploit Framework
- Mimikatz / Kiwi
- BloodHound
- Responder / ntlmrelay

## Chaîne d’attaque démontrée

Compromission utilisateur  
=> Abus NTLM / SMB  
=> Accès administrateur local  
=> Exécution distante (PsExec)  
=> Extraction de secrets  
=> Persistance locale  
=> Risque de compromission domaine

## Scénarios de compromission

### 1. Exécution distante via SMB (PsExec)

- Utilisation d’identifiants valides (mot de passe ou hash NTLM)
- Prise de contrôle distante d’un serveur Windows
- Accès administrateur local sans exploitation de vulnérabilité logicielle

### 2. Post-compromission avancée

- Reconnaissance système et domaine
- Mouvement latéral
- Vol de secrets (hashs NTLM, LSA)
- Exploitation de sessions à privilèges élevés

### 3. Persistance

- Création de comptes administrateurs locaux
- Maintien d’un accès durable après compromission

## Impact sécurité observé

**Gravité : critique

Une compromission initiale permet :
- l’exécution de commandes à distance,
- la compromission complète de serveurs Windows,
- le mouvement latéral dans le domaine,
- l’accès aux identifiants sensibles,
- une compromission durable de l’infrastructure Active Directory

Ces scénarios correspondent à des attaques fréquemment observées en environnement réel.

## Mesures de durcissement mises en place

- Activation obligatoire de la signature SMB
- Restriction progressive de NTLM
- Désactivation de LLMNR / NBT-NS
- Déploiement de Credential Guard
- Renforcement de la gestion des privilèges locaux
- Application du Tiering Model (Tier 0 / 1 / 2)

## Validation post-remédiation

|Mesure|Résultat|
|---|---|
|Signature SMB|Blocage des attaques PsExec relayées|
|Restriction NTLM|Échec des attaques NTLM Relay|
|Credential Guard|Blocage de l’extraction LSASS|
|Moindre privilège|Limitation forte de la post-compromission|

Les vecteurs d’attaque initiaux ne sont plus exploitables, tandis que les accès légitimes restent fonctionnels.

## Enseignements clés

- Les attaques AD reposent majoritairement sur NTLM + privilèges excessifs
- Les mécanismes de sécurité natifs Windows sont efficaces lorsqu’ils sont correctement configurés
- Une compromission locale peut rapidement devenir globale
- La validation post-correction est indispensable dans tout audit sérieux

Ce projet démontre qu’une sécurité Active Directory efficace repose avant tout sur la configuration, la supervision et la validation continue, bien plus que sur des solutions tierces complexes.
## Positionnement professionnel

Ce projet s’inscrit dans une démarche :

- **SOC / Blue Team**
- **Audit sécurité / GRC**
- **Administration système sécurisée**
- **Analyse d’incidents Active Directory**

Il met l’accent autant sur l’attaque que sur la défense, la remédiation et la validation.

> Projet réalisé dans un laboratoire personnel à des fins pédagogiques.
