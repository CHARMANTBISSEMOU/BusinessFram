# PLAN DE DÉVELOPPEMENT — Plateforme Avicole Intelligente
## Découpage en tâches pour 4 développeurs (P1, P2, P3, P4)

> Sources : `Analyse_App_Elevage.docx` (app existante ElevageApp) et `Resume_decisions_plateforme_avicole_intelligente.docx` (décisions validées).
> Chaque tâche a un identifiant **T-xxx** : ces identifiants seront repris tels quels dans le tableau HTML de suivi (GitHub Pages).
> Statuts utilisables : `À faire` → `Prise en charge` (par P1/P2/P3/P4) → `Terminée`. Commentaire facultatif par tâche.

---

## 0. VISION DU PRODUIT (à lire par tout le monde)

On développe la **plateforme avicole intelligente**, évolution de l'app ElevageApp existante :
- **Deux modules** : poulets de chair ET poules pondeuses (deux suivis de production distincts).
- **Gratuit** : toutes les fonctions classiques de gestion. **Payant** : chat IA (500 FCFA/mois), business plan chair (1 000 FCFA/doc), business plan pondeuses (2 000 FCFA/doc).
- **Hors ligne d'abord** : les opérations courantes marchent sans Internet, avec synchronisation automatique, anti-doublons et gestion des conflits.
- **Le serveur fait autorité** : compte, droits d'accès, paiements, identifiant permanent.
- **Paiements** : MTN Mobile Money et Orange Money uniquement, confirmation serveur obligatoire avant activation du service.
- **Principes** : aucune écriture sans confirmation explicite (« Enregistrer ») ; distinguer en permanence **données saisies / données calculées / estimations** ; calculs reproductibles et traçables ; jamais de valeur inventée (si une référence manque, on le signale).

### Organisation de l'équipe
| Profil | Responsabilités principales |
|---|---|
| **P1 — Socle & Serveur** | Backend, base de données, authentification SMS, comptes, synchronisation/conflits, sauvegardes, sécurité, permissions |
| **P2 — Mobile & Saisies** | Application mobile (React Native/Expo/TS) : exploitations, bâtiments, lots, fiches quotidiennes, saisies rapides, module pondeuses |
| **P3 — Stocks, Prix & Catalogues** | Moteurs de calcul : stocks, inventaire, prix aliment, conversions d'unités, moteur financier, catalogues |
| **P4 — Rapports & Services payants** | Rapports/statistiques/graphiques, exports PDF/Excel, chat IA, business plans, paiements, UI/UX transverse |

### Règles d'équipe (valables pour tous)
- Dépôt unique (monorepo) : `apps/mobile/`, `server/`, `docs/`.
- Une branche par tâche (`T-015-sync-conflicts`…), une Pull Request par tâche, **au moins 1 relecture par un autre membre** avant fusion.
- Aucune tâche « Terminée » sans ses critères d'acceptation validés + tests passés.
- Les points marqués **[À TRANCHER]** nécessitent une décision collective (ou du client) avant d'être codés.

## 1. PHASE 0 — CADRAGE COMMUN (tout le monde, 1 à 2 jours)

| ID | Tâche | Pour qui | Détail de ce qu'il y a à faire | Critères d'acceptation |
|---|---|---|---|---|
| T-000 | Valider l'architecture technique | Tous | Confirmer la stack proposée : mobile **React Native + Expo + TypeScript** (base existante ElevageApp), backend **Node.js + TypeScript + PostgreSQL**, API REST versionnée, file de synchronisation locale (SQLite). Alternative possible : rester 100 % Firebase/Firestore. | Décision écrite dans `docs/architecture.md` + schéma de base de données (comptes, exploitations, bâtiments, lots, mouvements, prix, ventes, stocks, transactions) |
| T-001 | Mettre en place le dépôt Git | Tous | Créer la structure monorepo, le README, les conventions Git (branches, commits, PR), la CI locale de tests, et y intégrer le fichier HTML de suivi des tâches | `npm test` tourne sur tout le dépôt ; le tableau de suivi est accessible |
| T-002 | Glossaire et contrats d'API | P1 + P2 | Écrire le glossaire (Lot = bande, exploitation = ferme, mouvement…) et le contrat d'API (format des échanges mobile ↔ serveur) avant tout développement | Document `docs/api-contract.md` approuvé par les 4 |

---

## 2. PHASE 1 — SOCLE SERVEUR (P1)

| ID | Tâche | Détail de ce qu'il y a à faire | Critères d'acceptation |
|---|---|---|---|
| T-010 | Authentification téléphone + mot de passe | Inscription/connexion avec **numéro de téléphone + mot de passe** ; vérification initiale par SMS (OTP) et récupération de compte par SMS ; **pas** de SMS à chaque connexion ; identifiant permanent unique attribué côté serveur ; protection contre les tentatives répétées (rate limiting) | Scénarios inscription / connexion / récupération testés automatiquement ; un OTP erroné ×5 est bloqué |
| T-011 | API comptes & profil | CRUD du profil (prénom, etc. — demandé **après** l'inscription, lors de la configuration de l'exploitation) ; gestion des sessions (JWT) ; suppression de compte | Tests d'intégration API verts |
| T-012 | API exploitations | Nom **obligatoire** ; localisation **GPS avec saisie manuelle possible** ; **plusieurs exploitations par compte** ; droits d'accès par utilisateur | Création de 2 exploitations sur un même compte testée |
| T-013 | API bâtiments | Gestion des bâtiments dès le départ ; affectation des lots aux bâtiments | CRUD + affectation testés |
| T-014 | API lots | Nom facultatif + **référence automatique LOT-001** si absent ; **âge à l'arrivée obligatoire** (en jours ou semaines) en plus de la date d'arrivée ; **effectif initial = oiseaux réellement présents** à la création ; **effectif courant calculé** à partir de l'effectif initial et des mouvements (mortalités, ventes…) | Test : création sans nom → LOT-001 ; effectif courant = initial − mouvements |
| T-015 | Moteur de synchronisation hors ligne | File d'attente locale ; envoi automatique au retour d'Internet ; **prévention des doublons** (clés d'idempotence) ; **détection des conflits** + **règle de résolution explicite écrite** (modifications concurrentes d'une même donnée) ; les opérations **distinctes** faites hors ligne sont toutes conservées | Tests : 2 modifications concurrentes résolues selon la règle ; opérations hors ligne non perdues ; aucun doublon après reconnexion |
| T-016 | Sauvegardes & restauration | Sauvegarde locale + serveur ; restauration après réinstallation ou changement de téléphone (après reconnexion au compte) ; documenter la limite : les données locales **jamais synchronisées** ne sont pas garanties récupérables | Test : suppression app → réinstallation → restauration des données sync |
| T-017 | Rôles & permissions | Rôles : **PDG, responsable du suivi, comptable, contrôleur, vendeur, partenaire, invité** ; permissions par ressource (entreprise / exploitation / lot / bâtiment) côté serveur ; invitations par téléphone ou code ; codes **sécurisés, révocables, avec durée de validité** [À TRANCHER : permissions détaillées de chaque rôle] | Matrice de permissions dans `docs/permissions.md` + tests d'accès refusés |

## 3. PHASE 2 — MOBILE & SAISIES QUOTIDIENNES (P2)

| ID | Tâche | Détail de ce qu'il y a à faire | Critères d'acceptation |
|---|---|---|---|
| T-020 | Écrans exploitations / bâtiments / lots | Création et gestion : exploitation (nom obligatoire, GPS + manuel), bâtiments, lots (nom facultatif, réf auto, âge à l'arrivée obligatoire, date d'arrivée, effectif initial réel) ; affectation lot → bâtiment | Parcours complet jouable sur téléphone |
| T-021 | Fiche quotidienne complète | Saisie de **toutes les rubriques de l'app existante** (mortalités, aliments, produits vétérinaires…) en une fiche ; boutons explicites **Enregistrer / Annuler / Reformuler** ; **aucune opération enregistrée sans action sur « Enregistrer »** ; champs indisponibles restent vides | Rien n'est écrit sans clic sur Enregistrer ; reprise de la fiche du jour possible |
| T-022 | Saisies rapides par opération | Formulaire par opération : **Mortalités** (nombre obligatoire, cause/observation facultatives ; effectif restant recalculé) ; **Aliments** (quantité **réellement consommée**, en sacs **ou en kg** avec conversion si poids du sac connu — **jamais d'estimation automatique**) ; **Produits vétérinaires utilisés** (nom + quantité ; ajout possible d'un produit absent au catalogue) ; **Achats vétérinaires** (produit, quantité, prix unitaire, date **uniquement**) ; **Autres dépenses** (catégorie personnalisable, montant FCFA, date **uniquement**) ; **Ventes de volailles** (quantité, prix unitaire, date ; total calculé ; **plusieurs ventes le même jour**) | Chaque opération est testée unitairement ; le total de vente est calculé automatiquement |
| T-023 | Module pondeuses | Suivi flexible : **œufs produits, œufs cassés/impropres, œufs vendus, stock d'œufs, alimentation** ; ventes d'œufs **en plateaux ou à l'unité** ; **contenance du plateau configurable** ; conversion automatique quand elle est connue | Vente « 3 plateaux » convertie correctement quand la contenance est renseignée |
| T-024 | Écrans de consultation | Accueil, historique, archives, alertes (traitées par boutons **accepter / refuser**, pas par saisie) [À TRANCHER plus tard : notifications] | Affichage correct des données déjà saisies |

---

## 4. PHASE 3 — STOCKS, PRIX & CATALOGUES (P3)

| ID | Tâche | Détail de ce qu'il y a à faire | Critères d'acceptation |
|---|---|---|---|
| T-030 | Moteur de stocks automatiques | Stocks calculés **à partir des achats et consommations enregistrés** (stock initial + tous les mouvements) ; **inventaire physique comparable au stock théorique** ; corrections **tracées dans l'historique** | Test : achat +10, consommation −4 → stock 6 ; correction d'inventaire tracée |
| T-031 | Prix des aliments & historique | Saisie manuelle des prix **avec date d'application** ; historique conservé ; les opérations passées utilisent **le prix applicable à leur période** ; **aucun prix ne modifie rétroactivement** les anciens calculs ; **poids du sac configurable par produit** (ne jamais supposer 50 kg) | Test : changer le prix aujourd'hui ne change pas les coûts du mois dernier |
| T-032 | Catalogues & unités | **Catalogue commun** (produits, ingrédients, unités, catégories…) **personnalisable par exploitation** ; les ajouts d'une exploitation **ne modifient pas** le catalogue commun ; anti-doublons ; vérification de cohérence des unités | Test : ajout perso visible seulement pour l'exploitation ; doublon refusé |
| T-033 | Moteur financier | Calculs de coûts, chiffre d'affaires, bénéfice estimé, rentabilité par lot ; fondés sur les **mouvements enregistrés** ; **reproductibles et traçables** ; distinction stricte saisies / calculées / estimées | Les calculs sont reproductibles à partir de l'historique des mouvements |

## 5. PHASE 4 — RAPPORTS & STATISTIQUES (P4)

| ID | Tâche | Détail de ce qu'il y a à faire | Critères d'acceptation |
|---|---|---|---|
| T-040 | Rapports de gestion | Rapports sur **effectifs, mortalités, alimentation, dépenses, achats, ventes et stocks** | Chaque rubrique a son rapport, fondé sur les données enregistrées |
| T-041 | Rapports financiers & graphiques | Coûts, CA, bénéfice estimé, rentabilité par lot, production des pondeuses ; **graphiques d'évolution** ; **comparaisons** entre lots / exploitations, et entre **prévisions et résultats réels** (clairement distingués) | Estimations et réels visuellement distincts |
| T-042 | Exports | Export des **rapports en PDF** et des **données en Excel** | PDF et Excel corrects et ouvrables |

---

## 6. PHASE 5 — SERVICES PAYANTS (P4 avec P1)

| ID | Tâche | Détail de ce qu'il y a à faire | Critères d'acceptation |
|---|---|---|---|
| T-050 | Chat IA (500 FCFA/mois) | Nécessite Internet ; dans le chat, **l'IA prépare une fiche à confirmer** ; aucune opération enregistrée sans « Enregistrer » ; l'utilisateur peut **annuler ou reformuler** [À TRANCHER : quota mensuel ou messages illimités] | Une fiche proposée par l'IA n'écrit rien sans confirmation |
| T-051 | Générateur de business plans | Business plan **chair : 1 000 FCFA** / **pondeuses : 2 000 FCFA** ; **génération + modifications illimitées** du document acheté ; **un nouveau projet = un nouvel achat** ; utilise des statistiques agrégées anonymisées + références externes vérifiées (source, date, région, unité, cohérence, taille d'échantillon, niveau de confiance) ; **si références insuffisantes → le signaler, ne jamais inventer** ; le **moteur de calcul** fait les calculs financiers, l'IA les explique ; aucune donnée individuelle d'exploitation exposée [À TRANCHER : parcours formulaire guidé vs conversation IA vs les deux] | Document généré avec sources citées ; achat = 1 projet ; modifs illimitées après achat |
| T-052 | Paiements MTN MoMo & Orange Money | Intégration **MTN Mobile Money et Orange Money uniquement** ; **le serveur confirme le paiement avant d'activer le service** ; historique des transactions conservé [À TRANCHER : prestataire et process de confirmation] | Un paiement non confirmé n'active rien ; historique complet |

---

## 7. PHASE 6 — QUALITÉ & LIVRAISON (tous)

| ID | Tâche | Détail de ce qu'il y a à faire | Critères d'acceptation |
|---|---|---|---|
| T-060 | Tests & QA croisée | Tests unitaires (moteurs de calcul, sync, conflits), tests d'intégration API, tests mobile sur les parcours clés ; QA croisée P1↔P3 et P2↔P4 | Couverture des moteurs critiques ; zéro bug bloquant connu |
| T-061 | Documentation | Documentation technique (API, schéma BD) + guide utilisateur | Docs à jour dans `docs/` |
| T-062 | Recette finale | Démo complète + validation des critères d'acceptation de chaque tâche | PV de recette signé |

---

## 8. DÉPENDANCES À RESPECTER
- T-002 bloque T-012→T-017 et T-020→T-024.
- T-014 (effectif courant par mouvements) doit exister avant T-022 (mortalités qui modifient l'effectif).
- T-015 (sync) est utilisé par **toutes** les saisies mobiles : le faire tôt.
- T-031 (poids du sac configurable) est requis avant T-022 (conversion sacs ↔ kg).
- T-033 (moteur financier) est requis avant T-051 (business plans) et T-041.
- T-017 (permissions) est requis avant T-050/T-051/T-052 (accès payants).

## 9. POINTS [À TRANCHER] — décision collective nécessaire
1. Notifications et alertes poussées (reportées).
2. Quota du chat IA (mensuel ou illimité).
3. Permissions détaillées de chaque rôle.
4. Règle exacte de résolution des conflits de sync.
5. Traitement des données locales jamais synchronisées.
6. Prestataire et process de confirmation des paiements MTN/Orange.
7. Validation et actualisation du catalogue de référence.
8. Portée des modifications d'un business plan (principe : illimité pour le document acheté).



