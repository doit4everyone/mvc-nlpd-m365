---
title: "AiTM / Cookie Hijacking M365: Detection, Mitigation and Structural Protection | DoIt4Everyone"
description: "Complete technical guide to AiTM attacks against Microsoft 365. CAE, WHfB, FIDO2, Cloud Kerberos Trust, CA Authentication Strength. Field-validated on a hybrid infrastructure, August 2026. Corrected November 2, 2026."
lang: en
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

🌐 English version · [Version française](./)

[← Back to the MVC nLPD bundle](../)

# AiTM / Cookie Hijacking M365

<p style="text-align:center;color:#2E74B5;font-size:1.3em;font-weight:bold;">Detection · Mitigation · Structural protection</p>

<p style="text-align:center;color:#555;">Standalone technical guide · Microsoft 365 Business Premium · v1.1 · August 2026, corrected on November 2, 2026<br>Field-validated on a hybrid infrastructure (Entra Connect + on-prem AD + WS2025)</p>

---

<blockquote class="warn">
<p><strong>Publication context</strong></p>
<p>This document was written following two major publications in August 2026: the ANY.RUN research on Mirage2FA and the Microsoft advisory on CVE-2026-69836. The two topics are separate: nothing indicates that Mirage2FA exploits this flaw.</p>
<p><strong>Mirage2FA (LinX Coders group, ANY.RUN research of August 18, 2026):</strong> a PhaaS platform using an AiTM technique based on a WebSocket proxy against Microsoft 365, active from 2024 to 2026. According to ANY.RUN estimates: 9,426 addresses targeted, 4,532 potentially compromised (48%), and 4,561 session cookie theft events, the most common outcome.</p>
<p><strong>CVE-2026-69836 (Microsoft advisory of August 20):</strong> remote code execution vulnerability (deserialization), CVSS 10.0, in Entra ID. It was fixed on Microsoft's infrastructure side, with no action required on the customer side. Microsoft initially reported active exploitation, then corrected its advisory on August 21, 2026 and confirmed that the vulnerability had not been exploited in the wild.</p>
<p>Audience: IT consultants / MSPs and in-house IT managers supporting SMEs on M365 Business Premium.</p>
<p>This document is standalone but complements the <a href="https://doit4everyone.github.io/Configuration-Purview-PME-Suisse-nLPD/">MVC Purview nLPD Guide (in French)</a>.</p>
</blockquote>

---

**Table of contents**

**Part A: Plan B, immediately actionable**

- [1. Context: MFA is no longer enough](#contexte)
- [2. CAE: Continuous Access Evaluation](#cae)
- [3. Indicators of compromise (IoCs)](#ioc)
- [4. Immediate response: SIEM alert received](#reponse)

**Part B: Plan A, structural protection**

- [5. Why WHfB / FIDO2 defeats the AiTM attack](#whfb)
- [6. Prerequisites and preliminary audit](#prerequis)
- [7. Deployment: Cloud Kerberos Trust and AzureADKerberos](#deploiement)
- [8. Rotating the AzureADKerberos key](#rotation)
- [9. Annex A: hardware FIDO2 for privileged accounts](#fido2)
- [10. Conclusion](#conclusion)
- [11. Annex B: why a SIEM is essential, even a free one](#siem)
- [12. References and resources](#renvois)

---

<div class="partie-banner">
<strong>PART A · Plan B: immediately actionable</strong><br>
<em>CAE · IoC detection · SIEM · Immediate response</em>
</div>

<a name="contexte"></a>

## 1. Context: MFA is no longer enough

### 1.1 What is an AiTM attack?

A modern AiTM (Adversary-in-the-Middle) attack no longer tries to break MFA. It lets MFA work normally and intercepts what comes after: the authenticated session cookie.

The Mirage2FA mechanism illustrates this approach perfectly:

1. **Phishing:** The user receives an email with a link, an attachment (.htm, .xhtml, .svg) or a QR code leading to a transparent WebSocket proxy, visually identical to login.microsoft.com.
2. **Transparent proxy:** The proxy relays every request to Microsoft in real time. The user completes their MFA normally.
3. **Interception:** The proxy captures the authenticated session cookie. The attacker now holds a valid cookie.
4. **Replay:** The attacker imports the cookie into their browser and accesses M365 without credentials or MFA. A password reset is not enough: the cookie stays valid until it is explicitly revoked.

<blockquote class="red">
<p><strong>Mirage2FA: ANY.RUN research figures (August 18, 2026, campaign from 2024 to 2026)</strong></p>
<p>9,426 addresses targeted · 4,532 Microsoft 365 accounts potentially compromised (48%)<br>
4,561 session cookie theft events (2,541 unique victims): the most common outcome, about 48.9% of the 9,332 potential compromise events recorded<br>
ANY.RUN estimates: potential impact, not individually confirmed compromises<br>
Vector: WebSocket proxy, undetectable by the user<br>
Insufficient countermeasure: password reset alone</p>
</blockquote>

### 1.2 CVE-2026-69836: when the vulnerability is in the infrastructure itself

On August 20, 2026, Microsoft published the CVE-2026-69836 advisory, a deserialization of untrusted data vulnerability in Entra ID (remote code execution, CVSS score 10.0). The fix was deployed on Microsoft's infrastructure side: no patching action is required on the customer side. Microsoft initially reported active exploitation, then corrected its advisory on August 21, 2026 and confirmed that the vulnerability had not been exploited in the wild. Nothing indicates that Mirage2FA exploits it: its mechanism relies on a phishing proxy.

Even though Microsoft reports no exploitation, the prudent question is not "is it fixed?" but "did anything happen in my tenant before the fix?"

<blockquote class="yellow">
<p><strong>Retrospective audit recommended as a precaution: period before August 20, 2026</strong></p>
<p>· New Global Admin / Privileged Role Admin role assignments<br>
· Credentials added to existing service principals<br>
· Changes to Conditional Access policies<br>
· Unusual OAuth consents<br>
· New MFA methods registered on privileged accounts<br>
· Cloud accounts created outside the synchronization process</p>
<p><strong>Limit:</strong> Entra ID audit and sign-in logs are kept for 30 days (P1 and P2). Beyond that, without routing them to Log Analytics or a storage account via Azure Monitor, they are no longer accessible. The Microsoft 365 audit log (Purview Audit Standard, including Exchange events) has its own retention: 180 days by default.<br>
→ Permanent recommendation: route Entra ID logs to Log Analytics.</p>
<p>Full retrospective audit procedure: see section 4.4.</p>
</blockquote>

### 1.3 Why classic MFA is structurally not enough

MFA protects the authentication but not the session that results from it. Once the session cookie is issued, it is valid proof of identity in the eyes of Microsoft 365, regardless of the authentication method used to obtain it.

| Measure | Effectiveness against AiTM | Remark |
|---|---|---|
| MFA (TOTP / SMS) | Insufficient | The proxy relays the MFA in real time |
| MFA (Authenticator push) | Insufficient | The user approves, the proxy intercepts |
| Password reset | Insufficient | The cookie stays valid after the reset |
| CAE (Continuous Access Evaluation) | Partial | Near real-time revocation (up to 15 minutes according to Microsoft) for compatible clients |
| Identity Protection (detailed detections: Entra ID P2) | Partial | Flags a replay from an unusual context (IP, country, anomalous token) |
| WHfB (key in the TPM) or FIDO2 passkey | Structural | The AiTM proxy cannot obtain a valid session, see Part B |

---

<a name="cae"></a>

## 2. CAE: Continuous Access Evaluation

### 2.1 Principle, token lifetime and nuances

CAE changes two aspects of the token lifecycle: their lifetime and their revocability.

| Token type | Lifetime | Revocability | AiTM scenario |
|---|---|---|---|
| Standard token (client not CAE-capable) | 1 hour by default | Revocation blocks new tokens, the access token already issued stays valid until it expires | Up to ~1h of access after revocation |
| CAE token (client AND resource both CAE-capable) | Up to 28 hours | Revocable by critical event (revocation, account disabled, password change, high user risk) | Longer window if undetected, near real-time revocation if detected (up to 15 minutes of propagation according to Microsoft) |

<blockquote class="warn">
<p><strong>The CAE paradox, from the attacker's side</strong></p>
<p>A CAE token can last up to 28 hours instead of one hour. For an attacker who steals a CAE token via AiTM, the potential exploitation window is therefore much longer than with a standard token, if nobody detects the incident.</p>
<p>The CAE model only holds if detection and revocation are fast. Without a SIEM or active monitoring, CAE can be an advantage for the attacker. This is why detection (SIEM + Identity Protection) is critical.</p>
</blockquote>

M365 applications supporting CAE on the server side: Exchange Online, SharePoint Online, OneDrive, Microsoft Teams, MS Graph. On the client side, CAE requires a compatible client. This was validated in the lab with Microsoft Edge on SharePoint Online. Firefox and the Outlook web app (outlook.office.com) did not show a CAE token in our tests.

<blockquote class="ok">
<p><strong>CAE: enabled by default, revocation validated live (August 23, 2026)</strong></p>
<p>According to Microsoft, critical event evaluation does not depend on a Conditional Access policy and is available in any tenant. On our lab tenant, CAE was active with no configuration: the option visible in Conditional Access (Customize continuous access evaluation) is used to disable or tighten it. Some older tenants that configured CAE in the legacy interface must migrate that setting to Conditional Access.</p>
<p>Measured in the lab on a Business Premium tenant (Microsoft states a latency of up to 15 minutes):<br>
· Active session revocation in ~1 minute (myaccount.microsoft.com)<br>
· ~3 to 4 minutes on SharePoint Online via Edge (CAE token confirmed, Is CAE Token = Yes)</p>
<p>Revocation works on all token types (CAE and standard) via "Revoke sessions" in Entra ID.</p>
</blockquote>

<blockquote>
<p><strong>Identity Protection: P1 vs P2 difference (source: Microsoft Learn)</strong></p>
<p><strong>Business Premium without add-on (P1):</strong><br>
· Only a few detections (anonymous IP address, leaked credentials, Microsoft Entra threat intelligence)<br>
· Other detections appear under the generic "Additional risk detected" label, without details<br>
· No risk-based Conditional Access (automatic revocation on detection)<br>
· Manual revocation only, after detection via the SIEM or manual monitoring</p>
<p><strong>Business Premium + Defender Suite add-on (P2):</strong><br>
· Detailed detections: Anomalous token, Unfamiliar sign-in properties, Malicious IP address, Token issuer anomaly<br>
· Impossible travel: also requires Defender for Cloud Apps (or Microsoft 365 E5)<br>
· Risk-based Conditional Access: automatic revocation when a risk is detected</p>
<p>In both cases: manual revocation works immediately via "Revoke sessions".</p>
</blockquote>

### 2.2 Checking that CAE is not disabled on the tenant

**Via PowerShell:**

```powershell
Connect-MgGraph -Scopes "Policy.Read.All" -UseDeviceAuthentication

$result = Invoke-MgGraphRequest -Method GET `
  -Uri "https://graph.microsoft.com/beta/identity/conditionalAccess/policies"
$result.value | Where-Object {
  $_.sessionControls.continuousAccessEvaluation -ne $null
} | Select-Object displayName, @{N='CAE';E={$_.sessionControls.continuousAccessEvaluation.mode}}
```

**Expected result:** no line returned = no Conditional Access policy disables or customizes CAE. ✅

This script queries the Graph beta API, which may change. If in doubt, the portal check below is authoritative.

**Via the portal:**

`entra.microsoft.com` → Conditional Access → Policies: check that no policy contains Session → Customize continuous access evaluation → Disable.

<blockquote>
<p><strong>What you see in the interface</strong></p>
<p>In Conditional Access → New policy → Session, the "Customize continuous access evaluation" option offers only two sub-options: Disable and Strictly enforce location policies (preview). There is no "Enable" option: CAE is enabled by default without any action.</p>
</blockquote>

### 2.3 Revocation test

1. Sign in with a test account on `myaccount.microsoft.com` in a private browsing window.
2. `entra.microsoft.com` → Users → [test account] → Overview → **Revoke sessions** → Yes.
3. Measure the delay before the session is invalidated.

**Result validated live (August 23, 2026):** session invalidated in ~1 minute. Confirmation toast (translated from the French UI): "Sign-in sessions revoked for [account]".

For a native CAE test: use SharePoint Online via Edge. CAE token confirmed (`Is CAE Token = Yes` in the Entra ID sign-in logs). Revocation takes effect at the client's next network call (page refresh).

---

<a name="ioc"></a>

## 3. Indicators of compromise (IoCs)

### 3.1 Signals in Identity Protection (details: Entra ID P2)

`entra.microsoft.com` → Identity Protection → Risk detections

The detections below require Entra ID P2. Without P2 (Business Premium alone), they appear under the generic "Additional risk detected" label, without details.

| Signal | Description | Relevance |
|---|---|---|
| Impossible travel | Sign-in from two incompatible geographies within a short time (also requires Defender for Cloud Apps) | Partial: only if the attacker replays from a geographically inconsistent IP |
| Unfamiliar sign-in properties | New IP, new user-agent, new country | Good: the attacker's infrastructure differs from the usual profile |
| Anomalous token | Token with unusual characteristics (abnormal lifetime, replay from an unfamiliar location) | Direct: the most specific signal for a replayed cookie |
| Malicious IP address | Sign-in from a known malicious IP | Depends on the freshness of the Microsoft TI feed |

### 3.2 Post-compromise behavioral signals

These signals appear within the first 30 minutes after a successful session theft. Monitor them in `security.microsoft.com` → Incidents & alerts.

- **Mail forwarding rules:** creation of Outlook rules forwarding to an external address
- **Mass SharePoint downloads:** abnormally high download volume over a short period
- **Mailbox delegation added:** Full Access or Send As delegation added on a mailbox
- **New MFA method registered:** the attacker tries to establish persistence
- **Unusual OAuth consent:** third-party application obtaining permissions on the tenant
- **Unusual user-agent:** AiTM infrastructure often uses a generic or slightly malformed UA

### 3.3 On-prem signals (hybrid environments)

- **Event ID 4624 LogonType 3:** network logon from an unrecognized IP after a successful cloud auth
- **Event ID 4648:** explicit logon with alternate credentials, an indicator of lateral movement

### 3.4 SIEM correlation: what you see depending on your UTMStack configuration

Reference: [Chapter 11 of the UTMStack lab (in French)](https://doit4everyone.github.io/utmstack-lab/docs/11-correlations-yaml.html)

**Block 1: Rules that work without extra configuration**

| UTMStack rule | Signal covered | Scenario |
|---|---|---|
| M2: OAuth Application Consent Granted | Suspicious OAuth consent after compromise | AiTM + Entra ID compromise (CVE-2026-69836 scenario) |
| M3: Impossible Travel | Sign-in from a geographically impossible country | AiTM |
| M5: MFA Required Interrupt | MFA interrupt visible even if the proxy relayed the token | AiTM |
| M6: Member Added to Group | Addition to an Entra ID group | Entra ID compromise (CVE-2026-69836 scenario) |
| M7: User Account Created Outside Sync | Cloud account created outside Entra Connect | Entra ID compromise (CVE-2026-69836 scenario) |
| M8: Unified Audit Log Disabled | Attempt to blind the SIEM after compromise | AiTM + Entra ID compromise (CVE-2026-69836 scenario) |
| M12: External Mailbox Access | External access to an Exchange mailbox via OAuth | AiTM |
| M14: Sign-In Blocked by CA | Sign-in blocked by Conditional Access | AiTM |

**Block 2: Two additional rules for the CVE-2026-69836 scenario**

| Rule | Entra ID action detected | Validation |
|---|---|---|
| M-new-1: Privileged Role Assigned | `Add member to role.` → Global Admin, Security Admin, etc. | ✅ Validated live: August 22, 2026 (latency ~7 min) |
| M-new-2: Credentials Added to Service Principal | `Add service principal credentials.` → persistence without MFA | ✅ Validated live: August 22, 2026 (latency ~4 min) |

<blockquote>
<p><strong>M-new-1 and M-new-2: fields available in the UTMStack alert</strong></p>
<p><strong>M-new-1:</strong> <code>action = "Add member to role."</code> · <code>log.ObjectId</code> = UPN of the assigned account · <code>log.ModifiedProperties</code>: Role.DisplayName · <code>origin.user</code> = actor · latency ~7 min</p>
<p><strong>M-new-2:</strong> <code>action = "Add service principal credentials."</code> · <code>log.Target</code> = service principal name · <code>origin.user</code> = actor · latency ~4 min</p>
<p><strong>Note:</strong> M-new-2 only triggers via a direct Graph API call (<code>addPassword</code>) or PowerShell. The Entra ID graphical interface generates a different label. Test: <code>POST https://graph.microsoft.com/v1.0/servicePrincipals/{id}/addPassword</code> via <a href="https://developer.microsoft.com/en-us/graph/graph-explorer">Graph Explorer</a>.</p>
</blockquote>

**Block 3: Visibility limits**

<blockquote class="red">
<p><strong>Structural blind spots: independent of any tool or license</strong></p>
<p>· CVE-2026-69836, initial phase: a flaw of this kind sits inside Microsoft's infrastructure. No event is generated in the customer logs, even though Microsoft now reports no exploitation. Only the subsequent actions inside the tenant are visible, which is what M-new-1 and M-new-2 cover.<br>
· Log retention: Entra ID audit and sign-in logs are kept for 30 days (P1 and P2). Beyond that, without routing them to Log Analytics or a storage account via Azure Monitor, they are no longer accessible. The Microsoft 365 audit log (Purview Audit Standard, including Exchange events) has its own retention: 180 days by default.<br>
→ Permanent recommendation: route Entra ID logs to Log Analytics.</p>
</blockquote>

<blockquote class="yellow">
<p><strong>UTMStack blind spot: technical limitation of the ETL v11 pipeline</strong></p>
<p>SharePoint exfiltration: FileDownloaded events (RecordType 6) are not collected. Rule M13 is documented as not implementable (see Chapter 11).</p>
<p>Partial coverage is available through Microsoft Purview Insider Risk Management (policy IRM-Fuites-Données-PME). It requires the Purview Suite add-on, outside the scope of Business Premium alone.</p>
<p>Recommended best practices before enabling IRM (with regard to art. 26 OLT 3, the Swiss Ordinance 3 to the Labour Act, and art. 6 nFADP, the revised Swiss Federal Act on Data Protection, known as nLPD in French):<br>
· Prior notification of employees in the staff regulations or the IT charter<br>
· Anonymization enabled by default in Purview<br>
· Written procedure for lifting anonymity (see Annex G of the full Purview guide)</p>
<p>Reference: <a href="https://doit4everyone.github.io/microsoft-purview-configuration-2026-nLPD/docs/09-fonctionnalites-avancees.html">Full Purview 2026 guide (in French), section 8.2</a></p>
</blockquote>

---

<a name="reponse"></a>

## 4. Immediate response: SIEM alert received

<blockquote class="red">
<p><strong>Critical point: when a replay alert comes in, the theft has usually already happened</strong></p>
<p>Post-authentication signals (replay, OAuth consent, mailbox rule) usually mean the attacker already holds the session cookie. The goal becomes limiting the exploitation time. An earlier signal (a click on a known phishing link, for example) can however allow action before the theft.</p>
<p>· Without CAE: revocation blocks new tokens, but the access token already issued remains usable until it expires (one hour by default).<br>
· With CAE: revocation applies in near real time to compatible clients, with a latency of up to 15 minutes according to Microsoft (1 to 4 minutes measured in the lab).</p>
<p><strong>CAE must be enabled BEFORE the incident for the response to be effective. See section 2.</strong></p>
</blockquote>

### 4.1 Sequence of signals to watch in the SIEM

| Phase | Typical delay | SIEM signals (UTMStack rules) | Interpretation |
|---|---|---|---|
| Phase 1: initial access | T+0 to T+5 min | M3 (Impossible Travel), M5 (MFA Interrupt) | The cookie has just been stolen and replayed. Act immediately. |
| Phase 2: exploitation | T+5 to T+30 min | M2 (OAuth Consent), M12 (External Mailbox Access) | The attacker establishes persistence and starts exfiltrating. |
| Phase 3: long-term persistence | T+30 min to several hours | M-new-1 (Role Assigned), M-new-2 (Credentials SPN), M7 (Account Created) | The attacker prepares durable access, independent of the cookie. |

<blockquote class="red">
<p><strong>Golden rule</strong></p>
<p>If you see Phase 2 or Phase 3 without having seen Phase 1 in the SIEM, the initial attack took place outside your detection window. Revocation is required immediately and the damage of Phase 2 may already be done. Run the full audit of section 4.3.</p>
</blockquote>

### 4.2 Immediate token revocation: do this within 2 minutes

1. **Revoke all sessions:** `entra.microsoft.com` → Users → [compromised account] → Overview → Revoke sessions → Yes. PowerShell equivalent, useful in scripts: `Revoke-MgUserSignInSession -UserId [UPN]`. It is the same operation: no need to do both.
2. **Check CAE propagation:** If CAE is enabled, test a SharePoint / Teams action on the affected account. The session should be invalidated within a few minutes (up to 15 minutes according to Microsoft).

<blockquote class="yellow">
<p><strong>Without CAE: what to do until the access token expires (one hour by default)?</strong></p>
<p>Immediately tell the affected user to stop doing anything in their session. Monitor in real time the actions in Exchange (mail forwarding), SharePoint (downloads), Teams (messages sent) and Entra ID (configuration changes). Document every observed action with a timestamp to build the incident file.</p>
</blockquote>

### 4.3 Damage audit: do this within 30 minutes

**Microsoft 365 audit log (Exchange events)**

`purview.microsoft.com` → Audit → filter on [incident date/time] → [compromised account]

- `Set-InboxRule` or `New-InboxRule`: mail forwarding rules created
- `Add-MailboxPermission`: mailbox delegations added

**Entra ID audit logs**

`entra.microsoft.com` → Audit logs → filter on [incident date/time] → [compromised account]

- `Consent to application`: OAuth applications consented
- `Update user` on authentication methods: MFA methods modified
- `Add member to role`: role assignments
- `Add service principal credentials`: SPN credentials added

**Entra ID sign-in logs**

`entra.microsoft.com` → Sign-in logs → filter on [compromised account] → incident period

Look for: unusual source IP, generic or malformed user-agent, inconsistent geolocation.

### 4.4 Retrospective audit after CVE-2026-69836

If the audit reveals suspicious actions in the window before August 20, 2026, apply the revocation sequence on the affected accounts, then specifically check:

- `entra.microsoft.com` → Roles & admins → check each Global Admin / Security Admin / Privileged Role Admin role
- `entra.microsoft.com` → Enterprise applications → Certificates & secrets → recently added secrets
- `entra.microsoft.com` → Conditional Access → check recent policy changes

<blockquote class="purple">
<p><strong>Building the evidence file for FDPIC notification (art. 24 nFADP)</strong></p>
<p>If the audit reveals an actual compromise, eDiscovery Premium lets you search all potentially exposed data in Exchange, SharePoint and Teams and build the evidence file for the notification to the FDPIC (Federal Data Protection and Information Commissioner, PFPDT in French).</p>
<p>Requires the Purview Suite add-on, outside the scope of Business Premium alone.<br>
Reference: <a href="https://doit4everyone.github.io/microsoft-purview-configuration-2026-nLPD/docs/09-fonctionnalites-avancees.html">Full Purview 2026 guide (in French), section 8.3</a></p>
</blockquote>

### 4.5 Full remediation sequence

<blockquote class="red">
<p><strong>Common mistake: resetting the password first</strong></p>
<p>The session cookie stays valid after a password reset. The sequence below must be run in the order given.</p>
</blockquote>

1. Revoke all sessions (portal or PowerShell, see 4.2)
2. Audit mail forwarding rules (Exchange Admin Center)
3. Audit mailbox delegations (Full Access / Send As)
4. Audit recently consented OAuth applications
5. Audit registered MFA methods: remove any unrecognized device
6. Reset the password: only after steps 1 to 5
7. Force a full MFA re-registration
8. Document the incident: timestamp, source IP, user-agent, detected actions

---

<div class="partie-banner">
<strong>PART B · Plan A: structural protection</strong><br>
<em>WHfB with TPM · Cloud Kerberos Trust · Legacy applications · AzureADKerberos rotation</em>
</div>

<blockquote>
<p><strong>Scope note</strong></p>
<p>This part covers advanced hardening measures, beyond the MVC Business Premium scope. Audience: IT consultants supporting SMEs with Hybrid Entra Join or full-cloud workstations. Field validation was performed on a hybrid environment: Entra Connect + on-prem AD + WS2025.</p>
</blockquote>

<a name="whfb"></a>

## 5. Why WHfB / FIDO2 defeats the AiTM attack

### 5.1 The cryptographic mechanism

Windows Hello for Business uses an asymmetric key pair generated at enrollment:

- **Private key:** stored in the TPM (if the TPM is required, see 7.2), non-exportable, never leaves the device
- **Public key:** registered in Entra ID at enrollment

At Windows sign-in, the Windows cloud authentication provider (Cloud AP) requests a nonce from Entra ID, signs it with the private key and returns it. Entra ID verifies the signature with the public key, then issues a Primary Refresh Token (PRT) bound to the device. Authentication is performed by Windows, not by typing anything into a web page.

FIDO2 passkeys (physical security key or passkey) rely on a different mechanism, WebAuthn: the passkey is bound to the domain it was registered for, and the browser only offers it to that domain.

<blockquote class="ok">
<p><strong>Why the AiTM attack fails with WHfB / FIDO2</strong></p>
<p>With WHfB, there is no password or code to type into a web page: the proxy has nothing to relay. With a FIDO2 passkey, the proxy's domain is not the one the passkey was registered for: the browser does not offer it. In both cases, the proxy obtains no valid session. Microsoft lists these methods in the "Phishing-resistant MFA" authentication strength (see 7.4).</p>
<p>This protection is only complete if Conditional Access requires a phishing-resistant method: as long as a weaker method remains allowed, the proxy can offer it to the user. Nor does it protect against the theft of a cookie already issued by other means (malware on the device, for example).</p>
</blockquote>

### 5.2 Comparison of deployment scenarios

| Workstation type | Account type | Solution | AzureADKerberos required? | Difficulty |
|---|---|---|---|---|
| Full cloud (Entra ID only) | Cloud-only or hybrid | FIDO2 / WHfB direct | No | Low |
| Hybrid Entra Join | Hybrid (AD-synced) | WHfB + Cloud Kerberos Trust | Yes | Medium |
| Hybrid Entra Join | Cloud-only | Not compatible | N/A | Impossible: no Windows session possible |
| Pure AD (no Entra Join) | On-prem AD only | Out of WHfB scope | Not applicable | N/A |

<blockquote class="yellow">
<p><strong>Critical point: cloud-only account on a hybrid workstation</strong></p>
<p>A cloud-only account cannot open a Windows session on a Hybrid Entra Join workstation. Windows tries to authenticate via Kerberos against the on-prem AD first. The account does not exist in the AD, so the sign-in fails.</p>
<p>Validated in the lab (August 24, 2026): test-m7 (cloud-only) refused on a hybrid workstation. Solution: use a hybrid account (synced via Entra Connect) on hybrid workstations.</p>
</blockquote>

---

<a name="prerequis"></a>

## 6. Prerequisites and preliminary audit

### 6.1 Technical prerequisites

| Prerequisite | Detail | Status to check |
|---|---|---|
| Entra ID P1 | Included in M365 Business Premium | ✅ Within the MVC scope |
| Windows 10 21H2+ / Windows 11 | On the workstations targeted by WHfB | 📋 Inventory to be done |
| TPM 2.0 | TPM chip enabled in the BIOS/UEFI (firmware or discrete) | 📋 Device Manager → Security devices |
| Hybrid Entra Join | Entra Connect configured on the hybrid workstations | 📋 Entra ID → Devices → confirm the status |
| Hybrid accounts | Cloud-only accounts cannot open a Windows session on a hybrid workstation | 📋 Check the source of the user accounts |
| Windows Server 2016+ DCs | For Cloud Kerberos Trust in hybrid, with enough writable DCs in each site where users authenticate | 📋 Check the DC version and distribution |
| Non-privileged accounts | Direct or indirect members of privileged built-in AD groups (Domain Admins, for example) cannot use Cloud Kerberos Trust | 📋 Test with a standard user account |
| Global Admin + Domain Admin rights | To create the AzureADKerberos object | 📋 Available for the duration of the operation |
| PowerShell 5.1 | The AzureADHybridAuthenticationManagement module does not work in PowerShell 7 | 📋 Check via `$PSVersionTable.PSVersion` |

<blockquote>
<p><strong>Note: privileged accounts and shared workstations</strong></p>
<p>Privileged accounts: the password replication policy of the AzureADKerberos object blocks high-privilege accounts, as for an RODC. Microsoft advises against relaxing this policy to unblock them, because of possible attack vectors from Entra ID to AD. A deployment test run with an administrator account will fail: use a standard account.</p>
<p>Shared workstations: Microsoft limits the number of WHfB enrollments to 10 per device. For devices used by more than 10 people, or for users who sign in to many devices (a support technician, for example), Microsoft recommends FIDO2 security keys.</p>
</blockquote>

<blockquote class="yellow">
<p><strong>Preliminary check: existing WHfB GPOs in a hybrid environment</strong></p>
<p>Active Directory WHfB GPOs take precedence over Intune policies on hybrid workstations. Check that no WHfB GPO is already in place before deployment:</p>
</blockquote>

```powershell
Get-GPO -All | Where-Object {
  (Get-GPOReport -Guid $_.Id -ReportType XML) -match "PassportForWork"
}
```

### 6.2 Legacy application audit: mandatory step

Before any WHfB deployment, identify the applications likely to regress. A deployment without an audit can block users on critical business tools.

**Quick audit method:**

1. Inventory of apps that interactively ask for Windows credentials
2. Identification of NTLM flows: Event ID 4624 LogonType 3 + AuthPackage NTLM in the DC logs
3. List of network shares / NAS accessible via Windows auth
4. Inventory of RDP sessions to servers without Remote Credential Guard
5. Check of Windows Credential Manager on the pilot workstations

### 6.3 Legacy applications: risks and mitigation

| Application type | Risk after WHfB | Mitigation |
|---|---|---|
| Apps interactively asking for the Windows password | Blocked: the user no longer knows their password | Migrate to modern auth or keep a documented fallback password |
| Applications forcing NTLM | Authentication failure or credentials prompt | Configure SPN and Kerberos on the target server |
| Misconfigured WIA (missing SPN) | Falls back to NTLM, fails | Create the missing SPNs, check the negotiation order |
| RDP to legacy servers | Password prompt | Enable Remote Credential Guard or Restricted Admin mode |
| M365 apps using legacy protocols (IMAP, POP3, SMTP AUTH) | Blocked if legacy protocols are not already blocked | Deploy a Conditional Access policy blocking legacy protocols |
| Windows Credential Manager | Renewal impossible if the password is forgotten | Document the reset procedure with IT assistance |

---

<a name="deploiement"></a>

## 7. Deployment: Cloud Kerberos Trust and AzureADKerberos

### 7.1 Creating the AzureADKerberos object

<blockquote>
<p><strong>What this object does</strong></p>
<p>AzureADKerberos is a virtual RODC-type krbtgt object created in the AD and published in Entra ID. It establishes a shared secret between the on-prem AD and Entra ID, allowing Entra ID to issue partial TGTs that on-prem DCs recognize and exchange for full TGTs. Result: the WHfB user accesses on-prem resources without a password and without PKI.</p>
</blockquote>

**Step 1: TLS 1.2**

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12
```

**Step 2: PowerShell module** (PowerShell 5.1 required, does not work in PowerShell 7)

```powershell
Install-Module -Name AzureADHybridAuthenticationManagement -AllowClobber -Force
Import-Module AzureADHybridAuthenticationManagement
```

Note: if the message "Required VC++ 2013 x64 Runtimes do not exist" appears, the module installs the runtime automatically. Let it finish.

**Step 3: Variables**

```powershell
$domain = $env:USERDNSDOMAIN
$domainCred = Get-Credential -Message "Domain Admin credentials"
```

**Step 4: Creation**

```powershell
Set-AzureADKerberosServer -Domain $domain `
  -UserPrincipalName 'admin@[tenant].onmicrosoft.com' `
  -DomainCredential $domainCred
```

An Entra ID sign-in window opens in the browser: sign in with the Global Admin or Hybrid Identity Admin account. No output = silent success.

**Step 5: Verification**

```powershell
Get-AzureADKerberosServer -Domain $domain `
  -UserPrincipalName 'admin@[tenant].onmicrosoft.com' `
  -DomainCredential $domainCred
```

<blockquote class="ok">
<p><strong>Expected output after a successful creation (validated in the lab, August 24, 2026)</strong></p>
<pre>
Id                 : [unique number]
UserAccount        : CN=krbtgt_AzureAD,CN=Users,DC=[domain],DC=[ext]
ComputerAccount    : CN=AzureADKerberos,OU=Domain Controllers,DC=[domain],DC=[ext]
DisplayName        : krbtgt_[number]
DomainDnsName      : [domain.ext]
KeyVersion         : [number]
CloudKeyVersion    : [same number as KeyVersion]  ← synchronization confirmed
CloudKeyUpdatedOn  : [same date as KeyUpdatedOn]  ← ideally 0 seconds apart
CloudTrustDisplay  : [empty on first deployment, normal]
</pre>
<p>Points to check: KeyVersion = CloudKeyVersion and a gap between KeyUpdatedOn / CloudKeyUpdatedOn of less than 1h.</p>
</blockquote>

### 7.2 Configuring WHfB in Intune

<blockquote class="yellow">
<p><strong>Tenant-wide policy or Account protection profile</strong></p>
<p>The tenant-wide policy applies at device enrollment, to the whole organization. After enrollment, Microsoft indicates using the "Account protection" profile (Endpoint security) or the settings catalog to manage WHfB on targeted groups. If both define the same settings (the PIN, for example), the WHfB policy applies: align the values to avoid unexpected behavior.</p>
<p>The method described below (tenant-wide policy + Cloud Kerberos Trust profile) is the one validated in the lab.</p>
</blockquote>

**Step A: configure the tenant-wide WHfB policy**

`intune.microsoft.com` → Devices → Enrollment → Windows → **Windows Hello for Business**

This policy exists on all tenants but is in the "Not configured" state by default.

| Setting | Recommended value | Remark |
|---|---|---|
| Configure Windows Hello for Business | Enabled | Enables WHfB on all enrolled devices |
| Use a Trusted Platform Module (TPM) | Required | Two possible values. **Required** (default): only devices with an accessible TPM can provision WHfB. **Preferred**: the device tries to use the TPM and, failing that, falls back to software encryption. The key is then no longer hardware-protected, which weakens the guarantees described in sections 5 and 9. Keep Preferred for a transition phase on a mixed fleet. |
| Minimum PIN length | 6 | Recommended minimum |
| Allow biometric authentication | Yes | Fingerprint, facial recognition |
| Use security keys for sign-in | Enabled | Enables FIDO2 in addition to WHfB |

**Step B: create the Cloud Kerberos Trust profile**

`intune.microsoft.com` → Devices → Configuration → Create → New policy

1. Platform: Windows 10 and later
2. Profile type: **Settings catalog** (not Templates)
3. Name: WHfB-Cloud-Kerberos-Trust
4. Add settings: search "Cloud Trust" → Windows Hello for Business category
5. Setting: **Use cloud trust for on-premises authentication** → Enabled
6. Assignment: initial pilot group (2 to 3 non-critical hybrid devices)
7. Review + create → Create

<blockquote class="ok">
<p><strong>Expected behavior at the first WHfB enrollment on a hybrid workstation (validated in the lab, August 24, 2026)</strong></p>
<p>1. Windows sign-in with password → Windows detects the WHfB policy<br>
2. Authenticator or registered MFA method requested to validate identity<br>
3. WHfB wizard opens → PIN creation (6 characters minimum)<br>
4. Reboot possible depending on the workstation: normal during the initial policy application<br>
5. Sign back in with the PIN → an Entra ID sign-in window opens once to create the PRT<br>
6. Subsequent sign-ins: PIN only, no more Microsoft sign-in window</p>
<p>Note: the Intune tenant-wide policy works without a GPO on hybrid workstations. If a WHfB GPO exists in the AD, it takes precedence over Intune: check section 6.1.</p>
</blockquote>

### 7.3 Checking the WHfB state on the workstation

From an administrator command prompt:

```cmd
dsregcmd /status
```

<blockquote class="ok">
<p><strong>Key fields to check in the output (validated in the lab, August 24, 2026)</strong></p>
<pre>
Device State :
  AzureAdJoined : YES  ← workstation joined to Entra ID
  DomainJoined  : YES  ← workstation joined to the on-prem AD (hybrid)
  TpmProtected  : YES  ← WHfB key in the TPM

User State :
  NgcSet    : YES      ← WHfB enrolled for this user
  NgcKeyId  : {GUID}   ← identifier of the WHfB key in the TPM

SSO State :
  AzureAdPrt : YES  ← Entra ID PRT active (M365 SSO)
  OnPremTgt  : YES  ← on-prem Kerberos ticket active
  CloudTgt   : YES  ← Cloud Kerberos Trust active ✅

Diagnostic :
  KeySignTest : PASSED  ← the WHfB key signs correctly
</pre>
</blockquote>

<blockquote class="yellow">
<p><strong>Troubleshooting: PreReqResult = WillNotProvision</strong></p>
<p>If <code>NgcSet : NO</code> and <code>PreReqResult : WillNotProvision</code> appear, check two points:</p>
<p><strong>1. Does the account have an MFA method in the new unified Entra ID system?</strong><br>
Check via: <code>entra.microsoft.com</code> → Users → [account] → Authentication methods<br>
Legacy methods (Authenticator registered via outlookMobile) may not be enough.<br>
Solution: register a native method via <code>aka.ms/mysecurityinfo</code>.</p>
<p><strong>2. Is the account subject to an active MFA policy?</strong><br>
An account excluded from every MFA policy cannot enroll in WHfB.<br>
WHfB requires strong identity validation during enrollment.</p>
</blockquote>

### 7.4 Conditional Access: require phishing-resistant MFA

Once WHfB and FIDO2 are deployed, create a Conditional Access policy that exclusively requires a phishing-resistant method, which removes the fallback to SMS or TOTP.

<blockquote class="ok">
<p><strong>License scope: Entra ID P1, within the Business Premium scope</strong></p>
<p>The "Require authentication strength" feature in Conditional Access only requires P1.</p>
</blockquote>

**Built-in authentication strengths in the interface:**

- **Multifactor authentication:** combinations that meet strong auth (Password + SMS, etc.)
- **Passwordless MFA:** Microsoft Authenticator, etc.
- **Phishing-resistant MFA:** FIDO2, WHfB, CBA only

<blockquote class="red">
<p><strong>Mandatory prerequisite before activation</strong></p>
<p>Every user subject to this policy must have at least one phishing-resistant method registered. Without it, the user will be blocked. First enable the policy in "Report-only" mode and check the logs before switching it to On. A minimum of 2 weeks of monitoring is recommended.</p>
</blockquote>

**Policy 1: privileged accounts (top priority)**

`entra.microsoft.com` → Conditional Access → Policies → New policy

1. Name: CA-Phishing-Resistant-Admins
2. Users: Directory roles → Global Administrator, Privileged Role Administrator, Security Administrator (at a minimum)
3. Target resources: All resources (formerly "All cloud apps")
4. Access controls → Grant: Grant access → Require authentication strength → Phishing-resistant MFA → Select
5. Enable: **Report-only** → Create

<blockquote class="yellow">
<p><strong>Microsoft warning when creating the policy</strong></p>
<p>"Don't lock yourself out! This policy affects the Azure portal." (translated from the French UI) It can be ignored in Report-only mode: Report-only mode never blocks access.</p>
</blockquote>

**Policy 2: all users (progressive rollout)**

Deploy only after confirming that all users have enrolled WHfB or a passkey.

1. Name: CA-Phishing-Resistant-AllUsers
2. Users: All users, excluding the break-glass group
3. Target resources: All resources (or critical applications only at first)
4. Access controls → Grant: Grant access → Require authentication strength → Phishing-resistant MFA → Select
5. Enable: Report-only → minimum 2 weeks of monitoring → switch to On

<blockquote>
<p><strong>What the policy does in On mode: exact behavior</strong></p>
<p>The policy does not force WHfB enrollment automatically. It requires that a phishing-resistant method be used to authenticate.</p>
<p>· User with WHfB enrolled: seamless sign-in via WHfB, policy satisfied.<br>
· User with a FIDO2 passkey: sign-in via passkey, policy satisfied.<br>
· User without a phishing-resistant method: access blocked, redirected to <code>aka.ms/mysecurityinfo</code>.</p>
<p>Older accounts without a registered phishing-resistant method will be blocked. Plan an enrollment campaign before switching to On.</p>
</blockquote>

<blockquote class="yellow">
<p><strong>Preliminary audit: identify non-compliant accounts</strong></p>
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
<p><strong>Note: passkeys by default, then SMS and voice retirement</strong></p>
<p>Starting September 1, 2026 (gradual rollout), Microsoft automatically enables passkeys for users configured for SMS or voice and prompts them to register one at MFA sign-in. Microsoft-provided SMS and voice will be retired on February 1, 2027 for all users except Global Administrators and external users, whose date is July 1, 2027. Internal guest users follow the February 1 date.</p>
<p>Organizations that need to keep SMS or voice can configure a telephony provider through Microsoft Security Store (available since October 30, 2026, at their own cost). Otherwise, plan the migration to WHfB or passkeys before these dates.</p>
</blockquote>

---

<a name="rotation"></a>

## 8. Rotating the AzureADKerberos key

### 8.1 Why and how often

| Cadence | Context | Remark |
|---|---|---|
| Same schedule as the AD krbtgt rotation | Microsoft recommendation | Microsoft does not mandate a fixed interval |
| More frequent | Sensitive environments | Governance choice to document (data security, art. 8 nFADP) |
| Immediately | Confirmed compromise of a Domain Admin account | Emergency procedure |

### 8.2 Standard rotation procedure

**Step 1: Rotation**

```powershell
Set-AzureADKerberosServer -Domain $domain `
  -UserPrincipalName 'admin@[tenant].onmicrosoft.com' `
  -DomainCredential $domainCred -RotateServerKey
```

**Step 2: Immediate verification:** run `Get-AzureADKerberosServer` (see 7.1). `KeyVersion` must equal `CloudKeyVersion`, and the gap between `KeyUpdatedOn` and `CloudKeyUpdatedOn` must stay under 1h.

**Step 3: Wait:** do not start another rotation for at least 10h (the default lifetime of a Kerberos TGT), ideally 24h.

**Step 4: Validation:** test a WHfB sign-in on a hybrid workstation to confirm that the rotation did not break authentication.

<blockquote>
<p><strong>Coordination with the AD krbtgt account rotation</strong></p>
<p>The AzureADKerberos object and the AD krbtgt account are two distinct objects with independent keys: their rotations are separate operations.</p>
<p>AD krbtgt rotation: classic best practice, recommended every 60 to 90 days, or immediately after any incident involving unauthorized Domain Admin access. The krbtgt rotation is done in two passes spaced by the maximum Kerberos ticket lifetime (10h by default) to avoid blocking active sessions.</p>
<p>Recommended order: AD krbtgt first (double rotation D+0 / D+10h minimum), then AzureADKerberos after confirming stability. Leave at least 10 to 24h between the two. Aligning both cycles on a common schedule, as Microsoft recommends, simplifies management and reduces the risk of forgetting.</p>
</blockquote>

### 8.3 Automation via Azure Automation

Periodic runbook logic:

1. Retrieve the credentials from Azure Key Vault (never in clear text in the script)
2. Run `Set-AzureADKerberosServer -RotateServerKey`
3. Run `Get-AzureADKerberosServer` → check that `KeyVersion` = `CloudKeyVersion` and that the gap between `KeyUpdatedOn` and `CloudKeyUpdatedOn` is under 1h
4. If the gap is greater than 1h: send an email / Teams alert → manual intervention required
5. Log the result in Log Analytics for the audit trail

Required rights: Hybrid Identity Administrator (Entra ID) + delegated Domain Admin rights (via a Credential stored in Key Vault).

---

<a name="fido2"></a>

## 9. Annex A: hardware FIDO2 for privileged accounts

### 9.1 When to prefer a physical key over TPM-based WHfB

| Criterion | WHfB / TPM | Physical FIDO2 key |
|---|---|---|
| Portability | Tied to the workstation | Portable between workstations |
| Cost | Zero (built into the workstation) | 30 to 80 CHF / unit depending on model |
| Declared lifespan | Tied to the workstation (5 to 10 years) | YubiKey: 5 years minimum, typically 7 to 10 years. Feitian: 5 years declared. Practical limit: physical loss or policy change (certified USB connector >10,000 insertions). |
| Deployment | Via Intune, scalable | Physical distribution, manual |
| Loss / theft | No risk (key in the TPM) | Physical risk: revocation procedure needed |
| Recommended accounts | All standard users | Global Admin, break-glass, PIM accounts |

### 9.2 Enabling FIDO2 in Entra ID

`entra.microsoft.com` → Authentication methods → Policies → **Passkey (FIDO2)**

Page: **Passkey (FIDO2) settings**

<blockquote class="yellow">
<p><strong>Important: FIDO2 is disabled by default</strong></p>
<p>Unlike CAE, the "Passkey (FIDO2)" method is disabled by default on all tenants. Manual activation is required.</p>
</blockquote>

**"Enable and target" tab:**

1. Enable toggle → **Enabled**
2. Include tab → All users (or a specific group depending on the deployment strategy)
3. Exclude tab → Add the break-glass group

**"Configure" tab:**

- Default passkey profile: Types = Device-bound, Synced, Restrictions = No
- For privileged accounts: create a dedicated profile with Device-bound only and specific YubiKey AAGUIDs

<blockquote>
<p><strong>Note on Microsoft Authenticator in the AAGUIDs</strong></p>
<p>The interface offers Windows Hello, Microsoft Authenticator and Enter AAGUID as shortcuts. Microsoft Authenticator = Synced passkey stored in the mobile app, synchronized via Microsoft. More convenient but slightly less robust than a Device-bound key for ultra-sensitive accounts. For Global Admin: prefer Device-bound only (YubiKey or TPM).</p>
</blockquote>

Enrollment: the user enrolls their key or passkey via `aka.ms/mysecurityinfo`.

---

<a name="conclusion"></a>

## 10. Conclusion: intellectual honesty about the level of protection

This document does not make a Microsoft 365 tenant unbreachable. It raises the difficulty for the attacker.

| Level | Measure | What it guarantees |
|---|---|---|
| Baseline | MFA only, no CAE | A stolen cookie is exploitable until revocation, then the access token stays valid for up to one hour |
| + CAE | CAE + Identity Protection | Near real-time revocation (up to 15 minutes according to Microsoft), provided detection follows |
| + SIEM | Rules M2/M3/M5 to M8/M12/M14 + M-new-1 + M-new-2 | Visibility on post-exploitation actions in the tenant |
| + Purview Suite (add-on) | IRM data-leak policy + eDiscovery Premium | SharePoint coverage and building nFADP evidence |
| + WHfB / FIDO2 | Structural protection (TPM + Cloud Kerberos Trust) | Proxy-based AiTM attack defeated (no valid session for the proxy) |
| Full posture | WHfB + key rotation + legacy audit + SIEM + IRM | Structural resistance + residual risk management |

Other vectors stay active regardless of WHfB: compromised endpoint (malware, keylogger), direct social engineering, compromise of the on-prem DC. WHfB closes the AiTM vector but does not replace an overall security posture.

*Recommended next step: make sure CAE is active immediately (a 20-minute check), then plan WHfB as a hardening project over 1 to 2 sessions.*

---

<a name="siem"></a>

## 11. Annex B: why a SIEM is essential, even a free one

### 11.1 The problem without a SIEM

The IoCs described in this document exist in the Entra ID, Exchange and Defender logs. Identity Protection sends alerts, but they land in a portal that nobody actively monitors. Without a SIEM, detection is manual, reactive and often too late.

A SIEM with correlation rules turns these signals into actionable alerts, in real time, sent to the channels the IT team actually watches (email, Teams, SMS). That is the difference between discovering a compromise 3 days after the fact and reacting within 7 minutes.

### 11.2 Open source / free options for an SME

| SIEM | Strengths | Weaknesses | M365 integration |
|---|---|---|---|
| Wazuh | Very popular, large community, abundant documentation, native endpoint agents, built-in XDR | Manual and complex M365 configuration, steep learning curve | Via the Wazuh Office 365 module (Management Activity API), manual rule configuration |
| UTMStack CE | Intuitive interface, native O365 integration, M-series rules validated in the lab on the AiTM and CVE-2026-69836 scenarios, documented lab available | Smaller community, fewer online resources | Native O365: M-series rules ready to import, validated in the lab |
| Graylog | Excellent for high-volume log management, clear interface | Less SIEM-oriented, limited correlation in the CE version | Via custom inputs, requires rule development |

### 11.3 Recommendation for an M365 SME

For an SME of 5 to 25 users on M365 Business Premium, UTMStack CE offers the best deployment / immediate coverage ratio for the scenarios documented in this guide.

<blockquote class="ok">
<p><strong>UTMStack lab: M365 rules validated in the lab</strong></p>
<p>The UTMStack lab documented on GitHub Pages covers all the M365 correlation rules used in this guide, with ready-to-import YAML files, documented trigger tests and the fields available in each alert.</p>
<p>· <a href="https://doit4everyone.github.io/utmstack-lab/docs/11-correlations-yaml.html">Chapter 11: M-series correlation rules (including M-new-1 and M-new-2), in French</a><br>
· <a href="https://doit4everyone.github.io/utmstack-lab/">Complete UTMStack lab (in French)</a></p>
</blockquote>

---

<a name="renvois"></a>

## 12. References and resources

| Document / Resource | License scope | Link |
|---|---|---|
| MVC Purview nLPD Guide, Swiss SMEs (in French) | Business Premium (no add-on) | [doit4everyone.github.io](https://doit4everyone.github.io/Configuration-Purview-PME-Suisse-nLPD/) |
| Full Purview 2026 guide: IRM (8.2), eDiscovery (8.3) (in French) | Business Premium + Purview Suite (add-on required) | [Section 8: Advanced features](https://doit4everyone.github.io/microsoft-purview-configuration-2026-nLPD/docs/09-fonctionnalites-avancees.html) |
| Shadow AI / Copilot Governance guide (in French) | Business Premium + Purview Suite (add-on required) | [doit4everyone.github.io](https://doit4everyone.github.io/shadow-ai-governance-microsoft-365-nLPD/) |
| UTMStack lab: Chapter 11 (rules M-new-1, M-new-2) (in French) | UTMStack CE (open source) | [11-correlations-yaml.html](https://doit4everyone.github.io/utmstack-lab/docs/11-correlations-yaml.html) |
| UTMStack lab: home (in French) | UTMStack CE (open source) | [utmstack-lab](https://doit4everyone.github.io/utmstack-lab/) |
| Microsoft: Cloud Kerberos Trust | Business Premium (included) | [learn.microsoft.com](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-hybrid-cloud-kerberos-trust) |
| Microsoft: CAE | Business Premium (included) | [learn.microsoft.com](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-continuous-access-evaluation) |
| Microsoft: FIDO2 security keys | Business Premium (included) | [learn.microsoft.com](https://learn.microsoft.com/en-us/entra/identity/authentication/howto-authentication-passwordless-security-key) |
| Microsoft: CVE-2026-69836 advisory | N/A | [msrc.microsoft.com](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2026-69836) |
| The Hacker News: correction of the CVE-2026-69836 exploitation status | N/A | [thehackernews.com](https://thehackernews.com/2026/08/microsoft-entra-id-flaw-cvss-100.html) |
| ANY.RUN: Mirage2FA research | N/A | [any.run](https://any.run/cybersecurity-blog/mirage2fa-phishing-targets-us-companies/) |
| The Hacker News: Mirage2FA | N/A | [thehackernews.com](https://thehackernews.com/2026/08/mirage2fa-surge-hits-4500-us-and-eu.html) |
| Microsoft: SMS and voice retirement (FAQ) | N/A | [learn.microsoft.com](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-sms-voice-retirement-faq) |

---

## 📥 Downloads

- [Word version (.docx), French only]({{ '/downloads/docx/MVC-nLPD_Annexe-AiTM_v1.1.docx' | relative_url }})

---

[← Back to the MVC nLPD bundle](../) · [🏠 Back to the main portal](https://doit4everyone.github.io/)

---

*ℹ️ References, structuring and drafting assistance provided by AI, with final human validation. Field validation performed on a real hybrid infrastructure, August 2026. Corrected on November 2, 2026: Mirage2FA figures, CVE status, SMS and voice retirement, technical clarifications (CAE, Identity Protection, WHfB, Cloud Kerberos Trust, rotation, logs).*
