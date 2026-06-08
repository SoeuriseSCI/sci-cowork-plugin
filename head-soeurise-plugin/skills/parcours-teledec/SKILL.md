---
name: parcours-teledec
description: Édition d'une liasse fiscale 2065/2033 sur teledec.fr pour une SCI à l'IS.
  Mobilisée quand l'utilisateur évoque Teledec, télédéclaration, liasse fiscale,
  déclaration impôts SCI, ou saisie de formulaires fiscaux 2065/2033/2069-RCI.
---

## Vue d'ensemble

Teledec (teledec.fr) est un service EDI agréé par la DGFiP pour la télétransmission des liasses fiscales. Il permet à une SCI à l'IS d'importer sa balance comptable, de saisir/corriger les données fiscales dans les formulaires CERFA, de générer la liasse PDF et de la transmettre à l'administration.

Ce skill couvre la procédure d'édition d'une liasse 2065/2033 sur teledec.fr depuis Cowork, en tandem avec Claude in Chrome qui pilote le navigateur. La télétransmission elle-même reste une action manuelle du gérant, hors du périmètre de ce skill.

## Activation

Quand l'utilisateur dit `teledec <année>` (ex: `teledec 2025`), la commande `/sci:teledec <année>` est lancée. Le skill couvre les termes : Teledec, télédéclaration, liasse fiscale, déclaration impôts SCI, saisie formulaires fiscaux. La commande fait l'orchestration de bout en bout — pas de question préalable de clarification, l'année est le seul argument nécessaire.

## Architecture en deux phases

### Phase 1 — Préparation des données (côté backend)

Trois prérequis backend, vérifiés via le serveur MCP `head-soeurise` :

1. **Pré-clôture effectuée** dans l'espace abonné FE (`/subscriber/precloture/<exercice>`). Statut exercice = PRE_CLOTURE, vérifié aussi par la présence du compte 695 ou 120 mouvementé. Tool MCP : `balance`.
2. **Balance Teledec fraîche** générée via `export_balance_teledec`. Le fichier doit contenir des comptes des classes 1 à 7 (bilan + gestion). Si les classes 6-7 manquent, la balance est incomplète et la suite est bloquée. Le `download_url` retourné est un token jetable valide 30 min.
3. **Données fiscales** générées via `export_donnees_fiscales`, retournant un CERFA JSON contenant : informations entreprise, réintégrations détaillées (ligne 330 — quotes-parts SCPI), déductions détaillées (ligne 350 — revenus SCPI comptabilisés), déficits antérieurs (ligne 360), composition du capital (2033-F : associés avec naissance/adresse), formulaires néant (2033-E, 2033-G, 2069-RCI).

Un récapitulatif est présenté au gérant avant de passer en Phase 2 (point de validation [PV1]).

### Phase 2 — Édition de la liasse (côté Claude in Chrome)

Saisie pilotée sur teledec.fr via 9 scripts JS générés par le tool MCP `generer_scripts_teledec`. Ordre d'exécution :

```
0. Permissions Chrome       → teledec.fr en "Always allow"
1. Import balance Excel     → génère 2033-A + 2033-B (partie comptable)
2. Corrections 2033-C       → corriger augmentations fictives immobilisations
3. Corrections 2033-D       → corriger augmentations fictives provisions + saisir déficits
4. 2033-A (validation)      → ouvrir, corriger zone 182, sauvegarder
5. 2033-B                   → réintégrations (330), déductions (350), déficits (360)
6. Néant 2033-E, 2033-G, 2069-RCI → checkbox Néant + confirmation modale
7. 2033-F                   → capital social (associés avec naissance/adresse)
8. 2065                     → ouvrir + sauvegarder pour forcer le recalcul
9. Vérification finale      → checklist exhaustive
10. Archivage PDF (mode transmission uniquement) → upload vers backend
```

L'ordre est important : les corrections 2033-C/D précèdent la validation du 2033-A pour éviter les allers-retours. Le 2033-B vient après 2033-D car il dépend des déficits.

## Mode preview vs transmission

Pour les SCI en double commande (cas Soeurise 2025 ou client historique avec EC en parallèle), le mode `preview` est invoqué via `/sci:teledec <année> preview`. Dans ce mode, la liasse est éditée sur Teledec mais l'archivage PDF (étape 10) est sauté — l'EC reste responsable de l'édition officielle et de la télétransmission côté son propre canal. Le mode par défaut est `transmission` (archivage côté Soeurise).

## Sécurité

La télétransmission est hors scope. Les boutons "Télétransmettre", "Transmettre", "Envoyer à la DGFiP" ne sont pas sollicités par l'agent. La Phase 2 s'arrête après vérification finale et archivage PDF. La télétransmission reste une action manuelle du client, déclenchée quand il sera prêt. Après télétransmission, l'accusé de réception DGFiP est archivé via le tool MCP `archiver_ar_teledec`.

Aucune génération de fichier Excel dans le navigateur (pas de SheetJS, pas de génération dynamique). La balance Excel provient toujours du backend via `export_balance_teledec`.

Aucune réutilisation d'un ancien fichier balance via `download_document` ou `documents` — un fichier frais est généré à chaque session.

## Vocabulaire

Le mot "simulation" n'est pas employé dans les échanges avec l'utilisateur. Les points de validation sont formulés : "Valides-tu ces propositions ?".

## Références opérationnelles

Le détail opérationnel de chaque sous-procédure est dans le dossier `references/`, à charger à la demande pendant la session :

- **`references/regles.md`** — Règles strictes de saisie : règle 0 (relire avant chaque session et après /compact), règle 1 (validation vs CERFA JSON), règle 2 (bouton "Sauvegarder et retour"), interdictions 1-4 (Tab, refs, Néant CAS A vs B, validation visuelle).
- **`references/pieges.md`** — 15 pièges connus de Teledec, 2 erreurs systématiques d'import (immobilisations financières 2033-C, provisions 2033-D), matrice de cohérence inter-formulaires (2033-A↔C, 2033-B↔D).
- **`references/dom-mapping.md`** — Mapping DOM complet des champs Teledec pour 2033-B, 2033-C, 2033-D, 2033-E, 2033-G, 2069-RCI, 2033-F, 2065.
- **`references/automation.md`** — Procédure détaillée étapes 0 à 11 : permissions Chrome, import balance, corrections 2033-C/D, validation 2033-A, saisie 2033-B/F, néants, recalcul 2065, archivage PDF (script JS d'upload via token jetable).
- **`references/verification.md`** — Checklist de vérification finale (statuts, corrections, données, cohérence, absence d'erreurs) et formulation du point de validation [PV2].

Ces références sont chargées au moment opportun par Claude pendant l'exécution. Après tout `/compact`, elles sont relues — le compact perd les détails fins (noms DOM, pièges, scripts).
