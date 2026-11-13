---
title: "Intune + Defender for Business : Annexes MVC nLPD"
description: "Annexes du guide Intune + Defender for Business : glossaire, checklist complète, références nLPD, adaptations sectorielles, POC et production, licences."
lang: fr
---
<style>
  header, footer { display: none !important; }
  .wrapper {
    max-width: 900px !important;
    margin: 0 auto !important;
    float: none !important;
    position: relative !important;
    padding: 40px 20px !important;
    font-family: "Helvetica Neue", Helvetica, Arial, sans-serif !important;
    font-size: 1.1em !important;
  }
  section {
    width: 100% !important;
    float: none !important;
    margin: 0 !important;
  }
  h1, h2 { text-align: center; }
  table { width: 100%; display: table; margin: 20px 0; }
</style>

<h1>Intune + Defender for Business : Annexes</h1>
<h2>MVC nLPD · Appareils et menaces · v3.7</h2>

> [← Retour au guide Intune + Defender](index.html) · [← Retour à la procédure de déploiement](procedure-deploiement.html)

## Sommaire

- [Annexe A : Glossaire](#annexe-a)
- [Annexe B : Checklist MVC complète avec références nLPD](#annexe-b)
- [Annexe C : Références nLPD, mesures techniques couvertes](#annexe-c)
- [Annexe D : Adaptations sectorielles](#annexe-d)
- [Annexe E : Différences POC et production](#annexe-e)
- [Annexe F : Licences](#annexe-f)

---

## Annexe A : Glossaire {#annexe-a}

| Terme | Définition |
|-------|------------|
| **AIR** | Investigation et remédiation automatisées (Auto-Investigation and Remediation). Réponse automatisée de Defender aux menaces détectées. Intégrée à l'antivirus depuis l'évolution décrite par Microsoft. |
| **APNs** | Apple Push Notification service. Certificat requis pour gérer les appareils iOS par Intune, à renouveler chaque année. |
| **ASR** | Attack Surface Reduction. Règles qui bloquent des comportements utilisés par les malwares (macros, scripts, vol d'identifiants). |
| **Autopilot** | Provisionnement automatique d'un poste Windows au premier démarrage, par Intune. |
| **BitLocker** | Chiffrement de disque natif de Windows. Clé de récupération stockée dans Entra ID. |
| **BitLocker To Go** | Extension de BitLocker aux supports amovibles. |
| **Centre d'actions** | Interface Defender XDR qui regroupe les actions automatisées et en attente. |
| **CVE** | Common Vulnerabilities and Exposures. Identifiant standard d'une faille connue. |
| **Device Control** | Contrôle des périphériques (USB, imprimantes, Bluetooth) par Intune. |
| **EDR** | Endpoint Detection and Response. Surveillance comportementale, isolation et investigation sur les postes. |
| **EICAR** | Fichier de test antivirus standardisé, inoffensif. |
| **Enhanced Phishing Protection** | Fonction de Windows 11 qui avertit l'utilisateur qui saisit son mot de passe Windows dans une application non approuvée. |
| **LAPS (Windows LAPS)** | Local Administrator Password Solution. Mot de passe administrateur local unique par poste, renouvelé automatiquement. |
| **Network Protection** | Extension de SmartScreen à toutes les connexions réseau du poste. |
| **PFPDT** | Préposé fédéral à la protection des données et à la transparence, autorité de contrôle suisse. |
| **Remote Help** | Prise en main à distance intégrée à Intune, avec journalisation. Licence complémentaire à vérifier. |
| **Safe Attachments** | Analyse des pièces jointes dans un bac à sable avant livraison. |
| **Safe Links** | Réécriture et vérification des URL des e-mails au moment du clic. |
| **Tamper Protection** | Empêche une application ou un script de désactiver Defender. |
| **Wipe sélectif (retrait)** | Suppression des données et applications de l'entreprise sans effacer les données personnelles. Soumis à validation administrative. |
| **Web Content Filtering** | Filtrage de catégories de sites web par Defender. |

---

## Annexe B : Checklist MVC complète avec références nLPD {#annexe-b}

### Prérequis et inscription

| ☐ | Action | Portail / Chemin | nLPD |
|---|--------|------------------|------|
| ☐ | Groupes `GRP-Appareils-Windows`, `GRP-Appareils-iOS`, `GRP-Appareils-Android` | entra.microsoft.com → Groupes | |
| ☐ | Connecteur Intune → Defender activé | Intune → Administration des clients | art. 8 |
| ☐ | Inscription automatique Windows activée | Intune → Appareils → Inscription Windows | |
| ☐ | APNs créé, date d'expiration documentée | Intune → Appareils → iOS | |
| ☐ | Google Play géré lié | Intune → Appareils → Android | |
| ☐ | Appareils inscrits et dans le bon groupe | Intune → Appareils | |

### Chiffrement et contrôle des périphériques

| ☐ | Action | Portail / Chemin | nLPD |
|---|--------|------------------|------|
| ☐ | BitLocker (algorithme et étendue choisis), clés dans Entra ID | Intune → Sécurité du point de terminaison → Chiffrement de disques | art. 8 |
| ☐ | BitLocker To Go configuré (si USB autorisées) | Intune → Sécurité du point de terminaison → Chiffrement de disques | art. 8 |
| ☐ | Device Control USB testé | Intune → Réduction de la surface d'attaque | art. 8 |

### Endpoint Security

| ☐ | Action | Portail / Chemin | nLPD |
|---|--------|------------------|------|
| ☐ | EDR : postes intégrés | Intune → Détection et réponse des points de terminaison | art. 8 |
| ☐ | Antivirus et Tamper Protection | Intune → Antivirus | art. 8 |
| ☐ | Pare-feu : trois profils actifs | Intune → Pare-feu | art. 8 |
| ☐ | ASR : Blocage après période d'audit | Intune → Réduction de la surface d'attaque | art. 8 |
| ☐ | Network Protection : Blocage après audit | Intune → Réduction de la surface d'attaque | art. 8 |
| ☐ | Windows LAPS, lecture limitée, revue des consultations | Intune → Protection du compte ; Entra ID → Appareils | art. 8 |

### Navigation web, investigation, vulnérabilités

| ☐ | Action | Portail / Chemin | nLPD |
|---|--------|------------------|------|
| ☐ | SmartScreen : Blocage | Intune → Profils de configuration | |
| ☐ | Enhanced Phishing Protection (Windows 11) | Intune → Profils de configuration | art. 8 |
| ☐ | Filtrage de contenu web : politique unique, exceptions documentées | security.microsoft.com → Paramètres | |
| ☐ | AIR : Centre d'actions suivi | security.microsoft.com → Centre d'actions | art. 8 |
| ☐ | Gestion des vulnérabilités : recommandations critiques traitées | security.microsoft.com → Vulnérabilités | art. 8 |

### Conformité et profils

| ☐ | Action | Portail / Chemin | nLPD |
|---|--------|------------------|------|
| ☐ | Conformité Windows | Intune → Appareils → Conformité | art. 8 |
| ☐ | Conformité iOS | Intune → Appareils → Conformité | art. 8 |
| ☐ | Conformité Android | Intune → Appareils → Conformité | art. 8 |
| ☐ | Actions de non-conformité, retrait validé par un administrateur | Intune → Conformité → Actions | art. 8 |
| ☐ | Windows Hello Entreprise | Intune → Inscription Windows | art. 8 |
| ☐ | Windows Update for Business : deux anneaux | Intune → Anneaux de mise à jour | art. 8 |
| ☐ | Messagerie mobile (Outlook mobile sur Android) | Intune → Profils et stratégies d'application | |

### Defender for Office 365

| ☐ | Action | Portail / Chemin | nLPD |
|---|--------|------------------|------|
| ☐ | Liens fiables, tous les utilisateurs | security.microsoft.com | art. 8 |
| ☐ | Pièces jointes fiables, Blocage avec livraison dynamique | security.microsoft.com | art. 8 |
| ☐ | Anti-hameçonnage, utilisateurs protégés | security.microsoft.com | art. 8 |
| ☐ | Anti-spam et quarantaine utilisateur | security.microsoft.com | |

### Alertes et intégration Entra ID

| ☐ | Action | Portail / Chemin | nLPD |
|---|--------|------------------|------|
| ☐ | Notifications e-mail critique et élevée | security.microsoft.com → Paramètres | art. 8 |
| ☐ | CA-04 en Application après analyse | entra.microsoft.com → Accès conditionnel | art. 8 |
| ☐ | CA-08 en Application après analyse | entra.microsoft.com → Accès conditionnel | art. 8 |
| ☐ | Procédure d'annonce des violations documentée par le client | Registre de sécurité | art. 24 |
| ☐ | Tests T1 à T17 documentés | Dossier de remise client | |

---

## Annexe C : Références nLPD, mesures techniques couvertes {#annexe-c}

Ce tableau peut servir de pièce justificative lors d'un audit du PFPDT. Référence : **nLPD RS 235.1**, en vigueur depuis le 1er septembre 2023. Délai d'annonce d'une violation : **« dès que possible »** (pas de délai fixe de 72 heures comme dans le RGPD). Autorité : **PFPDT**. Ces mesures techniques contribuent à la sécurité des données au sens de la nLPD ; elles ne garantissent pas à elles seules la conformité de l'entreprise.

| Article nLPD | Obligation | Mesure technique déployée | Section |
|--------------|------------|---------------------------|---------|
| **art. 8 al. 1** | Mesures techniques adaptées au risque | Antivirus, Tamper Protection, ASR, EDR | Partie 4 |
| **art. 8 al. 1** | Protection des données au repos | BitLocker et BitLocker To Go | §3.1, §3.2 |
| **art. 8 al. 1** | Prévention de l'exfiltration physique | Device Control USB | §3.3 |
| **art. 8 al. 1** | Réduction de la surface d'attaque | ASR, Network Protection, SmartScreen | §4.5, §4.6, Partie 5 |
| **art. 8 al. 1** | Limitation du mouvement latéral | Windows LAPS | §4.7 |
| **art. 8 al. 1** | Authentification forte des postes | Windows Hello Entreprise | §9.1 |
| **art. 8 al. 1** | Contrôle d'accès par état de sécurité | Conformité Intune et CA-08 | Parties 8 et 12 |
| **art. 8 al. 1** | Inventaire et correction des vulnérabilités | Gestion des vulnérabilités | Partie 7 |
| **art. 8 al. 1** | Maintien à jour | Windows Update for Business | §9.2 |
| **art. 8 al. 1** | Protection de la messagerie | Safe Links, Safe Attachments, anti-hameçonnage | Partie 10 |
| **art. 8 al. 1** | Détection et documentation des incidents (contribution) | AIR, Centre d'actions, notifications e-mail | §6.1, §11.1 |
| **art. 8 al. 2** | Éviter les violations de la sécurité des données (la loi ne prescrit pas de mesure nommée) | L'ensemble des mesures ci-dessus y contribue | Parties 3 à 10 |
| **art. 24** | Annonce des violations de la sécurité des données au PFPDT | Procédure organisationnelle du client. Les journaux et alertes de ce guide facilitent la détection et la documentation, sans remplacer l'annonce. | Checklist |
| **art. 6 al. 2** | Proportionnalité (appareils personnels) | User Enrollment iOS et profil de travail Android : pas d'accès aux données personnelles | Partie 2 |

---

## Annexe D : Adaptations sectorielles {#annexe-d}

| Secteur | Point de rupture | Adaptation recommandée |
|---------|------------------|------------------------|
| **PME générique** | Aucun | MVC tel quel. |
| **Fiduciaire / comptabilité** | Logiciels comptables anciens (Banana, Sage) bloqués par ASR | Repérer les faux positifs en mode Audit, ajouter des exclusions documentées. |
| **Cabinet médical** | Données médicales très sensibles | Device Control : blocage total. Score de risque Defender : « Aucun risque » exigé. Safe Attachments : vérifier que le délai de livraison dynamique est acceptable. |
| **Avocats / notaires** | Confidentialité maximale | Device Control : blocage total. Filtrage web : politique unique pour toute l'organisation ; le blocage de services précis (Dropbox, WeTransfer) n'est pas couvert par les catégories relevées, à valider. Enhanced Phishing Protection activée. |
| **Agence de communication** | Appareils personnels, réseaux sociaux métier | Device Control assoupli pour les postes créatifs. Filtrage web : ne pas bloquer la catégorie « Réseaux sociaux » (la politique est unique pour toute l'organisation). |
| **Architecture / ingénierie** | Fichiers volumineux (plans, BIM) | Safe Attachments : valider avec le client la tolérance au délai pour les gros fichiers. |

---

## Annexe E : Différences POC et production {#annexe-e}

| Point | POC (test) | Production (recommandé) |
|-------|------------|-------------------------|
| **Inscription Windows** | Manuelle, postes existants | Autopilot pour les nouveaux postes |
| **BitLocker** | TPM sans PIN | TPM + PIN pour la direction et les administrateurs |
| **Device Control USB** | Blocage total ou liste blanche simple | Liste blanche par identifiant fabricant et produit |
| **ASR** | Audit 14 jours, puis Blocage | Audit, validation sur un groupe pilote, puis Blocage règle par règle |
| **AIR** | Suivi occasionnel du Centre d'actions | Revue hebdomadaire documentée du Centre d'actions |
| **Score de risque Defender (conformité)** | « Faible » accepté | « Aucun risque » pour la direction et les administrateurs |
| **LAPS** | Lecture par l'administrateur général | Rôle dédié, revue périodique des journaux d'audit |
| **Gestion des vulnérabilités** | Revue mensuelle manuelle | Tâches de remédiation Intune |
| **Alertes** | E-mail administrateur uniquement | Liste de l'équipe de sécurité et ticketing si disponible |
| **Filtrage web** | Catégories malveillantes seulement | Catégories validées par la direction |
| **APNs iOS** | Géré manuellement | Rappel de renouvellement annuel au calendrier de l'administrateur |
| **Retrait des appareils non conformes** | Validation ad hoc | Procédure écrite de validation, notamment pour le BYOD |

---

## Annexe F : Licences {#annexe-f}

> **⚠️ ATTENTION** : les prix et l'inclusion de fonctionnalités dans les plans E3 et E5 évoluent. Ce guide ne publie volontairement aucun prix. Vérifiez la composition des licences sur la page officielle Microsoft et auprès du revendeur avant tout engagement.

| Fonctionnalité | Dans Microsoft 365 Business Premium | Remarque |
|----------------|-------------------------------------|----------|
| Intune Plan 1 (MDM, MAM) | Oui | |
| Entra ID P1 (accès conditionnel, MFA, SSPR) | Oui | |
| Defender for Business (EDR pour PME) | Oui | |
| Defender for Office 365 Plan 1 | Oui | |
| Windows LAPS, Quick Assist | Natifs de Windows | |
| Remote Help | À vérifier | Licence complémentaire selon le tenant (voir Partie 7bis) |
| Intune Plan 2 et fonctions Intune Suite (par exemple Endpoint Privilege Management, Cloud PKI) | À vérifier | Add-on ou plan supérieur |
| Entra ID P2 (PIM, Identity Protection), Defender for Cloud Apps | À vérifier | Hors périmètre MVC |
| Limite d'utilisateurs | 300 utilisateurs maximum | |

---

[🏠 Retour au portail principal](https://doit4everyone.github.io/)

---

ℹ️ *Références, structuration et aide à la rédaction assistées par IA, avec validation humaine finale.*
