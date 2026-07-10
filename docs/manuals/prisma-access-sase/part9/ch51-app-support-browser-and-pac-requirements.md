---
title: "App Support, Browser Guidelines & PAC File Requirements"
description: "What applications and browsers Explicit Proxy supports, known browser-specific limitations, PAC file operational requirements, and proxy chaining and port constraints."
date: 2026-05-28
tags:
  - prisma-access
  - sase
  - paloalto
  - explicit-proxy
  - cloud-swg
  - pac-file
  - browser
  - saml
  - microsoft-365
status: published
---

# Chapter 51 — App Support, Browser Guidelines & PAC File Requirements

Understanding what Explicit Proxy can and cannot handle is essential before deployment. This chapter covers supported traffic types, per-browser requirements, and the PAC file constraints that govern how traffic reaches the proxy.

---

## Supported Traffic

| Traffic Type | Supported | Notes |
|---|---|---|
| HTTP (port 80) | Yes | |
| HTTPS (port 443) | Yes | SSL decryption required |
| FTP | No | Returns DIRECT or fails |
| Private / data center apps | No | Use GlobalProtect or ZTNA Connector instead |
| Microsoft 365 desktop client | No | Uses non-web ports |
| Microsoft 365 web / Office Online | Yes | Browser-based access only |

> ⚠️ Users trying to access private applications through Explicit Proxy will fail. Communicate this scope clearly during rollout — direct users to GlobalProtect for private app access.

---

## Browser Guidelines

### Cookies

Explicit Proxy uses browser cookies for SAML session tracking:

> **Do not disable cookies in your browser.** If cookies are disabled, SAML authentication cannot complete and users will be unable to browse any web pages through the proxy.

### Microsoft Edge

Edge's **Tracking Prevention** feature can interfere with the SAML authentication flow:

| Setting Path | Required Value |
|---|---|
| `Settings > Privacy, Search, and Services > Tracking prevention` | **Basic** |

Set this before deploying Explicit Proxy to Edge users. Strict or Balanced tracking prevention blocks the SAML redirect cookies.

### Safari

Safari has known compatibility issues with Explicit Proxy. The recommended mitigation is to use a supported browser instead:

- **Recommended browsers:** Microsoft Edge, Firefox, Chrome, Internet Explorer
- **Not recommended:** Safari (may experience access issues on various websites)

### Microsoft 365 Desktop

The full desktop client for Microsoft 365 (Word, Excel, Teams desktop app, etc.) uses non-web protocols and ports that Explicit Proxy does not intercept:

- **Not supported:** Desktop Office apps (Outlook, Teams, Word, Excel desktop)
- **Supported:** Web-based Microsoft 365 / Office Online accessed via browser at `office.com`

---

## PAC File Requirements

### Authentication

- **Corrected 2026-07-09** — **SAML and Kerberos** are both current, GA authentication protocols for Explicit Proxy, confirmed via direct fetch (same sourcing as Chapter 49) — this was previously stated as "SAML is the only supported authentication protocol," which is no longer accurate
- LDAP and local authentication are still **not** supported
- SAML and Kerberos can both be configured in the same deployment — endpoints with access to your Kerberos infrastructure use it, others fall back to SAML
- See [Chapter 49](./ch49-how-explicit-proxy-works.md) for the confirmed prerequisites (Kerberos Keytab, Kerberos Realm, UPN-format usernames) — not repeated here

### Port

- **Corrected 2026-07-09** — Explicit Proxy listens on **port 8080 for SAML** and **port 8081 for Kerberos** (previously stated as "port 8080 only")
- `PROXY` statements in the PAC file must use the port matching the authentication method in use for that deployment

### Required Bypasses

The PAC file must include direct (bypass) rules for:

| Bypass Target | Reason |
|---|---|
| **IdP URLs** (e.g. `*.okta.com`, `*.azure.com`) | SAML auth must not route through the proxy itself |
| **ACS URLs** (`*.acs.prismaaccess.com`) | Assertion Consumer Service must be directly reachable |

Without these bypasses, SAML authentication enters an infinite redirect loop.

> ℹ️ **Kerberos deployments — pointer only, not elaborated here:** the bypass rules above are specific to SAML. Kerberos-based Explicit Proxy deployments have their own bypass/trust considerations (Kerberos infrastructure reachability rather than IdP/ACS URLs). This wasn't independently confirmed in enough detail to document accurately in this chapter — see the [Kerberos Authentication for Explicit Proxy Deployments documentation](https://docs.paloaltonetworks.com/prisma-access/administration/prisma-access-mobile-users/mobile-users-explicit-proxy/kerberos-authentication-for-explicit-proxy-deployments) rather than assuming the SAML rules above apply unchanged.

### Proxy Chaining

If forwarding from a third-party proxy to Explicit Proxy:
- Enter the **Explicit Proxy URL** in the upstream proxy as the forwarding destination
- The Explicit Proxy URL is found at: `Panorama > Cloud Services > Status > Network Details > Mobile Users — Explicit Proxy`

### On-Premises Support

Explicit Proxy is a **cloud-only** service — there is no on-premises equivalent. All traffic must route to the Prisma Access cloud infrastructure.

---

## Key Takeaways

- HTTP and HTTPS only — FTP, private apps, and desktop Office apps are not supported
- Safari is not recommended — use Edge, Firefox, or Chrome
- Edge requires Tracking Prevention set to **Basic** for SAML to work correctly
- Do not disable browser cookies — SAML authentication depends on them
- **Corrected 2026-07-09** — SAML is **not** the only authentication method; Kerberos is a current, GA alternative (see Chapter 49). For SAML specifically, the PAC file must bypass IdP and ACS URLs — Kerberos deployments have different bypass considerations, not detailed in this chapter
- **Corrected 2026-07-09** — Explicit Proxy listens on **port 8080 for SAML** and **port 8081 for Kerberos** — not "8080 only" as previously stated

---

*Previous: [Chapter 50 — Explicit Proxy Configuration Guidelines](./ch50-explicit-proxy-configuration-guidelines.md)* · *Next: [Chapter 52 — PAC File Guidelines — Detailed & Sample](./ch52-pac-file-guidelines-and-sample.md)*
