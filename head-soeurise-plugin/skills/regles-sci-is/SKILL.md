---
name: regles-sci-is
description: Regles comptables et fiscales des SCI soumises a l'IS.
  Utiliser quand l'utilisateur parle de comptabilite SCI, fiscalite,
  IS, liasse 2065/2033, cloture, pre-cloture, deficit, taux d'imposition.
---

## Regime fiscal SCI a l'IS

- La SCI soumise a l'IS est une personne morale. Elle tient une comptabilite d'engagement (PCG 2025 / reglement ANC 2022-06).
- Regime simplifie d'imposition (CA < 818 000 EUR).
- Non assujettie a la TVA (activite civile de gestion patrimoniale).

## Architecture comptable (Phase 5+ du plan)

- **Revue de fin d'exercice** : interactive, dans l'espace abonne FE
  (`/subscriber/revue/<exercice>`). Inclut alertes Phase A, questions Phase B,
  saisie cutoffs (PAR, CAP, ICNE) et evenements exceptionnels (depreciations
  SCPI/VMP, provisions, cessions, etc.).
- **Pre-cloture** : interactive, dans l'espace abonne FE
  (`/subscriber/precloture/<exercice>`). Inclut upload IFU SCPI, calcul IS,
  generation FEC + Grand Livre + Livre-journal + CERFA + PV pre-rempli +
  rapport de gestion + rapport de controles, archives dans sci_documents.
- **Cloture definitive** : interactive, dans l'espace abonne FE
  (`/subscriber/cloture/<exercice>`). Inclut interpretation PV AG, ecritures
  d'affectation du resultat, bilan d'ouverture N+1.
- **Liasse fiscale** : edition sur Teledec via canal C / Cowork (`/sci:teledec`),
  car Teledec est un site externe pilote au navigateur.
- **Teletransmission** : action manuelle du client sur Teledec (hors
  automatisation).

Pour Soeurise 2025 (cas de test, double commande avec CRP 2C) : la teletransmission
est faite par CRP 2C en parallele, et la liasse cote Soeurise est generee en
mode "preview" (`/sci:teledec 2025 preview`) sans archivage ni transmission.

## Taux IS

| Tranche | Taux |
|---------|------|
| Jusqu'a 42 500 EUR de benefice | 15% |
| Au-dela de 42 500 EUR | 25% |

Le taux reduit de 15% s'applique aux PME dont le CA < 10 MEUR et le capital est entierement libere.

## Calcul de l'IS

```
Resultat comptable (produits 7x - charges 6x)
  + Reintegrations (ligne 330) : quotes-parts fiscales SCPI
  - Deductions (ligne 350) : revenus SCPI comptabilises (solde 755)
  = Resultat fiscal brut
  - Deficit fiscal reportable (ligne 870)
  = Base imposable
  x Taux (15% puis 25%)
  = IS
```

## Deficit fiscal reportable

- Reportable sans limite de duree.
- Imputation : 1 000 000 EUR + 50% du benefice excedant ce seuil (plafonnement general).
- Distinct du deficit comptable (compte 119 RAN debiteur) : le deficit fiscal est extra-comptable.
- Stocke dans la table `donnees_fiscales_exercice` (ligne 870 du CERFA).

## Liasse fiscale

| Formulaire | Contenu |
|------------|---------|
| 2065 | Declaration de resultat IS (resultat comptable, reintegrations, deductions, IS) |
| 2033-A | Bilan simplifie (actif / passif) |
| 2033-B | Compte de resultat simplifie (charges / produits) |
| 2033-D | Provisions, amortissements, deficits |
| 2033-F | Composition du capital social |

Date limite de depot : 2e jour ouvre suivant le 1er mai (exercice cloture au 31/12).

## Ecritures IS

| Moment | Ecriture | Comptes |
|--------|----------|---------|
| Pre-cloture | Constatation IS (si > 0) | Debit 695 / Credit 444 |
| Pre-cloture | Constatation resultat | Debit 89 / Credit 120 (benefice) ou Debit 129 / Credit 89 (perte) |
| Cloture | Affectation resultat | Debit 120 / Credit 119 (absorption deficit) ou 110 (RAN crediteur) |

L'ecriture IS n'est creee que si IS > 0. En cas de resultat deficitaire ou
si le deficit reportable absorbe le resultat fiscal, pas d'ecriture IS.

## Logique d'affectation du resultat (regle PCG)

Si le compte 119 (Report a nouveau debiteur) a un solde non nul (deficit
reporte des exercices anterieurs), le benefice de l'exercice doit l'absorber
en priorite avant tout report en 110 :

- Absorption : Debit 120 / Credit 119, montant = min(benefice, |solde 119|)
- Reste eventuel : Debit 120 / Credit 110

Le PV d'AG doit refleter cette logique en formulant l'affectation comme
"au compte de report a nouveau" (formulation neutre), et NON "au compte 110"
(qui serait incorrect quand le 119 est debiteur).
