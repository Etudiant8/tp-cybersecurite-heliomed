# TP Cybersécurité & protection des données — HelioMed

Projet fil rouge : HelioMed, PME éditant une plateforme web de prise de RDV médicaux
(API Flask + Nginx + PostgreSQL, 2 VM Linux, appli mobile, données de santé RGPD).

Ce dépôt couvre le TP « Élaborer et mettre en œuvre des stratégies de cybersécurité
et de protection des données ». Le périmètre HelioMed est commun aux 5 exercices —
seul l'Exercice 1 est traité pour l'instant (les énoncés 2 à 5 manquaient dans le
PDF source).

## Structure

- `infra/` — fichiers de configuration réels (docker-compose, nginx, règles pare-feu…)
- `docs/logbook.md` — journal de bord du binôme
- `docs/exercice1/` — registre d'actifs, STRIDE, matrice de risque, EBIOS, objectifs de sécurité
- `outputs/exercice1/` — sorties brutes des outils (scan_helio.txt, exports ZAP/Trivy)

## Environnement de labo

- 2 VM VirtualBox (frontal + applicatif/BDD), réseau isolé, se pinguent.
- Cibles d'entraînement représentant les composants HelioMed : OWASP Juice Shop (frontal)
  + DVWA (applicatif) via `infra/docker-compose.yml`.

## Exercice 1 — Analyse de l'environnement de sécurité

Voir `docs/exercice1/`. Livrable à déposer 12h30 : note d'analyse (4-6 pages) +
fichiers de sortie bruts + logbook avec part IA explicitée.
