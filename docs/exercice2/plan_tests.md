# Plan de tests de sécurité — Étape 2.4

| Type de test | Outil | Cible | Critère de réussite | État actuel (Ex.1) | Fréquence prévue |
|---|---|---|---|---|---|
| Scan de vulnérabilités web (DAST) | OWASP ZAP (scan passif puis actif en Ex.3) | Application patient HelioMed (via `helio-frontal`) | 0 alerte **High** non traitée | 0 High / 116 Medium / 160 Low — déjà conforme au critère High, Medium à traiter avant Ex.3 | À chaque déploiement + mensuel |
| Scan dépendances/image (SCA) | Trivy | Image applicative (appli patient réelle en Ex.3, actuellement Juice Shop en démo) | 0 CVE **critique** restante | 8 CRITICAL sur l'image de démo (Juice Shop) — **non conforme**, à corriger en Ex.3 par mise à jour des dépendances (crypto-js notamment) | À chaque commit (CI, voir `.github/workflows/security.yml`) |
| Test de config / durcissement | Lynis | VM Linux (helio-frontal, helio-applicatif) | Score Lynis en hausse documentée | Non exécuté à ce jour — planifié avant la fin d'Ex.2 (ligne suivante) | Avant chaque mise en production + trimestriel |
| Test réseau | Nmap | Les 2 VM (10.10.10.10, 10.10.10.20) | Aucun port inutile ouvert | **Non conforme** : Samba (139/445) ouvert sans usage sur les 2 VM (constaté Ex.1) — correction prévue Ex.3 | Après tout changement réseau + mensuel |

## Exécution Lynis (baseline, Ex.2)

Premier passage Lynis 3.0.8 **réellement exécuté** sur les 2 VM (pas seulement planifié) pour établir
la référence (baseline) avant durcissement :

| VM | Hardening index | Suggestions | Warnings |
|---|---|---|---|
| helio-frontal (10.10.10.10) | **61/100** | 52 | 2 (paquet vulnérable détecté, pas de résolveur DNS secondaire) |
| helio-applicatif (10.10.10.20) | **62/100** | 53 | 2 |

Scores très proches (cohérent : les 2 VM sont clonées depuis la même base UBU-BASE). La cible Ex.3 est
de faire progresser ce score après application des mesures de durcissement #12-13.

Résultats bruts : `outputs/exercice2/lynis_helio-frontal_report.dat` + `.log` (et équivalent applicatif).
Le score de hausse documentée (critère du tableau ci-dessus) se mesurera par comparaison avec un
second passage après application des mesures de durcissement (#12-13 du plan de sécurité) en Ex.3.

Extrait des suggestions prioritaires (helio-frontal) : installer `fail2ban` (bannissement après
échecs d'authentification répétés), mot de passe GRUB, durcissement des règles de mot de passe
(`AUTH-9230/9262/9282/9286`), age minimum/maximum des mots de passe — cohérent avec les mesures
#1, #2, #9 du plan de sécurité.

## Priorisation des non-conformités constatées

1. **Nmap — Samba ouvert** : correction rapide (désinstallation), aucune dépendance bloquante → Ex.3 J1
2. **Trivy — CVE critiques image démo** : dépend du remplacement des images de démonstration par le
   code applicatif réel HelioMed en Ex.3, non corrigeable sur Juice Shop/DVWA eux-mêmes (volontairement
   vulnérables par conception)
3. **ZAP — alertes Medium** : ajout des en-têtes de sécurité (CSP, HSTS) au niveau Nginx → Ex.3 J1
