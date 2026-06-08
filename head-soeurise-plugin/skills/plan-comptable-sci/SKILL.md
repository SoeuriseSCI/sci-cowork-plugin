---
name: plan-comptable-sci
description: Plan comptable specifique aux SCI soumises a l'IS (V1.5, PCG 2025).
  Utiliser quand l'utilisateur parle de comptes, ecritures, bilan, balance,
  plan comptable, comptes de cutoff, ou demande quel compte utiliser.
---

## Hypotheses structurantes

SCI a l'IS, non soumises a la TVA, sans personnel, flux bancaires uniquement (virements et prelevements SEPA).
4 types d'activites : S (SCPI), V (valeurs mobilieres), I (immobilier direct), P (participations strategiques).

## Comptes de capitaux (classe 1) — 10 comptes tronc commun + 1 activite I

| Compte | Libelle | Activite | Usage |
|--------|---------|:--------:|-------|
| 101 | Capital social | T | Capital initial de la SCI |
| 1061 | Reserve legale | T | Si prevue par les statuts (non obligatoire en SCI) |
| 1068 | Autres reserves | T | Reserves facultatives, statutaires |
| 110 | Report a nouveau (crediteur) | T | Benefices accumules non distribues |
| 119 | Report a nouveau (debiteur) | T | Deficits accumules |
| 120 | Resultat de l'exercice (benefice) | T | Cree en pre-cloture, solde a zero en cloture |
| 129 | Resultat de l'exercice (perte) | T | Distinct de 120 car lignes CERFA differentes (DI vs DL) |
| 151 | Provisions pour risques et charges | T | Litiges, contentieux, sinistres. Constitution via 687, reprise via 787 |
| 164 | Emprunts | T | Emprunts immobiliers (capital restant du) |
| 165 | Depots et cautionnements recus | I | Depots de garantie des locataires (immobilier direct) |
| 1688 | Interets courus | T | Cutoff : interets courus non echus au 31/12 |

## Comptes d'immobilisations (classe 2) — 11 comptes activites

| Compte | Libelle | Activite | Usage |
|--------|---------|:--------:|-------|
| 211 | Terrains | I | Immobilier direct (non amortissable) |
| 2131 | Batiments | I | Immobilier direct (amortissable par composants) |
| 2135 | Installations, agencements | I | Immobilier direct (duree distincte du bati) |
| 261 | Titres de participation | P | Detention strategique (> 10 % ou controle) |
| 271 | Titres immobilises | S V | Parts de SCPI et valeurs mobilieres a long terme |
| 274 | Prets | P | Prets aux filiales ou associes |
| 275 | Depots et cautionnements verses | I | Depots verses (ex : compteur EDF) |
| 28131 | Amortissements des batiments | I | Cumul des amortissements |
| 28135 | Amortissements des installations | I | Cumul des amortissements |
| 296 | Depreciations titres de participation | P | Test de depreciation annuel |
| 297 | Depreciations autres immob. financieres | S V | Depreciation parts SCPI et VM long terme |

## Comptes de tiers (classe 4) — 9 comptes tronc commun + 5 activites

| Compte | Libelle | Activite | Usage |
|--------|---------|:--------:|-------|
| 404 | Fournisseurs d'immobilisations | I | Dettes pour acquisitions immobilieres (acte notarie, travaux immobilisables) |
| 411 | Locataires | I | Loyers factures non encaisses (immobilier direct) |
| 416 | Clients douteux | I | Creances locataires douteuses (reclassement depuis 411) |
| 4081 | Fournisseurs - factures non parvenues | T | Cutoff : charge engagee, facture pas encore recue |
| 4181 | Produits a recevoir | T | Cutoff : revenus acquis non encaisses (SCPI T4, dividendes) |
| 4423 | Retenues et prelevements a la source | T | PFU sur dividendes distribues (12,8 % IR + 17,2 % PS). Declaration 2777 |
| 444 | Etat - IS | T | IS a payer (cree en pre-cloture) |
| 4486 | Etat - charges a payer | T | Autres dettes fiscales (si applicable) |
| 455 | Comptes courants d'associes | T | Avances des associes a la SCI |
| 457 | Associes - dividendes a payer | T | Dividendes decides par l'AG, pas encore verses |
| 462 | Creances sur cessions d'immobilisations | T | Prix de cession a recevoir quand paiement differe |
| 486 | Charges constatees d'avance | T | Cutoff : charges payees concernant l'exercice suivant |
| 487 | Produits constates d'avance | I | Cutoff : produits encaisses concernant l'exercice suivant |
| 491 | Provisions pour depreciation clients | I | Depreciation creances locataires douteuses (416). Via 6817/7817 |

## Comptes financiers (classe 5) — 1 tronc commun + 2 activites

| Compte | Libelle | Activite | Usage |
|--------|---------|:--------:|-------|
| 503 | Valeurs mobilieres de placement | V | Actions, ETF (court terme, avec ISIN) |
| 512 | Banque | T | Compte courant bancaire |
| 590 | Depreciations des VMP | V | Si valeur de marche < cout d'acquisition |

## Comptes de charges (classe 6) — 9 comptes tronc commun + 11 activites

| Compte | Libelle | Activite | Usage |
|--------|---------|:--------:|-------|
| 606 | Fournitures non stockables | I | Eau, energie (immobilier direct) |
| 614 | Charges locatives et de copropriete | I | Charges non recuperables (immobilier direct) |
| 615 | Entretien et reparations | I | Travaux courants non immobilisables |
| 616 | Primes d'assurance | T | PNO, assurance emprunteur |
| 6226 | Honoraires | T | EC, notaire, avocat, gestionnaire, _Head.Soeurise |
| 6227 | Frais sur titres | S V P | Droits de garde, courtage. Frais d'acquisition immobilises (PCG 2005) |
| 626 | Frais postaux et telecommunications | I | Immobilier direct |
| 627 | Services bancaires | T | Frais bancaires, commissions |
| 635 | Impots directs (hors IS) | I P | Taxe fonciere, CFE (immobilier direct) ; droits d'enregistrement |
| 654 | Pertes sur creances irrecouvrables | I | Creance locataire definitivement perdue (passage 416 → 654) |
| 6611 | Interets des emprunts bancaires | T | Ventiles depuis le tableau d'amortissement |
| 6615 | Interets des comptes courants d'associes | T | Taux plafonne (TMP BdF), capital integralement libere |
| 667 | Charges nettes sur cessions VMP | V | Moins-value sur cession de VMP (503). Distinct de 657 (immobilisations) |
| 657 | Valeurs comptables des elements d'actif cedes | T | VNC a la date de cession (PCG 2025 : exploitation, ex-675) |
| 671 | Charges exceptionnelles sur operations de gestion | T | Penalites, amendes. Charges non deductibles : marquage pour reintegration |
| 672 | Charges sur exercices anterieurs | T | Corrections d'erreurs decouvertes apres cloture (PCG 2025) |
| 6811 | Dotations aux amortissements immob. corporelles | I | Amortissement annuel batiments/installations (irreversible) |
| 6816 | Dotations aux depreciations immob. corporelles | I | Perte de valeur exceptionnelle (reversible via 7816) |
| 6817 | Dotations aux depreciations actifs circulants | I | Depreciation creances locataires douteuses (491). Reversible via 7817 |
| 687 | Dotations aux provisions exceptionnelles | T | Constitution provisions pour risques (151). Reversible via 787 |
| 6866 | Dotations aux depreciations elements financiers | S V P | Depreciation titres (SCPI, VM, participations) |
| 695 | Impot sur les societes | T | IS de l'exercice (15 % PME / 25 %) |

## Comptes de produits (classe 7) — 2 comptes tronc commun + 8 activites

| Compte | Libelle | Activite | Usage |
|--------|---------|:--------:|-------|
| 752 | Revenus des immeubles | I | Loyers encaisses (immobilier direct) |
| 755 | Quotes-parts resultat operations en commun | S | Distributions SCPI (exclu de la base CVAE) |
| 761 | Produits de participations | P | Dividendes des filiales (regime mere-fille possible) |
| 762 | Produits autres immob. financieres | P | Interets sur prets aux filiales |
| 764 | Revenus des VMP | V | Dividendes et coupons sur VMP/titres immobilises VM |
| 767 | Produits nets sur cessions VMP | V | Plus-value sur cession de VMP (503). Distinct de 757 (immobilisations) |
| 757 | Produits des cessions d'elements d'actif | T | Prix de cession (methode brute : PV = 757 - 657). PCG 2025 : exploitation, ex-775 |
| 772 | Produits sur exercices anterieurs | T | Corrections d'erreurs decouvertes apres cloture. Symetrique de 672 |
| 787 | Reprises sur provisions exceptionnelles | T | Reprise provisions pour risques (151). Symetrique de 687 |
| 7816 | Reprises depreciations immob. corporelles | I | Reprise depreciation immobiliere (symetrique 6816) |
| 7817 | Reprises depreciations actifs circulants | I | Reprise depreciation creances locataires (symetrique 6817) |
| 7866 | Reprises depreciations elements financiers | S V P | Reprise depreciation titres (symetrique 6866) |

## Compte technique

| Compte | Libelle | Usage |
|--------|---------|-------|
| 89 | Bilan d'ouverture | Contrepartie universelle pour les ecritures d'ouverture |

## Comptes de cutoff (fin d'exercice)

Les cutoffs respectent le principe d'independance des exercices :

| Type | Debit | Credit | Sens |
|------|-------|--------|------|
| Interets courus (ICNE) | 6611 | 1688 | On ajoute la charge |
| Produits a recevoir (SCPI T4) | 4181 | 755 | On ajoute le produit |
| Charges a payer (honoraires) | 6226 | 4081 | On ajoute la charge |
| Charges constatees d'avance | 486 | 6xx | On retranche la charge |
| Produits constates d'avance | 7xx | 487 | On retranche le produit |

Chaque cutoff est extourne au 01/01/N+1 (ecriture inverse) pour eviter le double comptage.

## Comptes par activite

| Activite | Comptes specifiques (en plus du tronc commun T) |
|----------|--------------------------------------------------|
| **S — SCPI** | 271, 297, 6227, 6866, 755, 7866 |
| **V — Valeurs mobilieres** | 271 ou 503 (selon duree), 297 ou 590, 6227, 667, 6866, 764, 767, 7866 |
| **I — Immobilier direct** | 211, 2131, 2135, 28131, 28135, 165, 275, 404, 411, 416, 487, 491, 606, 614, 615, 626, 635, 654, 6811, 6816, 6817, 752, 7816, 7817 |
| **P — Participations** | 261, 274, 296, 6227, 635, 6866, 761, 762, 7866 |

## Decompte

- Tronc commun (T) : 33 comptes
- Total generique (4 activites) : 73 comptes
- Soeurise (S uniquement) : 39 comptes
