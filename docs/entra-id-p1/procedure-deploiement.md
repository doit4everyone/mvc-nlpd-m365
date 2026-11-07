---
title: "Entra ID P1 : Procédure de déploiement MVC nLPD"
description: "Procédure de déploiement Entra ID P1 pour PME suisses (Microsoft 365 Business Premium) : comptes break-glass, MFA, accès conditionnel CA-01 à CA-08, SSPR, tests et checklist de remise. Conforme nLPD art. 8."
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

<h1>Entra ID P1 : Procédure de déploiement</h1>
<h2>MVC nLPD · Identité et accès · v3.4</h2>

> [← Retour au guide Entra ID P1](index.html) · [← Retour au bundle MVC nLPD](../index.html)

> **⚠️ ATTENTION** : remplacez `[tenant]` et `[Nom du client]` par les informations réelles du client avant toute utilisation.

## Sommaire

- [Préface](#preface)
- [Partie 0 : Prérequis](#partie-0)
- [Partie 1 : Méthodes d'authentification](#partie-1)
- [Partie 2 : Emplacements nommés](#partie-2)
- [Partie 3 : CA-01, MFA pour tous les utilisateurs](#partie-3)
- [Partie 4 : CA-02, Blocage de l'authentification héritée](#partie-4)
- [Partie 5 : CA-03, Filtrage géographique](#partie-5)
- [Partie 6 : CA-04, Token Protection](#partie-6)
- [Partie 7 : CA-05, Session limitée sur appareils non gérés](#partie-7)
- [Partie 8 : CA-06, Blocage des connexions hors horaires](#partie-8)
- [Partie 9 : CA-08, Exiger un appareil conforme](#partie-9)
- [Partie 10 : SSPR, Réinitialisation du mot de passe](#partie-10)
- [Partie 11 : Tests et validation](#partie-11)
- [Checklist de remise client](#checklist)

---

## Préface {#preface}

> **ℹ️ INFO** : ce guide est un guide nLPD (nouvelle Loi fédérale sur la Protection des Données, Suisse, RS 235.1). Délai de notification d'une violation : « dès que possible », pas de délai fixe de 72 heures comme dans le RGPD européen. Autorité compétente : PFPDT, pas la CNIL.

Ce guide déploie la couche **identité et accès** du bundle MVC nLPD. Il s'inscrit dans un déploiement en 3 sessions : Session 1 MVC Purview (4 à 6 h), Session 2 MVC Entra ID P1 (2 à 3 h, ce guide), Session 3 MVC Intune + Defender (3 à 5 h). Total : 9 à 14 heures, client autonome à la fin. Il est conçu pour être déployé avant ou en parallèle du MVC Intune + Defender.

La nLPD art. 8 exige des mesures techniques de protection adaptées au risque. Le contrôle des identités est la première de ces mesures : un mot de passe volé contourne tous les autres contrôles.

| Pilier | Ce que ça apporte |
|--------|-------------------|
| **Deux comptes break-glass** | Accès de secours exclu de toutes les politiques CA : évite le verrouillage total du tenant |
| **Emplacements nommés** | Définit la zone géographique légitime pour les règles de filtrage `[nLPD art. 8]` |
| **MFA obligatoire (CA-01)** | Première barrière contre la compromission de compte `[nLPD art. 8]` |
| **Blocage auth. héritée (CA-02)** | Ferme le contournement du MFA via IMAP, POP3, SMTP, EAS `[nLPD art. 8]` |
| **Filtrage géographique (CA-03)** | Bloque les connexions depuis des pays non autorisés `[nLPD art. 8]` |
| **Token Protection (CA-04)** | Bloque le vol de session post-MFA (attaques AiTM) `[nLPD art. 8]` |
| **Session limitée (CA-05)** | Restreint les sessions sur appareils non gérés `[nLPD art. 8]` |
| **Hors horaires (CA-06)** | Signale ou bloque les connexions hors heures ouvrables `[nLPD art. 8]` |
| **Appareil conforme (CA-08)** | Exige un appareil conforme Intune, en mode Rapport puis Application `[nLPD art. 8]` |
| **SSPR** | Réinitialisation autonome et auditée des mots de passe `[nLPD art. 8]` |

> **ℹ️ INFO** : ce guide est strictement limité à Entra ID P1, inclus dans Microsoft 365 Business Premium. Les fonctionnalités Entra ID P2 (Identity Protection, PIM, Access Reviews) sont hors périmètre MVC : elles exigent un add-on de licence supplémentaire (Defender Suite).

> **ℹ️ INFO : vocabulaire du portail.** Dans le formulaire d'une stratégie, l'état « Application » de ce guide correspond au réglage **Activé** de « Activer une stratégie » (les autres choix sont « Rapport uniquement » et « Désactivé »). Les contrôles se règlent sous « Contrôles d'accès » : **Octroyer** (accorder ou bloquer) et **Session**.

> **ℹ️ INFO : dates et statuts vérifiés le 6 novembre 2026** sur la documentation et le centre de messages Microsoft. Les calendriers Microsoft évoluent : revérifiez les dates marquées « 2026 » avant chaque déploiement.

> **ℹ️ INFO** : il n'existe pas de politique CA-07 dans le MVC. La numérotation CA-01 à CA-06 puis CA-08 est conservée pour rester identique dans tous les guides du bundle (le nom `CA-08` est référencé dans le guide Intune + Defender).

---

## Partie 0 : Prérequis {#partie-0}

### 0.1 Portail et rôles

| N° | Étape | Action |
|----|-------|--------|
| 1 | Navigateur | Microsoft Edge ou Chrome. Évitez Firefox pour les portails admin. |
| 2 | URL | https://entra.microsoft.com |
| 3 | Connexion | Compte admin M365 (Administrateur général ou Administrateur de la sécurité). |
| 4 | Rôles requis | Administrateur de la sécurité → politiques CA, méthodes d'authentification. Administrateur de l'authentification → SSPR, méthodes, TAP. Administrateur de stratégie d'authentification → Authentication Strength. |

> **⚠️ ATTENTION** : après ajout d'un rôle, déconnectez-vous et reconnectez-vous. Les rôles ne prennent pas effet dans la session en cours.

### 0.2 Deux comptes break-glass

> **🚨 IMPORTANT** : créez les deux comptes break-glass AVANT d'activer toute politique CA. Sans eux, une mauvaise configuration peut vous bloquer définitivement hors du tenant, sans recours possible. Microsoft recommande deux comptes avec des méthodes d'authentification différentes.

| N° | Étape | Action |
|----|-------|--------|
| 1 | Compte BG-1 | entra.microsoft.com → Utilisateurs → Tous les utilisateurs → Nouvel(le) utilisateur(-trice) → Créer un utilisateur (onglet Informations de base : Nom d'utilisateur principal, Domaine, Nom d'affichage, Mot de passe). Nom d'utilisateur principal : `bga1@[tenant].onmicrosoft.com` (en production, utilisez des noms neutres de type humain, par exemple `prenom.nom@[tenant].onmicrosoft.com`, pour masquer la fonction du compte). Domaine : toujours `@[tenant].onmicrosoft.com` (cloud-only). Méthode MFA : FIDO2 (clé physique stockée en coffre). Pas de licence assignée. Mot de passe de 25 caractères ou plus. |
| 2 | Compte BG-2 | Même principe, nom neutre différent. Méthode MFA : Authenticator sur appareil dédié hors réseau. Mot de passe différent de BG-1. |
| 3 | Groupe break-glass | entra.microsoft.com → Groupes → Tous les groupes → Nouveau groupe. Type de groupe : Sécurité. Nom de groupe : `GRP-CA-BreakGlass`. Type d'appartenance : Affecté. Membres : Aucun membre sélectionné (lien) → ajouter bga1 + bga2 → Créer. |
| 4 | Enregistrement MFA | Enregistrer au moins une méthode MFA forte (FIDO2 ou CBA) sur chaque compte break-glass. Connexion test bga1 dans une fenêtre privée → accès au portail Entra ID. |

> **⚠️ ATTENTION : MFA des portails admin.** Microsoft impose le MFA au niveau des portails admin indépendamment des politiques CA : portails Azure, Entra et Intune depuis octobre 2024 ; centre d'administration Microsoft 365 depuis février 2025 (déploiement progressif) ; Azure CLI, Azure PowerShell, API REST, SDK et outils d'infrastructure as code depuis le 1er octobre 2025 (opérations de création, modification et suppression). Le report possible jusqu'au 1er juillet 2026 est expiré : l'application est désormais effective sur tous les tenants. Les comptes break-glass restent exclus de toutes les CA, mais une invite MFA portail apparaîtra. C'est pourquoi FIDO2 ou CBA doit être enregistré : sans méthode MFA, le compte break-glass ne peut plus accéder aux portails.

> **⚠️ ATTENTION : test trimestriel.** Testez les deux comptes chaque trimestre. Changez les mots de passe tous les 90 jours. Documentez dans le registre de sécurité.

### 0.3 Désactiver Security Defaults

> **⚠️ ATTENTION** : Security Defaults et l'accès conditionnel ne peuvent pas coexister. Si Security Defaults est activé, désactivez-le avant de créer des politiques CA. Les deux systèmes en conflit provoquent des comportements imprévisibles.

| N° | Étape | Action |
|----|-------|--------|
| 1 | Accès | entra.microsoft.com → Vue d'ensemble → Propriétés → descendre jusqu'à la section « Paramètres de sécurité par défaut ». |
| 2 | Désactiver | Toggle : Désactiver. Motif : Utilisation de l'accès conditionnel (P1). |
| 3 | Enregistrer | Cliquez Enregistrer. |

> **ℹ️ INFO** : si des politiques d'accès conditionnel existent déjà, le portail n'affiche pas l'interrupteur et indique : « Votre organisation utilise actuellement des stratégies d'accès conditionnel qui vous empêchent d'activer les paramètres de sécurité par défaut ». Security Defaults est alors déjà désactivé, rien à faire.

> **ℹ️ INFO** : si Security Defaults était actif, les utilisateurs déjà enregistrés en MFA gardent leur enregistrement. Procédez directement à la configuration des politiques CA.

---

## Partie 1 : Méthodes d'authentification {#partie-1}

### 1.1 Activer les méthodes recommandées

**Chemin** : entra.microsoft.com → Entra ID → Méthodes d'authentification → Stratégies

| N° | Étape | Action |
|----|-------|--------|
| 1 | Microsoft Authenticator | Cliquer → Activer → Tous les utilisateurs → Enregistrer. Méthode privilégiée : passkeys FIDO2 disponibles dans l'app depuis 2026. |
| 2 | Clé d'accès (FIDO2) | Cliquer → Activer → Tous les utilisateurs → Autoriser les passkeys synchronisés (Authenticator app) + liés à l'appareil (clés physiques YubiKey). Enregistrer. |
| 3 | SMS | Activer → Tous les utilisateurs → Enregistrer. Secours si l'application est inaccessible. |
| 4 | Appel vocal | Optionnel. Utile pour les utilisateurs sans smartphone compatible. |

> **ℹ️ INFO : migration des stratégies héritées.** Depuis le 30 septembre 2025, les anciennes stratégies MFA et SSPR sont déconnectées et les paramètres sont gérés par ces stratégies unifiées. La page affiche un bandeau « État de la migration » : terminez la migration (lien « changer ») si elle est « En cours ». Le portail peut aussi annoncer l'activation automatique des clés d'accès pour les utilisateurs qui utilisent le SMS ou l'appel vocal.

> **⚠️ ATTENTION : MFA per-user legacy.** Vérifiez qu'aucun utilisateur n'est en statut « Appliqué » (Enforced) dans l'ancienne interface MFA : admin.microsoft.com → Utilisateurs actifs → Authentification multifacteurs. Tous doivent être en « Désactivé ». Les deux systèmes en conflit provoquent des blocages inexpliqués.

### 1.2 Passe d'accès temporaire (TAP)

**Chemin** : Entra ID → Méthodes d'authentification → Stratégies → Passe d'accès temporaire (la page de réglages s'intitule « Paramètres Droit d'accès temporaire »)

| N° | Étape | Action |
|----|-------|--------|
| 1 | Activer | Onglet « Activer et cibler » → Activer : interrupteur sur Activé. Cible : Tous les utilisateurs. |
| 2 | Durée | Onglet « Configurer » → Modifier. Durée de vie minimale : 1 heure. Durée de vie maximale : 8 heures. Durée de vie par défaut : 1 heure (valeurs par défaut du portail). |
| 3 | Usage unique | Onglet « Configurer » → Une seule fois : Oui (recommandé pour l'onboarding). La valeur par défaut est Non. Longueur : 8 caractères (par défaut). |
| 4 | Enregistrer | Cliquer Enregistrer. |

> **ℹ️ INFO** : le TAP est le moyen recommandé pour enregistrer le premier facteur MFA (passkey ou FIDO2) lors de l'onboarding. Émission : entra.microsoft.com → Utilisateurs → [utilisateur] → Méthodes d'authentification → + Ajouter une méthode d'authentification → Passe d'accès temporaire (chemin à confirmer sur le portail).

### 1.3 Points forts d'authentification (Authentication Strength)

**Chemin** : Entra ID → Méthodes d'authentification → Points forts d'authentification (également accessible par Accès conditionnel → Points forts d'authentification)

| N° | Étape | Action |
|----|-------|--------|
| 1 | 3 niveaux intégrés | **Authentification multifacteur (niveau 1)** : combinaisons de méthodes qui répondent à une authentification forte, par exemple mot de passe + SMS. Utilisée par CA-01. **Authentification multifacteur sans mot de passe (niveau 2)** : méthodes sans mot de passe, comme Microsoft Authenticator. **MFA anti-hameçonnage (niveau 3)** : méthodes sans mot de passe résistantes au hameçonnage, comme la clé de sécurité FIDO2. |
| 2 | Vérification | Confirmer que les 3 niveaux intégrés (colonne Type : Intégré) sont visibles. Dans la liste, le portail peut les afficher sous leur nom anglais : Multifactor authentication, Passwordless MFA, Phishing-resistant MFA. Le détail d'une force s'ouvre en cliquant sur son nom (« Afficher la force de l'authentification »). Aucune création nécessaire sauf besoin spécifique. |

### 1.4 Campagne d'inscription

**Chemin** : Entra ID → Méthodes d'authentification → Campagne d'inscription

| N° | Étape | Action |
|----|-------|--------|
| 1 | État | Valeur par défaut observée : « Managé par Microsoft » (Microsoft ajuste les paramètres de la campagne). Les champs Méthode d'authentification, « Nombre de jours autorisés pour la répétition » et « Nombre limité de répétitions » sont grisés dans cet état. |
| 2 | Périmètre | Inclure : Tous les utilisateurs. Exclure → Sélectionner un groupe → `GRP-CA-BreakGlass`. |
| 3 | Enregistrer | Cliquer Enregistrer. |

> **ℹ️ INFO** : la campagne d'inscription invite (rappel après connexion) les utilisateurs à configurer Microsoft Authenticator ou une clé d'accès (FIDO2). Elle ne force pas l'enregistrement : c'est CA-01 qui l'impose. Les utilisateurs doivent avoir la méthode activée dans les stratégies pour être éligibles au rappel.

> **✅ CONSEIL** : envoyez un email aux utilisateurs AVANT l'activation de CA-01 avec le lien d'enregistrement : https://aka.ms/mfasetup. Sans préavis, ils seront bloqués à leur prochaine connexion.

> **⚠️ ATTENTION : CA pendant l'enregistrement.** Depuis le 13 juillet 2026 (déploiement du 6 au 13 juillet 2026, avis Microsoft MC1326253), les politiques CA ciblant l'action utilisateur « Inscrire des informations de sécurité » s'appliquent aussi pendant le provisionnement de Windows Hello Entreprise et de macOS Platform SSO. Le MVC ne crée pas de politique sur cette action : aucun impact. Si le client en ajoute une, testez-la en mode Rapport avant application, sous peine de bloquer l'enregistrement des nouveaux collaborateurs.

---

## Partie 2 : Emplacements nommés {#partie-2}

> **🇨🇭 nLPD art. 8 al. 1 : contextualisation géographique des accès.** La nLPD art. 8 exige des mesures techniques adaptées au risque. Définir les zones géographiques légitimes d'accès permet de bloquer les connexions depuis des pays à risque et de conditionner le MFA selon la localisation. Les emplacements nommés sont la base de la politique CA-03.

> **⚠️ ATTENTION** : créez les emplacements nommés AVANT les politiques CA qui les référencent. Un emplacement inexistant invalide la politique CA.

### 2.1 Pays autorisés

**Chemin** : entra.microsoft.com → Entra ID → Accès conditionnel → Emplacements nommés → + Emplacement des pays

| N° | Étape | Action |
|----|-------|--------|
| 1 | Nom | `Pays-Autorises-[Client]` |
| 2 | Méthode | Méthode de recherche de pays : « Déterminer l'emplacement avec l'adresse IP (IPv4 et IPv6) » (valeur par défaut). |
| 3 | Pays | Cocher : CH (Suisse). Ajouter selon les besoins réels du client : DE, FR, IT, AT, LI. Documenter chaque pays ajouté avec la justification métier. |
| 4 | Pays inconnus | Case « Inclure des pays/régions inconnus » : laisser DÉCOCHÉE. Les pays non listés seront bloqués par CA-03. |
| 5 | Créer | Cliquer Créer. |

### 2.2 Bureau (IP fixe)

**Chemin** : Emplacements nommés → menu « … » de la barre d'outils → + Emplacement des plages d'adresses IP

| N° | Étape | Action |
|----|-------|--------|
| 1 | Nom | `Bureau-[Client]` |
| 2 | Marqué approuvé | Cocher : Marquer comme emplacement approuvé. |
| 3 | Plage IP | Cliquer sur le bouton « + », puis saisir l'IP fixe du bureau en CIDR (exemple : 203.0.113.10/32). Demander à l'opérateur si nécessaire. |
| 4 | Créer | Cliquer Créer. |

> **⚠️ ATTENTION** : si le client n'a pas d'IP fixe, sautez cet emplacement. CA-01 s'appliquera avec MFA depuis le bureau, sans exception.

---

## Partie 3 : CA-01, MFA pour tous les utilisateurs {#partie-3}

> **🇨🇭 nLPD art. 8 al. 1 : première mesure de protection contre la compromission de compte.** La nLPD art. 8 exige des mesures techniques de protection proportionnelles au risque. Le MFA est la mesure la plus efficace contre la compromission de compte par credential stuffing : attaque automatisée testant des millions de mots de passe volés. Sans MFA, un mot de passe volé suffit à contourner toute la protection Purview.

> **🚨 IMPORTANT** : exigez le MFA sur le compte admin AVANT d'activer CA-01 en mode Application. Sans cela, CA-01 peut déclencher une boucle d'authentification sur le compte admin et bloquer l'accès au portail.

| N° | Étape | Action |
|----|-------|--------|
| C1 | Mode rapport | Activer CA-01 en mode Rapport uniquement pendant 7 jours. Vérifier dans les journaux (Surveillance → Journaux de connexion → colonne « Résultat de la politique CA ») que la politique s'applique aux bons utilisateurs. |
| 1 | Nom | `CA-01-MFA-Tous-Utilisateurs` |
| 2 | Utilisateurs → Inclure | Tous les utilisateurs. |
| 3 | Utilisateurs → Exclure | `GRP-CA-BreakGlass`. |
| 4 | Ressources cibles | Toutes les ressources (anciennement « Toutes les applications cloud »). |
| 5 | Contrôles d'accès → Octroyer | Accorder l'accès → cocher « Exiger la force de l'authentification » → choisir « Authentification multifacteur » → Sélectionner. Ne pas cocher en plus « Exiger une authentification multifacteur » : le portail refuse de combiner les deux. |
| 6 | Activer une stratégie | Rapport uniquement → 7 jours → puis Activé. |
| 7 | Créer | Cliquer Créer. |

> **✅ CONSEIL : test.** Connexion avec un compte utilisateur standard en navigation privée → l'invite MFA apparaît. Connexion bga1 → pas d'invite MFA de la politique CA (compte exclu).

---

## Partie 4 : CA-02, Blocage de l'authentification héritée {#partie-4}

> **🇨🇭 nLPD art. 8 al. 1 : fermeture du vecteur de contournement du MFA.** Les protocoles IMAP, POP3, SMTP AUTH et Exchange ActiveSync basique ne supportent pas l'accès conditionnel : ils contournent donc le MFA. Un compte avec CA-01 actif reste entièrement vulnérable si l'authentification héritée n'est pas bloquée.

| N° | Étape | Action |
|----|-------|--------|
| C1 | Sûreté | Application directe : les applications M365 modernes (Outlook, Teams, SharePoint, Outlook mobile) utilisent OAuth 2.0 et ne sont pas affectées. |
| 1 | Nom | `CA-02-Blocage-Auth-Heritee` |
| 2 | Utilisateurs → Inclure | Tous les utilisateurs. |
| 3 | Utilisateurs → Exclure | `GRP-CA-BreakGlass`. |
| 4 | Ressources cibles | Toutes les ressources (anciennement « Toutes les applications cloud »). |
| 5 | Conditions → Applications clientes | Configurer : Oui → cocher, sous « Clients d'authentification hérités » : Clients Exchange ActiveSync + Autres clients (IMAP, POP3, SMTP legacy). Laisser Navigateur et Applications mobiles et clients de bureau décochés. |
| 6 | Contrôles d'accès → Octroyer | Bloquer l'accès → Sélectionner. |
| 7 | Activer une stratégie | Activé → Créer. |

> **✅ CONSEIL : test.** Tentative de connexion IMAP ou Outlook 2010 avec un compte utilisateur → erreur « Accès bloqué par les politiques d'accès conditionnel ».

---

## Partie 5 : CA-03, Filtrage géographique {#partie-5}

> **🇨🇭 nLPD art. 8 al. 1 : réduction de la surface d'attaque géographique.** La grande majorité des attaques de credential stuffing provient de pays tiers. Bloquer les connexions depuis des pays non autorisés réduit significativement l'exposition, sans impact sur les utilisateurs légitimes connectés depuis la Suisse et les pays voisins configurés.

| N° | Étape | Action |
|----|-------|--------|
| C1 | Mode rapport | 14 jours en mode Rapport. Vérifier que personne de légitime ne serait bloqué, notamment les collaborateurs en déplacement. |
| 1 | Nom | `CA-03-Blocage-Pays-Non-Autorises` |
| 2 | Utilisateurs → Inclure | Tous les utilisateurs. |
| 3 | Utilisateurs → Exclure | `GRP-CA-BreakGlass`. |
| 4 | Ressources cibles | Toutes les ressources (anciennement « Toutes les applications cloud »). |
| 5 | Réseau (ou Conditions → Emplacements) → Configurer : Oui → Inclure | N'importe quel réseau ou emplacement. |
| 6 | Réseau → Exclure | Réseaux et emplacements sélectionnés → `Pays-Autorises-[Client]`. |
| 7 | Contrôles d'accès → Octroyer | Bloquer l'accès → Sélectionner. |
| 8 | Activer une stratégie | Rapport uniquement → 14 jours → puis Activé. |
| 9 | Créer | Cliquer Créer. |

> **⚠️ ATTENTION : avant d'activer.** Si des collaborateurs se connectent régulièrement depuis des pays non listés (déplacements professionnels), ajoutez ces pays à `Pays-Autorises-[Client]` avant de passer en Application.

---

## Partie 6 : CA-04, Token Protection {#partie-6}

> **🇨🇭 nLPD art. 8 al. 1 : protection contre le vol de session post-MFA.** Les attaques AiTM contournent le MFA en interceptant le cookie de session. Token Protection lie cryptographiquement le token au TPM de l'appareil, rendant un cookie volé inutilisable depuis un autre appareil. Prérequis : WHfB doit être actif sur les postes (configuré via MVC Intune) pour que la liaison TPM soit opérationnelle. Pour le contexte de l'attaque : [annexe AiTM / Cookie Hijacking](../aitm-cookie-hijacking/index.html).

> **⚠️ ATTENTION : conditions réelles pour une PME cloud-only (périmètre MVC).**
> Token Protection supporte les applications natives (Outlook, Teams, OneDrive, Word, Excel, PowerPoint, OneNote, Edge pour la connexion au profil uniquement, etc.) sur Windows, avec Exchange Online, SharePoint Online et Teams comme ressources. Dans un navigateur, seule une préversion limitée existe (Windows et macOS, pour certaines applications web qui accèdent à Azure Resource Manager) : les applications web courantes ne sont pas protégées, et rien n'est supporté dans un navigateur sur iOS/iPadOS. Côté Apple (macOS 14+, iOS/iPadOS 16+), la page de documentation Microsoft consultée le 7 novembre 2026 reste incohérente : son tableau indique la disponibilité générale pour les applications natives, alors qu'une autre section parle de « préversion ». Les appareils doivent être gérés par MDM, avec le plug-in Microsoft Enterprise SSO (ou Platform SSO sur macOS) ; les appareils Apple non gérés ne sont pas supportés. Ne pas activer en Application sans test. Les applications natives Mail et Calendrier d'Apple ne supportent pas Token Protection : les utilisateurs seraient bloqués.
> Types d'enregistrement Windows non supportés (accès bloqué, code `signInSessionStatusCode` 1003 dans les journaux) : appareils déployés par inscription en masse (bulk enrollment), Windows Autopilot en mode self-deploying, Cloud PC Windows 365 joints à Entra, hôtes de session Azure Virtual Desktop joints à Entra. Excluez-les avec un filtre d'appareil.
> En environnement hybride (Hybrid Entra Join + AD on-prem), le Cloud Kerberos Trust est requis en plus : hors périmètre MVC, voir l'[annexe AiTM](../aitm-cookie-hijacking/index.html) §6.
> Dans une PME suisse, la majorité des accès passe par navigateur ou appareils Apple : Token Protection ne protège pas ces vecteurs.

| N° | Étape | Action |
|----|-------|--------|
| C1 | Prérequis stricts | WHfB activé via MVC Intune (intune.microsoft.com → Appareils → Inscription → Windows → Windows Hello Entreprise, politique à l'échelle du tenant). En environnement cloud-only, c'est la seule étape requise : pas de Cloud Kerberos Trust, pas d'AzureADKerberos. Le Cloud Kerberos Trust n'est requis qu'en hybride (Hybrid Entra Join + AD on-prem), hors périmètre MVC. Parc Windows uniquement, TPM 2.0 actif, aucun accès navigateur ni appareil Apple dans le périmètre. |
| C2 | Déployer en 2 temps | Phase 1 (ce guide) : créer CA-04 en Rapport uniquement, groupe pilote Direction + admins. Phase 2 (après MVC Intune) : activer en Application uniquement après que WHfB est déployé et validé sur les postes (`dsregcmd /status` → `NgcSet : YES` + `TpmProtected : YES`) et que 100 % des appareils dans le périmètre sont Windows avec TPM 2.0. |
| 1 | Nom | `CA-04-Token-Protection-Exchange-SharePoint` |
| 2 | Utilisateurs → Inclure | Groupe pilote (Direction + admins). Ne pas inclure tous les utilisateurs avant validation terrain. |
| 3 | Utilisateurs → Exclure | `GRP-CA-BreakGlass`. |
| 4 | Ressources cibles | Office 365 Exchange Online + Office 365 SharePoint Online uniquement (pas « Toutes les ressources »). |
| 5 | Session | Contrôles d'accès → Session → cocher : Exiger une protection par jeton pour les sessions de connexion. |
| 6 | Activer une stratégie | Rapport uniquement → Créer. Passer en Application après MVC Intune et confirmation WHfB opérationnel (`dsregcmd /status` → `NgcSet : YES` + `TpmProtected : YES` sur tous les postes dans le périmètre). |

> **✅ CONSEIL : test en Rapport.** Journaux de connexion → filtrer sur CA-04 → colonne « Résultat CA ». Si des navigateurs ou appareils Apple apparaissent : ajustez le périmètre avant tout passage en Application.

---

## Partie 7 : CA-05, Session limitée sur appareils non gérés {#partie-7}

> **🇨🇭 nLPD art. 8 al. 1 : contrôle des accès depuis les appareils personnels.** Un appareil personnel non géré peut être compromis, partagé ou perdu sans que l'organisation en soit informée. Limiter les sessions SharePoint et OneDrive sur ces appareils (lecture seule, pas de téléchargement) réduit le risque d'exfiltration de données personnelles.

| N° | Étape | Action |
|----|-------|--------|
| C1 | Prérequis SharePoint | Centre admin SharePoint → Stratégies → Partage → Accès depuis des appareils non gérés → Autoriser un accès limité au web uniquement. Ce paramètre doit être actif pour que CA-05 soit efficace. |
| 1 | Nom | `CA-05-Session-Limitee-Appareils-Non-Geres` |
| 2 | Utilisateurs | Inclure : Tous les utilisateurs. Exclure : `GRP-CA-BreakGlass`. |
| 3 | Ressources cibles | Office 365 SharePoint Online. |
| 4 | Conditions → Filtre pour les appareils | Configurer : Oui → Inclure les appareils filtrés dans la stratégie → Syntaxe de la règle → Modifier : `device.isCompliant -eq False AND device.trustType -ne "ServerAD"` |
| 5 | Session | Contrôles d'accès → Session → cocher : Utiliser les restrictions appliquées par l'application. |
| 6 | Activer une stratégie | Rapport uniquement → 7 jours → puis Activé. |
| 7 | Créer | Cliquer Créer. |

> **✅ CONSEIL : test.** Connexion SharePoint depuis un appareil non géré → accès en lecture seule uniquement, bouton de téléchargement absent.

---

## Partie 8 : CA-06, Blocage des connexions hors horaires {#partie-8}

> **🇨🇭 nLPD art. 8 al. 1 : réduction de la surface d'attaque temporelle.** Les compromissions post-AiTM ou post-phishing se produisent statistiquement hors heures de bureau. CA-06 signale (mode Rapport) ou bloque les connexions en dehors des heures ouvrables.

> **⚠️ ATTENTION** : la condition temporelle n'est pas disponible dans l'interface graphique Entra ID : elle se configure via Microsoft Graph API (endpoint beta, inclus dans Entra ID P1). Excluez impérativement `GRP-CA-BreakGlass` avant toute activation en Application.

> **ℹ️ INFO : note nLPD.** Cette politique doit être documentée dans le registre des activités de traitement comme contrôle d'accès système (et non évaluation des employés), communiquée aux utilisateurs dans la politique de sécurité IT, et appliquée de manière proportionnée au risque identifié.

| N° | Étape | Action |
|----|-------|--------|
| C1 | Plage couverte | Lundi-vendredi de 07h00 à 19h00 CEST (été) ou CET (hiver). Week-end entier bloqué. En UTC : 05h00-17h00 en été (CEST = UTC+2), 06h00-18h00 en hiver (CET = UTC+1). Adapter le corps JSON selon la saison. |
| C2 | Exclusions | `GRP-CA-BreakGlass` impérativement. Exclure aussi les comptes de service si des traitements nocturnes légitimes existent (synchronisation Entra Connect, sauvegardes automatisées). |
| 1 | Créer la politique dans Entra ID | entra.microsoft.com → Accès conditionnel → + Nouvelle stratégie. Nom : `CA-06-Blocage-Hors-Horaires`. Utilisateurs : Tous, exclure `GRP-CA-BreakGlass`. Ressources : Toutes les ressources. Octroyer : Bloquer l'accès. Activer une stratégie : Rapport uniquement → Créer. |
| 2 | Récupérer l'ID de la politique | Graph Explorer (https://aka.ms/ge) → `GET https://graph.microsoft.com/beta/identity/conditionalAccess/policies` → rechercher `CA-06-Blocage-Hors-Horaires` → copier l'ID (format UUID). |
| 3 | PATCH Graph API | `PATCH https://graph.microsoft.com/beta/identity/conditionalAccess/policies/{ID}` avec le corps JSON ci-dessous. La condition exclut les heures ouvrables (lun-ven 05h00-17h00 UTC) : tout le reste (nuits et week-end) est couvert par la politique. |

**Corps JSON (été, CEST)**

```json
{
  "conditions": {
    "times": {
      "includeAllTimes": true,
      "excludeDays": {
        "daysOfWeek": ["monday", "tuesday", "wednesday", "thursday", "friday"],
        "timeZone": "UTC",
        "startTime": "05:00:00",
        "endTime": "17:00:00",
        "allDay": false
      }
    }
  }
}
```

> **✅ CONSEIL : test en Rapport.** Journaux de connexion → filtrer sur CA-06 → colonne « Résultat CA ». Tester une connexion le week-end ou après 19h00 : doit apparaître « Rapport - Échec ». Une connexion en heure ouvrable doit passer normalement.

> **✅ CONSEIL : test SIEM (UTMStack).** La règle UTMStack M14 (Sign-In Blocked by Conditional Access Policy) détecte les blocages CA via `log.LogonError = BlockedByConditionalAccess`. Corréler avec M3 (Impossible Travel) si le blocage hors horaires coïncide avec un pays inhabituel.

---

## Partie 9 : CA-08, Exiger un appareil conforme {#partie-9}

> **🇨🇭 nLPD art. 8 al. 1 : contrôle d'accès par état de sécurité de l'appareil.** Un appareil non géré ou non conforme (BitLocker inactif, antivirus absent, version OS obsolète) représente un risque d'exfiltration et de compromission. CA-08 bloque l'accès aux ressources M365 depuis tout appareil ne répondant pas aux critères de conformité Intune.

> **⚠️ ATTENTION** : cette politique est créée ici en mode Rapport uniquement. Elle ne sera activée en Application qu'après le déploiement du MVC Intune + Defender, une fois les politiques de conformité déployées et les appareils en statut « Conforme ».

| N° | Étape | Action |
|----|-------|--------|
| C1 | Déployer en 2 temps | Phase 1 (ce guide) : créer CA-08 en Rapport uniquement. Phase 2 (après MVC Intune) : passer en Application une fois les appareils conformes dans Intune. |
| 1 | Nom | `CA-08-Exiger-Appareil-Conforme` |
| 2 | Utilisateurs → Inclure | Tous les utilisateurs. |
| 3 | Utilisateurs → Exclure | `GRP-CA-BreakGlass`. |
| 4 | Ressources cibles | Toutes les ressources (anciennement « Toutes les applications cloud »). |
| 5 | Contrôles d'accès → Octroyer | Accorder l'accès → cocher « Exiger que l'appareil soit marqué comme conforme » → Sélectionner. |
| 6 | Activer une stratégie | Rapport uniquement → Créer. Passer en Application après déploiement Intune (voir MVC Intune + Defender, Partie 12). |

---

## Partie 10 : SSPR, Réinitialisation du mot de passe {#partie-10}

> **🇨🇭 nLPD art. 8 al. 1 : traçabilité des réinitialisations et continuité d'accès.** Le SSPR garantit que les réinitialisations de mots de passe sont auditées (journal Entra ID) : traçabilité exigée par la nLPD art. 8. Il réduit aussi les appels helpdesk post-activation MFA, fréquents lors des premiers jours de déploiement.

### 10.1 Activer SSPR

**Chemin** : entra.microsoft.com → Entra ID → Réinitialisation de mot de passe → Propriétés

| N° | Étape | Action |
|----|-------|--------|
| 1 | Statut | Réinitialisation de mot de passe libre-service : Sélectionné (pas Tout, pour exclure les comptes de service). |
| 2 | Groupe | + Sélectionner le groupe → créer `GRP-SSPR-Utilisateurs` → ajouter tous les utilisateurs standards. |
| 3 | Enregistrer | Cliquer Enregistrer. |

### 10.2 Méthodes et notifications

**Chemin** : Réinitialisation de mot de passe → Méthodes d'authentification

| N° | Étape | Action |
|----|-------|--------|
| 1 | Nombre de méthodes requises | 2 |
| 2 | Méthodes | Activer : Notification d'application mobile, Téléphone mobile, E-mail. Ne pas activer « Questions de sécurité » : le portail annonce leur mise hors service en mars 2027. |
| 3 | Enregistrer | Cliquer Enregistrer. |
| 4 | Notifications | Réinitialisation de mot de passe → Notifications. « Notifier les utilisateurs lors des réinitialisations de mot de passe ? » : Oui. « Notifier tous les administrateurs quand d'autres administrateurs réinitialisent leur mot de passe ? » : Oui (la valeur par défaut observée est Non). Enregistrer. |

> **ℹ️ INFO : environnement hybride.** En environnement AD on-prem avec Entra Connect : Réinitialisation de mot de passe → Intégration locale → cocher « Activer la réécriture du mot de passe pour les utilisateurs synchronisés ». La page affiche aussi l'état de l'agent de synchronisation Microsoft Entra Connect (« Configuration terminée » attendu) et la case « Voulez-vous autoriser les utilisateurs à déverrouiller les comptes sans réinitialiser leur mot de passe ? ». Sans la réécriture, le mot de passe est réinitialisé dans le cloud mais pas dans l'AD local.

> **✅ CONSEIL : test.** https://aka.ms/sspr → réinitialisation avec 2 méthodes → succès → notification reçue → connexion avec le nouveau mot de passe.

> **⚠️ ATTENTION : changement SSPR de novembre 2026.** Microsoft arrête de prendre en compte les coordonnées issues de l'annuaire (téléphone mobile, téléphone professionnel, email alternatif) pour vérifier l'identité lors d'une réinitialisation : seules les méthodes explicitement enregistrées par l'utilisateur (https://aka.ms/mfasetup ou mysignins.microsoft.com) sont acceptées. Un utilisateur sans méthode enregistrée ne peut plus réinitialiser son mot de passe seul. L'application, d'abord annoncée pour le 7 septembre 2026, a été repoussée par Microsoft à **début novembre 2026**, avec un déploiement de début à mi-novembre (les messages Microsoft indiquent le 7 ou le 9 novembre selon l'endroit). La campagne d'enregistrement Microsoft a démarré le 5 octobre 2026. Vérifiez la couverture avant et après : entra.microsoft.com → Méthodes d'authentification → Détails d'inscription des utilisateurs.

---

## Partie 11 : Tests et validation {#partie-11}

| Test | Scénario | Résultat attendu |
|------|----------|------------------|
| **T1** | MFA déclenché (CA-01) | Navigation privée → connexion utilisateur standard → invite MFA → connexion réussie après MFA. |
| **T2** | Break-glass non bloqué | Connexion bga1 → invite MFA portail (normale depuis 2025) mais pas de blocage CA-01 (compte exclu). FIDO2 ou Authenticator requis pour passer le MFA portail. Connexion bga2 → idem. |
| **T3** | Auth héritée bloquée (CA-02) | Client IMAP ou Outlook 2010 → erreur « Accès bloqué par les politiques d'accès conditionnel ». |
| **T4** | Pays bloqué (CA-03) | VPN vers un serveur hors `Pays-Autorises-[Client]` → connexion bloquée. |
| **T5** | Session limitée (CA-05) | Appareil non géré → SharePoint → accès en lecture seule, pas de téléchargement. |
| **T6** | SSPR | https://aka.ms/sspr → 2 méthodes → nouveau mot de passe → notification reçue. |
| **T7** | Journaux de connexion | entra.microsoft.com → Surveillance → Journaux de connexion → politiques CA visibles dans la colonne « Résultat de la politique CA ». |

---

## Checklist de remise client {#checklist}

| ☐ | Action | Vérification | nLPD |
|---|--------|--------------|------|
| ☐ | Deux comptes break-glass créés (bga1 + bga2, méthodes différentes) | entra.microsoft.com → Utilisateurs | |
| ☐ | Groupe `GRP-CA-BreakGlass` créé (bga1 + bga2) | entra.microsoft.com → Groupes | |
| ☐ | Security Defaults désactivé | entra.microsoft.com → Propriétés | |
| ☐ | Authenticator + FIDO2 + SMS activés | Entra ID → Méthodes d'auth | art. 8 |
| ☐ | Passe d'accès temporaire activée (usage unique, 1 h par défaut) | Entra ID → Méthodes → Passe d'accès temporaire | |
| ☐ | Campagne d'inscription active (tous les utilisateurs, sauf break-glass) | Entra ID → Méthodes → Campagne d'inscription | |
| ☐ | Emplacement `Pays-Autorises-[Client]` créé | CA → Emplacements nommés | art. 8 |
| ☐ | CA-01 MFA : Application (après 7 j de rapport) | CA → Stratégies → CA-01 | art. 8 |
| ☐ | CA-02 Auth héritée : Application directe | CA → Stratégies → CA-02 | art. 8 |
| ☐ | CA-03 Filtrage géo : Application (après 14 j de rapport) | CA → Stratégies → CA-03 | art. 8 |
| ☐ | CA-04 Token Protection : Rapport uniquement (Application après MVC Intune) | CA → Stratégies → CA-04 | art. 8 |
| ☐ | CA-05 Session limitée : Application | CA → Stratégies → CA-05 | art. 8 |
| ☐ | CA-06 Hors horaires : Rapport uniquement | CA → Stratégies → CA-06 | art. 8 |
| ☐ | CA-08 Appareil conforme : Rapport uniquement (Application après MVC Intune) | CA → Stratégies → CA-08 | art. 8 |
| ☐ | SSPR activé pour `GRP-SSPR-Utilisateurs` (2 méthodes) | Entra ID → Réinit. MDP | art. 8 |
| ☐ | Test T1 MFA déclenché validé | Navigation privée | |
| ☐ | Test T2 Break-glass non bloqué validé | Connexion bga1 | |
| ☐ | Test T3 Auth héritée bloquée validé | Client IMAP | |
| ☐ | Test T6 SSPR fonctionnel validé | https://aka.ms/sspr | |

> [Annexes A à F : glossaire, checklist complète, références nLPD, récapitulatif des politiques, adaptations sectorielles, POC vs production →](annexes.html)

---

[🏠 Retour au portail principal](https://doit4everyone.github.io/)

---

ℹ️ *Références, structuration et aide à la rédaction assistées par IA, avec validation humaine finale.*
