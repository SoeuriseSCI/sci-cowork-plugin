---
description: Generer et telecharger le Livre-journal comptable d'un exercice.
---

## Workflow

1. Appeler le tool MCP `journal` :
   ```
   Tool: journal
   Params: { "exercice": <exercice>, "date_arret": <date_arret ou null> }
   ```

2. Afficher un resume du Livre-journal genere :

   ```
   Livre-journal {exercice} genere :
   - Ecritures : {nb_ecritures}
   - Fichier : {filename} ({file_size} octets)
   ```

3. Fournir le lien de telechargement (`download_url`) a l'utilisateur.

4. Si l'utilisateur le demande, commenter la structure :
   - Toutes les ecritures en ordre chronologique
   - Colonnes : N° Ecriture | Date | Libelle | Compte Debit | Libelle Debit | Compte Credit | Libelle Credit | Montant | Type | Piece jointe | Validee
   - Un total general en derniere ligne

## Notes

- Disponible a tout moment, pas uniquement apres pre-cloture.
- Le parametre `date_arret` permet de generer une situation ponctuelle (ex. au 30/06).
- Le fichier est archive dans l'espace abonne et telechargeable via le lien fourni.
- Le lien de telechargement est valide 30 minutes (usage unique).
- Le Livre-journal est un document comptable obligatoire (article R123-174 du Code de commerce).
