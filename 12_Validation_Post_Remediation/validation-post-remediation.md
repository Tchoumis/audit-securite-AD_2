# XII Vérification post-correction et tests de non-régression

Après la mise en œuvre des mesures de durcissement (activation de la signature SMB, restriction/désactivation de NTLM, désactivation de LLMNR/NBT-NS et filtrage réseau), une phase de validation a été réalisée. L’objectif était de confirmer que :

1. Les vecteurs d’attaque exploités précédemment ne sont plus fonctionnels.
2. Les accès légitimes pour l’administration et les utilisateurs autorisés restent opérationnels

## XII.1 Tests d’exploitation via Metasploit

**Objectif :**  
S’assurer que les modules d’exploitation type `psexec / SMB` ne permettent plus de compromettre les serveurs après durcissement.

**Procédure :**
```bash
msfconsole search psexec use exploit/windows/smb/psexec set RHOSTS <IP_SERVEUR> run
```

**Résultats observés :**

- Tous les essais d’exploitation échouent.
- Les accès aux partages administratifs sont refusés.

**Interprétation :**

- L’activation de la signature SMB et la restriction de NTLM bloquent efficacement les attaques SMB Relay et l’exécution de code à distance non autorisée.

## XII.2 Vérification manuelle avec smbclient

**Objectif :**  
Confirmer le comportement du service SMB après durcissement.

**Procédure :**
```bash
`smbclient -L //<IP_SERVEUR> -U "DOMAINE/utilisateur"`
```
**Résultats observés :**

- Les identifiants sont correctement reconnus.
- L’accès aux partages administratifs est refusé si l’utilisateur n’a pas les droits nécessaires.

**Interprétation :**

- Les protections SMB fonctionnent comme attendu.
- Les vecteurs d’attaque précédemment exploitables sont neutralisés.

## XII.3 Validation de l’accès distant légitime (Impacket)

**Objectif :**  
Vérifier que les accès administratifs autorisés restent fonctionnels malgré le durcissement.

**Procédure :**
```bash
impacket-psexec 'DOMAINE/utilisateur:motdepasse@<IP_SERVEUR>'
```
**Résultats observés :**

- Authentification réussie uniquement avec des comptes disposant de droits administratifs explicites.
- Accès distant possible uniquement via authentification directe (non relayée).

**Interprétation :**

- Le durcissement SMB / NTLM bloque les attaques Relay.
- Les accès légitimes sont maintenus, garantissant la continuité administrative.

## XII.4 Tests complémentaires

- **Accès aux partages réseau standards** : opérationnel pour les utilisateurs autorisés.
- **Connexion via Kerberos** : confirmée et fonctionnelle, assurant la compatibilité avec la restriction NTLM.
- **Supervision et journalisation** : toutes les tentatives d’exploitation échouées sont enregistrées, et les connexions légitimes apparaissent correctement dans les logs.

## XII.5 Conclusion de la phase de validation

Les tests post-correction confirment que :
- Les vecteurs d’attaque exploités précédemment (SMB Relay, NTLM Relay, Responder) sont neutralisés.
- Les outils d’administration distante restent utilisables dans un cadre sécurisé.
- Les mesures de durcissement sont efficaces et cohérentes.
- Le risque de compromission interne est significativement réduit.

**Schéma synthétique :**

Tentative d’attaque SMB / NTLM
            ↓
──────────────────────────────
    Mesures de durcissement  
 ────────────────────────────
• Signature SMB activée     
• NTLM restreint            
• LLMNR / NBT-NS désactivé  
──────────────────────────────
            ↓
          Résultat
            ↓
────────────────────────────────────────────
  Attaques bloquées                        Accès légitime OK 
───────────────────                      ─────────────────── 
• SMB Relay                                     • Comptes valides   
• NTLM Relay                                   • Kerberos utilisé  
• Responder                                     • Partages accessible



## XII.6 Recommandations finales

- Forcer la signature SMB sur tous les serveurs et postes du domaine.
- Restreindre puis supprimer progressivement NTLM, après validation de la compatibilité applicative.
- Favoriser Kerberos comme protocole d’authentification unique.
- Maintenir la journalisation et supervision pour détecter toute tentative d’exploitation.

> Ces mesures garantissent la sécurisation de l’infrastructure tout en maintenant les accès légitimes nécessaires à l’administration.