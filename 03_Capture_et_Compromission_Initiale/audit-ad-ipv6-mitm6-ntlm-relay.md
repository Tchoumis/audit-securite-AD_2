# III.5 Vulnérabilité : Attaque IPv6 - MITM6 et NTLM Relay via LDAPS

Cette section présente une vulnérabilité avancée liée à l’activation par défaut d’IPv6 dans les environnements Active Directory, combinée à l’usage persistant de NTLM et à l’absence de mécanismes de signature obligatoires.  

Cette vulnérabilité montre qu’un attaquant interne peut utiliser IPv6 activé par défaut et NTLM autorisé pour relayer des authentifications et compromettre durablement Active Directory.

Contrairement à la section III.4, qui détaille un scénario d’exploitation ciblant spécifiquement Active Directory Certificate Services (ESC8), cette section se concentre sur la vulnérabilité structurelle liée à IPv6 et NTLM, qui constitue le socle technique rendant possibles ces attaques avancées.

## III.5.1 Description de la vulnérabilité

L’attaque MITM6 exploite une faiblesse structurelle fréquente dans les environnements Active Directory :
- IPv6/DHCPv6 activé par défaut sur les postes Windows

Dans ce contexte, un attaquant interne peut :
- Se positionner en Adversary-in-the-Middle (AitM) sur le réseau local
- Se faire passer pour un serveur IPv6/DNS malveillant
- Forcer les postes Windows à utiliser ce serveur pour la résolution de noms et la configuration WPAD
- Déclencher automatiquement des authentifications NTLM
- Relayer ces authentifications vers des services critiques tels que :
    - **LDAPS**
    - **SMB**
    - **HTTP**

L’attaque peut se dérouler sans interaction utilisateur et passer inaperçue pour les mécanismes de sécurité classiques.

## III.5.2 Conditions rendant l’attaque critique

L’attaque MITM6 devient hautement critique lorsque les conditions suivantes sont réunies :
- IPv6 activé par défaut sur les postes Windows
- NTLM autorisé dans le domaine
- Pré-requis réseau : IPv6 activé
- Protocoles vulnérables : NTLM autorisé, absence de SMB/LDAP signing, absence de Channel Binding
- Services sensibles : AD CS installé

Ces conditions permettent d’exploiter un scénario avancé de NTLM Relay vers LDAPS et AD CS, compromettant le domaine.

**Environnement audité :**

|Élément|Valeur|
|---|---|
|Domaine|cyber.lan|
|Contrôleur de domaine|192.168.1.29|
|OS|Windows Server 2019|
|IPv6|Activé par défaut|
|NTLM|Autorisé|
|AD CS|Installé (contexte de test)|

## III.5.3 Chaîne d’attaque

1. **Empoisonnement IPv6 via MITM6**
    - Répond aux requêtes DHCPv6 des postes Windows
    - Se déclare serveur DNS IPv6 prioritaire
    - Fournit une configuration WPAD malveillante
    - Déclenche automatiquement les authentifications NTLM
2. **Relais NTLM vers LDAPS avec ntlmrelayx**

```bash
python3 ntlmrelayx.py -6 -t ldaps://192.168.1.29 -wh fakewpad.cyber.lan -l lootme
```

    - Capture les authentifications NTLM automatiques
    - Relaye ces identifiants vers LDAPS
    - Interaction avec Active Directory pour lecture/modification d’objets

L’utilisation de LDAPS ne protège pas contre le relais NTLM en l’absence de LDAP Signing et de Channel Binding, ce qui rend cette attaque particulièrement dangereuse dans les environnements supposés sécurisés.

3. **Exploitation d’AD CS (ESC8)**
    - Obtention de certificats abusifs pour comptes ciblés
    - Authentification Kerberos légitime possible sans mot de passe
    - Persistance silencieuse sur le domaine

4. **Mouvement latéral et escalade**
    - Création/modification d’objets AD
    - Déplacement vers d’autres serveurs vulnérables
    - Escalade de privilèges jusqu’à Domain Admin

[ Attaquant – Kali Linux ]
        |
        | 1. MITM6 (DHCPv6 / DNS IPv6 / WPAD)
        v
[ Poste Windows du domaine ]
        |
        | 2. Authentification NTLM automatique
        v
[ NTLM capturé ]
        |
        | 3. NTLM Relay (ntlmrelayx)
        v
[ LDAPS / AD CS ]
        |
        | 4. Interaction Active Directory
        |    - Lecture / écriture objets AD
        |    - Demande de certificats
        v
[ Certificat AD CS frauduleux ]
        |
        | 5. Authentification Kerberos légitime
        v
[ Accès persistant au domaine AD ]


Ce scénario démontre qu’un simple accès réseau interne, combiné à des mécanismes
activés par défaut (IPv6, NTLM, WPAD), peut conduire à une compromission
persistante et silencieuse du domaine Active Directory.
## III.5.4 Impact de sécurité

| Impact                 | Description                                                                               |
| ---------------------- | ----------------------------------------------------------------------------------------- |
| Compromission AD       | Prise de contrôle potentielle du domaine                                                  |
| Vol d’identifiants     | Capture et relais NTLM                                                                    |
| Escalade de privilèges | Accès administrateur, possibilité de Domain Admin                                         |
| Mouvement latéral      | Exploitation d’autres serveurs via NTLM relay ou SMB                                      |
| Discrétion             | L’attaque peut se dérouler sans déclencher d’alerte, ce qui la rend difficile à détecter. |

**Niveau de criticité : critique**
## III.5.5 Recommandations de sécurité

Mesures prioritaires (immédiates) :
- Désactiver WPAD
- Forcer la signature LDAP et SMB
- Restreindre NTLM

Mesures structurelles :
- Désactiver IPv6 si non utilisé
- Durcir AD CS (audit des templates, restrictions d’enrôlement)
- Déployer supervision NTLM / LDAP / DNS

## III.5.7 Installation et exécution de MITM6

Découper en points clairs :

- MITM6 répond aux requêtes DHCPv6
- Se présente comme serveur DNS IPv6 prioritaire
- Fournit WPAD malveillant
- Déclenche les authentifications NTLM

**Exemple de commande :**

```bash
sudo mitm6 -d cyber.lan
```

Objectif : forcer les clients Windows du domaine à utiliser un serveur DNS / WPAD contrôlé par l’attaquant, sans aucune interaction de l’utilisateur.

Ce scénario prépare directement l’abus des services Active Directory Certificate Services (AD CS), détaillé dans la section suivante, en fournissant un vecteur d’authentification NTLM relayable vers les services d’annuaire.

## III.5.8 Conclusion globale de la Chaîne d’attaque Active Directory 

Les sections **III.2 à III.5** démontrent une chaîne d’attaque complète, progressive et réaliste contre une infrastructure Active Directory, reposant non pas sur des vulnérabilités logicielles, mais sur l’accumulation de configurations par défaut insuffisamment durcies.

L’audit met en évidence qu’une compromission initiale de faible niveau, obtenue via la capture de hashes NTLM (LLMNR / Responder), suffit à initier une escalade progressive conduisant à la prise de contrôle du domaine Active Directory.

La phase **III.2** illustre comment des politiques de mots de passe faibles et l’usage de protocoles legacy permettent la récupération d’identifiants exploitables hors ligne, servant de point d’entrée à des attaques de mouvement latéral.

La section **III.3** démontre ensuite l’impact critique du NTLM Relay, lorsque la signature SMB n’est pas imposée, permettant à un attaquant interne de s’authentifier sur des serveurs sans connaître de mot de passe et d’obtenir des privilèges administrateur locaux, voire d’extraire des hashes persistants (SAM).

Les sections **III.4 et III.5** révèlent un scénario d’attaque avancé et particulièrement critique, reposant sur l’activation par défaut d’IPv6, l’absence de mécanismes de protection (LDAP Signing, Channel Binding) et une configuration permissive d’Active Directory Certificate Services (AD CS).  
Cette combinaison permet une compromission silencieuse et persistante, via l’émission de certificats frauduleux (ESC8), contournant totalement les mécanismes traditionnels de sécurité basés sur les mots de passe.

L’ensemble de ces attaques s’inscrit dans une logique offensive cohérente, démontrant qu’un attaquant interne, disposant d’un simple accès réseau, peut :

- contourner les mécanismes d’authentification classiques ;
- escalader progressivement ses privilèges ;
- établir une persistance durable et difficilement détectable ;
- compromettre l’intégrité globale du domaine Active Directory.

Ce scénario souligne l’importance critique :
- du durcissement des protocoles hérités (NTLM, LLMNR, WPAD) ;
- de la sécurisation des services d’annuaire (SMB, LDAP, AD CS) ;
- et d’une supervision avancée des authentifications et des certificats

## III.5.9 Tableau de synthèse exécutif des Vulnérabilités Active Directory

| ID     | Vulnérabilité                | Description synthétique                                              | Impact principal                                  | Criticité    | MITRE ATT&CK                                                   |
| ------ | ---------------------------- | -------------------------------------------------------------------- | ------------------------------------------------- | ------------ | -------------------------------------------------------------- |
| III.2  | Craquage de hashes NTLMv2    | Capture et craquage hors ligne de hashes NTLM via LLMNR/Responder    | Vol d’identifiants, accès initial                 | **élevée**   | T1110 – Brute Force  <br>T1003 – OS Credential Dumping         |
| III.3  | SMB Relay                    | Relais NTLM vers SMB sans signature obligatoire                      | Accès administrateur, dump SAM, mouvement latéral | **critique** | T1557 – Adversary‑in‑the‑Middle  <br>T1021.002 – SMB           |
| III.4  | NTLM Relay vers AD CS (ESC8) | Relais NTLM permettant l’émission de certificats frauduleux          | Persistance AD, compromission silencieuse         | **critique** | T1557.001 – NTLM Relay  <br>T1649 – Abuse of Auth Certificates |
| III.5  | MITM6 + NTLM Relay via LDAPS | Exploitation d’IPv6 pour déclencher et relayer NTLM vers LDAPS/AD CS | Compromission complète du domaine                 | **critique** | T1557 – AitM  <br>T1078 – Valid Accounts                       |
| Global | Chaîne d’attaque AD          | Combinaison progressive des vulnérabilités précédentes               | Contrôle total du domaine Active Directory        | **critique** | T1068 – Privilege Escalation  <br>T1021 – Lateral Movement     |
