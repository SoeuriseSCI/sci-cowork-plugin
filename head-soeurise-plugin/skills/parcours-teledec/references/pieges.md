# Pièges connus et erreurs systématiques de Teledec

Référence détaillée des pièges rencontrés en saisie sur teledec.fr et des erreurs systématiques d'import. Chargée à la demande par le skill `parcours-teledec`.

## Pièges connus (15)

### 1. Champ 350 (EL-format) auto-calculé
Ne jamais saisir directement dans le champ EL-format (zone 350 déductions). Passer par les lignes de déduction via le lien "Lignes de déduction" qui ouvre un tableau dynamique `repetition2033BDeductions[i]`.

### 2. Checkbox Néant et dialogue modal
Après chaque clic sur une checkbox Néant, une **modale DOM** Oui/Non apparaît (PAS un `window.confirm()`). Cette modale est un élément HTML standard. Le script v3.0 clique automatiquement le bouton "Oui". Si le script ne fonctionne pas : chercher manuellement un bouton "Oui" dans la page et le cliquer.

**Noms des checkboxes Néant** (varient par formulaire) :
| Formulaire | Checkbox name |
|------------|---------------|
| 2033-C     | RQ            |
| 2033-D     | RQ            |
| 2033-E     | DB            |
| 2033-F     | GS            |
| 2033-G     | à identifier  |
| 2069-RCI   | AB            |

Le script néant v3.0 détecte automatiquement la checkbox (recherche par label "néant" + fallback noms connus).

### 3. Totaux 2033-F NON auto-calculés
Les zones 901-906 (totaux) ne sont PAS auto-calculées par Teledec. Il faut remplir explicitement les 6 champs de totaux (GT, GU, GV, GW, GX, GY) après avoir saisi les associés. **Oubli de la ligne 905 = erreur bloquante.**

### 4. Checkbox Néant 2033-F
La checkbox Néant du 2033-F s'appelle `GS` (pas `RQ` comme les autres formulaires).

### 5. Blocs PP limités à 2
Par défaut, seulement 2 blocs personnes physiques sont visibles. Cliquer sur "+ Ajouter une ligne" avant de remplir les associés supplémentaires.

### 6. Incohérence 2065 / 2033-B — Règle d'or
**JAMAIS modifier le 2065 directement**. Le 2065 est un formulaire AUTO-CALCULÉ à partir du 2033-B. Si le résultat fiscal du 2065 est faux : corriger le 2033-B (zones 330, 350, 360), sauvegarder, puis ouvrir le 2065 et sauvegarder pour forcer le recalcul. Si Teledec signale une incohérence entre le 2065 et le 2033-B, c'est le 2033-B qui fait foi.

### 7. Événements JS non déclenchés
Après saisie dans un champ, les événements `input`, `change` et `blur` doivent être déclenchés pour que Teledec prenne en compte la valeur.

### 8. Navigation post-sauvegarde
Après sauvegarde d'un formulaire, Teledec peut rediriger vers "Mes déclarations" au lieu de rester dans la liasse. Détecter par le titre de page et re-cliquer sur "Compléter".

### 9. Code postal et ville séparés (2033-F)
Les champs d'adresse du 2033-F ont des champs distincts pour code postal (`BA_3251_1`) et ville (`BA_3164_1`). Ne pas saisir "75013 Paris" dans un seul champ.

### 10. Validation obligatoire du 2033-A
Après l'import, le 2033-A reste en statut "Importé, à vérifier". Il faut l'ouvrir et le sauvegarder pour le valider, sinon la déclaration reste incomplète.

### 11. Erreurs systématiques de Teledec lors de l'import
Teledec importe incorrectement :
- Les immobilisations financières stables comme des "Augmentations" dans le 2033-C
- Les provisions stables comme des "Augmentations" dans le 2033-D

La correction est un DÉPLACEMENT (Augmentations → Début exercice), PAS une suppression. Ces erreurs doivent toujours être corrigées après chaque import (voir Étape 3a/3b dans `automation.md`).

### 12. Case Néant du 2033-E efface les données
Lorsque la case Néant est correctement cochée sur le 2033-E, Teledec efface automatiquement toutes les données importées. Si les données restent visibles, c'est que la case Néant n'a pas été correctement validée.

### 13. "Execution context destroyed" après script Néant
Après le clic sur "Oui" dans la modale de confirmation Néant, Teledec peut recharger la page. Cela détruit le contexte JS et génère l'erreur `{"code":-32000,"message":"Execution context was destroyed."}`.

**C'est NORMAL et ATTENDU**. Le néant a bien fonctionné. Vérifier le statut sur la vue d'ensemble. Ne PAS re-exécuter le script ni investiguer l'erreur.

### 14. Bouton "Ajouter une ligne" du 2033-F — section PM vs PP
Le 2033-F a DEUX boutons "Ajouter une ligne" identiques : un pour les Personnes Morales (PM) et un pour les Personnes Physiques (PP). Le script v3.2 identifie le bouton PP par sa position DOM (le premier bouton "Ajouter" situé APRÈS les champs `repetition2033FPhysiques`).

**VÉRIFICATION OBLIGATOIRE** après le clic : le script vérifie que le bloc PP attendu (ex: `repetition2033FPhysiques[2].BA_3036_1`) existe dans le DOM. Si le message "BLOC PP[2] NON TROUVÉ" apparaît en console : le clic a atterri dans la section PM. Cliquer manuellement sur le bouton Ajouter dans la section Personnes Physiques.

**Si le script échoue et qu'un script de rattrapage est nécessaire** : respecter le mapping exact des associés [0] et [1] déjà saisis. Utiliser les MÊMES champs (BA_3036_1 = nom complet, pas nom de famille seul). Consulter `export_donnees_fiscales` pour les données exactes.

### 15. Selects pays — accents et codes ISO
Les selects pays de Teledec peuvent utiliser des textes avec accents ("SUÈDE" et non "SUEDE") ou des codes ISO (value="SE"). Le script v3.1 cherche d'abord par texte exact, puis sans accents, puis par code ISO. Si le select reste vide : lister les options du select en console pour identifier le format exact utilisé par Teledec.

## Erreurs systématiques de Teledec lors de l'import

### Erreur 1 : Immobilisations financières (2033-C)

**Symptôme** : La zone 482 (Augmentations) contient la valeur des immobilisations financières (titres SCPI) alors qu'elles existaient déjà avant l'exercice.

**Cause** : Bug de Teledec qui importe les immobilisations stables de N-1 comme des augmentations de N au lieu de les placer en début d'exercice.

**Correction** (DÉPLACER, pas effacer) :
1. COPIER la valeur de la zone 482 (Augmentations, champ BJ-format)
2. COLLER cette valeur dans la zone 480 (Début exercice, champ AJ-format)
3. EFFACER la zone 482 (mettre 0 ou vide)
4. VÉRIFIER : zone 486 (Fin exercice) doit être inchangée et égale à 2033-A zone 040

**Formule de contrôle** : 486 = 480 + 482 - 484. Si on efface 482 sans remplir 480, la zone 486 tombe à 0 = ERREUR GRAVE.

### Erreur 2 : Provisions sur immobilisations (2033-D)

**Symptôme** : La colonne "Augmentations" (zone 632) de la ligne "Provisions pour dépréciation des immobilisations" contient une valeur alors que ces provisions existaient déjà avant l'exercice.

**Cause** : Bug de Teledec qui importe les provisions stables de N-1 comme des augmentations de N au lieu de les placer en début d'exercice.

**Correction** (DÉPLACER, pas effacer) :
1. COPIER la valeur de la zone 632 (Augmentations)
2. COLLER cette valeur dans la zone 630 (Début exercice)
3. EFFACER la zone 632 (mettre 0 ou vide)
4. VÉRIFIER : zone 636 (Fin exercice) doit être inchangée

**Formule de contrôle** : 636 = 630 + 632 - 634. Si on efface 632 sans remplir 630, la zone 636 tombe à 0 = ERREUR GRAVE.

## Matrice de cohérence inter-formulaires

Certaines valeurs DOIVENT être cohérentes entre formulaires :

| Source | Zone | Destination | Zone | Contrainte |
|--------|------|-------------|------|------------|
| 2033-A | 040 (Immob. financières) | 2033-C | 486 (Fin exercice) | Égalité stricte |
| 2033-D | 983 (Déficits imputés) | 2033-B | 360 (Déficits antérieurs) | Égalité stricte |
| 2033-B | 360 (Déficits imputés) | 2033-D | 982 (Début exercice) | 982 ≥ 360 |

**VÉRIFICATION OBLIGATOIRE** :
- Après saisie de la 2033-B zone 360 : vérifier que 2033-D zone 983 = même valeur
- Après saisie de la 2033-D : vérifier que 2033-B zone 360 = 2033-D zone 983
- Si incohérence : ERREUR BLOQUANTE → corriger immédiatement
