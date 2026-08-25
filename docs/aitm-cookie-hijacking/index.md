---
title: "AiTM / Cookie Hijacking M365 — Détection, mitigation et protection structurelle | DoIt4Everyone"
description: "Guide technique complet sur les attaques AiTM adversary-in-the-middle contre Microsoft 365. CAE, WHfB, FIDO2, Cloud Kerberos Trust, CA Authentication Strength. Validé terrain sur infra hybride — août 2026."
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
    font-size: 1.05em !important;
  }
  section {
    width: 100% !important;
    float: none !important;
    margin: 0 !important;
  }
  h1 { text-align: center; font-size: 2em; margin-bottom: 0.2em; }
  h2 { border-bottom: 2px solid #2E74B5; padding-bottom: 6px; color: #1F3864; margin-top: 2em; }
  h3 { color: #2E74B5; margin-top: 1.5em; }
  table { width: 100%; display: table; margin: 16px 0; border-collapse: collapse; font-size: 0.95em; }
  th { background: #1F3864; color: #fff; padding: 8px 10px; text-align: left; }
  td { padding: 7px 10px; border-bottom: 1px solid #dde4f0; }
  tr:nth-child(even) td { background: #EEF4FB; }
  .badge { display: inline-block; background: #1F3864; color: #fff; font-size: 0.78em; padding: 2px 8px; border-radius: 3px; margin-left: 6px; vertical-align: middle; }
  .callout { border-left: 4px solid #2E74B5; background: #D6E4F0; padding: 10px 14px; margin: 16px 0; border-radius: 0 4px 4px 0; }
  .callout-warn { border-left: 4px solid #C55A11; background: #FCE4D6; padding: 10px 14px; margin: 16px 0; border-radius: 0 4px 4px 0; }
  .callout-ok { border-left: 4px solid #375623; background: #E2EFDA; padding: 10px 14px; margin: 16px 0; border-radius: 0 4px 4px 0; }
  .callout-red { border-left: 4px solid #C00000; background: #FDECEA; padding: 10px 14px; margin: 16px 0; border-radius: 0 4px 4px 0; }
  .callout-yellow { border-left: 4px solid #C55A11; background: #FFF2CC; padding: 10px 14px; margin: 16px 0; border-radius: 0 4px 4px 0; }
  .step { margin: 6px 0 6px 20px; }
  .step strong { color: #1F3864; }
  code { background: #f0f4f8; padding: 2px 5px; border-radius: 3px; font-size: 0.9em; font-family: "SFMono-Regular", Consolas, monospace; }
  pre { background: #f0f4f8; padding: 14px; border-radius: 4px; overflow-x: auto; font-size: 0.88em; line-height: 1.5; }
  .nav-links { text-align: center; margin: 30px 0 10px; font-size: 0.95em; }
  .toc { background: #f8f9fa; border: 1px solid #dee2e6; padding: 16px 20px; border-radius: 4px; margin: 20px 0; }
  .toc ul { margin: 6px 0; padding-left: 20px; }
  .toc li { margin: 4px 0; }
  .partie-banner { background: #1F3864; color: #fff; text-align: center; padding: 14px 20px; border-radius: 4px; margin: 30px 0 20px; }
  .partie-banner h2 { color: #fff; border: none; margin: 0 0 4px; font-size: 1.3em; }
  .partie-banner p { margin: 0; color: #CCDDEE; font-style: italic; font-size: 0.95em; }
  .validated { color: #375623; font-weight: bold; }
  .pending { color: #C55A11; font-weight: bold; }
</style>

<div class="nav-links">

[← Retour au bundle MVC nLPD](../)

</div>

# AiTM / Cookie Hijacking M365

<h2 style="text-align:center; border:none; color:#2E74B5; margin-top:0;">Détection · Mitigation · Protection structurelle</h2>

<p style="text-align:center; color:#555;">
Guide technique autonome · Microsoft 365 Business Premium · v1.0 · Août 2026<br>
Validé terrain sur infrastructure hybride (Entra Connect + AD on-prem + WS2025)
</p>

---

<div class="callout-warn">

**Contexte de publication**

Ce document a été rédigé suite à la divulgation publique de deux incidents majeurs en août 2026.

**Mirage2FA (groupe LinX Coders, 20 août) :** plateforme PhaaS exploitant une technique AiTM par proxy WebSocket contre Microsoft 365. 9 426 comptes ciblés, 4 561 vols de session confirmés.

**CVE-2026-69836 (Microsoft, 21 août) :** vulnérabilité RCE CVSS 10.0 dans Entra ID, exploitation confirmée avant patch, corrigée côté infrastructure Microsoft sans action requise côté client.

**Audience :** consultants IT / MSP et responsables IT internes accompagnant des PME sous M365 Business Premium.

Ce document est indépendant mais complémentaire au [Guide MVC Purview nLPD](https://doit4everyone.github.io/Configuration-Purview-PME-Suisse-nLPD/).

</div>

---

<div class="toc">

**Table des matières**

**Partie A — Plan B : actionnable immédiatement**
- [1. Contexte : le MFA ne suffit plus](#1-contexte)
- [2. CAE : Continuous Access Evaluation](#2-cae)
- [3. Indicateurs de compromission (IoC)](#3-ioc)
- [4. Réponse immédiate : alerte SIEM reçue](#4-reponse)

**Partie B — Plan A : protection structurelle**
- [5. Pourquoi WHfB / FIDO2 rend le rejeu AiTM impossible](#5-whfb)
- [6. Prérequis et audit préalable](#6-prerequis)
- [7. Déploiement : Cloud Kerberos Trust et AzureADKerberos](#7-deploiement)
- [8. Rotation de la clé AzureADKerberos](#8-rotation)
- [9. Annexe A : FIDO2 hardware pour comptes à privilèges](#9-fido2)
- [10. Conclusion](#10-conclusion)
- [11. Annexe B : pourquoi un SIEM est indispensable, même gratuit](#11-siem)
- [12. Renvois et ressources](#12-renvois)

</div>

---

<div class="partie-banner">
<h2>PARTIE A · Plan B : actionnable immédiatement</h2>
<p>CAE · Détection des IoC · SIEM · Réponse immédiate</p>
</div>

## 1. Contexte : le MFA ne suffit plus {#1-contexte}

### 1.1 Qu'est-ce qu'une attaque AiTM ?

Une attaque AiTM (Adversary-in-the-Middle) moderne ne cherche plus à casser le MFA. Elle le laisse fonctionner normalement et intercepte ce qui vient après : le cookie de session authentifié.

Le mécanisme de Mirage2FA illustre parfaitement cette approche :

<div class="step"><strong>1 · Hameçonnage</strong> — L'utilisateur reçoit un lien pointant vers un proxy WebSocket transparent, visuellement identique à login.microsoft.com.</div>
<div class="step"><strong>2 · Proxy transparent</strong> — Le proxy relaie chaque requête vers Microsoft en temps réel. L'utilisateur voit la vraie page Microsoft et complète son MFA normalement.</div>
<div class="step"><strong>3 · Interception</strong> — Le proxy capture le cookie de session authentifié. L'attaquant dispose d'un cookie valide.</div>
<div class="step"><strong>4 · Rejeu</strong> — L'attaquant importe le cookie dans son navigateur et accède à M365 sans credentials ni MFA. Le reset du mot de passe ne suffit pas : le cookie reste valide jusqu'à révocation explicite.</div>

<div class="callout-red">

**Mirage2FA : chiffres de l'incident (20 août 2026)**

- 9 426 comptes Microsoft 365 ciblés
- 4 561 vols de session confirmés (~48 % de taux de succès)
- Vecteur : proxy WebSocket, indétectable par l'utilisateur
- Contre-mesure insuffisante : reset du mot de passe seul

</div>

### 1.2 CVE-2026-69836 : quand la vulnérabilité est dans l'infrastructure elle-même

Le 21 août 2026, Microsoft a divulgué CVE-2026-69836, une vulnérabilité de désérialisation de données non fiables dans Entra ID avec un score CVSS de 10.0 et une exploitation confirmée avant le patch. La correction a été déployée côté infrastructure Microsoft : aucune action patch requise côté client.

La question critique n'est pas "est-ce corrigé ?" mais "est-ce que quelque chose s'est passé dans mon tenant avant le fix ?"

<div class="callout-yellow">

**Audit rétrospectif recommandé — période précédant le 21 août 2026**

- Nouveaux role assignments Global Admin / Privileged Role Admin
- Credentials ajoutés sur des service principals existants
- Modifications de Conditional Access policies
- Consentements OAuth inhabituels
- Nouvelles méthodes MFA enregistrées sur des comptes à privilèges
- Comptes cloud créés hors du processus de synchronisation

**Limite :** si la rétention des logs est à 30 jours (défaut Business Premium sans forwarding externe), les événements antérieurs au 22 juillet 2026 ne sont plus accessibles. → Recommandation permanente : forwarder les logs Entra ID vers Log Analytics.

</div>

### 1.3 Pourquoi le MFA classique ne suffit pas structurellement

Le MFA protège l'authentification mais pas la session qui en résulte. Une fois le cookie de session émis, il représente une preuve d'identité valide aux yeux de Microsoft 365, indépendamment de la méthode d'authentification utilisée pour l'obtenir.

| Mesure | Efficacité contre AiTM | Remarque |
|---|---|---|
| MFA (TOTP / SMS) | Insuffisante | Le proxy relaie le MFA en temps réel |
| MFA (Authenticator push) | Insuffisante | L'utilisateur approuve, le proxy intercepte |
| Reset du mot de passe | Insuffisante | Le cookie reste valide après reset |
| CAE (Continuous Access Evaluation) | Partielle | Réduit la fenêtre à quelques minutes après révocation |
| Identity Protection (Impossible Travel) | Partielle | Détecte si l'IP de rejeu est géographiquement incohérente |
| FIDO2 / WHfB (passkey TPM) | Structurelle | Le rejeu est cryptographiquement impossible — voir Partie B |

---

## 2. CAE : Continuous Access Evaluation {#2-cae}

### 2.1 Principe, durée de vie des tokens et nuances

CAE modifie deux aspects du cycle de vie des tokens : leur durée de vie et leur révocabilité.

| Type de token | Durée de vie | Révocabilité | Scénario AiTM |
|---|---|---|---|
| Token standard (client non CAE-capable) | 60 à 90 minutes | Révocation manuelle uniquement | Fenêtre d'exploitation ~1h sans détection |
| Token CAE (client ET ressource CAE-capables) | 20 à 28 heures | Révocable en temps réel via signal serveur | Fenêtre plus longue si non détecté, mais révocable quasi-instantanément si détecté |

<div class="callout-warn">

**Paradoxe CAE vu du côté attaquant**

Un token CAE dure 20 à 28 heures au lieu de 60 à 90 minutes. Pour un attaquant qui vole un token CAE via AiTM, la fenêtre d'exploitation potentielle est donc beaucoup plus longue qu'avec un token standard, si personne ne détecte l'incident.

Le modèle CAE tient seulement si la détection et la révocation sont rapides. Sans SIEM ni surveillance active, CAE peut être un avantage pour l'attaquant.

C'est pour cette raison que la détection (SIEM + Identity Protection) est critique : elle conditionne directement l'efficacité de CAE comme contre-mesure.

</div>

Applications M365 supportant CAE côté serveur : Exchange Online, SharePoint Online, OneDrive, Microsoft Teams, MS Graph. Côté client, CAE nécessite un client compatible — validé en lab avec Microsoft Edge sur SharePoint Online. Firefox et l'application web Outlook (outlook.office.com) n'ont pas montré de token CAE dans nos tests (token standard émis).

<div class="callout-ok">

**CAE : actif par défaut — révocation validée en live (23 août 2026)**

CAE est inclus dans Entra ID P1 et actif par défaut : aucune configuration requise pour l'activer. L'option visible dans Accès conditionnel (Personnaliser l'évaluation de l'accès continu) sert uniquement à le désactiver si nécessaire.

- Validé sur tenant Business Premium : révocation session active en ~1 minute (myaccount.microsoft.com)
- ~3 à 4 minutes sur SharePoint Online via Edge (token CAE confirmé Is CAE Token = Oui)
- La révocation fonctionne sur tous les types de tokens (CAE et standard) via "Révoquer les sessions" dans Entra ID

</div>

<div class="callout">

**Identity Protection : différence P1 vs P2 (source : Microsoft learn.microsoft.com, juin 2026)**

**Business Premium sans add-on (P1) :**
- Visibilité partielle Identity Protection : utilisateurs à risque medium/high visibles, sans détails
- Pas de risk-based Conditional Access (révocation automatique sur détection)
- Révocation manuelle uniquement après détection via SIEM ou surveillance manuelle

**Business Premium + Defender Suite add-on (P2) :**
- Identity Protection complet : Impossible Travel, Token anomaly, Unfamiliar sign-in
- Risk-based Conditional Access : révocation automatique sur détection de risque

Dans les deux cas : la révocation manuelle fonctionne immédiatement via "Révoquer les sessions".

</div>

### 2.2 Vérifier que CAE n'est pas désactivé sur le tenant

**Via PowerShell :**

```powershell
Connect-MgGraph -Scopes "Policy.Read.All" -UseDeviceAuthentication

$result = Invoke-MgGraphRequest -Method GET `
  -Uri "https://graph.microsoft.com/beta/identity/conditionalAccess/policies"
$result.value | Where-Object {
  $_.sessionControls.continuousAccessEvaluation -ne $null
} | Select-Object displayName, @{N='CAE';E={$_.sessionControls.continuousAccessEvaluation.mode}}
```

**Résultat attendu :** aucune ligne retournée = CAE actif par défaut, non désactivé. ✅

Si l'API `/beta/identity/continuousAccessEvaluationPolicy` retourne 404 : c'est normal. Microsoft ne crée cet objet que si CAE a été explicitement personnalisé ou désactivé.

**Via le portail :**

Portail : `entra.microsoft.com` → Accès conditionnel → Stratégies

Vérifier qu'aucune policy existante ne contient Session → Personnaliser l'évaluation de l'accès continu → Désactiver.

<div class="callout">

**Ce que vous voyez dans l'interface**

Dans Accès conditionnel → Nouvelle stratégie → Session, l'option "Personnaliser l'évaluation de l'accès continu" propose deux sous-options uniquement :
- Désactiver
- Appliquer strictement les stratégies de localisation (préversion)

Il n'existe pas d'option "Activer" : CAE est actif par défaut sans aucune action.

</div>

### 2.3 Test de révocation

<div class="step"><strong>1 · Ouvrir une session test</strong> — Connecter un compte test sur myaccount.microsoft.com dans une fenêtre de navigation privée.</div>
<div class="step"><strong>2 · Révoquer les sessions</strong> — entra.microsoft.com → Utilisateurs → [compte test] → Vue d'ensemble → Révoquer les sessions.</div>
<div class="step"><strong>3 · Confirmer</strong> — Cliquer Oui dans la boîte "Voulez-vous révoquer toutes les sessions de l'utilisateur ?"</div>
<div class="step"><strong>4 · Chronométrer</strong> — Mesurer le délai avant invalidation de la session myaccount.microsoft.com.</div>

**Résultat validé en live (23 août 2026) :** session invalidée en ~1 minute. Toast de confirmation : "Sessions de connexion révoquées pour [compte]".

Pour un test CAE natif : utiliser SharePoint Online via Edge — token CAE confirmé (`Is CAE Token = Oui` dans les logs de connexion Entra ID). La révocation se produit au prochain appel réseau du client (refresh de page).

---

## 3. Indicateurs de compromission (IoC) {#3-ioc}

### 3.1 Signaux dans Identity Protection (Entra ID P1)

Portail : `entra.microsoft.com` → Identity Protection → Détections de risques

| Signal | Description | Pertinence |
|---|---|---|
| Impossible Travel | Connexion depuis deux géographies incompatibles en quelques minutes | Partielle : si l'attaquant rejette depuis une IP géographiquement incohérente |
| Unfamiliar sign-in properties | Nouvelle IP, nouveau user-agent, nouveau pays | Bonne : l'infra de l'attaquant diffère du profil habituel |
| Token anomaly | Token dont les propriétés ne correspondent pas à l'appareil enregistré | Directe : signal le plus spécifique pour un cookie rejoué |
| Anomalous Token | Durée de vie ou propriétés du token inhabituelles | Complémentaire |
| Malicious IP address | Connexion depuis une IP connue malveillante | Dépend de la fraîcheur du feed TI Microsoft |

### 3.2 Signaux comportementaux post-compromission

Ces signaux apparaissent dans les 30 premières minutes suivant un vol de session réussi. À surveiller dans `security.microsoft.com` → Incidents & alertes.

- **Règles de redirection mail :** création de règles Outlook redirigeant vers une adresse externe
- **Téléchargements massifs SharePoint :** volume de téléchargement anormalement élevé sur une courte période
- **Ajout de délégation mailbox :** délégation Full Access ou Send As ajoutée sur une boîte aux lettres
- **Nouvelle méthode MFA enregistrée :** l'attaquant tente d'établir une persistance
- **Consentement OAuth inhabituel :** application tierce obtenant des permissions sur le tenant
- **User-agent inhabituel :** l'infrastructure AiTM utilise souvent un UA générique ou légèrement malformé

### 3.3 Signaux on-prem (environnements hybrides)

- **Event ID 4624 LogonType 3 :** connexion réseau depuis une IP non reconnue après une auth cloud réussie
- **Event ID 4648 :** connexion explicite avec credentials alternatifs, indicateur de mouvement latéral

### 3.4 Corrélation SIEM : ce que vous voyez selon votre configuration UTMStack

Cette section croise les IoC des deux vecteurs (AiTM / CVE-2026-69836) avec les règles de corrélation UTMStack documentées dans le [Chapitre 11 du lab UTMStack](https://doit4everyone.github.io/utmstack-lab/docs/11-correlations-yaml.html).

**Bloc 1 — Règles opérationnelles sans configuration supplémentaire**

| Règle UTMStack | Signal couvert | Vecteur |
|---|---|---|
| M2 : OAuth Application Consent Granted | Consentement OAuth suspect post-compromission | AiTM + CVE-2026-69836 |
| M3 : Impossible Travel | Connexion depuis pays géographiquement impossible | AiTM |
| M5 : MFA Required Interrupt | Interrupt MFA visible même si proxy a relayé le token | AiTM |
| M6 : Member Added to Group | Ajout à un groupe Entra ID | CVE-2026-69836 |
| M7 : User Account Created Outside Sync | Compte cloud créé hors Entra Connect | CVE-2026-69836 |
| M8 : Unified Audit Log Disabled | Tentative d'aveugler le SIEM post-compromission | AiTM + CVE-2026-69836 |
| M12 : External Mailbox Access | Accès externe à une boîte Exchange via OAuth | AiTM |
| M14 : Sign-In Blocked by CA | Connexion bloquée par Conditional Access | AiTM |

**Bloc 2 — Deux règles complémentaires pour couvrir CVE-2026-69836**

| Règle | Action Entra ID détectée | Validation |
|---|---|---|
| M-new-1 : Privileged Role Assigned | `Add member to role.` → Global Admin, Security Admin, etc. | ✅ Validée en live : 22 août 2026 (latence ~7 min) |
| M-new-2 : Credentials Added to Service Principal | `Add service principal credentials.` → persistance sans MFA | ✅ Validée en live : 22 août 2026 (latence ~4 min) |

<div class="callout">

**M-new-1 et M-new-2 : champs disponibles dans l'alerte UTMStack**

**M-new-1 :** `action = "Add member to role."` · `log.ObjectId` = UPN du compte assigné · `log.ModifiedProperties` : Role.DisplayName · `origin.user` = acteur · latence ~7 min

**M-new-2 :** `action = "Add service principal credentials."` · `log.Target` = nom du service principal · `log.ModifiedProperties` : KeyDescription avec KeyIdentifier · `origin.user` = acteur · latence ~4 min

**Note :** M-new-2 se déclenche uniquement via appel API Graph direct (`addPassword`) ou PowerShell. L'interface graphique Entra ID génère un libellé différent et ne déclenche pas cette règle. Test de déclenchement : `POST https://graph.microsoft.com/v1.0/servicePrincipals/{id}/addPassword` via [Graph Explorer](https://developer.microsoft.com/en-us/graph/graph-explorer).

</div>

**Bloc 3 — Limites de visibilité**

<div class="callout-red">

**Angles morts structurels : indépendants de tout outil ou licence**

- **CVE-2026-69836 phase initiale :** l'exploitation s'est produite dans l'infrastructure Microsoft. Aucun event n'est généré dans les journaux client. Seules les actions consécutives dans le tenant sont visibles, c'est ce que couvrent M-new-1 et M-new-2.
- **Rétention des logs :** défaut Business Premium = 30 jours. Sans forwarding vers Log Analytics, les événements antérieurs au 22 juillet 2026 ne sont plus accessibles. → Recommandation permanente : forwarder les logs Entra ID vers Log Analytics.

</div>

<div class="callout-yellow">

**Angle mort UTMStack : limitation technique du pipeline ETL v11**

Exfiltration SharePoint : les events FileDownloaded (RecordType 6) ne sont pas collectés. La règle M13 est documentée comme non implémentable (voir Chapitre 11).

Couverture partielle disponible via Microsoft Purview Insider Risk Management (stratégie IRM-Fuites-Données-PME). Nécessite l'add-on Purview Suite, hors périmètre Business Premium seul.

Prérequis légal obligatoire avant activation IRM (art. 26 OLT3 + art. 6 nLPD) :
- Information préalable des employés dans le règlement du personnel ou la charte IT
- Anonymisation activée par défaut dans Purview
- Procédure écrite de levée d'anonymat (voir Annexe G du guide Purview complet)

Référence : [Guide complet Purview 2026, section 8.2](https://doit4everyone.github.io/microsoft-purview-configuration-2026-nLPD/docs/09-fonctionnalites-avancees.html)

</div>

---

## 4. Réponse immédiate : alerte SIEM reçue {#4-reponse}

<div class="callout-red">

**Point critique : si le SIEM remonte un IoC AiTM, c'est déjà trop tard pour prévenir le vol**

Le cookie de session est déjà entre les mains de l'attaquant au moment où l'alerte arrive. L'objectif devient de limiter la durée d'exploitation, pas de prévenir le vol.

- **Sans CAE :** après détection (~7 min de latence SIEM), l'attaquant dispose encore de ~53 minutes de cookie valide même après révocation immédiate.
- **Avec CAE :** la révocation prend effet en quelques minutes. La fenêtre tombe à moins de 10 minutes.

CAE doit être actif AVANT l'incident pour que la réponse soit efficace. Voir section 2.

</div>

### 4.1 Séquence des signaux à surveiller dans le SIEM

| Phase | Délai typique | Signaux SIEM (règles UTMStack) | Interprétation |
|---|---|---|---|
| Phase 1 : accès initial | T+0 à T+5 min | M3 (Impossible Travel), M5 (MFA Interrupt) | Le cookie vient d'être volé et rejoué. Agir immédiatement. |
| Phase 2 : exploitation | T+5 à T+30 min | M2 (OAuth Consent), M12 (External Mailbox Access) | L'attaquant établit sa persistance et commence l'exfiltration. |
| Phase 3 : persistance longue durée | T+30 min à plusieurs heures | M-new-1 (Role Assigned), M-new-2 (Credentials SPN), M7 (Account Created) | L'attaquant prépare un accès durable, indépendant du cookie. |

<div class="callout-red">

**Règle d'or**

Si vous voyez Phase 2 ou Phase 3 sans avoir vu Phase 1 dans le SIEM, l'attaque initiale a eu lieu en dehors de votre fenêtre de détection. La révocation s'impose immédiatement et les dégâts de Phase 2 sont peut-être déjà faits. Lancer l'audit complet de la section 4.3.

</div>

### 4.2 Révocation immédiate des tokens : à faire dans les 2 minutes

<div class="step"><strong>1 · Révoquer toutes les sessions actives</strong> — entra.microsoft.com → Utilisateurs → [compte compromis] → Vue d'ensemble → Révoquer les sessions → Oui</div>
<div class="step"><strong>2 · Révoquer les refresh tokens</strong> — PowerShell : <code>Revoke-MgUserSignInSession -UserId [UPN]</code></div>
<div class="step"><strong>3 · Vérifier la propagation CAE</strong> — Si CAE est actif : tester une action SharePoint / Teams sur le compte concerné, la session doit être invalidée en moins de 5 minutes.</div>

<div class="callout-yellow">

**Sans CAE : que faire pendant les ~53 minutes restantes ?**

Alerter immédiatement l'utilisateur concerné de ne plus rien faire sur sa session. Surveiller en temps réel les actions dans Exchange (redirection mail), SharePoint (téléchargements), Teams (messages envoyés) et Entra ID (modifications de configuration). Documenter chaque action observée avec horodatage pour constituer le dossier d'incident.

</div>

### 4.3 Audit des dégâts : à faire dans les 30 minutes

**Logs Entra ID — journaux d'audit**

Portail : `entra.microsoft.com` → Journaux d'audit → filtrer sur [date/heure de l'incident] → [compte compromis]

- `Set-InboxRule` ou `New-InboxRule` : règles de redirection mail créées
- `Add-MailboxPermission` : délégations mailbox ajoutées
- `Consent to application` : applications OAuth consenties
- `Update user` sur les méthodes d'authentification : méthodes MFA modifiées
- `Add member to role` : role assignments
- `Add service principal credentials` : credentials SPN ajoutés

**Logs Entra ID — journaux de connexion**

Portail : `entra.microsoft.com` → Journaux de connexion → filtrer sur [compte compromis] → période de l'incident

- IP source inhabituelle, user-agent générique ou malformé, géolocalisation incohérente

### 4.4 Audit rétrospectif post-CVE-2026-69836

Si l'audit révèle des actions suspectes dans la fenêtre précédant le 21 août 2026, appliquer la séquence de révocation sur les comptes concernés, puis vérifier spécifiquement :

- `entra.microsoft.com` → Rôles et administrateurs → vérifier chaque rôle Global Admin / Security Admin / Privileged Role Admin
- `entra.microsoft.com` → Applications d'entreprise → Certificats et secrets → secrets récemment ajoutés
- `entra.microsoft.com` → Accès conditionnel → vérifier les modifications récentes de policies

<div class="callout" style="border-color:#4B0082; background:#EDE7F6;">

**Constitution de preuves pour notification PFPDT (art. 24 nLPD)**

Si l'audit révèle une compromission effective, eDiscovery Premium permet de rechercher toutes les données potentiellement exposées dans Exchange, SharePoint et Teams et de constituer le dossier de preuve pour la notification au PFPDT.

Nécessite l'add-on Purview Suite, hors périmètre Business Premium seul.
Référence : [Guide complet Purview 2026, section 8.3](https://doit4everyone.github.io/microsoft-purview-configuration-2026-nLPD/docs/09-fonctionnalites-avancees.html)

</div>

### 4.5 Séquence de remédiation complète

<div class="callout-red">

**Erreur fréquente : réinitialiser le mot de passe en premier**

Le cookie de session reste valide après un reset de mot de passe. La séquence ci-dessous doit être exécutée dans l'ordre indiqué.

</div>

<div class="step"><strong>1</strong> · Révoquer toutes les sessions actives</div>
<div class="step"><strong>2</strong> · Révoquer les refresh tokens (PowerShell)</div>
<div class="step"><strong>3</strong> · Auditer les règles de redirection mail (Exchange Admin Center)</div>
<div class="step"><strong>4</strong> · Auditer les délégations mailbox (Full Access / Send As)</div>
<div class="step"><strong>5</strong> · Auditer les applications OAuth consenties récemment</div>
<div class="step"><strong>6</strong> · Auditer les méthodes MFA enregistrées — supprimer tout dispositif non reconnu</div>
<div class="step"><strong>7</strong> · Réinitialiser le mot de passe — uniquement après les étapes 1 à 6</div>
<div class="step"><strong>8</strong> · Forcer une re-registration MFA complète</div>
<div class="step"><strong>9</strong> · Documenter l'incident — horodatage, IP source, user-agent, actions détectées</div>

---

<div class="partie-banner">
<h2>PARTIE B · Plan A : protection structurelle</h2>
<p>WHfB par TPM · Cloud Kerberos Trust · Applications legacy · Rotation AzureADKerberos</p>
</div>

<div class="callout">

**Note de portée**

Cette partie couvre des mesures de durcissement avancées, au-delà du périmètre MVC Business Premium. Audience : consultants IT accompagnant des PME avec postes Hybrid Entra Join ou full cloud. Validation terrain effectuée sur un environnement hybride : Entra Connect + AD on-prem + WS2025.

</div>

## 5. Pourquoi WHfB / FIDO2 rend le rejeu AiTM impossible {#5-whfb}

### 5.1 Le mécanisme cryptographique

Windows Hello for Business utilise une paire de clés asymétriques générée à l'enrôlement :

- **Clé privée :** stockée dans le TPM, non extractible, ne quitte jamais le poste
- **Clé publique :** publiée dans Entra ID à l'enrôlement

À chaque authentification, Entra ID envoie un challenge signé qui inclut l'origine du site (login.microsoft.com). Le TPM signe {challenge + origine} avec la clé privée. La réponse est vérifiée par Entra ID avec la clé publique.

<div class="callout-ok">

**Pourquoi le rejeu AiTM est impossible avec WHfB / FIDO2**

Le proxy AiTM a une origine différente de login.microsoft.com. Le TPM signe {challenge + origine\_proxy} : la signature est cryptographiquement invalide. Entra ID rejette l'authentification. L'attaquant ne peut pas rejouer, même avec le cookie. Cette protection est structurelle : elle ne dépend pas de la détection ou de la révocation.

</div>

### 5.2 Comparaison des scénarios de déploiement

| Type de poste | Type de compte | Solution | AzureADKerberos requis ? | Difficulté |
|---|---|---|---|---|
| Full cloud (Entra ID seul) | Cloud-only ou hybride | FIDO2 / WHfB direct | Non | Faible |
| Hybrid Entra Join | Hybride (synchronisé AD) | WHfB + Cloud Kerberos Trust | Oui | Moyenne |
| Hybrid Entra Join | Cloud-only | Non compatible | N/A | Impossible : pas de session Windows possible |
| AD pur (pas de Entra Join) | AD on-prem uniquement | Hors scope WHfB | Non applicable | N/A |

<div class="callout-yellow">

**Point critique : compte cloud-only sur poste hybride**

Un compte cloud-only (créé directement dans Entra ID, non synchronisé depuis l'AD) ne peut pas ouvrir de session Windows sur un poste Hybrid Entra Join. Windows tente d'authentifier via Kerberos contre l'AD on-prem en premier — le compte n'existe pas dans l'AD, la connexion échoue.

Validé en lab (24 août 2026) : test-m7 (cloud-only) refusé sur poste hybride. Solution : utiliser un compte hybride (synchronisé via Entra Connect) sur les postes hybrides. Les postes full cloud acceptent les deux types de comptes sans restriction.

</div>

---

## 6. Prérequis et audit préalable {#6-prerequis}

### 6.1 Prérequis techniques

| Prérequis | Détail | Statut à vérifier |
|---|---|---|
| Entra ID P1 | Inclus dans M365 Business Premium | ✅ Dans le périmètre MVC |
| Windows 10 21H2+ / Windows 11 | Sur les postes ciblés par WHfB | 📋 Inventaire à faire |
| TPM 2.0 | Puce TPM active dans le BIOS/UEFI (firmware ou discret) | 📋 Gestionnaire de périphériques → Modules de plateforme sécurisée |
| Hybrid Entra Join | Entra Connect configuré sur les postes hybrides | 📋 Entra ID → Appareils → confirmer le statut |
| Comptes hybrides | Les comptes cloud-only ne peuvent pas ouvrir de session Windows sur un poste hybride | 📋 Vérifier la source des comptes utilisateurs |
| DC Windows Server 2016+ | Pour Cloud Kerberos Trust en hybride | 📋 Vérifier la version des DC |
| Droits Global Admin + Domain Admin | Pour la création de l'objet AzureADKerberos | 📋 Disponibles le temps de l'opération |
| PowerShell 5.1 | Le module AzureADHybridAuthenticationManagement ne fonctionne pas en PowerShell 7 | 📋 Vérifier via `$PSVersionTable.PSVersion` |

<div class="callout">

**Note : TPM firmware vs discret pour postes partagés**

TPM firmware (Intel PTT / AMD fTPM, intégré au CPU) : 7 à 8 slots de clés WHfB. Adapté aux postes mono-utilisateur (cas le plus fréquent en PME 5-25 personnes).

TPM discret (puce physique séparée) : 32 slots minimum. Recommandé pour les postes partagés avec rotation d'utilisateurs.

Si le TPM est plein, Windows supprime automatiquement les clés les moins récemment utilisées. L'utilisateur concerné doit re-enrôler WHfB à sa prochaine connexion sur ce poste.

</div>

<div class="callout-yellow">

**Vérification préalable : GPO WHfB existantes en environnement hybride**

Les GPO Active Directory WHfB prennent le dessus sur les policies Intune sur les postes hybrides. Vérifier qu'aucune GPO WHfB n'est déjà en place avant le déploiement :

```powershell
Get-GPO -All | Where-Object {
  (Get-GPOReport -Guid $_.Id -ReportType XML) -match "PassportForWork"
}
```

Si des GPO WHfB existent : les aligner avec la configuration Intune souhaitée ou les supprimer pour laisser Intune gérer.

</div>

### 6.2 Audit des applications legacy : étape obligatoire

Avant tout déploiement WHfB, identifier les applications susceptibles de régresser. Un déploiement sans audit peut bloquer des utilisateurs sur des outils métier critiques.

**Méthode d'audit rapide :**
1. Inventaire des apps qui demandent des credentials Windows interactivement
2. Identification des flux NTLM : Event ID 4624 LogonType 3 + AuthPackage NTLM dans les logs DC
3. Liste des partages réseau / NAS accessibles via auth Windows
4. Inventaire des sessions RDP vers serveurs sans Remote Credential Guard
5. Vérification du Gestionnaire d'informations d'identification Windows sur les postes pilotes

### 6.3 Applications legacy : risques et mitigation

| Type d'application | Risque après WHfB | Mitigation |
|---|---|---|
| Apps demandant mot de passe Windows interactif | Blocage : l'utilisateur ne connaît plus son mot de passe | Migrer vers auth moderne ou maintenir un mot de passe de secours documenté |
| Applications forçant NTLM | Échec d'authentification ou demande de credentials | Configurer SPN et Kerberos sur le serveur cible |
| WIA mal configurée (SPN manquant) | Chute en NTLM, échec | Créer les SPN manquants, vérifier l'ordre de négociation |
| RDP vers serveurs legacy | Demande de mot de passe | Activer Remote Credential Guard ou Restricted Admin mode sur le serveur cible |
| Apps M365 utilisant protocoles hérités (IMAP, POP3, SMTP AUTH, Exchange ActiveSync legacy) | Blocage si les protocoles hérités ne sont pas déjà bloqués | Déployer une politique Conditional Access bloquant les protocoles hérités pour tous les utilisateurs |
| Credentials Manager Windows | Renouvellement impossible si mot de passe oublié | Documenter la procédure de reset avec assistance IT |

### 6.4 KDClocal : atténuation partielle en cours de déploiement

Microsoft déploie progressivement KDClocal, un mini-KDC Kerberos embarqué dans Windows qui émet des tickets localement sans nécessiter de contact avec un DC. Statut : preview / roadmap selon les tenants (août 2026).

Ce que KDClocal résout partiellement : sessions RDP vers serveurs compatibles Kerberos, scénarios de line-of-sight DC intermittente (télétravail, VPN instable), certains cas WIA avec SPN correct mais DC injoignable. Ce que KDClocal ne résout pas : applications forçant NTLM au niveau protocole, apps demandant un mot de passe interactif, délégation Kerberos cassée côté serveur.

---

## 7. Déploiement : Cloud Kerberos Trust et AzureADKerberos {#7-deploiement}

### 7.1 Création de l'objet AzureADKerberos

<div class="callout">

**Ce que fait cet objet**

AzureADKerberos est un objet krbtgt de type RODC virtuel créé dans l'AD et publié dans Entra ID. Il établit un secret partagé entre l'AD on-prem et Entra ID, permettant à Entra ID d'émettre des TGT partiels que les DC on-prem reconnaissent et échangent contre des TGT complets. Résultat : l'utilisateur WHfB accède aux ressources on-prem sans mot de passe, sans PKI.

</div>

<div class="step"><strong>1 · TLS 1.2</strong> — Activer TLS 1.2 pour l'accès à PowerShell Gallery :</div>

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12
```

<div class="step"><strong>2 · Module PowerShell</strong> — Installer le module (PowerShell 5.1 requis, ne fonctionne pas en PowerShell 7) :</div>

```powershell
Install-Module -Name AzureADHybridAuthenticationManagement -AllowClobber -Force
Import-Module AzureADHybridAuthenticationManagement
```

Note : si le message "Required VC++ 2013 x64 Runtimes do not exist" apparaît, le module installe automatiquement le runtime — laisser terminer.

<div class="step"><strong>3 · Variables</strong> — Préparer les credentials :</div>

```powershell
$domain = $env:USERDNSDOMAIN
$domainCred = Get-Credential -Message "Domain Admin credentials"
```

<div class="step"><strong>4 · Création</strong> — Créer l'objet avec authentification interactive Entra ID :</div>

```powershell
Set-AzureADKerberosServer -Domain $domain `
  -UserPrincipalName 'admin@[tenant].onmicrosoft.com' `
  -DomainCredential $domainCred
```

Une fenêtre de connexion Entra ID s'ouvre dans le browser — se connecter avec le compte Global Admin ou Hybrid Identity Admin. Pas de sortie = succès silencieux.

<div class="step"><strong>5 · Vérification</strong> — Contrôler que l'objet est bien créé et synchronisé :</div>

```powershell
Get-AzureADKerberosServer -Domain $domain `
  -UserPrincipalName 'admin@[tenant].onmicrosoft.com' `
  -DomainCredential $domainCred
```

<div class="callout-ok">

**Sortie attendue après création réussie (validée en lab, 24 août 2026)**

```
Id                 : [numéro unique]
UserAccount        : CN=krbtgt_AzureAD,CN=Users,DC=[domaine],DC=[ext]
ComputerAccount    : CN=AzureADKerberos,OU=Domain Controllers,DC=[domaine],DC=[ext]
DisplayName        : krbtgt_[numéro]
DomainDnsName      : [domaine.ext]
KeyVersion         : [numéro]
KeyUpdatedOn       : [date et heure]
KeyUpdatedFrom     : [nom du DC]
CloudKeyVersion    : [même numéro que KeyVersion]  ← synchronisation confirmée
CloudKeyUpdatedOn  : [même date que KeyUpdatedOn]  ← écart idéalement 0 seconde
CloudTrustDisplay  : [vide au premier déploiement — normal]
```

Points à vérifier : `KeyVersion = CloudKeyVersion` et écart `KeyUpdatedOn / CloudKeyUpdatedOn` inférieur à 1h.

</div>

### 7.2 Configuration WHfB dans Intune

La configuration WHfB dans Intune se fait en deux étapes distinctes : la policy tenant-wide pour les paramètres généraux WHfB, et un profil Catalogue de paramètres pour le Cloud Kerberos Trust.

<div class="callout-yellow">

**Note importante : ne pas utiliser le profil Protection de compte (Sécurité du point de terminaison)**

Le profil "Protection de compte" sous Sécurité du point de terminaison entre en conflit avec la policy tenant-wide WHfB existante sur tous les tenants. Utiliser uniquement les deux méthodes documentées ci-dessous.

</div>

**Étape A : configurer la policy tenant-wide WHfB**

Portail : `intune.microsoft.com` → Appareils → Inscription → Windows → Windows Hello Entreprise

Cette policy est présente sur tous les tenants mais en état "Non configuré" par défaut. La configurer avec les paramètres suivants :

| Paramètre | Valeur recommandée | Remarque |
|---|---|---|
| Configurer Windows Hello Entreprise | Activé | Active WHfB sur tous les appareils inscrits |
| Utiliser un module de plateforme sécurisée (TPM) | Préféré | Préféré = WHfB fonctionne aussi sans TPM. Passer à Obligatoire sur un parc homogène récent. |
| Longueur minimale du code PIN | 6 | Minimum recommandé |
| Longueur maximale du code PIN | 127 | Valeur par défaut |
| Autoriser l'authentification biométrique | Oui | Empreinte, reconnaissance faciale |
| Utilisez des clés de sécurité pour la connexion | Activé | Active FIDO2 en complément de WHfB |

**Étape B : créer le profil Cloud Kerberos Trust**

Portail : `intune.microsoft.com` → Appareils → Configuration → Créer → Nouvelle stratégie

<div class="step"><strong>1 · Plateforme</strong> — Windows 10 et ultérieur</div>
<div class="step"><strong>2 · Type de profil</strong> — Catalogue de paramètres (pas Modèles)</div>
<div class="step"><strong>3 · Nom</strong> — WHfB-Cloud-Kerberos-Trust</div>
<div class="step"><strong>4 · Ajouter des paramètres</strong> — Rechercher "Cloud Trust" → catégorie Windows Hello Entreprise</div>
<div class="step"><strong>5 · Paramètre</strong> — Utiliser l'approbation cloud pour l'authentification locale → Activé</div>
<div class="step"><strong>6 · Affectation</strong> — Groupe pilote initial (2 à 3 appareils hybrides non-critiques)</div>
<div class="step"><strong>7 · Créer</strong> — Vérifier + créer → Créer</div>

<div class="callout-ok">

**Comportement attendu au premier enrôlement WHfB sur poste hybride (validé en lab, 24 août 2026)**

1. Connexion Windows avec mot de passe → Windows détecte la policy WHfB
2. Authenticator ou méthode MFA enregistrée demandée pour valider l'identité
3. Assistant WHfB s'ouvre → création du PIN (6 caractères minimum)
4. Reboot possible selon le poste — normal lors de l'application initiale des policies
5. Reconnexion avec le PIN → une fenêtre de connexion Entra ID s'ouvre une seule fois pour créer le PRT (Primary Refresh Token) lié au poste
6. Connexions suivantes : PIN seul, plus de fenêtre de connexion Microsoft

Note : la policy tenant-wide Intune fonctionne sans GPO sur les postes hybrides. Si une GPO WHfB existe dans l'AD, elle prend le dessus sur Intune — vérifier section 6.1.

</div>

### 7.3 Vérification de l'état WHfB sur le poste

Après enrôlement, vérifier l'état complet du poste depuis une invite de commande administrateur :

```cmd
dsregcmd /status
```

<div class="callout-ok">

**Champs clés à vérifier dans la sortie (validés en lab, 24 août 2026)**

```
Device State :
  AzureAdJoined : YES  ← poste joint à Entra ID
  DomainJoined  : YES  ← poste joint à l'AD on-prem (hybride)
  TpmProtected  : YES  ← clé WHfB dans le TPM

User State :
  NgcSet    : YES      ← WHfB enrôlé pour cet utilisateur
  NgcKeyId  : {GUID}   ← identifiant de la clé WHfB dans le TPM

SSO State :
  AzureAdPrt : YES  ← PRT Entra ID actif (SSO M365)
  OnPremTgt  : YES  ← ticket Kerberos on-prem actif
  CloudTgt   : YES  ← Cloud Kerberos Trust actif ✅

Diagnostic :
  KeySignTest : PASSED  ← la clé WHfB signe correctement
```

</div>

<div class="callout-yellow">

**Dépannage : PreReqResult = WillNotProvision**

Si `NgcSet : NO` et `PreReqResult : WillNotProvision` apparaissent, vérifier deux points :

1. **Le compte a-t-il une méthode MFA dans le nouveau système unifié Entra ID ?**
   Vérifier via : `entra.microsoft.com` → Utilisateurs → [compte] → Méthodes d'authentification
   Les méthodes legacy (Authenticator enregistré via outlookMobile) peuvent ne pas suffire.
   Solution : enregistrer une méthode native via `aka.ms/mysecurityinfo` (Authenticator standard ou passkey).

2. **Le compte est-il soumis à une policy MFA active ?**
   Un compte exclu de toute policy MFA ne peut pas s'enrôler en WHfB.
   WHfB nécessite une validation d'identité forte lors de l'enrôlement.

</div>

### 7.4 Conditional Access : exiger un MFA résistant au phishing

Une fois WHfB et FIDO2 déployés, créer une Conditional Access policy qui exige exclusivement une méthode résistante au phishing — supprimant le fallback vers SMS ou TOTP.

<div class="callout-ok">

**Périmètre licence : Entra ID P1 — dans le périmètre Business Premium**

La fonctionnalité "Exiger la force de l'authentification" dans Conditional Access nécessite uniquement P1.
Source : [Microsoft learn.microsoft.com](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-strengths)

</div>

**Forces d'authentification disponibles dans l'interface (libellés validés terrain, 24 août 2026) :**

- **Authentification multifacteur :** combinaisons qui répondent à une auth forte (Mot de passe + SMS, etc.)
- **Authentification multifacteur sans mot de passe :** Microsoft Authenticator, etc.
- **MFA anti-hameçonnage** (libellé dans la liste : `Phishing-resistant MFA`) : méthodes sans mot de passe sans hameçonnage — FIDO2, WHfB, CBA uniquement

Note : la liste déroulante affiche "Phishing-resistant MFA" en anglais même en interface française.

<div class="callout-red">

**Prérequis obligatoire avant activation**

Chaque utilisateur soumis à cette policy doit avoir au moins une méthode résistante au phishing enregistrée (WHfB sur son poste ou passkey FIDO2). Sans cela, l'utilisateur sera bloqué.

Recommandation : au minimum deux méthodes enregistrées par utilisateur.

Activer d'abord en mode "Rapport uniquement" et vérifier les logs avant de passer en Activé. Minimum 2 semaines de monitoring en mode Rapport recommandé.

</div>

**Policy 1 : comptes à privilèges (priorité maximale)**

Portail : `entra.microsoft.com` → Accès conditionnel → Stratégies → Nouvelle stratégie

<div class="step"><strong>1 · Nom</strong> — CA-Phishing-Resistant-Admins</div>
<div class="step"><strong>2 · Utilisateurs</strong> — Rôles d'annuaire → Administrateur général, Administrateur de rôle privilégié, Administrateur de la sécurité (au minimum)</div>
<div class="step"><strong>3 · Ressources cibles</strong> — Toutes les ressources (anciennement « Toutes les applications cloud »)</div>
<div class="step"><strong>4 · Contrôles d'accès → Octroyer</strong> — Accorder l'accès → Exiger la force de l'authentification → Phishing-resistant MFA → Sélectionner</div>
<div class="step"><strong>5 · Activer</strong> — Rapport uniquement → Créer</div>

<div class="callout-yellow">

**Avertissement Microsoft lors de la création**

"Ne bloquez pas votre accès ! Cette stratégie affecte le Portail Azure." Ignorable en mode Rapport uniquement — le mode Rapport ne bloque jamais l'accès. À prendre en compte uniquement lors du passage en mode Activé.

</div>

**Policy 2 : tous les utilisateurs (déploiement progressif)**

À déployer uniquement après confirmation que tous les utilisateurs ont enrôlé WHfB ou une passkey.

<div class="step"><strong>1 · Nom</strong> — CA-Phishing-Resistant-AllUsers</div>
<div class="step"><strong>2 · Utilisateurs</strong> — Tous les utilisateurs — exclure le groupe break-glass</div>
<div class="step"><strong>3 · Ressources cibles</strong> — Toutes les ressources (ou applications critiques uniquement dans un premier temps)</div>
<div class="step"><strong>4 · Contrôles d'accès → Octroyer</strong> — Accorder l'accès → Exiger la force de l'authentification → Phishing-resistant MFA → Sélectionner</div>
<div class="step"><strong>5 · Activer</strong> — Rapport uniquement → minimum 2 semaines de monitoring → passer en Activé</div>

<div class="callout">

**Ce que fait la policy en mode Activé — comportement exact**

La policy ne force pas l'enrôlement WHfB automatiquement. Elle exige qu'une méthode résistante au phishing soit utilisée pour s'authentifier.

- Utilisateur avec WHfB enrôlé : connexion transparente via WHfB, policy satisfaite.
- Utilisateur avec passkey FIDO2 : connexion via passkey, policy satisfaite.
- Utilisateur sans méthode résistante au phishing : accès bloqué, redirigé vers `aka.ms/mysecurityinfo` pour enregistrer WHfB ou une passkey.

Les comptes anciens sans méthode résistante enregistrée seront bloqués. Prévoir une campagne d'enrôlement avant le passage en mode Activé.

</div>

<div class="callout-yellow">

**Audit préalable : identifier les comptes non conformes**

```powershell
Get-MgUser -All | ForEach-Object {
  $methods = Get-MgUserAuthenticationMethod -UserId $_.Id
  $resistant = $methods | Where-Object {
    $_.AdditionalProperties.'@odata.type' -match 'fido2|windowsHello|x509Certificate'
  }
  if (-not $resistant) {
    [PSCustomObject]@{ UPN = $_.UserPrincipalName; DisplayName = $_.DisplayName }
  }
}
```

Les utilisateurs retournés n'ont pas de méthode résistante au phishing et seront bloqués si la policy est activée sans campagne d'enrôlement préalable.

</div>

<div class="callout-yellow">

**Note : retrait SMS et voix prévu septembre 2026**

Microsoft a annoncé le retrait de l'authentification par SMS et par appel vocal à partir de septembre 2026. Si des utilisateurs utilisent encore ces méthodes, planifier la migration vers WHfB ou passkeys avant cette date.

</div>

---

## 8. Rotation de la clé AzureADKerberos {#8-rotation}

### 8.1 Pourquoi et à quelle fréquence

L'objet AzureADKerberos contient une clé krbtgt qui signe les TGT partiels. Si cette clé est compromise, un attaquant peut forger des TGT valides. La rotation régulière limite la fenêtre d'exploitation.

| Cadence | Contexte | Remarque |
|---|---|---|
| 30 jours | Recommandation Microsoft minimale | Ne pas dépasser |
| 15 jours | Environnements sensibles | Recommandé en contexte nLPD art. 8 |
| Immédiatement | Compromission avérée d'un compte Domain Admin | Procédure d'urgence, voir 8.3 |

### 8.2 Procédure de rotation standard

<div class="step"><strong>1 · Rotation</strong> — Exécuter la rotation de clé :</div>

```powershell
Set-AzureADKerberosServer -Domain $domain `
  -UserPrincipalName 'admin@[tenant].onmicrosoft.com' `
  -DomainCredential $domainCred -RotateServerKey
```

<div class="step"><strong>2 · Vérification immédiate</strong> — Contrôler la synchronisation : KeyLastRotated et CloudKeyLastSynced doivent être à moins de 1h d'écart.</div>
<div class="step"><strong>3 · Attente</strong> — Ne pas supprimer l'ancien objet : attendre minimum 10h (durée de vie d'un TGT Kerberos), idéalement 24h.</div>
<div class="step"><strong>4 · Validation</strong> — Tester une connexion WHfB sur un poste hybride pour confirmer que la rotation n'a pas cassé l'auth.</div>

<div class="callout">

**Coordination avec la rotation du compte krbtgt AD**

L'objet AzureADKerberos et le compte krbtgt de l'AD sont deux objets distincts avec des clés indépendantes : leurs rotations sont des opérations séparées.

Rotation krbtgt AD : bonne pratique classique, recommandée tous les 60 à 90 jours, ou immédiatement après tout incident impliquant un accès Domain Admin non autorisé. La rotation krbtgt se fait en deux passes espacées de la durée de vie maximale du ticket Kerberos (10h par défaut) pour éviter de bloquer les sessions actives.

Les deux rotations peuvent être planifiées le même jour mais ne doivent pas être exécutées simultanément. Ordre recommandé : krbtgt AD en premier (double rotation J+0 / J+10h minimum), puis AzureADKerberos après confirmation de stabilité. Espacer d'au moins 10 à 24h entre les deux.

Aligner les deux cycles sur un calendrier mensuel simplifie la gestion et réduit le risque d'oubli.

</div>

### 8.3 Automatisation via Azure Automation

Script PowerShell de rotation automatisée à déployer dans un runbook Azure Automation, cadence mensuelle ou selon la politique définie en 8.1.

Logique du runbook :
1. Récupérer les credentials depuis Azure Key Vault (jamais en clair dans le script)
2. Exécuter `Set-AzureADKerberosServer -RotateServerKey`
3. Exécuter `Get-AzureADKerberosServer` → vérifier que `CloudKeyLastSynced` est inférieur à 1h
4. Si écart supérieur à 1h : envoyer une alerte email / Teams → intervention manuelle requise
5. Logger le résultat dans Log Analytics pour audit trail

Droits requis pour le compte de service : Hybrid Identity Administrator (Entra ID) + droits délégués Domain Admin (via Credential stocké dans Key Vault).

---

## 9. Annexe A : FIDO2 hardware pour comptes à privilèges {#9-fido2}

### 9.1 Quand préférer une clé physique à WHfB par TPM

WHfB par TPM lie l'authentification forte à un poste spécifique. Pour les comptes à privilèges élevés (Global Admin, Privileged Role Administrator) qui peuvent s'authentifier depuis plusieurs postes, une clé FIDO2 roaming offre la même garantie cryptographique avec une portabilité entre postes.

| Critère | WHfB / TPM | Clé FIDO2 physique |
|---|---|---|
| Portabilité | Lié au poste | Portable entre postes |
| Coût | Zéro (intégré au poste) | 30 à 80 CHF / unité selon modèle |
| Durée de vie déclarée | Liée au poste (5 à 10 ans) | YubiKey : 5 ans minimum, typiquement 7 à 10 ans. Feitian : 5 ans déclarés. Limite pratique : perte physique ou changement de politique, pas la défaillance matérielle (connecteur USB certifié >10 000 insertions). |
| Déploiement | Via Intune, scalable | Distribution physique, manuelle |
| Perte / vol | Risque nul (clé dans TPM) | Risque physique : procédure de révocation nécessaire |
| Comptes recommandés | Tous les utilisateurs standard | Global Admin, break-glass, comptes PIM |

### 9.2 Activation FIDO2 dans Entra ID

Portail : `entra.microsoft.com` → Méthodes d'authentification → Stratégies → Clé d'accès (FIDO2)

Page : **Paramètres de clé d'accès (FIDO2)**

<div class="callout-yellow">

**Important : FIDO2 est désactivé par défaut**

Contrairement à CAE, la méthode "Clé d'accès (FIDO2)" est désactivée par défaut sur tous les tenants. Une activation manuelle est requise.

</div>

**Onglet "Activer et cibler" :**

<div class="step"><strong>1 · Activer</strong> — Toggle Activer → Activé</div>
<div class="step"><strong>2 · Inclure</strong> — Onglet Inclure → Tous les utilisateurs (ou groupe spécifique selon la stratégie de déploiement)</div>
<div class="step"><strong>3 · Exclure</strong> — Onglet Exclure → Ajouter le groupe break-glass</div>

**Onglet "Configurer" :**

- Default passkey profile : Types = Device-bound, Synced, Restrictions = Non
- Pour les comptes à privilèges : créer un profil dédié avec Device-bound uniquement et AAGUIDs spécifiques YubiKey (via Ajouter un profil)

<div class="callout">

**Note sur Microsoft Authenticator dans les AAGUIDs**

L'interface propose Windows Hello, Microsoft Authenticator et Enter AAGUID comme raccourcis. Microsoft Authenticator = passkey Synced stockée dans l'app mobile, synchronisée via Microsoft. Plus pratique mais légèrement moins robuste qu'une clé Device-bound pour les comptes ultra-sensibles. Pour Global Admin : préférer Device-bound (YubiKey ou TPM) uniquement.

</div>

**Enrôlement :** l'utilisateur enrôle sa clé ou passkey via `aka.ms/mysecurityinfo`.

---

## 10. Conclusion : honnêteté intellectuelle sur le niveau de protection {#10-conclusion}

Ce document ne rend pas un tenant Microsoft 365 inattaquable. Il déplace le curseur de difficulté pour l'attaquant. La hiérarchie est claire :

| Niveau | Mesure | Ce que ça garantit |
|---|---|---|
| Base | MFA seul, sans CAE | Cookie volé exploitable ~1h après compromission |
| + CAE | CAE + Identity Protection | Fenêtre réduite à quelques minutes, détection améliorée |
| + SIEM | Règles M2/M3/M5 à M8/M12/M14 + M-new-1 + M-new-2 | Visibilité sur les actions post-exploitation dans le tenant |
| + Purview Suite (add-on) | IRM Fuites-Données + eDiscovery Premium | Couverture SharePoint et constitution de preuves nLPD |
| + WHfB / FIDO2 | Protection structurelle (TPM + Cloud Kerberos Trust) | Rejeu AiTM cryptographiquement impossible |
| Posture complète | WHfB + rotation clé + audit legacy + SIEM + IRM | Résistance structurelle + gestion du risque résiduel |

Les autres vecteurs restent actifs indépendamment de WHfB : endpoint compromis (malware, keylogger), ingénierie sociale directe, compromission du DC on-prem. WHfB ferme le vecteur AiTM mais ne remplace pas une posture de sécurité globale.

*Prochaine étape recommandée : déployer CAE (Partie A, vérification en 20 minutes) immédiatement, puis planifier WHfB comme projet de durcissement sur 1 à 2 sessions.*

---

## 11. Annexe B : pourquoi un SIEM est indispensable, même gratuit {#11-siem}

### 11.1 Le problème sans SIEM

Les IoC décrits dans ce document existent dans les journaux Entra ID, Exchange et Defender. Identity Protection envoie des alertes, mais elles arrivent dans un portail que personne ne surveille activement. Sans SIEM, la détection est manuelle, réactive et souvent trop tardive.

Un SIEM avec des règles de corrélation transforme ces signaux en alertes actionnables, en temps réel, vers les canaux que l'équipe IT surveille effectivement (email, Teams, SMS). C'est la différence entre découvrir une compromission 3 jours après les faits et réagir en 7 minutes.

### 11.2 Les options open source / gratuites pour une PME

| SIEM | Points forts | Points faibles | Intégration M365 |
|---|---|---|---|
| Wazuh | Très populaire, grande communauté, documentation abondante, agents endpoint Windows/Linux natifs, XDR intégré | Configuration M365 manuelle et complexe, courbe d'apprentissage élevée, ressources serveur importantes | Via module Wazuh Office 365 (API Management Activity), configuration manuelle des règles de corrélation |
| UTMStack CE | Interface intuitive, intégration O365 native, règles M-series validées en live sur les scénarios AiTM et CVE-2026-69836, lab documenté disponible | Communauté plus petite, moins de ressources en ligne que Wazuh | Native O365 : règles M-series prêtes à importer, validées en production |
| Graylog | Excellent pour la gestion de logs en volume, interface claire, performant sur de grands parcs | Moins orienté SIEM que les deux autres, corrélation limitée en version CE | Via inputs personnalisés, nécessite développement des règles de corrélation |

### 11.3 Recommandation pour une PME M365

Pour une PME de 5 à 25 utilisateurs sous M365 Business Premium, UTMStack CE offre le meilleur rapport déploiement / couverture immédiate pour les scénarios documentés dans ce guide. Les règles M-series couvrent les vecteurs AiTM et CVE-2026-69836 sans développement supplémentaire.

Wazuh est une excellente alternative si la PME dispose déjà d'un responsable IT à l'aise avec la configuration avancée ou si elle souhaite une couverture endpoint plus large.

<div class="callout-ok">

**Lab UTMStack : règles M365 validées en live**

Le lab UTMStack documenté sur GitHub Pages couvre l'ensemble des règles de corrélation M365 utilisées dans ce guide, avec les YAML prêts à importer, les tests de déclenchement documentés et les champs disponibles dans chaque alerte.

- [Chapitre 11 : règles de corrélation M-series (dont M-new-1 et M-new-2)](https://doit4everyone.github.io/utmstack-lab/docs/11-correlations-yaml.html)
- [Lab UTMStack complet](https://doit4everyone.github.io/utmstack-lab/)

</div>

---

## 12. Renvois et ressources {#12-renvois}

| Document / Ressource | Périmètre licence | Lien |
|---|---|---|
| Guide MVC Purview nLPD — PME Suisse | Business Premium (sans add-on) | [doit4everyone.github.io](https://doit4everyone.github.io/Configuration-Purview-PME-Suisse-nLPD/) |
| Guide complet Purview 2026 — IRM (8.2), eDiscovery (8.3) | Business Premium + Purview Suite (add-on requis) | [Section 8 — Fonctionnalités avancées](https://doit4everyone.github.io/microsoft-purview-configuration-2026-nLPD/docs/09-fonctionnalites-avancees.html) |
| Guide Shadow AI / Gouvernance Copilot | Business Premium + Purview Suite (add-on requis) | [doit4everyone.github.io](https://doit4everyone.github.io/shadow-ai-governance-microsoft-365-nLPD/) |
| Lab UTMStack — Chapitre 11 (règles M-new-1, M-new-2) | UTMStack CE (open source) | [11-correlations-yaml.html](https://doit4everyone.github.io/utmstack-lab/docs/11-correlations-yaml.html) |
| Lab UTMStack — accueil | UTMStack CE (open source) | [utmstack-lab](https://doit4everyone.github.io/utmstack-lab/) |
| Microsoft — Cloud Kerberos Trust | Business Premium (inclus) | [learn.microsoft.com](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-hybrid-cloud-kerberos-trust) |
| Microsoft — CAE | Business Premium (inclus) | [learn.microsoft.com](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-continuous-access-evaluation) |
| Microsoft — FIDO2 security keys | Business Premium (inclus) | [learn.microsoft.com](https://learn.microsoft.com/en-us/entra/identity/authentication/howto-authentication-passwordless-security-key) |
| Microsoft — CVE-2026-69836 advisory | N/A | [msrc.microsoft.com](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2026-69836) |

---

## 📥 Téléchargements

- [Version Word (.docx)]({{ '/downloads/docx/MVC-nLPD_Annexe-AiTM_v1.0.docx' | relative_url }})

---

<div class="nav-links">

[← Retour au bundle MVC nLPD](../) · [🏠 Retour au portail principal](https://doit4everyone.github.io/)

</div>

---

*ℹ️ Références, structuration et aide à la rédaction assistées par IA, avec validation humaine finale. Validation terrain effectuée sur infrastructure hybride réelle — août 2026.*
