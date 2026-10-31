# BusinessFram — Plateforme Avicole Intelligente

Projet collaboratif à **4 développeurs (P1, P2, P3, P4)** : plateforme de gestion d'élevage
de volailles (poulets de chair **et** poules pondeuses) — saisies quotidiennes, stocks et
prix, rapports, chat IA et business plans, avec mode **hors ligne** et paiements
**MTN Mobile Money / Orange Money**.

## Contenu du dépôt
- `index.html` — **tableau de suivi des tâches** (prise en charge par P1-P4, bouton
  « Terminer », commentaires facultatifs). Ouvrable en local ou via GitHub Pages.
- `etat-suivi.json` — état partagé du suivi, synchronisé par la page HTML.
- `docs/PLAN_TACHES_PLATEFORME_AVICOLE.md` — plan de développement (25 tâches T-xxx).
- `docs/UML2/` — modélisation UML2 (11 diagrammes PlantUML + images SVG/PNG).

## Organisation de l'équipe
- `main` : version stable, toujours fonctionnelle.
- 4 branches de travail : `P1-socle-serveur`, `P2-mobile-saisies`, `P3-stocks-prix`,
  `P4-rapports-payants`.
- Chacun travaille sur **sa branche** : une Pull Request par tâche (nommée `T-xxx-...`),
  **au moins 1 relecture par un autre membre** avant fusion dans `main`.
- Une tâche n'est « Terminée » dans le suivi que si ses critères d'acceptation
  (dans le plan) sont validés.

## Utilisation du tableau de suivi
1. Ouvrir `index.html` (GitHub Pages ou double-clic local).
2. Cocher **« Prise en charge »** sur une tâche et sélectionner **qui vous êtes (P1/P2/P3/P4)**.
3. Cliquer **« Terminer »** quand la tâche est finie (« Rouvrir » pour annuler).
4. Laisser un **commentaire facultatif** par tâche.
5. Partage avec l'équipe : saisir un **jeton GitHub** (droits d'écriture sur le dépôt) dans
   le panneau « Synchronisation » → la page lit/écrit `etat-suivi.json` sur le dépôt ;
   sinon, utiliser **Exporter / Importer** le JSON.

## Rappel des rôles
| Profil | Responsabilités |
|---|---|
| P1 | Socle & serveur : API, base de données, auth SMS, synchronisation/conflits, permissions |
| P2 | Mobile & saisies : exploitations, lots, fiches quotidiennes, saisies rapides, pondeuses |
| P3 | Stocks, prix & catalogues : moteurs de calcul, inventaire, conversions d'unités |
| P4 | Rapports & services payants : rapports, exports, chat IA, business plans, paiements |
