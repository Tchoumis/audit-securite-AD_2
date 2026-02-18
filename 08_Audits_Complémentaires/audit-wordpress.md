# VIII Audit de sécurité - Application Web WordPress

## VIII.2.1 Contexte et objectif

L’audit a été réalisé sur l’application WordPress exposée sur le réseau interne de l’entreprise ICMAC.

**Objectifs :**

- Identifier les vulnérabilités connues de WordPress.
- Détecter les mauvaises pratiques de configuration.
- Évaluer les risques liés aux attaques par bruteforce et à la fuite d’informations.
- Proposer des mesures de mitigation adaptées pour renforcer la sécurité de l’application.
## VIII.2.2 Méthodologie et outil utilisé

**Outil principal :** WPScan (scan passif et agressif)
```bash
sudo wpscan --url http://192.168.1.45:8080/ --api-token <API_TOKEN> -e
```

![](assets/images/Pasted-image-20260109133824.png)
![](assets/images/Pasted-image-20260109133926.png)
![](assets/images/Pasted-image-20260109134030.png)
**Observations principales :**

- Le noyau WordPress est à jour (version 6.8.1).
- Aucun plugin ou thème vulnérable n'a été détecté.
- Les fichiers sensibles et les exports de base de données ne sont pas exposés.

## VIII.2.3 Analyse de la configuration WordPress

|Fonctionnalité|État|Risque|
|---|---|---|
|xmlrpc.php|Activé|Bruteforce / Pingback DoS|
|readme.html|Présent|Fuite d’informations|
|wp-cron.php|Activé|DoS potentiel|
**Analyse :**

- **`xmlrpc.php`** : Cette fonctionnalité peut être exploitée pour des attaques par force brute distribuées (attaque par relais ou par piratage de comptes). Elle peut être détectée dans les journaux du serveur web si activée.
- **`readme.html`** : Le fichier permet de connaître la version exacte de WordPress utilisée, facilitant ainsi les attaques ciblées sur des vulnérabilités spécifiques à cette version.
- **`wp-cron.php`** : Cette fonctionnalité peut être abusée pour générer une surcharge serveur (DoS) si elle est déclenchée de manière répétée ou malveillante.

## VIII.2.4 Énumération et risques utilisateurs

WPScan a identifié le compte utilisateur :

|Utilisateur|Méthode|
|---|---|
|admin|ID auteur + erreurs de connexion|
**Risque :** Le compte **`admin`** est fréquemment ciblé lors d'attaques par force brute. Ce compte est souvent laissé par défaut et exposé publiquement, ce qui facilite les tentatives d’attaque.
## VIII.2.5 Tentative de bruteforce contrôlée

Une tentative de bruteforce a été réalisée avec une faible volumétrie, sans impact sur la disponibilité du service, dans un cadre strictement contrôlé.

```bash
sudo wpscan --url http://192.168.1.45:8080/ \
--usernames user.txt \
--passwords /usr/share/wordlists/rockyou.txt
```

 **Impact potentiel en cas de succès :**

- Accès à l’interface d’administration de WordPress.
- Téléversement de webshell (accès distant au serveur).
- Compromission complète du site WordPress et accès à des informations sensibles.

## VIII.2.6 Synthèse des risques identifiés

|Risque|Gravité|
|---|---|
|Compte `admin` exposé|Élevée|
|`xmlrpc.php` activé|Moyenne|
|Fuite d’informations via `readme.html`|Faible|
|Absence de protection contre le bruteforce|Moyenne

## VIII.2.7 Recommandations

| Recommandation                                      | Justification                                              |
| --------------------------------------------------- | ---------------------------------------------------------- |
| Supprimer le fichier `readme.html`                  | Réduit la fuite d’informations sur la version de WP.       |
| Désactiver `xmlrpc.php` si inutile                  | Réduit le risque d’attaque par bruteforce ou DoS.          |
| Supprimer ou renommer l’utilisateur `admin`         | Empêche les attaques ciblées sur un compte commun.         |
| Mettre en place une protection contre le bruteforce | Utilisation de Fail2Ban ou d’un mécanisme similaire.       |
| Activer HTTPS                                       | Protection des identifiants et des données sensibles.      |
| Journaliser les connexions                          | Améliore la détection d’attaques et d’accès non autorisés. |
| Limiter les tentatives de connexion                 | Réduction des attaques automatisées.                       |
## VIII.2.8 Limites de l’audit

- Audit réalisé enboîte noire (sans authentification préalable).
- Tentatives de bruteforce contrôlées et limitées.
- Aucun audit du code source ou de la base de données de l’application.

## VIII.2.9 Évaluation globale

**Risque global :** **Moyenne**

- Le noyau WordPress est à jour, et aucun plugin ou thème vulnérable n’a été détecté.
- Présence de vecteurs classiques d’attaque (bruteforce, énumération, fuite d’informations).
- Absence de protections adéquates contre les attaques automatisées.

**Impact potentiel sur le SI :**

- Point d’entrée pour un attaquant qui pourrait exploiter la configuration de l’application pour compromettre le serveur.
- Dépôt de webshells permettant un accès persistant.
- Réutilisation des identifiants compromis pour un mouvement latéral vers Active Directory (AD) ou d’autres ressources critiques.

## VIII.2.10 Suggestions supplémentaires

- **Journalisation et détection** : Mettre en place des protections comme Fail2Ban ou une supervision proactive des connexions pour détecter les activités suspectes, en particulier les tentatives de connexion échouées répétées.

- **Vérification continue** : Réaliser des audits de sécurité réguliers, incluant des tests de bruteforce et l’analyse de nouvelles configurations ou plugins, afin de s’assurer qu’aucune nouvelle vulnérabilité n’est introduite.

**Conclusion :**  
L’application WordPress est correctement maintenue avec un noyau et des plugins à jour. Cependant, certaines configurations par défaut exposent inutilement la plateforme à des attaques classiques, telles que les attaques par bruteforce. Un durcissement simple de la configuration, y compris la suppression de fichiers inutiles et l'activation de protections contre le bruteforce, permettrait de réduire considérablement le risque.

