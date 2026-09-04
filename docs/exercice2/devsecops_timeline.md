# Intégration dans le cycle projet (DevSecOps) — Étape 2.2.1

## Timeline conception → build → run

| Phase | Mesures placées (voir `plan_securite.md`) | Point de contrôle go/no-go |
|---|---|---|
| **Conception** | #4 Segmentation réseau (schéma cible §2.3) · #6-7 Choix des mécanismes de chiffrement · #8 Choix du gestionnaire de secrets · Modélisation des menaces (Ex.1 STRIDE/EBIOS) | **Go/no-go archi** : le schéma d'architecture doit être validé et raccordé aux VM réelles avant tout déploiement (contrainte du sujet) |
| **Build** (CI, à chaque commit) | #15 Scan SCA image (Trivy) · scan de secrets (gitleaks) · #14 Analyse statique OWASP sur le code applicatif | **Go/no-go CI** : *aujourd'hui (Ex.2)* le pipeline tourne en mode rapport (non bloquant) car les images cibles (Juice Shop/DVWA) sont volontairement vulnérables. *À partir d'Ex.3*, sur le code HelioMed réel remédié : **0 CVE critique → merge autorisé, sinon build rejeté** (`exit-code 1`) |
| **Déploiement (pré-run)** | #1-3 MFA + clés SSH individuelles + régénération secrets avant mise en service · #5 Règles pare-feu appliquées · #12-13 Durcissement + patch appliqué | **Go/no-go déploiement** : checklist de durcissement signée (Lynis, cf. plan de tests §2.4) avant bascule en production |
| **Run** (exploitation continue) | #9 Rotation des secrets (90 j) · #10 Supervision SIEM · #11 Sauvegardes testées · Scans périodiques (Nmap/ZAP mensuels) | **Go/no-go continu** : toute alerte SIEM critique déclenche le plan de réponse aux incidents (§2.5) avant reprise normale du service |

## Principe

Aucune mesure n'est reportée à la fin du projet : les mesures d'authentification/secrets (#1-3, #8-9)
sont **immédiates** car elles corrigent des failles déjà exploitables constatées en Ex.1 ; les mesures
de détection (#15, scan CI) sont introduites **dès le premier commit** de code applicatif, pas
après coup — c'est tout l'objet du pipeline ci-dessous.
