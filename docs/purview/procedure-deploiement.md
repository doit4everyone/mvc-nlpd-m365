---
title: "Procédure de déploiement Purview MVC (nLPD Suisse) | DoIt4Everyone"
description: "Procédure technique complète de déploiement Microsoft Purview MVC pour PME suisses. Configuration nLPD en 4 à 6 heures : SITs, étiquettes, DLP, rétention, break-glass RMS. Adaptations sectorielles."
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

> [← Retour au guide Purview](../)

<h1>Procédure de déploiement MVC — Microsoft Purview</h1>
<h2>Guide consultant · Version 1.2 · Durée : 4 à 6 heures</h2>

| Champ | Valeur |
|-------|--------|
| Licence | Microsoft 365 Business Premium + Purview Suite |
| Tenant | `[tenant].onmicrosoft.com` — à remplacer |
| Client | `[Nom du client]` — à remplacer |
| Version | 1.2 — 2026 |

---

## Sommaire

- [Préface](#préface)
- [Partie 0 : Prérequis](#partie-0--prérequis)
  - [0.1 Accéder au portail Purview](#01-accéder-au-portail-purview)
  - [0.2 PowerShell en administrateur](#02-powershell-en-administrateur)
  - [0.3 Attribuer les rôles](#03-attribuer-les-rôles)
  - [0.4 Vérifier la licence Purview Suite](#04-vérifier-la-licence-purview-suite)
  - [0.5 Activer EnableMIPLabels](#05-activer-enablemiplabels-une-seule-fois-par-tenant)
  - [0.6 Activer le MFA sur tous les comptes](#06-activer-le-mfa-sur-tous-les-comptes)
  - [0.7 Bloquer le partage externe SharePoint](#07-bloquer-le-partage-externe-au-niveau-du-tenant-sharepoint)
- [Partie 1 : Activer l'audit](#partie-1--activer-laudit)
- [Partie 2 : Créer les 2 étiquettes de sensibilité](#partie-2--créer-les-2-étiquettes-de-sensibilité)
  - [2.1 Créer les SITs personnalisés](#21-créer-les-sits-personnalisés)
  - [2.2 Créer les 2 étiquettes](#22-créer-les-2-étiquettes)
  - [2.3 Publier les étiquettes](#23-publier-les-étiquettes)
  - [2.4 Auto-labelling côté service](#24-auto-labelling-côté-service-fichiers-sharepoint-existants)
  - [2.5 Break-glass RMS](#25-break-glass-rms-accès-de-secours-aux-fichiers-chiffrés)
- [Partie 3 : DLP Exchange](#partie-3--dlp-exchange)
- [Partie 4 : Rétention automatique des données](#partie-4--rétention-automatique-des-données)
- [Partie 5 : Vérification et remise](#partie-5--vérification-et-remise)
- [Annexe A : Glossaire simplifié](#annexe-a--glossaire-simplifié)
- [Annexe B : Checklist MVC complète](#annexe-b--checklist-mvc-complète)
- [Annexe C : Mots-clés pour les SIT personnalisés](#annexe-c--mots-clés-pour-les-sit-personnalisés)
- [Annexe E : Différences POC vs Production](#annexe-e--différences-poc-vs-production)
- [Annexe F : Adaptations sectorielles](#annexe-f--adaptations-sectorielles)
- [Annexe I : Registre des activités de traitement](#annexe-i--registre-des-activités-de-traitement-art-12-nlpd)

---

## Préface

> **⚠️ ATTENTION — Personnalisation obligatoire**
> Remplacez `[domaine]`, `[tenant]` et `[tenant].sharepoint.com` par les informations réelles du client. L'Annexe I est pré-remplie pour Axonix SA (POC demo.ch) ; adaptez-la au client avant remise.

Ce guide est conçu pour un consultant qui installe une protection Minimum Viable de Conformité (MVC) dans une PME suisse de 5 à 25 personnes. Objectif : une session de 4 à 6 heures, client autonome à la fin.

**Qu'est-ce que le MVC ?** Le Minimum Viable de Conformité est l'ensemble minimal de mesures techniques défendables devant le Préposé fédéral à la protection des données (PFPDT). Il ne vise pas la perfection technique mais la capacité de démontrer une démarche proportionnée et documentée. Pour une PME de 5 à 25 personnes sans IT dédié, c'est le point de départ réaliste et maintenable.

**Piliers MVC couverts :**

| Pilier | Ce que ça apporte |
|--------|------------------|
| MFA (tous les comptes) | Première barrière contre la compromission de compte |
| Blocage partage externe SharePoint | Fermeture du vecteur de fuite via liens SharePoint |
| Audit (traçabilité PFPDT) | Journal de toutes les actions sur les données, 1 an |
| 2 étiquettes (Interne + Confidentiel) | Classification + chiffrement AES-256 des données sensibles |
| Auto-labelling service | Détection et étiquetage automatique des fichiers SharePoint existants |
| DLP Exchange | Détection et traçabilité des envois de données sensibles |
| Rétention automatique | Conservation 10 ans des données RH et Finances (CO art. 958f) |
| Break-glass RMS | Accès de secours aux fichiers chiffrés en cas d'urgence |

> **ℹ️ INFO — Hors périmètre MVC**
> IRM, Endpoint DLP, Managed Environments, eDiscovery, Communication Compliance, DSPM for AI. Ces fonctionnalités peuvent être déployées progressivement après le socle MVC.

---

## Partie 0 : Prérequis

### 0.1 Accéder au portail Purview

| N° | Étape | Action |
|----|-------|--------|
| 1 | Navigateur | Microsoft Edge ou Google Chrome. Évitez Firefox. |
| 2 | URL | `https://purview.microsoft.com` (ou `https://compliance.microsoft.com`, redirige automatiquement). |
| 3 | Connexion | Compte admin M365 (Global Admin ou Compliance Admin). |
| ⚠️ | Accès refusé | « Vous n'avez pas accès » → problème de rôles, allez à la section 0.3. |

### 0.2 PowerShell en administrateur

| N° | Étape | Action |
|----|-------|--------|
| 1 | Ouvrir | Démarrer → Windows PowerShell → clic droit → Exécuter en tant qu'administrateur. Le titre affiche « Administrateur ». |
| 2 | Autoriser les scripts | `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser` → O → Entrée. |

### 0.3 Attribuer les rôles

**Entra ID** (`entra.microsoft.com` → Rôles et administrateurs)

| Rôle Entra ID | Pourquoi |
|---------------|----------|
| Administrateur de la sécurité | Signaux de risque dans Purview |
| Administrateur de mise en conformité | Politiques Purview |

→ Rôles et administrateurs → cherchez le rôle → + Ajouter des attributions → votre compte.

**Exchange Online** (`admin.exchange.microsoft.com` → Rôles → Organization Management)

\+ Ajouter des membres → votre compte. Attendez 5 à 10 minutes avant de continuer.

> **🚨 IMPORTANT**
> Sans Organization Management, la commande PowerShell d'audit s'exécute sans erreur mais ne fait rien.

**Purview RBAC** (Paramètres → Rôles et étendues → Groupes de rôles)

| Groupe Purview | Accès |
|----------------|-------|
| Compliance Administrator | Labels, DLP, rétention, politiques |
| Content Explorer Content Viewer | Voir le contenu des fichiers |
| Content Explorer List Viewer | Voir la liste des fichiers |

### 0.4 Vérifier la licence Purview Suite

`admin.microsoft.com` → Utilisateurs actifs → votre compte → Licences et applications → vérifiez que **Microsoft Purview Suite** est coché. Si absent : cochez et Enregistrer les modifications.

### 0.5 Activer EnableMIPLabels (une seule fois par tenant)

Requis pour que les étiquettes de type Groupes et sites fonctionnent.

```powershell
Install-Module Microsoft.Graph -Scope CurrentUser
Connect-MgGraph -Scopes "Directory.ReadWrite.All"
$t = Get-MgBetaDirectorySettingTemplate -All | Where-Object { $_.DisplayName -eq "Group.Unified" }
New-MgBetaDirectorySetting -BodyParameter @{ templateId = $t.Id; values = @(@{ name = "EnableMIPLabels"; value = "True" }) }
```

→ Déconnectez-vous et reconnectez-vous au portail Purview avant de continuer.

### 0.6 Activer le MFA sur tous les comptes

> **🚨 IMPORTANT — Priorité absolue**
> Sans MFA, Purview est un château de cartes. Un mot de passe volé donne accès à toutes les données sensibles.

| N° | Étape | Action |
|----|-------|--------|
| 1 | Accès | `admin.microsoft.com` → Utilisateurs actifs → Authentification multifacteurs (bouton en haut de la liste). |
| 2 | Activer | Sélectionnez tous les utilisateurs → Activer → Confirmer. |
| 3 | Alternative recommandée | `entra.microsoft.com` → Protection → Accès conditionnel → stratégie MFA pour Tous les utilisateurs (permet des exclusions ciblées). |

### 0.7 Bloquer le partage externe au niveau du tenant SharePoint

Cette étape ferme le principal vecteur de fuite non surveillé : les liens SharePoint vers l'extérieur.

> **⚠️ ATTENTION**
> Vérifiez au préalable qu'aucun site SharePoint actif ne dépend du partage externe. `admin.sharepoint.com` → Sites actifs → colonne Partage externe : identifiez les sites Activé et validez avec le client avant de bloquer.

| N° | Étape | Action |
|----|-------|--------|
| 1 | Accès | `https://admin.sharepoint.com` → Stratégies → Partage. |
| 2 | Partage externe | Curseur SharePoint → **Uniquement les personnes de votre organisation**. Faites de même pour OneDrive. |
| 3 | Enregistrer | Cliquez sur Enregistrer. |

> **ℹ️ INFO**
> Ce paramètre est un plafond global : aucun site ne peut être partagé en externe même si un administrateur essaie de l'activer site par site. Les envois de pièces jointes par email ne sont pas affectés et restent surveillés par la DLP Exchange (Partie 3).

---

## Partie 1 : Activer l'audit

> **🚨 IMPORTANT — À faire AVANT les étiquettes**
> Sans audit actif, vous n'avez aucune trace en cas d'incident nLPD.

```powershell
Install-Module -Name ExchangeOnlineManagement -Force
Connect-ExchangeOnline
Set-AdminAuditLogConfig -UnifiedAuditLogIngestionEnabled $true
Get-AdminAuditLogConfig | FL UnifiedAuditLogIngestionEnabled
```

→ Résultat attendu : `UnifiedAuditLogIngestionEnabled : True`

```powershell
Disconnect-ExchangeOnline -Confirm:$false
```

> **✅ CONSEIL**
> Vérification dans Purview : Purview → Audit (menu gauche) → Rechercher → bandeau vert = audit actif. Si bouton **Démarrer l'enregistrement** visible → cliquez dessus.

---

## Partie 2 : Créer les 2 étiquettes de sensibilité

| Ordre | Étiquette | Rôle |
|-------|-----------|------|
| 1 | 1 - Interne | Sans chiffrement. Étiquette par défaut pour tous les documents internes. |
| 2 | 2 - Confidentiel | Avec chiffrement AES-256. Pour AVS, salaires, médical, contrats, données bancaires. |

### 2.1 Créer les SITs personnalisés

> **ℹ️ INFO**
> SIT = Types d'informations sensibles dans l'interface Purview. Propagation : attendez 15 à 30 minutes après création avant de configurer les étiquettes.

#### SIT 1 : AVS-Suisse-PME

Détecte les numéros AVS suisses au format exact `756.XXXX.XXXX.XX` en 4 langues nationales + anglais.

| N° | Étape | Action |
|----|-------|--------|
| 1 | Navigation | Purview → Protection des données → Classifieurs → Types d'informations sensibles → + Créer un type d'informations sensibles |
| 2 | Page 1 : Nommer | Nom : `AVS-Suisse-PME` \| Description : Détection numéro AVS suisse 756.XXXX.XXXX.XX, 4 langues nationales + anglais. → Suivant |
| 3 | Page 2 : Modèle | + Ajouter un modèle → Niveau de confiance : **Moyen** → + Ajouter un élément principal → Expression régulière → ID : `regex-AVS-Suisse` → collez : `756\\.\\d{4}\\.\\d{4}\\.\\d{2}` → Correspondance de chaîne → Terminé |
| 4 | Page 2 : Mots-clés | + Ajouter des éléments de soutien → Liste de mots-clés → ID : `recherche-AVS-Suisse` → saisissez un mot-clé par ligne (voir Annexe C) → Terminé |
| 5 | Page 3 : Tester | Suivant → testez avec `756.1234.5678.94` → vérifiez la détection → Suivant → Créer |
| ⚠️ | Propagation | Attendez 15 à 30 minutes avant de configurer les règles qui utilisent ce SIT. |

> **ℹ️ INFO**
> `756.1234.5678.94` est un numéro de test avec checksum EAN-13 valide. Utilisez-le pour tester AVS-Suisse-PME et le SIT natif *Swiss Social Security Number AHV* simultanément.

#### SIT 2 : Medical-RH-Suisse

Détecte les données médicales RH (arrêts maladie, certificats, incapacité de travail) en 4 langues nationales + anglais.

| N° | Étape | Action |
|----|-------|--------|
| 1 | Navigation | Purview → Protection des données → Classifieurs → Types d'informations sensibles → + Créer un type d'informations sensibles |
| 2 | Page 1 : Nommer | Nom : `Medical-RH-Suisse` \| Description : Détection données médicales RH, 4 langues nationales + anglais. → Suivant |
| 3 | Page 2 : Modèle | + Ajouter un modèle → Niveau de confiance : **Moyen** → + Ajouter un élément principal → Liste de mots-clés → ID : `medical-rh-suisse` → saisissez un mot-clé par ligne (voir Annexe C) → Terminé |
| 4 | Page 3 : Tester | Suivant → testez avec « arrêt maladie » → vérifiez la détection → Suivant → Créer. Attendez 15 à 30 minutes avant de continuer. |

### 2.2 Créer les 2 étiquettes

#### Étiquette 1 : 1 - Interne (sans chiffrement, par défaut)

| N° | Étape | Action |
|----|-------|--------|
| 1 | Navigation | Purview → Protection des données → Étiquettes de confidentialité → + Créer une étiquette |
| 2 | Page 1 : Nommer | Nom : `1 - Interne` \| Description : Document à usage interne uniquement. \| Couleur : Bleu → Suivant |
| 3 | Page 2 : Étendue | Cochez : Fichiers et autres ressources de données, Emails, Réunions → Suivant |
| 4 | Page 3 : Protection | Sélectionnez **Aucune protection** → Suivant |
| 5 | Finaliser | Laissez les pages suivantes par défaut → Créer une étiquette → Ne créez pas encore de stratégie → Terminer. |

#### Étiquette 2 : 2 - Confidentiel (avec chiffrement AES-256)

> **ℹ️ INFO — Groupes mail-enabled requis pour le chiffrement RMS**
> Azure Rights Management exige que les groupes assignés aient une adresse email. En tenant cloud pur (sans AD on-prem) :
> ```powershell
> Connect-ExchangeOnline
> New-DistributionGroup -Name "GRP-Confidentiel-Purview" -Alias "grp-confidentiel-purview" -Type Security -PrimarySmtpAddress "grp-confidentiel-purview@[domaine]"
> Add-DistributionGroupMember -Identity "GRP-Confidentiel-Purview" -Member utilisateur@[domaine]
> ```
> En environnement hybride (AD on-prem) : créez le groupe dans AD et attendez la synchronisation Entra Connect.

| N° | Étape | Action |
|----|-------|--------|
| 1 | Navigation | Purview → Protection des données → Étiquettes de confidentialité → + Créer une étiquette |
| 2 | Page 1 : Nommer | Nom : `2 - Confidentiel` \| Description : Données personnelles sensibles (AVS, salaires, médical, contrats, IBAN). \| Couleur : Bordeaux → Suivant |
| 3 | Page 2 : Étendue | Cochez : Fichiers et autres ressources de données, Emails, Réunions → Suivant |
| 4 | Page 3 : Protection | Sélectionnez **Contrôler l'accès** + Appliquer le marquage de contenu → Suivant |
| 5 | Contrôle d'accès | Configurer les paramètres \| Attribuer maintenant \| Expiration : Jamais \| Hors connexion : 30 jours \| + Attribuer des autorisations → ajoutez (1) `GRP-Confidentiel-Purview`, (2) `admin@[domaine]`, (3) Tous les utilisateurs authentifiés \| Niveau : **Éditeur (Co-auteur)** → Enregistrer |
| 6 | Marquage | Filigrane : CONFIDENTIEL (40, diagonal, rouge) \| En-tête : CONFIDENTIEL — Usage interne uniquement \| Pied : Document protégé nLPD, Ne pas diffuser → Suivant |
| 7 | Auto-labelling client | Activez le toggle → + Ajouter une condition → Le contenu contient \| Groupe : `Donnees-Sensibles` \| Opérateur : OU \| Ajoutez : AVS-Suisse-PME (min 1, Moyen) + IBAN (min 1, Élevé) + Medical-RH-Suisse (min 2, Moyen) → Ajouter → Suivant |
| 8 | Finaliser | Pages suivantes par défaut → Créer une étiquette → Ne créez pas encore de stratégie → Terminer. |
| ⚠️ | Propagation | Attendez 15 à 30 minutes avant de configurer la DLP et l'auto-labelling service. |

> **⚠️ ATTENTION — Deux mécanismes d'auto-labelling distincts**
> L'étape 7 configure l'auto-labelling **côté client** (ouverture dans Word ou Outlook). Les fichiers déjà présents dans SharePoint ne sont pas couverts. La section 2.4 configure l'auto-labelling **côté service**. Les deux mécanismes sont complémentaires.

> **ℹ️ INFO — Accès des destinataires externes**
> Destinataire avec compte Microsoft → déchiffrement automatique. Destinataire sans compte Microsoft (Gmail, AFC, caisse AVS) → code à usage unique par email, valable 15 minutes, saisi dans le navigateur. Informez le client de cette friction potentielle avant la mise en production.

### 2.3 Publier les étiquettes

| N° | Étape | Action |
|----|-------|--------|
| 1 | Navigation | Purview → Protection des données → Stratégies → Stratégies de publication d'étiquettes → + Publier l'étiquette |
| 2 | Étiquettes | Sélectionnez : `1 - Interne`, `2 - Confidentiel` → Ajouter → Suivant |
| 3 | Unités admin | Laissez Répertoire complet → Suivant |
| 4 | Utilisateurs | Laissez Tous utilisateurs et groupes → Suivant |
| 5 | Paramètres | Cochez : (1) Les utilisateurs doivent fournir une justification pour abaisser le niveau \| (2) Obliger les utilisateurs à appliquer une étiquette à leurs courriers et documents → Suivant |
| 6 | Défauts | Étiquette par défaut documents et e-mails : `1 - Interne`. Laissez le reste par défaut → Suivant |
| 7 | Nom | Nom : `Politique-Labels-[client]` → Suivant |
| 8 | Soumettre | Vérifiez le résumé puis cliquez sur Envoyer. Propagation : jusqu'à 24 heures. |

### 2.4 Auto-labelling côté service (fichiers SharePoint existants)

| N° | Étape | Action |
|----|-------|--------|
| 1 | Navigation | Purview → Protection des données → Stratégies → Stratégies d'étiquetage automatique → + Créer une stratégie d'étiquetage automatique |
| 2 | Catégorie | Faites défiler la liste → sélectionnez **Personnaliser** → Suivant |
| 3 | Nommer | Nom : `Auto-Label-Confidentiel` \| Description : Détection automatique AVS, IBAN, médical dans SharePoint et OneDrive. → Suivant |
| 4 | Étiquette | Cliquez sur choisir une étiquette → sélectionnez `2 - Confidentiel` → Appliquer → Suivant |
| 5 | Emplacements | Activez : Sites SharePoint (Tous les sites), Comptes OneDrive (Tous les utilisateurs), Courrier Exchange (Tous les utilisateurs) → Suivant |
| 6 | Règles | Sélectionnez Règles communes → Suivant → + Nouvelle règle |
| 7 | Règle AVS | Nom : `Règle-Donnees-Sensibles` \| Le contenu contient → + Ajouter → AVS-Suisse-PME (Moyen, min 1) + IBAN (Élevé, min 1) + Medical-RH-Suisse (Moyen, min 2) → Enregistrer |
| 8 | Mode | Sélectionnez **Exécuter la stratégie en mode de simulation** → Suivant → Créer une stratégie |

> **ℹ️ INFO**
> Après 24 à 48 heures de simulation : Purview → Stratégies d'étiquetage automatique → cliquez sur `Auto-Label-Confidentiel` → Afficher les résultats de simulation. Si résultat correct → cliquez sur **Activer la stratégie**.

### 2.5 Break-glass RMS (accès de secours aux fichiers chiffrés)

> **🚨 IMPORTANT — Obligatoire avant mise en production**
> Sans compte Break-glass configuré, un départ conflictuel ou une corruption du tenant peut entraîner une perte irréversible des données chiffrées.

**Étape 1 — Installer le module et se connecter (Windows PowerShell 5.1 uniquement) :**

```powershell
Install-Module -Name AIPService -Force
Connect-AipService
```

**Étape 2 — Activer la fonctionnalité Super User et désigner le compte de secours :**

```powershell
Enable-AipServiceSuperUserFeature
Add-AipServiceSuperUser -EmailAddress admin@[domaine]
```

**Étape 3 — Vérifier et désactiver après configuration :**

```powershell
Get-AipServiceSuperUser
Disable-AipServiceSuperUserFeature
```

> **⚠️ ATTENTION**
> Réactivez le Super User uniquement en cas d'urgence réelle. Chaque activation est tracée dans les journaux d'audit Purview. Stockez les identifiants du compte de secours dans un coffre-fort physique.

---

## Partie 3 : DLP Exchange

> **ℹ️ INFO**
> Dans ce MVC, l'email est le seul canal légitime d'envoi de données vers l'extérieur (SharePoint est bloqué). Les deux règles partent en simulation 2 à 4 semaines avant activation.

> **⚠️ ATTENTION — Prérequis**
> Les SIT (section 2.1) doivent être créés et propagés (30 minutes) avant de créer cette politique.

### 3.1 Créer la politique DLP-PME-Surveillance

| N° | Étape | Action |
|----|-------|--------|
| 1 | Navigation | Purview → Protection contre la perte de données → Stratégies → + Créer une stratégie → Personnalisé → Personnalisé → Suivant |
| 2 | Nommer | Nom : `DLP-PME-Surveillance` → Suivant |
| 3 | Emplacements | Activez uniquement : **Courrier Exchange** (Tous les utilisateurs). Désactivez SharePoint, OneDrive, Teams. → Suivant |
| 4 | Paramètres | Sélectionnez Créer ou personnaliser des règles DLP avancées → Suivant |
| 5 | Règles | Cliquez + Créer une règle. Créez les 2 règles détaillées ci-dessous. |
| 6 | Mode | Page Mode : **Exécuter en mode simulation** → Suivant |
| 7 | Soumettre | Vérifiez et cliquez Soumettre. |

### 3.2 Règle 1 : Avertir-Donnees-Sensibles

| Paramètre | Configuration |
|-----------|---------------|
| Nom de la règle | `Avertir-Donnees-Sensibles` |
| Condition 1 | Le contenu contient → OU : AVS-Suisse-PME (Moyen, min 1) + IBAN (Élevé, min 1) + Medical-RH-Suisse (Moyen, min 2) |
| Condition 2 | Du contenu est partagé avec des personnes extérieures à mon organisation |
| Actions | Aucune (avertissement seulement) |
| Notifications utilisateur | Activez les conseils de stratégie → message : Ce document contient des données sensibles. Vérifiez que le destinataire est autorisé. |
| Remplacements | Activez + exigez justification. |
| Rapport d'incident | Niveau Moyen. Alerte email : `admin@[domaine]`. |

→ Cliquez sur Enregistrer. Puis + Créer une règle pour la règle suivante.

### 3.3 Règle 2 : Avertir-IBAN-Externe

| Paramètre | Configuration |
|-----------|---------------|
| Nom de la règle | `Avertir-IBAN-Externe` |
| Condition 1 | Le contenu contient → IBAN (confiance Élevée, min 1) |
| Condition 2 | Du contenu est partagé avec des personnes extérieures à mon organisation |
| Actions | Aucune (avertissement seulement en phase simulation) |
| Notifications utilisateur | Activez les conseils de stratégie → message : Ce document contient un IBAN. Toute transmission externe est enregistrée conformément à la nLPD. Assurez-vous d'avoir l'accord du responsable. |
| Remplacements | Activez + exigez justification → la justification est enregistrée dans l'Activity Explorer. |
| Rapport d'incident | Niveau Élevé. Alerte email : `admin@[domaine]`. |

→ Cliquez sur Enregistrer.

> **ℹ️ INFO**
> Après 2 à 4 semaines : consultez l'Activity Explorer → si les faux positifs sont acceptables → modifiez la politique → Mode → **Activer immédiatement**.

---

## Partie 4 : Rétention automatique des données

> **🚨 IMPORTANT**
> Sans politique de rétention configurée, le registre des activités de traitement (Annexe I) indique « 10 ans (CO art. 958f) » sans mesure technique correspondante. C'est une incohérence documentaire immédiatement visible en audit PFPDT.

### 4.1 Retention-RH-10ans

| N° | Étape | Action |
|----|-------|--------|
| 1 | Navigation | Purview → Gestion du cycle de vie des données → Stratégies de rétention → + Nouvelle stratégie de rétention |
| 2 | Nommer | Nom : `Retention-RH-10ans` \| Description : Rétention documents RH, 10 ans CO art. 958f. → Suivant |
| 3 | Unités admin | Répertoire complet → Suivant |
| 4 | Type | Sélectionnez **Statique** → Suivant |
| 5 | Emplacements | Activez uniquement Sites SharePoint → Modifier → saisissez `https://[tenant].sharepoint.com/sites/RH` → cliquez + → Terminé. Désactivez tous les autres emplacements. → Suivant |
| 6 | Paramètres | Conserver les éléments pendant une période spécifique → **10 ans** → Démarrer la période fondée sur : Lorsque les éléments ont été modifiés en dernier lieu → À la fin : **Supprimer automatiquement les éléments** → Suivant → Soumettre. |

### 4.2 Retention-Finances-10ans

Même procédure que 4.1 avec les adaptations suivantes :

| Paramètre | Valeur |
|-----------|--------|
| Nom | `Retention-Finances-10ans` |
| Description | Rétention documents Finances, 10 ans CO art. 958f. |
| URL SharePoint | `https://[tenant].sharepoint.com/sites/Finances` |
| Durée et action | 10 ans → Supprimer automatiquement |

---

## Partie 5 : Vérification et remise

### Test 1 : Audit actif

```powershell
Get-AdminAuditLogConfig | FL UnifiedAuditLogIngestionEnabled
```
→ Résultat attendu : `True`. Alternative : Purview → Audit → Rechercher → bandeau vert.

### Test 2 : Étiquettes visibles dans Word

Ouvrez Word sur un PC utilisateur. Ruban → **Sensibilité**. Les 2 étiquettes doivent être présentes. Si absent : propagation jusqu'à 24 heures.

### Test 3 : Détection automatique (auto-labelling client)

Dans Word, tapez `756.1234.5678.94`. Attendez 30 secondes. L'étiquette `2 - Confidentiel` doit s'appliquer automatiquement.

### Test 4 : Avertissement DLP

Envoyez par email à une adresse externe un document contenant `756.1234.5678.94`. Le message d'avertissement doit apparaître dans Outlook avant l'envoi.

### Test 5 : Audit

Purview → Audit → Rechercher. Aujourd'hui. Activités : Fichier accédé. Des entrées doivent correspondre à vos tests.

### Test 6 : Blocage partage SharePoint

Dans SharePoint, tentez de partager un fichier avec une adresse externe. Le partage doit être refusé : *Votre organisation ne permet pas le partage avec des personnes extérieures*.

### Checklist de remise client

| | Élément à valider | Statut |
|-|-------------------|--------|
| ☐ | MFA activé sur tous les comptes | à valider |
| ☐ | Partage externe SharePoint bloqué au niveau tenant | à valider |
| ☐ | Audit actif (True dans PowerShell) | à valider |
| ☐ | SITs AVS-Suisse-PME et Medical-RH-Suisse créés et propagés | à valider |
| ☐ | Étiquettes 1 - Interne et 2 - Confidentiel créées et publiées | à valider |
| ☐ | Auto-labelling service activé en mode simulation (section 2.4) | à valider |
| ☐ | Politiques de rétention RH et Finances créées (section 4) | à valider |
| ☐ | DLP-PME-Surveillance créée en mode simulation (2 règles) | à valider |
| ☐ | Break-glass RMS configuré (section 2.5) | à valider |
| ☐ | Étiquettes visibles dans Word sur un PC utilisateur (Test 2) | à valider |
| ☐ | Détection AVS fonctionnelle avec 756.1234.5678.94 (Test 3) | à valider |
| ☐ | Avertissement DLP déclenché sur envoi externe (Test 4) | à valider |
| ☐ | Partage SharePoint externe refusé (Test 6) | à valider |
| ☐ | Registre des activités (Annexe I) remis et adapté au client | à valider |

---

## Annexe A : Glossaire simplifié

| Terme | Définition |
|-------|------------|
| MVC (Minimum Viable de Conformité) | Ensemble minimal de mesures techniques défendables devant le PFPDT. Ne vise pas la perfection technique mais la capacité de démontrer une démarche proportionnée et documentée. |
| SIT | Détecteur de motifs qui cherche des numéros AVS, IBAN, etc. dans les documents. |
| Étiquette de sensibilité | Marqueur appliqué à un document qui indique son niveau de confidentialité et active les protections associées (chiffrement, filigrane). |
| Auto-labelling client | S'applique quand l'utilisateur ouvre un document dans Word ou Outlook. Configuré lors de la création de l'étiquette (section 2.2). |
| Auto-labelling service | Analyse asynchrone des fichiers déjà présents dans SharePoint et OneDrive par Purview. Configuré via une stratégie dédiée (section 2.4). |
| Chiffrement AES-256 / Azure RMS | Technologie qui verrouille le contenu d'un document. Sans la clé, le fichier est illisible même s'il est volé ou copié sur une clé USB. |
| DLP (Data Loss Prevention) | Politique qui surveille et peut bloquer les envois de données sensibles par email ou partage. |
| Rétention Purview | Politique qui définit la durée de conservation des documents et déclenche leur suppression automatique à l'échéance. |
| Break-glass RMS | Procédure de secours permettant à un compte désigné d'accéder aux fichiers chiffrés en cas d'urgence. |
| Audit log | Journal qui enregistre toutes les actions sur les données. Obligatoire nLPD art. 24. |
| PFPDT | Préposé fédéral à la protection des données. Autorité de contrôle suisse nLPD. |
| nLPD | Nouvelle loi fédérale sur la protection des données (en vigueur depuis septembre 2023). |

---

## Annexe B : Checklist MVC complète

| Réf. | Action à valider | Statut |
|------|------------------|--------|
| [0.3] | Rôles Entra ID (Sécurité + Conformité) attribués | ☐ |
| [0.3] | Organization Management Exchange attribué | ☐ |
| [0.3] | Compliance Administrator Purview attribué | ☐ |
| [0.4] | Licence Purview Suite attribuée au compte admin | ☐ |
| [0.5] | EnableMIPLabels activé (PowerShell) | ☐ |
| [0.6] | MFA activé sur TOUS les comptes (pas seulement admin) | ☐ |
| [0.7] | Partage externe bloqué au niveau tenant SharePoint | ☐ |
| [1] | Audit actif : UnifiedAuditLogIngestionEnabled = True | ☐ |
| [2.1] | SIT AVS-Suisse-PME créé et propagé (30 min) | ☐ |
| [2.1] | SIT Medical-RH-Suisse créé et propagé (30 min) | ☐ |
| [2.2] | Groupe GRP-Confidentiel-Purview créé (mail-enabled) | ☐ |
| [2.2] | Étiquette 1 - Interne créée | ☐ |
| [2.2] | Étiquette 2 - Confidentiel créée (chiffrement + auto-labelling client) | ☐ |
| [2.3] | Les 2 étiquettes publiées (Politique-Labels-[client]) | ☐ |
| [2.4] | Stratégie Auto-Label-Confidentiel créée et simulée | ☐ |
| [2.5] | Break-glass RMS configuré (Super User RMS désigné) | ☐ |
| [3] | DLP-PME-Surveillance créée (2 règles, mode simulation) | ☐ |
| [4] | Retention-RH-10ans créée (SharePoint RH) | ☐ |
| [4] | Retention-Finances-10ans créée (SharePoint Finances) | ☐ |
| [5] | Test 1 : Audit actif vérifié | ☐ |
| [5] | Test 2 : Étiquettes visibles dans Word | ☐ |
| [5] | Test 3 : Détection AVS avec 756.1234.5678.94 fonctionnelle | ☐ |
| [5] | Test 4 : Avertissement DLP sur envoi externe déclenché | ☐ |
| [5] | Test 6 : Partage SharePoint externe refusé | ☐ |
| [I] | Registre des activités adapté et remis au client | ☐ |

---

## Annexe C : Mots-clés pour les SIT personnalisés

> **ℹ️ INFO — Astuce copier-coller**
> Copiez les mots-clés dans un document Word vierge, Ctrl+H : Rechercher « / » → Remplacer par `^p`. Vous obtenez la liste prête à coller dans Purview, un mot-clé par ligne.

### C.1 : AVS-Suisse-PME

Expression régulière : `756\\.\\d{4}\\.\\d{4}\\.\\d{2}` | Niveau de confiance : Moyen | min : 1

| Langue | Mots-clés |
|--------|-----------|
| Français | AVS / numéro AVS / No. AVS / N° AVS / assurance vieillesse / assurance sociale / numéro d'assurance |
| Allemand | AHV / AHV-Nummer / AHV-Nr / Altersvorsorge / Sozialversicherung / Versicherungsausweis / AHV-Ausweis |
| Italien | AVS / numero AVS / No. AVS / assicurazione vecchiaia / assicurazione sociale / numero d'assicurazione |
| Romanche | numer AVS / AVS / assicuranza da vegliadetgna / assicuranza sociala / document d'assicuranza / attest d'assicuranza |
| Anglais | AHV number / AVS number / Swiss social security / social security number CH / Swiss insurance number |

### C.2 : Medical-RH-Suisse

Niveau de confiance : Moyen | min : 2

| Langue | Mots-clés |
|--------|-----------|
| Français | arrêt maladie / certificat médical / incapacité de travail / congé maladie / aptitude au travail / inapte au travail / convalescence / accident de travail / maladie professionnelle |
| Allemand | Krankmeldung / ärztliches Zeugnis / Arbeitsunfähigkeit / Krankheitsfall / Arbeitsunfähigkeitszeugnis / Genesungsurlaub / Berufsunfall / Berufskrankheit / arbeitsunfähig |
| Italien | certificato medico / incapacità lavorativa / congedo malattia / idoneità al lavoro / inidoneità al lavoro / convalescenza / infortunio professionale / malattia professionale |
| Anglais | sick leave / medical certificate / incapacity to work / fit for work / unfit for work / occupational accident / occupational disease / sick note / medical leave |
| Romanche | annunzia da malsogna / certificat medical / incapacità da lavurar / congedi da malsogna / abilita da lavurar / nun abel da lavurar / convalescenza / accident da lavur / malsogna professiunala |

---

## Annexe E : Différences POC vs Production

| Point | En production |
|-------|---------------|
| Compte admin dans le chiffrement | Supprimez `admin@[domaine]` des autorisations de chiffrement. Gérez l'accès via le rôle Super User RMS uniquement. |
| DLP mode simulation | Après 2 à 4 semaines : modifiez la politique → Mode → **Activer immédiatement**. Vérifiez l'Activity Explorer avant. |
| Noms des groupes | Remplacez par les groupes réels du client. Créez-les mail-enabled via Exchange Online. |
| Registre des activités | Complétez l'Annexe I avec les activités réelles du client. Supprimez les références à Axonix SA et demo.ch. |
| Auto-labelling service | Vérifiez les résultats de simulation après 48h avant d'activer. |
| Rétention | Remplacez les URLs SharePoint par les URLs réelles des sites RH et Finances du client. |
| Break-glass RMS | Utilisez un compte cloud-only dédié (non nominatif). Stockez les identifiants dans un coffre-fort physique. |
| MFA | Préférez l'Accès conditionnel Entra pour permettre des exclusions ciblées. |

---

## Annexe F : Adaptations sectorielles

> **ℹ️ INFO**
> Appliquez uniquement les adaptations correspondant au secteur du client.

### F.1 : PME générique (MVC tel quel)

> **✅ CONSEIL** — Aucune adaptation requise. Déployez le MVC tel quel.

### F.2 : Fiduciaire et comptabilité

Points de rupture : chiffrement RMS vers AFC et banques (ces institutions refusent les flux OTP Microsoft), rétention globale SharePoint (viole le principe de minimisation nLPD art. 6).

**Adaptation 1 — Étiquette 3 - Externe-Fiduciaire**

| Paramètre | Configuration |
|-----------|---------------|
| Nom | `3 - Externe-Fiduciaire` |
| Chiffrement RMS | Aucun |
| Marquage visuel | Filigrane : CONFIDENTIEL — USAGE EXTERNE AUTORISÉ \| En-tête : Transmission autorisée nLPD |
| Usage | Bilans, déclarations fiscales, décomptes TVA, documents bancaires |

**Adaptation 2 — Affiner la rétention**

Purview → Stratégies de rétention → modifiez `Retention-RH-10ans` et `Retention-Finances-10ans` → Condition : appliquer uniquement aux éléments portant les étiquettes `2 - Confidentiel` et `3 - Externe-Fiduciaire`.

**Option — SIT IDE/UID**

| Paramètre | Valeur |
|-----------|--------|
| Nom | `IDE-UID-Suisse` |
| Expression régulière | `CHE-\\d{3}\\.\\d{3}\\.\\d{3}` |
| Niveau de confiance | Moyen, min : 1 |
| Mots-clés | IDE / UID / numéro IDE / numéro UID / Unternehmens-Identifikationsnummer / numero IDI |

### F.3 : Cabinet d'architectes et ingénieurs civils

> **⚠️ ATTENTION** — Ne pas appliquer la section 0.7 (blocage tenant SharePoint) pour ce secteur.

| N° | Étape | Action |
|----|-------|--------|
| 1 | Partage externe SharePoint | Curseur SharePoint → **Personnes spécifiques**. Curseur OneDrive → Personnes spécifiques. |
| 2 | Type de lien par défaut | Liens des fichiers et des dossiers → sélectionnez **Personnes spécifiques**. Permission par défaut : Consultation. |
| 3 | Paramètres supplémentaires | Paramètres de partage externe supplémentaires → cochez L'accès invité expirera automatiquement → **30 jours**. |
| 4 | Enregistrer | Cliquez sur Enregistrer. |
| 5 | Restreindre par site | Pour les sites RH et Finances : `admin.sharepoint.com` → Sites actifs → site → Paramètres → Partage → **Uniquement les personnes de votre organisation**. |

### F.4 : Étude d'avocats et notaires

**Adaptation — Étiquette 3 - Juridique-Externe (sans chiffrement RMS)**

| Paramètre | Configuration |
|-----------|---------------|
| Nom | `3 - Juridique-Externe` |
| Chiffrement RMS | Aucun |
| Marquage visuel | Filigrane : CONFIDENTIEL — TRANSMISSION AUTORISÉE \| En-tête : Document juridique — Destinataire unique |
| Usage | Pièces de procédure, actes notariés, correspondance inter-cabinets, transmissions aux greffes |

> **✅ CONSEIL** — Pour les transmissions via Justitia.swiss : utilisez le portail directement. Réservez l'étiquette `3 - Juridique-Externe` aux transmissions par email.

### F.5 : Agence de communication et marketing

**Adaptation 1 — Exclure les sites créatifs de l'auto-labelling service**

Dans `Auto-Label-Confidentiel` → Emplacements → Sites SharePoint → saisissez uniquement les URLs des sites RH et Finances. Excluez les sites créatifs (Projets, BAT, Assets).

**Adaptation 2 — Exclure les comptes créatifs de la DLP Exchange**

Dans `DLP-PME-Surveillance` → Emplacements → Courrier Exchange → Exclure → ajoutez les comptes des créatifs et chargés de projet.

> **⚠️ ATTENTION** — Compensez par une charte informatique signée interdisant l'envoi de données personnelles depuis les comptes créatifs. Documentez ce choix dans le registre des activités (Annexe I).

### F.6 : Cabinet médical et paramédical

**Adaptation — Étiquette 3 - Patient-Externe (sans chiffrement RMS)**

| Paramètre | Configuration |
|-----------|---------------|
| Nom | `3 - Patient-Externe` |
| Chiffrement RMS | Aucun |
| Marquage visuel | En-tête : Document médical personnel — Confidentiel \| Pied : Ne pas transmettre à des tiers |
| Usage | Résultats d'analyses, ordonnances, rappels de consultation, factures patients |

> **✅ CONSEIL** — Flux HIN : pour les échanges interprofessionnels (médecin → hôpital), utilisez HIN Mail directement. Le MVC Purview couvre les données internes et les communications patients uniquement.

### F.7 : Synthèse des adaptations

| Secteur | Blocage SharePoint | RMS externe | Auto-labelling | DLP | Complexité |
|---------|-------------------|-------------|----------------|-----|------------|
| PME générique | ✅ Bloquer | ✅ OTP | ✅ Tous sites | ✅ Tous comptes | Aucune |
| Fiduciaire | ✅ Bloquer | ⚠️ Étiquette ext. | ✅ Tous sites | ✅ Tous comptes | Faible |
| Architectes | ⚠️ Partage contrôlé | ✅ OTP | ✅ Tous sites | ✅ Tous comptes | Modérée |
| Avocats | ✅ Bloquer | ⚠️ Étiquette ext. | ✅ Tous sites | ✅ Tous comptes | Faible |
| Communication | ✅ Bloquer | ✅ OTP | ⚠️ Sites ciblés | ⚠️ Comptes ciblés | Faible |
| Cabinet médical | ✅ Bloquer | ⚠️ Étiquette ext. | ✅ Tous sites | ✅ Tous comptes | Faible |

---

## Annexe I : Registre des activités de traitement (art. 12 nLPD)

> **⚠️ ATTENTION — Exemple pré-rempli**
> Remplacez Axonix SA, demo.ch et admin@demo.ch par les données réelles du client.

**Informations générales de l'organisation**

| Champ | À compléter |
|-------|-------------|
| Raison sociale | `[Nom de la société]` |
| Responsable du traitement (art. 5 let. j nLPD) | `[Nom, Prénom, Fonction]` |
| DPO / Conseiller en protection des données | `[Nom, Prénom ou Externe / Pas de DPO désigné]` |
| Date de dernière mise à jour | `[JJ/MM/AAAA]` |

**Activité 1 : Gestion des ressources humaines et de la paie**

| Champ | Valeur — Axonix SA |
|-------|-------------------|
| Finalité | Administrer les contrats de travail, calculer et verser les salaires, gérer les absences. |
| Personnes concernées | Employés actifs et anciens employés. |
| Catégories de données | Numéro AVS, coordonnées, salaire, évaluations, données médicales (certificats), IBAN. |
| Destinataires | Fiduciaire, caisse AVS, assureur LAA, assureur maladie, Microsoft (sous-traitant). |
| Conservation | 10 ans après fin du contrat (CO art. 958f). |
| Base légale | Exécution du contrat de travail (art. 31 al. 2 let. a nLPD). |
| Mesures techniques | Étiquette `2 - Confidentiel` (AES-256, Azure RMS). `DLP-PME-Surveillance`. `Retention-RH-10ans`. MFA actif. |

**Activité 2 : Comptabilité et gestion financière**

| Champ | Valeur — Axonix SA |
|-------|-------------------|
| Finalité | Tenir la comptabilité, gérer paiements fournisseurs et encaissements clients. |
| Personnes concernées | Fournisseurs, clients (contacts de facturation), employés (notes de frais). |
| Catégories de données | IBAN, coordonnées de facturation, montants, références de contrats. |
| Destinataires | Fiduciaire, banque, AFC, administrations fiscales cantonales, Microsoft (sous-traitant). |
| Conservation | 10 ans (CO art. 958f). |
| Base légale | Obligation légale (CO art. 957 ss). |
| Mesures techniques | Étiquette `2 - Confidentiel`. Règle DLP `Avertir-IBAN-Externe` (justification obligatoire). `Retention-Finances-10ans`. MFA actif. |

**Activité 3 : Gestion de la relation client**

| Champ | Valeur — Axonix SA |
|-------|-------------------|
| Finalité | Exécuter les contrats de prestation, suivi commercial, facturation. |
| Personnes concernées | Contacts clients (personnes physiques, contexte B2B principalement). |
| Catégories de données | Nom, coordonnées professionnelles, historique des commandes et contrats. |
| Conservation | Durée du contrat + 5 ans (prescription CO art. 127). |
| Base légale | Exécution du contrat (art. 31 al. 2 let. a nLPD). |
| Mesures techniques | Étiquette `1 - Interne`. Accès restreint par site SharePoint. `DLP-PME-Surveillance`. |

→ Ajoutez ici les activités supplémentaires spécifiques au client.

---

[← Retour au guide Purview](../) · [🏠 Retour au portail principal](https://doit4everyone.github.io/)

---

ℹ️ *Références, structuration et aide à la rédaction assistées par IA, avec validation humaine finale.*
