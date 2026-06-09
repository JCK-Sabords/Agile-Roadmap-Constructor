# 🚀 Product Planner — Roadmap agile

Une application de **planification de type Gantt** locale, simple et efficace pour visualiser et gérer vos projets en temps réel.

## ✨ Caractéristiques principales

- 📊 **Gantt interactif** avec barres colorées et glisser-déposer vertical
- 🎯 **Système de priorités** (P1-P4) avec codes couleur automatiques
- 📅 **Gestion des deadlines** avec alertes en rouge + double ligne verticale sur la frise
- 🗓️ **Date de démarrage** optionnelle par projet (un projet ne démarre jamais avant) — visible dans la colonne Dates et sur la frise (ligne verte avec flèches)
- ✏️ **Sous-titre produit éditable** dans la topbar (clique sur "· votre produit" pour personnaliser, persisté en localStorage)
- 💬 **Tooltip hover** sur les barres Gantt — nom, dates, avancement, devs, deadline (rouge si dépassée)
- ↩️ **Undo (Ctrl+Z)** — annule la dernière modification (30 niveaux, toast de confirmation)
- ◆ **Milestones éditables** — jalons transversaux datés (◆ dans le header + ligne overlay sur toutes les lignes) ; nom, date et couleur éditables inline dans la modale
- 🙈 **Barre de config masquable** — révélée au survol d'une zone de 4px sous la topbar
- 🔄 **Scheduling intelligent** avec 100% d'utilisation des développeurs et parallélisation
- ✏️ **Édition inline** — clic direct sur les colonnes (nom, priorité, devs, avancement, BV, deadline)
- 📈 **Durée allongée précise** calculée semaine par semaine selon la capacité réelle
- 🎨 **Zoom dynamique** (7 niveaux) pour explorer votre planning
- 💾 **Persistance localStorage** — aucune base de données requise
- 📤 **Export/Import JSON** pour sauvegarder et restaurer vos données
- ⚡ **Validation stricte** des champs obligatoires
- 🌐 **Aucune dépendance** — pur HTML/CSS/JavaScript

## 🚀 Démarrage rapide

### Option 1 : Directement dans votre navigateur (recommandé)

```bash
# 1. Cloner le repo
git clone https://github.com/JCK-Sabords/Agile-Roadmap-Constructor.git
cd Agile-Roadmap-Constructor

# 2. Ouvrir dans votre navigateur
# Sur Windows:
start index.html

# Sur Mac:
open index.html

# Sur Linux:
xdg-open index.html
```

### Option 2 : Avec un serveur local

```bash
# 1. Cloner le repo
git clone https://github.com/JCK-Sabords/Agile-Roadmap-Constructor.git
cd Agile-Roadmap-Constructor

# 2. Lancer le serveur (Node.js requis)
npx serve -p 5555 .

# 3. Ouvrir http://localhost:5555 dans votre navigateur
```

## 📖 Utilisation

### Configuration initiale

1. **Définir la date de départ** — par défaut, c'est aujourd'hui
2. **Entrer le nombre de développeurs** dans l'équipe
3. Tous les projets seront planifiés à partir de ces paramètres

### Ajouter un projet

Cliquez sur **+ Nouveau projet** et remplissez :

- **Nom du projet** ⭐ (obligatoire)
- **Charge totale** (semaines) ⭐ (obligatoire)
- **Avancement actuel** (%)
- **Développeurs nécessaires** ⭐ (obligatoire)
- **Priorité globale** (P1–P4) ⭐ (obligatoire)
- **Business Value** (optionnel)
- **Date de démarrage** (optionnel — le projet ne commencera pas avant cette date)
- **Deadline** (optionnel — affichée en rouge si dépassée)

### Édition rapide (inline)

Cliquez directement sur une cellule du tableau pour la modifier sans ouvrir la modale :

- **Projet** → renomme le projet
- **P.** → liste déroulante de la priorité P1–P4
- **Devs** → nombre de développeurs
- **Avancement** → pourcentage (0–100)
- **BV** → Business Value
- **Dates** → deadline

Pour les champs non visibles dans le tableau, survolez la ligne et cliquez sur le crayon ✎.

### Organiser vos projets

- **Glisser-déposer** verticalement (via la colonne Ordre ou les barres) pour réorganiser par priorité
- **Couleurs dynamiques** :
  - 🟦 **Gris** = non démarré (0%)
  - 🟦 **Bleu** = en cours (0%–100%)
  - 🟩 **Vert** = fini (100%)
  - 🔴 **Rouge** = hors deadline
  - 🟨 **Ambre** = durée allongée (devs insuffisants)

### Naviguer dans le planning

- **← / →** : naviguer dans le temps
- **Scrollbar horizontale** : avancer/reculer dans les semaines
- **Ligne rouge verticale** : date d'aujourd'hui
- **Double ligne verticale rouge** : deadline d'un projet (visible même au-delà de la fin planifiée)
- **Ligne verte pleine avec flèches** : date de démarrage obligatoire d'un projet (▶ haut et bas)
- **Boutons − / +** : zoomer/dézoomer (7 niveaux)

### Exporter / Importer

- **Exporter** : sauvegarde horodatée en JSON (format : `product-planner_YYYY-MM-DD_HHhMM.json`)
- **Importer** : restaure un ancien snapshot avec confirmation

## 🎨 Système de couleurs

### Par état du projet

| État | Couleur | Signification |
|------|---------|---------------|
| Hors deadline | Rouge | Fin prévue > deadline — **priorité absolue** |
| Durée allongée | Ambre | Devs insuffisants (scaled) |
| Non démarré | Gris | Progress = 0% |
| En cours | Bleu | 0% < Progress < 100% |
| Fini | Vert | Progress = 100% |

### Par priorité globale (P1–P4)

Chaque état a une **nuance différente** selon la priorité :
- **P1 (Critique)** → couleur très saturée
- **P2 (Haute)** → couleur saturée
- **P3 (Normale)** → couleur standard
- **P4 (Basse)** → couleur plus pâle

## 🔧 Architecture

### Single-file design

Tout est contenu dans **`index.html`** :
- HTML (structure)
- CSS (styling + responsive)
- JavaScript (logique + localStorage)

Avantages :
- ✅ Aucun build step
- ✅ Aucune dépendance externe
- ✅ Portable partout
- ✅ Versionnable facilement

### Persistance

- Données stockées dans `localStorage` avec clé `product-planner-v3`
- Survit aux rechargements du navigateur
- Aucune synchronisation cloud (vous contrôlez vos données)

### Algorithme de scheduling

**Bin-packing événementiel** avec priorités :
1. Les projets sont traités par ordre de priorité
2. À chaque libération de dev, on affecte des tâches
3. Un projet ne démarre jamais avant sa **date de démarrage** (si renseignée), ni avant le démarrage d'un projet plus prioritaire
4. Si les devs sont insuffisants pour un projet prioritaire, **la durée s'allonge** — calculée semaine par semaine
5. Cible : **100% d'utilisation des devs** en permanence

**Calcul de la durée allongée** (semaine par semaine) :
- Budget total = `charge_restante × devs_requis` (en semaines-dev)
- Chaque semaine, on consomme `min(devs libres, devs_requis)` de capacité — un projet ne peut pas absorber plus de devs que sa limite de parallélisation
- Les devs occupés sur d'autres projets sont décomptés
- La durée = la semaine où le budget est épuisé (comptage inclusif)

**Exemple** :
- Projet requiert 2 devs × 3 semaines = 6 semaines-dev
- Seuls 1 dev disponible au départ, 1 autre se libère plus tard
- → Durée recalculée selon la capacité réelle disponible chaque semaine (affichée `X→Y` dans la colonne Restant)

## 📋 Historique des features

Pour voir les **29 itérations** de développement, consultez [`FEATURES.md`](./FEATURES.md).

## 📱 Navigateurs supportés

- Chrome/Edge 90+
- Firefox 88+
- Safari 14+
- Brave, Opera, etc. (tout navigateur moderne)

## 💡 Conseils d'utilisation

1. **Exportez régulièrement** vos données pour en garder une sauvegarde
2. **Utilisez les priorités P1–P4** pour vraiment optimiser votre planning
3. **Ajustez les deadlines** pour voir immédiatement l'impact sur le Gantt
4. **Zoom out** pour avoir une vue d'ensemble, **zoom in** pour les détails

## 🛠️ Développement local

Si vous voulez modifier le code :

```bash
# Lancez le serveur avec auto-reload (via npx serve)
npx serve -p 5555 .

# Éditez index.html dans votre éditeur préféré
# Le navigateur se recharge automatiquement (si vous activez l'auto-refresh)
```

Toutes les modifications sont immédiatement visibles. Pas de compilation requise !

## 📄 Licence

MIT — utilisez, modifiez et partagez librement.

---

**Fait avec ❤️ pour les PM et les équipes agiles.**

Questions ? Consultez [`FEATURES.md`](./FEATURES.md) pour plus de détails techniques.
