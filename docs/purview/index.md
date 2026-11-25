---
title: "Guide MVC Microsoft Purview (nLPD Suisse) | DoIt4Everyone"
description: "Guide MVC Microsoft Purview pour PME suisses, classification, étiquetage de sensibilité, DLP Exchange, rétention automatique, break-glass RMS. Conformité nLPD en 4 à 6 heures."
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

> [← Retour au bundle MVC nLPD](../index.html)

<h1>Guide MVC : Microsoft Purview</h1>
<h2>Minimum Viable de Conformité nLPD pour PME suisses 🇨🇭</h2>

Ce guide couvre le déploiement et l'exploitation du socle Purview dans le périmètre **Business Premium + Purview Suite**. Il se compose de deux documents complémentaires.

---

## 📄 Documents de ce guide

| Document | Destinataire | Objet |
|----------|-------------|-------|
| [Procédure de déploiement](procedure-deploiement.html) | Consultant / MSP | Mise en place complète en 4 à 6 heures |
| [Guide d'exploitation](guide-exploitation.html) | Responsable IT client | Maintien quotidien et conformité nLPD |

---

## 🧩 Piliers MVC couverts

| Pilier | Ce que ça apporte |
|--------|------------------|
| MFA (tous les comptes) | Première barrière contre la compromission de compte |
| Blocage partage externe SharePoint | Fermeture du vecteur de fuite via liens SharePoint |
| Audit (traçabilité PFPDT) | Journal des actions sur les données : 180 jours par défaut, 1 an pour Entra ID, Exchange, OneDrive et SharePoint avec Audit Premium |
| 3 étiquettes (Interne, Confidentiel, Confidentiel-Externe) | Classification + chiffrement AES-256 des données sensibles, accès interne seul pour les données confidentielles |
| Auto-labelling service | Détection et étiquetage automatique des fichiers SharePoint existants |
| DLP Exchange | Envoi externe de données AVS ou médicales soumis à justification, simple avertissement pour l'IBAN |
| Rétention automatique | Pièces comptables conservées 10 ans (CO art. 958f), dossiers RH protégés 10 ans contre la suppression (CO art. 127, 128 ch. 3 et 328b) |
| Break-glass RMS | Accès de secours aux fichiers chiffrés en cas d'urgence |

---

## 🇨🇭 Conformité nLPD

| Article | Exigence | Mesure MVC incluse |
|---------|----------|--------------------|
| Art. 8 | Mesures techniques et organisationnelles | Chiffrement RMS + DLP + audit + rétention + blocage SharePoint |
| Art. 12 | Registre des activités de traitement | Modèle pré-rempli en Annexe H |
| Art. 24 | Traçabilité et notification des violations | Journaux exportables, incidents DLP horodatés, révocation RMS, fiche réflexe rançongiciel (Annexe G) |

---

## 📥 Téléchargements

- [Procédure de déploiement, Word (.docx)]({{ '/downloads/docx/MVC-nLPD_01-Purview_Procedure_v1.3.docx' | relative_url }})
- [Guide d'exploitation client, Word (.docx)]({{ '/downloads/docx/MVC-nLPD_01-Purview_Exploitation_v1.3.docx' | relative_url }})
- [Procédure de déploiement, PDF]({{ '/downloads/pdf/MVC-nLPD_01-Purview_Procedure_v1.3.pdf' | relative_url }})
- [Guide d'exploitation client, PDF]({{ '/downloads/pdf/MVC-nLPD_01-Purview_Exploitation_v1.3.pdf' | relative_url }})

---

## 🗂️ Adaptations sectorielles incluses

Fiduciaires et comptabilités · Cabinets d'architectes et ingénieurs civils · Études d'avocats et notaires · Agences de communication et marketing · Cabinets médicaux et paramédicaux

Voir [Procédure de déploiement, Annexe E](procedure-deploiement.html#annexe-e--adaptations-sectorielles).

---

## ❌ Hors périmètre MVC

IRM (Insider Risk Management), Endpoint DLP, eDiscovery Premium, Communication Compliance, DSPM for AI, Power Platform Managed Environments, gouvernance Copilot. Ces fonctionnalités sont couvertes dans le [guide Copilot Governance]({{ '/copilot-governance/' | relative_url }}).

---

[🏠 Retour au portail principal](https://doit4everyone.github.io/)

---

ℹ️ *Références, structuration et aide à la rédaction assistées par IA, avec validation humaine finale.*
