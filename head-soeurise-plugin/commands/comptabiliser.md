---
description: Soumettre des documents comptables (PDF) pour extraction et comptabilisation.
---

## Workflow

1. Recueillir le ou les documents PDF du lot a comptabiliser :
   - L'utilisateur les glisse-depose dans la conversation, ou en indique le
     chemin.
   - Types acceptes : releves bancaires, factures, appels de fonds, avis
     d'echeance, bulletins SCPI, avis d'operation titres, quittances, etc.
   - Un lot trimestriel forme une unite atomique : ses documents partent
     ensemble, en une seule soumission.

2. Obtenir une URL d'upload via l'outil MCP `generer_upload_url_comptabilisation` :
   ```
   Tool: generer_upload_url_comptabilisation
   Params: { "exercice": <exercice> }
   ```
   L'outil retourne un champ `upload_url` (valide 30 min, usage unique).

   Le lot transite par cette URL, et non en base64 dans l'appel MCP : un PDF
   trimestriel reel pese plusieurs Mo, ce qui depasse le budget de tokens d'un
   appel d'outil. L'URL permet de transmettre le lot entier, d'un bloc.

3. Televerser le ou les PDF vers `upload_url` par un POST multipart, depuis le
   shell :
   ```
   curl -X POST -F "files=@chemin/vers/lot.pdf" "<upload_url>"
   ```
   Pour plusieurs fichiers, repeter l'option : `-F "files=@facture.pdf" -F "files=@releve.pdf"`.

   Le serveur extrait alors les documents (Claude Vision) et genere les
   ecritures comptables. La reponse JSON contient un token de proposition et le
   nombre d'ecritures proposees.

4. Annoncer le resultat en une reponse courte et factuelle, en **RELAYANT le champ
   `message` renvoye par le serveur** (il contient le lien de validation a jour) :
   ```
   Lot <trimestre> ingere — <N> ecritures proposees.
   Validez-le sur votre espace abonne : <lot_url>
   (lien egalement envoye par email au valideur).
   ```
   Ne **PAS** inventer de consigne de validation, ni mentionner un « token » ou
   « [_Head] VALIDE » : reprendre le `message` / `lot_url` du serveur. Si le serveur
   indique une supersession (« lot precedent remplace »), le relayer. La conversation
   se limite a ce resume ; le detail ligne par ligne se consulte sur la page du lot.

## Validation (mode deliberation)

- A l'issue de l'ingestion, le valideur recoit un **email-invitation** avec un lien
  vers la **page de deliberation du lot** dans l'espace abonne (gerant en mode
  ASSISTE ; EC partenaire en mode CERTIFIE).
- La validation se fait **sur cette page, evenement par evenement** (valider /
  reserve / en attente / rejeter), puis « Clore le lot ». C'est la trace formelle,
  datee et opposable.
- Les ecritures sont inserees **a la cloture du lot**. La validation en bloc par
  token email est **desactivee** (releguee a un secours) : ne pas la proposer.

## Resoumission & recevabilite

- **Ne jamais refuser de soi-meme** une (re)soumission au motif d'un doublon : le
  **serveur** decide. Si l'utilisateur demande de (re)comptabiliser, **proceder**
  (televerser le lot) et relayer la reponse du serveur.
- **Resoumission du meme trimestre** : autorisee. Le serveur gere par **supersession**
  — le nouveau lot **remplace** le precedent lot en attente du meme trimestre (statut
  `REMPLACEE`, retire de l'espace abonne). Une proposition non validee n'affecte jamais
  la comptabilite : reinjecter est sans risque.
- **Recevabilite (sequence des trimestres)** : le serveur n'accepte un nouveau lot que
  pour le trimestre courant (le suivant du dernier valide). S'il repond
  `status: "rejected"` / `recevable: false`, **relayer tel quel le `message`** (ex.
  « Validez d'abord le lot du T3 2025 … ») — ne pas reessayer ni contourner.

## Regles

- Seuls les fichiers PDF sont acceptes.
- Un lot trimestriel reste d'un seul tenant (unite atomique) : le fractionner
  disperserait des ecritures liees — par exemple l'engagement et l'extinction
  d'une meme facture — et fausserait l'extraction.
- Les ecritures suivent le plan comptable SCI IS (skill plan-comptable-sci) et
  sont en partie double (debit = credit).
- La comptabilite reste inchangee tant que la reponse email de validation du
  valideur n'est pas parvenue.
