# V.4 Post‑exploitation avancée : Secrets et élévation de privilèges

Après l’obtention d’une session Meterpreter avec privilèges administrateur local, des actions de post‑exploitation ont été réalisées afin d’évaluer l’accès aux secrets d’authentification et la capacité de mouvement latéral dans l’environnement Active Directory.
## V.4.1 Post-exploitation via Meterpreter

|Extension|Objectif|
|---|---|
|stdapi|Gestion fichiers, processus et réseau|
|priv|Manipulation des privilèges locaux|
|incognito|Énumération et usurpation de tokens|
|kiwi|Extraction d’identifiants (équivalent Mimikatz, LSASS, SAM, LSA secrets)|

Le module `kiwi` permet d’accéder aux fonctionnalités de type Mimikatz directement depuis Meterpreter, notamment l’extraction de secrets en mémoire et des hashs locaux.

### Escalade de privilèges : Token Impersonation

- Énumération des tokens disponibles sur le système cible ;
- Usurpation du token `CYBER\Administrateur` ;
- Ouverture d’un shell Windows exécuté avec des privilèges élevés (`SYSTEM` / administrateur domaine si token approprié).

Cette élévation de privilèges est réalisée sans exploitation de vulnérabilité logicielle, mais via l’abus de sessions administrateur existantes et l’absence d’isolation stricte des privilèges.

**Preuve :**

![](../assets/images/Pasted-image-20260109130947.png)

```bash
whoami
CYBER\Administrateur
```

## V.4.2 Extraction des hashs locaux (SAM)

- Les hashs des mots de passe locaux ont été extraits depuis la base SAM ;
- Ces identifiants peuvent être réutilisés pour des attaques de type Pass-the-Hash et faciliter la propagation latérale dans le domaine.

![](../assets/images/Pasted-image-20260109131239.png)

## V.4.3 Impact opérationnel et risques métier

**Gravité : critique**

Cette phase démontre que même un accès limité à un compte administrateur local permet :

- Exécution de commandes à distance sur le serveur ciblé ;
- Création de comptes administrateurs persistants ;
- Vol de mots de passe et de hashs NTLM ;
- Mouvement latéral rapide dans le domaine ;
- Compromission potentielle complète du SI Active Directory.

**Faiblesses exploitées et conséquences :**

| Faiblesse identifiée                         | Conséquence directe                           |
| -------------------------------------------- | --------------------------------------------- |
| Réutilisation d’identifiants                 | Pass-the-Hash / Pass-the-Ticket               |
| Comptes administrateurs locaux non segmentés | Prise de contrôle complète des serveurs       |
| Absence de Credential Guard                  | Vol de credentials en mémoire (LSASS)         |
| NTLM autorisé sur le réseau                  | Exploitation PsExec / Pass-the-Hash facilitée |
| Segmentation réseau insuffisante             | Propagation latérale rapide                   |
## V.4.4 Conclusion

Même un accès administrateur local sur un serveur membre permet :

- L’extraction des secrets d’authentification du système ;
- L’usurpation de comptes à privilèges élevés du domaine ;
- La propagation latérale et le compromis potentiel de l’ensemble du domaine.

**Recommandations clés :**

- Activer Credential Guard et l’isolation des sessions administrateur ;
- Mettre en place LAPS pour gérer les mots de passe locaux de manière dynamique ;
- Restreindre et surveiller l’utilisation de NTLM ;
- Segmenter les comptes administrateurs et le réseau ;
- Déployer un SIEM / EDR pour détecter la création de services et l’extraction de credentials.