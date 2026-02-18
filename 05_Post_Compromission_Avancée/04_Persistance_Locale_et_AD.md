
# V.5 Persistance locale et impact Active Directory

Suite à l’obtention de privilèges administrateur local sur un poste ou serveur membre du domaine, une technique de persistance simple a été mise en œuvre afin d’évaluer la capacité à maintenir un accès durable au système compromis.

Même un compte compromis avec privilèges administrateur local permet de maintenir un accès durable sur un poste ou serveur, facilitant le mouvement latéral et la reprise d’accès ultérieure.
## V.5.1 Création d’un compte administrateur local (Persistance)


```bash
net user pentest Pass1234! /add
net localgroup administrators pentest /add
```

Cette persistance ne nécessite aucune vulnérabilité logicielle ; elle repose uniquement sur les privilèges administrateur locaux existants.


![](../assets/images/Pasted-image-20260109123110.png)
![](../assets/images/Pasted-image-20260109123243.png)
## V.5.2 Analyse

- Création d’un compte local supplémentaire ;
- Ajout au groupe Administrateurs ;
- Mise en place d’une porte dérobée persistante à partir d'un compte administrateur ;
- Accès maintenu même après changement du mot de passe initial.

Dans un environnement Active Directory, cette persistance locale permet à un attaquant de reprendre pied sur un hôte compromis et de relancer des attaques de mouvement latéral.

## V.5.3 Événements de sécurité générés

|Event ID|Description|
|---|---|
|4720|Création d’un compte utilisateur|
|4732|Ajout à un groupe Administrateurs|
|4688|Création de processus|
|7045|Création de service|
Ces événements sont générés automatiquement par Windows lors de modifications de comptes ou de groupes, même en l’absence d’activité malveillante. Une corrélation via un SIEM ou une analyse régulière des journaux peut permettre de détecter un comportement suspect.

## V.5.4 Recommandations

- Audit régulier des comptes locaux sur les postes et serveurs ;
- Déploiement de LAPS / Windows LAPS ;
- Restriction des droits administrateur locaux ;
- Supervision des événements 4720 / 4732 / 7045 ;
- Application du Tiering Model (séparation stricte des privilèges).

## V.5.5 Leçon clé

La persistance locale est une technique simple mais extrêmement efficace.  Sans contrôle strict des comptes administrateurs locaux, une compromission ponctuelle peut devenir un accès durable à l’infrastructure Active Directory.

## V.5.6 Facteurs favorisant la persistance locale

- Comptes locaux rarement inventoriés ;
- Faible visibilité SIEM ;
- Confiance excessive dans le changement de mots de passe AD.
