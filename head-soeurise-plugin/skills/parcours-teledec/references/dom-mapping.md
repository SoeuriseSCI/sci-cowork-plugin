# Mapping DOM des champs Teledec

Référence détaillée des sélecteurs DOM utilisés par les scripts JS de saisie. Chargée à la demande par le skill `parcours-teledec`.

## 2033-B (données fiscales à saisir)

| Zone | Champ DOM | Description |
|------|-----------|-------------|
| 330 | `repetition2033BReintegrations[i].LA` (libellé) / `.LB-format` (montant) | Réintégrations (lignes dynamiques) |
| 350 | `repetition2033BDeductions[i].MA` (libellé) / `.MB-format` (montant) | Déductions (lignes dynamiques) |
| 360 | `EP-format` | Déficits antérieurs imputés |

**IMPORTANT** : Les lignes de réintégration/déduction sont dans des **tableaux dynamiques cachés**. Il faut CLIQUER sur les liens "Lignes de réintégration" et "Lignes de déduction" pour les rendre visibles dans le DOM AVANT de pouvoir remplir les champs `repetition2033BReintegrations[i].*` et `repetition2033BDeductions[i].*`. Sans cette étape, les sélecteurs ne trouveront rien.

**Note historique** : Avant `teledec_scripts.py` v6 (03/05/2026), le champ EP-format (zone 360) ne persistait pas via `nativeInputValueSetter` et nécessitait une saisie clavier manuelle après save. Depuis v6, tous les champs sont saisis via simulation clavier (`document.execCommand('insertText')`) qui persiste après save. Si une regression survient, vérifier en console les logs `_setByName` après save.

## 2033-C (immobilisations)

| Zone | Champ DOM | Description |
|------|-----------|-------------|
| 480 | `AJ-format` | Début exercice |
| 482 | `BJ-format` | Augmentations — Erreur systématique Teledec |
| 484 | `CJ-format` | Diminutions |
| 486 | `DJ-format` | Fin exercice |

Checkbox Néant : `input[name="RQ"]`

## 2033-D (provisions + déficits)

### Section I — Provisions

Ligne détail "Sur immobilisations" :

| Zone | Champ DOM | Description |
|------|-----------|-------------|
| 630 détail | `AD-format` | Provisions début exercice — DÉPLACER ici depuis BD-format |
| 632 détail | `BD-format` | Provisions augmentations — Erreur systématique Teledec |
| 634 détail | `CD-format` | Provisions diminutions |
| 636 détail | `DD-format` | Provisions fin exercice |

Ligne total provisions :

| Zone | Champ DOM | Description |
|------|-----------|-------------|
| 630 total | `AH-format` | Total provisions début exercice — DÉPLACER ici depuis BH-format |
| 632 total | `BH-format` | Total provisions augmentations |
| 634 total | `CH-format` | Total provisions diminutions |
| 636 total | `DH-format` | Total provisions fin exercice |

**PIÈGE** : Quand il n'y a qu'une seule ligne de provisions, le détail ET le total ont la même valeur. Le script doit corriger les DEUX lignes (détail AD/BD ET total AH/BH), pas juste une.

### Section II — Déficits

| Zone | Champ DOM | Description |
|------|-----------|-------------|
| 982 | `PG-format` | Déficits début exercice — OBLIGATOIRE si déficits |
| 983 | `PH-format` | Déficits imputés — OBLIGATOIRE si déficits |
| 984 | Auto-calculé | Déficits restant à reporter (= 982 - 983) |

Checkbox Néant : `input[name="RQ"]`

## 2033-E, 2033-G, 2069-RCI (néant)

- 2033-E : Checkbox `input[name="DB"]` + modale DOM "Oui"
- 2033-G : Checkbox à identifier (auto-détection par label "néant") + modale DOM "Oui"
- 2069-RCI : Checkbox `input[name="AB"]` + modale DOM "Oui"

**Spécificité 2033-E** : Cocher Néant efface automatiquement les données importées.

## 2033-F (capital social)

Préfixe par associé PP : `repetition2033FPhysiques[index].`

| Champ (après préfixe) | Description |
|------------------------|-------------|
| `BA_3036_1` | Nom complet |
| `BA_3036_2` | Nom d'usage |
| `BR` | Pourcentage |
| `BD` | Nombre de parts |
| `BE` | Date naissance (JJ/MM/AAAA) |
| `BG_3251_1` | Département naissance |
| `BG_3164_1` | Commune naissance — Effacer avant correction |
| `BG_3207_1` | Pays naissance (select) |
| `BA_3042_1` | Adresse (numéro, rue) |
| `BA_3251_1` | Code postal |
| `BA_3164_1` | Ville |
| `BA_3207_1` | Pays adresse (select) |

Totaux (obligatoires, pas auto-calculés) :

| Zone | Champ | Description |
|------|-------|-------------|
| 901 | `GT` | Nb associés PM |
| 902 | `GU` | Nb parts PM |
| 903 | `GV` | Nb associés PP |
| 904 | `GW` | Nb parts PP |
| 905 | `GX` | Total associés — **NE PAS OUBLIER** |
| 906 | `GY` | Total parts |

Checkbox Néant : `input[name="GS"]` (pas RQ).

## 2065

**Règle d'or** : JAMAIS modifier les champs calculés du 2065. Si le résultat est faux, corriger le 2033-B. Ouvrir et sauvegarder après les autres formulaires pour forcer le recalcul.
