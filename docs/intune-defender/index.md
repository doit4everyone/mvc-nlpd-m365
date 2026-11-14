---
title: "Intune + Defender for Business : Appareils et menaces, MVC nLPD"
description: "Guide Intune + Defender for Business pour PME suisses de 5 à 25 utilisateurs sur Microsoft 365 Business Premium : inscription, chiffrement, EDR, antivirus, ASR, LAPS, conformité, Defender for Office 365 Plan 1. Mesures techniques nLPD art. 8."
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

<h1>Intune + Defender for Business : Appareils et menaces</h1>
<h2>Minimum Viable de Conformité · nLPD RS 235.1 · v3.8</h2>

> [← Retour au bundle MVC nLPD](../index.html)

> **ℹ️ INFO : statut. Procédure publiée, validation terrain en cours.** Chemins et libellés relevés sur le portail. Les valeurs marquées « à confirmer » et les tests sur appareils réels restent à valider lors de la validation terrain.

Ce guide déploie la couche **appareils et menaces** du bundle : inscription et chiffrement des appareils, protection des postes, conformité, protection de la messagerie. La nLPD art. 8 exige des mesures techniques adaptées au risque : un poste non géré ou non chiffré est un point faible que ni les identités ni les données ne compensent seules.

| | |
|---|---|
| **Licence** | Microsoft 365 Business Premium (Intune Plan 1, Defender for Business, Defender for Office 365 Plan 1 inclus) |
| **Portails** | `intune.microsoft.com` et `security.microsoft.com` |
| **Durée** | 4 à 5 heures de travail technique (hors périodes d'observation) |
| **Prérequis** | Étape 0 du guide Entra ID P1 (Parties 0 à 3) et CA-02 déployées |
| **Version** | 3.8 |

---

## 📘 Contenu du guide

| Document | Contenu |
|----------|---------|
| [Procédure de déploiement](procedure-deploiement.html) | Parties 0 à 13 : prérequis et groupes par plateforme, inscription, chiffrement, Endpoint Security, filtrage web, AIR, vulnérabilités, prise en main à distance, conformité, profils, Defender for Office 365, alertes, intégration CA-04 et CA-08, tests, checklist de remise client |
| [Annexes](annexes.html) | Glossaire, checklist complète, références nLPD, adaptations sectorielles, différences POC et production, licences |

---

## 🧱 Ce que le guide met en place

| Pilier | Ce que ça apporte |
|--------|-------------------|
| **Groupes par plateforme** | `GRP-Appareils-Windows`, `GRP-Appareils-iOS`, `GRP-Appareils-Android` |
| **Connecteur Intune → Defender** | Intégration EDR et signaux de risque |
| **BitLocker et Device Control** | Données au repos et exfiltration physique `[nLPD art. 8]` |
| **EDR, antivirus, Tamper Protection, ASR** | Protection du poste `[nLPD art. 8]` |
| **Windows LAPS** | Mot de passe administrateur local unique `[nLPD art. 8]` |
| **SmartScreen, Network Protection, filtrage web** | Protection de la navigation `[nLPD art. 8]` |
| **AIR et Centre d'actions** | Contribution à la détection et à la remédiation `[nLPD art. 8]` |
| **Conformité Intune et CA-08** | Accès réservé aux appareils conformes `[nLPD art. 8]` |
| **Defender for Office 365 Plan 1** | Protection de la messagerie `[nLPD art. 8]` |

> **ℹ️ INFO** : hors périmètre MVC : Defender for Endpoint Plan 2, Defender for Cloud Apps, Sentinel. L'art. 24 (annonce des violations) reste une procédure organisationnelle du client : les journaux et alertes de ce guide la facilitent.

---

## 🔗 Liens avec les autres guides

- **Entra ID P1** : [guide Entra ID P1](../entra-id-p1/index.html). CA-04 et CA-08 y sont créées en mode Rapport, puis passées en Application dans la Partie 12 de ce guide.
- **Purview** : [guide Purview](../purview/index.html), classification, étiquetage et protection des données.

---

## ⬇️ Téléchargements

| Format | Fichier |
|--------|---------|
| Word (.docx) | [MVC-nLPD_03-Intune-Defender_Procedure_v3.8.docx]({{ '/downloads/docx/MVC-nLPD_03-Intune-Defender_Procedure_v3.8.docx' | relative_url }}) |
| PDF | [MVC-nLPD_03-Intune-Defender_Procedure_v3.8.pdf]({{ '/downloads/pdf/MVC-nLPD_03-Intune-Defender_Procedure_v3.8.pdf' | relative_url }}) |

---

[🏠 Retour au portail principal](https://doit4everyone.github.io/)

---

ℹ️ *Références, structuration et aide à la rédaction assistées par IA, avec validation humaine finale.*
