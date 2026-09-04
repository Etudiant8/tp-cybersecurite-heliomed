# Objectifs de sécurité & choix d'outils — HelioMed

## 5 objectifs de sécurité mesurables (rattachés au DICP)

| # | Objectif mesurable | Critère DICP | Risque(s) STRIDE couvert(s) (voir `stride_risques.md`) |
|---|---|---|---|
| 1 | Aucun service exposé sans authentification (0 endpoint public non protégé détecté par ZAP) | Confidentialité | Tampering (CSP absent sur l'appli patient) et Elevation of privilege (DVWA exposé sans isolation) |
| 2 | 100% des données de santé chiffrées au repos (BDD + sauvegardes) | Confidentialité | Information disclosure (BDD MySQL, risque 12 — le plus élevé du registre) |
| 3 | 0 CVE critique/haute non corrigée depuis > 30 jours sur les images en production (Trivy) | Intégrité | Elevation of privilege (805 CVE sur l'image DVWA) et, indirectement, Information disclosure (CVE crypto-js dans l'image de l'appli patient) |
| 4 | Disponibilité de l'API ≥ 99% mesurée sur le mois (protection anti-DoS sur le frontal) | Disponibilité | Denial of service (reverse proxy Nginx, seul point d'entrée public) |
| 5 | 100% des accès admin/déploiement tracés et attribuables (logs horodatés, pas de compte partagé) | Preuve/traçabilité | Repudiation (comptes télétravail, mot de passe partagé) et Spoofing (clés SSH dupliquées entre les 2 VM) |

> Les 6 lignes de la matrice STRIDE sont donc toutes couvertes par au moins un des 5 objectifs — aucun risque identifié n'est laissé sans objectif de traitement associé.

## Justification du choix des outils

- **Nmap** (`-sV -sC` sur 10.10.10.0/24) : cartographie rapide des services/ports exposés sur les 2 VM. A permis de découvrir un risque non anticipé : Samba (139/445) exposé inutilement sur les deux VM (hérité du clonage de la VM socle) et des **clés hôte SSH identiques** entre helio-frontal et helio-applicatif — invisible sans une découverte réseau systématique en amont de tout scan applicatif.
- **OWASP ZAP** (scan passif, spider + observation, sans attaque active) : adapté à l'interface web patient (Juice Shop simulant l'appli HelioMed), détecte les failles de configuration HTTP/OWASP Top 10 sans risquer de perturber le service — pertinent pour une appli manipulant potentiellement des données de santé en production, où un scan actif serait à proscrire sans fenêtre de maintenance. A remonté 116 alertes Medium (CSP absent, CORS permissif) exploitables directement dans le registre d'actifs.
- **Trivy** (scan des 3 images du docker-compose) : HelioMed est conteneurisé, donc les CVE dans les dépendances (npm pour l'appli patient type Juice Shop/Flask, paquets système des images de base) sont un risque direct invisible sans SCA. Complète ZAP (qui ne voit que la surface HTTP) en couvrant la chaîne d'approvisionnement logicielle.
- **Pourquoi pas d'autres outils en Ex.1** : Nikto (redondant avec ZAP sur ce périmètre, moins adapté aux SPA modernes type Juice Shop) et Burp Suite (usage manuel/interactif, moins adapté à une preuve automatisée reproductible pour ce livrable) n'apportaient pas de valeur incrémentale pour cette étape de cartographie initiale.
