## III.2 Exploitation post‑capture : Craquage des hashes NTLM

Après la capture de hashes NTLMv2 via une attaque de type LLMNR Poisoning, une phase
de craquage hors ligne a été réalisée afin de tenter de récupérer les mots de passe en clair.
### III.2.1 Préparation de l’environnement d’attaque

Les outils nécessaires ont été installés sur la machine d’audit (Kali Linux) :
```bash
sudo apt install hashcat wordlists gedit
```
Un fichier contenant le hash capturé a ensuite été créé :
```bash
nano ntlmhash.txt
```

Cette approche permet de travailler entièrement hors ligne, sans générer de trafic réseau  
supplémentaire détectable par les systèmes cibles.

### III.2.2 Craquage des hashes avec Hashcat

L’outil Hashcat a été utilisé afin de tenter de casser les mots de passe associés aux hashes  
NTLMv2 capturés.

Une attaque par dictionnaire a été menée à l’aide de la wordlist rockyou.txt, largement  
utilisée lors des audits de sécurité.
```bash
hashcat -m 5600 ntlmhash.txt /usr/share/wordlists/rockyou.txt
```

Cette étape permet de démontrer l’impact direct de l’utilisation de mots de passe faibles  
dans un environnement Active Directory.

Le succès du craquage dépend principalement de :
- la complexité du mot de passe ;
- la présence de mots de passe communs ou réutilisés.

Le hash NTLMv2 intègre un challenge serveur, ce qui impose un craquage hors ligne plutôt qu’une simple conversion directe..

Dans un contexte d’audit interne, cette phase permet de mesurer l’impact réel des politiques de mots de passe et des protocoles legacy activés.
## III.2.3 Impact de la vulnérabilité

L’exploitation de la vulnérabilité LLMNR Poisoning, combinée à des mots de passe faibles,  
peut entraîner des conséquences critiques pour la sécurité du domaine Active Directory.

| Impact                  | Description                                                        |
|------------------------|--------------------------------------------------------------------|
| Vol d’identifiants      | Capture et craquage hors ligne de hashes NTLMv2                    |
| Élévation de privilèges | Accès à des comptes à privilèges élevés après réutilisation        |
| Mouvement latéral       | Attaques de type Pass‑the‑Hash / réutilisation d’identifiants      |
| Compromission AD        | Prise de contrôle partielle ou totale du domaine Active Directory  |

Cette vulnérabilité augmente significativement le risque de compromission progressive du domaine Active Directory.

### III.2.3.1 Correspondance MITRE ATT&CK

Cette phase d’exploitation post-capture s’inscrit dans les tactiques et techniques suivantes du framework MITRE ATT&CK :

| Tactique MITRE ATT&CK | Technique               | ID        | Description |
|----------------------|-------------------------|-----------|-------------|
| Credential Access     | Brute Force             | T1110    | Craquage hors ligne de hashes NTLMv2 |
| Credential Access     | OS Credential Dumping   | T1003    | Exploitation d’identifiants Windows obtenus |
| Lateral Movement      | Pass-the-Hash           | T1550.002 | Réutilisation des hashs NTLM pour accès distant |


**Position dans la chaîne d’attaque :**  
Credential Access → Lateral Movement → Privilege Escalation

## III.2.4 Recommandations de sécurité

Afin de réduire significativement les risques liés à cette vulnérabilité, les mesures suivantes sont recommandées :
#### Désactivation des protocoles legacy
- Désactiver LLMNR via GPO :  
    `Computer Configuration => Administrative Templates => Network => DNS Client => Turn off Multicast Name Resolution`
- Désactiver NBT-NS sur l’ensemble des postes Windows
- Forcer l’utilisation exclusive d’un serveur DNS centralisé et fiable

#### Renforcement des mécanismes d’authentification
- Activer la signature SMB
- Restreindre ou désactiver NTLM lorsque possible
- Appliquer des politiques de mots de passe robustes (longueur minimale élevée, complexité, interdiction de mots de passe communs)..

#### Détection et segmentation
- Déployer des solutions de détection (EDR, IDS/IPS) capables d’identifier les attaques de type Responder
- Segmenter le réseau interne afin de limiter la portée des attaques de spoofing et de mouvement latéral
### III.2.5 Limites de l’attaque

- Les mots de passe robustes résistent au craquage par dictionnaire
- NTLMv2 n’est pas réversible sans faiblesse de mot de passe
- Le succès dépend fortement des politiques de sécurité du domaine

### III.2.6 Conclusion de la phase

Cette phase démontre qu’un simple accès réseau interne, combiné à des mécanismes legacy activés et à des mots de passe faibles, peut conduire à une compromission active du domaine Active Directory.

Les identifiants obtenus peuvent ensuite être exploités pour l’énumération Active Directory avancée (PowerView) et l’analyse des relations de privilèges (BloodHound), menant potentiellement à une compromission complète du domaine.


