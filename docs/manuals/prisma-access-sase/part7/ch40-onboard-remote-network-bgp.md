---
title: "Onboard Remote Network — Single IPSec Tunnel & BGP"
description: "Step-by-step onboarding of a branch site using a single IPSec tunnel with BGP for dynamic route exchange — BGP peer settings, route advertisement options, and backup WAN BGP config."
date: 2026-05-28
tags:
  - prisma-access
  - sase
  - paloalto
  - remote-networks
  - ipsec
  - bgp
  - onboarding
  - panorama
  - dynamic-routing
status: published
---

# Chapter 40 — Onboard Remote Network — Single IPSec Tunnel & BGP

BGP enables dynamic route exchange between Prisma Access and the branch CPE. Instead of manually declaring static routes, the CPE advertises branch subnets via eBGP and Prisma Access learns them automatically. This is the preferred method for branches with dynamic or changing subnet configurations.

---

## When to Use BGP

| Scenario | Use BGP |
|---|---|
| Branch subnets change frequently | Yes |
| CPE supports eBGP peering | Yes |
| Need Prisma Access to advertise a default route to the branch | Yes |
| Need to summarise mobile user routes toward the branch | Yes |
| CPE does not support BGP | Use static routes (ch39) |

---

## Onboarding Steps — BGP Configuration

Steps 1–4 (name, location, termination node, primary tunnel) are **identical to ch39**. The difference begins at Step 8 — the BGP tab.

### Step 8 — Enable BGP

Click the **BGP** tab on the remote network configuration screen.

Toggle **Enable** to turn on BGP for this remote network connection.

**Navigation (Strata Cloud Manager):**
`Configuration > NGFW and Prisma Access > Configuration Scope > Prisma Access > Remote Networks > (add/edit the site) > Enable BGP for Dynamic Routing`

---

### BGP Route Advertisement Options

Three options control what Prisma Access advertises to the branch CPE:

| Option | Effect | When to Enable |
|---|---|---|
| **Summarize Mobile User Routes** | Prisma Access aggregates mobile user /24 blocks into larger summary prefixes before advertising to the CPE | CPE has limited route table capacity; reduces advertisement volume |
| **Advertise Default Route** | Prisma Access originates a `0.0.0.0/0` default route and advertises it to the branch via BGP | Branch should send all internet traffic through Prisma Access (full-tunnel mode) |
| **Don't Export Routes** | **Corrected 2026-07-09** — Prisma Access **stops sending its own BGP advertisements to the branch** (local routes, and prefixes learned from other service connections, remote networks, and mobile user subnets), but **continues learning routes from the branch as normal**. This is the opposite of what this table previously said. | Branch should not learn about the rest of the Prisma Access-connected network via BGP — e.g. the branch only needs its own local subnets reachable, not full mesh visibility |

> ⚠️ **Advertise Default Route** can cause routing loops if the branch also has a local default route — use with care and verify the CPE routing table after enabling.

> 📷 [PaloAlto screenshot — BGP tab in remote network onboarding](https://docs.paloaltonetworks.com/prisma-access/administration/prisma-access-remote-networks/enable-routing-for-your-remote-network)

> ℹ️ **If you configure both static routes and BGP for the same subnet, static routes take precedence.** This applies regardless of management platform. Static routes are declared per Chapter 39 — if a subnet is covered by both a static route and a BGP-learned route, the static entry wins.

---

### BGP Peer Settings

| Field | Notes |
|---|---|
| **Peer AS** | BGP AS number of the branch CPE (RFC 6996 private AS: 64512–65534) |
| **Peer IP Address** | IP address of the CPE's BGP-peering interface (inside the IPSec tunnel) |
| **Local IP Address** | Optional — only required if the CPE needs an explicit source IP for the BGP session |
| **Secret / Passphrase** | BGP MD5 authentication — must match the CPE configuration |
| **MRAI Timer** | Minimum Route Advertisement Interval: 1–600 seconds (default: 30s) — controls how often BGP updates are sent |

**Strata Cloud Manager:** confirmed the same field names as Chapter 31's Service Connection BGP section — **Peer IP Address**, **Peer AS**, **Local IP Address**, **Secret** — cross-reference that chapter rather than repeating the field list. MRAI (1–600s, default 30s) is confirmed the same for Remote Networks as it was for Service Connections in Chapter 31. One terminology note worth knowing: some Remote Networks documentation and setup screens refer to the same two fields by alternate names — **"Router ID"** (for Peer IP Address) and **"Branch Router Autonomous System (AS) Number"** (for Peer AS) — the underlying configuration fields are the same, just labeled differently depending on which screen or doc you're looking at.

---

### Secondary WAN BGP Configuration

If a **backup tunnel** is configured (Secondary WAN), BGP can be configured for it separately:

- Select **Same as Primary WAN** to reuse the same BGP settings (peer AS, options) — only the peer IP differs
- Or configure separate BGP settings for the backup peer if the backup CPE interface has different parameters

---

## Commit & Push

Same as ch39:
1. `Commit > Commit and Push`
2. Edit Selections → **Prisma Access** → **Remote Networks** → **OK** → **Commit and Push**

> 📷 [PaloAlto screenshot — Commit and Push for BGP remote network](https://docs.paloaltonetworks.com/prisma-access/administration/prisma-access-remote-networks/onboard-a-remote-network)

---

## Verify BGP Status

`Panorama > Cloud Services > Status > Remote Networks`

Confirm:
- **Tunnel Status = Connected**
- **BGP Status = Established**

If BGP stays in **Active** or **Connect** state, check:
- Peer AS matches on both sides
- Peer IP is reachable through the IPSec tunnel
- MD5 passphrase matches
- CPE is sending BGP Open messages (firewall between CPE and tunnel interface may be blocking TCP 179)

---

## Key Differences vs Static Routes

| Aspect | Static Routes | BGP |
|---|---|---|
| Route updates | Manual — requires re-push for changes | Automatic — CPE advertises new subnets dynamically |
| CPE requirement | Any CPE | CPE must support eBGP |
| Default route to branch | Not supported | Supported via Advertise Default Route |
| Mobile user route summarisation | Not applicable | Optional — reduces advertisement volume |
| MED attribute from CPE | N/A | **Not honoured** by Prisma Access — load balancing decisions are not influenced by CPE MED |

---

## Key Takeaways

- BGP enables dynamic subnet distribution — CPE advertises branch routes without manual static route config
- Three advertisement options: Summarize Mobile User Routes, Advertise Default Route, Don't Export Routes — enable only what is needed
- **Don't Export Routes** stops Prisma Access from *sending* its own BGP advertisements to the branch — it still *learns* routes from the branch as normal (corrected 2026-07-09; this table previously had the direction backwards)
- Advertise Default Route sends a `0.0.0.0/0` to the branch — verify CPE routing before enabling
- If both static routes and BGP cover the same subnet, the static route wins — applies regardless of management platform
- Prisma Access does **not** honour BGP MED attributes from the CPE — cannot use MED for load balancing
- MRAI timer controls BGP update frequency (default 30s); reduce for faster convergence in lab/low-latency environments
- SCM's BGP peer fields match Chapter 31's Service Connection fields exactly; "Router ID" and "Branch Router Autonomous System (AS) Number" are alternate names seen for Peer IP Address/Peer AS on some screens

---

*Previous: [Chapter 39 — Onboard Remote Network — Static Route](./ch39-onboard-remote-network-static-route.md)* · *Next: [Chapter 41 — Onboard Remote Network — ECMP & BGP](./ch41-onboard-remote-network-ecmp-bgp.md)*
