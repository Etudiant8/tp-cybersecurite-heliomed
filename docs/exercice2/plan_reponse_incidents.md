# Plan de réponse aux incidents (ébauche) — Étape 2.5

## Rôles

| Rôle | Qui (HelioMed) | Responsabilité |
|---|---|---|
| **Décideur** (Incident Manager) | Responsable technique/DSI | Déclenche/lève le plan, arbitre les décisions à impact métier (ex. coupure de service), valide la notification CNIL |
| **Agit** (équipe technique) | Administrateur système/réseau + développeur d'astreinte | Confinement technique, éradication, restauration des services |
| **Communique** | Responsable RGPD / DPO (ou décideur si absent) | Notification CNIL (72h), information des patients concernés si risque élevé, communication interne |
| **Support** | Prestataire infogérance (si sous contrat) | Renfort technique, accès aux VM/infrastructure en cas d'incapacité de l'équipe interne |

**Astreinte** : un canal de contact unique (téléphone + messagerie chiffrée) doit être défini et testé
avant mise en production — hors périmètre technique de ce TP, mais condition de fonctionnement du plan.

## Niveaux de gravité

| Niveau | Exemple | Délai de réaction |
|---|---|---|
| **P1 — Critique** | Fuite avérée de données de santé, ransomware actif, service de santé totalement indisponible | Immédiat (< 30 min), décideur alerté sans délai |
| **P2 — Élevé** | Compromission d'un compte admin, CVE critique activement exploitée, dégradation majeure du service | < 2 h |
| **P3 — Modéré** | Alerte SIEM suspecte non confirmée, vulnérabilité découverte non exploitée, port ouvert inattendu (type Samba, cf. Ex.1) | < 24 h |
| **P4 — Faible** | Alerte de configuration mineure (ex. alerte ZAP Low), écart de durcissement | Traitement dans le cycle normal (backlog sécurité) |

## Les 6 phases NIST (SP 800-61)

1. **Préparation** — plan documenté (ce document), astreinte définie, SIEM opérationnel (§2.3), sauvegardes
   testées (mesure #11), accès d'urgence (bastion) préparés à l'avance.
2. **Identification** — détection via alerte SIEM (Wazuh, §2.3) ou signalement, qualification du niveau
   de gravité (tableau ci-dessus), ouverture d'un ticket incident horodaté.
3. **Confinement** — isolement réseau de l'actif compromis (ex. retrait de la zone applicative via le
   pare-feu interne, §2.3), rotation immédiate des secrets potentiellement exposés (mesure #9), sans
   attendre l'éradication complète pour limiter la propagation.
4. **Éradication** — suppression de la cause racine (patch, suppression d'un accès compromis,
   regénération des clés — cf. mesure #3 directement issue de la découverte Ex.1 des clés SSH dupliquées).
5. **Rétablissement** — restauration depuis sauvegarde testée (mesure #11) ou remise en service
   progressive avec surveillance renforcée, validation avant retour en exploitation normale.
6. **Retour d'expérience (post-mortem)** — analyse sans recherche de faute, mise à jour du registre
   d'actifs et de la matrice STRIDE (Ex.1) si un risque non anticipé est découvert, mise à jour de ce
   plan si une lacune de procédure est identifiée.

## Obligation RGPD — notification CNIL sous 72h

HelioMed traite des **données de santé** (catégorie particulière, art. 9 RGPD). En cas de violation de
données (accès, divulgation ou perte non autorisés) présentant un risque pour les droits et libertés
des personnes concernées :

- **Notification à la CNIL sous 72 heures** après la prise de connaissance de la violation
  (art. 33 RGPD), même si toutes les informations ne sont pas encore disponibles — une notification
  initiale incomplète peut être complétée ultérieurement.
- **Information des personnes concernées** (art. 34 RGPD) si le risque est **élevé** — obligatoire pour
  une fuite de données de santé compte tenu de leur sensibilité, sauf mesures de protection ayant rendu
  les données inintelligibles (ex. chiffrement effectif, cf. mesure #6).
- Le déclenchement du délai de 72h correspond à la phase **2 — Identification** ci-dessus : le
  chronométrage RGPD démarre dès la qualification de l'incident comme violation de données, pas
  seulement après confirmation complète — d'où l'importance d'un rôle « Communique » alerté dès P1/P2.
- Registre des violations tenu par le DPO, même pour les incidents non notifiés à la CNIL (obligation
  de traçabilité art. 33.5 RGPD).
