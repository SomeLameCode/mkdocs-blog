---
title: "Known Documentation Ambiguities"
description: "A consolidated tracker of every place Palo Alto's own current documentation genuinely contradicts itself, and every claim investigated during this project that could not be confirmed one way or the other."
date: 2026-07-10
tags:
  - prisma-access
  - sase
  - paloalto
  - ambiguity
  - contradiction
  - unconfirmed
  - reference
  - audit-trail
status: published
---

# Appendix D — Known Documentation Ambiguities

This appendix is not a list of errors in this manual. It's a consolidated record of two different things this project ran into while researching Palo Alto's own current documentation: (1) cases where two genuinely current, live documentation pages state directly conflicting facts, and (2) cases where a specific claim — often one that looked plausible and kept resurfacing in search results — could not be verified against any primary source, and was therefore left out of the manual or explicitly flagged as unconfirmed rather than asserted as fact. In every case below, excluding or flagging the claim was the correct call at the time; this appendix exists so those decisions are trackable in one place instead of scattered across individual chapters' `pdf_delta` fields. Several of these — especially the two confirmed contradictions — could likely be resolved in minutes by checking the actual setting in a live Prisma Access tenant, which is often faster than further documentation research.

---

## Confirmed Contradictions in Palo Alto's Own Documentation

### 1. Backbone Routing default (Service Connections) — ch13, ch14

Which backbone routing option is the actual default for a new Service Connection deployment.

- **Quote A** — ["Configure a Service Connection"](https://docs.paloaltonetworks.com/prisma-access/administration/prisma-access-service-connections/configure-a-service-connection): *"you can take advantage of load balancing in your Prisma Access deployment by selecting Allow asymmetric routing and load sharing across Service Connections **(the default setting)**."* — i.e. **asymmetric-routing-with-load-share** is the default.
- **Quote B** — ["Configure the Prisma Access Service Infrastructure"](https://docs.paloaltonetworks.com/prisma-access/administration/prisma-access-setup/configure-the-prisma-access-service-infrastructure), under its own "Backbone Routing" heading: *"By default, the Prisma Access backbone requires that you have a symmetric network path for the traffic returning from the data center or headquarters location by way of a service connection."* — i.e. **no-asymmetric-routing** (symmetric) is the default.

Both pages were confirmed live and current at time of research — this isn't a stale-page-vs-current-page situation. ch13 reports this as an unresolved, flagged contradiction rather than picking one side; ch14 cross-references the same finding.

**Resolve it yourself:** check your tenant's actual Backbone Routing setting under Service Connection / Service Infrastructure configuration — this is a quick, definitive check that settles the question for your deployment without relying on either doc page.

### 2. Aggregate vs. Site-Based bandwidth model default (Remote Networks) — ch06, ch09, ch11, ch38, appA

Which bandwidth allocation model (Aggregate, pooled per compute location, vs. Site-Based, guaranteed per branch) is the default for a new Remote Network deployment on Prisma Access 6.0+.

- **Quote A** — ["Onboard a Remote Network"](https://docs.paloaltonetworks.com/prisma-access/administration/prisma-access-remote-networks/onboard-a-remote-network): *"New Prisma Access deployments starting with Prisma Access 6.0 allocate bandwidth per site and this procedure does not apply."* — i.e. **Site-Based** is the default, and the Aggregate-model procedure on that page no longer applies to new deployments.
- **Quote B** — ["Planning Checklist for Remote Networks"](https://docs.paloaltonetworks.com/prisma-access/administration/prisma-access-remote-networks/planning-checklist-for-remote-networks): *"Bandwidth for your remote network locations are allocated at an aggregate level per compute location... The aggregate bandwidth model is available for all new deployments."* — i.e. **Aggregate** is (still) available and implied as the default for new deployments.

A third page, ["Allocate Remote Network Bandwidth"](https://docs.paloaltonetworks.com/prisma-access/administration/prisma-access-remote-networks/allocate-remote-network-bandwidth), muddies this further by describing both models as a choice on 6.0+ ("you can choose between allocating bandwidth per site or by compute location") rather than confirming a single default either way — consistent with ch38's own hypothesis that this may reflect terminology drift or pages updated on different schedules rather than a clean either/or answer.

This finding recurs across ch06 (licensing units), ch09 (Remote Networks planning), and ch11 (resiliency design, since Site-Based-specific redundancy features depend on which model applies) — all cross-reference ch38 rather than re-litigating it.

**Resolve it yourself:** onboard a test remote network site (or check an existing one) and look at which bandwidth configuration screen/model it actually landed on — faster and more reliable than reconciling these pages.

---

## Claims Investigated But Not Confirmed

### SOCKS5 proxy support for Explicit Proxy — ch49

**Claimed:** That Explicit Proxy supports SOCKS5 proxying, not just HTTP/HTTPS forward proxying. **Origin:** Surfaced repeatedly across aggregated search-engine summaries, phrased specifically enough to sound like a real, documented feature. **Why unconfirmed:** Eight separate search/fetch attempts could not locate a verbatim description, port number, or availability caveat for SOCKS5 on any actual Palo Alto documentation page. Not included as fact in ch49 — flagged as unconfirmed, with a note to verify directly with Palo Alto if this capability matters to your deployment.

### Mobile User Regional Redundancy as a standalone feature — ch11

**Claimed:** That "Mobile User Regional Redundancy" exists as its own distinct, named feature (as the original source material implied). **Origin:** A legacy, versioned Panorama-admin documentation URL. **Why unconfirmed:** That URL redirected to a generic Prisma Access overview page on every attempt — it never resolved to content actually describing a standalone feature by that name. ch11 rewrote the section to describe this honestly as "previously stated" rather than confirmed, and flagged it as likely just a restatement of the already-covered Service Connection Multi-Cloud Redundancy content rather than a genuinely separate capability.

### "Unlimited" Service Connections / ZTNA Connectors licensing tier — ch04

**Claimed:** That a licensing tier exists offering unlimited Service Connections or ZTNA Connectors per tenant. **Origin:** Traced to a search-aggregation artifact — a plausible-sounding claim that appeared in synthesized search results but not in any primary source. **Why unconfirmed:** The actual Prisma Access 6.0 release notes, fetched directly, do not mention any such tier. The current, confirmed limit (up to 200 Service Connections per tenant) was kept unchanged; the unlimited claim was not added.

### AppFabric as the protocol basis for Secure Fabric Links — ch15, ch16

**Claimed:** That "AppFabric" is a specific, distinct, proprietary tunnel-encapsulation protocol underlying Secure Fabric Links (Prisma SD-WAN's native ION-to-ION communication mechanism). **Origin:** A single secondary/blog source. **Why unconfirmed:** "AppFabric" itself is a real, first-party Palo Alto term (Prisma SD-WAN's branded name for its secure application fabric concept generally) — but a direct, targeted fetch of the actual Secure Fabric Tunnels configuration page did not mention AppFabric at all, so the more specific claim that it's the protocol basis for Secure Fabric Links specifically could not be confirmed. ch15 flags this as an open question for a future decision rather than asserting the linkage; ch16 documents the confirmed Secure Fabric Links correction on its own, independent of this unresolved detail.

### Route summarization + Hot Potato routing stops AS-PATH prepending — ch13, ch14

**Claimed:** That enabling route summarization together with Hot Potato routing causes Prisma Access to stop AS-PATH prepending, contradicting ch14's core claim that Hot Potato is "unaffected by route summarisation." **Origin:** Appeared repeatedly, attributed to a specific documentation page, in aggregated search-engine summaries. **Why unconfirmed:** Multiple direct fetches of that exact page, including an exact-phrase search, found nothing supporting the claim. This matches an established pattern in this project (see SOCKS5 above, the unlimited-licensing claim above, and Mobile User Regional Redundancy above) of WebSearch-synthesis surfacing specific, plausible-sounding claims that don't actually exist on the page they're attributed to. The correction was **not** applied — ch14's original claim stands, softened only to note it isn't a directly-quoted fact either, just unrefuted by everything checked. This also required correcting ch13, which had independently picked up the same unverified claim before ch14's review caught it.

### "5 locations combined across Mobile Users + Remote Networks" — ch10

**Claimed:** That the Local license's default 5-location limit is a single combined pool shared across Mobile User and Remote Network compute locations, rather than two separate 5-location allowances. **Origin:** A Palo Alto community forum thread, not official documentation. **Why unconfirmed:** No official Palo Alto documentation page states this either way. Left as-is in ch10 since it's consistent with the community source, but explicitly not asserted as independently docs-confirmed.

### SCM zone-creation mechanism for Remote Networks — ch37

**Claimed (implicitly, by omission):** That Strata Cloud Manager has a documented, direct equivalent to Panorama's manual Remote-Trust/Remote-Untrust zone creation and mapping steps. **Why unconfirmed:** A direct fetch plus two targeted searches found that the primary onboarding documentation page documents detailed Panorama zone steps, but the parallel SCM section of the same page doesn't mention zones at all, and no dedicated SCM zone-configuration page for Remote Networks could be found. ch37 flags this as an open documentation gap rather than assuming SCM either has or lacks an equivalent step.

### Local-firewall-rule layer position in SCM's policy model — ch34

**Claimed (implicitly):** Where a Strata Cloud Manager-managed Prisma Access deployment's "local firewall rule" layer sits relative to Shared/folder-level pre-rules and post-rules, mirroring Panorama's rule evaluation order. **Why unconfirmed:** Could not be confirmed from the fetched SCM documentation. ch34 flagged this rather than assuming the Panorama ordering carries over unchanged.

### "Region" field for SCM Remote Network onboarding — ch39

**Claimed (implicitly):** That SCM's Remote Network onboarding form includes a distinct "Region" field alongside Site Name, Prisma Access Location, and IPSec Termination Node. **Why unconfirmed:** Could not be confirmed on the fetched SCM documentation. Deliberately left out of ch39's confirmed SCM field list rather than assumed present.

### IP Allowlist step for SCM Mobile Users onboarding — ch44

**Claimed (implicitly):** That SCM's Mobile Users onboarding flow includes an equivalent to Panorama's Step 7 (IP Allowlist for trusted SaaS access). **Why unconfirmed:** This was the one step in ch44's 11-step Panorama flow with literally no SCM mention anywhere in the source documentation — not even in the chapter's own top-level SCM consolidation note. ch44 added an explicit "not independently confirmed for SCM" callout rather than silently omitting or assuming parity.

### 200-entry split-tunnel domain/application limit for SCM — ch46

**Claimed (implicitly):** That SCM's GlobalProtect split-tunnel configuration carries the same 200-domain-entry / 200-application-entry limit documented for Panorama. **Why unconfirmed:** This specific limit was not found restated anywhere in the SCM Tunnel Settings documentation. ch46 flagged it as unconfirmed for SCM rather than assuming the Panorama figure applies unchanged.

### Dedicated SCM staged-rollout guide for GlobalProtect app upgrades — ch48

**Claimed (implicitly):** That Strata Cloud Manager has its own documented staged/pilot-group rollout mechanism for GlobalProtect app upgrades, parallel to Panorama's. **Why unconfirmed:** Only a Panorama-specific "Stagger GlobalProtect App Updates" page could be found; no dedicated SCM equivalent guide was located. ch48 notes this rather than inventing an SCM procedure to force parity.

### Panorama minimum version 10.0.5 for Explicit Proxy — ch50

**Claimed:** That Panorama 10.0.5 is the minimum supported version for Explicit Proxy. **Origin:** The chapter's own prior content (inherited from earlier research). **Why unconfirmed:** Could not be reconfirmed anywhere on the current live Explicit Proxy configuration guidelines page during this pass. ch50 flags this as possibly stale rather than silently keeping it or silently deleting it.

### SCM PAC file upload navigation path for Explicit Proxy — ch52

**Claimed (implicitly):** That Strata Cloud Manager has a documented, Explicit-Proxy-specific PAC file upload path mirroring Panorama's `Cloud Services > Status > Network Details` location. **Why unconfirmed:** Could not be confirmed. A general NGFW cloud-management PAC file path does exist in SCM documentation, but it's a different, unrelated context (not specific to Explicit Proxy) — ch52 explicitly notes this as a trap to avoid, not presented as the answer to the original question.

### Specific Cloud Identity Engine (CIE) bypass domain pattern for PAC files — ch52

**Claimed (implicitly):** A specific domain pattern to hardcode as a PAC file bypass rule for Cloud Identity Engine traffic. **Why unconfirmed:** Palo Alto's own Explicit Proxy Best Practices guidance confirms in principle that CIE domains must be bypassed ("bypass all SAML, CIE, and Authentication Cache Service (ACS) URLs"), but no specific CIE domain pattern was found to hardcode as an example. ch52 (and appA/appB/appC, which cross-reference this same fix) deliberately does not invent a placeholder domain — confirm your own tenant's actual CIE domain(s) before writing this bypass rule.

### Kerberos-specific PAC file bypass/trust considerations — ch51, ch52

**Claimed (implicitly):** That Kerberos-based Explicit Proxy deployments follow the same PAC file bypass rules as SAML deployments (IdP/ACS URL bypasses). **Why unconfirmed:** Not found documented in the sources checked. ch51/ch52 note that Kerberos deployments likely have their own, different bypass/trust considerations (Kerberos infrastructure reachability rather than IdP/ACS URLs) but don't assert specifics that weren't independently confirmed.

### Trusted Source Address routing behavior — ch49, ch52

**Claimed (implicitly):** Whether traffic from an allowlisted Trusted Source Address still routes through the Explicit Proxy via the PAC file's `PROXY` statement (just without an authentication challenge), or bypasses proxy routing entirely. **Why unconfirmed:** Not confirmed in available documentation either way. ch49/ch52 note this as an open question rather than assuming one behavior.

### "Simplified forwarding profiles" as a Prisma Access Agent differentiator — ch05

**Claimed:** That "simplified forwarding profiles" is a specific, named capability distinguishing Prisma Access Agent from GlobalProtect. **Origin:** Raised as a candidate differentiator during ch05's Prisma Access Agent research. **Why unconfirmed:** No primary-source confirmation was found for this specific claim. Not asserted in ch05's otherwise substantial Prisma Access Agent section.

### "100+ PoPs across 76+ countries" — ch03

**Claimed:** That Prisma Access's global footprint spans 76+ countries specifically (in addition to 100+ points of presence). **Why unconfirmed:** A direct fetch of Palo Alto's Prisma Access Locations page confirmed the "100+ locations" figure but found no country count stated in any current source. ch03 softened this to "100+ locations worldwide," removing the unconfirmed country-count figure rather than keeping it on the strength of the original source material alone.

### SCM Configuration API response/request body schemas — appB

**Claimed (implicitly):** Specific field names for the Prisma Access Configuration API's job-status responses and remote-network create/modify request bodies. **Why unconfirmed:** Direct fetches of the relevant pan.dev endpoint pages confirmed the exact URL paths and methods, but not the exact response body schema (e.g. the job-status field name and its possible enum values) or the full request body schema for writing a remote network object. appB describes these operations in prose and points to pan.dev directly rather than inventing plausible-looking field names or example JSON.

---

*Previous: [Appendix C — Architecture Quick-Reference](./appC-architecture-quick-reference.md)*
