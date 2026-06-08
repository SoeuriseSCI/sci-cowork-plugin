---
name: workflow-cloture
description: Sequence complete de cloture d'exercice pour une SCI a l'IS,
  de la revue de fin d'exercice a la teletransmission. Utiliser quand
  l'utilisateur parle de cloture, pre-cloture, revue, cutoffs, liasse
  fiscale, affectation du resultat, bilan d'ouverture, fin d'exercice.
---

## Regle fondamentale : humain dans la boucle

**Toute mise a jour de la base comptable passe par des propositions qui doivent etre validees.**
Ne JAMAIS valider une proposition sans accord explicite de l'utilisateur.

Concretement : toujours afficher le resultat/recapitulatif a l'utilisateur et demander
"Valides-tu ces propositions ?" avant de proceder a la validation.

Ne JAMAIS utiliser le mot "simulation". L'etape avec `execute: false` est un **calcul**,
l'etape avec `execute: true` est une **validation**.

## Vue d'ensemble — 6 etapes du parcours de cloture

La cloture d'un exercice se deroule en 6 etapes sequentielles, reparties entre
le canal A (espace abonne FE, interactif) et le canal C (Cowork, pilote
navigateur pour Teledec) :

```
1. Revue de fin d'exercice           [Canal A : /subscriber/revue/<ex>]
   |
   v
2. Pre-cloture (avec IFU SCPI)        [Canal A : /subscriber/precloture/<ex>]
   |
   v
3. Generation liasse fiscale          [Canal C : /sci:teledec <ex>]
   |
   v
4. Cloture definitive (PV AG)         [Canal A : /subscriber/cloture/<ex>]
   |
   v
5. Teletransmission via Teledec       [Canal C : /sci:teledec <ex> + action manuelle client]
   |
   v
6. Archivage AR DGFiP                 [Canal C : /sci:teledec ou tool MCP `archiver_ar_teledec`]
```

**Important** : les etapes 1, 2 et 4 sont **interactives** (decisions metier
en temps reel). Elles se font dans l'espace abonne FE (canal A), pas dans
Cowork. Cowork sert uniquement aux etapes 3, 5, 6 qui pilotent le navigateur
sur le site Teledec.

## Etape 1 : Revue de fin d'exercice (canal A)

URL : `https://app.soeuri.se/subscriber/revue/<exercice>`

Contenu :
- **Phase A — alertes automatiques** : detection des charges/produits
  recurrents avec frequence anormale ou montant atypique. Le gerant repond
  ou ignore chaque alerte.
- **Phase B — questions au gerant** : sollicitation explicite sur les sujets
  recurrents (litiges, depreciations, evenements post-cloture, conventions
  reglementees, etc.).
- **Cutoffs** : saisie des charges et produits a rattacher a l'exercice :
  - Produits a recevoir (PAR) : D 4181 / C 755 (ex: distribution SCPI T4)
  - Charges a payer / FNP : D 6226 / C 4081 (ex: honoraires expert-comptable)
  - Interets courus non echus (ICNE) : D 6611 / C 1688
- **Evenements exceptionnels** : depreciations SCPI/VMP/participations,
  provisions, cessions, ajustements fiscaux, etc.
- **Validation toutes ecritures** : un clic genere les ecritures au 31/12/N
  ET les extournes au 01/01/N+1 (uniquement pour les vrais cutoffs ;
  depreciations/provisions ne s'extournent pas).
- **Finalisation** : statut revue passe en FINALISEE, exercice toujours
  OUVERT en attente de pre-cloture.

Une fois la revue finalisee, passer a l'etape 2.

## Etape 2 : Pre-cloture (canal A)

URL : `https://app.soeuri.se/subscriber/precloture/<exercice>`

Prerequisites : revue FINALISEE, releve fiscal SCPI (IFU) disponible.

Contenu :
- Upload du releve fiscal SCPI (PDF) et extraction de la quote-part fiscale
- Calcul du resultat comptable (produits 7x - charges 6x)
- Calcul de l'IS (reintegrations L330, deductions L350, deficit L870)
- Insertion ecriture IS (si IS > 0) : D 695 / C 444
- Constatation du resultat : D 89 / C 120 (benefice) ou D 129 / C 89 (perte)
- Generation et archivage automatique dans `sci_documents` :
  - FEC (Fichier des Ecritures Comptables)
  - Grand Livre
  - Livre-journal
  - CERFA JSON (donnees pour la liasse fiscale)
  - PV d'AG pre-rempli (.docx)
  - Rapport de gestion (.docx)
  - Rapport de controles automatiques (.json)
- Statut exercice passe a PRE_CLOTURE
- Creation de l'exercice N+1 en EN_PREPARATION

Une fois la pre-cloture effectuee, passer a l'etape 3.

## Etape 3 : Generation liasse fiscale (canal C — Cowork)

Commande : `/sci:teledec <exercice>` (mode `transmission`) ou
`/sci:teledec <exercice> preview` (mode `preview` pour double commande).

Prerequisites : exercice en PRE_CLOTURE.

Contenu (cf. skill `parcours-teledec` pour le detail) :
- Recuperation des donnees fiscales depuis le CERFA archive
- Pilotage de Teledec via Claude in Chrome
- Saisie / verification des formulaires : 2033-A, 2033-B, 2033-C, 2033-D,
  2033-F, 2033-G, 2033-E, 2069-RCI, 2065
- Archivage de la liasse PDF dans `sci_documents` (mode `transmission`)
  ou pas d'archivage (mode `preview`)

Cette etape **ne fait pas la teletransmission** — elle prepare seulement
la liasse pour validation client.

## Etape 4 : Cloture definitive (canal A)

URL : `https://app.soeuri.se/subscriber/cloture/<exercice>`

Prerequisites : liasse generee (etape 3), PV d'AG signe (le PV pre-rempli
genere a l'etape 2 sert de base, le client le complete avec date, heure,
signatures).

Contenu :
- Saisie/upload du PV d'AG signe
- Interpretation par Claude des decisions d'affectation
- Calcul de l'affectation du resultat
- Validation des ecritures :
  - Si benefice + 119 debiteur : D 120 / C 119 (absorption), puis D 120 / C 110 si reste
  - Si benefice sans 119 debiteur : D 120 / C 110
  - Si perte : D 119 / C 120
- Bilan d'ouverture N+1 (reprise des comptes de bilan via compte 89)
- Report des extournes sur N+1
- Statut exercice passe a CLOTURE (plus aucune modification possible)
- Statut exercice N+1 passe a OUVERT

## Etape 5 : Teletransmission (canal C — Cowork)

Commande : `/sci:teledec <exercice>` (apres cloture) ou action manuelle
du client sur Teledec.

Prerequisites : exercice CLOTURE, liasse complete sur Teledec.

L'action de teletransmission est **toujours manuelle** par le client (jamais
automatisee par Claude). Cowork peut neanmoins :
- Verifier que la liasse est complete sur Teledec
- Rappeler le client de teletransmettre
- Une fois teletransmis, archiver l'AR DGFiP (etape 6)

## Etape 6 : Archivage AR DGFiP

Apres teletransmission, l'AR (Accuse de Reception) DGFiP est telecharge
depuis Teledec et archive via le tool MCP `archiver_ar_teledec` ou la
commande `/sci:teledec`.

## Cas particuliers

### Double commande (Soeurise 2025 et historiques)

Quand un EC externe (ex: CRP 2C) etablit la liasse en parallele :
- Etapes 1, 2, 4 : faites cote nous pour valider notre moteur
- Etape 3 : faite en mode `preview` (`/sci:teledec <ex> preview`)
  → liasse generee sur Teledec mais NON archivee chez nous
- Etapes 5, 6 : sautees (l'EC se charge de la teletransmission via son canal)

### Resultat deficitaire

- Pas d'ecriture IS (compte 695 absent)
- Resultat constate : D 129 / C 89 (compte 129 pour perte)
- Affectation : D 119 / C 129 (perte reportee en RAN debiteur)
- Le deficit fiscal reportable est suivi separement dans
  `donnees_fiscales_exercice` (ligne 870 du CERFA), reportable sans limite
  de duree.

## Affectation du resultat — regle PCG

Le compte 120 (resultat) doit etre solde a zero. Si le compte 119 (RAN
debiteur, deficit reporte des exercices anterieurs) a un solde non nul,
le benefice de l'exercice doit l'**absorber en priorite** avant tout
report en 110 :

| Situation | Ecritures |
|-----------|-----------|
| Benefice + deficit anterieur (119 debiteur) | D 120 / C 119 (absorption), puis D 120 / C 110 (excedent eventuel) |
| Benefice sans deficit anterieur | D 120 / C 110 |
| Perte | D 119 / C 129 (perte reportee, augmentation du RAN debiteur) |

Le PV d'AG doit refleter cette logique en formulant l'affectation comme
"au compte de report a nouveau" (formulation neutre), et NON "au compte 110"
(qui serait incorrect quand le 119 est debiteur).

## Bilan d'ouverture N+1

Tous les comptes de bilan (classes 1 a 5) sont repris via le compte 89 :
- Comptes a solde crediteur : D 89 / C xxx
- Comptes a solde debiteur : D xxx / C 89
- Le compte 89 se solde a zero (contrepartie universelle)

Les comptes de charges (6x) et produits (7x) ne sont PAS repris : ils
repartent a zero pour le nouvel exercice.

Les extournes des cutoffs (passes au 01/01/N+1 lors de la validation des
ecritures de revue) figurent deja dans l'exercice N+1 et seront soldes
quand les vrais flux interviendront.
