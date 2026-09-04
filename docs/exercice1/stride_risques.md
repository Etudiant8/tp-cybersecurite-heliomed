# STRIDE & matrice de risque — HelioMed

## Modélisation STRIDE (≥ 6 actifs requis)

Catégories : Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege.

> Les 6 lignes ci-dessous ciblent 6 actifs **distincts et nommés**, chacun repris tel quel du
> registre d'actifs (`registre_actifs.md`) — pas de menace rattachée à un simple constat de
> vulnérabilité (ex. « Samba exposé ») sans actif du registre derrière.

| Menace (STRIDE) | Actif visé (registre) | Vraisemblance (1-4) | Impact (1-4) | Risque (V×I) | Traitement envisagé |
|---|---|---|---|---|---|
| Information disclosure | BDD (conteneur MySQL, données santé simulées) | 3 | 4 | 12 — Élevé | Chiffrement au repos + cloisonnement réseau + secret manager (retirer le mot de passe en clair de docker-compose.yml) |
| Spoofing | VM helio-frontal / VM helio-applicatif (clés hôte SSH identiques, constaté par nmap) | 3 | 3 | 9 — Élevé | Régénérer les clés SSH après tout clonage de VM (`ssh-keygen -A` après suppression de `/etc/ssh/ssh_host_*`) |
| Tampering | Application patient (Juice Shop) — absence de Content-Security-Policy constatée par ZAP (116 alertes Medium) | 3 | 3 | 9 — Élevé | Ajouter les en-têtes de sécurité (CSP, X-Frame-Options, HSTS) au niveau Nginx |
| Repudiation | Comptes d'accès télétravail (SSH VM, VBoxManage) — mot de passe root partagé sans rotation, aucune action individuellement attribuable | 2 | 3 | 6 — Moyen | Comptes nominatifs + clés SSH individuelles + logs d'authentification centralisés (Wazuh, Ex. suivants) |
| Denial of service | Reverse proxy Nginx (helio-frontal) — seul point d'entrée public, aucune limitation de débit constatée | 2 | 3 | 6 — Moyen | Rate limiting (`limit_req` Nginx) + WAF en amont |
| Elevation of privilege | Composant applicatif de test (DVWA, volontairement vulnérable, exposé sur 10.10.10.20:8080) | 3 | 4 | 12 — Élevé | Isoler strictement du segment applicatif réel, jamais exposé hors réseau interne |

## Matrice Vraisemblance × Impact

Échelle 1 (faible) à 4 (critique) sur chaque axe. Risque = Vraisemblance × Impact.

| | Impact 1 | Impact 2 | Impact 3 | Impact 4 |
|---|---|---|---|---|
| **Vraisemblance 4** | 4 | 8 | 12 | 16 |
| **Vraisemblance 3** | 3 | 6 | 9 | 12 |
| **Vraisemblance 2** | 2 | 4 | 6 | 8 |
| **Vraisemblance 1** | 1 | 2 | 3 | 4 |

Seuils suggérés : 1-4 Faible · 5-8 Moyen · 9-12 Élevé · 13-16 Critique.
