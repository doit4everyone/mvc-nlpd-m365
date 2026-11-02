---
title: "AiTM / Cookie Hijacking M365 : Détection, mitigation et protection structurelle | DoIt4Everyone"
description: "Guide technique complet sur les attaques AiTM contre Microsoft 365. CAE, WHfB, FIDO2, Cloud Kerberos Trust, CA Authentication Strength. Validé terrain sur infra hybride, août 2026."
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
  section { width: 100% !important; float: none !important; margin: 0 !important; }
  h1 { text-align: center; }
  h2 { border-bottom: 2px solid #2E74B5; padding-bottom: 6px; color: #1F3864; margin-top: 2em; }
  h3 { color: #2E74B5; margin-top: 1.5em; }
  table { width: 100%; border-collapse: collapse; margin: 16px 0; font-size: 0.95em; }
  th { background: #1F3864; color: #fff; padding: 8px 10px; text-align: left; }
  td { padding: 7px 10px; border-bottom: 1px solid #dde4f0; }
  tr:nth-child(even) td { background: #EEF4FB; }
  blockquote { border-left: 4px solid #2E74B5; background: #D6E4F0; padding: 10px 14px; margin: 16px 0; border-radius: 0 4px 4px 0; }
  blockquote p { margin: 4px 0; }
  code { background: #f0f4f8; padding: 2px 5px; border-radius: 3px; font-size: 0.9em; font-family: "SFMono-Regular", Consolas, monospace; }
  pre { background: #f0f4f8; padding: 14px; border-radius: 4px; overflow-x: auto; font-size: 0.88em; line-height: 1.5; }
  .partie-banner { background: #1F3864; color: #fff; text-align: center; padding: 14px 20px; border-radius: 4px; margin: 30px 0 20px; }
  .warn { border-left-color: #C55A11 !important; background: #FCE4D6 !important; }
  .ok { border-left-color: #375623 !important; background: #E2EFDA !important; }
  .red { border-left-color: #C00000 !important; background: #FDECEA !important; }
  .yellow { border-left-color: #C55A11 !important; background: #FFF2CC !important; }
  .purple { border-left-color: #4B0082 !important; background: #EDE7F6 !important; }
</style>

🌐 [English version](index.en.html) · Version française

[← Retour au bundle MVC nLPD](../)

# AiTM / Cookie Hijacking M365

<p style="text-align:center;color:#2E74B5;font-size:1.3em;font-weight:bold;">Détection · Mitigation · Protection structurelle</p>

<p style="text-align:center;color:#555;">Guide technique autonome · Microsoft 365 Business Premium · v1.1 · Août 2026, corrigée le 2 novembre 2026<br>Validé terrain sur infrastructure hybride (Entra Connect + AD on-prem + WS2025)</p>

---

<blockquote class="warn">
<p><strong>Contexte de publication</strong></p>
<p>Ce document a été rédigé suite à deux publications majeures d'août 2026 : l'étude ANY.RUN sur Mirage2FA et l'avis Microsoft sur CVE-2026-69836. Les deux sujets sont distincts : rien n'indique que Mirage2FA exploite cette faille.</p>
<p><strong>Mirage2FA (groupe LinX Coders, étude ANY.RUN du 18 août 2026) :</strong> plateforme PhaaS exploitant une technique AiTM par proxy WebSocket contre Microsoft 365, active de 2024 à 2026. Selon les estimations d'ANY.RUN : 9 426 adresses ciblées, 4 532 potentiellement compromises (48 %), et 4 561 événements de vol de cookie de session, l'issue la plus fréquente.</p>
<p><strong>CVE-2026-69836 (avis Microsoft du 20 août) :</strong> vulnérabilité d'exécution de code à distance (désérialisation) CVSS 10.0 dans Entra ID, corrigée côté infrastructure Microsoft sans action requise côté client. Microsoft a initialement indiqué une exploitation active, avant de corriger son avis le 21 août 2026 et de confirmer que la vulnérabilité n'avait pas été exploitée dans la nature.</p>
<p>Audience : consultants IT / MSP et responsables IT internes accompagnant des PME sous M365 Business Premium.</p>
<p>Ce document est indépendant mais complémentaire au <a href="https://doit4everyone.github.io/Configuration-Purview-PME-Suisse-nLPD/">Guide MVC Purview nLPD</a>.</p>
</blockquote>

---

**Table des matières**

**Partie A · Plan B : actionnable immédiatement**

- [1. Contexte : le MFA ne suffit plus](#contexte)
- [2. CAE : Continuous Access Evaluation](#cae)
- [3. Indicateurs de compromission (IoC)](#ioc)
- [4. Réponse immédiate : alerte SIEM reçue](#reponse)

**Partie B · Plan A : protection structurelle**

- [5. Pourquoi WHfB / FIDO2 neutralise l'attaque AiTM](#whfb)
- [6. Prérequis et audit préalable](#prerequis)
- [7. Déploiement : Cloud Kerberos Trust et AzureADKerberos](#deploiement)
- [8. Rotation de la clé AzureADKerberos](#rotation)
- [9. Annexe A : FIDO2 hardware pour comptes à privilèges](#fido2)
- [10. Conclusion](#conclusion)
- [11. Annexe B : pourquoi un SIEM est indispensable, même gratuit](#siem)
- [12. Renvois et ressources](#renvois)

---

<div class="partie-banner">
<strong>PARTIE A · Plan B : actionnable immédiatement</strong><br>
<em>CAE · Détection des IoC · SIEM · Réponse immédiate</em>
</div>

<a name="contexte"></a>

## 1. Contexte : le MFA ne suffit plus

### 1.1 Qu'est-ce qu'une attaque AiTM ?

Une attaque AiTM (Adversary-in-the-Middle) moderne ne cherche plus à casser le MFA. Elle le laisse fonctionner normalement et intercepte ce qui vient après : le cookie de session authentifié.

Le mécanisme de Mirage2FA illustre parfaitement cette approche :

1. **Hameçonnage** : l'utilisateur reçoit un e-mail avec un lien, une pièce jointe (.htm, .xhtml, .svg) ou un QR code menant vers un proxy WebSocket transparent, visuellement identique à login.microsoft.com.
2. **Proxy transparent** : le proxy relaie chaque requête vers Microsoft en temps réel. L'utilisateur complète son MFA normalement.
3. **Interception** : le proxy capture le cookie de session authentifié. L'attaquant dispose d'un cookie valide.
4. **Rejeu** : l'attaquant importe le cookie dans son navigateur et accède à M365 sans credentials ni MFA. Le reset du mot de passe ne suffit pas : le cookie reste valide jusqu'à révocation explicite.

<blockquote class="red">
<p><strong>Mirage2FA : chiffres de l'étude ANY.RUN (18 août 2026, campagne 2024 à 2026)</strong></p>
<p>9 426 adresses ciblées · 4 532 comptes Microsoft 365 potentiellement compromis (48 %)<br>
4 561 événements de vol de cookie de session (2 541 victimes uniques) : l'issue la plus fréquente, environ 48,9 % des 9 332 événements de compromission potentielle recensés<br>
Estimations d'ANY.RUN : impact potentiel, pas des compromissions confirmées une à une<br>
Vecteur : proxy WebSocket, indétectable par l'utilisateur<br>
Contre-mesure insuffisante : reset du mot de passe seul</p>
</blockquote>

### 1.2 CVE-2026-69836 : quand la vulnérabilité est dans l'infrastructure elle-même

Le 20 août 2026, Microsoft a publié l'avis CVE-2026-69836, une vulnérabilité de désérialisation de données non fiables dans Entra ID (exécution de code à distance, score CVSS 10.0). La correction a été déployée côté infrastructure Microsoft : aucune action patch requise côté client. Microsoft a initialement indiqué une exploitation active, avant de corriger son avis le 21 août 2026 et de confirmer que la vulnérabilité n'avait pas été exploitée dans la nature. Rien n'indique que Mirage2FA l'exploite : son mécanisme repose sur un proxy d'hameçonnage.

Même si Microsoft indique l'absence d'exploitation, la question prudente n'est pas "est-ce corrigé ?" mais "est-ce que quelque chose s'est passé dans mon tenant avant le fix ?"

<blockquote class="yellow">
<p><strong>Audit rétrospectif recommandé par précaution : période précédant le 20 août 2026</strong></p>
<p>· Nouveaux role assignments Global Admin / Privileged Role Admin<br>
· Credentials ajoutés sur des service principals existants<br>
· Modifications de Conditional Access policies<br>
· Consentements OAuth inhabituels<br>
· Nouvelles méthodes MFA enregistrées sur des comptes à privilèges<br>
· Comptes cloud créés hors du processus de synchronisation</p>
<p><strong>Limite :</strong> journaux d'audit et de connexion Entra ID conservés 30 jours (P1 et P2). Au-delà, sans routage vers Log Analytics ou un compte de stockage via Azure Monitor, ils ne sont plus accessibles. Le journal d'audit Microsoft 365 (Purview Audit Standard, événements Exchange compris) a sa propre rétention : 180 jours par défaut.<br>
→ Recommandation permanente : router les logs Entra ID vers Log Analytics.</p>
<p>Procédure complète d'audit rétrospectif : voir section 4.4.</p>
</blockquote>

### 1.3 Pourquoi le MFA classique ne suffit pas structurellement

Le MFA protège l'authentification mais pas la session qui en résulte. Une fois le cookie de session émis, il représente une preuve d'identité valide aux yeux de Microsoft 365, indépendamment de la méthode d'authentification utilisée pour l'obtenir.

| Mesure | Efficacité contre AiTM | Remarque |
|---|---|---|
| MFA (TOTP / SMS) | Insuffisante | Le proxy relaie le MFA en temps réel |
| MFA (Authenticator push) | Insuffisante | L'utilisateur approuve, le proxy intercepte |
| Reset du mot de passe | Insuffisante | Le cookie reste valide après reset |
| CAE (Continuous Access Evaluation) | Partielle | Révocation en quasi temps réel (jusqu'à 15 minutes selon Microsoft) pour les clients compatibles |
| Identity Protection (détections détaillées : Entra ID P2) | Partielle | Signale un rejeu depuis un contexte inhabituel (IP, pays, jeton anormal) |
| WHfB (clé dans le TPM) ou passkey FIDO2 | Structurelle | Le proxy AiTM ne peut pas obtenir de session valide : voir Partie B |

---

<a name="cae"></a>

## 2. CAE : Continuous Access Evaluation

### 2.1 Principe, durée de vie des tokens et nuances

CAE modifie deux aspects du cycle de vie des tokens : leur durée de vie et leur révocabilité.

| Type de token | Durée de vie | Révocabilité | Scénario AiTM |
|---|---|---|---|
| Token standard (client non CAE-capable) | 1 heure par défaut | La révocation bloque les nouveaux jetons, le jeton d'accès déjà émis reste valide jusqu'à expiration | Jusqu'à ~1h d'accès après révocation |
| Token CAE (client ET ressource CAE-capables) | Jusqu'à 28 heures | Révocable par événement critique (révocation, désactivation, changement de mot de passe, risque élevé) | Fenêtre plus longue si non détecté, révocation en quasi temps réel si détecté (jusqu'à 15 minutes de propagation selon Microsoft) |

<blockquote class="warn">
<p><strong>Paradoxe CAE vu du côté attaquant</strong></p>
<p>Un token CAE peut durer jusqu'à 28 heures au lieu d'une heure. Pour un attaquant qui vole un token CAE via AiTM, la fenêtre d'exploitation potentielle est donc beaucoup plus longue qu'avec un token standard, si personne ne détecte l'incident.</p>
<p>Le modèle CAE tient seulement si la détection et la révocation sont rapides. Sans SIEM ni surveillance active, CAE peut être un avantage pour l'attaquant. C'est pour cette raison que la détection (SIEM + Identity Protection) est critique.</p>
</blockquote>

Applications M365 supportant CAE côté serveur : Exchange Online, SharePoint Online, OneDrive, Microsoft Teams, MS Graph. Côté client, CAE nécessite un client compatible, validé en lab avec Microsoft Edge sur SharePoint Online. Firefox et l'application web Outlook (outlook.office.com) n'ont pas montré de token CAE dans nos tests.

<blockquote class="ok">
<p><strong>CAE : actif par défaut, révocation validée en live (23 août 2026)</strong></p>
<p>Selon Microsoft, l'évaluation des événements critiques ne dépend pas d'une stratégie d'accès conditionnel et est disponible dans tout tenant. Sur notre tenant de lab, CAE était actif sans configuration : l'option visible dans Accès conditionnel (Personnaliser l'évaluation de l'accès continu) sert à le désactiver ou à le durcir. Certains tenants anciens ayant configuré CAE dans l'ancienne interface doivent migrer leur réglage vers l'accès conditionnel.</p>
<p>Mesuré en lab sur tenant Business Premium (Microsoft annonce une latence pouvant aller jusqu'à 15 minutes) :<br>
· Révocation session active en ~1 minute (myaccount.microsoft.com)<br>
· ~3 à 4 minutes sur SharePoint Online via Edge (token CAE confirmé Is CAE Token = Oui)</p>
<p>La révocation fonctionne sur tous les types de tokens (CAE et standard) via "Révoquer les sessions" dans Entra ID.</p>
</blockquote>

<blockquote>
<p><strong>Identity Protection : différence P1 vs P2 (source : Microsoft Learn)</strong></p>
<p><strong>Business Premium sans add-on (P1) :</strong><br>
· Quelques détections seulement (adresse IP anonyme, identifiants divulgués, renseignement sur les menaces Microsoft Entra)<br>
· Les autres détections apparaissent sous le libellé générique "Additional risk detected", sans détails<br>
· Pas de risk-based Conditional Access (révocation automatique sur détection)<br>
· Révocation manuelle uniquement après détection via SIEM ou surveillance manuelle</p>
<p><strong>Business Premium + Defender Suite add-on (P2) :</strong><br>
· Détections détaillées : Anomalous token, Unfamiliar sign-in properties, Malicious IP address, Token issuer anomaly<br>
· Impossible travel : exige en plus Defender for Cloud Apps (ou Microsoft 365 E5)<br>
· Risk-based Conditional Access : révocation automatique sur détection de risque</p>
<p>Dans les deux cas : la révocation manuelle fonctionne immédiatement via "Révoquer les sessions".</p>
</blockquote>

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

**Résultat attendu :** aucune ligne retournée = aucune stratégie d'accès conditionnel ne désactive ou ne personnalise CAE. ✅

Ce script interroge l'API Graph beta, susceptible d'évoluer. En cas de doute, la vérification via le portail ci-dessous fait foi.

**Via le portail :**

`entra.microsoft.com` → Accès conditionnel → Stratégies, puis vérifier qu'aucune policy ne contient Session → Personnaliser l'évaluation de l'accès continu → Désactiver.

<blockquote>
<p><strong>Ce que vous voyez dans l'interface</strong></p>
<p>Dans Accès conditionnel → Nouvelle stratégie → Session, l'option "Personnaliser l'évaluation de l'accès continu" propose deux sous-options uniquement : Désactiver et Appliquer strictement les stratégies de localisation (préversion). Il n'existe pas d'option "Activer" : CAE est actif par défaut sans aucune action.</p>
</blockquote>

### 2.3 Test de révocation

1. Connecter un compte test sur `myaccount.microsoft.com` dans une fenêtre de navigation privée.
2. `entra.microsoft.com` → Utilisateurs → [compte test] → Vue d'ensemble → **Révoquer les sessions** → Oui.
3. Mesurer le délai avant invalidation de la session.

**Résultat validé en live (23 août 2026) :** session invalidée en ~1 minute. Toast de confirmation : "Sessions de connexion révoquées pour [compte]".

Pour un test CAE natif : utiliser SharePoint Online via Edge : token CAE confirmé (`Is CAE Token = Oui` dans les logs de connexion Entra ID). La révocation se produit au prochain appel réseau du client (refresh de page).

---

<a name="ioc"></a>

## 3. Indicateurs de compromission (IoC)

### 3.1 Signaux dans Identity Protection (détails : Entra ID P2)

`entra.microsoft.com` → Identity Protection → Détections de risques

Les détections ci-dessous exigent Entra ID P2. Sans P2 (Business Premium seul), elles apparaissent sous le libellé générique "Additional risk detected", sans détails.

| Signal | Description | Pertinence |
|---|---|---|
| Impossible travel | Connexion depuis deux géographies incompatibles en peu de temps (exige aussi Defender for Cloud Apps) | Partielle : si l'attaquant rejoue depuis une IP géographiquement incohérente |
| Unfamiliar sign-in properties | Nouvelle IP, nouveau user-agent, nouveau pays | Bonne : l'infra de l'attaquant diffère du profil habituel |
| Anomalous token | Jeton aux caractéristiques inhabituelles (durée de vie anormale, rejeu depuis un emplacement inconnu) | Directe : signal le plus spécifique pour un cookie rejoué |
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

Référence : [Chapitre 11 du lab UTMStack](https://doit4everyone.github.io/utmstack-lab/docs/11-correlations-yaml.html)

**Bloc 1 : règles opérationnelles sans configuration supplémentaire**

| Règle UTMStack | Signal couvert | Scénario |
|---|---|---|
| M2 : OAuth Application Consent Granted | Consentement OAuth suspect post-compromission | AiTM + compromission Entra ID (scénario CVE-2026-69836) |
| M3 : Impossible Travel | Connexion depuis pays géographiquement impossible | AiTM |
| M5 : MFA Required Interrupt | Interrupt MFA visible même si proxy a relayé le token | AiTM |
| M6 : Member Added to Group | Ajout à un groupe Entra ID | Compromission Entra ID (scénario CVE-2026-69836) |
| M7 : User Account Created Outside Sync | Compte cloud créé hors Entra Connect | Compromission Entra ID (scénario CVE-2026-69836) |
| M8 : Unified Audit Log Disabled | Tentative d'aveugler le SIEM post-compromission | AiTM + compromission Entra ID (scénario CVE-2026-69836) |
| M12 : External Mailbox Access | Accès externe à une boîte Exchange via OAuth | AiTM |
| M14 : Sign-In Blocked by CA | Connexion bloquée par Conditional Access | AiTM |

**Bloc 2 : deux règles complémentaires pour le scénario CVE-2026-69836**

| Règle | Action Entra ID détectée | Validation |
|---|---|---|
| M-new-1 : Privileged Role Assigned | `Add member to role.` → Global Admin, Security Admin, etc. | ✅ Validée en live : 22 août 2026 (latence ~7 min) |
| M-new-2 : Credentials Added to Service Principal | `Add service principal credentials.` → persistance sans MFA | ✅ Validée en live : 22 août 2026 (latence ~4 min) |

<blockquote>
<p><strong>M-new-1 et M-new-2 : champs disponibles dans l'alerte UTMStack</strong></p>
<p><strong>M-new-1 :</strong> <code>action = "Add member to role."</code> · <code>log.ObjectId</code> = UPN du compte assigné · <code>log.ModifiedProperties</code> : Role.DisplayName · <code>origin.user</code> = acteur · latence ~7 min</p>
<p><strong>M-new-2 :</strong> <code>action = "Add service principal credentials."</code> · <code>log.Target</code> = nom du service principal · <code>origin.user</code> = acteur · latence ~4 min</p>
<p><strong>Note :</strong> M-new-2 se déclenche uniquement via appel API Graph direct (<code>addPassword</code>) ou PowerShell. L'interface graphique Entra ID génère un libellé différent. Test : <code>POST https://graph.microsoft.com/v1.0/servicePrincipals/{id}/addPassword</code> via <a href="https://developer.microsoft.com/en-us/graph/graph-explorer">Graph Explorer</a>.</p>
</blockquote>

**Bloc 3 : limites de visibilité**

<blockquote class="red">
<p><strong>Angles morts structurels : indépendants de tout outil ou licence</strong></p>
<p>· CVE-2026-69836, phase initiale : un défaut de ce type se situe dans l'infrastructure Microsoft. Aucun event n'est généré dans les journaux client, même si Microsoft indique aujourd'hui l'absence d'exploitation. Seules les actions consécutives dans le tenant sont visibles, c'est ce que couvrent M-new-1 et M-new-2.<br>
· Rétention des logs : journaux d'audit et de connexion Entra ID conservés 30 jours (P1 et P2). Au-delà, sans routage vers Log Analytics ou un compte de stockage via Azure Monitor, ils ne sont plus accessibles. Le journal d'audit Microsoft 365 (Purview Audit Standard, événements Exchange compris) a sa propre rétention : 180 jours par défaut.<br>
→ Recommandation permanente : router les logs Entra ID vers Log Analytics.</p>
</blockquote>

<blockquote class="yellow">
<p><strong>Angle mort UTMStack : limitation technique du pipeline ETL v11</strong></p>
<p>Exfiltration SharePoint : les events FileDownloaded (RecordType 6) ne sont pas collectés. La règle M13 est documentée comme non implémentable (voir Chapitre 11).</p>
<p>Couverture partielle disponible via Microsoft Purview Insider Risk Management (stratégie IRM-Fuites-Données-PME). Nécessite l'add-on Purview Suite, hors périmètre Business Premium seul.</p>
<p>Bonnes pratiques recommandées avant activation IRM (au regard de l'art. 26 OLT3 et de l'art. 6 nLPD) :<br>
· Information préalable des employés dans le règlement du personnel ou la charte IT<br>
· Anonymisation activée par défaut dans Purview<br>
· Procédure écrite de levée d'anonymat (voir Annexe G du guide Purview complet)</p>
<p>Référence : <a href="https://doit4everyone.github.io/microsoft-purview-configuration-2026-nLPD/docs/09-fonctionnalites-avancees.html">Guide complet Purview 2026, section 8.2</a></p>
</blockquote>

---

<a name="reponse"></a>

## 4. Réponse immédiate : alerte SIEM reçue

<blockquote class="red">
<p><strong>Point critique : quand une alerte de rejeu remonte, le vol a généralement déjà eu lieu</strong></p>
<p>Les signaux post-authentification (rejeu, consentement OAuth, règle de boîte aux lettres) indiquent le plus souvent que l'attaquant détient déjà le cookie de session. L'objectif devient de limiter la durée d'exploitation. Un signal plus précoce (clic sur un lien d'hameçonnage connu, par exemple) peut en revanche permettre d'agir avant le vol.</p>
<p>· Sans CAE : la révocation bloque les nouveaux jetons, mais le jeton d'accès déjà émis reste utilisable jusqu'à son expiration (une heure par défaut).<br>
· Avec CAE : la révocation s'applique en quasi temps réel aux clients compatibles, avec une latence pouvant aller jusqu'à 15 minutes selon Microsoft (1 à 4 minutes mesurées en lab).</p>
<p><strong>CAE doit être actif AVANT l'incident pour que la réponse soit efficace. Voir section 2.</strong></p>
</blockquote>

### 4.1 Séquence des signaux à surveiller dans le SIEM

| Phase | Délai typique | Signaux SIEM (règles UTMStack) | Interprétation |
|---|---|---|---|
| Phase 1 : accès initial | T+0 à T+5 min | M3 (Impossible Travel), M5 (MFA Interrupt) | Le cookie vient d'être volé et rejoué. Agir immédiatement. |
| Phase 2 : exploitation | T+5 à T+30 min | M2 (OAuth Consent), M12 (External Mailbox Access) | L'attaquant établit sa persistance et commence l'exfiltration. |
| Phase 3 : persistance longue durée | T+30 min à plusieurs heures | M-new-1 (Role Assigned), M-new-2 (Credentials SPN), M7 (Account Created) | L'attaquant prépare un accès durable, indépendant du cookie. |

<blockquote class="red">
<p><strong>Règle d'or</strong></p>
<p>Si vous voyez Phase 2 ou Phase 3 sans avoir vu Phase 1 dans le SIEM, l'attaque initiale a eu lieu en dehors de votre fenêtre de détection. La révocation s'impose immédiatement et les dégâts de Phase 2 sont peut-être déjà faits. Lancer l'audit complet de la section 4.3.</p>
</blockquote>

### 4.2 Révocation immédiate des tokens : à faire dans les 2 minutes

1. **Révoquer toutes les sessions** : `entra.microsoft.com` → Utilisateurs → [compte compromis] → Vue d'ensemble → Révoquer les sessions → Oui. Équivalent PowerShell, utile en script : `Revoke-MgUserSignInSession -UserId [UPN]`. Il s'agit de la même opération : inutile de faire les deux.
2. **Vérifier la propagation CAE** : si CAE est actif, tester une action SharePoint / Teams sur le compte concerné. La session doit être invalidée en quelques minutes (jusqu'à 15 minutes selon Microsoft).

<blockquote class="yellow">
<p><strong>Sans CAE : que faire jusqu'à l'expiration du jeton d'accès (une heure par défaut) ?</strong></p>
<p>Alerter immédiatement l'utilisateur concerné de ne plus rien faire sur sa session. Surveiller en temps réel les actions dans Exchange (redirection mail), SharePoint (téléchargements), Teams (messages envoyés) et Entra ID (modifications de configuration). Documenter chaque action observée avec horodatage pour constituer le dossier d'incident.</p>
</blockquote>

### 4.3 Audit des dégâts : à faire dans les 30 minutes

**Journal d'audit Microsoft 365 (événements Exchange)**

`purview.microsoft.com` → Audit → filtrer sur [date/heure de l'incident] → [compte compromis]

- `Set-InboxRule` ou `New-InboxRule` : règles de redirection mail créées
- `Add-MailboxPermission` : délégations mailbox ajoutées

**Journaux d'audit Entra ID**

`entra.microsoft.com` → Journaux d'audit → filtrer sur [date/heure de l'incident] → [compte compromis]

- `Consent to application` : applications OAuth consenties
- `Update user` sur les méthodes d'authentification : méthodes MFA modifiées
- `Add member to role` : role assignments
- `Add service principal credentials` : credentials SPN ajoutés

**Journaux de connexion Entra ID**

`entra.microsoft.com` → Journaux de connexion → filtrer sur [compte compromis] → période de l'incident

Chercher : IP source inhabituelle, user-agent générique ou malformé, géolocalisation incohérente.

### 4.4 Audit rétrospectif post-CVE-2026-69836

Si l'audit révèle des actions suspectes dans la fenêtre précédant le 20 août 2026, appliquer la séquence de révocation sur les comptes concernés, puis vérifier spécifiquement :

- `entra.microsoft.com` → Rôles et administrateurs → vérifier chaque rôle Global Admin / Security Admin / Privileged Role Admin
- `entra.microsoft.com` → Applications d'entreprise → Certificats et secrets → secrets récemment ajoutés
- `entra.microsoft.com` → Accès conditionnel → vérifier les modifications récentes de policies

<blockquote class="purple">
<p><strong>Constitution de preuves pour notification PFPDT (art. 24 nLPD)</strong></p>
<p>Si l'audit révèle une compromission effective, eDiscovery Premium permet de rechercher toutes les données potentiellement exposées dans Exchange, SharePoint et Teams et de constituer le dossier de preuve pour la notification au PFPDT.</p>
<p>Nécessite l'add-on Purview Suite, hors périmètre Business Premium seul.<br>
Référence : <a href="https://doit4everyone.github.io/microsoft-purview-configuration-2026-nLPD/docs/09-fonctionnalites-avancees.html">Guide complet Purview 2026, section 8.3</a></p>
</blockquote>

### 4.5 Séquence de remédiation complète

<blockquote class="red">
<p><strong>Erreur fréquente : réinitialiser le mot de passe en premier</strong></p>
<p>Le cookie de session reste valide après un reset de mot de passe. La séquence ci-dessous doit être exécutée dans l'ordre indiqué.</p>
</blockquote>

1. Révoquer toutes les sessions (portail ou PowerShell, voir 4.2)
2. Auditer les règles de redirection mail (Exchange Admin Center)
3. Auditer les délégations mailbox (Full Access / Send As)
4. Auditer les applications OAuth consenties récemment
5. Auditer les méthodes MFA enregistrées : supprimer tout dispositif non reconnu
6. Réinitialiser le mot de passe, uniquement après les étapes 1 à 5
7. Forcer une re-registration MFA complète
8. Documenter l'incident : horodatage, IP source, user-agent, actions détectées

---

<div class="partie-banner">
<strong>PARTIE B · Plan A : protection structurelle</strong><br>
<em>WHfB par TPM · Cloud Kerberos Trust · Applications legacy · Rotation AzureADKerberos</em>
</div>

<blockquote>
<p><strong>Note de portée</strong></p>
<p>Cette partie couvre des mesures de durcissement avancées, au-delà du périmètre MVC Business Premium. Audience : consultants IT accompagnant des PME avec postes Hybrid Entra Join ou full cloud. Validation terrain effectuée sur un environnement hybride : Entra Connect + AD on-prem + WS2025.</p>
</blockquote>

<a name="whfb"></a>

## 5. Pourquoi WHfB / FIDO2 neutralise l'attaque AiTM

### 5.1 Le mécanisme cryptographique

Windows Hello for Business utilise une paire de clés asymétriques générée à l'enrôlement :

- **Clé privée :** stockée dans le TPM (si le TPM est exigé, voir 7.2), non extractible, ne quitte jamais le poste
- **Clé publique :** enregistrée dans Entra ID à l'enrôlement

À l'ouverture de session Windows, le fournisseur d'authentification cloud de Windows (Cloud AP) demande un nonce à Entra ID, le signe avec la clé privée et le renvoie. Entra ID vérifie la signature avec la clé publique, puis émet un Primary Refresh Token (PRT) lié à l'appareil. L'authentification est réalisée par Windows, pas par une saisie dans une page web.

Les passkeys FIDO2 (clé de sécurité physique ou passkey) reposent sur un autre mécanisme, WebAuthn : la passkey est liée au domaine pour lequel elle a été enregistrée, et le navigateur ne la propose qu'à ce domaine.

<blockquote class="ok">
<p><strong>Pourquoi l'attaque AiTM échoue avec WHfB / FIDO2</strong></p>
<p>Avec WHfB, il n'y a ni mot de passe ni code à saisir dans une page web : le proxy n'a rien à relayer. Avec une passkey FIDO2, le domaine du proxy n'est pas celui pour lequel la passkey a été enregistrée : le navigateur ne la propose pas. Dans les deux cas, le proxy n'obtient aucune session valide. Microsoft classe ces méthodes dans la force d'authentification "Phishing-resistant MFA" (voir 7.4).</p>
<p>Cette protection n'est complète que si l'accès conditionnel exige une méthode résistante à l'hameçonnage : tant qu'une méthode plus faible reste autorisée, le proxy peut la proposer à l'utilisateur. Elle ne protège pas non plus contre le vol d'un cookie déjà émis par un autre moyen (malware sur le poste, par exemple).</p>
</blockquote>

### 5.2 Comparaison des scénarios de déploiement

| Type de poste | Type de compte | Solution | AzureADKerberos requis ? | Difficulté |
|---|---|---|---|---|
| Full cloud (Entra ID seul) | Cloud-only ou hybride | FIDO2 / WHfB direct | Non | Faible |
| Hybrid Entra Join | Hybride (synchronisé AD) | WHfB + Cloud Kerberos Trust | Oui | Moyenne |
| Hybrid Entra Join | Cloud-only | Non compatible | N/A | Impossible : pas de session Windows possible |
| AD pur (pas de Entra Join) | AD on-prem uniquement | Hors scope WHfB | Non applicable | N/A |

<blockquote class="yellow">
<p><strong>Point critique : compte cloud-only sur poste hybride</strong></p>
<p>Un compte cloud-only ne peut pas ouvrir de session Windows sur un poste Hybrid Entra Join. Windows tente d'authentifier via Kerberos contre l'AD on-prem en premier : le compte n'existe pas dans l'AD, la connexion échoue.</p>
<p>Validé en lab (24 août 2026) : test-m7 (cloud-only) refusé sur poste hybride. Solution : utiliser un compte hybride (synchronisé via Entra Connect) sur les postes hybrides.</p>
</blockquote>

---

<a name="prerequis"></a>

## 6. Prérequis et audit préalable

### 6.1 Prérequis techniques

| Prérequis | Détail | Statut à vérifier |
|---|---|---|
| Entra ID P1 | Inclus dans M365 Business Premium | ✅ Dans le périmètre MVC |
| Windows 10 21H2+ / Windows 11 | Sur les postes ciblés par WHfB | 📋 Inventaire à faire |
| TPM 2.0 | Puce TPM active dans le BIOS/UEFI (firmware ou discret) | 📋 Gestionnaire de périphériques → Modules de plateforme sécurisée |
| Hybrid Entra Join | Entra Connect configuré sur les postes hybrides | 📋 Entra ID → Appareils → confirmer le statut |
| Comptes hybrides | Les comptes cloud-only ne peuvent pas ouvrir de session Windows sur un poste hybride | 📋 Vérifier la source des comptes utilisateurs |
| DC Windows Server 2016+ | Pour Cloud Kerberos Trust en hybride, avec suffisamment de DC accessibles en écriture dans chaque site où les utilisateurs s'authentifient | 📋 Vérifier la version et la répartition des DC |
| Comptes non privilégiés | Les membres directs ou indirects des groupes intégrés privilégiés de l'AD (Admins du domaine, par exemple) ne peuvent pas utiliser Cloud Kerberos Trust | 📋 Tester avec un compte utilisateur standard |
| Droits Global Admin + Domain Admin | Pour la création de l'objet AzureADKerberos | 📋 Disponibles le temps de l'opération |
| PowerShell 5.1 | Le module AzureADHybridAuthenticationManagement ne fonctionne pas en PowerShell 7 | 📋 Vérifier via `$PSVersionTable.PSVersion` |

<blockquote>
<p><strong>Note : comptes privilégiés et postes partagés</strong></p>
<p>Comptes privilégiés : la stratégie de réplication des mots de passe de l'objet AzureADKerberos bloque les comptes à hauts privilèges, comme pour un RODC. Microsoft déconseille d'assouplir cette stratégie pour les débloquer, en raison des vecteurs d'attaque possibles d'Entra ID vers l'AD. Un test de déploiement mené avec un compte administrateur échouera : utiliser un compte standard.</p>
<p>Postes partagés : Microsoft limite à 10 le nombre d'inscriptions WHfB par appareil. Pour les postes utilisés par plus de 10 personnes, ou pour les utilisateurs qui se connectent à de nombreux postes (technicien de support, par exemple), Microsoft recommande des clés de sécurité FIDO2.</p>
</blockquote>

<blockquote class="yellow">
<p><strong>Vérification préalable : GPO WHfB existantes en environnement hybride</strong></p>
<p>Les GPO Active Directory WHfB prennent le dessus sur les policies Intune sur les postes hybrides. Vérifier qu'aucune GPO WHfB n'est déjà en place avant le déploiement :</p>
</blockquote>

```powershell
Get-GPO -All | Where-Object {
  (Get-GPOReport -Guid $_.Id -ReportType XML) -match "PassportForWork"
}
```

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
| RDP vers serveurs legacy | Demande de mot de passe | Activer Remote Credential Guard ou Restricted Admin mode |
| Apps M365 utilisant protocoles hérités (IMAP, POP3, SMTP AUTH) | Blocage si les protocoles hérités ne sont pas déjà bloqués | Déployer une politique Conditional Access bloquant les protocoles hérités |
| Credentials Manager Windows | Renouvellement impossible si mot de passe oublié | Documenter la procédure de reset avec assistance IT |

---

<a name="deploiement"></a>

## 7. Déploiement : Cloud Kerberos Trust et AzureADKerberos

### 7.1 Création de l'objet AzureADKerberos

<blockquote>
<p><strong>Ce que fait cet objet</strong></p>
<p>AzureADKerberos est un objet krbtgt de type RODC virtuel créé dans l'AD et publié dans Entra ID. Il établit un secret partagé entre l'AD on-prem et Entra ID, permettant à Entra ID d'émettre des TGT partiels que les DC on-prem reconnaissent et échangent contre des TGT complets. Résultat : l'utilisateur WHfB accède aux ressources on-prem sans mot de passe, sans PKI.</p>
</blockquote>

**Étape 1 : TLS 1.2**

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12
```

**Étape 2 : module PowerShell** (PowerShell 5.1 requis, ne fonctionne pas en PowerShell 7)

```powershell
Install-Module -Name AzureADHybridAuthenticationManagement -AllowClobber -Force
Import-Module AzureADHybridAuthenticationManagement
```

Note : si le message "Required VC++ 2013 x64 Runtimes do not exist" apparaît, le module installe automatiquement le runtime : laisser terminer.

**Étape 3 : variables**

```powershell
$domain = $env:USERDNSDOMAIN
$domainCred = Get-Credential -Message "Domain Admin credentials"
```

**Étape 4 : création**

```powershell
Set-AzureADKerberosServer -Domain $domain `
  -UserPrincipalName 'admin@[tenant].onmicrosoft.com' `
  -DomainCredential $domainCred
```

Une fenêtre de connexion Entra ID s'ouvre dans le browser : se connecter avec le compte Global Admin ou Hybrid Identity Admin. Pas de sortie = succès silencieux.

**Étape 5 : vérification**

```powershell
Get-AzureADKerberosServer -Domain $domain `
  -UserPrincipalName 'admin@[tenant].onmicrosoft.com' `
  -DomainCredential $domainCred
```

<blockquote class="ok">
<p><strong>Sortie attendue après création réussie (validée en lab, 24 août 2026)</strong></p>
<pre>
Id                 : [numéro unique]
UserAccount        : CN=krbtgt_AzureAD,CN=Users,DC=[domaine],DC=[ext]
ComputerAccount    : CN=AzureADKerberos,OU=Domain Controllers,DC=[domaine],DC=[ext]
DisplayName        : krbtgt_[numéro]
DomainDnsName      : [domaine.ext]
KeyVersion         : [numéro]
CloudKeyVersion    : [même numéro que KeyVersion]  ← synchronisation confirmée
CloudKeyUpdatedOn  : [même date que KeyUpdatedOn]  ← écart idéalement 0 seconde
CloudTrustDisplay  : [vide au premier déploiement, normal]
</pre>
<p>Points à vérifier : KeyVersion = CloudKeyVersion et écart KeyUpdatedOn / CloudKeyUpdatedOn inférieur à 1h.</p>
</blockquote>

### 7.2 Configuration WHfB dans Intune

<blockquote class="yellow">
<p><strong>Stratégie tenant-wide ou profil Protection de compte</strong></p>
<p>La stratégie tenant-wide s'applique au moment de l'inscription des appareils, à toute l'organisation. Après l'inscription, Microsoft indique d'utiliser le profil "Protection de compte" (Sécurité du point de terminaison) ou le catalogue de paramètres pour gérer WHfB sur des groupes ciblés. Si les deux définissent les mêmes paramètres (le PIN, par exemple), c'est la stratégie WHfB qui s'applique : aligner les valeurs pour éviter des comportements inattendus.</p>
<p>La méthode décrite ci-dessous (stratégie tenant-wide + profil Cloud Kerberos Trust) est celle validée en lab.</p>
</blockquote>

**Étape A : configurer la policy tenant-wide WHfB**

`intune.microsoft.com` → Appareils → Inscription → Windows → **Windows Hello Entreprise**

Cette policy est présente sur tous les tenants mais en état "Non configuré" par défaut.

| Paramètre | Valeur recommandée | Remarque |
|---|---|---|
| Configurer Windows Hello Entreprise | Activé | Active WHfB sur tous les appareils inscrits |
| Utiliser un module de plateforme sécurisée (TPM) | Obligatoire | Deux valeurs possibles. **Obligatoire** (valeur par défaut) : seuls les appareils dotés d'un TPM accessible peuvent provisionner WHfB. **Préféré** : l'appareil tente d'utiliser le TPM et, à défaut, recourt au chiffrement logiciel. La clé n'est alors plus protégée par le matériel, ce qui affaiblit les garanties décrites aux sections 5 et 9. Réserver Préféré à une phase de transition sur un parc mixte. |
| Longueur minimale du code PIN | 6 | Minimum recommandé |
| Autoriser l'authentification biométrique | Oui | Empreinte, reconnaissance faciale |
| Utilisez des clés de sécurité pour la connexion | Activé | Active FIDO2 en complément de WHfB |

**Étape B : créer le profil Cloud Kerberos Trust**

`intune.microsoft.com` → Appareils → Configuration → Créer → Nouvelle stratégie

1. Plateforme : Windows 10 et ultérieur
2. Type de profil : **Catalogue de paramètres** (pas Modèles)
3. Nom : WHfB-Cloud-Kerberos-Trust
4. Ajouter des paramètres : rechercher "Cloud Trust" → catégorie Windows Hello Entreprise
5. Paramètre : **Utiliser l'approbation cloud pour l'authentification locale** → Activé
6. Affectation : groupe pilote initial (2 à 3 appareils hybrides non-critiques)
7. Vérifier + créer → Créer

<blockquote class="ok">
<p><strong>Comportement attendu au premier enrôlement WHfB sur poste hybride (validé en lab, 24 août 2026)</strong></p>
<p>1. Connexion Windows avec mot de passe → Windows détecte la policy WHfB<br>
2. Authenticator ou méthode MFA enregistrée demandée pour valider l'identité<br>
3. Assistant WHfB s'ouvre → création du PIN (6 caractères minimum)<br>
4. Reboot possible selon le poste, normal lors de l'application initiale des policies<br>
5. Reconnexion avec le PIN → une fenêtre de connexion Entra ID s'ouvre une seule fois pour créer le PRT<br>
6. Connexions suivantes : PIN seul, plus de fenêtre de connexion Microsoft</p>
<p>Note : la policy tenant-wide Intune fonctionne sans GPO sur les postes hybrides. Si une GPO WHfB existe dans l'AD, elle prend le dessus sur Intune : vérifier section 6.1.</p>
</blockquote>

### 7.3 Vérification de l'état WHfB sur le poste

Depuis une invite de commande administrateur :

```cmd
dsregcmd /status
```

<blockquote class="ok">
<p><strong>Champs clés à vérifier dans la sortie (validés en lab, 24 août 2026)</strong></p>
<pre>
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
</pre>
</blockquote>

<blockquote class="yellow">
<p><strong>Dépannage : PreReqResult = WillNotProvision</strong></p>
<p>Si <code>NgcSet : NO</code> et <code>PreReqResult : WillNotProvision</code> apparaissent, vérifier deux points :</p>
<p><strong>1. Le compte a-t-il une méthode MFA dans le nouveau système unifié Entra ID ?</strong><br>
Vérifier via : <code>entra.microsoft.com</code> → Utilisateurs → [compte] → Méthodes d'authentification<br>
Les méthodes legacy (Authenticator enregistré via outlookMobile) peuvent ne pas suffire.<br>
Solution : enregistrer une méthode native via <code>aka.ms/mysecurityinfo</code>.</p>
<p><strong>2. Le compte est-il soumis à une policy MFA active ?</strong><br>
Un compte exclu de toute policy MFA ne peut pas s'enrôler en WHfB.<br>
WHfB nécessite une validation d'identité forte lors de l'enrôlement.</p>
</blockquote>

### 7.4 Conditional Access : exiger un MFA résistant au phishing

Une fois WHfB et FIDO2 déployés, créer une Conditional Access policy qui exige exclusivement une méthode résistante au phishing, ce qui supprime le fallback vers SMS ou TOTP.

<blockquote class="ok">
<p><strong>Périmètre licence : Entra ID P1, dans le périmètre Business Premium</strong></p>
<p>La fonctionnalité "Exiger la force de l'authentification" dans Conditional Access nécessite uniquement P1.</p>
</blockquote>

**Forces disponibles dans l'interface (libellés validés terrain, 24 août 2026) :**

- **Authentification multifacteur :** combinaisons qui répondent à une auth forte (Mot de passe + SMS, etc.)
- **Authentification multifacteur sans mot de passe :** Microsoft Authenticator, etc.
- **MFA anti-hameçonnage** (libellé dans la liste déroulante : `Phishing-resistant MFA`) : FIDO2, WHfB, CBA uniquement

Note : la liste déroulante affiche "Phishing-resistant MFA" en anglais même en interface française.

<blockquote class="red">
<p><strong>Prérequis obligatoire avant activation</strong></p>
<p>Chaque utilisateur soumis à cette policy doit avoir au moins une méthode résistante au phishing enregistrée. Sans cela, l'utilisateur sera bloqué. Activer d'abord en mode "Rapport uniquement" et vérifier les logs avant de passer en Activé. Minimum 2 semaines de monitoring recommandé.</p>
</blockquote>

**Policy 1 : comptes à privilèges (priorité maximale)**

`entra.microsoft.com` → Accès conditionnel → Stratégies → Nouvelle stratégie

1. Nom : CA-Phishing-Resistant-Admins
2. Utilisateurs : Rôles d'annuaire → Administrateur général, Administrateur de rôle privilégié, Administrateur de la sécurité (au minimum)
3. Ressources cibles : Toutes les ressources (anciennement « Toutes les applications cloud »)
4. Contrôles d'accès → Octroyer : Accorder l'accès → Exiger la force de l'authentification → Phishing-resistant MFA → Sélectionner
5. Activer : **Rapport uniquement** → Créer

<blockquote class="yellow">
<p><strong>Avertissement Microsoft lors de la création</strong></p>
<p>"Ne bloquez pas votre accès ! Cette stratégie affecte le Portail Azure." Ignorable en mode Rapport uniquement : le mode Rapport ne bloque jamais l'accès.</p>
</blockquote>

**Policy 2 : tous les utilisateurs (déploiement progressif)**

À déployer uniquement après confirmation que tous les utilisateurs ont enrôlé WHfB ou une passkey.

1. Nom : CA-Phishing-Resistant-AllUsers
2. Utilisateurs : tous les utilisateurs, en excluant le groupe break-glass
3. Ressources cibles : Toutes les ressources (ou applications critiques uniquement dans un premier temps)
4. Contrôles d'accès → Octroyer : Accorder l'accès → Exiger la force de l'authentification → Phishing-resistant MFA → Sélectionner
5. Activer : Rapport uniquement → minimum 2 semaines de monitoring → passer en Activé

<blockquote>
<p><strong>Ce que fait la policy en mode Activé : comportement exact</strong></p>
<p>La policy ne force pas l'enrôlement WHfB automatiquement. Elle exige qu'une méthode résistante au phishing soit utilisée pour s'authentifier.</p>
<p>· Utilisateur avec WHfB enrôlé : connexion transparente via WHfB, policy satisfaite.<br>
· Utilisateur avec passkey FIDO2 : connexion via passkey, policy satisfaite.<br>
· Utilisateur sans méthode résistante au phishing : accès bloqué, redirigé vers <code>aka.ms/mysecurityinfo</code>.</p>
<p>Les comptes anciens sans méthode résistante enregistrée seront bloqués. Prévoir une campagne d'enrôlement avant le passage en mode Activé.</p>
</blockquote>

<blockquote class="yellow">
<p><strong>Audit préalable : identifier les comptes non conformes</strong></p>
</blockquote>

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

<blockquote class="yellow">
<p><strong>Note : passkeys par défaut, puis retrait du SMS et de la voix</strong></p>
<p>À partir du 1er septembre 2026 (déploiement progressif), Microsoft active automatiquement les passkeys pour les utilisateurs configurés en SMS ou en voix et les invite à en enregistrer une lors de la connexion MFA. Le SMS et la voix fournis par Microsoft seront retirés le 1er février 2027 pour tous les utilisateurs, sauf les administrateurs généraux et les utilisateurs externes, pour qui la date est le 1er juillet 2027. Les invités internes suivent la date du 1er février.</p>
<p>Les organisations qui doivent conserver le SMS ou la voix peuvent configurer un fournisseur de téléphonie via Microsoft Security Store (disponible depuis le 30 octobre 2026, coûts à leur charge). Sinon, planifier la migration vers WHfB ou passkeys avant ces dates.</p>
</blockquote>

---

<a name="rotation"></a>

## 8. Rotation de la clé AzureADKerberos

### 8.1 Pourquoi et à quelle fréquence

| Cadence | Contexte | Remarque |
|---|---|---|
| Même calendrier que la rotation krbtgt de l'AD | Recommandation Microsoft | Microsoft n'impose pas d'intervalle fixe |
| Plus fréquente | Environnements sensibles | Choix de gouvernance à documenter (sécurité des données, art. 8 nLPD) |
| Immédiatement | Compromission avérée d'un compte Domain Admin | Procédure d'urgence |

### 8.2 Procédure de rotation standard

**Étape 1 : rotation**

```powershell
Set-AzureADKerberosServer -Domain $domain `
  -UserPrincipalName 'admin@[tenant].onmicrosoft.com' `
  -DomainCredential $domainCred -RotateServerKey
```

**Étape 2 (vérification immédiate)** : exécuter `Get-AzureADKerberosServer` (voir 7.1). `KeyVersion` doit être égal à `CloudKeyVersion`, et l'écart entre `KeyUpdatedOn` et `CloudKeyUpdatedOn` doit rester inférieur à 1h.

**Étape 3 (attente)** : ne lancer aucune nouvelle rotation avant au moins 10h (durée de vie par défaut d'un TGT Kerberos), idéalement 24h.

**Étape 4 (validation)** : tester une connexion WHfB sur un poste hybride pour confirmer que la rotation n'a pas cassé l'auth.

<blockquote>
<p><strong>Coordination avec la rotation du compte krbtgt AD</strong></p>
<p>L'objet AzureADKerberos et le compte krbtgt de l'AD sont deux objets distincts avec des clés indépendantes : leurs rotations sont des opérations séparées.</p>
<p>Rotation krbtgt AD : bonne pratique classique, recommandée tous les 60 à 90 jours, ou immédiatement après tout incident impliquant un accès Domain Admin non autorisé. La rotation krbtgt se fait en deux passes espacées de la durée de vie maximale du ticket Kerberos (10h par défaut) pour éviter de bloquer les sessions actives.</p>
<p>Ordre recommandé : krbtgt AD en premier (double rotation J+0 / J+10h minimum), puis AzureADKerberos après confirmation de stabilité. Espacer d'au moins 10 à 24h entre les deux. Aligner les deux cycles sur un calendrier commun, comme le recommande Microsoft, simplifie la gestion et réduit le risque d'oubli.</p>
</blockquote>

### 8.3 Automatisation via Azure Automation

Logique du runbook périodique :

1. Récupérer les credentials depuis Azure Key Vault (jamais en clair dans le script)
2. Exécuter `Set-AzureADKerberosServer -RotateServerKey`
3. Exécuter `Get-AzureADKerberosServer` → vérifier que `KeyVersion` = `CloudKeyVersion` et que l'écart entre `KeyUpdatedOn` et `CloudKeyUpdatedOn` est inférieur à 1h
4. Si écart supérieur à 1h : envoyer une alerte email / Teams → intervention manuelle requise
5. Logger le résultat dans Log Analytics pour audit trail

Droits requis : Hybrid Identity Administrator (Entra ID) + droits délégués Domain Admin (via Credential stocké dans Key Vault).

---

<a name="fido2"></a>

## 9. Annexe A : FIDO2 hardware pour comptes à privilèges

### 9.1 Quand préférer une clé physique à WHfB par TPM

| Critère | WHfB / TPM | Clé FIDO2 physique |
|---|---|---|
| Portabilité | Lié au poste | Portable entre postes |
| Coût | Zéro (intégré au poste) | 30 à 80 CHF / unité selon modèle |
| Durée de vie déclarée | Liée au poste (5 à 10 ans) | YubiKey : 5 ans minimum, typiquement 7 à 10 ans. Feitian : 5 ans déclarés. Limite pratique : perte physique ou changement de politique (connecteur USB certifié >10 000 insertions). |
| Déploiement | Via Intune, scalable | Distribution physique, manuelle |
| Perte / vol | Risque nul (clé dans TPM) | Risque physique : procédure de révocation nécessaire |
| Comptes recommandés | Tous les utilisateurs standard | Global Admin, break-glass, comptes PIM |

### 9.2 Activation FIDO2 dans Entra ID

`entra.microsoft.com` → Méthodes d'authentification → Stratégies → **Clé d'accès (FIDO2)**

Page : **Paramètres de clé d'accès (FIDO2)**

<blockquote class="yellow">
<p><strong>Important : FIDO2 est désactivé par défaut</strong></p>
<p>Contrairement à CAE, la méthode "Clé d'accès (FIDO2)" est désactivée par défaut sur tous les tenants. Une activation manuelle est requise.</p>
</blockquote>

**Onglet "Activer et cibler" :**

1. Toggle Activer → **Activé**
2. Onglet Inclure → Tous les utilisateurs (ou groupe spécifique selon la stratégie de déploiement)
3. Onglet Exclure → Ajouter le groupe break-glass

**Onglet "Configurer" :**

- Default passkey profile : Types = Device-bound, Synced, Restrictions = Non
- Pour les comptes à privilèges : créer un profil dédié avec Device-bound uniquement et AAGUIDs spécifiques YubiKey

<blockquote>
<p><strong>Note sur Microsoft Authenticator dans les AAGUIDs</strong></p>
<p>L'interface propose Windows Hello, Microsoft Authenticator et Enter AAGUID comme raccourcis. Microsoft Authenticator = passkey Synced stockée dans l'app mobile, synchronisée via Microsoft. Plus pratique mais légèrement moins robuste qu'une clé Device-bound pour les comptes ultra-sensibles. Pour Global Admin : préférer Device-bound (YubiKey ou TPM) uniquement.</p>
</blockquote>

Enrôlement : l'utilisateur enrôle sa clé ou passkey via `aka.ms/mysecurityinfo`.

---

<a name="conclusion"></a>

## 10. Conclusion : honnêteté intellectuelle sur le niveau de protection

Ce document ne rend pas un tenant Microsoft 365 inattaquable. Il déplace le curseur de difficulté pour l'attaquant.

| Niveau | Mesure | Ce que ça garantit |
|---|---|---|
| Base | MFA seul, sans CAE | Cookie volé exploitable jusqu'à révocation, puis jeton d'accès valide jusqu'à une heure |
| + CAE | CAE + Identity Protection | Révocation en quasi temps réel (jusqu'à 15 minutes selon Microsoft), à condition que la détection suive |
| + SIEM | Règles M2/M3/M5 à M8/M12/M14 + M-new-1 + M-new-2 | Visibilité sur les actions post-exploitation dans le tenant |
| + Purview Suite (add-on) | IRM Fuites-Données + eDiscovery Premium | Couverture SharePoint et constitution de preuves nLPD |
| + WHfB / FIDO2 | Protection structurelle (TPM + Cloud Kerberos Trust) | Attaque AiTM par proxy neutralisée (aucune session valide pour le proxy) |
| Posture complète | WHfB + rotation clé + audit legacy + SIEM + IRM | Résistance structurelle + gestion du risque résiduel |

Les autres vecteurs restent actifs indépendamment de WHfB : endpoint compromis (malware, keylogger), ingénierie sociale directe, compromission du DC on-prem. WHfB ferme le vecteur AiTM mais ne remplace pas une posture de sécurité globale.

*Prochaine étape recommandée : déployer CAE (vérification en 20 minutes) immédiatement, puis planifier WHfB comme projet de durcissement sur 1 à 2 sessions.*

---

<a name="siem"></a>

## 11. Annexe B : pourquoi un SIEM est indispensable, même gratuit

### 11.1 Le problème sans SIEM

Les IoC décrits dans ce document existent dans les journaux Entra ID, Exchange et Defender. Identity Protection envoie des alertes, mais elles arrivent dans un portail que personne ne surveille activement. Sans SIEM, la détection est manuelle, réactive et souvent trop tardive.

Un SIEM avec des règles de corrélation transforme ces signaux en alertes actionnables, en temps réel, vers les canaux que l'équipe IT surveille effectivement (email, Teams, SMS). C'est la différence entre découvrir une compromission 3 jours après les faits et réagir en 7 minutes.

### 11.2 Les options open source / gratuites pour une PME

| SIEM | Points forts | Points faibles | Intégration M365 |
|---|---|---|---|
| Wazuh | Très populaire, grande communauté, documentation abondante, agents endpoint natifs, XDR intégré | Configuration M365 manuelle et complexe, courbe d'apprentissage élevée | Via module Wazuh Office 365 (API Management Activity), configuration manuelle des règles |
| UTMStack CE | Interface intuitive, intégration O365 native, règles M-series validées en lab sur les scénarios AiTM et CVE-2026-69836, lab documenté disponible | Communauté plus petite, moins de ressources en ligne | Native O365 : règles M-series prêtes à importer, validées en lab |
| Graylog | Excellent pour la gestion de logs en volume, interface claire | Moins orienté SIEM, corrélation limitée en version CE | Via inputs personnalisés, nécessite développement des règles |

### 11.3 Recommandation pour une PME M365

Pour une PME de 5 à 25 utilisateurs sous M365 Business Premium, UTMStack CE offre le meilleur rapport déploiement / couverture immédiate pour les scénarios documentés dans ce guide.

<blockquote class="ok">
<p><strong>Lab UTMStack : règles M365 validées en lab</strong></p>
<p>Le lab UTMStack documenté sur GitHub Pages couvre l'ensemble des règles de corrélation M365 utilisées dans ce guide, avec les YAML prêts à importer, les tests de déclenchement documentés et les champs disponibles dans chaque alerte.</p>
<p>· <a href="https://doit4everyone.github.io/utmstack-lab/docs/11-correlations-yaml.html">Chapitre 11 : règles de corrélation M-series (dont M-new-1 et M-new-2)</a><br>
· <a href="https://doit4everyone.github.io/utmstack-lab/">Lab UTMStack complet</a></p>
</blockquote>

---

<a name="renvois"></a>

## 12. Renvois et ressources

| Document / Ressource | Périmètre licence | Lien |
|---|---|---|
| Guide MVC Purview nLPD, PME Suisse | Business Premium (sans add-on) | [doit4everyone.github.io](https://doit4everyone.github.io/Configuration-Purview-PME-Suisse-nLPD/) |
| Guide complet Purview 2026 : IRM (8.2), eDiscovery (8.3) | Business Premium + Purview Suite (add-on requis) | [Section 8 : Fonctionnalités avancées](https://doit4everyone.github.io/microsoft-purview-configuration-2026-nLPD/docs/09-fonctionnalites-avancees.html) |
| Guide Shadow AI / Gouvernance Copilot | Business Premium + Purview Suite (add-on requis) | [doit4everyone.github.io](https://doit4everyone.github.io/shadow-ai-governance-microsoft-365-nLPD/) |
| Lab UTMStack, chapitre 11 (règles M-new-1, M-new-2) | UTMStack CE (open source) | [11-correlations-yaml.html](https://doit4everyone.github.io/utmstack-lab/docs/11-correlations-yaml.html) |
| Lab UTMStack, accueil | UTMStack CE (open source) | [utmstack-lab](https://doit4everyone.github.io/utmstack-lab/) |
| Microsoft : Cloud Kerberos Trust | Business Premium (inclus) | [learn.microsoft.com](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-hybrid-cloud-kerberos-trust) |
| Microsoft : CAE | Business Premium (inclus) | [learn.microsoft.com](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-continuous-access-evaluation) |
| Microsoft : FIDO2 security keys | Business Premium (inclus) | [learn.microsoft.com](https://learn.microsoft.com/en-us/entra/identity/authentication/howto-authentication-passwordless-security-key) |
| Microsoft : CVE-2026-69836 advisory | N/A | [msrc.microsoft.com](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2026-69836) |
| The Hacker News : correction du statut d'exploitation de CVE-2026-69836 | N/A | [thehackernews.com](https://thehackernews.com/2026/08/microsoft-entra-id-flaw-cvss-100.html) |
| ANY.RUN : étude Mirage2FA | N/A | [any.run](https://any.run/cybersecurity-blog/mirage2fa-phishing-targets-us-companies/) |
| The Hacker News : Mirage2FA | N/A | [thehackernews.com](https://thehackernews.com/2026/08/mirage2fa-surge-hits-4500-us-and-eu.html) |
| Microsoft : retrait du SMS et de la voix (FAQ) | N/A | [learn.microsoft.com](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-sms-voice-retirement-faq) |

---

## 📥 Téléchargements

- [Version Word (.docx)]({{ '/downloads/docx/MVC-nLPD_Annexe-AiTM_v1.1.docx' | relative_url }})

---

[← Retour au bundle MVC nLPD](../) · [🏠 Retour au portail principal](https://doit4everyone.github.io/)

---

*ℹ️ Références, structuration et aide à la rédaction assistées par IA, avec validation humaine finale. Validation terrain effectuée sur infrastructure hybride réelle, août 2026. Version corrigée le 2 novembre 2026 : chiffres Mirage2FA, statut de la CVE, retrait SMS et voix, précisions techniques (CAE, Identity Protection, WHfB, Cloud Kerberos Trust, rotation, journaux).*
