
Ce projet présente un audit de sécurité Active Directory réalisé dans un environnement Windows Server simulant un contexte d’entreprise réel. L’objectif était d’évaluer l’impact qu’une compromission d’un compte utilisateur standard peut avoir sur la sécurité globale du domaine.

L’audit a couvert l’ensemble du cycle d’attaque Active Directory : reconnaissance, énumération avancée, cartographie des relations de privilèges, compromission initiale, élévation de privilèges et post-compromission. Des outils spécialisés tels que PowerView, BloodHound, CrackMapExec et Impacket ont été utilisés afin d’identifier et d’exploiter des faiblesses de configuration.

Les résultats démontrent qu’à partir d’un simple accès utilisateur, il est possible d’escalader les privilèges jusqu’à obtenir un contrôle administratif sur des systèmes critiques du domaine. Plusieurs vulnérabilités structurelles ont été identifiées, notamment :

- Une gestion insuffisante des privilèges et des groupes AD
- Une exposition aux attaques Pass-the-Hash et NTLM relay
- Des configurations permissives facilitant la persistance

Ces constats mettent en évidence un niveau de risque élevé en cas de compromission interne. Des recommandations techniques priorisées sont proposées afin de réduire la surface d’attaque, renforcer la segmentation des privilèges et améliorer les capacités de détection et de réponse aux incidents.