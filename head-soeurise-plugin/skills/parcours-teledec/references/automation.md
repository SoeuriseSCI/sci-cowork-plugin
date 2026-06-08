# Procédure d'automatisation Teledec — Phase 2 détaillée

Référence détaillée de la procédure de saisie sur teledec.fr (étapes 0 à 11). Chargée à la demande par le skill `parcours-teledec`.

## Étape 0 : Configurer les permissions Chrome (PRÉREQUIS UTILISATEUR)

CRITIQUE pour éviter les interruptions constantes lors de la saisie sur Teledec.

**Problème** : Par défaut, Claude in Chrome demande une autorisation pour chaque action sur un site externe (clic, saisie, sauvegarde). Cela rend l'automatisation inutilisable (20-30 interruptions par session).

**Solution** : L'utilisateur doit configurer les permissions **AVANT** de lancer la Phase 2. Deux méthodes :

### Méthode A : Lors de la première demande de permission
Quand Claude in Chrome demande la permission d'agir sur teledec.fr, cliquer sur **"Always allow actions on this site"** au lieu de "Allow this action".

### Méthode B : Via les paramètres de l'extension
1. Cliquer sur l'icône Claude in Chrome
2. Cliquer sur les **trois points** en haut à droite du panneau
3. Aller dans **Settings > Permissions**
4. Ajouter teledec.fr dans les sites approuvés ("Always allow")

**Vérification** : Demander à l'utilisateur AVANT de commencer : "Pour que l'automatisation fonctionne sans interruption, teledec.fr doit être dans vos sites approuvés dans Claude in Chrome (Settings > Permissions > Always allow). Pouvez-vous vérifier ?"

Si l'utilisateur confirme que c'est configuré : procéder sans interruption.
Si l'utilisateur ne sait pas : lui expliquer la Méthode A ou B ci-dessus.

## Étape 0bis : Health-check Claude in Chrome (AVANT Phase 2)

Le test du 03/05/2026 a montré 15 min de cafouillage initial parce que Claude in Chrome n'était pas correctement appairé à la session Cowork (extension installée mais pas signée avec le bon compte, ou en attente d'un clic « Connect » utilisateur). Il vaut mieux **détecter ce problème en 30 secondes** plutôt qu'en 15 minutes après plusieurs tentatives ratées.

**Procédure obligatoire AVANT de lancer la Phase 2** :

1. **Tool MCP** : appeler `mcp__claude-in-chrome__list_connected_browsers` (ou équivalent dans la session courante)
2. **Si la liste est vide** (« 0 browser detected ») :
   - Ne PAS continuer la Phase 2.
   - Demander à l'utilisateur : *« Claude in Chrome n'est pas appairé à cette session. Vérifie : (a) extension Claude in Chrome installée et active dans Chrome, (b) signée avec le compte Anthropic utilisé pour cette session Cowork, (c) éventuellement cliquer "Connect" ou "Pair" dans le popup de l'extension. Quand c'est fait, redis "prêt" — je relance le check. »*
   - Attendre confirmation, retester.
3. **Si la liste contient ≥1 browser** : sélectionner celui où teledec.fr est ouvert et continuer.

**Bénéfice** : on attrape le problème de pairing en 30 secondes au lieu de 15 minutes (ce que le test du 03/05 a montré comme coût d'absence de cette vérification).

## Prérequis Phase 2

- Phase 1 terminée (balance Excel + données fiscales JSON disponibles)
- Le champ `download_url` de la réponse `export_balance_teledec` doit être disponible (URL avec token jetable, aucun header Authorization requis)
- Vérifier que la balance contient des comptes de TOUTES les classes nécessaires : classes 1-5 (bilan) ET classes 6-7 (gestion : charges et produits). Une petite SCI peut n'avoir que ~24 comptes au total, c'est normal. Ce qui compte : la PRÉSENCE de comptes classe 6 (ex: 695) et classe 7 (ex: 762). Si classes 6-7 absentes : STOP, la balance est incomplète (manque le compte de résultat)
- Client connecté à son compte Teledec dans Chrome
- Claude in Chrome actif avec accès au portail Teledec

## INTERDIT : génération de fichier Excel en JavaScript

NE JAMAIS générer le fichier Excel dans le navigateur (pas de SheetJS, pas de génération dynamique). Le fichier Excel DOIT provenir du serveur via `export_balance_teledec` (MCP tool). La génération en JavaScript a produit une balance incomplète (seulement 24 comptes bilan, sans les comptes de gestion classes 6-7) qui a rendu le 2033-B vide et fausse tous les calculs fiscaux.

## INTERDIT : réutilisation d'un ancien fichier balance

NE JAMAIS télécharger un fichier balance existant via `download_document` ou `documents`. TOUJOURS appeler `export_balance_teledec` pour générer un fichier FRAIS à chaque session. Il peut exister d'anciennes versions du fichier en base (issues de sessions échouées) qui contiennent des données incomplètes. Seul `export_balance_teledec` garantit une balance fraîche générée à partir de la comptabilité courante.

Le `download_url` retourné par `export_balance_teledec` est le SEUL URL valide pour l'import dans Teledec.

## Modales de confirmation Teledec (DOM, PAS window.confirm)

Teledec utilise des **modales DOM** (éléments HTML) et NON `window.confirm()`. L'ancien script de désactivation `window.confirm = function() { return true; }` NE FONCTIONNE PAS. Les scripts v3.0 gèrent automatiquement les modales en cliquant le bouton "Oui" dans le DOM.

Si un script ne détecte pas la modale : chercher manuellement un bouton "Oui" visible dans la page.

---

## Étape 1 : Vérifier la session Teledec

- [Chrome] Naviguer vers teledec.fr
- Si page de login : demander au client de se connecter
- Si "Mes déclarations" : la session est active

## Étape 2 : Accéder à la liasse

**Prérequis bloquant — état attendu de la liste des déclarations** :

La liasse de l'exercice cible **ne doit PAS exister** dans la liste « Mes déclarations » au démarrage du workflow. C'est le client qui s'en assure AVANT de lancer la commande Cowork (il doit avoir supprimé toute liasse résiduelle de tests précédents pour le même exercice).

- **Si la liasse N'EXISTE PAS (cas attendu)** : cliquer **« Remplir une liasse pour un autre exercice »**, sélectionner les dates de l'exercice (préremplies normalement), valider via le bouton vert **« Sauvegarder »**. Une nouvelle liasse vide est créée. Cliquer **« Compléter »** dessus pour arriver sur la liste des formulaires.

- **Si la liasse EXISTE DÉJÀ (cas anormal)** : **STOP**. Ne pas continuer le workflow. Remonter à l'utilisateur :
  > « Une liasse pour l'exercice <année> existe déjà sur Teledec. Pour éviter d'écraser des données ou de créer un état incohérent, supprime-la manuellement sur Teledec (« Mes déclarations » → action de suppression sur la ligne concernée) puis relance la commande. »
  
  Cette règle évite les conflits avec d'éventuels tests antérieurs ou liasses partiellement remplies. Le ménage relève de l'utilisateur, pas de l'agent.

## Étape 2bis : Robustesse côté serveur MCP

Si un appel à un tool MCP `head-soeurise` échoue avec une erreur de connexion (« serveur indisponible », « connection refused », « MCP server not connected », timeout, etc.) :

- **Ne PAS solliciter immédiatement l'utilisateur**. Le serveur Soeurise est sur Render avec un plan payant (pas de mise en veille théorique). Une indisponibilité momentanée est généralement transitoire (redéploiement, hiccup réseau, latence Cloudflare).
- **Retry automatique** : attendre 5 secondes puis ré-appeler le même tool. Faire jusqu'à 3 tentatives espacées de 5, 10, puis 20 secondes (backoff progressif).
- **Si après 3 tentatives** le tool échoue toujours : alors seulement remonter à l'utilisateur avec un message factuel : « Le serveur Soeurise n'a pas répondu après 3 tentatives sur 35 secondes. Vérifie l'état sur https://head-soeurise-web.onrender.com/health. Si le `/health` répond OK, c'est un bug à investiguer ; sinon attendre que Render ait redémarré le service. »
- **Pendant les retries**, faire silence (pas de message intermédiaire à l'utilisateur). Une erreur transitoire ne doit pas générer de friction conversationnelle.

## Étape 3 : Importer la balance Excel

### Méthode 1 : Import automatique via JavaScript (PRÉFÉRÉ)

- [Chrome] Cliquer "Autres actions"
- [Chrome] Cliquer "Remplir cette liasse à partir d'un logiciel comptable"
- [Chrome/JS] Exécuter le script suivant en remplaçant DOWNLOAD_URL par la valeur exacte de `download_url` retournée par `export_balance_teledec`. IMPORTANT : l'URL contient un token jetable — aucun header Authorization requis.

```javascript
const fileInput = document.querySelector('input[type="file"]');
if (!fileInput) {
  console.error("ERREUR : input[type='file'] non trouve");
} else {
  fetch('DOWNLOAD_URL')
  .then(r => {
    if (!r.ok) throw new Error('HTTP ' + r.status);
    return r.blob();
  })
  .then(blob => {
    const file = new File([blob], 'balance_teledec.xlsx',
      {type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'});
    const dt = new DataTransfer();
    dt.items.add(file);
    fileInput.files = dt.files;
    fileInput.dispatchEvent(new Event('change', {bubbles: true}));
    console.log("OK:", file.name, file.size, "octets");
  })
  .catch(err => console.error("ERREUR:", err));
}
```

- [Chrome] Attendre 2 secondes
- [Chrome] VÉRIFIER que le nom du fichier apparaît dans l'interface
- [Chrome] Cliquer "Importer la balance"
- Attendre la confirmation d'import

**VÉRIFICATION CRITIQUE APRÈS IMPORT** :
- Retourner à la liste des formulaires
- Vérifier que le 2033-A ET le 2033-B sont TOUS LES DEUX passés en statut "Importé, à vérifier" (ou équivalent)
- **SI le 2033-B est resté vide / non renseigné** : l'import est INCOMPLET. Cela signifie que le fichier Excel ne contenait pas les comptes de gestion (classes 6-7). STOP IMMÉDIAT : ne PAS continuer la saisie. Signaler le problème à l'utilisateur et recommencer la Phase 1.
- **SI les deux formulaires sont remplis** : continuer normalement

### Méthode 2 : Import manuel (FALLBACK)

Si la méthode automatique échoue (aucun fichier n'apparaît après le script) :

- [Chrome] Demander à l'utilisateur : "L'import automatique a échoué. Pouvez-vous télécharger manuellement le fichier balance_teledec.xlsx depuis [LIEN] et l'uploader dans Teledec ?"
- [USER] Télécharge le fichier et l'uploade manuellement
- [Chrome] Attendre que l'utilisateur confirme l'import terminé
- Vérifier que les formulaires 2033-A et 2033-B passent en statut "Importé, à vérifier"

## Étape 3 (suite) : SAISIE PAR SCRIPTS JS BATCH

Au lieu de saisir champ par champ (~50 actions, 2h+), utiliser les scripts JS générés par le backend (~7 exécutions, 30-45 min).

### 3-pre. Obtenir les scripts (MCP)

- Appeler le tool MCP `generer_scripts_teledec` avec l'exercice
- Le serveur retourne **9 scripts** JS prêts à exécuter :
  * 3 corrections (2033-C, 2033-D, 2033-A)
  * 1 saisie 2033-B
  * **3 néant séparés** (2033-E, 2033-G, 2069-RCI) — un script par formulaire
  * 1 saisie 2033-F
  * 1 recalcul 2065
- CONSERVER la réponse complète pour les étapes suivantes
- L'ordre d'exécution est indiqué dans le champ `ordre_execution`

**PRINCIPE** : Pour chaque formulaire ci-dessous, la procédure est :
1. [Chrome] Ouvrir le formulaire ("Compléter")
2. [Chrome/JS] Exécuter le script JS correspondant dans la console
3. [Chrome] Lire les logs console pour vérifier le résultat
4. [Chrome] Vérifier VISUELLEMENT que les valeurs sont correctes
5. [Chrome] Cliquer **"Sauvegarder et retour"** (JAMAIS "Retourner à la déclaration")

CRITIQUE : Teledec commet des erreurs systématiques lors de l'import. Corriger 2033-C et 2033-D AVANT de valider le 2033-A.

### Étape 3a : Corriger 2033-C (script_correction_2033C + saisie clavier)

**ATTENTION — refonte 04/05/2026** : même logique que 3b. Les zones immobilisations sont RECALCULÉES par Teledec après save. Le script logge le PLAN de saisie ligne par ligne ; Cowork saisit au CLAVIER RÉEL.

**Source de vérité** : `cerfa_2033c.immobilisations` retourné par `export_donnees_fiscales`. Pour chaque ligne (A à J, sans I) avec une valeur active, ce dict donne les 4 zones (ouverture/acquisitions/cessions/fin) calculées depuis les écritures (BILAN_OUVERTURE débit = ouverture, EVENEMENT débit = acquisition, EVENEMENT crédit = cession).

**Convention Teledec spécifique 2033-C** : lettres A-H = lignes 1-8 (incorporelles, terrains, constructions, etc.), **lettre J = ligne 9 (Immobilisations financières — Teledec saute la lettre I qui ressemble au chiffre 1)**, ligne K = TOTAL (calculé auto).

**INTERDIT** : appliquer une heuristique uniforme du genre "déplacer XX2 vers XX0". Une acquisition réelle de l'exercice (immo achetée pendant N) doit RESTER en colonne acquisitions (XX2). Ne déplacer en ouverture (XX0) que ce qui est réellement reporté de N-1 — info portée par le plan généré.

**INTERDIT** : saisir manuellement le total ligne K (zones 490/492/494/496). Teledec calcule auto la somme des autres lignes.

**INTERDIT** : confondre zones AJ/BJ (ligne J = Immo financières) avec un "total". Ce sont les zones de la ligne 9 — le total est en ligne K.

**Procédure** :

1. [Chrome] Ouvrir le formulaire 2033-C ("Compléter")
2. [Chrome/JS] Exécuter `script_correction_2033C` dans la console — il logge le PLAN ligne par ligne (pour Soeurise 2025 typiquement : ligne J = SCPI ouverture 500 032, acquisitions 0, cessions 0)
3. [Chrome] Pour chaque ligne du plan, saisir au CLAVIER RÉEL via Claude in Chrome :
   * `find` le champ par son nom (ex: `[name="AJ-format"]`)
   * `left_click` dessus
   * `type` la valeur
   * Cliquer directement sur le champ suivant (PAS de Tab)
4. [Chrome] Cliquer **"Sauvegarder et retour"**
5. [Chrome] **VÉRIFIER LA PERSISTANCE** : rouvrir le 2033-C, confirmer que les valeurs sont bien là, que la zone D? affiche le `fin` attendu, et que la ligne K (TOTAL, zones 490/492/494/496) est cohérente.
6. NE PAS cocher Néant si une ligne du plan est active.

### Étape 3b : Corriger 2033-D (script_correction_2033D + saisie clavier)

**ATTENTION — refonte 04/05/2026** : les zones provisions sont RECALCULÉES par Teledec après save. Toute saisie via JS dispatchEvent est écrasée. Seule la **saisie clavier réelle** persiste (find + left_click + type via Claude in Chrome). Le script JS ne fait donc plus que LOGGER les valeurs cibles + saisir les déficits. Les provisions sont à saisir au clavier ligne par ligne.

**Source de vérité** : `cerfa_2033d.provisions` retourné par `export_donnees_fiscales`. Pour chaque ligne (A à G) avec une valeur active, ce dict donne les 4 zones (ouverture/dotation/reprise/fin) calculées correctement depuis les écritures comptables (séparation BILAN_OUVERTURE = ouverture, EVENEMENT crédit = dotation, EVENEMENT débit = reprise).

**INTERDIT** : appliquer une heuristique uniforme du genre "toujours déplacer XX2 vers XX0". Une provision NOUVELLE (= dotation pure de l'exercice, sans report N-1) doit RESTER en colonne dotation (XX2) et NE PAS être déplacée en colonne ouverture (XX0). Le script de correction donne les valeurs correctes ligne par ligne — les suivre à la lettre.

**INTERDIT** : saisir manuellement le total ligne H (zones 680/682/684/686). Teledec recalcule automatiquement la somme des autres lignes. Toute saisie sur la ligne H sera incohérente.

**Procédure** :

1. [Chrome] Ouvrir le formulaire 2033-D ("Compléter")
2. [Chrome/JS] Exécuter `script_correction_2033D` dans la console — il logge :
   * Pour chaque ligne active (D, G, etc.) : les 4 valeurs cibles avec les noms de champs DOM (ex: `BG-format = 160 (zone 662 dotation)`)
   * L'état DOM courant des mêmes champs (avant ta saisie)
   * La saisie automatique des déficits zones 982/983 (champs PG/PH non recalculables)
3. [Chrome] Pour chaque ligne du plan, faire au CLAVIER RÉEL via Claude in Chrome :
   * `find` le champ par son nom (ex: `[name="BG-format"]`)
   * `left_click` dessus pour le focus
   * `type` la valeur (ex: `160`)
   * Passer au champ suivant en cliquant directement (PAS de Tab — Tab peut écraser le champ d'à côté)
4. [Chrome] Cliquer **"Sauvegarder et retour"**
5. [Chrome] **VÉRIFIER LA PERSISTANCE** : rouvrir le 2033-D et confirmer que les valeurs saisies sont toujours là. Si une valeur a été écrasée, la re-saisir au clavier réel et re-sauvegarder.
6. [Chrome] VÉRIFIER que les zones D? (DD-format, DG-format, etc.) affichent les `fin` attendues du plan, et que la ligne H total est cohérente.
7. SI le plan ne contient AUCUNE ligne (CERFA dit pas de provisions) ET pas de déficits : cocher Néant (checkbox RQ) + confirmer.

### Étape 3c : Valider 2033-A (script_correction_2033A)

**Refonte 04/05/2026** : la zone 182 (Coût de revient des immobilisations acquises durant l'exercice) reçoit maintenant la valeur **calculée** (pas hardcodée à 0). Le script lit `cerfa_2033a.acquisitions_exercice_182.montant` (= total col B du 2033-C, somme des acquisitions de l'exercice toutes lignes confondues).

Deux cas selon la valeur cible :
- **Cible = 0** (pas d'acquisition dans l'exercice, cas Soeurise 2025) : le script tente l'effacement JS. Si Teledec écrase au save, Cowork doit re-saisir au clavier réel (`find` + `left_click` + Backspace pour vider).
- **Cible > 0** (acquisitions réelles) : le script logge la valeur cible + le sélecteur DOM. Cowork doit saisir au CLAVIER RÉEL via `find` + `left_click` + `type`.

**Procédure** :

1. [Chrome] Ouvrir le formulaire 2033-A ("Compléter")
2. [Chrome/JS] Exécuter `script_correction_2033A` dans la console — il affiche `VALEUR ATTENDUE (CERFA JSON) : <valeur>`
3. Suivre l'action loguée (effacement automatique si valeur=0, ou plan de saisie clavier si >0)
4. [Chrome] VÉRIFIER visuellement :
   * Zone 040 (Immobilisations financières brut) = valeur attendue
   * Zone 182 = valeur attendue (cohérente avec total col B 2033-C)
   * Bilan équilibré (Total actif net = Total passif)
5. [Chrome] Cliquer **"Sauvegarder et retour"** → statut "Renseigné" (vert)
6. [Chrome] **VÉRIFIER PERSISTANCE** zone 182 après save : si Teledec a écrasé, re-saisir au clavier réel.

## Étape 4 : Saisir 2033-B (script_2033B)

- [Chrome] Ouvrir le formulaire 2033-B ("Compléter")
- [Chrome] PRÉREQUIS : Cliquer sur "Lignes de réintégration" et "Lignes de déduction" pour ouvrir les tableaux dynamiques. Les champs de détail ne sont PAS dans le DOM tant que ces liens ne sont pas cliqués. NOTE : Le script v3.0 tente d'ouvrir ces liens automatiquement (recherche par texte), mais si les liens ne sont pas trouvés, les ouvrir MANUELLEMENT avant d'exécuter le script.
- [Chrome/JS] Exécuter `script_2033B` dans la console
- Le script v3.0 :
  1. Tente d'ouvrir les tableaux dynamiques (liens "Lignes de réintégration/déduction")
  2. Saisit les réintégrations dans `repetition2033BReintegrations[i].LA` / `.LB-format`
  3. Saisit les déductions dans `repetition2033BDeductions[i].MA` / `.MB-format`
  4. Saisit les déficits imputés dans `EP-format` (zone 360)
  5. Affiche les VALEURS ATTENDUES et une checklist de vérification
- [Chrome] VÉRIFIER dans la console :
  * Aucun message "CHAMP MANQUANT" (si présent = le tableau n'était pas ouvert)
  * Les valeurs attendues correspondent à ce qui est affiché
- [Chrome] VÉRIFIER visuellement (VALIDATION OBLIGATOIRE vs CERFA JSON) :
  * Zone 330 : réintégrations = valeur attendue CERFA JSON
  * Zone 350 : déductions = valeur attendue CERFA JSON
  * Zone 360 : déficits imputés = valeur attendue CERFA JSON
  * Zone 324 : IS (normalement auto-rempli par l'import)
- **SI les valeurs ne correspondent PAS** : NE PAS sauvegarder, investiguer l'écart
- [Chrome] Cliquer **"Sauvegarder et retour"**
- **APRÈS sauvegarde** : ré-ouvrir le 2033-B et vérifier que les valeurs sont bien persistées. Les scripts v6+ utilisent une simulation clavier qui persiste après save, mais une vérification reste prudente (au cas où Teledec changerait son framework ou un bug se glisserait).

## Étape 5 : Marquer Néant les formulaires vides (3 scripts séparés)

Pour CHAQUE formulaire néant, utiliser le script SPÉCIFIQUE :

| Formulaire | Script               | Checkbox |
|------------|----------------------|----------|
| 2033-E     | `script_neant_2033E` | DB       |
| 2033-G     | `script_neant_2033G` | auto     |
| 2069-RCI   | `script_neant_2069RCI` | AB     |

Pour chaque formulaire :
- [Chrome] Ouvrir le formulaire
- [Chrome/JS] Exécuter le script néant correspondant dans la console
- Le script v3.0 :
  * Détecte automatiquement la checkbox néant (recherche par label + fallback noms connus)
  * Coche la checkbox
  * Attend la modale DOM et clique le bouton "Oui" (PAS window.confirm)
- [Chrome] VÉRIFIER pour le 2033-E :
  * Les données importées (valeur ajoutée) doivent être EFFACÉES
  * Si elles restent : décocher puis recocher manuellement
- [Chrome] Cliquer **"Sauvegarder et retour"**
- Vérifier que le statut passe à "Renseigné à néant" (vert)

## Étape 6 : Saisir 2033-F (script_2033F)

C'était le formulaire le plus chronophage (~40 champs, 1h+ manuellement). Le script remplit TOUT en une seule exécution.

- [Chrome] Ouvrir le formulaire 2033-F ("Compléter")
- [Chrome/JS] Exécuter `script_2033F` dans la console
- Le script v3.0 automatiquement :
  * Décoche Néant (GS) si coché (gère la modale DOM "Oui")
  * Ajoute des lignes d'associés si plus de 2 PP
  * Remplit CHAQUE associé (nom, %, parts, naissance, adresse, pays)
  * Les adresses proviennent du CERFA JSON (clé "ligne1" du KBIS)
  * Remplit les 6 totaux (zones 901-906)
  * Affiche l'adresse attendue de chaque associé en console
- [Chrome] VÉRIFIER dans la console les logs de chaque associé
- [Chrome] VÉRIFIER visuellement (VALIDATION OBLIGATOIRE vs CERFA JSON) :
  * Chaque associé : nom, %, parts, date naissance
  * **Adresses** : vérifier que chaque adresse correspond au CERFA JSON (JAMAIS inventer ou deviner une adresse si le champ est vide)
  * ATTENTION : vérifier que les noms/adresses n'ont pas débordé (pas de concaténation dans un même champ — c'était le bug Tab)
  * Totaux : zone 905 (total associés) et 906 (total parts) remplis
- [Chrome] Cliquer **"Sauvegarder et retour"**

## Étape 7 : Recalcul 2065 + annexe (script_2065_annexe)

**MÉCANISME** : Le 2065 ne se met PAS à jour automatiquement. Après correction du 2033-B, son statut passe à "Incohérences" puis "À compléter". Il faut l'ouvrir, cocher l'annexe néant, et sauvegarder pour forcer le recalcul.

- [Chrome] Vérifier le statut du 2065 sur la page principale :
  * Si "Incohérences" : sauvegarder d'abord le 2033-B une dernière fois
  * Attendre que le statut du 2065 passe à "À compléter"
- [Chrome] Ouvrir le formulaire 2065
- [Chrome] Vérifier page 1 : le résultat fiscal doit correspondre à la valeur attendue du CERFA JSON (affichée en console par le script). **SI le résultat est différent** : STOP, le 2033-B est incomplet (zones 330/350/360)
- [Chrome/JS] Exécuter `script_2065_annexe` dans la console
- Le script scrolle vers l'annexe 2065-BIS et coche la case néant (distributions)
- [Chrome] Cliquer **"Sauvegarder et retour"** (force le recalcul)
- [Chrome] Vérifier que le statut passe à "Renseigné" (vert)

**PIÈGES** :
- **Règle d'or : JAMAIS modifier les champs calculés du 2065** (bénéfice, IS, résultat fiscal)
- Si le résultat est faux : corriger le 2033-B (source de vérité), PAS le 2065
- Sans la case néant annexe 2065-BIS, le 2065 reste "À compléter"
- Si "Incohérences" persiste : vérifier que le 2033-B est COMPLET (zones 330, 350, 360)

## Étape 8 : Vérification finale

- [Chrome] Retourner à la liste des formulaires (page principale de la déclaration)
- [Chrome] Prendre un screenshot de la page principale
- [Chrome] Utiliser la CHECKLIST de `verification.md` pour vérifier TOUS les points

## Étape 11 : Archivage automatique de la liasse PDF (OBLIGATOIRE en mode transmission)

**AVERTISSEMENT : NE PAS IMPROVISER.** Utiliser EXACTEMENT le script §11c ci-dessous, sans modification. La session #6 a montré que toute improvisation (FormData, XMLHttpRequest, approche alternative) échoue avec une erreur CORS. La raison technique : le serveur backend autorise les requêtes JSON cross-origin (Content-Type: application/json) mais PAS les requêtes FormData multipart. Le script §11c utilise arrayBuffer() + btoa() + POST JSON avec `contenu_base64` — c'est le SEUL format qui fonctionne.

Cette étape est AUTOMATIQUE — ne PAS demander confirmation supplémentaire. Le navigateur est sur teledec.fr (session authentifiée). On utilise un script JS qui télécharge le PDF et l'envoie directement au backend via une URL jetable.

**IMPORTANT : Générer l'URL d'upload (11a) au DERNIER MOMENT**, juste avant d'exécuter le script (11c). L'URL est à usage unique : si le script échoue et doit être ré-exécuté, rappeler `generer_upload_url_liasse` pour obtenir une nouvelle URL. Ne jamais réutiliser une URL après un échec.

### 11a. Obtenir l'URL d'upload (MCP) — juste avant 11c

- Appeler le tool MCP `generer_upload_url_liasse` avec l'exercice
- Le serveur retourne une `upload_url` (token jetable, valide 30 min, **usage unique**)
- CONSERVER cette URL pour l'étape 11c immédiate

### 11b. Identifier l'URL de téléchargement de la liasse sur Teledec

- [Chrome] Sur la page de la déclaration, cliquer **"Autres actions"** (pas "Imprimer")
- [Chrome] Repérer le lien "Télécharger la liasse" (ou "Exporter PDF")
- [Chrome] NE PAS cliquer dessus — copier l'URL du lien (clic droit > copier l'adresse)
- Si pas de lien visible : chercher un bouton ou lien contenant "PDF", "liasse", "export"
- CONSERVER cette URL (ex: `/declarations/12345/pdf` ou similaire)

### 11c. Exécuter le script JS d'archivage dans la console Chrome

- [Chrome/JS] Exécuter ce script en remplaçant les 2 variables :

```javascript
(async () => {
  const TELEDEC_PDF_URL = '/declarations/XXXXX/pdf';  // URL identifiee en 11b
  const UPLOAD_URL = 'https://head-soeurise-web.onrender.com/api/v1/teledec/liasse/upload/TOKEN';  // URL de 11a
  const EXERCICE = 2025;  // annee de l'exercice

  try {
    // 1. Telecharger le PDF depuis Teledec (cookies de session automatiques)
    const resp = await fetch(TELEDEC_PDF_URL);
    if (!resp.ok) throw new Error('Telechargement echoue: ' + resp.status);
    const buf = await resp.arrayBuffer();
    console.log('PDF telecharge:', buf.byteLength, 'octets');

    // 2. Convertir en base64
    const bytes = new Uint8Array(buf);
    let binary = '';
    for (let i = 0; i < bytes.length; i++) binary += String.fromCharCode(bytes[i]);
    const b64 = btoa(binary);
    console.log('Base64:', b64.length, 'caracteres');

    // 3. Envoyer au backend (token dans l URL, pas de header Authorization)
    // STRUCTURE OBLIGATOIRE : { fichier: { contenu_base64, nom_fichier, type_mime } }
    // NE PAS envoyer contenu_base64 et nom_fichier a la racine — erreur 422
    let uploadResp;
    try {
      uploadResp = await fetch(UPLOAD_URL, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          fichier: {
            contenu_base64: b64,
            nom_fichier: 'liasse-fiscale-' + EXERCICE + '.pdf',
            type_mime: 'application/pdf'
          }
        })
      });
    } catch (netErr) {
      // Erreur reseau / CORS / DNS : aucune reponse HTTP recue
      console.error('UPLOAD_NETWORK_ERROR:', netErr.message,
                    '(causes possibles : CORS bloque, DNS, deploy en cours, URL incorrecte)');
      return;
    }

    // Reponse HTTP recue — diagnostiquer status et body meme si !ok
    const status = uploadResp.status;
    const bodyTxt = await uploadResp.text();
    let body;
    try { body = JSON.parse(bodyTxt); } catch { body = bodyTxt; }

    if (uploadResp.ok && body && body.success) {
      console.log('ARCHIVAGE REUSSI [' + status + ']:', body.message);
    } else if (status === 404) {
      console.error('UPLOAD_TOKEN_INTROUVABLE [404]:', JSON.stringify(body),
                    '(le token n existe pas — verifier l URL ou regenerer via generer_upload_url_liasse)');
    } else if (status === 409) {
      console.error('UPLOAD_TOKEN_DEJA_UTILISE [409]:', JSON.stringify(body),
                    '(le token a deja servi a uploader un fichier — generer une nouvelle URL si retry necessaire)');
    } else if (status === 410) {
      console.error('UPLOAD_TOKEN_EXPIRE [410]:', JSON.stringify(body),
                    '(token > 30 min — generer une nouvelle URL)');
    } else if (status === 413) {
      console.error('UPLOAD_PDF_TROP_GROS [413]: PDF =', buf.byteLength, 'octets');
    } else if (status === 422) {
      console.error('UPLOAD_FORMAT_INVALIDE [422]:', JSON.stringify(body),
                    '(structure JSON incorrecte — verifier { fichier: { contenu_base64, nom_fichier, type_mime } })');
    } else {
      console.error('UPLOAD_ECHEC [' + status + ']:', JSON.stringify(body));
    }
  } catch (e) {
    console.error('ERREUR ARCHIVAGE (inattendue):', e.message, e.stack);
  }
})();
```

### 11d. Vérifier le résultat

- [Chrome] Lire la console : si "ARCHIVAGE REUSSI" → succès
- Si "ECHEC" ou "ERREUR" → STOP, remonter l'erreur à l'utilisateur
- Confirmer à l'utilisateur : "La liasse fiscale a été archivée et publiée dans votre espace abonné."
- Ne PAS considérer la Phase 2 comme terminée sans archivage réussi.

## Fin de la Phase 2

La télétransmission (Phase 3) n'est PAS implémentée. Ne JAMAIS procéder à la télétransmission.

Indiquer au client que :
- La liasse est prête sur Teledec
- La liasse PDF a été archivée dans l'espace abonné
- La télétransmission doit être effectuée manuellement par le client quand il sera prêt (hors scope automatisation)
- Après télétransmission, l'accusé de réception DGFiP devra être archivé via la commande dédiée (tool MCP `archiver_ar_teledec`)

## Bonnes pratiques de saisie

### Navigation entre champs
- TOUJOURS cliquer directement sur le champ cible
- JAMAIS Tab dans un textarea (libellé, nom, adresse)
- Tab est OK uniquement dans les champs numériques simples (input type number)
- En cas de doute : cliquer

### Correction d'erreur
- Effacer COMPLÈTEMENT le champ avant de re-saisir (sélectionner tout + supprimer)
- Ne jamais saisir par-dessus une valeur existante (risque de concaténation)
- Screenshot après correction pour vérifier

### Si redirection inattendue
- Après sauvegarde, Teledec peut rediriger vers "Mes déclarations"
- Solution : re-cliquer "Compléter" sur la déclaration
- Vérifier le titre de la page après chaque action
