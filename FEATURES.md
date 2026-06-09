# Product Planner — Gantt · Historique des features demandées

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
