# Règles de saisie Teledec

Référence détaillée des règles strictes à appliquer durant la saisie sur teledec.fr.
Chargée à la demande par le skill `parcours-teledec` lorsque la session de saisie commence.

## Règle 0 : RELIRE le skill avant chaque session ET après chaque /compact

OBLIGATOIRE — Relire INTÉGRALEMENT le skill `parcours-teledec` :
- Avant de commencer une session de saisie Teledec
- **Après tout /compact ou résumé de contexte** (le compact perd les détails fins comme les noms de champs DOM, les pièges, et les procédures exactes)

Les sessions précédentes ont montré que des erreurs critiques et une perte de focus surviennent quand les instructions ne sont pas relues.

## Règle 1 : VALIDER les résultats Teledec vs CERFA JSON

OBLIGATOIRE après chaque sauvegarde de formulaire et en vérification finale :
- Le tool MCP `generer_scripts_teledec` retourne un champ `valeurs_attendues`
- Chaque script affiche en console les `VALEURS ATTENDUES (CERFA JSON)`
- Après sauvegarde, COMPARER les valeurs affichées par Teledec avec les valeurs attendues
- Si écart : NE PAS continuer au formulaire suivant, CORRIGER d'abord

**Valeurs à valider obligatoirement** :
- Zone 330 (réintégrations) : total doit correspondre au CERFA JSON
- Zone 350 (déductions) : total doit correspondre au CERFA JSON
- Zone 360 (déficits imputés) : doit correspondre au CERFA JSON
- Résultat fiscal du 2065 : doit correspondre au `resultat_fiscal_final` du CERFA JSON
- Adresses des associés (2033-F) : doivent correspondre au CERFA JSON, JAMAIS inventées

**Si une valeur ne correspond pas** : c'est soit un bug du script, soit un problème de sauvegarde. Dans les deux cas, STOP et remonter le problème.

## Règle 2 : SAUVEGARDER avec le bon bouton

Teledec affiche 3 boutons en bas de chaque formulaire :
1. **"Sauvegarder et suivant"** — sauvegarde et ouvre le formulaire suivant (ordre Teledec)
2. **"Sauvegarder et retour"** — sauvegarde et retourne à la liste des formulaires (**RECOMMANDÉ**)
3. **"Retourner à la déclaration"** — retourne à la liste **SANS sauvegarder** (**INTERDIT après modification**)

**TOUJOURS utiliser "Sauvegarder et retour"** (bouton #2) après chaque script ou correction.
- On ne traite PAS les formulaires dans l'ordre séquentiel de Teledec, donc "et suivant" n'est pas adapté.
- "Retourner à la déclaration" NE SAUVEGARDE PAS : toutes les modifications JS seraient perdues. C'est potentiellement la cause des corrections 2033-C/D qui "n'ont pas tenu" lors des sessions précédentes.

**EXCEPTION** : "Retourner à la déclaration" est OK uniquement si on a déjà sauvegardé et qu'on veut juste quitter le formulaire (par exemple après un "Sauvegarder et suivant" qui a ouvert un formulaire non désiré).

## Interdiction 1 : JAMAIS utiliser Tab pour sortir d'un champ textarea

JAMAIS utiliser la touche Tab après avoir saisi du texte dans un champ de type textarea (libellé, adresse, nom...).

**Pourquoi** : Dans un textarea, Tab insère une tabulation DANS le contenu au lieu de naviguer vers le champ suivant. Le curseur reste dans le champ et les données suivantes (%, nb parts, date...) finissent concaténées dans le même champ.

**CORRECT** : Après avoir saisi un libellé ou un nom, CLIQUER directement sur le champ suivant (% détention, nb parts, etc.).

**INCORRECT** : Saisir "Pauline Bergsten" puis Tab → insère une tabulation, les données suivantes s'ajoutent au nom : "Pauline Bergsten	49.9	499..."

## Interdiction 2 : JAMAIS utiliser form_input avec refs

Utiliser les sélecteurs `[name="..."]` du DOM pour identifier les champs.

**Pourquoi** : Les refs (ref_1, ref_2...) sont temporaires et pointent souvent vers les mauvais champs après navigation.

## Interdiction 3 : JAMAIS cocher "Néant" sans distinguer CAS A vs CAS B

**CAS A : Données à CONSERVER (corrections = DÉPLACER)**
- Le formulaire contient des données qui DOIVENT être déclarées
- Teledec a mal placé ces données (erreur systématique)
- **ACTION** : DÉPLACER de "Augmentations" vers "Début exercice", NE PAS cocher Néant
- **Exemples** :
  * 2033-C : Parts SCPI existent → DÉPLACER zone 482 vers zone 480, NE PAS cocher Néant
  * 2033-D : Provisions existent → DÉPLACER zone 632 vers zone 630, NE PAS cocher Néant (sauf si aucun déficit)

**CAS B : Données NON APPLICABLES (effacement)**
- Le formulaire contient des données qui NE DOIVENT PAS exister
- Teledec a importé des données par erreur
- **ACTION** : COCHER Néant pour EFFACER
- **Exemples** :
  * 2033-E : Valeur ajoutée (SCI passive = pas d'activité commerciale) → COCHER Néant
  * 2033-G : Filiales (SCI sans participation) → COCHER Néant
  * 2069-RCI : Crédits d'impôts (SCI sans crédits) → COCHER Néant

## Interdiction 4 : JAMAIS saisir sans validation visuelle

OBLIGATOIRE après chaque saisie de zone critique :
- Prendre un screenshot
- Vérifier que la valeur affichée = valeur attendue
- Si écart → effacer complètement et recommencer

**Zones critiques** :
- 2033-B zones 330, 350, 360
- 2033-D zones 982, 983
- 2033-F tous les champs associés + totaux (905, 906)

## Règle d'or 2065 / 2033-B

**JAMAIS modifier les champs calculés du 2065** (bénéfice, IS, résultat fiscal). Le 2065 est AUTO-CALCULÉ à partir du 2033-B.

Si le résultat fiscal du 2065 est faux : corriger le 2033-B (zones 330, 350, 360), sauvegarder, puis ouvrir le 2065 et sauvegarder pour forcer le recalcul.

Si Teledec signale une incohérence entre le 2065 et le 2033-B, c'est le 2033-B qui fait foi.

## Sécurité : Télétransmission interdite

NE JAMAIS cliquer sur "Télétransmettre", "Transmettre", "Envoyer à la DGFiP" ou tout bouton de soumission définitive. La Phase 2 s'arrête après la vérification finale et l'archivage PDF. La télétransmission est une action manuelle hors scope, effectuée par le client quand il sera prêt.

## Règle compact : pas d'improvisation

Après tout /compact, relire ce fichier ET les autres références (`pieges.md`, `dom-mapping.md`, `automation.md`, `verification.md`) AVANT de continuer la saisie. Le compact perd les détails fins (noms DOM, pièges, procédures d'archivage). Sans relecture, les procédures sont improvisées au lieu d'être suivies, ce qui cause des erreurs et double le temps d'exécution.

NE JAMAIS improviser une procédure d'archivage, de saisie ou de correction sans avoir relu les références. Elles contiennent les scripts exacts qui fonctionnent.
