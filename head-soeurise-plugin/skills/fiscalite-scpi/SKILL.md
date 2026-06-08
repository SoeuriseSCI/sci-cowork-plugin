---
name: fiscalite-scpi
description: Traitement fiscal des parts SCPI detenues par une SCI a l'IS.
  Utiliser quand l'utilisateur parle de SCPI, quotes-parts fiscales,
  reintegrations, deductions, releve fiscal, ligne 330, ligne 350.
---

## Les parts SCPI sont des immobilisations

- Comptabilisees en compte 271 (titres immobilises), pas en 503 (VMP) ni en 261 (participations).
- Les parts SCPI n'ont PAS d'ISIN. Ne pas confondre avec des valeurs mobilieres.
- Pas d'amortissement (contrairement aux immeubles en direct).
- Depreciation eventuelle via provision (compte 297).

## Ecart comptable / fiscal

Les distributions recues des SCPI (cash) ne correspondent PAS aux resultats fiscaux de la SCPI.

| Aspect | Comptabilite | Fiscalite |
|--------|-------------|-----------|
| Quoi | Distributions recues (tresorerie) | Quote-part du resultat fiscal SCPI |
| Source | Releves bancaires + avis de distribution | Releve fiscal annuel (IFU) de la societe de gestion |
| Compte | 755 (produits) | Extra-comptable (liasse uniquement) |
| Frequence | Trimestrielle (T1, T2, T3, T4) | Annuelle |

## Reintegrations et deductions dans la liasse

```
Ligne 330 (reintegrations) = quotes-parts des resultats fiscaux SCPI
Ligne 350 (deductions)     = quotes-parts SCPI comptabilisees (solde du compte 755)
```

**Exemple :**
```
Distributions SCPI recues (compte 755) :  26 000 EUR
Quote-part resultat fiscal (IFU SCPI) :   27 000 EUR

Ligne 330 : +27 000 EUR (reintegration)
Ligne 350 : -26 000 EUR (deduction)
Impact net sur le resultat fiscal : +1 000 EUR
```

## Releve fiscal SCPI

Le releve fiscal (IFU) est un document emis annuellement par la societe de gestion de la SCPI. Il contient la quote-part du resultat fiscal attribuable a chaque associe.

Ce document est requis pour la pre-cloture. Il est transmis via le tool MCP `precloture_mcp` (PDF en base64), via le frontend web, ou par email a _Head.

Les donnees extraites sont stockees dans la table `quotes_parts_scpi` :
- nom_scpi, societe_gestion, quote_part_resultat, exercice

## Attention

- La quote-part SCPI est un montant fiscal, pas comptable. Il n'y a PAS d'ecriture comptable correspondante.
- L'ajustement se fait uniquement dans la liasse fiscale (lignes 330/350 du CERFA 2065).
- Si la SCI detient plusieurs SCPI, chaque quote-part est traitee individuellement.
