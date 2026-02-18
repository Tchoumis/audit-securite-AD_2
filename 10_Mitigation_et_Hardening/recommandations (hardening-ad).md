
## X.1 Découverte réseau

### X.1.1 Contexte et criticité

**Point de sécurité critique :**  
La découverte réseau constitue une surface d'attaque importante dans un environnement Active Directory. Elle doit être désactivée sur tous les serveurs Windows, sauf dans les cas où une justification métier explicite est fournie et documentée.

Laisser la découverte réseau activée expose l'infrastructure à plusieurs risques :

- Identification facile des hôtes sur le réseau
- Exposition de services internes non sécurisés
- Facilitation d'attaques telles que SMB Relay, LLMNR Poisoning, et d'autres attaques de type Man-in-the-Middle (MITM)

Ces vulnérabilités permettent aux attaquants de mieux se positionner pour des attaques internes ou d'augmenter leur surface d'attaque.

### X.1.2 Procédure recommandée (Windows Server / postes Windows)

1. Ouvrir le Panneau de configuration
2. Aller dans Réseau et Internet → Centre Réseau et partage
3. Cliquer sur Modifier les paramètres de partage avancés
4. Pour le profil réseau actif (Privé ou Public) :
    - **Désactiver la découverte réseau**
    - **Désactiver la configuration automatique des périphériques**
5. Enregistrer les modifications

> **Remarque importante :** Cette configuration doit être appliquée à tous les serveurs Windows (y compris Windows Server 2019 et versions ultérieures), à moins qu'une justification métier validée ne nécessite son activation.

### X.1.3 Conclusion de l’audit

L’audit de l’infrastructure de l’entreprise ICMAC a révélé plusieurs faiblesses critiques liées aux configurations Active Directory et SMB insuffisamment sécurisées, notamment des points d'entrée facilitant l'exploitation des vulnérabilités internes.

Les tests effectués ont démontré la possibilité des attaques suivantes :

- **LLMNR Poisoning**
- **Craquage de hashes NTLM**
- **SMB Relay**
- **Exploitation de la base SAM**

Ces attaques montrent que tout attaquant disposant d’un accès interne peut :

1. **Compromettre un serveur Windows complet**
2. **Étendre sa compromission à l’ensemble du domaine Active Directory**


**Causes principales identifiées :**

- Usage persistant de NTLM dans les authentifications
- Absence de signature SMB obligatoire, facilitant les attaques de type Relay
- Gestion permissive des privilèges, notamment sur les administrateurs locaux et de domain

### X.1.4 Recommandations clés

1. **Désactiver LLMNR / NBT-NS** :  
    Bloquer les protocoles de résolution de noms faibles et vulnérables qui facilitent les attaques de type Poisoning.
2. **Durcir SMB** :
    - Forcer l'activation des signatures SMB pour éviter les attaques de Relay.
    - Implémenter un filtrage réseau SMB pour limiter les accès entre segments réseau.
3. **Restreindre NTLM** :
    - Limiter l’utilisation de NTLM à des cas strictement nécessaires et migrer vers des méthodes d’authentification plus sécurisées, comme Kerberos.
4. **Segmenter le réseau et limiter les privilèges** :
    - Appliquer des stratégies de segmentation réseau pour réduire la propagation des attaques et restreindre l'accès aux ressources critiques.
    - Mettre en place une gestion stricte des privilèges utilisateurs et administrateurs.
5. **Appliquer ces mesures à tous les serveurs et postes critiques** :
    - Cela inclut tous les serveurs Windows, mais aussi les machines de mouvement latéral potentielles sur le réseau.

> **Impact attendu :** L'application de ces mesures permettra de réduire de manière significative le risque de compromission globale du système d'information. Un durcissement ciblé des services réseau est essentiel pour limiter les risques liés aux attaques internes.

### X.1.5 Suggestions supplémentaires

**Journalisation et détection proactive :**

- Configurer une journalisation centralisée des événements réseau critiques, notamment les événements SMB et NTLM.
- Mettre en place des alertes automatiques pour détecter rapidement les tentatives de relai SMB, Pass-the-Hash ou autres activités suspectes.

**Tests de pénétration réguliers :**

- Réaliser des tests internes périodiques pour simuler des attaques et vérifier la robustesse des configurations réseau, SMB et NTLM.
- Permettre d’identifier de nouvelles vulnérabilités ou mauvaises pratiques avant qu’elles ne soient exploitées en production.

> Ces mesures complètent le durcissement des services et renforcent la sécurité de l’infrastructure en assurant une surveillance et un contrôle continus.
### Conclusion

L'audit met en évidence des points de sécurité critiques concernant la configuration de l'infrastructure Windows, en particulier sur le réseau. La mise en place de recommandations telles que la désactivation de la découverte réseau et le durcissement des services SMB/NTLM permettra de réduire les risques de compromission liés à ces vecteurs d'attaque. Il est impératif de maintenir ces configurations et de poursuivre les tests pour identifier de nouvelles vulnérabilités potentielles.