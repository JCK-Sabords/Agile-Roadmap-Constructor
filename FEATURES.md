# Product Planner — Roadmap agile · Historique des features demandées

> Projet : application de planification Gantt locale (HTML/CSS/JS, stockage localStorage)
> Fichier : `C:\Users\jean-christophe.kour\planner\index.html`

---

## 1. Application initiale

**Demande :**
> Développe une application de planification de type Gantt pour mon produit, qui tourne, stockée, visualisée et hébergée en local sur mon PC.

**Features livrées :**
- Application single-file `index.html` — aucune dépendance serveur, stockage `localStorage`
- **Configuration globale** : date de départ (défaut = aujourd'hui) + nombre de développeurs dans l'équipe
- **Saisie d'un projet** : nom, charge totale (semaines), % d'avancement à date, nombre de devs requis
- Priorité par défaut = dernière position
- **Vue Gantt** avec colonnes de semaines, barres colorées par projet, overlay de progression
- **Glisser-déposer vertical** (drag & drop) sur les lignes pour modifier l'ordre de priorité → recalcul automatique de la planification
- Algorithme de scheduling : assignation greedy par priorité avec parallélisme réel (plusieurs projets peuvent tourner en même temps si la capacité dev le permet)
- Colonnes sticky à gauche : poignée ≡, nom + badge priorité, nb devs, avancement, semaines restantes, dates début/fin
- Persistance automatique à chaque modification

---

## 2. Corrections de bugs (session 1)

**Demande :**
> Le Gantt ne s'affiche pas bien. Le message "aucun projet pour l'instant" s'affiche alors que j'ai déjà des projets créés.

**Bugs corrigés :**
- **Bug 1 — Empty state toujours visible** : la classe CSS `.hidden` n'était pas définie globalement (seul `.overlay.hidden` existait) → ajout de `.hidden { display: none !important }`
- **Bug 2 — Barres Gantt invisibles** : `WEEK_PX` lu via `getComputedStyle('--week-w')` retournait `NaN` selon l'environnement → valeur hardcodée `const WEEK_PX = 64`
- **Bug 3 — Classe CSS corrompue** : concaténation sans espace `"bar-cell" + "wk-today-col"` → `"bar-cellwk-today-col"` → correction de l'interpolation

---

## 3. Affichage du % dans les barres + règle métier devs insuffisants

**Demande :**
> Dans le Gantt, dans chaque barre, je veux que soit vu le % d'avancement du projet à date. Par exemple : "projet 1 - 60%".
> Il y a un problème dans le calcul : il faut que tous les devs soient occupés. 100% des devs doivent être occupés en permanence.

**Features livrées :**
- **Label dans la barre** : `Nom du projet — 60%` (masqué si barre trop étroite)
- **Algorithme bin-packing événementiel** : à chaque libération de dev, remplissage de la capacité disponible par priorité, en sautant les projets qui ne rentrent pas pour affecter des projets plus petits → 0 dev idle si du travail est disponible

---

## 4. Navigation temporelle + ligne aujourd'hui + semaines lundi + scaling proportionnel

**Demande :**
> Ajoute un ascenseur horizontal qui permet de se déplacer dans le temps, y compris en arrière.
> La date d'aujourd'hui doit être symbolisée par une ligne verticale rouge.
> Les semaines commencent par le lundi.
> Nouvelle règle métier : si il n'y a pas assez de devs disponibles pour un job prioritaire, il faut tout de même faire le job, mais il durera proportionnellement plus longtemps. Par exemple, si 6 devs sont nécessaires sur 2 semaines, mais que seuls 3 sont disponibles, alors on prendra 4 semaines avec ces 3 devs.

**Features livrées :**
- **Scrollbar horizontale toujours visible** (`overflow-x: scroll`) pour naviguer dans le temps
- **8 semaines de passé** affichées avant la date de départ + buffer futur — scroll auto-centré sur aujourd'hui au chargement
- **Ligne rouge verticale** traversant toutes les lignes à la colonne "aujourd'hui" (div `.today-line` injectée dans chaque cellule)
- **Semaines alignées sur le lundi** : fonction `getMondayOf(dateStr)` recalcule le lundi de la semaine contenant `startDate`
- **Durée proportionnelle si devs insuffisants** : le projet de plus haute priorité démarre immédiatement avec les devs disponibles, durée = `⌈ (semaines × devs_requis) / devs_disponibles ⌉`. Indicateur visuel ambre + chip `↗` dans la colonne "Restant"

---

## 5. Deadline, Business Value, Priorité globale P1–P4 + logique de couleurs

**Demande :**
> Ajoute une configuration pour chaque projet : date limite d'attente pour livraison (deadline).
> Ajoute une autre configuration : la business value.
> Ajoute une autre configuration : la priorité globale (de P1 à P4), et elle définira les couleurs des barres (un dégradé d'une même couleur).
>
> Pour les couleurs :
> - Un dégradé d'une couleur si le projet est à 0% d'avancement
> - Un dégradé d'une autre couleur (dépendant de la priorité globale) si le projet est à plus que 0% : projet démarré
> - Un dégradé de rouge, si d'après la priorité et l'ordre du Gantt, le projet est sensé finir après la date limite d'attente du projet

**Features livrées :**
- **Nouveaux champs projet** :
  - `Priorité globale` : P1 (Critique) / P2 (Haute) / P3 (Normale) / P4 (Basse)
  - `Business Value` : valeur numérique libre (optionnel)
  - `Deadline` : date limite de livraison (optionnel)
- **Logique de couleur des barres** (ordre de priorité d'évaluation) :
  1. Fin planifiée > deadline → 🔴 dégradé rouge alarme
  2. Durée allongée (devs insuffisants) → 🟡 dégradé ambre
  3. progress = 0% → ⬛ dégradé ardoise (non démarré)
  4. progress > 0% → couleur par priorité globale :
     - P1 → dégradé rose-rouge
     - P2 → dégradé orange
     - P3 → dégradé bleu
     - P4 → dégradé violet
- **Trait blanc vertical** dans la barre = position de la deadline à l'intérieur de la plage
- **Nouvelle colonne sticky** `P.` (badge P1–P4 coloré) + colonne `BV`
- **Colonne Dates** enrichie : deadline affichée avec ⚠ rouge si dépassée
- **Chip "Restant"** passe rouge avec ⚠ si projet hors deadline
- Rétrocompatibilité : projets existants en localStorage mis à jour avec les valeurs par défaut

---

## 6. Export / Import

**Demande :**
> Génère aussi un bouton pour exporter, et importer une stack de projets pour revenir à une ancienne version.

**Features livrées :**
- **Bouton Exporter** : génère un fichier `product-planner_YYYY-MM-DD_HHhMM.json` horodaté contenant l'intégralité du state (config + projets + nextId)
- **Bouton Importer** : sélecteur de fichier `.json`, dialogue de confirmation avec résumé (nom du fichier, date d'export, nb de projets, avertissement remplacement), restauration complète du state
- Rétrocompatibilité à l'import : champs manquants comblés avec valeurs par défaut
- Scroll repositionné sur "aujourd'hui" après import
- Input file réinitialisé pour permettre de ré-importer le même fichier

---

## 7. Zoom / Dézoom des colonnes de semaines

**Demande :**
> Ajoute un moyen aussi de zoomer et dézoomer pour que les semaines soient plus ou moins larges.

**Features livrées :**
- **Boutons − / +** dans la barre de configuration avec indicateur de largeur actuelle (ex. `64px`)
- **7 niveaux de zoom** : `24 → 32 → 48 → 64 → 96 → 128 → 180 px` par semaine
- Niveau de zoom persisté dans `localStorage` (survit aux rechargements)
- Scroll recentré sur aujourd'hui à chaque changement de zoom
- **Labels adaptatifs** selon le niveau :
  - ≥ 64 px → `15 juin` (label complet)
  - ≥ 40 px → `15` (jour seul)
  - ≥ 28 px → nom du mois sur la 1ère semaine seulement
  - 24 px → aucun texte (colonnes purement visuelles)

---

## 8. Correction bug zoom — désynchronisation barres / colonnes

**Demande :**
> La dernière feature a un petit bug : quand je dézoome au maximum, le projet est dézoomé, mais les semaines ne le sont plus.

**Bug corrigé :**
- **Cause** : le texte des dates dans les `<th>` (ex. « 15 juin ») forçait les colonnes à s'élargir au-delà de la largeur demandée. Les barres (calculées en JS) utilisaient la bonne taille, les colonnes non → désynchronisation visuelle.
- **Fix** : ajout d'un `<div style="width:${wp}px; overflow:hidden">` wrapper dans chaque `<th>` de semaine → le navigateur ne peut plus élargir la colonne au-delà de la valeur cible.

---

## 9. Correction bug longueur des barres Gantt

**Demande :**
> Le bug de la longueur des barres du Gantt n'est pas résolu. Par exemple, quand il reste 8 semaines de développement à un projet, il doit avoir une longueur de 8 semaines dans le graphique.

**Bug corrigé :**
- **Cause** : les `<td class="wk-cell">` n'avaient pas de largeur inline explicite — le navigateur décidait de leur taille et dérivait par rapport à la valeur JS
- **Fix 1** : `mkEmptyCell()` génère désormais `style="width:Npx;min-width:Npx;max-width:Npx"` sur chaque cellule → largeur verrouillée
- **Fix 2** : la barre utilise `left:3px; right:3px` au lieu de `width:barSpan×WEEK_PX()−6` → elle épouse la cellule colspan réelle sans calcul JS intermédiaire

---

## 10. Date de démarrage par projet (règle métier)

**Demande :**
> Rajoute une règle métier à la création d'un projet : la date de démarrage d'un projet. Normalement cette valeur restera vide, mais si cette valeur est remplie, on ne doit pas commencer ce projet avant la date précisée.

**Features livrées :**
- **Nouveau champ projet** `Date de démarrage` (optionnel) dans le formulaire de création/édition
- Helper `dateToWeek(dateStr)` : convertit une date en numéro de semaine relatif au lundi de la `startDate`
- Le scheduler calcule `minStartWeek` pour chaque projet → un projet ne démarre jamais avant sa date imposée

---

## 11. Ordre de démarrage respectant la priorité (parallélisation conservée)

**Demande :**
> Il faut être en capacité à paralléliser. Mais il ne faut pas commencer le projet N si le projet M (plus prioritaire que N) n'a pas commencé.

**Features livrées :**
- Garde-fou dans la boucle de scheduling : `currentWeek` respecte le `minStartWeek` du projet le plus prioritaire en attente avant d'affecter des devs
- La parallélisation (bin-packing) est conservée : plusieurs projets tournent en parallèle si la capacité le permet, mais jamais un projet moins prioritaire avant le démarrage d'un plus prioritaire

---

## 12. Édition inline par colonne + suppression des poignées ⠿

**Demande :**
> Retire les 6 points à gauche des projets. Cliquer sur les colonnes édite directement la valeur (projet → renommer, P. → liste déroulante priorité, Devs → nb de devs, etc.). Les autres champs restent accessibles via l'icône crayon au survol.

**Features livrées :**
- Suppression de la poignée de drag ⠿ → remplacée par un **badge Ordre** (1→N) sticky et stylé
- **Édition inline** au clic sur chaque colonne visible : `inlineEditName`, `inlineEditPriority`, `inlineEditDevs`, `inlineEditProgress` (clamp 0–100), `inlineEditBV` (0–1000 ou vide), `inlineEditDeadline`
- Sauvegarde sur blur/Enter, annulation sur Escape
- Le drag-to-reorder se fait via la colonne Ordre ou les barres ; le crayon ✎ (au survol) ouvre la modale complète
- Suppression du mini-rectangle BV redondant dans la colonne Projet (colonne BV dédiée déjà présente)

---

## 13. Labels de zoom — priorité aux mois

**Demande :**
> En zoom 48px il faut privilégier les mois (jours seulement si ≥64px). En zoom 24px, afficher les mois même si ça déborde sur les lignes verticales.

**Bug/amélioration livrée :**
- Logique de labels revue : le mois est prioritaire et affiché en début de mois (`getDate() <= 7`)
- Les jours ne s'affichent qu'à partir de 64px
- En dessous de 28px, `overflow:visible` autorise l'écriture du mois par-dessus les lignes verticales

---

## 14. Calcul correct de la durée allongée (capacité semaine par semaine)

**Demande (sur plusieurs itérations) :**
> Le calcul des semaines restantes en cas de durée allongée est faux. Ne pas se baser sur les devs disponibles la 1ère semaine, mais accumuler la capacité semaine par semaine (capacité = X×N, on retire chaque semaine le travail fourni). Tenir compte des devs occupés sur d'autres projets. La durée réelle compte les semaines 0,1,2 = 3 semaines.

**Features livrées :**
- Helper `calculateProjectEndWeek(startWeek, devWeekBudget, devFreeAt, maxParallel)` : simulation semaine par semaine
- Budget = `remainingWeeks × devsNeeded`
- Chaque semaine, capacité ajoutée = `min(devs libres, devsNeeded)` — un projet ne peut absorber que `devsNeeded` devs par semaine (limite de parallélisation)
- Les devs occupés sur d'autres projets sont décomptés via `devFreeAt`
- Comptage inclusif des semaines (`+1`) : travailler sem 0,1,2 = 3 semaines
- Colonne **Restant** affiche `X→Y` (restant initial → nouvelle valeur allongée)

---

## 15. Deadline visible sur la frise chronologique (double ligne verticale)

**Demande :**
> Quand pour un projet nous avons une deadline donnée, il faut que cette deadline soit visible sur la frise chronologique, par exemple avec une double ligne verticale à la date.

**Features livrées :**
- Nouveau style `.deadline-line` : **double ligne verticale rouge** (bordures gauche + droite, 4px d'écart)
- La deadline est calculée en semaine (`dlWeek`) puis dessinée dans la cellule correspondante de la frise
- **Visible même hors de la barre** : si la deadline tombe après la fin planifiée du projet, la ligne apparaît dans une cellule vide de la frise (avant, ce repère blanc n'existait qu'à l'intérieur de la barre)
- `mkEmptyCell(w, tw, dlWeek)` accepte la semaine de deadline pour injecter la ligne au bon endroit
- À l'intérieur de la barre, l'ancien repère blanc (`deadline-flag`) est remplacé par la même double ligne rouge pour la cohérence

---

## 16. Rouge prioritaire sur toute autre couleur en cas de dépassement de deadline

**Demande :**
> Si un projet dépasse une deadline, il faut que le projet soit en rouge. Même s'il y a allongement de durée ou toute autre raison. Le rouge est prioritaire.

**Fix livré :**
- Dans `getBarGradient`, l'évaluation "hors deadline → rouge" passe **en premier**, avant la détection "durée allongée → ambre"
- Un projet scaled ET hors deadline est rouge (pas ambre)
- Ordre final : 1) Hors deadline → rouge · 2) Scaled → ambre · 3) Fini → vert · 4) Non démarré → gris · 5) En cours → couleur par priorité

---

## 17. Date de démarrage visible sur la frise chronologique

**Demande :**
> Si une date de démarrage obligatoire est définie pour un projet, la rendre visible sur la frise sur la ligne du projet concerné.

**Features livrées :**
- Nouveau style `.startdate-line` : **ligne verticale verte pointillée** (2px dashed #16a34a), visuellement distincte de la deadline rouge
- Calcul de `sdWeek` et `sdDayOffset` (même logique que la deadline : `dateToWeek` + offset sub-semaine)
- La ligne s'affiche à la position exacte du jour (pas seulement le lundi de la colonne)
- Visible dans les cellules vides ET à l'intérieur de la barre si la contrainte tombe dans la plage du projet
- `mkEmptyCell` accepte maintenant `sdWeek` et `sdDayOffset` en plus des paramètres deadline

---

## 18. Colonnes gauches figées (split-panel)

**Demande :**
> Toute la zone à gauche de la frise doit rester statique quand on utilise l'ascenseur horizontal.

**Features livrées :**
- Refonte structurelle en **split-panel** : deux tables côte à côte (`#gantt-left-table` dans `.gantt-frozen` + `#gantt-right-table` dans `.gantt-wrapper`)
- `.gantt-frozen` : `overflow:hidden`, shadow droite, z-index 10 — jamais scrollé
- `.gantt-wrapper` : `overflow-x:scroll` — seule partie qui bouge horizontalement
- `render()` génère deux jeux de lignes synchronisés (`rowsLeft` / `rowsRight`) avec le même `data-id`
- Drag-and-drop rebranché sur `#gantt-body-left` uniquement ; `mirrorRowClass()` propage les classes visuelles aux deux panels
- Auto-scroll initial recalculé sans offset sticky

---

## 19. Date de démarrage dans la colonne Dates/Deadline

**Demande :**
> Si un projet a une date de démarrage définie, mieux visualiser cette date dans la colonne Dates/Deadline avec une émoji et mettre la date en vert gras.

**Features livrées :**
- Affichage `▶︎ dém : JJ mois AA` en **vert (#16a34a) et gras** dans la cellule Dates/Deadline
- Style `.startdate-line` changé en **ligne pleine** avec pseudo-éléments `::before` / `::after` formant deux flèches droite (haut et bas de la ligne)
- Emoji changé de 🟢 à ▶︎ (plus cohérent avec l'idée de démarrage)

---

## 20. Renommage de l'application

**Demande :**
> Change le nom de l'outil en "Product Planner - Roadmap agile".

**Features livrées :**
- Titre de l'onglet navigateur : `Product Planner — Roadmap agile`
- Topbar logo mis à jour
- README.md et FEATURES.md mis à jour en conséquence

---

## 21. Resserrement des colonnes gauches

**Demande :**
> Resserre les colonnes de la partie gauche pour laisser plus de place au graphe à droite.

**Features livrées :**
- Réduction de toutes les colonnes figées (total : 824 px → 557 px, soit −267 px libérés pour le Gantt) :
  - Ordre : 50 → 30 px
  - Projet : 210 → 80 px (tronquage `text-overflow:ellipsis`)
  - GP : 52 → 46 px
  - Devs : 64 → 54 px
  - Prog : 110 → 92 px
  - Restant : 82 → 72 px
  - BV : 56 → 48 px
  - Dates : 200 → 135 px
- Offsets cumulatifs `--l-*` recalculés en conséquence

---

## 22. Sous-titre produit éditable dans la topbar

**Demande :**
> Ajoute un contenu modifiable par le user après "Roadmap agile", qui ait l'air écrit d'une traite avec le reste du titre.

**Features livrées :**
- Span `#product-subtitle` avec `contenteditable="true"` inséré après "Roadmap agile"
- `display:inline-block; min-width:90px` pour garantir une zone cliquable même à vide
- Placeholder `· votre produit` via CSS `::before` (opacité 0.28, disparaît dès qu'il y a du contenu)
- Soulignement subtil au survol (`box-shadow`) pour signaler l'interactivité
- Aucun outline au focus — police, couleur et poids identiques au reste du span → rendu seamless
- Persisté en `localStorage` (clé `product-subtitle`), restauré au chargement
- `Enter` déclenche `blur()` (pas de saut de ligne)

---

## 23. Séparateur flèche → dans la topbar

**Demande :**
> Entre "Product Planner" et "Roadmap agile", remplace le tiret par une flèche vers la droite.

**Features livrées :**
- Remplacement de `—` par `→` dans la topbar logo

---

## 24. Undo — Ctrl+Z

**Demande :**
> Ajouter un undo (Ctrl+Z) pour annuler la dernière action.

**Features livrées :**
- `UNDO_STACK` (tableau en mémoire, limite 30 entrées) + `pushUndo()` / `undoState()`
- `pushUndo()` appelé juste avant chaque mutation réelle dans les 9 fonctions : `onCfgChange`, `saveProject`, `deleteProject`, `inlineEditName/Priority/Devs/Progress/BV/Deadline`, `onDrop`
- Raccourci `Ctrl+Z` (ou `Cmd+Z` Mac) via `keydown` sur `document` — ignoré si focus dans un champ de texte ou `contenteditable`
- Toast "Annulé ✓" affiché 1,8 s en bas de l'écran après chaque undo
- `showToast()` réutilisable (timeleur auto-reset)

---

## 25. Tooltip hover sur les barres Gantt

**Demande :**
> Voir les infos du projet au survol d'une barre Gantt sans ouvrir la modale.

**Features livrées :**
- `<div id="gantt-tooltip">` positionné en `position:fixed`, suivi du curseur via `mousemove` sur `#gantt-wrapper`
- Contenu : nom (gras), dates début→fin planifiées, avancement %, devs requis, deadline (rouge ⚠ si dépassée), BV si renseignée
- Dates stockées en `data-start` / `data-end` sur chaque `.g-bar` lors du `render()`
- Disparaît sur `mouseleave` du panel Gantt ou quand le curseur quitte une barre
- Style dark card (fond `#1e293b`, `border-radius:9px`, `opacity` animée)

---

## 26. Barre de config masquable (révélée au survol)

**Demande :**
> Masquer la barre de config (date départ, devs, zoom) et la révéler au survol d'une zone de 3px en haut de la frise.

**Features livrées :**
- `.cfg-bar` démarre avec la classe `cfg-hidden` (`max-height:0; opacity:0; pointer-events:none`)
- Transition CSS fluide sur `max-height`/`opacity`/`padding`
- `.cfg-handle` (4px, dégradé indigo) positionné juste sous la topbar — hover → révèle la barre
- La barre reste visible tant que la souris est dessus ; se rétracte après 250 ms de mouseout (évite les fermetures involontaires)
- Bouton **◆ Jalons** ajouté dans la barre de config

---

## 27. Milestones — jalons transversaux

**Demande :**
> Jalons datés (◆ sur la frise, partagés entre projets) pour les releases, sprint ends, comités...

**Features livrées :**
- `state.milestones[]` : `{id, name, date, color}` + `state.nextMsId`
- `msMap` calculé à chaque `render()` : semaine → [{name, color, dayOff}]
- **Header frise** : ◆ `ms-hdr-pin` positionné au jour exact avec nom tronqué en dessous
- **Toutes les lignes** : ligne verticale pointillée colorée via `mkMsLines()` dans `mkEmptyCell()` ET dans les barres Gantt
- Modal **◆ Jalons** : liste triée par date avec swatches de couleur + bouton supprimer, formulaire d'ajout (nom + date + `<input type="color">`)
- Undo compatible (`pushUndo()` sur ajout et suppression)

---

## 28. Correction bug jalons — overlay absolu

**Demande :**
> La ligne du jalon est décalée quand elle arrive sur certains projets.

**Cause identifiée :**
- Les lignes `ms-line` étaient injectées cellule par cellule dans le tableau. Pour les projets dont la barre commence avant la zone visible (`barStart < wStart`), la position était calculée depuis le début hors-écran de la barre → décalage visible proportionnel au dépassement.
- Tentative de fix `cellStart = Math.max(barStart, wStart)` insuffisante car la largeur réelle des colonnes (avec bordures) diffère de `WEEK_PX()`.

**Fix livré :**
- Suppression de toutes les `ms-line` par cellule et de `mkMsLines()`
- **Overlay absolu unique** : `<div id="ms-gantt-overlay">` en `position:absolute; top:0; left:0; height:9999px` enfant de `.gantt-wrapper` (qui est `position:relative`)
- La position de chaque ligne est lue depuis le DOM réel après rendu : `th.getBoundingClientRect().left - wrapperRect.left + scrollLeft + dayOff * th.width / 7` → alignement pixel-perfect indépendant des bordures/padding
- La ligne traverse toute la hauteur du Gantt en un seul élément, sans dépendance aux cellules individuelles

---

## 29. Jalons éditables inline

**Demande :**
> Rendre les jalons éditables (couleur, nom et date) directement dans l'onglet Jalons.

**Features livrées :**
- Chaque ligne de la liste jalons affiche désormais trois champs directement éditables :
  - **`<input type="color">`** remplace le simple swatch — clic → color picker natif, changement immédiat
  - **`<input type="text">`** pour le nom — sauvegarde au blur ou à la touche Entrée
  - **`<input type="date">`** pour la date — sauvegarde à la sélection
- Nouvelle fonction `updateMilestone(id, field, value)` : ignore les valeurs vides ou inchangées, `pushUndo()` avant toute mutation → Ctrl+Z compatible
- Styles doux : fond `var(--surface2)` au hover/focus, invisible au repos — rendu cohérent avec l'édition inline du reste de l'interface

---

## Stack technique

| Élément | Choix |
|---|---|
| Runtime | Navigateur (aucun serveur requis) |
| Stockage | `localStorage` (clé `product-planner-v3`) |
| Rendu | HTML table avec colonnes sticky + barres `position:absolute` |
| Algorithme | Bin-packing événementiel greedy par priorité |
| Serveur dev | `npx serve -p 5555 .` (optionnel) |
| Dépendances | **Aucune** — vanilla HTML/CSS/JS |

---

## Fichiers

```
C:\Users\jean-christophe.kour\planner\
├── index.html      ← application complète
├── README.md       ← documentation utilisateur
└── FEATURES.md     ← ce fichier (historique des itérations)
```

> ⚙️ **Convention** : à chaque développement ajouté, mettre à jour le README (`README.md`) et ajouter une itération à ce fichier.
