---
title: "GlobalProtect App Settings"
description: "How to configure GlobalProtect app behavior in Prisma Access: connection method, user control over disable/uninstall/upgrade, and sign-out options — configured via the Portal agent config."
date: 2026-05-28
tags:
  - prisma-access
  - sase
  - paloalto
  - mobile-users
  - globalprotect
  - app-settings
  - portal
  - connect-method
  - panorama
status: published
---

# Chapter 47 — GlobalProtect App Settings

The GlobalProtect app settings control how the client behaves on the endpoint: how it connects, whether users can disable or uninstall it, and how upgrades are handled. These settings are configured per agent config in the Portal.

---

## Navigation

**Panorama:**
`Panorama > Network > GlobalProtect > Portals > [select portal] > Agent > [Default Config] > App tab`

**Strata Cloud Manager:**
`Configuration > NGFW and Prisma Access > Configuration Scope > Prisma Access > GlobalProtect > GlobalProtect App > App Settings > App Configuration` — this per-config screen holds Connect Method, Disable, Uninstall, Upgrade, and Sign Out settings, confirmed matching the structure below. The **Agent Override Key** (needed for the Allow with Ticket disable option — see below) is set separately, at the **Global App Settings** level rather than per-config — the same Global/per-config distinction Chapter 48 already established for version activation, not repeated here.

---

## Connection Method

The **Connect Method** determines when and how GlobalProtect establishes the VPN tunnel:

| Connect Method | Behaviour | Requires |
|---|---|---|
| **User-logon (Always On)** | GP connects automatically when the user logs into the OS. Stays connected. | User credentials |
| **Pre-logon (Always On)** | GP connects before the user logs in using a machine certificate. Provides pre-logon tunnel for domain login scripts and GPO. | Machine certificate |
| **On-demand** | User manually opens the GP app and clicks Connect. No automatic connection. | — |
| **Pre-logon then On-demand** | Pre-logon tunnel active before user login; after login, user controls connection manually. Allows password changes while maintaining pre-logon security. | Machine certificate |

> **Recommendation:** Use **User-logon (Always On)** for corporate managed devices where persistent protection is required. Use **On-demand** for BYOD devices where users manage their own connectivity.

**Strata Cloud Manager:** confirmed via direct fetch — all four Connect Method labels above are used **identically** in SCM. No terminology difference here, unlike Portal Name Type (Chapter 44) or the Upgrade options below — checked directly rather than assumed given that pattern.

---

## User Control Options

### Disable GlobalProtect

Controls whether users can temporarily turn off the GlobalProtect app:

| Setting | Behaviour |
|---|---|
| **Allow** | User can disable GP without restriction |
| **Allow with Passcode** | User must enter an admin-defined passcode to disable |
| **Allow with Ticket** | Confirmed real, previously missing from this chapter — see workflow below |
| **Disallow** | User cannot disable GP — enforced always-on |

**Allow with Ticket — a genuinely missing feature, not just an SCM navigation gap, confirmed via direct fetch on both platform-specific docs pages:** instead of a static admin-defined passcode, the user requests a one-time disable and GlobalProtect displays a **request number**. The user shares that number with the GlobalProtect admin, who generates a **ticket** from it (using the **Agent Override Key** configured at the Global App Settings level — see the Navigation section above) and shares the ticket number back with the user, who enters it into the app to disable GlobalProtect. This is an **admin-mediated** exchange — no self-service ticket generation was found documented.

> ℹ️ **Scope confirmed: Disable only, not Uninstall.** Both fetched pages exclusively discuss ticket-based disabling — neither mentions a ticket-based option for the separate Uninstall GlobalProtect setting below. Don't assume this option extends there without checking.

---

### Uninstall GlobalProtect

Controls whether users can remove the GlobalProtect app from their device:

| Setting | Behaviour |
|---|---|
| **Allow** | User can uninstall GP without restriction |
| **Allow with Passcode** | User must enter a passcode to uninstall |
| **Disallow** | Uninstallation is blocked |

---

### Upgrade GlobalProtect

Controls how the app handles new version availability (covered in detail in Chapter 48):

| Setting | Behaviour |
|---|---|
| **Allow with Prompt** | GP notifies the user when a new version is available; user decides when to upgrade |
| **Allow Transparently** | GP upgrades automatically in the background without user interaction |
| **Allow Manually** | User manually checks for updates via "Check Version" in the GP menu |
| **Allow When on Corporate Network** | Upgrade only happens when connected to the internal corporate network |
| **Disallow** | No upgrades permitted — version remains fixed |

**Cross-checked against Chapter 48's corrected table, not re-derived:** this table already had all 5 options (unlike Chapter 48's original version, which was missing 2 of 5). The one gap here: **Strata Cloud Manager labels the "Allow When on Corporate Network" row `Internal`** — confirmed via Chapter 48's sourcing, not repeated here. Everything else in this table is confirmed identical across both platforms.

---

### Sign Out

Controls whether users can sign out of the GlobalProtect app (disconnect the VPN session and remove cached credentials):

| Setting | Behaviour |
|---|---|
| **Allow** | Users can sign out at will |
| **Disallow** | Sign-out option is hidden from users |

> Disabling sign-out is typical in high-security deployments where persistent identity enforcement is required.

---

## Recommended Settings by Deployment Type

| Setting | Corporate Managed | BYOD |
|---|---|---|
| **Connect Method** | User-logon (Always On) | On-demand |
| **Disable** | Disallow or Allow with Passcode | Allow |
| **Uninstall** | Disallow | Allow |
| **Upgrade** | Allow Transparently | Allow with Prompt |
| **Sign Out** | Disallow | Allow |

---

## Commit & Push

After configuring app settings:

1. `Commit > Commit and Push`
2. Edit Selections → Select **Prisma Access** → **Mobile Users**
3. Click **OK** → **Commit and Push**

**Strata Cloud Manager:** Commit is replaced with **Push Config**, per the terminology already established in Chapter 28 — not re-explained here.

---

## Key Takeaways

- Connect Method is the most impactful setting: User-logon (Always On) for managed endpoints; Pre-logon for pre-authentication domain scenarios; On-demand for BYOD
- Pre-logon requires machine certificates deployed to endpoints — not just user credentials
- Disable/Uninstall/Sign-out settings enforce or relax user control over the GP client
- App upgrade settings are configured here but the available version is activated separately (Chapter 48)
- Multiple agent configs can be created and ordered — first-match wins; assign different groups of users to different configs for differentiated policies
- **Allow with Ticket** is a real, confirmed 4th option for Disable GlobalProtect (admin-mediated request-number/ticket exchange via the Agent Override Key) — not previously documented in this chapter, and confirmed **not** to apply to Uninstall GlobalProtect
- Connect Method labels are confirmed identical across Panorama and SCM; the Upgrade GlobalProtect corporate-network option is the one confirmed terminology difference — SCM calls it **Internal** (see Chapter 48)
- In SCM, the Agent Override Key is set at **Global App Settings**, separate from the per-config **App Settings** screen where Disable/Uninstall/Upgrade/Sign Out live

---

*Previous: [Chapter 46 — GlobalProtect Split Tunneling](./ch46-globalprotect-split-tunneling.md)* · *Next: [Chapter 48 — GlobalProtect App Upgrades — Staged Rollout](./ch48-globalprotect-app-upgrades.md)*
