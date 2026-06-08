# Checklist de vérification finale Teledec

Référence détaillée de la checklist à passer en fin de saisie sur teledec.fr, avant le point de validation [PV2]. Chargée à la demande par le skill `parcours-teledec`.

Avant de déclarer la saisie terminée, vérifier TOUS ces points.

## Statuts des formulaires

- [ ] 2033-A : Statut "Renseigné" (PAS "Importé, à vérifier")
- [ ] 2033-B : Statut "Renseigné"
- [ ] 2033-C : Statut "Renseigné" ou "Renseigné à néant"
- [ ] 2033-D : Statut "Renseigné" (si déficits) ou "Renseigné à néant" (sinon)
- [ ] 2033-E : Statut "Renseigné à néant"
- [ ] 2033-F : Statut "Renseigné"
- [ ] 2033-G : Statut "Renseigné à néant"
- [ ] 2069-RCI : Statut "Renseigné à néant"
- [ ] 2065 : Statut "Renseigné" (PAS "À compléter" — vérifier annexe 2065-BIS)
- [ ] 2065 annexe 2065-BIS : case "Si dépose néant" COCHÉE (distributions/rémunérations)

## Corrections des erreurs systématiques Teledec (DÉPLACER, pas effacer)

- [ ] 2033-C : Zone 480 (Début exercice) = valeur des immobilisations financières
- [ ] 2033-C : Zone 482 (Augmentations) = 0 ou vide (si pas d'acquisition réelle)
- [ ] 2033-C : Zone 486 (Fin exercice) = INCHANGÉE par rapport à avant correction
- [ ] 2033-D : Zone 630 (Provisions début exercice) = valeur des provisions
- [ ] 2033-D : Zone 632 (Provisions augmentations) = 0 ou vide (si pas d'augmentation réelle)
- [ ] 2033-D : Zone 636 (Provisions fin exercice) = INCHANGÉE par rapport à avant correction
- [ ] 2033-A : Zone 182 (Coût immobilisations acquises) = 0 ou vide (si pas d'acquisition réelle)

## Données saisies correctement

- [ ] 2033-B : Réintégrations saisies (zone 330)
- [ ] 2033-B : Déductions saisies (zone 350)
- [ ] 2033-B : Déficits imputés saisis (zone 360) si applicable — CAUSE #1 D'ERREUR
- [ ] 2033-D : Ligne 982 (Déficits début exercice) SAISIE si déficits
- [ ] 2033-D : Ligne 983 (Déficits imputés) SAISIE si déficits
- [ ] 2033-E : Données importées EFFACÉES (case Néant cochée correctement)
- [ ] 2033-F : Tous les associés saisis correctement
- [ ] 2033-F : **LIGNE 905 (Total associés) SAISIE** — CRITIQUE
- [ ] 2033-F : Ligne 906 (Total parts) SAISIE
- [ ] 2033-F : Pas de double saisie dans les champs (vérifier commune de naissance)

## Cohérence inter-formulaires

- [ ] 2033-D zone 983 = 2033-B zone 360 (déficits imputés)
- [ ] 2033-C zone 486 = 2033-A zone 040 (immobilisations financières)

## Absence d'erreurs

- [ ] **Aucune erreur bloquante** affichée sur la page principale
- [ ] **Aucune incohérence** affichée sur la page principale
- [ ] Aucun message d'erreur en rouge sur aucun formulaire

## Conclusion de la checklist

### Si toutes les cases sont cochées
→ La saisie est COMPLÈTE et VALIDE
→ Informer l'utilisateur que la liasse est prête pour validation finale et télétransmission

### Si au moins une case n'est PAS cochée
→ La saisie est INCOMPLÈTE
→ Corriger les points manquants avant de déclarer la saisie terminée
→ NE PAS dire à l'utilisateur que c'est terminé

## [PV2] Point de validation

Présenter au client :
- Liste des formulaires avec leur statut
- Récap des valeurs saisies (réintégrations, déductions, déficits, associés)
- Résultat de la checklist (tous les points validés)
- Demander : "La liasse est complète sur Teledec. Valides-tu ces saisies ?"

Si validé : passer à l'archivage PDF (étape 11 dans `automation.md`).
