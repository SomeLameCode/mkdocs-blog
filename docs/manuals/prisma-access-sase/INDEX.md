---
title: "Prisma Access Training Manual"
description: "Updated Markdown training material for PaloAlto Prisma Access — nine modules covering SASE concepts, architecture, design, deployment, and day-2 operations."
date: "2026-05-28"
tags: [prisma-access, sase, paloalto, network-security, ztna]
status: complete
---

# Prisma Access Training Manual

> A practical manual compiled from nine PaloAlto Prisma Access training modules.
>
> **How to use this manual:**
> - First-time learners: read Parts 1–3 in order (SASE overview, planning, routing fundamentals).
> - Architects & designers: focus on Parts 1–3 for conceptual grounding.
> - Deploying engineers: start with Part 5 (Activate & Configure), then Parts 7–8 for site and user onboarding.
> - Operations teams: Parts 6–8 cover Panorama, Remote Networks, and Mobile Users day-2 tasks.
> - Security architects: Parts 4 and 9 cover ZTNA Connector and Cloud Secure Web Gateway.
> - Chapters marked `[GAP]` are authored from knowledge/web sources, not the original training PDFs.

---

## Part 1 — Prisma Access Overview

| #   | Chapter                                                                                                          | Source |
| --- | ---------------------------------------------------------------------------------------------------------------- | ------ |
| 01  | [Traditional VPN & Branch Architecture — Problems & Drivers](./part1/ch01-traditional-network-challenges.md)     | vault  |
| 02  | [The Perimeter Is Now Everywhere — SASE Concepts & Key Components](./part1/ch02-sase-concepts-and-components.md) | vault  |
| 03  | [Prisma Access Architecture — Components, Presence & Services](./part1/ch03-prisma-access-architecture.md)       | vault  |
| 04  | [Prisma Access for Networks](./part1/ch04-prisma-access-for-networks.md)                                         | vault  |
| 05  | [Prisma Access for Users & SaaS Design Benefits](./part1/ch05-prisma-access-for-users.md)                        | vault  |

---

## Part 2 — Planning & Designing

| # | Chapter | Source |
|---|---------|--------|
| 06 | [Prisma Access Licensing & Management Models (Cloud vs Panorama)](./part2/ch06-licensing-and-management-models.md) | vault |
| 07 | [Service Infrastructure & Subnet Planning](./part2/ch07-service-infrastructure-planning.md) | vault |
| 08 | [Service Connections Planning (Networks & Mobile Users)](./part2/ch08-service-connections-planning.md) | vault |
| 09 | [Remote Networks Planning (Bandwidth Model & Compute Location)](./part2/ch09-remote-networks-planning.md) | vault |
| 10 | [Mobile User Deployment Planning](./part2/ch10-mobile-user-deployment-planning.md) | vault |
| 11 | [Resiliency & Redundancy Design Considerations](./part2/ch11-resiliency-and-redundancy-design.md) | vault |

---

## Part 3 — Routing & SD-WAN Design

| # | Chapter | Source |
|---|---------|--------|
| 12 | [Traffic Flow Scenarios (RN, MU, SC Permutations)](./part3/ch12-traffic-flow-scenarios.md) | vault |
| 13 | [Default Routing (Cold Potato) — With & Without Backbone](./part3/ch13-default-routing-and-backbone.md) | vault |
| 14 | [Hot Potato Routing](./part3/ch14-hot-potato-routing.md) | vault |
| 15 | [Prisma SD-WAN Overview & Architecture](./part3/ch15-sd-wan-overview-and-architecture.md) | vault |
| 16 | [SD-WAN Controllers, ION Device Communication & Trust Chain](./part3/ch16-sd-wan-controllers-and-ion-security.md) | vault |
| 17 | [ION Device Family & High-Availability Design](./part3/ch17-ion-devices-and-high-availability.md) | vault |
| 18 | [SD-WAN Design Considerations (PATH, QoS, NAT)](./part3/ch18-sd-wan-design-considerations.md) | vault |
| 19 | [Data Center Scaling Design](./part3/ch19-data-center-scaling-design.md) | vault |

---

## Part 4 — ZTNA Connector

| # | Chapter | Source |
|---|---------|--------|
| 20 | [ZTNA Connector Overview — Components, Connectors, Groups & Targets](./part4/ch20-ztna-connector-overview-and-components.md) | vault |
| 21 | [ZTNA Use Cases & Packet Flow](./part4/ch21-ztna-use-cases-and-packet-flow.md) | vault |
| 22 | [ZTNA Connector vs Service Connections & Supported Hosting Environments](./part4/ch22-ztna-connector-vs-service-connections.md) | vault |
| 23 | [Network Requirements & Prerequisites for ZTNA](./part4/ch23-ztna-network-requirements-and-prerequisites.md) | vault |
| 24 | [Enable & Configure ZTNA Connector](./part4/ch24-enable-and-configure-ztna-connector.md) | vault |
| 25 | [Onboard ZTNA Connector in VMware ESXi & Upgrade](./part4/ch25-onboard-ztna-connector-esxi-and-upgrade.md) | vault |

---

## Part 5 — Activate & Configure

| # | Chapter | Source |
|---|---------|--------|
| 26 | [Prisma Access Activation Planning Checklist](./part5/ch26-activation-planning-checklist.md) | vault |
| 27 | [Integrating Panorama with Prisma Access](./part5/ch27-integrating-panorama-with-prisma-access.md) | vault |
| 28 | [Configuring the Service Infrastructure](./part5/ch28-configuring-service-infrastructure.md) | vault |
| 29 | [IPSec Tunnel Configuration for Service Connections](./part5/ch29-ipsec-tunnel-configuration.md) | vault |
| 30 | [Configure Service Connection — Static Routes](./part5/ch30-service-connection-static-routes.md) | vault |
| 31 | [Configure Service Connection — BGP](./part5/ch31-service-connection-bgp.md) | vault |

---

## Part 6 — Panorama Operations for Prisma Access

| # | Chapter | Source |
|---|---------|--------|
| 32 | [Templates & Template Stacks](./part6/ch32-templates-and-template-stacks.md) | vault |
| 33 | [Device Groups — Concepts, Components & Hierarchy](./part6/ch33-device-groups-concepts-and-hierarchy.md) | vault |
| 34 | [Device Group Policies & Objects](./part6/ch34-device-group-policies-and-objects.md) | vault |
| 35 | [Inheritance Precedence & Predefined Templates for Prisma Access](./part6/ch35-inheritance-and-predefined-templates.md) | vault |
| 36 | [Prisma Access Zones](./part6/ch36-prisma-access-zones.md) | vault |

---

## Part 7 — Remote Networks

| # | Chapter | Source |
|---|---------|--------|
| 37 | [Remote Network Templates, Device Groups & Zone Mapping](./part7/ch37-remote-network-templates-and-zone-mapping.md) | vault |
| 38 | [Remote Network Bandwidth Allocation](./part7/ch38-remote-network-bandwidth-allocation.md) | vault |
| 39 | [Onboard Remote Network — Single IPSec Tunnel & Static Route](./part7/ch39-onboard-remote-network-static-route.md) | vault |
| 40 | [Onboard Remote Network — Single IPSec Tunnel & BGP](./part7/ch40-onboard-remote-network-bgp.md) | vault |
| 41 | [Onboard Remote Network — Multiple IPSec Tunnels, ECMP & BGP](./part7/ch41-onboard-remote-network-ecmp-bgp.md) | vault |

---

## Part 8 — Mobile Users

| # | Chapter | Source |
|---|---------|--------|
| 42 | [Mobile User Templates, Device Groups & Zone Mapping](./part8/ch42-mobile-user-templates-and-zone-mapping.md) | vault |
| 43 | [LDAP Server & Authentication Profile](./part8/ch43-ldap-and-authentication-profile.md) | vault |
| 44 | [Onboard Mobile Users — GlobalProtect](./part8/ch44-onboard-mobile-users-globalprotect.md) | vault |
| 45 | [Verify Mobile Users — GlobalProtect](./part8/ch45-verify-mobile-users-globalprotect.md) | vault |
| 46 | [GlobalProtect Split Tunneling](./part8/ch46-globalprotect-split-tunneling.md) | vault |
| 47 | [GlobalProtect App Settings](./part8/ch47-globalprotect-app-settings.md) | vault |
| 48 | [GlobalProtect App Upgrades — Staged Rollout](./part8/ch48-globalprotect-app-upgrades.md) | vault |

---

## Part 9 — Cloud Secure Web Gateway

| # | Chapter | Source |
|---|---------|--------|
| 49 | [How Explicit Proxy Works](./part9/ch49-how-explicit-proxy-works.md) | vault |
| 50 | [Explicit Proxy Configuration Guidelines](./part9/ch50-explicit-proxy-configuration-guidelines.md) | vault |
| 51 | [App Support, Browser Guidelines & PAC File Requirements](./part9/ch51-app-support-browser-and-pac-requirements.md) | vault |
| 52 | [PAC File Guidelines — Detailed & Sample](./part9/ch52-pac-file-guidelines-and-sample.md) | vault |
| 53 | [Configure Azure AD SAML Profile & Authentication Profile](./part9/ch53-configure-azure-ad-saml-profile.md) | vault |
| 54 | [Configure SAML in Azure AD for Prisma Access Explicit Proxy](./part9/ch54-configure-saml-in-azure-ad.md) | vault |

---

## Appendix

| # | Topic | Source |
|---|-------|--------|
| A | [Glossary of Terms](./appendix/appA-glossary.md) | GAP |
| B | [Key CLI & API Commands Reference](./appendix/appB-cli-and-api-commands.md) | GAP |
| C | [Architecture Quick-Reference (topology diagrams & component summary)](./appendix/appC-architecture-quick-reference.md) | GAP |
| D | [Known Documentation Ambiguities](./appendix/appD-known-documentation-ambiguities.md) | GAP |

---

## Source Material Reference

### PDF → Chapter Mapping

| Input PDF | Manual Chapter(s) |
|-----------|-------------------|
| `input-docs/1_Prisma+Access+Overview.pdf` | Ch 01–05 |
| `input-docs/2_Planning+&+Designing.pdf` | Ch 06–11 |
| `input-docs/3_Routing+&+SD-WAN+Design.pdf` | Ch 12–19 |
| `input-docs/4_ZTNA+Connector.pdf` | Ch 20–25 |
| `input-docs/5_Activate+&+Configure.pdf` | Ch 26–31 |
| `input-docs/6_Panorama+Operations+for+Prisma+Access.pdf` | Ch 32–36 |
| `input-docs/7_Remote+Networks.pdf` | Ch 37–41 |
| `input-docs/8_Mobile+Users.pdf` | Ch 42–48 |
| `input-docs/9_Cloud+Secure+Web+Gateway.pdf` | Ch 49–54 |

### GAP Entries (authored from knowledge/web)

Appendix entries have no direct PDF source section. Content will be synthesised from:
- PaloAlto Networks documentation (<https://docs.paloaltonetworks.com>)
- Prisma Access Admin Guide
- Content compiled across all nine source PDFs

GAP entries: App A, App B, App C, App D
