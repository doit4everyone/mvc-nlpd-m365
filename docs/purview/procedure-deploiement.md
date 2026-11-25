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

<h1>Procédure de déploiement MVC : Microsoft Purview</h1>
<h2>Guide consultant · Version 1.3 · Durée : 4 à 6 heures</h2>

| Champ | Valeur |
|-------|--------|
| Licence | Microsoft 365 Business Premium + Suite Microsoft Purview pour Microsoft 365 Business Premium (module complémentaire) |
| Tenant | `[tenant].onmicrosoft.com` : à remplacer |
| Client | `[Nom du client]` : à remplacer |
| Version | 1.3, novembre 2026 (libellés relevés dans le portail Purview le 21.11.2026) |

---

## Sommaire

- [Préface](#préface)
- [Partie 0 : Prérequis](#partie-0--prérequis)
  - [0.1 Accéder au portail Purview](#01-accéder-au-portail-purview)
  - [0.2 PowerShell en administrateur](#02-powershell-en-administrateur)
  - [0.3 Attribuer les rôles](#03-attribuer-les-rôles)
  - [0.4 Vérifier la licence Purview](#04-vérifier-la-licence-purview)
  - [0.5 Étiquettes de groupes et sites (non requis)](#05-étiquettes-de-groupes-et-sites-non-requis)
  - [0.6 Activer le MFA sur tous les comptes](#06-activer-le-mfa-sur-tous-les-comptes)
  - [0.7 Bloquer le partage externe SharePoint](#07-bloquer-le-partage-externe-au-niveau-du-tenant-sharepoint)
- [Partie 1 : Activer l'audit](#partie-1--activer-laudit)
- [Partie 2 : Créer les 3 étiquettes de sensibilité](#partie-2--créer-les-3-étiquettes-de-sensibilité)
  - [2.1 Créer les SITs personnalisés](#21-créer-les-sits-personnalisés)
  - [2.2 Créer les 3 étiquettes](#22-créer-les-3-étiquettes)
  - [2.3 Publier les étiquettes](#23-publier-les-étiquettes)
  - [2.4 Auto-labelling côté service](#24-auto-labelling-côté-service-fichiers-sharepoint-existants)
  - [2.5 Break-glass RMS](#25-break-glass-rms-accès-de-secours-aux-fichiers-chiffrés)
- [Partie 3 : DLP Exchange](#partie-3--dlp-exchange)
- [Partie 4 : Rétention automatique des données](#partie-4--rétention-automatique-des-données)
- [Partie 5 : Vérification et remise](#partie-5--vérification-et-remise)
- [Annexe A : Glossaire simplifié](#annexe-a--glossaire-simplifié)
- [Annexe B : Checklist MVC complète](#annexe-b--checklist-mvc-complète)
- [Annexe C : Mots-clés pour les SIT personnalisés](#annexe-c--mots-clés-pour-les-sit-personnalisés)
- [Annexe D : Différences POC vs Production](#annexe-d--différences-poc-vs-production)
- [Annexe E : Adaptations sectorielles](#annexe-e--adaptations-sectorielles)
- [Annexe F : Suivi nLPD dans le Gestionnaire de conformité (option)](#annexe-f--suivi-nlpd-dans-le-gestionnaire-de-conformité-option)
- [Annexe G : Fiche réflexe rançongiciel](#annexe-g--fiche-réflexe-rançongiciel)
- [Annexe H : Registre des activités de traitement](#annexe-h--registre-des-activités-de-traitement-art-12-nlpd)

---

## Préface

> **⚠️ ATTENTION : Personnalisation obligatoire**
> Remplacez `[domaine]`, `[tenant]` et `[tenant].sharepoint.com` par les informations réelles du client. L'Annexe H est pré-remplie pour Axonix SA (POC demo.ch) ; adaptez-la au client avant remise.

Ce guide est conçu pour un consultant qui installe une protection Minimum Viable de Conformité (MVC) dans une PME suisse de 5 à 25 personnes. Objectif : une session de 4 à 6 heures, client autonome à la fin.

**Qu'est-ce que le MVC ?** Le Minimum Viable de Conformité est l'ensemble minimal de mesures techniques défendables devant le Préposé fédéral à la protection des données (PFPDT). Il ne vise pas la perfection technique mais la capacité de démontrer une démarche proportionnée et documentée. Pour une PME de 5 à 25 personnes sans IT dédié, c'est le point de départ réaliste et maintenable.

**Piliers MVC couverts :**

| Pilier | Ce que ça apporte |
|--------|------------------|
| MFA (tous les comptes) | Première barrière contre la compromission de compte |
| Blocage partage externe SharePoint | Fermeture du vecteur de fuite via liens SharePoint |
| Audit (traçabilité) | Journal des actions sur les données : 180 jours par défaut (Audit Standard), 1 an pour Entra ID, Exchange, OneDrive et SharePoint avec Audit Premium |
| 3 étiquettes (Interne, Confidentiel, Confidentiel-Externe) | Classification + chiffrement AES-256 des données sensibles, accès interne seul pour les données confidentielles |
| Auto-labelling service | Détection et étiquetage automatique des fichiers SharePoint existants |
| DLP Exchange | Envoi externe de données AVS ou médicales soumis à justification, simple avertissement pour l'IBAN |
| Rétention | Pièces comptables conservées 10 ans (CO art. 958f), dossiers RH protégés 10 ans contre la suppression (CO art. 127, 128 ch. 3 et 328b) |
| Break-glass RMS | Accès de secours aux fichiers chiffrés en cas d'urgence |

> **ℹ️ INFO : Hors périmètre MVC**
> IRM, Endpoint DLP, Managed Environments, eDiscovery, Communication Compliance, DSPM for AI. Ces fonctionnalités peuvent être déployées progressivement après le socle MVC.

> **ℹ️ INFO : MVC ou guide complet**
> Ce guide est la version minimale. Le guide complet « Configuration Microsoft Purview 2026 » couvre des besoins plus larges, par exemple un site dédié à la transmission externe au lieu du blocage du partage sur tout le tenant. Les deux guides sont des alternatives : n'appliquez pas l'un par-dessus l'autre. Pour passer du MVC au guide complet, modifiez ou renommez les types d'informations, étiquettes et stratégies déjà créés plutôt que d'en créer de nouveaux, sinon les détections et les incidents seront comptés en double. Attention aux noms proches : `AVS Suisse` dans ce guide, `AVS-Suisse` dans le guide complet.

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

**Exchange Online** (`admin.exchange.microsoft.com` → Rôles → Compliance Management)

\+ Ajouter des membres → votre compte. Attendez 5 à 10 minutes avant de continuer.

> **🚨 IMPORTANT**
> L'activation de l'audit exige le rôle Audit Logs d'Exchange Online. Selon Microsoft, les groupes de rôles Compliance Management et Organization Management l'ont par défaut ; Compliance Management donne moins de droits. Source : [Microsoft Learn, activer ou désactiver l'audit](https://learn.microsoft.com/purview/audit-log-enable-disable).

**Purview RBAC** (Paramètres → Rôles et étendues → Groupes de rôles : page « Groupes de rôles pour les solutions Microsoft Purview », onglet Groupes de rôle)

| Groupe Purview | Accès |
|----------------|-------|
| Compliance Administrator | Labels, DLP, rétention, politiques |
| Content Explorer Content Viewer | Voir le contenu des fichiers |
| Content Explorer List Viewer | Voir la liste des fichiers |

→ Cliquez sur le groupe → Modifier → ajoutez votre compte. Les noms des groupes s'affichent en anglais, même dans le portail en français.

### 0.4 Vérifier la licence Purview

`admin.microsoft.com` → Utilisateurs actifs → votre compte → Licences et applications → vérifiez que **Suite Microsoft Purview pour Microsoft 365 Business Premium** est coché. Si absent : cochez et Enregistrer les modifications.

> **ℹ️ INFO : Délai d'activation**
> Juste après l'attribution, certaines pages de Purview (par exemple la liste des stratégies d'étiquetage automatique) peuvent afficher « Erreur du client » en indiquant d'attendre 15 minutes. Patientez avant de conclure à un problème de licence.

### 0.5 Étiquettes de groupes et sites (non requis)

Aucune étiquette du MVC n'a la portée Groupes et sites (elle est décochée en 2.2) : l'activation de `EnableMIPLabels` et la synchronisation `Execute-AzureAdLabelSync` ne sont pas nécessaires. Elles ne le deviennent que si le client veut étiqueter des équipes Teams, des groupes Microsoft 365 ou des sites SharePoint ; voir alors [Microsoft Learn, attribuer des étiquettes aux groupes Microsoft 365](https://learn.microsoft.com/entra/identity/users/groups-assign-sensitivity-labels).

### 0.6 Activer le MFA sur tous les comptes

> **🚨 IMPORTANT : Priorité absolue**
> Sans MFA, Purview est un château de cartes. Un mot de passe volé donne accès à toutes les données sensibles.

| N° | Étape | Action |
|----|-------|--------|
| 1 | Voie recommandée | Accès conditionnel Entra ID : stratégie MFA pour tous les utilisateurs, avec exclusion des comptes d'urgence. Suivez le guide MVC Entra ID P1 (même série), qui détaille cette configuration. |
| 2 | Contrôle | Vérifiez qu'aucun compte actif n'est exclu de la stratégie MFA, hors comptes d'urgence documentés. |

> **ℹ️ INFO : MFA par utilisateur**
> L'ancien MFA par utilisateur (`admin.microsoft.com` → Utilisateurs actifs → Authentification multifacteur) existe encore, mais ne permet pas d'exclusions ciblées. Ne combinez pas les deux mécanismes.

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

> **🚨 IMPORTANT : À faire AVANT les étiquettes**
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
> Vérification dans Purview : Purview → Audit (menu gauche) → Rechercher. Si le formulaire de recherche s'affiche directement (Date et plage horaire, Activités, Utilisateurs), l'audit est actif. Si un bandeau propose **Démarrer l'enregistrement** → cliquez dessus.

---

## Partie 2 : Créer les 3 étiquettes de sensibilité

| Ordre | Étiquette | Rôle |
|-------|-----------|------|
| 1 | 1 - Interne | Sans chiffrement. Étiquette par défaut pour tous les documents internes. |
| 2 | 2 - Confidentiel | Avec chiffrement AES-256, ouverture réservée aux personnes internes autorisées. Pour AVS, salaires, médical, contrats, données bancaires. |
| 3 | 3 - Confidentiel-Externe | Avec chiffrement AES-256, ouverture possible par un destinataire externe. À choisir manuellement avant d'envoyer un document confidentiel à l'extérieur. |

### 2.1 Créer les SITs personnalisés

> **ℹ️ INFO**
> SIT = type d'informations sensibles. L'interface affiche selon les pages « Types d'infos confidentielles » ou « Sensitive info types (classic) » (libellé anglais dans le menu). Un bandeau annonce l'unification des classifieurs (« Go to Classifiers (preview) ») : la procédure ci-dessous utilise la page classique. Propagation : attendez 15 à 30 minutes après création avant de configurer les étiquettes.
>
> Types natifs utiles pour la Suisse : *Swiss Social Security Number AHV*, *International Banking Account Number (IBAN)*, *Switzerland Physical Addresses*.

#### SIT 1 : AVS Suisse

Détecte les numéros AVS suisses au format exact `756.XXXX.XXXX.XX` en 4 langues nationales + anglais.

| N° | Étape | Action |
|----|-------|--------|
| 1 | Navigation | Purview → Protection des données → Classifieurs → **Sensitive info types (classic)** → + Créer un type d'informations sensibles. La page s'intitule « Types d'infos confidentielles ». |
| 2 | Étape Nom | Nom : `AVS Suisse` \| Description (obligatoire) : Détection numéro AVS suisse 756.XXXX.XXXX.XX, 4 langues nationales + anglais. → Suivant |
| 3 | Étape Modèles | + Créer un modèle → volet « Nouveau modèle » → Niveau de confiance : **Confiance moyenne** (par défaut : Confiance élevée) → Élément principal : Ajout d'un élément principal → Expression régulière → ID : `regex-AVS-Suisse` → collez : `\b756\.\d{4}\.\d{4}\.\d{2}\b` → Terminé |
| 4 | Éléments complémentaires | Dans le même volet → Éléments complémentaires → Liste de mots clés → ID : `recherche-AVS-Suisse` → saisissez un mot-clé par ligne (voir Annexe C) → Terminé → Créer. Laissez la proximité de caractère à 300. |
| 5 | Étapes Niveau de confiance recommandé et Terminer | Laissez la valeur proposée → Suivant → Créer. Testez ensuite la détection avec `756.1234.5678.94` (Test 3). |
| ⚠️ | Propagation | Attendez 15 à 30 minutes avant de configurer les règles qui utilisent ce SIT. |

> **ℹ️ INFO**
> `756.1234.5678.94` est un numéro de test fictif dont la clé de contrôle (EAN-13, dernier chiffre) est **volontairement fausse**. `AVS Suisse` ne contrôle pas la clé : il détecte tout numéro au format AVS, y compris les numéros mal saisis ou inventés, ce qui est le comportement voulu. Le SIT natif *Swiss Social Security Number AHV*, lui, vérifie la clé et ignore ce numéro. Pour tester aussi le SIT natif, utilisez un numéro à clé valide, par exemple `756.1234.5678.97`.
>
> Saisissez l'expression régulière avec **un seul** antislash devant chaque `.` et chaque `d`, exactement comme affiché : `\b756\.\d{4}\.\d{4}\.\d{2}\b`.

> **✅ CONSEIL : Combiner le SIT natif et le SIT personnalisé**
> Dans les règles (étiquette 2, étiquetage automatique, DLP), ajoutez les deux types dans le même groupe avec l'opérateur « N'importe lequel de ces éléments » :
> - *Swiss Social Security Number AHV* (natif, Confiance élevée) : contrôle la clé, reconnaît aussi les numéros sans points et ses mots-clés couvrent le français, l'allemand et l'anglais.
> - `AVS Suisse` (personnalisé, Confiance moyenne) : détecte les numéros mal saisis ou inventés et apporte les mots-clés italiens et romanches de l'Annexe C.
>
> Le format `756.XXXX.XXXX.XX` avec points est très spécifique et le mot-clé exigé dans les 300 caractères limite les faux positifs du SIT personnalisé.

#### SIT 2 : Medical-RH-Suisse

Détecte les données médicales RH (arrêts maladie, certificats, incapacité de travail) en 4 langues nationales + anglais.

| N° | Étape | Action |
|----|-------|--------|
| 1 | Navigation | Purview → Protection des données → Classifieurs → **Sensitive info types (classic)** → + Créer un type d'informations sensibles |
| 2 | Étape Nom | Nom : `Medical-RH-Suisse` \| Description : Détection données médicales RH, 4 langues nationales + anglais. → Suivant |
| 3 | Étape Modèles | + Créer un modèle → Niveau de confiance : **Confiance moyenne** → Ajout d'un élément principal → Liste de mots clés → ID : `medical-rh-suisse` → saisissez un mot-clé par ligne (voir Annexe C) → Terminé → Créer |
| 4 | Terminer | Suivant → Créer. Attendez 15 à 30 minutes avant de continuer. |

> **ℹ️ INFO : Termes volontairement exclus (faux positifs)**
> La liste de l'Annexe C.2 ne contient que des termes qui désignent directement une donnée de santé, et la règle exige au moins 2 occurrences. Sont exclus :
> - les codes de diagnostic CIM-10 (forme `F41.1`) : une lettre suivie de chiffres et d'un point se confond avec des références d'articles, de plans ou de versions ;
> - LAMal, TARMED, TARDOC : présents dans tout courrier d'assurance ou toute facture, sans indiquer forcément une donnée médicale sur un employé ;
> - les termes d'aptitude (« aptitude au travail », « fit for work », « idoneità al lavoro », « abilita da lavurar ») : fréquents dans les offres d'emploi et les contrats.

### 2.2 Créer les 3 étiquettes

> **🚨 IMPORTANT : Ordre de création**
> Créez les étiquettes dans cet ordre : `1 - Interne`, puis `3 - Confidentiel-Externe`, puis `2 - Confidentiel`. Le portail attribue à chaque nouvelle étiquette la priorité la plus élevée (« Par défaut, cette étiquette se verra attribuer la priorité la plus élevée »). Ainsi, `2 - Confidentiel` passe au-dessus de `3 - Confidentiel-Externe`, et chaque passage de 2 à 3 avant un envoi externe exige une justification (voir l'étiquette 3). Créez d'abord le groupe `GRP-Confidentiel-Purview` décrit sous l'étiquette 2.

#### Étiquette 1 : 1 - Interne (sans chiffrement, par défaut)

| N° | Étape | Action |
|----|-------|--------|
| 1 | Navigation | Purview → Protection des données → Étiquettes de confidentialité → **+** → **Étiquette** |
| 2 | Détails de l'étiquette | Nom et Afficher le nom : `1 - Interne` \| Priorité de l'étiquette : laissez la valeur proposée \| Description pour les utilisateurs (obligatoire) : Document à usage interne uniquement. \| Couleur d'étiquette : Bleu → Suivant |
| 3 | Portée | Gardez cochés : Fichiers et autres ressources de données, Courriers, Réunions. **Décochez Groupes et sites** → Suivant |
| 4 | Éléments | Ne cochez rien (ni Contrôler l'accès, ni Appliquer le marquage de contenu) → Suivant jusqu'à Terminer |
| 5 | Terminer | Créer une étiquette → écran « Étapes suivantes » : **Publier l'étiquette est présélectionné**, cochez **Ne créez pas encore de stratégie** → Terminé. |

#### Étiquette 2 : 2 - Confidentiel (avec chiffrement AES-256)

> **ℹ️ INFO : Groupes mail-enabled requis pour le chiffrement RMS**
> Azure Rights Management exige que les groupes assignés aient une adresse email. En tenant cloud pur (sans AD on-prem) :
> ```powershell
> Connect-ExchangeOnline
> New-DistributionGroup -Name "GRP-Confidentiel-Purview" -Alias "grp-confidentiel-purview" -Type Security -PrimarySmtpAddress "grp-confidentiel-purview@[domaine]"
> Add-DistributionGroupMember -Identity "GRP-Confidentiel-Purview" -Member utilisateur@[domaine]
> ```
> En environnement hybride (AD on-prem) : créez le groupe dans AD et attendez la synchronisation Entra Connect.

| N° | Étape | Action |
|----|-------|--------|
| 1 | Navigation | Purview → Protection des données → Étiquettes de confidentialité → **+** → **Étiquette** |
| 2 | Détails de l'étiquette | Nom et Afficher le nom : `2 - Confidentiel` \| Description pour les utilisateurs : Données personnelles sensibles (AVS, salaires, médical, contrats), usage interne. Pour un envoi externe, choisissez 3 - Confidentiel-Externe. \| Couleur d'étiquette : Bordeaux → Suivant |
| 3 | Portée | Gardez cochés : Fichiers et autres ressources de données, Courriers, Réunions. **Décochez Groupes et sites** → Suivant |
| 4 | Éléments | Cochez **Contrôler l'accès** et **Appliquer le marquage de contenu** → Suivant |
| 5 | Contrôle de l'accès | Configurer les paramètres de contrôle d'accès \| Attribuer des autorisations maintenant \| Expiration de l'accès de l'utilisateur au contenu : Jamais \| Autoriser l'accès hors connexion : **Uniquement pendant un nombre de jours défini** → saisissez `30` \| Attribuer des autorisations → Ajouter des utilisateurs ou des groupes : `GRP-Confidentiel-Purview` et `admin@[domaine]` uniquement. **N'ajoutez pas** « Ajouter tout utilisateur authentifié » (voir l'avertissement ci-dessous) \| Choisir les autorisations : **Éditeur** → Enregistrer |
| 6 | Marquage de contenu | Activez l'interrupteur « Marquage de contenu » → Ajouter un filigrane → Personnaliser le texte : CONFIDENTIEL, taille 40, couleur Rouge, disposition Diagonal \| Ajouter un en-tête : CONFIDENTIEL, Usage interne uniquement \| Ajouter un pied de page : Document protégé nLPD, Ne pas diffuser → Suivant |
| 7 | Étiquetage automatique des fichiers et des courriers | Activez l'interrupteur → Ajouter une condition → **Le contenu inclut** \| Nom du groupe : `Donnees-Sensibles` \| Opérateur de groupe : N'importe lequel de ces éléments \| Ajouter → Types d'infos confidentielles : AVS Suisse (Confiance moyenne, 1 instance) + Swiss Social Security Number AHV (Confiance élevée, 1 instance) + Medical-RH-Suisse (Confiance moyenne, 2 instances) \| Lorsque le contenu correspond à ces conditions : **Appliquer automatiquement l'étiquette** → Suivant |
| 8 | Groupes et sites | Page sans objet (portée décochée) → Suivant |
| 9 | Terminer | Créer une étiquette → écran « Étapes suivantes » : cochez **Ne créez pas encore de stratégie** (Publier l'étiquette est présélectionné) → Terminé. |
| ⚠️ | Propagation | Attendez 15 à 30 minutes avant de configurer la DLP et l'auto-labelling service. |

> **⚠️ ATTENTION : Deux mécanismes d'auto-labelling distincts**
> L'étape 7 configure l'auto-labelling **côté client** (ouverture dans Word ou Outlook). Les fichiers déjà présents dans SharePoint ne sont pas couverts. La section 2.4 configure l'auto-labelling **côté service**. Les deux mécanismes sont complémentaires.

> **ℹ️ INFO : Pourquoi l'IBAN ne déclenche pas l'étiquette 2**
> Presque toutes les factures contiennent un IBAN. S'il déclenchait l'étiquette 2, chaque facture serait chiffrée et réservée à l'interne, et devrait être réétiquetée avant chaque envoi. Un IBAN seul n'est pas une donnée sensible au sens de l'art. 5 let. c nLPD. Les fichiers vraiment sensibles qui contiennent des IBAN, comme les listes de salaires, contiennent aussi des numéros AVS et restent étiquetés. L'IBAN est traité par la seule règle DLP 2 (section 3.3), qui avertit sans bloquer.

> **⚠️ ATTENTION : Pourquoi l'étiquette 2 reste interne**
> Selon la documentation Microsoft, le réglage « Ajouter tout utilisateur authentifié » **ne restreint pas qui peut ouvrir le contenu** : toute personne qui s'authentifie avec une identité prise en charge peut l'ouvrir. Avec ce réglage, un document volé (rançongiciel avec exfiltration, boîte mail compromise) s'ouvre avec n'importe quel compte Microsoft : le chiffrement ne protège plus la confidentialité. C'est pourquoi `2 - Confidentiel` est réservée aux personnes autorisées, et l'envoi externe passe par l'étiquette 3, qui ne concerne que les documents destinés à sortir. Source : [Microsoft Learn, chiffrement des étiquettes de confidentialité](https://learn.microsoft.com/purview/encryption-sensitivity-labels).

> **ℹ️ INFO : Avertissement affiché par le portail**
> Lorsque l'étiquetage automatique est activé sur une étiquette qui chiffre, le portail rappelle que les fichiers Office et PDF concernés seront chiffrés, que leur ouverture et leur enregistrement seront plus lents et que certaines fonctionnalités SharePoint et OneDrive seront limitées.

#### Étiquette 3 : 3 - Confidentiel-Externe (chiffrement, envoi externe)

Même assistant que l'étiquette 2 (écrans détaillés ci-dessus), avec les différences suivantes. Créez-la **avant** l'étiquette 2.

| N° | Étape | Action |
|----|-------|--------|
| 1 | Navigation | Purview → Protection des données → Étiquettes de confidentialité → **+** → **Étiquette** |
| 2 | Détails de l'étiquette | Nom et Afficher le nom : `3 - Confidentiel-Externe` \| Description pour les utilisateurs : Données sensibles à transmettre par e-mail à un destinataire externe. \| Couleur d'étiquette : Orange → Suivant |
| 3 | Portée | Comme l'étiquette 2 : **décochez Groupes et sites** → Suivant |
| 4 | Éléments | Cochez **Contrôler l'accès** et **Appliquer le marquage de contenu** → Suivant |
| 5 | Contrôle de l'accès | Mêmes réglages que l'étiquette 2 (Jamais, 30 jours, `GRP-Confidentiel-Purview` et `admin@[domaine]`, **Éditeur**), plus **Ajouter tout utilisateur authentifié** → Enregistrer |
| 6 | Marquage de contenu | Ajouter un en-tête : CONFIDENTIEL, Transmission externe autorisée \| Ajouter un pied de page : Document protégé nLPD, Ne pas diffuser → Suivant |
| 7 | Étiquetage automatique des fichiers et des courriers | **Laissez l'interrupteur désactivé** : cette étiquette est toujours choisie volontairement par l'utilisateur → Suivant |
| 8 | Groupes et sites | Page sans objet → Suivant |
| 9 | Terminer | Créer une étiquette → **Ne créez pas encore de stratégie** → Terminé. |

> **ℹ️ INFO : Accès des destinataires externes**
> Selon la documentation Microsoft : le code à usage unique et les fournisseurs sociaux (Google) ne fonctionnent que pour les **e-mails et invitations** via Exchange Online. Pour un **document** chiffré joint ou transmis, le destinataire doit disposer d'un compte Microsoft (compte professionnel ou compte Microsoft personnel) ouvert avec les applications Office ou la visionneuse Purview Information Protection. Informez le client de cette friction avant la mise en production.

> **⚠️ ATTENTION : Avant chaque envoi externe**
> Un document étiqueté automatiquement `2 - Confidentiel` ne s'ouvrira pas chez un destinataire externe. Avant l'envoi, l'utilisateur change l'étiquette en `3 - Confidentiel-Externe` (ruban Sensibilité). Créée avant l'étiquette 2, l'étiquette 3 a une priorité plus basse : le changement est un déclassement et affiche la fenêtre « Justification requise » (« Votre organisation demande une justification pour modifier cette étiquette. »). C'est voulu : ouvrir un document interne à l'extérieur est le moment où une trace a le plus de valeur. Le passage inverse, de 3 à 2, se fait sans justification (comportement vérifié sur le lab le 23.11.2026).

### 2.3 Publier les étiquettes

| N° | Étape | Action |
|----|-------|--------|
| 1 | Navigation | Purview → Protection des données → Stratégies → Stratégies de publication. La page s'intitule « Stratégies des étiquettes » → **Publier l'étiquette** |
| 2 | Étiquettes à publier | Sélectionnez les étiquettes de confidentialité à publier → cochez `1 - Interne`, `2 - Confidentiel` et `3 - Confidentiel-Externe` → Ajouter → Suivant |
| 3 | Unités d'administration | Laissez Répertoire complet → Suivant |
| 4 | Utilisateurs et groupes | Laissez E-mail Exchange : Tous les utilisateurs et groupes → Suivant |
| 5 | Paramètres | Cochez : (1) Les utilisateurs doivent fournir une justification pour supprimer une étiquette ou abaisser son niveau de classification \| (2) Obliger les utilisateurs à appliquer une étiquette à leurs courriers et documents → Suivant |
| 6 | Sous-étape Documents | Appliquer une étiquette par défaut aux documents → Étiquette par défaut : `1 - Interne` → Suivant |
| 7 | Sous-étape E-mails | Étiquette par défaut : `1 - Interne` (par défaut « Identique au document »). Laissez « Hériter l'étiquette des pièces jointes » par défaut → Suivant |
| 8 | Autres sous-étapes | Réunions, Sites et groupes, Engagé (aperçu), Structure et Power BI : laissez les valeurs par défaut → Suivant |
| 9 | Nom | Nom : `Politique-Labels-[client]` → Suivant |
| 10 | Terminer | Vérifiez le résumé puis cliquez sur Envoyer. Propagation : jusqu'à 24 heures. |

### 2.4 Auto-labelling côté service (fichiers SharePoint existants)

| N° | Étape | Action |
|----|-------|--------|
| 1 | Navigation | Purview → Protection des données → Stratégies → Stratégies d'étiquetage automatique → **+** |
| 2 | Type de stratégie | Choisissez **Appliquer automatiquement uniquement les étiquettes** (les deux autres choix servent à supprimer des étiquettes ou à appliquer l'étiquette par défaut des bibliothèques SharePoint) |
| 3 | Informations à étiqueter | Catégories → faites défiler → **Personnaliser** → Stratégie personnalisée → Suivant |
| 4 | Nom | Nom : `Auto-Label-Confidentiel` \| Description : Détection automatique AVS et médical dans SharePoint et OneDrive. → Suivant |
| 5 | Étiquette | Choisir une étiquette → cochez `2 - Confidentiel` → Ajouter → Suivant |
| 6 | Unités d'administration | Laissez Répertoire complet → Suivant |
| 7 | Emplacements | Tous les emplacements sont **désactivés par défaut**. Activez : Courrier Exchange, Sites SharePoint, Comptes OneDrive (étendue : tous) → Suivant |
| 8 | Règles de stratégie | Laissez **Règles communes** → Suivant → Nouvelle règle |
| 9 | Règle | Nom : `Règle-Donnees-Sensibles` \| Ajouter une condition → **Le contenu contient** → Ajouter → Types d'infos confidentielles : AVS Suisse (Confiance moyenne, 1 instance) + Swiss Social Security Number AHV (Confiance élevée, 1 instance) + Medical-RH-Suisse (Confiance moyenne, 2 instances) → Enregistrer → Suivant |
| 10 | Paramètres d'étiquette supplémentaire | Laissez les cases décochées (pas de remplacement des étiquettes existantes, pas de chiffrement des e-mails reçus de l'extérieur) → Suivant |
| 11 | Mode de la stratégie | Laissez **Exécuter la stratégie en mode de simulation** (présélectionné). Ne cochez pas l'activation automatique après 7 jours → Suivant → Finaliser → créez la stratégie |

> **ℹ️ INFO**
> Le bouton « Activer la stratégie » reste grisé tant que la simulation n'a pas tourné. Après 24 à 48 heures : Purview → Stratégies d'étiquetage automatique → cliquez sur `Auto-Label-Confidentiel` → consultez les résultats de simulation. Si le résultat est correct → activez la stratégie.

### 2.5 Break-glass RMS (accès de secours aux fichiers chiffrés)

> **🚨 IMPORTANT : Obligatoire avant mise en production**
> Sans compte Break-glass configuré, un départ conflictuel ou une corruption du tenant peut entraîner une perte irréversible des données chiffrées.

**Étape 1 : Installer le module et se connecter (Windows PowerShell 5.1 uniquement) :**

```powershell
Install-Module -Name AIPService -Force
Connect-AipService
```

**Étape 2 : Activer la fonctionnalité Super User et désigner le compte de secours :**

```powershell
Enable-AipServiceSuperUserFeature
Add-AipServiceSuperUser -EmailAddress admin@[domaine]
```

**Étape 3 : Vérifier et désactiver après configuration :**

```powershell
Get-AipServiceSuperUser
Disable-AipServiceSuperUserFeature
```

> **⚠️ ATTENTION**
> Réactivez le Super User uniquement en cas d'urgence réelle. Selon Microsoft, l'activation et la désactivation du Super User, ainsi que l'ajout ou le retrait d'un Super User, sont consignés dans le journal d'administration du service (`Get-AipServiceAdminLog`), et non dans l'audit Purview. Les déchiffrements apparaissent dans les journaux d'utilisation, sans indiquer qu'il s'agit d'un Super User. Stockez les identifiants du compte de secours dans un coffre-fort physique.

---

## Partie 3 : DLP Exchange

> **ℹ️ INFO**
> Dans ce MVC, l'email est le seul canal légitime d'envoi de données vers l'extérieur (SharePoint est bloqué). Les deux règles partent en simulation 2 à 4 semaines avant activation.

> **⚠️ ATTENTION : Prérequis**
> Les SIT (section 2.1) doivent être créés et propagés (30 minutes) avant de créer cette politique.

### 3.1 Créer la politique DLP-PME-Surveillance

| N° | Étape | Action |
|----|-------|--------|
| 1 | Navigation | Purview → Protection contre la perte de données → Stratégies → créer une stratégie (bouton parfois regroupé sous « … ») → « Quelles informations voulez-vous protéger ? » : **Appareils et applications d'entreprise** |
| 2 | Modèle ou stratégie personnalisée | Catégories → faites défiler → **Personnaliser** → Stratégie personnalisée → Suivant |
| 3 | Nom | Nom : `DLP-PME-Surveillance` → Suivant |
| 4 | Unités d'administration | Laissez Répertoire complet → Suivant |
| 5 | Emplacements | Sept emplacements sont **cochés par défaut** (Courrier Exchange, Sites SharePoint, Comptes OneDrive, Messages de conversation et de canal Teams, Appareils, Instances, Référentiels locaux). Gardez **uniquement Courrier Exchange**, décochez les six autres → Suivant |
| 6 | Paramètres de la stratégie | Laissez **Créer ou personnaliser des règles de DLP avancées** → Suivant |
| 7 | Règles de DLP avancées | Créer une règle. Créez les 2 règles détaillées ci-dessous. → Suivant |
| 8 | Mode de la stratégie | **Exécuter la stratégie en mode simulation** + laissez cochée **Afficher les conseils de stratégie en mode simulation**. Ne cochez pas l'activation automatique après quinze jours → Suivant |
| 9 | Terminer | « Vérifier et finaliser » → Envoyer. |

> **⚠️ ATTENTION : Emplacements et conditions**
> La condition « Du contenu est partagé à partir de Microsoft 365 » n'apparaît que si les emplacements se limitent aux services Microsoft 365 (ici Courrier Exchange). Avec Appareils ou Instances cochés, le panneau ne propose que trois conditions.

### 3.2 Règle 1 : Bloquer-Donnees-Sensibles

| Paramètre | Configuration |
|-----------|---------------|
| Nom de la règle | `Bloquer-Donnees-Sensibles` |
| Condition 1 | Le contenu contient → N'importe lequel de ces éléments : AVS Suisse (Confiance moyenne, 1 instance) + Swiss Social Security Number AHV (Confiance élevée, 1 instance) + Medical-RH-Suisse (Confiance moyenne, 2 instances). L'IBAN est traité par la règle 2 uniquement, pour éviter deux incidents par envoi. |
| Condition 2 | Du contenu est partagé à partir de Microsoft 365 → **avec des personnes externes à mon organisation** |
| Actions | **Ajouter une action** → **Restreindre l'accès ou chiffrer le contenu dans des emplacements Microsoft 365** → **Ne bloquez que les personnes extérieures à votre organisation.** |
| Notifications d'utilisateur | Activez l'interrupteur → cochez la case des conseils de stratégie (libellée « Conseils Azure Policy » dans le portail en français) → Personnalisez le texte du conseil de stratégie : Ce message contient des données personnelles sensibles (AVS ou santé). L'envoi externe exige une justification. |
| Remplacements par l'utilisateur | Cochez **Autoriser les remplacements à partir de fichiers Microsoft 365 et d'éléments Microsoft Fabric** puis **Exiger une justification pour ignorer**. |
| Rapports d'incident | Niveau **Élevé**. Alerte email : `admin@[domaine]`. |

→ Cliquez sur Enregistrer. Puis + Créer une règle pour la règle suivante.

### 3.3 Règle 2 : Avertir-IBAN-Externe

| Paramètre | Configuration |
|-----------|---------------|
| Nom de la règle | `Avertir-IBAN-Externe` |
| Condition 1 | Le contenu contient → International Banking Account Number (IBAN) (Confiance élevée, 1 instance) |
| Condition 2 | Du contenu est partagé à partir de Microsoft 365 → **avec des personnes externes à mon organisation** |
| Actions | N'ajoutez aucune action : simple avertissement. |
| Notifications d'utilisateur | Activez l'interrupteur → case « Conseils Azure Policy » → Personnalisez le texte du conseil de stratégie : Ce message contient un IBAN. Vérifiez que le destinataire est le bon. |
| Remplacements par l'utilisateur | Laissez décoché : sans action restrictive, il n'y a rien à remplacer et aucune justification n'est enregistrée. |
| Rapports d'incident | Niveau **Moyen**. Alerte email : `admin@[domaine]`. |

→ Cliquez sur Enregistrer.

> **ℹ️ INFO : Pourquoi l'IBAN n'exige pas de justification**
> Presque toutes les factures contiennent un IBAN, ne serait-ce que celui de l'entreprise. Une justification à chaque facture produirait des justifications machinales sans valeur de preuve. La justification est réservée aux données AVS et médicales, plus rares et plus sensibles (art. 5 let. c nLPD pour les données de santé).

> **ℹ️ INFO : Justification et simulation**
> En mode simulation, rien n'est bloqué et aucune justification n'est demandée. La règle 1 ne bloque, avec possibilité de remplacement justifié, qu'après l'activation de la stratégie. Le portail précise que l'affichage des conseils de stratégie varie selon les applications, les versions d'Outlook et les licences.

> **ℹ️ INFO**
> Dans la liste des stratégies, le mode s'affiche « Dans la simulation avec notifications ». Après 2 à 4 semaines : consultez l'Activity Explorer → si les faux positifs sont acceptables → modifiez la politique → Mode de la stratégie → **Activer immédiatement la stratégie**.

---

## Partie 4 : Rétention des données

> **🚨 IMPORTANT**
> Sans politique de rétention configurée, les durées inscrites au registre des activités de traitement (Annexe H) n'ont pas de mesure technique correspondante. C'est une incohérence documentaire visible en audit.

### 4.1 Retention-RH-10ans

Le dossier du personnel n'a pas de durée de conservation unique : elle dépend de la catégorie de document (voir l'avertissement ci-dessous). La stratégie protège les documents RH contre une suppression trop précoce pendant la durée la plus longue, 10 ans, sans les supprimer automatiquement.

| N° | Étape | Action |
|----|-------|--------|
| 1 | Navigation | Purview → Gestion du cycle de vie des données → Stratégies → Stratégies de rétention → « … » → **Nouvelle stratégie de rétention** |
| 2 | Nom | Nom : `Retention-RH-10ans` \| Description : Conservation minimale des documents RH, 10 ans (CO art. 127), sans suppression automatique. → Suivant |
| 3 | Unités administratives | Répertoire complet → Suivant |
| 4 | Type | **Adaptative est présélectionné** : sélectionnez **Statique** → Suivant |
| 5 | Emplacements | Quatre interrupteurs sont **activés par défaut** (Boîtes aux lettres Exchange, Sites SharePoint classiques et de communication, Comptes OneDrive, Boîtes aux lettres et sites de groupe Microsoft 365). Désactivez-les tous sauf **Sites SharePoint classiques et de communication** → colonne Inclus : Modifier → recherchez le site RH (`https://[tenant].sharepoint.com/sites/RH`) → cochez-le → OK → Suivant |
| 6 | Paramètres de rétention | **Conserver des éléments pendant une période donnée** → **10 ans** (choix proposés : 5, 7, 10 ans ou Personnalisé, défaut 7) → Démarrer la période de rétention fondée sur : **Lorsque les éléments ont été modifiés en dernier lieu** → À la fin de la période de rétention : **Ne rien faire** → Suivant → Terminer → Envoyer. |

> **⚠️ ATTENTION : RH, la loi ne fixe pas une durée unique pour le dossier du personnel**
> La stratégie `Retention-RH-10ans` couvre la durée la plus longue qui peut se justifier pour un document RH. Elle ne dispense pas de trier le dossier par catégorie :
>
> | Catégorie | Durée | Base |
> |-----------|-------|------|
> | Décomptes de salaire, pièces comptables | 10 ans après la fin de l'exercice | CO art. 958f |
> | Données utiles au certificat de travail (fonction, durée, évaluations, formations) | 10 ans après la fin des rapports de travail, car le droit au certificat se prescrit par 10 ans | CO art. 127 |
> | Créances de salaire, vacances, heures supplémentaires | 5 ans après leur exigibilité | CO art. 128 ch. 3 |
> | Dossiers de candidature | À restituer ou détruire, au plus tard à la fin des rapports de travail | CO art. 328b |
> | Le reste du dossier | À nettoyer régulièrement : ce qui ne sert plus est détruit | CO art. 328b, art. 6 al. 4 nLPD |
>
> Deux conséquences techniques à connaître :
> 1. **Une stratégie de rétention empêche la suppression définitive avant son échéance.** Un fichier supprimé du site RH (une candidature non retenue, une ancienne évaluation) est déplacé dans la bibliothèque de conservation du site et y reste jusqu'au terme des 10 ans. Il n'est plus visible des utilisateurs, mais il est conservé. Documentez cette limite dans le registre (Annexe H, activité 1).
> 2. **La durée part de la dernière modification du fichier, pas du départ du collaborateur.** Une suppression automatique effacerait le document d'un collaborateur encore en poste, non modifié depuis 10 ans. C'est pourquoi le MVC choisit « Ne rien faire » à la fin de la période : le tri se fait manuellement, lors de la révision annuelle.
>
> En production, une rétention conforme à ce tableau passe par des étiquettes de rétention par catégorie, avec une durée qui démarre à un événement (fin des rapports de travail) : elles sortent du périmètre MVC. Sources : [Microsoft Learn, rétention pour SharePoint et OneDrive](https://learn.microsoft.com/purview/retention-policies-sharepoint) ; [WEKA, délais de conservation des données des collaborateurs](https://www.weka.ch/themes/ressources-humaines/contrats-de-travail-et-reglements/protection-des-donnees/article/donnees-rh-quels-sont-les-delais-de-conservation-des-donnees-des-collaborateurs/) ; [Swissmem, prescription d'une inscription dans le dossier personnel](https://www.swissmem.ch/fr/competences/ressources-humaines/droit-du-travail/quand-une-inscription-dans-un-dossier-personnel-est-elle-prescrite.html).

> **ℹ️ INFO : Décomptes de salaire**
> Les décomptes et justificatifs de salaire sont des pièces comptables (CO art. 958f, 10 ans). Rangez-les dans le site Finances, couvert par la stratégie 4.2.

> **ℹ️ INFO : Archive SharePoint**
> La page des paramètres propose une case d'archivage vers Microsoft 365 Archive, facturée au paiement à l'utilisation. Laissez-la décochée dans le MVC.

### 4.2 Retention-Finances-10ans

Même procédure que 4.1 avec les adaptations suivantes :

| Paramètre | Valeur |
|-----------|--------|
| Nom | `Retention-Finances-10ans` |
| Description | Conservation des pièces comptables, 10 ans (CO art. 958f). |
| URL SharePoint | `https://[tenant].sharepoint.com/sites/Finances` |
| Durée et action | 10 ans → **Supprimer automatiquement les éléments** |

> **ℹ️ INFO**
> N'étendez pas cette stratégie à d'autres sites : la suppression automatique après 10 ans ne se justifie que pour les pièces comptables. Une rétention plus fine par type de document passe par des étiquettes de rétention, hors périmètre MVC.

---

## Partie 5 : Vérification et remise

### Test 1 : Audit actif

```powershell
Get-AdminAuditLogConfig | FL UnifiedAuditLogIngestionEnabled
```
→ Résultat attendu : `True`. Alternative : Purview → Audit → Rechercher → le formulaire de recherche s'affiche sans bandeau « Démarrer l'enregistrement ».

### Test 2 : Étiquettes visibles dans Word

Ouvrez Word sur un PC utilisateur. Ruban → **Sensibilité**. Les 3 étiquettes doivent être présentes. Si absent : propagation jusqu'à 24 heures. Appliquez `2 - Confidentiel` à un document, puis changez-la en `3 - Confidentiel-Externe` : la fenêtre « Justification requise » s'affiche (« Votre organisation demande une justification pour modifier cette étiquette. ») avec trois choix : L'étiquette précédente ne s'applique plus, L'étiquette précédente était incorrecte, Autres (expliquer). Revenez à `2 - Confidentiel` : aucune justification n'est demandée.

### Test 3 : Détection automatique (auto-labelling client)

Dans Word, tapez `756.1234.5678.94` accompagné d'un mot-clé de l'Annexe C (par exemple « numéro AVS »), puis enregistrez le document. L'étiquette `2 - Confidentiel` doit s'appliquer automatiquement (mode « Appliquer automatiquement l'étiquette » choisi à l'étape 7 de la section 2.2). Le délai d'application dépend de la version de Word.

### Test 4 : Avertissement DLP

Envoyez par email à une adresse externe un document contenant `756.1234.5678.94`. Le conseil de stratégie doit apparaître dans Outlook avant l'envoi. En mode simulation, il ne s'affiche que si la case **Afficher les conseils de stratégie en mode simulation** est restée cochée (section 3.1, étape 8). Le comportement des conseils de stratégie varie selon la version d'Outlook.

### Test 5 : Audit

Purview → Audit → Rechercher → Date et plage horaire (UTC) : aujourd'hui → Activités : noms conviviaux : fichier consulté (ou Activités : noms d'opérations : `FileAccessed`) → Rechercher. Des entrées doivent correspondre à vos tests.

### Test 6 : Blocage partage SharePoint

Dans SharePoint, tentez de partager un fichier avec une adresse externe. Le partage doit être refusé : *Votre organisation ne permet pas le partage avec des personnes extérieures*.

### Checklist de remise client

| | Élément à valider | Statut |
|-|-------------------|--------|
| ☐ | MFA imposé à tous les comptes (accès conditionnel) | à valider |
| ☐ | Partage externe SharePoint bloqué au niveau tenant | à valider |
| ☐ | Audit actif (True dans PowerShell) | à valider |
| ☐ | SITs AVS Suisse et Medical-RH-Suisse créés et propagés | à valider |
| ☐ | Étiquettes 1 - Interne, 2 - Confidentiel (interne seul) et 3 - Confidentiel-Externe créées et publiées | à valider |
| ☐ | Auto-labelling service activé en mode simulation (section 2.4) | à valider |
| ☐ | Politiques de rétention RH et Finances créées (section 4) | à valider |
| ☐ | DLP-PME-Surveillance créée en mode simulation (2 règles) | à valider |
| ☐ | Break-glass RMS configuré (section 2.5) | à valider |
| ☐ | Étiquettes visibles dans Word sur un PC utilisateur (Test 2) | à valider |
| ☐ | Détection AVS fonctionnelle avec 756.1234.5678.94 (Test 3) | à valider |
| ☐ | Avertissement DLP déclenché sur envoi externe (Test 4) | à valider |
| ☐ | Partage SharePoint externe refusé (Test 6) | à valider |
| ☐ | Registre des activités (Annexe H) remis et adapté au client | à valider |

---

## Annexe A : Glossaire simplifié

| Terme | Définition |
|-------|------------|
| MVC (Minimum Viable de Conformité) | Ensemble minimal de mesures techniques défendables devant le PFPDT. Ne vise pas la perfection technique mais la capacité de démontrer une démarche proportionnée et documentée. |
| SIT | Détecteur de motifs qui cherche des numéros AVS, IBAN, etc. dans les documents. |
| Étiquette de sensibilité | Marqueur appliqué à un document qui indique son niveau de confidentialité et active les protections associées (chiffrement, filigrane). |
| Auto-labelling client | S'applique quand l'utilisateur modifie un fichier Office ou rédige un e-mail dans Outlook. Configuré lors de la création de l'étiquette (section 2.2), en mode « Appliquer automatiquement » ou « Recommander aux utilisateurs ». |
| Auto-labelling service | Analyse asynchrone des fichiers déjà présents dans SharePoint et OneDrive par Purview. Configuré via une stratégie dédiée (section 2.4). |
| Chiffrement AES-256 / Azure RMS | Technologie qui verrouille le contenu d'un document. Un fichier étiqueté `2 - Confidentiel` reste illisible pour les personnes non autorisées, même volé ou copié sur une clé USB, sauf si le compte d'une personne autorisée est compromis. Avec `3 - Confidentiel-Externe`, toute personne qui s'authentifie peut l'ouvrir. |
| DLP (Data Loss Prevention) | Politique qui surveille et peut bloquer les envois de données sensibles par email ou partage. |
| Rétention Purview | Politique qui empêche la suppression des documents pendant une durée donnée et peut, à l'échéance, les supprimer automatiquement ou ne rien faire. |
| Break-glass RMS | Procédure de secours permettant à un compte désigné d'accéder aux fichiers chiffrés en cas d'urgence. |
| Audit log | Journal qui enregistre les actions sur les données. Mesure de sécurité (art. 8 nLPD) qui permet aussi de documenter une violation de la sécurité des données à annoncer au PFPDT (art. 24 nLPD). La journalisation n'est une obligation légale (OPDo art. 4) que pour les traitements automatisés de données sensibles à grande échelle ou le profilage à risque élevé ; pour une PME, elle reste une bonne pratique. |
| PFPDT | Préposé fédéral à la protection des données. Autorité de contrôle suisse nLPD. |
| nLPD | Nouvelle loi fédérale sur la protection des données (en vigueur depuis septembre 2023). |

---

## Annexe B : Checklist MVC complète

| Réf. | Action à valider | Statut |
|------|------------------|--------|
| [0.3] | Rôles Entra ID (Sécurité + Conformité) attribués | ☐ |
| [0.3] | Compliance Management Exchange attribué | ☐ |
| [0.3] | Compliance Administrator Purview attribué | ☐ |
| [0.4] | Licence Suite Microsoft Purview attribuée au compte admin | ☐ |
| [0.6] | MFA imposé à TOUS les comptes par accès conditionnel (pas seulement admin) | ☐ |
| [0.7] | Partage externe bloqué au niveau tenant SharePoint | ☐ |
| [1] | Audit actif : UnifiedAuditLogIngestionEnabled = True | ☐ |
| [2.1] | SIT AVS Suisse créé et propagé (30 min) | ☐ |
| [2.1] | SIT Medical-RH-Suisse créé et propagé (30 min) | ☐ |
| [2.2] | Groupe GRP-Confidentiel-Purview créé (mail-enabled) | ☐ |
| [2.2] | Étiquette 1 - Interne créée | ☐ |
| [2.2] | Étiquette 2 - Confidentiel créée (chiffrement interne seul + auto-labelling client) | ☐ |
| [2.2] | Étiquette 3 - Confidentiel-Externe créée (chiffrement, tout utilisateur authentifié, sans auto-labelling) | ☐ |
| [2.3] | Les 3 étiquettes publiées (Politique-Labels-[client]) | ☐ |
| [2.4] | Stratégie Auto-Label-Confidentiel créée et simulée | ☐ |
| [2.5] | Break-glass RMS configuré (Super User RMS désigné) | ☐ |
| [3] | DLP-PME-Surveillance créée (2 règles, mode simulation) | ☐ |
| [4] | Retention-RH-10ans créée (SharePoint RH, « Ne rien faire » à l'échéance) | ☐ |
| [4] | Retention-Finances-10ans créée (SharePoint Finances) | ☐ |
| [5] | Test 1 : Audit actif vérifié | ☐ |
| [5] | Test 2 : Étiquettes visibles dans Word | ☐ |
| [5] | Test 3 : Détection AVS avec 756.1234.5678.94 fonctionnelle | ☐ |
| [5] | Test 4 : Avertissement DLP sur envoi externe déclenché | ☐ |
| [5] | Test 6 : Partage SharePoint externe refusé | ☐ |
| [H] | Registre des activités adapté et remis au client | ☐ |

---

## Annexe C : Mots-clés pour les SIT personnalisés

> **ℹ️ INFO : Astuce copier-coller**
> Copiez les mots-clés dans un document Word vierge, Ctrl+H : Rechercher « / » → Remplacer par `^p`. Vous obtenez la liste prête à coller dans Purview, un mot-clé par ligne.

### C.1 : AVS Suisse

Expression régulière : `\b756\.\d{4}\.\d{4}\.\d{2}\b` | Niveau de confiance : Confiance moyenne | min : 1

| Langue | Mots-clés |
|--------|-----------|
| Français | AVS / numéro AVS / No. AVS / N° AVS / assurance vieillesse / assurance sociale / numéro d'assurance |
| Allemand | AHV / AHV-Nummer / AHV-Nr / Altersvorsorge / Sozialversicherung / Versicherungsausweis / AHV-Ausweis |
| Italien | AVS / numero AVS / No. AVS / assicurazione vecchiaia / assicurazione sociale / numero d'assicurazione |
| Romanche | numer AVS / AVS / assicuranza da vegliadetgna / assicuranza sociala / document d'assicuranza / attest d'assicuranza |
| Anglais | AHV number / AVS number / Swiss social security / social security number CH / Swiss insurance number |

### C.2 : Medical-RH-Suisse

Niveau de confiance : Confiance moyenne | min : 2

| Langue | Mots-clés |
|--------|-----------|
| Français | arrêt maladie / certificat médical / incapacité de travail / congé maladie / inapte au travail / convalescence / accident de travail / maladie professionnelle |
| Allemand | Krankmeldung / ärztliches Zeugnis / Arbeitsunfähigkeit / Krankheitsfall / Arbeitsunfähigkeitszeugnis / Genesungsurlaub / Berufsunfall / Berufskrankheit / arbeitsunfähig |
| Italien | certificato medico / incapacità lavorativa / congedo malattia / inidoneità al lavoro / convalescenza / infortunio professionale / malattia professionale |
| Anglais | sick leave / medical certificate / incapacity to work / unfit for work / occupational accident / occupational disease / sick note / medical leave |
| Romanche | annunzia da malsogna / certificat medical / incapacità da lavurar / congedi da malsogna / nun abel da lavurar / convalescenza / accident da lavur / malsogna professiunala |

---

## Annexe D : Différences POC vs Production

| Point | En production |
|-------|---------------|
| Compte admin dans le chiffrement | Supprimez `admin@[domaine]` des autorisations de chiffrement. Gérez l'accès via le rôle Super User RMS uniquement. |
| DLP mode simulation | Après 2 à 4 semaines : modifiez la politique → Mode → **Activer immédiatement**. Vérifiez l'Activity Explorer avant. |
| Noms des groupes | Remplacez par les groupes réels du client. Créez-les mail-enabled via Exchange Online. |
| Registre des activités | Complétez l'Annexe H avec les activités réelles du client. Supprimez les références à Axonix SA et demo.ch. |
| Auto-labelling service | Vérifiez les résultats de simulation après 48h avant d'activer. |
| Rétention | Remplacez les URLs SharePoint par les URLs réelles des sites RH et Finances du client. |
| Break-glass RMS | Utilisez un compte cloud-only dédié (non nominatif). Stockez les identifiants dans un coffre-fort physique. |
| MFA | Préférez l'Accès conditionnel Entra pour permettre des exclusions ciblées. |

---

## Annexe E : Adaptations sectorielles

> **ℹ️ INFO**
> Appliquez uniquement les adaptations correspondant au secteur du client. Les étiquettes sectorielles `3 - ...` (sans chiffrement) remplacent `3 - Confidentiel-Externe` dans le secteur concerné.

### E.1 : PME générique (MVC tel quel)

> **✅ CONSEIL** : Aucune adaptation requise. Déployez le MVC tel quel.

### E.2 : Fiduciaire et comptabilité

Points de rupture : chiffrement RMS vers l'AFC et les banques (leurs destinataires n'ont pas toujours de compte Microsoft, et le code à usage unique ne fonctionne pas pour les documents), rétention des dossiers de mandants (ce ne sont pas des pièces comptables du cabinet ; une conservation trop large viole le principe de minimisation, art. 6 nLPD).

**Adaptation 1 : Étiquette 3 - Externe-Fiduciaire**

| Paramètre | Configuration |
|-----------|---------------|
| Nom | `3 - Externe-Fiduciaire` |
| Chiffrement RMS | Aucun |
| Marquage visuel | Filigrane : CONFIDENTIEL, USAGE EXTERNE AUTORISÉ \| En-tête : Transmission autorisée nLPD |
| Usage | Bilans, déclarations fiscales, décomptes TVA, documents bancaires |

**Adaptation 2 : Limiter la rétention aux pièces comptables du cabinet**

Les dossiers des mandants ne sont pas des pièces comptables du cabinet. Appliquez `Retention-Finances-10ans` au seul site qui contient la comptabilité du cabinet, pas aux sites des mandats. La durée de conservation des dossiers de mandats se définit avec le client et s'inscrit au registre (Annexe H).

**Option : SIT IDE/UID**

| Paramètre | Valeur |
|-----------|--------|
| Nom | `IDE-UID-Suisse` |
| Expression régulière | `\bCHE-\d{3}\.\d{3}\.\d{3}\b` |
| Niveau de confiance | Moyen, min : 1 |
| Mots-clés | IDE / UID / numéro IDE / numéro UID / Unternehmens-Identifikationsnummer / numero IDI |

### E.3 : Cabinet d'architectes et ingénieurs civils

> **⚠️ ATTENTION** : Ne pas appliquer la section 0.7 (blocage tenant SharePoint) pour ce secteur.

| N° | Étape | Action |
|----|-------|--------|
| 1 | Partage externe SharePoint | Curseur SharePoint → **Personnes spécifiques**. Curseur OneDrive → Personnes spécifiques. |
| 2 | Type de lien par défaut | Liens des fichiers et des dossiers → sélectionnez **Personnes spécifiques**. Permission par défaut : Consultation. |
| 3 | Paramètres supplémentaires | Paramètres de partage externe supplémentaires → cochez L'accès invité expirera automatiquement → **30 jours**. |
| 4 | Enregistrer | Cliquez sur Enregistrer. |
| 5 | Restreindre par site | Pour les sites RH et Finances : `admin.sharepoint.com` → Sites actifs → site → Paramètres → Partage → **Uniquement les personnes de votre organisation**. |

### E.4 : Étude d'avocats et notaires

**Adaptation : Étiquette 3 - Juridique-Externe (sans chiffrement RMS)**

| Paramètre | Configuration |
|-----------|---------------|
| Nom | `3 - Juridique-Externe` |
| Chiffrement RMS | Aucun |
| Marquage visuel | Filigrane : CONFIDENTIEL, TRANSMISSION AUTORISÉE \| En-tête : Document juridique, Destinataire unique |
| Usage | Pièces de procédure, actes notariés, correspondance inter-cabinets, transmissions aux greffes |

> **✅ CONSEIL** : Pour les transmissions via Justitia.swiss : utilisez le portail directement. Réservez l'étiquette `3 - Juridique-Externe` aux transmissions par email.

### E.5 : Agence de communication et marketing

**Adaptation 1 : Exclure les sites créatifs de l'auto-labelling service**

Dans `Auto-Label-Confidentiel` → Emplacements → Sites SharePoint → saisissez uniquement les URLs des sites RH et Finances. Excluez les sites créatifs (Projets, BAT, Assets).

**Adaptation 2 : Exclure les comptes créatifs de la DLP Exchange**

Dans `DLP-PME-Surveillance` → Emplacements → Courrier Exchange → Exclure → ajoutez les comptes des créatifs et chargés de projet.

> **⚠️ ATTENTION** : Compensez par une charte informatique signée interdisant l'envoi de données personnelles depuis les comptes créatifs. Documentez ce choix dans le registre des activités (Annexe H).

### E.6 : Cabinet médical et paramédical

**Adaptation : Étiquette 3 - Patient-Externe (sans chiffrement RMS)**

| Paramètre | Configuration |
|-----------|---------------|
| Nom | `3 - Patient-Externe` |
| Chiffrement RMS | Aucun |
| Marquage visuel | En-tête : Document médical personnel, Confidentiel \| Pied : Ne pas transmettre à des tiers |
| Usage | Résultats d'analyses, ordonnances, rappels de consultation, factures patients |

> **✅ CONSEIL** : Flux HIN : pour les échanges interprofessionnels (médecin → hôpital), utilisez HIN Mail directement. Le MVC Purview couvre les données internes et les communications patients uniquement.

### E.7 : Synthèse des adaptations

| Secteur | Blocage SharePoint | RMS externe | Auto-labelling | DLP | Complexité |
|---------|-------------------|-------------|----------------|-----|------------|
| PME générique | ✅ Bloquer | ✅ 3 - Confidentiel-Externe | ✅ Tous sites | ✅ Tous comptes | Aucune |
| Fiduciaire | ✅ Bloquer | ⚠️ Étiquette ext. | ✅ Tous sites | ✅ Tous comptes | Faible |
| Architectes | ⚠️ Partage contrôlé | ✅ 3 - Confidentiel-Externe | ✅ Tous sites | ✅ Tous comptes | Modérée |
| Avocats | ✅ Bloquer | ⚠️ Étiquette ext. | ✅ Tous sites | ✅ Tous comptes | Faible |
| Communication | ✅ Bloquer | ✅ 3 - Confidentiel-Externe | ⚠️ Sites ciblés | ⚠️ Comptes ciblés | Faible |
| Cabinet médical | ✅ Bloquer | ⚠️ Étiquette ext. | ✅ Tous sites | ✅ Tous comptes | Faible |

---

## Annexe F : Suivi nLPD dans le Gestionnaire de conformité (option)

> **🚨 IMPORTANT : Licence supplémentaire requise**
> La réglementation « Switzerland - Federal Act on Data Protection (FADP) » est un modèle **Premium** du Gestionnaire de conformité. Elle n'est **pas incluse** dans le MVC ni dans la licence Suite Microsoft Purview. Selon Microsoft, certains contrats de licence permettent d'utiliser trois réglementations Premium sans frais supplémentaires ; sinon, la réglementation doit être achetée. Avant de créer l'évaluation, vérifiez le compteur « Licences réglementaires achetées utilisées » en haut de la page Réglementations. Source : [Microsoft Learn, liste des réglementations du Gestionnaire de conformité](https://learn.microsoft.com/purview/compliance-manager-regulations-list).

Microsoft ne propose aucun modèle DLP propre à la Suisse (le filtre par pays des modèles DLP ne contient pas la Suisse). Le seul contenu suisse dédié est cette réglementation du Gestionnaire de conformité, qui sert de tableau de suivi des mesures nLPD.

| N° | Étape | Action |
|----|-------|--------|
| 1 | Navigation | Purview → Gestionnaire de conformité → Évaluations → icône **Ajouter une évaluation** (en haut à gauche de la liste). L'assistant « Créer une évaluation » compte 4 étapes : Réglementation, Nom et groupe, Services, Vérifier et finaliser. |
| 2 | Réglementation | Sélectionner une réglementation → le volet affiche les compteurs de licences (« Réglementations Microsoft activées », par exemple 1/3) → Rechercher : `Switzerland` → cochez **Switzerland - Federal Act on Data Protection (FADP)** → Enregistrer. Vérifiez : Disponibilité Premium, Activation Actif → Suivant |
| 3 | Nom et groupe | Nom de l'évaluation : `LPD [Nom du client]` (suggestion du portail : « Switzerland FADP Évaluation ») \| Groupe d'évaluations : **Utiliser un groupe existant** → Default Group → Suivant |
| 4 | Services | Cochez **Microsoft 365** (non coché par défaut) → Suivant |
| 5 | Vérifier et finaliser | Contrôlez le récapitulatif → **Créer une évaluation** |
| 6 | Suivi | Purview → Gestionnaire de conformité → Évaluations → ouvrez l'évaluation. La liste affiche la progression, vos actions d'amélioration et les actions gérées par Microsoft. |

> **ℹ️ INFO**
> Microsoft n'indique pas à quelle version de la loi (révision en vigueur depuis septembre 2023) correspond ce modèle. Utilisez l'évaluation comme aide au suivi, pas comme attestation de conformité devant le PFPDT.

---

## Annexe G : Fiche réflexe rançongiciel

À remplir avec le client, imprimer et remettre avec le guide d'exploitation (section 4.3), qui reprend la même fiche.

> **🚨 IMPORTANT : Le chiffrement Purview ne sert que s'il limite l'accès**
> Un rançongiciel moderne vole souvent les données avant de les chiffrer, puis menace de les publier. Un document étiqueté `2 - Confidentiel` (réservé aux personnes internes autorisées) reste illisible pour l'attaquant, sauf s'il utilise le compte d'un employé autorisé. Un document non étiqueté, ou étiqueté `3 - Confidentiel-Externe` (tout utilisateur authentifié), doit être considéré comme lisible. Le chiffrement Purview ne protège pas contre le chiffrement des fichiers par le rançongiciel : seules des sauvegardes intactes le permettent.

**Préparer avant l'incident** `[nLPD art. 8]`

| N° | Étape | Action |
|----|-------|--------|
| P1 | Comptes d'urgence | Comptes d'urgence Entra exclus de l'accès conditionnel et testés : voir le guide MVC Entra ID P1. Compte Break-glass RMS configuré (section 2.5 de la procédure). |
| P2 | Étiquette 2 | `2 - Confidentiel` sans « Ajouter tout utilisateur authentifié ». |
| P3 | Poste d'administration | Sur le poste qui servira en cas d'incident, ouvrez Windows PowerShell en administrateur et installez une fois les modules Graph, à la même version : `Install-Module Microsoft.Graph.Authentication, Microsoft.Graph.Users, Microsoft.Graph.Users.Actions -Scope AllUsers -Force`. Avec `-Scope CurrentUser`, l'accès contrôlé aux dossiers de Microsoft Defender peut bloquer l'installation lorsque les Documents sont redirigés vers OneDrive. Fermez puis rouvrez PowerShell, et testez la révocation sur un seul compte de test (voir le script ci-dessous). Refaites ce test lors de la révision annuelle. |
| P4 | Contacts | Remplissez le tableau des contacts ci-dessous et imprimez cette fiche : pendant l'incident, le réseau et les postes peuvent être inutilisables. |

**Pendant l'incident : premières heures** `[nLPD art. 8]`

| N° | Étape | Action |
|----|-------|--------|
| 1 | Isoler les postes | `security.microsoft.com` → Ressources → Appareils → cliquez sur l'appareil → « … » → **Isoler le dispositif**. Poste non géré par Defender : débranchez le câble réseau et coupez le Wi-Fi. N'éteignez pas les postes : les traces en mémoire servent à l'enquête. |
| 2 | Choisir le compte admin | Utilisez un compte administrateur sain, depuis un poste sain. Le compte d'urgence Entra ne sert que si les comptes administrateurs sont compromis ou bloqués. Chaque utilisation est documentée. |
| 3 | Comptes compromis | `entra.microsoft.com` → Utilisateurs → cliquez sur l'utilisateur → Vue d'ensemble : (1) carte **État du compte** → **Modifier** → désactivez le compte, (2) **Réinitialiser le mot de passe**, (3) **Révoquer les sessions**. Désactiver d'abord empêche l'attaquant d'obtenir de nouveaux jetons pendant l'opération. |
| 4 | Étendue inconnue | Révoquez les sessions de tous les utilisateurs (PowerShell ci-dessous). Les jetons d'accès déjà émis restent valides jusqu'à leur expiration, 1 heure par défaut selon Microsoft. Révoquer sans changer le mot de passe d'un compte compromis ne sert à rien : l'attaquant se reconnecte. |
| 5 | Portes dérobées | Avec votre consultant, contrôlez ce que l'attaquant a pu ajouter : méthode d'authentification (MFA) inconnue, règle de transfert dans une boîte mail, application autorisée, compte créé. Point de départ : `entra.microsoft.com` → Utilisateurs → Journaux d'audit. |
| 6 | Environnement hybride | Si un Active Directory local est synchronisé avec Entra, considérez-le comme compromis : faites appel à un spécialiste de réponse à incident. |
| 7 | Sauvegardes | Coupez l'accès des comptes qui peuvent atteindre les sauvegardes. Ne restaurez rien avant d'avoir identifié et fermé le point d'entrée. |

```powershell
# Dans une fenêtre Windows PowerShell neuve (modules installés à l'étape P3)
Connect-MgGraph -Scopes "User.ReadWrite.All" -UseDeviceCode
# Test sur un seul compte (préparation) : remplacez la ligne suivante par
# Revoke-MgUserSignInSession -UserId compte-test@[domaine]
Get-MgUser -All | ForEach-Object { Revoke-MgUserSignInSession -UserId $_.Id }
# Contrôle : la date doit correspondre au moment de la révocation (heure UTC)
Get-MgUser -UserId compte-test@[domaine] -Property SignInSessionsValidFromDateTime | Select-Object SignInSessionsValidFromDateTime
```

`Connect-MgGraph` affiche un code et une adresse : ouvrez l'adresse dans un navigateur, saisissez le code et connectez-vous avec un compte administrateur. Chaque révocation réussie renvoie `True`.

> **ℹ️ INFO : Dépannage**
> L'erreur « Impossible de charger le fichier ou l'assembly 'Microsoft.Graph.Authentication' » signale des modules Graph installés à des versions différentes. Relancez la commande d'installation de l'étape P3 (avec `-Force`), fermez PowerShell, rouvrez une fenêtre neuve et recommencez. Script testé le 25.11.2026 sous Windows PowerShell 5.1, sur un compte de test.

**Après la maîtrise de l'incident** `[nLPD art. 24]`

| N° | Étape | Action |
|----|-------|--------|
| 8 | Évaluer | Quelles données ont pu être lues ou volées ? Appliquez la règle de l'encadré ci-dessus : documents `2 - Confidentiel` illisibles si aucun compte autorisé n'est compromis, les autres considérés comme lisibles. |
| 9 | Annoncer au PFPDT | Si la violation entraîne vraisemblablement un risque élevé pour les personnes concernées : annonce au PFPDT dans les meilleurs délais (art. 24 nLPD), formulaire sur `www.edoeb.admin.ch`. |
| 10 | Signaler à l'OFCS | Signalement volontaire recommandé à l'Office fédéral de la cybersécurité : `https://www.report.ncsc.admin.ch/fr/`. Les exploitants d'infrastructures critiques ont une obligation de signaler dans les 24 heures. |
| 11 | Autres contacts | Assurance cyber (avant toute décision importante), police (dépôt de plainte). |
| 12 | Documenter | Chronologie, comptes et appareils touchés, données concernées, mesures prises. Ce document sert de preuve devant le PFPDT. |

| Contact | Coordonnées |
|---------|-------------|
| Consultant / réponse à incident | `[Prénom Nom]`, `[téléphone]` |
| Assurance cyber | `[Compagnie]`, `[n° de police]`, `[téléphone d'urgence]` |
| PFPDT | `www.edoeb.admin.ch` |
| OFCS | `https://www.report.ncsc.admin.ch/fr/` |

Sources : [Microsoft Learn, révoquer l'accès utilisateur dans Microsoft Entra ID](https://learn.microsoft.com/entra/identity/users/users-revoke-access) ; [OFCS, FAQ sur l'obligation de signaler](https://www.ncsc.admin.ch/ncsc/fr/home/meldepflicht/faq.html).

---

## Annexe H : Registre des activités de traitement (art. 12 nLPD)

> **ℹ️ INFO : Registre obligatoire ou volontaire**
> Les entreprises de moins de 250 collaborateurs sont dispensées de tenir un registre (OPDo art. 24), sauf si elles traitent des données sensibles à grande échelle ou font du profilage à risque élevé. Pour une PME de 5 à 25 personnes, le registre est donc le plus souvent volontaire. Il reste recommandé : il documente la démarche et sert de base à l'information des personnes concernées.

> **⚠️ ATTENTION : Exemple pré-rempli**
> Remplacez Axonix SA, demo.ch et admin@demo.ch par les données réelles du client.

**Informations générales de l'organisation**

| Champ | À compléter |
|-------|-------------|
| Raison sociale | `[Nom de la société]` |
| Responsable du traitement (art. 5 let. j nLPD) | `[Nom, Prénom, Fonction]` |
| DPO / Conseiller en protection des données | `[Nom, Prénom ou Externe / Pas de DPO désigné]` |
| Date de dernière mise à jour | `[JJ/MM/AAAA]` |

**Activité 1 : Gestion des ressources humaines et de la paie**

| Champ | Valeur : Axonix SA |
|-------|-------------------|
| Finalité | Administrer les contrats de travail, calculer et verser les salaires, gérer les absences. |
| Personnes concernées | Employés actifs et anciens employés. |
| Catégories de données | Numéro AVS, coordonnées, salaire, évaluations, données médicales (certificats), IBAN. |
| Destinataires | Fiduciaire, caisse AVS, assureur LAA, assureur maladie, Microsoft (sous-traitant). |
| Conservation | Par catégorie : décomptes de salaire, 10 ans (CO art. 958f) ; données utiles au certificat de travail, 10 ans après la fin des rapports de travail (CO art. 127) ; créances de salaire, vacances et heures supplémentaires, 5 ans après leur exigibilité (CO art. 128 ch. 3) ; dossiers de candidature, détruits au plus tard à la fin des rapports de travail (CO art. 328b) ; reste du dossier, nettoyé régulièrement (art. 6 al. 4 nLPD). Suppression manuelle lors de la révision annuelle. Limite : un fichier supprimé du site RH reste dans la bibliothèque de conservation jusqu'au terme des 10 ans de `Retention-RH-10ans`. |
| Base légale | Exécution du contrat de travail (art. 31 al. 2 let. a nLPD). |
| Mesures techniques | Étiquette `2 - Confidentiel` (AES-256, Azure RMS). `DLP-PME-Surveillance`, règle `Bloquer-Donnees-Sensibles` (justification obligatoire). `Retention-RH-10ans`. MFA actif. |

**Activité 2 : Comptabilité et gestion financière**

| Champ | Valeur : Axonix SA |
|-------|-------------------|
| Finalité | Tenir la comptabilité, gérer paiements fournisseurs et encaissements clients. |
| Personnes concernées | Fournisseurs, clients (contacts de facturation), employés (notes de frais). |
| Catégories de données | IBAN, coordonnées de facturation, montants, références de contrats. |
| Destinataires | Fiduciaire, banque, AFC, administrations fiscales cantonales, Microsoft (sous-traitant). |
| Conservation | 10 ans pour les livres et pièces comptables (CO art. 958f). |
| Base légale | Obligation légale (CO art. 957 ss). |
| Mesures techniques | Étiquette `1 - Interne` ou `2 - Confidentiel` selon le contenu. Règle DLP `Avertir-IBAN-Externe` (avertissement). `Retention-Finances-10ans`. MFA actif. |

**Activité 3 : Gestion de la relation client**

| Champ | Valeur : Axonix SA |
|-------|-------------------|
| Finalité | Exécuter les contrats de prestation, suivi commercial, facturation. |
| Personnes concernées | Contacts clients (personnes physiques, contexte B2B principalement). |
| Catégories de données | Nom, coordonnées professionnelles, historique des commandes et contrats. |
| Conservation | Durée du contrat + 10 ans : prescription générale des créances (CO art. 127) et conservation des pièces comptables comme les factures (CO art. 958f). Délai de 5 ans pour certaines créances (CO art. 128 : redevances périodiques, travail des artisans, ventes au détail, soins médicaux, services d'avocats et de notaires) : adaptez si l'activité du client en relève. Ne conservez au-delà du contrat que ce qui est nécessaire à ces fins (art. 6 al. 4 nLPD). |
| Base légale | Exécution du contrat (art. 31 al. 2 let. a nLPD). |
| Mesures techniques | Étiquette `1 - Interne`. Accès restreint par site SharePoint. `DLP-PME-Surveillance`. |

→ Ajoutez ici les activités supplémentaires spécifiques au client.

---

[← Retour au guide Purview](../) · [🏠 Retour au portail principal](https://doit4everyone.github.io/)

---

ℹ️ *Références, structuration et aide à la rédaction assistées par IA, avec validation humaine finale.*
