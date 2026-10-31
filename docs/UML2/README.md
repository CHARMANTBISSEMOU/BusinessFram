# Modélisation UML2 — Plateforme Avicole Intelligente

Démarche d'ingénierie logicielle suivie (les étapes concrètes des vrais ingénieurs) :

1. **Exigences** → `01-cas-utilisation.puml` : qui fait quoi, acteurs et fonctionnalités.
2. **Analyse du domaine** → `02-classes.puml` : le vocabulaire du métier (entités et relations).
3. **Conception comportementale** :
   - `03-sequence-inscription.puml` : inscription téléphone + OTP SMS.
   - `04-sequence-sync-hors-ligne.puml` : saisie hors ligne, synchronisation, conflits.
   - `05-sequence-paiement.puml` : paiement MTN MoMo / Orange Money, activation du service.
   - `06-activite-saisie-rapide.puml` : processus de saisie d'une opération.
   - `07-etats-lot.puml` : cycle de vie d'un lot (cycle 77 jours).
   - `08-etats-transaction.puml` : cycle de vie d'une transaction de paiement.
   - `09-etats-synchronisation.puml` : cycle de vie d'une opération synchronisée.
4. **Architecture** :
   - `10-composants.puml` : découpage logiciel (mobile, serveur, moteurs, services externes).
   - `11-deploiement.puml` : infrastructure (téléphones, serveurs, services externes).

## Comment visualiser les diagrammes
- **Fichiers `.puml`** : format PlantUML (UML2), texte pur → versionnable dans Git.
- **Rendu** : les images SVG sont générées dans `svg/` avec `plantuml.jar` (moteur Smetana, sans Graphviz).
- À la main : `java -jar plantuml.jar -tsvg -Playout=smetana *.puml`
- Ou l'extension VS Code « PlantUML » (Alt+D pour prévisualiser).

## Correspondance avec le plan de tâches
- 01 couvre le périmètre global (T-010 à T-052).
- 02 structure les données des tâches T-012, T-014, T-022, T-030 à T-033.
- 03 ↔ T-010 ; 04 ↔ T-015 ; 05 ↔ T-052 ; 06 ↔ T-022 ; 07 ↔ T-014 ; 08 ↔ T-050/T-051/T-052 ; 09 ↔ T-015 ; 10/11 ↔ T-000.
