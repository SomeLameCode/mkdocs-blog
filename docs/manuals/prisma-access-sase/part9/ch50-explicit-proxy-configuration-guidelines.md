---
title: "Explicit Proxy Configuration Guidelines"
description: "Licensing, network, and software prerequisites for deploying Prisma Access Explicit Proxy — SSL decryption requirement, HTTP/2 limitations, TLS constraints, and Panorama version requirements."
date: 2026-05-28
tags:
  - prisma-access
  - sase
  - paloalto
  - explicit-proxy
  - cloud-swg
  - ssl-decryption
  - panorama
  - prerequisites
status: published
---

# Chapter 50 — Explicit Proxy Configuration Guidelines

Before deploying Explicit Proxy, verify that licensing, software versions, and network requirements are satisfied. Skipping these prerequisites is the most common cause of failed deployments.

---

## Licensing Requirements

Explicit Proxy requires a **Prisma Access license that includes the Explicit Proxy (Cloud SWG) add-on**. Confirm with your Palo Alto Networks account team that the Explicit Proxy feature is activated on your tenant before proceeding.

### Multitenancy Restriction — Confirmed, Not Previously Documented

If an existing non-multitenant Prisma Access deployment is converted to multitenant, **only the first (originally migrated) tenant supports Explicit Proxy** — any tenant created after that conversion does not support it at all. Confirmed via a compatibility table listing "Supported on first tenant" across every Explicit Proxy connection method (Agentless SAML, Agentless Kerberos, Proxy Mode on Agent, Proxy Mode on Remote Networks, and Prisma Browser), scoped to the "Multitenant Panorama" row specifically. If you're planning a multitenant deployment and need Explicit Proxy on more than one tenant, this is a hard constraint to account for during planning — not something that can be worked around later.

---

## Panorama and Content Version Requirements

| Requirement | Minimum Version |
|---|---|
| **Panorama** | 10.0.5 *(see note below — could not be reconfirmed on current docs)* |
| **Antivirus Content** | Version **3590** (installed on Panorama — required for predefined security policies) |

To check and update antivirus content:
`Panorama > Dynamic Updates > Antivirus`

> ⚠️ **Panorama version requirement — investigated, not silently kept or removed.** Fetched the current live Explicit Proxy guidelines page directly: it lists both Panorama-managed and Strata Cloud Manager-managed as supported deployment options but states **no minimum Panorama version at all** — the "10.0.5" figure isn't restated anywhere on the current guidelines page. This could mean the requirement has since been relaxed, or simply that this particular page no longer repeats it while it's still accurate elsewhere — I couldn't determine which from what's available. Left as-is with this caveat rather than deleted, since I can't confirm it's wrong, only that it's unconfirmed.
>
> **For a pure Strata Cloud Manager-managed deployment, this requirement doesn't apply at all** — confirmed via the Strata Cloud Manager prerequisites documentation: there is no Panorama in the picture for a pure SCM deployment, and Panorama is mentioned there only as an optional, separate management path (e.g. connecting an existing Panorama-managed NGFW fleet), not a version prerequisite for SCM itself. If your deployment is hybrid (Panorama present for other purposes), the Panorama-side figure above may still be relevant to double-check directly.

**Navigation (Strata Cloud Manager):** `Configuration > NGFW and Prisma Access > Configuration Scope > Prisma Access > Mobile Users` — the same starting point already established for GlobalProtect Mobile Users onboarding in Chapter 44, not repeated here.

---

## Network Requirements

### SSL Decryption — Mandatory

Explicit Proxy **requires an SSL decryption policy** for all traffic passing through it:

- Configure a decryption rule in the `Explicit_Proxy_Device_Group` that decrypts all HTTPS traffic
- Without SSL decryption, HTTPS sessions are forwarded uninspected and users appear as **unidentified** in traffic logs
- Create a matching decryption profile specifying a **maximum TLS version of TLS 1.3**

> ⚠️ SSL decryption is not optional for Explicit Proxy — it is a hard architectural requirement. Plan for CA certificate distribution to endpoints before deployment.

> ⚠️ **Forward Trust Certificate — confirmed via direct fetch, a precise requirement, not just implied by "SSL decryption is mandatory" above.** You must have at least one SSL Forward Proxy certificate specified as a **Forward Trust Certificate**. Exact quote: *"Failure to have a forward trust certificate will cause a commit error when you commit your explicit proxy changes."* This is a **commit-blocking** requirement, not just a "traffic goes uninspected" consequence. Confirmed only in the Panorama-facing source documentation — not independently verified as identically worded for Strata Cloud Manager, though SSL decryption is mandatory on both platforms.

> ℹ️ **Chrome TLS 1.3 Kyber flag — confirmed current as of the source docs' last-updated date (Jul 6, 2026), just days before this check.** If decryption is enabled, disable Chrome's TLS 1.3 Kyber feature: open Chrome, enter `chrome://flags/#enable-tls13-kyber` in the address bar, and disable it. Browser flags like this change over time — re-verify if this guidance is more than a few months old by the time you're reading it.

### HTTP/2 Limitation

- Explicit Proxy does **not** support HTTP/2 natively
- HTTP/2 requests are **automatically downgraded to HTTP/1.1**
- No configuration is required for this — the downgrade is automatic

### Proxy Chaining

If using a third-party proxy upstream that forwards to Explicit Proxy:
- Configure the upstream proxy to forward to the **Explicit Proxy URL** (from `Panorama > Cloud Services > Status > Network Details > Mobile Users — Explicit Proxy`)
- The Explicit Proxy URL is only available after onboarding and the first Commit & Push

### Remote-Site and HQ Users

If mobile users connect from remote sites (via Remote Networks) or from headquarters (via Service Connection):
- The endpoint must be able to **reach and route to**:
  - The **IdP** (SAML identity provider) — or your Kerberos infrastructure, if using Kerberos authentication instead (see Chapter 49's correction; this section previously assumed SAML-only)
  - The **ACS FQDN** (`*.acs.prismaaccess.com`) — SAML deployments only
  - The **Explicit Proxy URL** (`*.proxy.prismaaccess.com:8080` for SAML, `:8081` for Kerberos)
  - The **PAC file URL** (hosted by Prisma Access)

Verify routing from those network segments allows access to these cloud FQDNs before deployment.

---

## Configuration Prerequisites Checklist

| Item | Status |
|---|---|
| Explicit Proxy license activated | Verify with account team |
| Panorama ≥ 10.0.5 (Panorama-managed only — not applicable to pure SCM deployments; see note above) | Check Panorama > Dashboard |
| Antivirus content ≥ 3590 | Check Panorama > Dynamic Updates |
| SSL decryption policy planned | Required before onboarding |
| At least one Forward Trust Certificate configured | Commit-blocking if missing — see note above |
| CA cert distribution to endpoints planned | Required for SSL inspection |
| Authentication method configured — SAML IdP (Azure AD / Okta, see ch53–ch54) **or** Kerberos infrastructure (see ch49) | Not SAML-only — corrected 2026-07-09 |
| ACS/IdP FQDNs (SAML) or Kerberos infrastructure (Kerberos) accessible from all user locations | Verify routing |
| Only the first tenant supports Explicit Proxy, if using multitenant Prisma Access | See Multitenancy Restriction note above |

---

## Key Takeaways

- Panorama minimum version 10.0.5 (Panorama-managed only — unconfirmed on current docs, flagged rather than trusted at face value); antivirus content minimum version 3590 — applies where relevant
- Pure Strata Cloud Manager deployments have no Panorama version prerequisite at all — confirmed there's no Panorama in the picture
- SSL decryption is not optional — it is required for all Explicit Proxy traffic, and specifically requires at least one Forward Trust Certificate or the commit fails outright
- Create the decryption profile with TLS 1.3 as the maximum version; disable Chrome's TLS 1.3 Kyber flag if decryption is enabled (confirmed current as of Jul 2026 — re-verify if this guidance ages)
- HTTP/2 is auto-downgraded to HTTP/1.1 — no action required
- Remote-site and HQ users need routable access to the IdP or Kerberos infrastructure, ACS FQDN (SAML only), proxy URL, and PAC file URL — not SAML-only, corrected 2026-07-09
- Only the first tenant in a multitenant Prisma Access deployment supports Explicit Proxy — plan accordingly

---

*Previous: [Chapter 49 — How Explicit Proxy Works](./ch49-how-explicit-proxy-works.md)* · *Next: [Chapter 51 — App Support, Browser Guidelines & PAC File Requirements](./ch51-app-support-browser-and-pac-requirements.md)*
