---
description: Afficher la balance des comptes d'un exercice.
---

## Workflow

1. Appeler le tool MCP `balance` :
   ```
   Tool: balance
   Params: { "exercice": <exercice> }
   ```

2. Afficher la balance **par GRANDES MASSES**, dans l'ordre **ACTIF / PASSIF /
   CHARGES / PRODUITS** (chaque ligne porte son champ `masse` ; la reponse fournit
   `totaux_par_masse`). Un sous-total par masse, puis le total general :

   **ACTIF**
   | Compte | Libelle | Solde debiteur | Solde crediteur |
   |--------|---------|----------------|-----------------|
   | 271 | Titres immobilises (SCPI) | xxx | |
   | ... | ... | ... | ... |
   | **Sous-total ACTIF** | | **xxx** | **xxx** |

   (idem **PASSIF**, **CHARGES**, **PRODUITS**), puis :
   | **TOTAL GENERAL** | | **xxx** | **xxx** |

   ⚠️ La masse (actif/passif) vient de la **NATURE** du compte (champ `masse` fourni
   par le serveur), **PAS** du sens du solde — ne pas reclasser soi-meme.

3. Verifier que la balance est equilibree (total debits = total credits).

4. Si l'utilisateur le demande, commenter uniquement des FAITS lisibles dans la
   balance, sans jamais supposer ni annoncer le statut de l'exercice (la balance
   ne revele pas le statut) :
   - Compte 120 : resultat de l'exercice (solde a zero apres cloture)
   - Compte 119 : deficit reporte (si present)
   - Compte 444 : IS a payer (si present)
   - Comptes de cutoff (1688, 4081, 4181, 486, 487) : leur presence n'indique
     PAS le statut. Des mouvements sur ces comptes au 01/01, soldes a zero,
     correspondent le plus souvent aux a-nouveaux + extournes des cutoffs de
     l'exercice precedent — situation NORMALE dans un exercice OUVERT.

## Statut de l'exercice : ne rien supposer

- ⚠️ Le statut (OUVERT / PRE_CLOTURE / CLOTURE) ne se deduit JAMAIS des comptes
  presents dans la balance. La presence de comptes de cutoff ne signifie pas
  « pre-cloture » (cf. ci-dessus : ce sont souvent les extournes de l'an dernier).
- En cas de besoin du statut, il provient de l'etat reel de l'exercice, pas d'une
  inference sur la balance. Dans le doute, presenter les faits et laisser
  l'utilisateur trancher plutot qu'affirmer un statut (risque d'induire en erreur).

## Notes

- La balance inclut tous les comptes ayant eu au moins un mouvement sur l'exercice.
- Les comptes sont tries par numero.
- Les classes 6 et 7 (charges/produits) sont a zero apres cloture (remis a zero
  dans le nouvel exercice) ; leur presence non soldee ne distingue pas a elle
  seule un exercice OUVERT d'un exercice en PRE_CLOTURE.
