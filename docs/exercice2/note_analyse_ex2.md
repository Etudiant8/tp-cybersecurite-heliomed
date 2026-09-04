# Note d'analyse — Exercice 2 : Conception et planification de la sécurité HelioMed

**Binôme :** Deven Imazoute
**Périmètre :** suite directe de l'Ex.1 — les mesures et l'architecture ci-dessous répondent aux
risques identifiés dans `docs/exercice1/` (STRIDE, EBIOS, registre d'actifs).

---

## 1. Mesures de sécurité (Étape 2.1)

15 mesures (minimum 12 requis) — détail complet, mapping ISO/IEC 27002:2022 + ANSSI dans
`plan_securite.md`. 5 d'entre elles corrigent directement une vulnérabilité **réelle** constatée
en Ex.1 (clés SSH dupliquées, secrets en clair, Samba superflu) plutôt que d'être génériques.

## 2. Intégration DevSecOps (Étape 2.2)

- **Timeline** conception → build → run avec points de contrôle go/no-go explicites : voir
  `devsecops_timeline.md`.
- **Pipeline CI réel** : `.github/workflows/security.yml` — gitleaks (scan de secrets) + Trivy
  (scan filesystem/IaC + scan des 3 images du docker-compose), déclenché à chaque push.
  **Vérifié en conditions réelles**, pas seulement écrit : poussé sur GitHub, exécuté, et corrigé
  une fois (version d'action invalide) jusqu'à obtenir 5/5 jobs verts.
  Preuve : [github.com/Etudiant8/tp-cybersecurite-heliomed/actions](https://github.com/Etudiant8/tp-cybersecurite-heliomed/actions)
  — le scanner de secrets Trivy a même détecté une clé privée SSL embarquée dans l'image DVWA,
  confirmant que l'outil analyse réellement le contenu et ne se contente pas de tourner à vide.

## 3. Architecture de sécurité cible (Étape 2.3)

Schéma en 3 zones (DMZ frontale / applicative / données), **ancré sur les IP réelles du labo**
(`helio-frontal` 10.10.10.10, `helio-applicatif` 10.10.10.20 — pas un schéma générique) :
WAF + reverse proxy TLS en DMZ, pare-feu internes avec flux allow-list explicites, chiffrement au
repos sur la zone données, collecte de logs centralisée vers un SIEM (Wazuh — traite directement
le risque *Repudiation* identifié dans la matrice STRIDE d'Ex.1).

Fichiers : `assets/architecture_cible.drawio` (source éditable), `assets/architecture_cible.svg`
et `.png` (export image). Le schéma indique explicitement les écarts avec l'état actuel (BDD
co-localisée, canal interne non chiffré) pour rester honnête sur ce qui reste à déployer en Ex.3.

## 4. Plan de tests de sécurité (Étape 2.4)

Table complète dans `plan_tests.md`, avec pour chaque test l'**état actuel réel** (pas une case
à cocher théorique) :

| Test | Conforme aujourd'hui ? |
|---|---|
| ZAP — 0 alerte High | ✅ Oui (0 High constaté en Ex.1) |
| Trivy — 0 CVE critique | ❌ Non (8 CRITICAL sur l'image de démo) |
| Lynis — score en hausse | Baseline **réellement exécutée** sur les 2 VM ce jour (voir `outputs/exercice2/`) |
| Nmap — aucun port inutile | ❌ Non (Samba ouvert sans usage, constaté en Ex.1) |

## 5. Plan de réponse aux incidents (Étape 2.5)

Rôles (décideur/agit/communique/support), 4 niveaux de gravité (P1-P4), 6 phases NIST, et
obligation RGPD de notification CNIL sous 72h pour toute violation de données de santé — détail
dans `plan_reponse_incidents.md`.

## 6. Logbook et part IA

Voir `docs/logbook.md` (section Exercice 2). Le pipeline CI a été activement testé et corrigé
(pas seulement rédigé) et l'audit Lynis a été réellement exécuté sur les 2 VM plutôt que planifié
sur le papier — cohérent avec la méthode adoptée depuis l'Ex.1 : produire des preuves réelles,
pas des livrables génériques.
