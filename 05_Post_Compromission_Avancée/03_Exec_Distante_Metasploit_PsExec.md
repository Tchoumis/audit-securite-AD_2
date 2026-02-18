# V.3 Variante : PsExec via Metasploit Framework

## V.3.1 Résultat et impact opérationnel

Obtenir une exécution de code distante sur un serveur Windows à l’aide d’identifiants disposant de privilèges administrateur local.

## V.3.2 Technique utilisée

**Module :** `exploit/windows/smb/psexec` (Metasploit)

- Authentification SMB via mot de passe ou hash NTLM ;
- Création d’un service Windows distant ;
- Exécution d’un payload Meterpreter.

Comme avec `psexec.py`, le module crée un service Windows temporaire sur la machine cible afin d’exécuter le payload. Cette action peut générer un événement 7045 (création de service – Service Control Manager) dans les journaux système Windows.

**Paramètres principaux :**

|Paramètre|Valeur|
|---|---|
|RHOSTS|192.168.1.29|
|SMBUser|jules.ferry|
|SMBPass|mot de passe / hash NTLM|
|Payload|`meterpreter/reverse_tcp`|

**Paramètres principaux utilisés**

![](../assets/images/Pasted-image-20260109130226.png)

![](../assets/images/Pasted-image-20260109130422.png)
---
![](../assets/images/Pasted-image-20260109130626.png)

**Résultat :** session Meterpreter ouverte avec des privilèges administrateur local sur le serveur cible.
### V.3.3 Vérification des privilèges obtenus

![](../assets/images/Pasted-image-20260109130822.png)

Une fois la session Meterpreter ouverte, une vérification du contexte d’exécution est réalisée :

```meterpreter
getuid
```

Résultat :
- Exécution dans le contexte d’un compte disposant de privilèges administrateur local.

Vérification complémentaire :
```bash
sysinfo
```
Confirme :
- Nom de l’hôte compromis ;
- Système Windows ciblé ;
- Accès interactif réussi.

Cette exploitation est rendue possible par :

- l’autorisation d’authentifications NTLM ;
- l’absence de restriction sur les comptes administrateurs locaux ;
- l’absence de contrôle strict des services distants créés via SMB ;
- et l’absence de supervision avancée (EDR / SIEM).

Ce scénario confirme qu’aucune vulnérabilité logicielle n’est nécessaire pour compromettre un serveur Windows lorsque des identifiants valides et des privilèges administrateur locaux sont disponibles. Il s’agit d’un abus de fonctionnalités légitimes du système (SMB et gestion des services Windows), exploité à l’aide d’identifiants valides.

L’utilisation de PsExec via Metasploit permet une exploitation rapide, automatisable et industrialisable des accès SMB. Cette technique est fréquemment observée lors d’attaques réelles de type ransomware ou intrusion Active Directory, notamment dans les phases de mouvement latéral et de déploiement massif de charges malveillantes.