# Plan de tests de sécurité — Étape 2.4

| Type de test | Outil | Cible | Critère de réussite | État actuel (Ex.1) | Fréquence prévue |
|---|---|---|---|---|---|
| Scan de vulnérabilités web (DAST) | OWASP ZAP (scan passif puis actif en Ex.3) | Application patient HelioMed (via `helio-frontal`) | 0 alerte **High** non traitée | 0 High / 116 Medium / 160 Low — déjà conforme au critère High, Medium à traiter avant Ex.3 | À chaque déploiement + mensuel |
| Scan dépendances/image (SCA) | Trivy | Image applicative (appli patient réelle en Ex.3, actuellement Juice Shop en démo) | 0 CVE **critique** restante | 8 CRITICAL sur l'image de démo (Juice Shop) — **non conforme**, à corriger en Ex.3 par mise à jour des dépendances (crypto-js notamment) | À chaque commit (CI, voir `.github/workflows/security.yml`) |
| Test de config / durcissement | Lynis | VM Linux (helio-frontal, helio-applicatif) | Score Lynis en hausse documentée | Non exécuté à ce jour — planifié avant la fin d'Ex.2 (ligne suivante) | Avant chaque mise en production + trimestriel |
| Test réseau | Nmap | Les 2 VM (10.10.10.10, 10.10.10.20) | Aucun port inutile ouvert | **Non conforme** : Samba (139/445) ouvert sans usage sur les 2 VM (constaté Ex.1) — correction prévue Ex.3 | Après tout changement réseau + mensuel |

## Exécution Lynis (baseline, Ex.2)

Premier passage Lynis exécuté sur les 2 VM pour établir la référence (baseline) avant durcissement.
Résultats bruts : `outputs/exercice2/lynis_helio-frontal.txt`, `outputs/exercice2/lynis_helio-applicatif.txt`.
Le score de hausse documentée (critère du tableau ci-dessus) se mesurera par comparaison avec un
second passage après application des mesures de durcissement (#12-13 du plan de sécurité) en Ex.3.

## Priorisation des non-conformités constatées

1. **Nmap — Samba ouvert** : correction rapide (désinstallation), aucune dépendance bloquante → Ex.3 J1
2. **Trivy — CVE critiques image démo** : dépend du remplacement des images de démonstration par le
   code applicatif réel HelioMed en Ex.3, non corrigeable sur Juice Shop/DVWA eux-mêmes (volontairement
   vulnérables par conception)
3. **ZAP — alertes Medium** : ajout des en-têtes de sécurité (CSP, HSTS) au niveau Nginx → Ex.3 J1
