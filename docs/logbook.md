# Logbook — TP Cybersécurité HelioMed

> Format attendu : à chaque étape, 2-3 lignes — ce qui a été fait, ce qui a échoué, comment ça a été corrigé.
> La part IA (assistance Claude Code) doit être explicitée pour chaque entrée où elle a été utilisée.

## Binôme
- Membre 1 : Deven Imazoute
- Membre 2 : _(TP réalisé en solo)_

## Phase 0 — Installation environnement

| Date/Heure | Action | Résultat | Part IA |
|---|---|---|---|
| 2026-09-04 | Installation VirtualBox 7.2.2, Nmap, Trivy, OWASP ZAP (winget) | OK | Claude Code : commandes winget exécutées |
| 2026-09-04 09:16 | Création helio-frontal/helio-applicatif via ISO Debian 12 netinst + unattended install | Échec : les 2 VM en parallèle ont saturé la RAM hôte (943 Mo libres/16 Go) → VirtualBox les a auto-pausées après ~45 min. Corrigé en arrêtant Docker Desktop/WSL (non nécessaires à ce stade) puis en supprimant les 2 VM pour repartir sur une base plus légère | Claude Code : diagnostic (captures d'écran comparées, compteurs mémoire hôte) + décision de bascule |
| 2026-09-04 10:18 | Clonage de UBU-BASE (VM socle Debian 12 déjà préparée pour un TP antérieur) en helio-frontal + helio-applicatif, 1,5 Go RAM/1 CPU chacune | OK, boot en quelques secondes (vs 40+ min bloqué avec l'ISO) | Claude Code : VBoxManage clonevm + modifyvm |
| 2026-09-04 10:25 | Connexion VBoxManage guestcontrol (root + mdp hérité de UBU-BASE) | Échec initial : `guestcontrol run` duplique le 1er argument après `--` en plus de --exe comme argv[0] → hostname changé par erreur en littéral "hostname". Corrigé en ne répétant plus le nom du programme après `--` | Claude Code : diagnostic de la syntaxe VBoxManage |
| 2026-09-04 14:35 | **Revue sécurité du rapport Ex.1** : le mot de passe root hérité de UBU-BASE avait été commité en clair dans ce logbook et `preuve_vm_reseau.txt` — ironique pour un TP sécurité qui liste ce même anti-pattern comme vulnérabilité d'HelioMed | Corrigé : mot de passe changé sur les 2 VM (`chpasswd`), nouveau mot de passe stocké uniquement dans `.credentials.local` (gitignore), fichiers déjà commités nettoyés et historique Git réécrit | Claude Code, sur demande explicite de correction sévère |
| 2026-09-04 10:27 | Correction hostnames (helio-frontal/helio-applicatif) + IP statique sur enp0s8 (10.10.10.10/10.10.10.20, réseau interne isolé "helio-lab") | OK | Claude Code : hostnamectl + fichiers interfaces.d poussés via guestcontrol copyto |
| 2026-09-04 10:30 | Test ping helio-frontal → helio-applicatif | OK, 0% perte (voir outputs/exercice1/preuve_vm_reseau.txt) | Claude Code |
| 2026-09-04 10:40 | Tentative installation Docker sur helio-applicatif | Échec : kernel panic pendant l'install (module loading), puis pauses répétées au redémarrage malgré plus de RAM/CPU | Claude Code : diagnostic initial orienté (à tort) mémoire/CPU |
| 2026-09-04 10:45 | Investigation racine du problème | **Cause réelle trouvée** : disque C: hôte à 0 octet libre (475 Go utilisés) → écritures sur le .vdi impossibles → kernel panic + pauses VirtualBox. Non lié à la RAM/CPU comme supposé initialement | Claude Code : `Get-PSDrive`, analyse des dossiers VirtualBox VMs |
| 2026-09-04 10:47 | Nettoyage disque : suppression de 3 dossiers de VM orphelins (Win client, debian, PFsense — non enregistrés dans VirtualBox) | 25,75 Go libérés | Claude Code (validé par l'utilisateur avant suppression) |
| 2026-09-04 10:48 | Nouvelle tentative Docker sur helio-applicatif | ✅ OK, aucun crash — confirme que la cause était bien le disque plein | Claude Code |
| 2026-09-04 10:52 | Déploiement infra/docker-compose.yml (Juice Shop + DVWA + MySQL) sur helio-applicatif | ✅ 3 conteneurs up, ports 3000/8080 en écoute | Claude Code |
| 2026-09-04 10:55 | Config Nginx reverse proxy sur helio-frontal -> 10.10.10.20:3000/8080 | ✅ HTTP 200 (Juice Shop) et 302 (DVWA) via localhost | Claude Code |
| 2026-09-04 12:08 | nmap -sV -sC sur 10.10.10.0/24 depuis helio-applicatif | ✅ 2 hôtes découverts ; trouvaille inattendue : clés hôte SSH identiques entre les 2 VM (clonage sans régénération) + Samba exposé inutilement | Claude Code |
| 2026-09-04 12:11 | Scan passif ZAP (spider + observation) sur helio-frontal via redirection NAT host:8888 | ✅ 319 alertes (0 High/116 Medium/160 Low/43 Info) | Claude Code |
| 2026-09-04 12:16 | Scan Trivy des 3 images **depuis l'hôte Windows** | Échec répété : timeouts après 10, 20 puis 40 min sur l'image Juice Shop (node_modules volumineux). Diagnostic : réseau et disque OK, cause probable = Windows Defender interceptant l'extraction de milliers de petits fichiers. Corrigé en installant Trivy **dans** helio-applicatif et en scannant les images locales déjà pullées (< 3 min au lieu de 40+) | Claude Code |
| 2026-09-04 13:30 | Scan Trivy local (dans la VM) des 3 images | ✅ Juice Shop: 50 CVE (8 CRIT/42 HIGH) ; DVWA: 805 CVE (254 CRIT/551 HIGH) ; MySQL: ~102 CVE cumulées | Claude Code |

## Exercice 1 — Analyse de l'environnement de sécurité

Toutes les étapes (1.1 découverte réseau/ZAP/Trivy, 1.2 registre des actifs, 1.3 STRIDE/EBIOS,
1.4 objectifs) sont couvertes dans le journal Phase 0 ci-dessus (colonne « Date/Heure ») et dans
`docs/exercice1/note_analyse.md` — pas de table séparée pour éviter la duplication.

## Exercice 2 — Conception et planification de la sécurité

| Date/Heure | Étape | Ce qui a été fait | Échec rencontré | Correction | Part IA |
|---|---|---|---|---|---|
| 2026-09-04 15:40 | 2.1 Plan de sécurité | 15 mesures rédigées, mappées ISO 27002:2022 + ANSSI, dont 5 corrigent directement des failles trouvées en Ex.1 | — | — | Claude Code : rédaction + mapping référentiels |
| 2026-09-04 15:42 | 2.2 Timeline DevSecOps + pipeline CI | `.github/workflows/security.yml` (gitleaks + Trivy fs + Trivy sur les 3 images) | Version `aquasecurity/trivy-action@0.24.0` inexistante → échec des 4 jobs Trivy au 1er push | Épinglé sur `@master`, tous les jobs passent au 2e push | Claude Code : écriture + debug via `gh run view` |
| 2026-09-04 15:45 | 2.3 Schéma d'architecture cible | SVG + PNG + `.drawio` — 3 zones (DMZ/applicative/données), IP réelles du labo, WAF/pare-feu/TLS/SIEM | — | — | Claude Code : conception + export (headless Edge) |
| 2026-09-04 15:47 | 2.4 Plan de tests | Table remplie + **audit Lynis réel exécuté** sur les 2 VM (pas seulement planifié) | — | — | Claude Code : installation + exécution Lynis via guestcontrol |
| 2026-09-04 15:50 | 2.5 Plan de réponse aux incidents | Rôles, 4 niveaux de gravité, 6 phases NIST, obligation CNIL 72h rédigés | — | — | Claude Code |

**Contexte** : Ex.2 à déposer 17h30 le jour même de la remise d'Ex.1, délai serré (~2h). Le pipeline CI
a été **vérifié en le poussant réellement sur GitHub** (`gh run view`) plutôt que simplement écrit et
supposé fonctionnel — il a d'ailleurs révélé une vraie clé privée embarquée dans l'image DVWA via le
scanner de secrets Trivy, preuve supplémentaire que l'outil tourne réellement.
