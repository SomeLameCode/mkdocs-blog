---
title: "PAC File Guidelines — Detailed & Sample"
description: "PAC file rules for Prisma Access Explicit Proxy: format constraints, upload procedure, bypass requirements, PROXY statement syntax, and a complete annotated sample PAC file."
date: 2026-05-28
tags:
  - prisma-access
  - sase
  - paloalto
  - explicit-proxy
  - cloud-swg
  - pac-file
  - javascript
  - proxy-auto-config
status: published
---

# Chapter 52 — PAC File Guidelines — Detailed & Sample

The PAC (Proxy Auto-Config) file is a JavaScript function that instructs the browser where to route web traffic. For Explicit Proxy, the PAC file directs browser traffic to the Prisma Access proxy endpoint at port 8080.

---

## PAC File Constraints

| Rule | Detail |
|---|---|
| **One PAC file per tenant (baseline case)** | **Investigated 2026-07-09, confirmed accurate as the baseline — not a contradiction with Forwarding Profiles below.** Exact quote: "You can only host one PAC file for use with Prisma Access." Only a single PAC file is supported in this baseline model — all users share it. |
| **Format** | ASCII text only — create and save in a text editor (VI, Vim, Notepad++) |
| **Upload timing** | Upload **after** the Explicit Proxy onboarding Commit & Push completes |
| **Commit required after upload?** | No — upload takes effect immediately, no commit/push needed |
| **IPv4 only** | Only IPv4 addresses in `PROXY` statements — do not use IPv6 |
| **Deletion** | PAC files **cannot be deleted** — upload a new file to overwrite the existing one |
| **Proxy URL in PROXY statement** | At least one valid Explicit Proxy URL must appear in every non-bypassed `return "PROXY ..."` statement |

> ℹ️ **Forwarding Profiles — a separate, additional mechanism, not a contradiction of the rule above.** Confirmed GA (referenced in Palo Alto's "What's New" documentation dated at least May 2024, not a preview feature). Exact description: *"Enables the use of multiple PAC files for different user groups or systems. Also supports the creation of forwarding rules for defining the direction of web traffic, providing a simpler alternative to creating and maintaining a PAC file."* In other words: this chapter's single-PAC-file model is still the accurate baseline for a straightforward deployment — Forwarding Profiles is the escape hatch if you need multiple PAC files (one per user group/system) or want to skip authoring a PAC file entirely in favor of simpler forwarding rules. Full configuration of Forwarding Profiles is out of scope for this chapter.

---

## Upload Location

**Panorama:**
`Panorama > Cloud Services > Configuration > Mobile Users — Explicit Proxy > PAC File`

**Strata Cloud Manager — could not be confirmed, flagged rather than guessed:** a general NGFW cloud-management PAC/Proxy Auto Configuration path exists (`Device Settings > Configuration Scope`), but this is confirmed to be a **different, unrelated NGFW context**, not the Prisma Access Explicit Proxy-specific screen this chapter is about — exactly the mix-up worth avoiding. No Explicit-Proxy-specific SCM PAC upload path could be confirmed from available documentation despite targeted searching. If you're on SCM, verify the current path directly in your tenant (likely under `Configuration > NGFW and Prisma Access > Configuration Scope > Prisma Access > Mobile Users`, consistent with the pattern established elsewhere in Part 8/9, but this specific screen wasn't independently confirmed).

The Explicit Proxy URL required for the `PROXY` statement is found at:
`Panorama > Cloud Services > Status > Network Details > Mobile Users — Explicit Proxy`

---

## Required Bypass Rules

Every PAC file deployed with Explicit Proxy must bypass:

| Traffic | Why |
|---|---|
| `*.acs.prismaaccess.com` | SAML Assertion Consumer Service — must be directly reachable |
| IdP domain (e.g. `*.okta.com`, `*.azure.com`) | SAML authentication redirects must not loop through the proxy |
| Cloud Identity Engine (CIE) domain(s) | **Confirmed missing from this chapter, added 2026-07-09.** Palo Alto's Explicit Proxy Best Practices guidance: *"When setting up the PAC file, bypass all SAML, CIE, and Authentication Cache Service (ACS) URLs."* No specific CIE domain pattern was confirmed in the source — deliberately not inventing a placeholder here; confirm the exact domain(s) for your CIE tenant before writing the bypass rule. |

Additional common bypasses:
- Localhost and RFC 1918 (private) IP ranges — prevent internal traffic from routing to the cloud proxy
- FTP URLs — Explicit Proxy does not handle FTP

**Kerberos — brief cross-reference, not elaborated here:** Kerberos-authenticated traffic (port 8081, see Chapter 49) targets servers, IoT devices, and headless machines rather than typical browser users, and doesn't go through a browser SAML redirect. No source checked (the PAC File Guidelines page, the Explicit Proxy overview, or the Explicit Proxy Best Practices page) documents Kerberos traffic needing its own `PROXY` statement or PAC-specific handling — the reasonable inference is that it's handled outside the PAC/browser-redirect mechanism entirely, but this wasn't explicitly confirmed either way, so treat it as likely rather than certain.

**Trusted Source Address — brief cross-reference, not elaborated here:** see Chapter 49 for the full feature (source-IP allowlist that skips authentication). Whether trusted-IP traffic still routes through the Explicit Proxy via the PAC file's `PROXY` statement (just without an auth challenge) or bypasses proxy routing entirely was **not confirmed** in available documentation — don't assume either behavior without checking your own deployment.

---

## PROXY Statement Syntax

```
return "PROXY <fqdn>.proxy.prismaaccess.com:8080";
```

- The FQDN comes from `Status > Network Details > Mobile Users — Explicit Proxy`
- Port must be **8080**
- IPv4 addresses may be used but FQDNs are recommended (IPs can change after re-provisioning)

---

## Sample PAC File

The sample below is the reference implementation from PaloAlto training materials. Adapt the bypass rules to match your environment.

```javascript
function FindProxyForURL(url, host) {

  /* Bypass localhost and private IP ranges */
  var resolved_ip = dnsResolve(host);
  if (isPlainHostName(host) ||
      shExpMatch(host, "*.local") ||
      isInNet(resolved_ip, "10.0.0.0",     "255.0.0.0")    ||
      isInNet(resolved_ip, "172.16.0.0",   "255.240.0.0")  ||
      isInNet(resolved_ip, "192.168.0.0",  "255.255.0.0")  ||
      isInNet(resolved_ip, "127.0.0.0",    "255.255.255.0"))
    return "DIRECT";

  /* Bypass FTP */
  if (url.substring(0, 4) == "ftp:")
    return "DIRECT";

  /* Bypass SAML IdP — replace *.okta.com with your IdP domain if different */
  if (shExpMatch(host, "*.okta.com") ||
      shExpMatch(host, "*.oktacdn.com"))
    return "DIRECT";

  /* Bypass ACS — required for Prisma Access SAML authentication */
  if (shExpMatch(host, "*.acs.prismaaccess.com"))
    return "DIRECT";

  /* Bypass Cloud Identity Engine (CIE) if used — confirmed required by Palo Alto's
     Explicit Proxy Best Practices guidance ("bypass all SAML, CIE, and ACS URLs"),
     but no specific CIE domain pattern was confirmed to hardcode here. Add your
     tenant's actual CIE domain(s) as a shExpMatch() rule following the pattern above. */

  /* Forward all remaining traffic to Prisma Access Explicit Proxy */
  return "PROXY foo.proxy.prismaaccess.com:8080";
}
```

> Replace `foo.proxy.prismaaccess.com` with the actual Explicit Proxy URL from `Status > Network Details > Mobile Users — Explicit Proxy`.

---

## Adapting for Azure AD

If using Azure AD as the SAML IdP (see ch53–ch54), replace the Okta bypass with:

```javascript
  /* Bypass Azure AD SAML IdP */
  if (shExpMatch(host, "*.microsoftonline.com") ||
      shExpMatch(host, "*.microsoft.com") ||
      shExpMatch(host, "login.windows.net"))
    return "DIRECT";
```

> Always include bypasses for both the login domain and the token endpoint domain for your IdP.

---

## Key Takeaways

- One PAC file per tenant is the accurate baseline — all users share the same file, craft it carefully — but this is **not the whole picture**: Forwarding Profiles (confirmed GA, investigated 2026-07-09) is a separate, additional mechanism for multiple PAC files or PAC-free forwarding rules, layered on top rather than contradicting the baseline
- Upload the PAC file **after** the Explicit Proxy Commit & Push — not before
- No commit/push needed after PAC file upload — changes are immediate
- Must bypass IdP domain, `*.acs.prismaaccess.com`, and **Cloud Identity Engine (CIE) domains** (confirmed missing, added 2026-07-09) — missing these causes a SAML redirect loop
- IPv4 only in `PROXY` statements — do not use IPv6 addresses
- PAC files cannot be deleted — upload a new version to replace the current one
- Kerberos traffic (ch49) likely doesn't need PAC-specific handling — not explicitly confirmed, treated as probable rather than certain
- SCM's Explicit-Proxy-specific PAC upload path could not be confirmed — a general NGFW PAC path exists but is a different, unrelated context; don't conflate the two

---

*Previous: [Chapter 51 — App Support, Browser Guidelines & PAC File Requirements](./ch51-app-support-browser-and-pac-requirements.md)* · *Next: [Chapter 53 — Configure Azure AD SAML Profile & Authentication Profile](./ch53-configure-azure-ad-saml-profile.md)*
