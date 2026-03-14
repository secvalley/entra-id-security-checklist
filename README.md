[![MIT License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![GitHub Stars](https://img.shields.io/github/stars/secvalley/entra-id-security-checklist?style=social)](https://github.com/secvalley/entra-id-security-checklist/stargazers)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/secvalley/entra-id-security-checklist/pulls)

# Entra ID Security Checklist

A practical, no-nonsense security checklist for Microsoft Entra ID (formerly Azure Active Directory). Built from real-world tenant hardening work, incident response lessons, and way too many misconfigured conditional access policies.

Whether you're standing up a new tenant or auditing an existing one, this checklist covers the controls that actually matter.

**Homepage:** [secvalley.com](https://secvalley.com)

---

## Table of Contents

- [How to Use This Checklist](#how-to-use-this-checklist)
- [Authentication & MFA](#authentication--mfa)
- [Conditional Access](#conditional-access)
- [Privileged Identity Management (PIM)](#privileged-identity-management-pim)
- [Application Security](#application-security)
- [Guest & External Access](#guest--external-access)
- [Monitoring & Logging](#monitoring--logging)
- [Going Further](#going-further)
- [Related Projects](#related-projects)
- [Contributing](#contributing)
- [License](#license)

---

## How to Use This Checklist

Each control follows this format:

- **Severity** indicates business impact if the control is missing: `Critical`, `High`, `Medium`, or `Low`.
- Check each item against your tenant. If it does not apply to your environment, mark it as N/A and move on.
- Links point to the relevant Microsoft documentation so you can dig deeper.

This is not meant to be an exhaustive list of every toggle in the Entra portal. It focuses on the settings that, in practice, cause the most damage when they are wrong.

---

## Authentication & MFA

- [ ] **[Critical]** Enforce phishing-resistant MFA (FIDO2, Windows Hello for Business, or certificate-based auth) for all administrators. Legacy MFA methods like SMS and voice calls are trivially bypassable via SIM swap and social engineering. Do not rely on them for privileged accounts.
  - [Phishing-resistant MFA methods](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-strengths)

- [ ] **[Critical]** Require MFA for all users, not just admins. Attackers routinely compromise regular user accounts first, then escalate. A tenant where only admins have MFA is a tenant waiting for an incident.
  - [Enable combined MFA registration](https://learn.microsoft.com/en-us/entra/identity/authentication/howto-registration-mfa-sspr-combined)

- [ ] **[Critical]** Block legacy authentication protocols (IMAP, POP3, SMTP AUTH, ActiveSync with basic auth). These protocols do not support MFA at all and are the single most common entry point in password spray attacks. If you do nothing else on this list, do this.
  - [Block legacy authentication](https://learn.microsoft.com/en-us/entra/identity/conditional-access/block-legacy-authentication)

- [ ] **[High]** Disable SMS and voice call as MFA methods for the entire tenant, or at minimum for all admin roles. Migrate users to the Microsoft Authenticator app, passkeys, or hardware tokens.
  - [Authentication methods policy](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-methods-manage)

- [ ] **[High]** Configure the Authenticator app to show number matching and additional context (application name, location). This significantly reduces MFA fatigue attacks where users blindly approve push notifications.
  - [Number matching in MFA](https://learn.microsoft.com/en-us/entra/identity/authentication/how-to-mfa-number-match)

- [ ] **[High]** Enable self-service password reset (SSPR) with appropriate verification methods and combine it with MFA registration. Make sure SSPR does not use weaker methods than your MFA policy requires.
  - [SSPR deployment](https://learn.microsoft.com/en-us/entra/identity/authentication/howto-sspr-deployment)

- [ ] **[Medium]** Set a password protection policy that bans common passwords and includes a custom banned password list with your organization name, product names, and other easily guessable terms.
  - [Entra ID password protection](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-password-ban-bad)

- [ ] **[Medium]** Review and restrict authentication methods available in the tenant. Disable any methods you are not actively using. Fewer options means a smaller attack surface.
  - [Manage authentication methods](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-methods-manage)

---

## Conditional Access

- [ ] **[Critical]** Create a "break glass" emergency access account that is excluded from all conditional access policies. Store the credentials securely offline. Test the account periodically to make sure it still works. This is the account you reach for when something goes catastrophically wrong with your CA policies.
  - [Emergency access accounts](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/security-emergency-access)

- [ ] **[Critical]** Deploy a baseline policy that requires MFA for all users, all cloud apps, on all platforms. Start here and layer additional policies on top. Do not try to build a patchwork of narrow policies first.
  - [Common CA policies](https://learn.microsoft.com/en-us/entra/identity/conditional-access/howto-conditional-access-policy-all-users-mfa)

- [ ] **[Critical]** Block sign-ins from countries and regions where your organization does not operate. Named locations make this straightforward. This will not stop a determined attacker with a VPN, but it cuts out the vast majority of spray traffic.
  - [Location-based conditional access](https://learn.microsoft.com/en-us/entra/identity/conditional-access/howto-conditional-access-policy-location)

- [ ] **[High]** Require compliant or hybrid-joined devices for access to sensitive applications. Token theft is a growing problem, and device-based controls are one of the few effective mitigations available today.
  - [Require compliant devices](https://learn.microsoft.com/en-us/entra/identity/conditional-access/howto-conditional-access-policy-compliant-device)

- [ ] **[High]** Configure sign-in risk and user risk policies using Entra ID Protection. Set medium and high risk sign-ins to require MFA or block access. Set high risk users to require password change.
  - [Risk-based conditional access](https://learn.microsoft.com/en-us/entra/id-protection/howto-identity-protection-configure-risk-policies)

- [ ] **[High]** Use report-only mode for new policies before enforcing them. Review the "What If" tool results against real sign-in data. A misconfigured CA policy can lock out your entire organization in seconds.
  - [CA report-only mode](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-report-only)

- [ ] **[Medium]** Require re-authentication for sensitive actions (sign-in frequency controls). For high-privilege admin portals, consider a 1-hour session maximum. For general users, balance security with usability.
  - [Sign-in frequency](https://learn.microsoft.com/en-us/entra/identity/conditional-access/howto-conditional-access-session-lifetime)

- [ ] **[Medium]** Block or restrict access from unmanaged devices to prevent data exfiltration via personal machines. Use app-enforced restrictions or Conditional Access App Control for session-level controls.
  - [App-enforced restrictions](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-session#application-enforced-restrictions)

---

## Privileged Identity Management (PIM)

- [ ] **[Critical]** Enable PIM for all Entra ID directory roles. No one should hold Global Administrator, Exchange Administrator, or any other privileged role as a permanent assignment. If a role is permanently assigned, it is permanently exposed.
  - [Deploy PIM](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-deployment-plan)

- [ ] **[Critical]** Limit the number of Global Administrators to no more than five (Microsoft recommends fewer than five). Audit who currently holds this role. In most tenants we review, this number is higher than anyone expects.
  - [Best practices for Entra roles](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/best-practices)

- [ ] **[High]** Require MFA and justification for every PIM role activation. Configure approval workflows for Critical-impact roles like Global Administrator, Privileged Role Administrator, and Security Administrator.
  - [Configure PIM role settings](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-how-to-change-default-settings)

- [ ] **[High]** Set activation duration to the minimum practical time window. Four hours is a reasonable default for most roles. One hour is better for Global Administrator. Do not set 24-hour activation windows just because it is more convenient.
  - [PIM role settings](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-how-to-change-default-settings)

- [ ] **[High]** Configure recurring access reviews for all privileged role assignments. Quarterly reviews are a good starting point. Stale eligible assignments are nearly as dangerous as permanent ones because nobody is watching them.
  - [Access reviews for PIM](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-create-roles-and-resource-roles-review)

- [ ] **[High]** Enable PIM for Azure resource roles (Owner, Contributor, User Access Administrator) in addition to Entra directory roles. The Azure control plane is just as sensitive as the identity plane.
  - [PIM for Azure resources](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-resource-roles-assign-roles)

- [ ] **[Medium]** Set up PIM alerts and review them regularly. Pay attention to alerts for roles being activated outside of expected hours, roles that are never used (remove them), and permanent assignments that were added outside PIM.
  - [PIM security alerts](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-how-to-configure-security-alerts)

- [ ] **[Medium]** Use PIM for Groups to manage membership in security groups that grant sensitive access. This extends just-in-time access beyond directory roles to any permission gated by group membership.
  - [PIM for Groups](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/concept-pim-for-groups)

---

## Application Security

- [ ] **[Critical]** Restrict who can register applications in Entra ID. By default, all users can register apps and grant them permissions. This is almost never what you want. Set "Users can register applications" to No and manage app registrations through a controlled process.
  - [App registration permissions](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/delegate-app-roles#restrict-who-can-create-applications)

- [ ] **[Critical]** Review and minimize applications with high-privilege Microsoft Graph permissions (Directory.ReadWrite.All, Mail.ReadWrite, etc.). Audit admin-consented permissions across your tenant. Overprivileged apps are a favorite persistence mechanism for attackers.
  - [Review app permissions](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/manage-application-permissions)

- [ ] **[High]** Configure the admin consent workflow so users can request permissions through a governed process instead of being blocked entirely. Without this, users tend to find workarounds that are worse than the original risk.
  - [Admin consent workflow](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/configure-admin-consent-workflow)

- [ ] **[High]** Restrict user consent to verified publishers only, or disable user consent entirely. Consent phishing (illicit consent grants) is an increasingly common attack vector and it completely bypasses MFA.
  - [Configure user consent](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/configure-user-consent)

- [ ] **[High]** Rotate application credentials (client secrets and certificates) on a defined schedule. Set expiration to no more than 12 months. Track expiration dates so you are not scrambling when a production integration breaks at 2 AM.
  - [App credential management](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/certificate-management)

- [ ] **[High]** Use managed identities instead of service principals with secrets wherever possible. Managed identities eliminate the need to manage credentials entirely for Azure-hosted workloads.
  - [Managed identities overview](https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/overview)

- [ ] **[Medium]** Audit and remove stale application registrations and enterprise applications. Unused apps with granted permissions are an unnecessary risk. If nothing has signed in with an app in 90 days, investigate whether it is still needed.
  - [Review inactive apps](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/clean-up-unmanaged-accounts)

- [ ] **[Medium]** Ensure multi-tenant applications are configured intentionally. Single-tenant is the correct default for internal apps. An app accidentally set to multi-tenant can accept sign-ins from any Entra tenant in the world.
  - [Single vs multi-tenant apps](https://learn.microsoft.com/en-us/entra/identity-platform/single-and-multi-tenant-apps)

---

## Guest & External Access

- [ ] **[Critical]** Review cross-tenant access settings and configure inbound/outbound trust policies explicitly. The defaults are permissive. Define which external organizations you trust and what level of access their users get.
  - [Cross-tenant access settings](https://learn.microsoft.com/en-us/entra/external-id/cross-tenant-access-overview)

- [ ] **[High]** Restrict who can invite guest users. By default, all users including other guests can send invitations. Limit invitation permissions to specific roles or groups.
  - [Guest invitation settings](https://learn.microsoft.com/en-us/entra/external-id/external-collaboration-settings-configure)

- [ ] **[High]** Configure guest access restrictions to limit what directory information guests can see. By default, guests can enumerate users, groups, and other directory objects. Restrict guest permissions to properties of their own directory objects.
  - [Guest user permissions](https://learn.microsoft.com/en-us/entra/fundamentals/users-default-permissions#member-and-guest-users)

- [ ] **[High]** Set up recurring access reviews for all guest users. Guest accounts tend to accumulate over time and rarely get cleaned up. Quarterly reviews with auto-removal of unreviewed accounts is a reasonable approach.
  - [Guest access reviews](https://learn.microsoft.com/en-us/entra/id-governance/create-access-review)

- [ ] **[High]** Apply conditional access policies to guest and external users. Guests should at minimum be required to use MFA. Consider requiring terms of use acceptance and blocking access from risky locations.
  - [CA for external users](https://learn.microsoft.com/en-us/entra/external-id/authentication-conditional-access)

- [ ] **[Medium]** Use entitlement management with access packages for structured external collaboration. This gives you defined workflows, automatic expiration, and an audit trail instead of ad-hoc guest invitations floating around.
  - [Entitlement management](https://learn.microsoft.com/en-us/entra/id-governance/entitlement-management-overview)

- [ ] **[Medium]** Restrict which domains can be invited as guests. If you only collaborate with a handful of partner organizations, use an allow list. If you have too many partners for an allow list, at minimum maintain a deny list for known consumer email domains.
  - [Allow or block domains](https://learn.microsoft.com/en-us/entra/external-id/allow-deny-list)

- [ ] **[Low]** Configure B2B direct connect for close partner organizations that need tighter integration. This provides a more seamless experience than traditional guest accounts while still maintaining tenant boundaries.
  - [B2B direct connect](https://learn.microsoft.com/en-us/entra/external-id/b2b-direct-connect-overview)

---

## Monitoring & Logging

- [ ] **[Critical]** Stream Entra ID sign-in logs and audit logs to a SIEM or Log Analytics workspace. The built-in log retention in Entra is limited (7 days for free tenants, 30 days for P1/P2). If an attacker was in your tenant 45 days ago, you need to know about it.
  - [Integrate logs with Azure Monitor](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/howto-integrate-activity-logs-with-azure-monitor-logs)

- [ ] **[Critical]** Enable the unified audit log in Microsoft Purview and verify it is actually collecting events. It is surprising how often this is turned off or partially configured.
  - [Audit log activities](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/reference-audit-activities)

- [ ] **[High]** Create alerts for high-impact events: new Global Administrator assignments, conditional access policy changes, new application consent grants, bulk user modifications, and changes to authentication methods policies.
  - [Entra ID monitoring](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/overview-monitoring-health)

- [ ] **[High]** Monitor Entra ID Protection risk detections actively. Do not just configure risk policies and walk away. Review the risky users and risky sign-ins reports at least weekly. Investigate and remediate or dismiss each detection.
  - [Investigate risk](https://learn.microsoft.com/en-us/entra/id-protection/howto-identity-protection-investigate-risk)

- [ ] **[High]** Enable and review the Entra ID recommendations blade regularly. Microsoft surfaces tenant-specific security recommendations here based on your actual configuration. It is free and often catches things that checklists miss.
  - [Entra ID recommendations](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/overview-recommendations)

- [ ] **[High]** Set up diagnostic settings to capture non-interactive sign-in logs, service principal sign-in logs, and managed identity sign-in logs in addition to interactive sign-ins. Attackers frequently use service principals and non-interactive flows that do not appear in the default sign-in log view.
  - [Sign-in log categories](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/concept-sign-ins)

- [ ] **[Medium]** Implement a workbook or dashboard that tracks conditional access policy coverage. Identify which users and applications are not covered by any policy. Gaps in CA coverage are where attackers get in.
  - [Conditional access insights workbook](https://learn.microsoft.com/en-us/entra/identity/conditional-access/howto-conditional-access-insights-reporting)

- [ ] **[Medium]** Review the application credential activity report to identify credentials expiring soon, credentials that have never been used, and credentials with excessively long validity periods.
  - [App credential activity](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/howto-get-list-of-all-application-credentials)

---

## Going Further

This checklist is a point-in-time assessment. Identity configurations drift constantly. For continuous Entra ID security monitoring with automated checks, conditional access gap analysis, and PIM assessment, check out [SecValley CSPM](https://secvalley.com/cspm.html).

---

## Related Projects

- [cloud-security-checklist](https://github.com/secvalley/cloud-security-checklist) - Broad cloud security checklist covering AWS, Azure, and GCP
- [m365-security-baseline](https://github.com/secvalley/m365-security-baseline) - Microsoft 365 security baseline configuration guide

---

## Contributing

Contributions are welcome. If you have found a control that should be on this list, or if a Microsoft docs link has gone stale, open a PR or file an issue. Please keep entries practical and grounded in real-world configuration, not theoretical risks.

---

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

---

Maintained by [SecValley](https://secvalley.com)
