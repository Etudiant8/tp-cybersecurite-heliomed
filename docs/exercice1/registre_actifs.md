# Registre des actifs et vulnérabilités — HelioMed

> À compléter à partir des résultats réels des scans (nmap, ZAP, Trivy), pas d'une liste générique.
> Classement de sensibilité RGPD : les données de santé sont sensibles par défaut.

| Actif | Type | Sensibilité | Vulnérabilité constatée | Source (outil) |
|---|---|---|---|---|
| Reverse proxy Nginx (helio-frontal, 10.10.10.10) | Infrastructure | Moyenne | Nginx 1.22.1 exposé sur le port 80 (HTTP non chiffré) ; en-tête serveur visible (`Server: nginx/1.22.1`, fingerprinting facilité) | nmap/ZAP |
| Application patient (Juice Shop via proxy) | Application | Élevée | 0 High / 116 Medium / 160 Low sur scan passif ZAP : Content-Security-Policy absent, en-têtes de sécurité manquants (X-Frame-Options, HSTS…), CORS permissif (`Access-Control-Allow-Origin: *`) | ZAP |
| Composant applicatif de test (DVWA, 10.10.10.20:8080) | Application | Critique | Application intentionnellement vulnérable exposée sur le réseau interne, servie par Apache 2.4.25 | nmap |
| BDD (conteneur MySQL, données simulant patients/santé) | Donnée | Critique (santé) | Mot de passe root faible codé en clair dans docker-compose.yml (`changeme_lab_only`), CVE à compléter (voir trivy_mysql.txt) | Trivy/revue manuelle |
| VM helio-frontal (10.10.10.10) | Infrastructure | Moyenne | Service Samba (139/445) exposé alors qu'inutile pour HelioMed (hérité du clonage de la VM socle) ; clé hôte SSH ED25519/ECDSA **identique** à helio-applicatif (VM clonées sans régénération) | nmap |
| VM helio-applicatif/BDD (10.10.10.20) | Infrastructure | Critique | Idem : Samba exposé inutilement, clé hôte SSH dupliquée avec helio-frontal (risque de spoofing/MITM SSH indétectable) | nmap |
| Image Juice Shop (appli patient) | Application | Élevée | 50 CVE (8 CRITICAL, 42 HIGH) — dont crypto-js CVE-2023-46233 (PBKDF2 1000x plus faible que prévu) | Trivy |
| Image DVWA (composant applicatif de test) | Application | Critique | **805 CVE (254 CRITICAL, 551 HIGH)** — image basée sur Debian 9/Apache 2.4.25 obsolète, illustre le risque d'une image non maintenue en production | Trivy |
| Image MySQL 5.7 (BDD) | Donnée | Critique (santé) | ~102 CVE cumulées (32 HIGH couche OS, 9 HIGH composant Python, 61 dont 4 CRITICAL sur runc/containerd embarqués) | Trivy |
| Dépôt Git (code + configs + secrets potentiels) | Donnée | Élevée | Mot de passe MySQL présent en clair dans infra/docker-compose.yml commité | revue manuelle |
| Comptes d'accès télétravail (SSH VM, VBoxManage) | Identité | Élevée | Mot de passe identique root/admintp partagé sur toutes les VM clonées depuis UBU-BASE (pas de rotation par machine) | revue manuelle |
