---
description: Préparer les données fiscales Teledec et éditer la liasse sur teledec.fr.
---

## Vocabulaire

Le mot "simulation" est évité dans les échanges utilisateur. Les points de validation sont formulés : "Valides-tu ces propositions ?".

## Périmètre

Cette commande couvre la préparation des données fiscales (Phase 1, côté backend) et l'édition de la liasse fiscale 2065/2033 sur teledec.fr (Phase 2, côté Claude in Chrome). La télétransmission est hors périmètre — elle reste une action manuelle du gérant, déclenchée quand il sera prêt.

Les boutons "Télétransmettre", "Transmettre", "Envoyer à la DGFiP" ne sont pas sollicités par cette commande.

## Mode preview vs transmission

Si la SCI est en double commande (cas Soeurise 2025 ou client historique avec EC en parallèle), le mode `preview` est utilisé : la liasse est générée et éditée sur Teledec mais l'archivage côté Soeurise est sauté. L'EC s'occupe de l'édition officielle et de la télétransmission côté son propre canal.

Invocation : `/sci:teledec 2025 preview` (ou `mode=preview`). En mode preview, l'étape 10 (archivage PDF) est sautée. Mode par défaut : `transmission`.

## Posture de réalisation

Aucun bricolage : le fichier Excel est toujours généré par le backend via `export_balance_teledec`, jamais dans le navigateur. Aucune valeur calculée (résultat comptable, etc.) n'est saisie manuellement. Si une étape échoue, la commande s'arrête et remonte le problème plutôt que de chercher un contournement.

Après tout `/compact`, le skill `parcours-teledec` et ses références (`references/*.md`) sont relus avant de continuer la saisie. Le compact perd les détails fins (noms DOM, pièges, procédures d'archivage). Les sessions précédentes ont montré qu'une procédure improvisée double le temps d'exécution et cause des erreurs. Les scripts exacts qui fonctionnent sont dans le skill et ses références.

## Workflow

### Phase 1 — Préparation des données

1. **Vérifier la pré-clôture** :
   - Tool MCP `balance` avec l'exercice demandé
   - Statut attendu : PRE_CLOTURE (compte 695 présent ou à défaut compte 120 mouvementé — résultat constaté)
   - Si la pré-clôture n'est pas faite : informer l'utilisateur qu'elle se fait dans l'espace abonné sur `https://head-soeurise-web.onrender.com/subscriber/precloture/<exercice>` (la commande `/sci:precloture` n'existe plus depuis Phase 5 du plan, le workflow interactif est en FE).

2. **Générer la balance Teledec fraîche** :
   - Tool MCP `export_balance_teledec` avec l'exercice
   - Pas de réutilisation d'un ancien fichier (éviter `download_document` / `documents` pour la balance)
   - Vérifier que la balance contient des comptes classes 6 ET 7 (pas juste bilan) via le tool `balance` — comptes 6xx et 7xx attendus
   - Si classes 6-7 absentes : la balance est incomplète, la commande s'arrête
   - Le `download_url` retourné est un token jetable valide 30 min
   - Confirmer à l'utilisateur que la balance est générée

3. **Générer les données fiscales** :
   - Tool MCP `export_donnees_fiscales` avec l'exercice
   - Le serveur retourne un CERFA JSON contenant : informations entreprise (SIREN, dénomination, adresse), réintégrations détaillées (ligne 330 — quotes-parts SCPI), déductions détaillées (ligne 350 — revenus SCPI comptabilisés), déficits antérieurs (ligne 360), composition du capital (2033-F : associés avec naissance/adresse), formulaires néant (2033-E, 2033-G, 2069-RCI)

4. **Présenter le récapitulatif** :
   - Nombre de comptes dans la balance (classes 1-7 présentes)
   - Résultat fiscal final
   - IS à payer
   - Nombre de réintégrations / déductions
   - Liste des associés
   - Formulaires néant

5. **Vérifier le statut de transmission** :
   - Tool MCP `statut_transmission_teledec` avec l'exercice
   - Afficher le statut courant

### [PV1] Point de validation

Demander : "Les données sont prêtes. Valides-tu pour procéder à la saisie sur Teledec ?"

Si l'utilisateur refuse : la commande s'arrête et indique les corrections nécessaires.

### Phase 2 — Édition de la liasse (Claude in Chrome)

Si l'utilisateur valide [PV1] :

6. **Vérifier les permissions Chrome pour teledec.fr** :
   - Demander à l'utilisateur : "teledec.fr est-il dans vos sites approuvés dans Claude in Chrome (Settings > Permissions > Always allow) ?"
   - Si non : expliquer comment ajouter le site (voir `skills/parcours-teledec/references/automation.md`, étape 0)
   - Sans cette config, chaque action demandera une autorisation, ce qui rend l'automatisation inutilisable

7. **Vérifier les prérequis Claude in Chrome** :
   - Claude in Chrome actif
   - Client connecté à son compte Teledec dans Chrome
   - Si un prérequis manque : guider le client

8. **Suivre la procédure du skill `parcours-teledec`** (voir `references/automation.md`) :
   - Utiliser le `download_url` de l'étape 2 pour l'import (token jetable, pas de header Authorization)
   - Après import : vérifier que 2033-A ET 2033-B sont remplis. Si 2033-B est vide, l'import est incomplet et la commande s'arrête
   - Ordre : Import → Corrections 2033-C/D → Validation 2033-A (+ zone 182) → 2033-B → Néant → 2033-F → 2065
   - Règles de saisie strictes : voir `references/regles.md` (Tab interdit dans textarea, "Sauvegarder et retour" obligatoire, etc.)
   - Pièges spécifiques à Teledec : voir `references/pieges.md`
   - Mapping DOM des champs : voir `references/dom-mapping.md`

9. **À la fin de la saisie, présenter le [PV2]** :
   - Liste des formulaires avec leur statut
   - Récap des valeurs saisies
   - Résultat de la checklist de vérification finale (`references/verification.md`)
   - Demander : "La liasse est complète sur Teledec. Valides-tu ces saisies ?"

10. **Archivage automatique** (mode `transmission`, sauté en mode `preview`) :
    - En mode `preview` : indiquer "Mode preview : liasse générée sur Teledec mais non archivée. L'EC s'occupe de l'édition officielle et de la télétransmission."
    - En mode `transmission` (défaut) :
      * Tool MCP `generer_upload_url_liasse` pour obtenir une URL d'upload jetable
      * Identifier l'URL de téléchargement PDF sur Teledec (lien "Télécharger la liasse")
      * Exécuter le script JS d'archivage (voir `references/automation.md`, étape 11c)
      * Le script télécharge le PDF depuis Teledec et l'envoie au backend via l'URL jetable
      * Si échec : la commande s'arrête et remonte l'erreur (la Phase 2 n'est pas terminée sans archivage réussi)

### Fin

Indiquer que :
- La liasse est prête sur Teledec
- La liasse PDF a été archivée dans l'espace abonné (mode transmission)
- La télétransmission doit être effectuée manuellement par le client quand il sera prêt (hors scope automatisation)
- Après télétransmission, l'accusé de réception DGFiP est archivé via le tool MCP `archiver_ar_teledec`

## Notes

- Les données fiscales proviennent du CERFA JSON archivé lors de la pré-clôture (jamais recalculées).
- Si le CERFA JSON n'existe pas, `export_donnees_fiscales` retourne une erreur 412 : la pré-clôture est un préalable.
- Le fichier Excel est au format standard attendu par Teledec (import balance).
- Les données personnelles des associés (naissance, adresse) proviennent du KBIS.
- Si des informations manquent (naissance, adresse), elles sont saisies manuellement sur Teledec.
