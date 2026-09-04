# STRIDE & matrice de risque — HelioMed

## Modélisation STRIDE (≥ 6 actifs requis)

Catégories : Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege.

> Les 6 lignes ci-dessous ciblent 6 actifs **distincts et nommés**, chacun repris tel quel du
> registre d'actifs (`registre_actifs.md`) — pas de menace rattachée à un simple constat de
> vulnérabilité (ex. « Samba exposé ») sans actif du registre derrière.

### Grille de cotation (critères, pas de chiffres arbitraires)

| Niveau | Vraisemblance (V) | Impact (I) |
|---|---|---|
| 1 | Nécessite un accès/privilège que rien dans nos scans ne suggère atteignable | Gêne mineure, aucune donnée ni service affecté |
| 2 | Possible mais demande une étape supplémentaire non constatée (ex. compromission préalable d'un poste) | Dégradation limitée, contournable, pas de donnée de santé exposée |
| 3 | Réalisable **avec les moyens et accès déjà constatés dans nos propres scans** (accès réseau interne, service exposé) | Perte de confidentialité/intégrité significative, ou service public indisponible |
| 4 | Déjà démontré/trivial avec l'accès existant, ou vulnérabilité déjà exploitée publiquement (CVE connue) | Donnée de santé exposée (RGPD, art. 9) et/ou arrêt total d'un actif critique |

| Menace (STRIDE) | Actif visé (registre) | V | I | Risque | Justification (V / I) | Traitement envisagé |
|---|---|---|---|---|---|---|
| Information disclosure | BDD (conteneur MySQL, données santé simulées) | 3 | 4 | 12 — Élevé | V3 : accès réseau interne (constaté) + credentials en clair dans le repo suffisent, pas de 0-day requis. I4 : données de santé (art. 9 RGPD), sensibilité maximale. | Chiffrement au repos + cloisonnement réseau + secret manager (retirer le mot de passe en clair de docker-compose.yml) |
| Spoofing | VM helio-frontal / VM helio-applicatif (clés hôte SSH identiques, constaté par nmap) | 3 | 3 | 9 — Élevé | V3 : duplication de clé **déjà constatée** par nmap, il ne manque qu'un positionnement réseau interne (pas un accès privilégié). I3 : compromet l'intégrité des échanges admin, mais n'expose pas directement les données patients. | Régénérer les clés SSH après tout clonage de VM (`ssh-keygen -A` après suppression de `/etc/ssh/ssh_host_*`) |
| Tampering | Application patient (Juice Shop) — absence de Content-Security-Policy constatée par ZAP (116 alertes Medium) | 3 | 3 | 9 — Élevé | V3 : faille de configuration déjà mesurée par ZAP sans authentification requise. I3 : ouvre la porte à de l'injection de contenu côté client, pas un accès direct aux données. | Ajouter les en-têtes de sécurité (CSP, X-Frame-Options, HSTS) au niveau Nginx |
| Repudiation | Comptes d'accès télétravail (SSH VM, VBoxManage) — mot de passe root partagé sans rotation, aucune action individuellement attribuable | 2 | 3 | 6 — Moyen | V2 : demande qu'un accès légitime soit d'abord détourné (pas d'exploitation directe constatée). I3 : rend une investigation post-incident impossible à attribuer, impact organisationnel significatif mais pas vital. | Comptes nominatifs + clés SSH individuelles + logs d'authentification centralisés (Wazuh, Ex. suivants) |
| Denial of service | Reverse proxy Nginx (helio-frontal) — seul point d'entrée public, aucune limitation de débit constatée | 2 | 3 | 6 — Moyen | V2 : aucune protection anti-DoS constatée, mais suppose un attaquant qui cible spécifiquement ce point unique (pas une exposition triviale de masse). I3 : coupe l'accès à toute la plateforme (frontal unique) sans toucher aux données. | Rate limiting (`limit_req` Nginx) + WAF en amont |
| Elevation of privilege | Composant applicatif de test (DVWA, volontairement vulnérable, exposé sur 10.10.10.20:8080) | 3 | 4 | 12 — Élevé | V3 : application **conçue pour être exploitée**, exposée sur le réseau interne sans isolation constatée. I4 : un pivot depuis DVWA atteint directement le même segment que la BDD de santé. | Isoler strictement du segment applicatif réel, jamais exposé hors réseau interne |

## Matrice Vraisemblance × Impact

Échelle 1 (faible) à 4 (critique) sur chaque axe. Risque = Vraisemblance × Impact.

| | Impact 1 | Impact 2 | Impact 3 | Impact 4 |
|---|---|---|---|---|
| **Vraisemblance 4** | 4 | 8 | 12 | 16 |
| **Vraisemblance 3** | 3 | 6 | 9 | 12 |
| **Vraisemblance 2** | 2 | 4 | 6 | 8 |
| **Vraisemblance 1** | 1 | 2 | 3 | 4 |

Seuils suggérés : 1-4 Faible · 5-8 Moyen · 9-12 Élevé · 13-16 Critique.
