# EBIOS Risk Manager (ANSSI) — HelioMed

## Sources de risque (≥ 3 requises)

| Source de risque | Motivation | Ressources/capacités |
|---|---|---|
| Cybercriminel opportuniste | Gain financier (revente données de santé, rançongiciel) | Automatisé, scanners de vulnérabilités connues |
| Concurrent / acteur malveillant ciblé | Espionnage, déstabilisation | Ciblé, compétences moyennes à élevées |
| Employé/prestataire négligent ou malveillant (interne) | Erreur, vengeance, revente d'accès | Accès légitime au SI, télétravail |

## Scénarios stratégiques (≥ 3 requis)

Pour chaque scénario : source de risque → objectif visé → chemin d'attaque de haut niveau (parties prenantes intermédiaires).

1. **Vol de données de santé via compromission de la BDD**
   Source : cybercriminel opportuniste → cible : BDD (données patients) → chemin : exploitation d'une CVE dans une image conteneur non patchée (constaté par Trivy) ou d'une des 116 alertes de configuration remontées par ZAP (CSP absent, CORS permissif) sur l'appli patient → pivot vers le serveur applicatif → exfiltration.

2. **Rançongiciel via poste de développeur en télétravail**
   Source : cybercriminel opportuniste → cible : disponibilité de la plateforme → chemin : phishing sur un développeur → accès Git/déploiement compromis → déploiement de charge malveillante.

3. **Abus d'accès interne par un prestataire**
   Source : interne négligent/malveillant → cible : confidentialité des données patients → chemin : accès legitime mal cloisonné → export massif de données.

4. **Usurpation d'un serveur HelioMed via clé SSH clonée (scénario constaté, pas seulement théorique)**
   Source : cybercriminel ciblé ou interne → cible : intégrité/disponibilité (spoofing d'un des 2 serveurs) → chemin : nmap révèle que helio-frontal et helio-applicatif partagent la **même clé hôte SSH** (clonage de VM sans régénération) → un attaquant positionné sur le réseau interne peut usurper l'un des deux serveurs sans déclencher d'alerte "clé inconnue" côté client SSH → MITM ou redirection de trafic administratif.

> Scénario 4 illustré par une preuve directe de l'Étape 1.1 (scan_helio.txt) — les scénarios 1-3 restent stratégiques/génériques, à approfondir en scénarios opérationnels si le temps le permet.
