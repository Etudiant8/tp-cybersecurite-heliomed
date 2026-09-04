# Plan de sécurité HelioMed — Étape 2.1

15 mesures (minimum 12 requis), chacune reliée à un contrôle **ISO/IEC 27002:2022** (Annexe A) et
à une recommandation du **Guide d'hygiène informatique ANSSI** (42 mesures). Les mesures marquées
**[Ex.1]** corrigent directement une vulnérabilité réelle constatée lors de l'audit initial —
elles ne sont pas génériques.

> **Note de méthode** : les numéros de règles ANSSI ci-dessous ont été **vérifiés a posteriori**
> contre le contenu réel du guide (via une recherche web ce jour, cf. logbook) après qu'une
> première relecture sévère a révélé que plusieurs numéros cités initialement de mémoire étaient
> erronés — ex. la segmentation réseau avait été rattachée à « R15-17 » alors que c'est la
> **règle 21**. Les numéros ci-dessous s'appuient sur la version « 40 règles » (édition de
> référence la plus largement documentée en ligne) ; la numérotation de l'édition « 42 mesures »
> en vigueur peut différer de ±1-2 sur certaines sections non recoupées avec certitude.

| # | Mesure | Catégorie | ISO/IEC 27002:2022 | ANSSI (guide d'hygiène) |
|---|---|---|---|---|
| 1 | Authentification multifacteur (MFA) sur tous les accès admin (GitHub, VPN, bastion SSH) | Authentification | 5.17 Authentication information · 8.5 Secure authentication | R9-R10 — Règles de mots de passe + moyens techniques pour faire respecter l'authentification |
| 2 | Clés SSH **individuelles et nominatives**, jamais de compte/mot de passe partagé entre machines **[Ex.1]** | Authentification | 5.16 Identity management · 8.2 Privileged access rights | R8 — Identifier nommément chaque individu ayant accès au SI |
| 3 | Régénération systématique des clés hôte SSH et secrets après tout clonage de VM **[Ex.1]** | Authentification / Durcissement | 8.9 Configuration management · 8.24 Use of cryptography | R12 — Renouveler systématiquement les éléments d'authentification par défaut |
| 4 | Segmentation réseau en 3 zones : DMZ frontale, zone applicative, zone données (voir schéma §2.3) | Cloisonnement réseau | 8.22 Segregation of networks · 8.20 Networks security | R21 — Mettre en place des réseaux cloisonnés (sous-réseau protégé) |
| 5 | Pare-feu avec règles de flux explicites (allow-list), aucun flux zone données → Internet | Cloisonnement réseau | 8.20 Networks security · 8.21 Security of network services | R4 — Limiter les accès Internet au strict nécessaire · R21 |
| 6 | Chiffrement des données de santé au repos (BDD + volumes de sauvegarde, AES-256) | Chiffrement | 8.24 Use of cryptography | R31 — Chiffrer les données sensibles, en particulier sur les équipements exposés |
| 7 | Chiffrement en transit : TLS 1.3 sur le reverse proxy **et** entre frontal/applicatif (actuellement HTTP en interne, cf. Ex.1) | Chiffrement | 8.24 Use of cryptography · 5.14 Information transfer | R23 — Utiliser systématiquement des applications et protocoles sécurisés |
| 8 | Gestionnaire de secrets (Vault/SOPS) — suppression des mots de passe en clair dans `docker-compose.yml` **[Ex.1]** | Gestion des secrets | 8.24 Use of cryptography · 5.17 Authentication information | R9 — Règles de choix et de dimensionnement des mots de passe |
| 9 | Rotation programmée des secrets (90 jours) et rotation immédiate en cas de fuite constatée **[Ex.1]** | Gestion des secrets | 8.24 · 5.18 Access rights | R12 — Renouveler les éléments d'authentification par défaut |
| 10 | Journalisation centralisée (auth SSH, Nginx, applicatif) vers un SIEM avec horodatage NTP synchronisé | Journalisation | 8.15 Logging · 8.16 Monitoring activities | R26-R27 — Objectifs de supervision + analyse des événements journalisés |
| 11 | Sauvegardes chiffrées et testées de la BDD santé (RPO ≤ 24h, restauration testée trimestriellement) | Sauvegardes | 8.13 Information backup | R36 — Disposer d'un plan de sauvegarde des données essentielles, le tenir à jour |
| 12 | Durcissement des serveurs : désinstallation des services inutiles (**Samba trouvé actif sans usage, Ex.1**), désactivation des accès root direct | Durcissement | 8.9 Configuration management · 8.8 Management of technical vulnerabilities | R14 — Niveau de sécurité homogène sur l'ensemble du parc |
| 13 | Gestion des correctifs (patch management) avec délai maximal (7 j critique / 30 j haute) | Durcissement | 8.8 Management of technical vulnerabilities | R6-R7 — Connaître les modalités de MAJ + définir une politique de mise à jour |
| 14 | Sécurité applicative OWASP Top 10 : en-têtes CSP/HSTS/X-Frame-Options manquants sur l'appli patient **[Ex.1]**, validation des entrées, requêtes préparées | Sécurité applicative | 8.26 Application security requirements · 8.28 Secure coding | R23 — Applications et protocoles sécurisés |
| 15 | Scan de vulnérabilités (SCA/DAST) intégré au pipeline CI/CD à chaque commit (voir §2.2) | Sécurité applicative / DevSecOps | 8.29 Security testing in development · 8.8 | R40 — Audits de sécurité périodiques (le scan continu en est la version automatisée) |

## Priorisation

Les mesures **1, 2, 3, 8, 9** traitent directement les failles **critiques constatées en Ex.1**
(clés SSH dupliquées, secrets en clair) et sont classées **prioritaires immédiates**. Les mesures
**4-7, 12, 14** sont **prioritaires court terme** (déployées en Ex.3). Les mesures **10, 11, 13, 15**
sont **structurelles** (mise en place progressive, cf. timeline §2.2).
