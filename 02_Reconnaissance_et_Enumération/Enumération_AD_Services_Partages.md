# II.1 Énumération des services et partages Active Directory

Cette section correspond à la phase d’énumération de l’infrastructure Active Directory, étape clé de tout audit de sécurité interne.  
L’objectif est d’identifier les services exposés, les ports ouverts et les partages réseau accessibles afin d’évaluer la surface d’attaque réelle du domaine.  
Cette phase permet de déterminer dans quelle mesure un utilisateur authentifié standard peut collecter des informations sensibles et préparer des attaques post‑compromission telles que Kerberoasting, Pass-the-Hash ou l’exploitation de mauvaises configurations SMB.
## II.1.1 Ports ouverts et services détectés

Lors de la phase d’énumération réseau, plusieurs ports critiques ont été identifiés comme ouverts sur la machine cible 192.168.1.30, indiquant la présence d’un serveur Windows Active Directory.

![](assets/images/Pasted-image-20260109132541.png)

#### Résumé des ports détectés

|Port|Service|Description|Risque potentiel|
|---|---|---|---|
|53|DNS|Résolution de noms|Enumération AD, reconnaissance interne|
|80|HTTP|Serveur web|Vulnérabilités applicatives|
|88|Kerberos|Authentification AD|Kerberoasting|
|135|MSRPC|Services Windows distants|Mouvement latéral|
|139|NetBIOS|SMBv1|Obsolète, vulnérable si activé|
|389|LDAP|Annuaire AD|Enumération utilisateurs / groupes|
|445|SMB|Partages Windows|Vol de données, Pass‑the‑Hash|
|464|kpasswd|Changement MDP Kerberos|Attaques ciblant Kerberos|
|593|RPC over HTTP|Services distants|Exposition inutile|
|636|LDAPS|LDAP sécurisé|Requêtes AD chiffrées|
|3268|Global Catalog|AD multi‑domaines|Enumération avancée|
|3269|GC sécurisé|Global Catalog via SSL|Accès aux métadonnées AD|
|5985|WinRM|Exécution distante|Exécution de commandes à distance|
**Observation :** Les services exposés (LDAP, SMB, Kerberos, WinRM, Global Catalog) permettent l’énumération du domaine et constituent des vecteurs pour Kerberoasting, Pass-the-Hash, exploitation de GPP et mouvement latéral. (MITRE ATT&CK : Discovery, Credential Access, Lateral Movement)
## II.1.2 Analyse

La présence combinée de LDAP, Kerberos, SMB, WinRM et Global Catalog confirme que la machine cible est un contrôleur de domaine Active Directory.

Cette exposition de services critiques ouvre de nombreux vecteurs d’attaque classiques, notamment :
- Kerberoasting
- Pass‑the‑Hash
- Exploitation de GPP
- Exécution distante via PsExec ou WinRM

## II.1.3 Impact sécurité

- augmentation significative de la surface d’attaque Active Directory
- exposition de services critiques accessibles aux utilisateurs authentifiés
- préparation facilitée des attaques post‑compromission (Kerberoasting, PtH, GPP, exécution distante)

**Synthèse des risques**

| Élément       | Type              | Niveau de risque | Justification                                   | Tactique MITRE ATT&CK                |
| ------------- | ----------------- | ---------------- | ----------------------------------------------- | ------------------------------------ |
| DNS (53)      | Service           | Moyen            | Enumération interne possible                    | Discovery                            |
| LDAP (389)    | Service           | Élevé            | Permet de récupérer comptes / groupes AD        | Discovery / Credential Access        |
| SMB (445)     | Service / Partage | Élevé            | Pass-the-Hash, vol de fichiers                  | Credential Access / Lateral Movement |
| WinRM (5985)  | Service           | Élevé            | Exécution distante sur postes/serveurs          | Execution / Lateral Movement         |
| SYSVOL        | Partage           | Très élevé       | Contient GPO, scripts, mots de passe potentiels | Persistence / Lateral Movement       |
| NETLOGON      | Partage           | Élevé            | Scripts de connexion accessibles                | Persistence                          |
| cyberpartages | Partage           | Moyen à élevé    | Accès complet à un utilisateur standard         | Discovery / Lateral Movement         |

## II.1.4 Accès SMB avec un compte valide

Un test de connexion SMB a été réalisé à l’aide d’un compte utilisateur valide du domaine pour énumérer les partages accessibles.
```bash
smbclient -L //192.168.1.30/ -U nom_utilisateur_windows
```
### II.1.4.1 Partages détectés

**Constat :**
- Les partages **SYSVOL** et **NETLOGON** sont accessibles à tous les utilisateurs authentifiés.
- Ils peuvent contenir : scripts de connexion, fichiers de configuration, mots de passe en clair (GPP).

**Impact MITRE :** vecteurs possibles pour _Discovery_, _Lateral Movement_ et _Credential Access_. 

### II.1.4.2 Accès au partage réseau personnalisé _cyberpartages_

#### Contexte

Un test d’accès au partage réseau cyberpartages a été réalisé à l’aide d’un compte utilisateur standard du domaine, sans privilèges particuliers.

```bash
smbclient //192.168.1.30/cyberpartages -U cyber.lan\\jules.ferry
```


![](assets/images/Pasted-image-20260212135827.png)
_Figure 2 : Connexion réussie au partage cyberpartages avec un compte standard._

#### Résultat observé

- Connexion SMB réussie sans élévation de privilèges
- Accès complet au contenu du partage
- Téléchargement possible pour analyse locale

#### Risques identifiés

- Accès excessif pour les utilisateurs standards
- Fuite potentielle de données sensibles (documents internes, scripts, configurations)
- Stockage possible de mots de passe, scripts d’administration, fichiers oubliés
- Facilitation de la reconnaissance interne et du mouvement latéral

_Contrôle insuffisant des permissions = point d’appui pour exfiltration, escalade de privilèges et mouvement latéral_ (Discovery, Credential Access, Lateral Movement)

#### Impact sécurité

- fuite potentielle de données internes
- exposition de secrets exploitables pour une élévation de privilèges
- point d’appui fréquent pour des attaques Active Directory avancées

### II.1.5 Recommandations

- Appliquer le principe du moindre privilège (lecture seule / accès restreint)
- Auditer régulièrement les ACL des partages SMB
- Séparer les partages utilisateurs et administratifs
- Interdire le stockage de mots de passe ou de scripts sensibles
- Surveiller les accès SMB anormaux (Event ID 5140, 5145)

**Niveau de risque : élevé, pouvant devenir critique en cas d’exposition de secrets.

**Lien MITRE :** **Lien MITRE :** Limiter la surface d’attaque des services et partages SMB (Discovery, Lateral Movement, Credential Access).

| Action recommandée                             | Tactique MITRE                       |
| ---------------------------------------------- | ------------------------------------ |
| Appliquer principe du moindre privilège        | Discovery / Lateral Movement         |
| Auditer ACL des partages SMB                   | Credential Access / Lateral Movement |
| Séparer partages utilisateurs / administratifs | Discovery / Lateral Movement         |
| Interdire stockage mots de passe ou scripts    | Credential Access / Persistence      |
| Surveiller accès SMB anormaux                  | Discovery / Lateral Movement         |
### II.1.6 Conclusion de la phase d’énumération

Les informations collectées démontrent que l’infrastructure Active Directory expose de nombreux services accessibles aux utilisateurs authentifiés.

Même un compte standard peut exploiter ces services et partages mal sécurisés pour attaquer des comptes sensibles, facilitant Kerberoasting, Pass-the-Hash et mouvement latéral.

Cette phase d’énumération prépare plusieurs tactiques du framework MITRE ATT&CK : _Discovery_, _Credential Access_ et _Lateral Movement_.
