# Note d'analyse — Exercice 1 : Analyse de l'environnement de sécurité HelioMed

**Binôme :** Deven Imazoute
**Périmètre :** plateforme HelioMed (API Flask/Nginx/PostgreSQL, 2 serveurs Linux, données de santé RGPD), simulée dans un labo isolé avec OWASP Juice Shop (interface patient) et DVWA (composant applicatif de test).

---

## 1. Cartographie du labo (Étape 1.1)

**Architecture déployée :**
- `helio-frontal` (10.10.10.10) : reverse proxy Nginx 1.22.1, clone d'une VM socle Debian 12
- `helio-applicatif` (10.10.10.20) : Docker, héberge 3 conteneurs — Juice Shop (port 3000), DVWA (port 8080), MySQL (BDD, port 3306 interne)
- Réseau interne isolé `helio-lab` (10.10.10.0/24), NAT séparé pour l'installation des paquets uniquement
- Connectivité vérifiée : ping helio-frontal ↔ helio-applicatif, 0% de perte (voir `outputs/exercice1/preuve_vm_reseau.txt`)

**Outils exécutés et fichiers de sortie bruts** (dans `outputs/exercice1/`) :
- `scan_helio.txt` — nmap `-sV -sC` sur 10.10.10.0/24 depuis helio-applicatif, complété par un scan ciblé depuis helio-frontal
- `zap_alerts.json`, `zap_report.html`, `zap_alerts_summary.txt` — scan passif ZAP (spider + observation, sans attaque active) sur l'interface web via redirection NAT
- `trivy_juiceshop.txt`, `trivy_dvwa.txt`, `trivy_mysql.txt` — scan Trivy des 3 images du docker-compose

---

## 2. Registre des actifs et vulnérabilités (Étape 1.2)

Voir tableau complet dans `docs/exercice1/registre_actifs.md`. Points saillants issus des scans réels :

| Actif | Sensibilité | Vulnérabilité constatée |
|---|---|---|
| BDD (MySQL, données santé simulées) | Critique (santé) | Mot de passe root en clair dans `docker-compose.yml` |
| Application patient (Juice Shop) | Élevée | 0 High / 116 Medium / 160 Low (ZAP) : CSP absent, CORS permissif, en-têtes de sécurité manquants |
| helio-frontal / helio-applicatif | Moyenne/Critique | Samba (139/445) exposé inutilement ; **clés hôte SSH identiques** entre les deux VM (clonage sans régénération) |
| Composant applicatif de test (DVWA) | Critique | Application volontairement vulnérable exposée sur le réseau interne |
| Image Juice Shop | Élevée | 50 CVE (8 CRITICAL, 42 HIGH), dont crypto-js CVE-2023-46233 |
| Image DVWA | Critique | **805 CVE (254 CRITICAL, 551 HIGH)** — base Debian 9/Apache 2.4.25 obsolète |
| Image MySQL 5.7 | Critique (santé) | ~102 CVE cumulées sur les différents composants (OS, Python, runc) |

**Découverte non anticipée la plus significative :** le clonage des VM (`helio-frontal`/`helio-applicatif` depuis une base commune) a produit des **clés hôte SSH identiques** sur les deux serveurs — un attaquant positionné sur le réseau interne pourrait usurper l'un des deux serveurs sans que les clients SSH ne signalent d'anomalie de clé. Ce risque est invisible sans cartographie réseau systématique (nmap `ssh-hostkey` + script `Possible duplicate hosts`).

---

## 3. Menaces et analyse de risque (Étape 1.3)

### STRIDE (6 actifs, voir `docs/exercice1/stride_risques.md`)

6 lignes, 6 actifs **distincts et nommés** du registre (§2) — pas de menace rattachée à un simple constat sans actif identifié. Chaque V et I est justifié par un critère explicite (grille de cotation dans `stride_risques.md`), pas assigné arbitrairement :

| Menace | Actif (registre) | Risque (V×I) |
|---|---|---|
| Information disclosure | BDD (MySQL) | 12 — Élevé |
| Spoofing | VM helio-frontal / helio-applicatif (clés SSH dupliquées) | 9 — Élevé |
| Tampering | Application patient (CSP absent) | 9 — Élevé |
| Repudiation | Comptes d'accès télétravail (mdp root partagé, non attribuable) | 6 — Moyen |
| Denial of service | Reverse proxy Nginx (seul point d'entrée public) | 6 — Moyen |
| Elevation of privilege | DVWA exposé | 12 — Élevé |

### EBIOS Risk Manager (voir `docs/exercice1/ebios.md`)

3 sources de risque identifiées (cybercriminel opportuniste, acteur ciblé, interne négligent/malveillant) et 4 scénarios stratégiques, **chacun avec une partie prenante intermédiaire explicite** (mainteneur d'image Docker tierce, poste personnel du développeur en BYOD, prestataire d'infogérance à accès mutualisé, poste d'administration à l'origine du clonage VM) — dont un directement étayé par une preuve technique de ce TP (usurpation via clé SSH clonée).

---

## 4. Objectifs de sécurité et choix d'outils (Étape 1.4)

Voir `docs/exercice1/objectifs_securite.md` pour le détail. 5 objectifs mesurables rattachés au DICP, couvrant l'authentification, le chiffrement des données de santé, les CVE non corrigées, la disponibilité et la traçabilité des accès admin. **Traçabilité complète** : chaque objectif référence explicitement le(s) risque(s) STRIDE qu'il traite, et les 6 lignes de la matrice STRIDE (§3) sont toutes couvertes par au moins un des 5 objectifs.

**Choix des outils** justifié par la complémentarité constatée en pratique :
- **Nmap** a révélé un risque d'infrastructure (clés SSH dupliquées, Samba superflu) invisible aux scanners applicatifs.
- **ZAP** (scan passif) a quantifié 319 alertes de configuration HTTP sur l'application patient, sans risque de perturber le service — adapté à un contexte de données de santé.
- **Trivy** couvre la chaîne d'approvisionnement logicielle (CVE dans les images Docker), angle mort des deux outils précédents.

---

## 5. Logbook et part IA

Voir `docs/logbook.md`. La quasi-totalité de la mise en œuvre technique (création des VM, configuration réseau, déploiement, scans) a été réalisée avec Claude Code en pilotage direct (VBoxManage, guestcontrol), sous supervision et validation du binôme à chaque étape sensible (suppression de fichiers, choix d'architecture). Un incident réel a été rencontré et documenté : un kernel panic sur `helio-applicatif` pendant l'installation de Docker, dont la cause racine (disque hôte saturé, et non un problème de RAM comme supposé initialement) a été diagnostiquée et corrigée — cet incident est lui-même un exemple concret d'analyse de cause racine transférable à un contexte professionnel.
