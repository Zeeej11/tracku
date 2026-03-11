# TrackU — CLAUDE.md

Application de suivi d'assiduité universitaire pour étudiants français.

## Structure du projet

- **Un seul fichier** : tout le code est dans `index.html` (3490 lignes)
  - CSS (lignes 22–1554)
  - HTML (lignes 1556–1971)
  - JavaScript (lignes 1972–3490)

## Stack technique

| Dépendance | Version | Rôle |
|---|---|---|
| Chart.js | 4.4.0 | Graphiques (bar, line, doughnut) |
| Firebase App + Auth + Database | 10.7.1 (compat) | Auth email/password + Realtime DB |
| jsPDF + jsPDF-autotable | 2.5.1 / 3.8.2 | Export rapport PDF |
| Google Fonts | — | IBM Plex Mono + Staatliches |

Toutes les dépendances sont chargées via CDN — aucun bundler, aucun package.json.

## Architecture JavaScript

Tout le code JS est un **objet singleton `app`** initialisé au `DOMContentLoaded` :

```js
const app = {
    data: { courses, attendances, revisions, ... },
    firebase, database, auth, currentUser,
    init(), render(), renderDashboard(), ...
}
```

Aucun framework, aucun module ES — vanilla JS pur.

## Modèle de données

Stockage : `localStorage` clé `tracku-data` + Firebase Realtime DB (optionnel).

```js
// Cours (emploi du temps fixe)
{ id, matiere, jour: 1-5, heureDebut, heureFin, salle, type: "CM"|"TD"|"TP", couleur }

// Présences
{ id, coursId, date: ISO_string, statut: "present"|"absent" }

// Révisions (sessions chrono)
{ id, matiere, couleur, startTime: timestamp, endTime: timestamp, duration: ms }

// Plans (planificateur)
{ id, date: "YYYY-MM-DD", matiere, couleur, note: string, statut: "todo"|"done"|"late", createdAt: timestamp }
```

## Sections de l'app

| Section | ID HTML | Rendu par |
|---|---|---|
| Tableau de bord | `#dashboard` | `renderDashboard()` |
| Emploi du temps | `#emploi` | `renderSchedule()` |
| Historique présences | `#historique` | `renderHistorique()` |
| Révisions / chrono | `#revisions` | `renderRevisions()` |
| Planificateur | `#planificateur` | `renderPlanificateur()` |
| Statistiques | `#statistiques` | `renderStatistics()` |
| Paramètres / sync | `#parametres` | — |

## Créneaux horaires

Fixes et hardcodés : `08:00`, `09:30`, `11:00`, `(12:20 pause)`, `14:00`, `15:30`, `17:00`
Durée de chaque cours : **1h20**

## Firebase

- Projet : `tracku-63e13`
- Région DB : `europe-west1`
- Auth : email/password uniquement
- Structure DB : `users/{uid}/{ courses, attendances, revisions, lastSync }`
- La config Firebase (apiKey, etc.) est exposée dans le code client — c'est intentionnel pour une app web publique ; les règles de sécurité Firebase côté serveur protègent les données.

## Design system

- Style **néobrutaliste** : bordures épaisses, ombres décalées (`box-shadow: 8px 8px 0`)
- Couleurs : orange `#FF6B35` (primary), bleu `#004E89` (secondary), jaune `#F7B801` (accent)
- Fonts : `Staatliches` (titres display), `IBM Plex Mono` (corps)
- Dark mode : toggle en haut à droite, persisté dans `localStorage` clé `darkMode`
- Responsive : breakpoints à 1024px et 768px

## Visualisation "croissance" (révisions)

Animation de construction de bâtiment selon le temps écoulé :
- `stage-0` < 5min → Plans
- `stage-1` ≥ 5min → Fondations
- `stage-2` ≥ 15min → Rez-de-chaussée
- `stage-3` ≥ 30min → Petit immeuble
- `stage-4` ≥ 60min → Immeuble 4 étages
- `stage-5` ≥ 120min → Gratte-ciel
- `stage-6` ≥ 180min → Tour épique
- `stage-7` ≥ 240min → Burj Khalifa

## Conventions importantes

- **Langue** : UI et noms de variables en **français**
- **Pas de séparation fichiers** : tout reste dans `index.html` sauf décision contraire explicite
- **Pas de build** : modifications directement dans `index.html`, déploiement = push GitHub
- Les IDs sont générés avec `Date.now().toString(36) + Math.random().toString(36).substr(2)`
- Auto-save toutes les 30 secondes + sync cloud automatique si connecté

## Planificateur (feature ajoutée)

- Sélecteur de plage de dates libre → colonnes par jour
- Tâche = `{ matiere, note, statut: todo|done|late }`
- `checkLateStatuses()` appelé au `init()` et à chaque rendu : toute tâche `todo` dont la date est passée → `late`
- Méthodes : `renderPlanificateur`, `openAddPlanModal(dateStr)`, `editPlan(id)`, `savePlan(event)`, `markPlanStatus(id, statut)`, `deletePlanFromModal`, `getPlansByDate(dateStr)`, `checkLateStatuses`
- La couleur de la tâche est héritée de la matière via `data-couleur` sur l'option du select

## Points d'attention

- Seul le premier cours par créneau est affiché (pas de gestion des conflits d'horaires)
- Les `alert()` / `confirm()` natifs sont utilisés partout pour les feedbacks utilisateur
- La semaine affichée dans l'historique est relative à `currentWeekOffset` (navigation ±1 semaine)
