---
description: Generer et telecharger le Grand Livre comptable d'un exercice.
---

## Workflow

1. Appeler le tool MCP `grand_livre` :
   ```
   Tool: grand_livre
   Params: { "exercice": <exercice>, "date_arret": <date_arret ou null> }
   ```

2. Afficher un resume du Grand Livre genere :

   ```
   Grand Livre {exercice} genere :
   - Comptes : {nb_comptes}
   - Mouvements : {nb_mouvements}
   - Fichier : {filename} ({file_size} octets)
   ```

3. Fournir le lien de telechargement (`download_url`) a l'utilisateur.

4. Si l'utilisateur le demande, commenter la structure :
   - Chaque onglet correspond a un compte
   - Colonnes : Date | N° Ecriture | Libelle | Contrepartie | Debit | Credit | Solde
   - La colonne Contrepartie indique l'autre compte de l'ecriture (debit/credit)
   - Le solde d'ouverture est affiche en premiere ligne
   - Les totaux sont en derniere ligne

## Notes

- Disponible a tout moment, pas uniquement apres pre-cloture.
- Le parametre `date_arret` permet de generer une situation ponctuelle (ex. au 30/06).
- Le fichier est archive dans l'espace abonne et telechargeable via le lien fourni.
- Le lien de telechargement est valide 30 minutes (usage unique).
