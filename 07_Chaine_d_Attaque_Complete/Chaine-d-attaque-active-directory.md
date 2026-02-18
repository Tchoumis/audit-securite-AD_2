
# VII Synthèse de la Chaîne d’attaque Active Directory

## VII.1 Chaîne d’attaque globale observée

Compte administrateur local compromis (également administrateur de domaine)  
→ Pass‑the-Hash / Pass-the-Ticket  
→ Ouverture d’un shell SYSTEM  
→ Extraction des secrets (SAM / LSA)  
→ Réutilisation des identités et des tickets Kerberos  
→ Préparation et escalade vers des comptes Domain Admin

Cette chaîne correspond à un scénario réaliste d’attaques internes modernes, ne reposant sur aucune vulnérabilité logicielle, mais exclusivement sur des faiblesses de configuration, d’isolation des privilèges et de gouvernance des identités.

## VII.2 Lecture de la chaîne d’attaque

Cette chaîne illustre comment la compromission d’un compte administrateur local avec droits de domaine peut rapidement conduire à un contrôle étendu de l’infrastructure Active Directory.

Chaque étape repose sur :
- une mauvaise gestion des comptes et identités ;
- une absence de segmentation des privilèges (Tiering Model non appliqué) ;
- des mécanismes de défense insuffisants contre les attaques NTLM et post-compromission.

La chaîne peut être répétée sur plusieurs hôtes via la réutilisation de comptes administrateurs locaux et de domaine et la propagation latérale, augmentant la surface de compromission.

|Facteur|Conséquence|
|---|---|
|Réutilisation de comptes admin locaux et de domaine|Propagation rapide et accès étendu|
|Absence de LAPS / gestion dynamique des mots de passe|Accès persistant aux hôtes compromis|
|NTLM autorisé|Facilitation des relais et Pass-the-Hash / Pass-the-Ticket|
|Supervision insuffisante / absence de corrélation SIEM|Détection tardive et difficile des activités malveillantes|

> **Note forensic** : Les actions génèrent des événements Windows (4720, 4732, 7045 pour la création de comptes et services, 4769 pour Kerberoasting), mais ceux-ci sont rarement corrélés sans un SIEM ou un EDR centralisé

## VII.3 Lien avec les phases précédentes de l’audit

|Phase|Résultat clé|
|---|---|
|Énumération AD|Identification des cibles et services critiques|
|Post-compromission|Accès administrateur local et de domaine, mouvement latéral|
|Extraction des secrets|Vol d’identités et tickets, préparation d’escalade|
|Synthèse|Risque de compromission complète du domaine à court terme|


## VII.4 Impact global

- Compromission silencieuse et progressive de l’infrastructure
- Génération d’événements détectables mais peu corrélés (4720/4732/7045/4769)
- Risque critique pour l’intégrité et la disponibilité du domaine
- Possibilité de compromission totale de l’Active Directory
- Difficulté élevée de détection sans EDR ou supervision centralisé

## VII.5 Conclusion stratégique

Sans mesures de gouvernance et durcissement des privilèges (LAPS, restriction NTLM, Tiering Model, supervision centralisée), la compromission d’un compte administrateur local avec privilèges de domaine suffit pour initier une chaîne d’attaque complète et prendre potentiellement le contrôle total du domaine Active Directory.

Cette chaîne démontre que la sécurité Active Directory dépend avant tout de la gouvernance des identités et de la segmentation stricte des privilèges, et non uniquement de la correction des vulnérabilités logicielles.