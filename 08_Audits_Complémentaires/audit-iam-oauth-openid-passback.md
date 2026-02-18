
# VIII.1 Vulnérabilité : Détournement de redirection OAuth (Passback du flux d’authentification)

Cette vulnérabilité, parfois confondue avec une simple Open Redirect, est plus critique car elle permet l’interception directe des artefacts d’authentification (code OAuth, token, assertion SAML) sans nécessiter de vol de mot de passe.

## VIII.1.1 Description

Le détournement de redirection OAuth exploite un défaut de validation des redirections dans des systèmes d’authentification fédérée tels que :

- OAuth 2.0
- OpenID Connect
- SAML

Un attaquant peut forcer la redirection de la réponse d’authentification (code, token, assertion) vers un domaine sous son contrôle, permettant l’usurpation d’identité et l’accès non autorisé aux ressources utilisateur.

## VIII.1.2 Principe de fonctionnement

1. L’utilisateur s’authentifie correctement auprès du fournisseur d’identité.
2. La réponse d’authentification est transmise via une redirection HTTP.
3. L’application cliente ne valide pas correctement :
    - `redirect_uri`
    - Paramètre `state` (OAuth) ou `relayState` (SAML)
4. L’attaquant détourne la redirection vers un domaine malveillant et récupère les artefacts d’authentification.

## VIII.1.3 Scénario typique (OAuth)

1. L’utilisateur initie une connexion via OAuth/OpenID.
2. Le fournisseur authentifie l’utilisateur.
3. Une redirection HTTP transmet le code ou le token.
4. Une validation insuffisante permet à l’attaquant de forcer la redirection vers une URL malveillante.

**Conséquences :**

- Interception du code ou token OAuth
- Usurpation de session
- Accès non autorisé aux ressources de l’utilisateur

## VIII.1.4 Causes fréquentes

- Validation insuffisante du `redirect_uri`
- Redirections dynamiques non sécurisées
- Paramètre `state` absent ou mal vérifié
- Autorisation de domaines externes non maîtrisés
- Combinaison possible avec XSS, CSRF ou attaques MITM

## VIII.1.5 Gravité et impact

|Critère|Évaluation|
|---|---|
|Impact|Élevé à critique|
|Exploitabilité|Moyenne|
|Discrétion|Élevée|
|Détection|Faible sans journalisation|

**Impact potentiel :**

- Vol de tokens OAuth/OpenID
- Usurpation d’identité et de session
- Accès non autorisé à des API et données sensibles
- Contournement de l’authentification fort

## VIII.1.6 Périmètre concerné

- Applications web utilisant OAuth, OpenID Connect ou SAML
- Services tiers d’authentification et portails SSO exposés
- Tout flux d’authentification fédérée mal configuré

## VIII.1.7 Indicateurs de compromission (IoC)

- Connexions inhabituelles depuis des IP externes
- Sessions actives sans authentification récente
- Utilisation de tokens depuis des domaines externes
- Paramètre `state` absent ou incohérent
- Réutilisation d’un `authorization_code` non prévu

## VIII.1.8 Recommandations de sécurité

### VIII.1.8.1 Validation stricte des redirections

- Valider systématiquement le `redirect_uri`
- Implémenter une liste blanche de domaines autorisés
- Interdire toute redirection dynamique non validée

### VIII.1.8.2 Sécurisation des flux OAuth/OpenID

- Utiliser systématiquement le paramètre `state` et vérifier sa cohérence
- Limiter la durée de vie des tokens (`access_token` / `refresh_token`)
- Associer les tokens à un client précis (`client_id`, `audience`)
- Pour les clients publics : implémenter PKCE (Proof Key for Code Exchange)

### VIII.1.8.3 Protection des communications

- Forcer HTTPS sur tous les flux d’authentification
- Activer HSTS
- Chiffrer les tokens au repos et en transit

### VIII.1.8.4 Supervision et journalisation

- Journaliser toutes les redirections OAuth, erreurs de validation et tokens émis
- Détecter et alerter sur :
    - Redirections vers des domaines non autorisés
    - Utilisation de tokens depuis des IP inhabituelles

> **Bilan** : La sécurisation des flux OAuth/OpenID/SAML et une supervision active sont indispensables pour prévenir le vol de tokens et la compromission d’identités.

## VIII.1.9 Évaluation du risque

|Élément|Niveau|
|---|---|
|Confidentialité|Élevée|
|Intégrité|Élevée|
|Disponibilité|Faible|
|Risque global|Élevé|

## VIII.1.10 Conclusion

Le détournement de redirection OAuth (Passback Attack) constitue une menace sérieuse dans les environnements utilisant l’authentification fédérée.

Même sans exploitation technique avancée, une mauvaise configuration des redirections peut permettre la compromission complète d’un compte utilisateur.

**Mesures critiques à appliquer :**

- Validation stricte des `redirect_uri`
- Implémentation systématique du paramètre `state`
- Limitation et journalisation des tokens
- Usage exclusif de HTTPS/HSTS

## VIII.1.11 Mapping MITRE ATT&CK

|Tactique|Technique|ID|Description|
|---|---|---|---|
|Credential Access|PKCE (Proof Key for Code Exchange) pour les clients publics|T1557|Détournement de flux d’authentification|
|Initial Access|Valid Accounts|T1078|Usurpation d’identité OAuth|