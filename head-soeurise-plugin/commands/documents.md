---
description: Lister et telecharger les documents generes pour un exercice.
---

## Workflow

1. Appeler le tool MCP `documents` :
   ```
   Tool: documents
   Params: { "exercice": <exercice> }
   ```

2. Afficher la liste des documents sous forme de tableau :

   | # | Type | Nom | Date de generation |
   |---|------|-----|--------------------|
   | 1 | fec | FEC_2025.txt | 2025-12-31 |
   | 2 | grand_livre | GrandLivre_2025.pdf | 2025-12-31 |
   | 3 | cerfa | CERFA_2065_2025.pdf | 2025-12-31 |
   | ... | ... | ... | ... |

3. Demander a l'utilisateur s'il souhaite telecharger un document.

4. Si oui, appeler le tool MCP `download_document` :
   ```
   Tool: download_document
   Params: { "document_id": <id_du_document> }
   ```
   Le document est retourne en base64 ou sous forme de lien de telechargement.

## Types de documents

| Type | Description | Genere par |
|------|-------------|------------|
| `fec` | Fichier des Ecritures Comptables (format DGFiP) | Pre-cloture |
| `grand_livre` | Grand Livre comptable (Excel) | On-demand ou pre-cloture |
| `livre_journal` | Livre-journal comptable (Excel) | On-demand ou pre-cloture |
| `cerfa` | Liasse fiscale 2065 + 2033-A/B/D/F (PDF) | Pre-cloture |

## Notes

- Le Grand Livre et le Livre-journal sont disponibles a tout moment via `/sci:grand-livre` et `/sci:journal`.
- Le FEC et les CERFA ne sont disponibles qu'apres la pre-cloture.
- Le FEC est au format reglementaire (article A.47 A-1 du LPF).
- Les CERFA sont pre-remplis avec les donnees de l'exercice.
