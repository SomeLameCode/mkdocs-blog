---
title: "Prisma Access / SASE — Training Manual"
description: "A self-contained Palo Alto Prisma Access and SASE reference manual compiled from a nine-module training course — from traditional network limitations through Zero Trust, remote networks, mobile users, and explicit proxy."
date: 2026-07-10
tags:
  - prisma-access
  - sase
  - paloalto
  - network-security
  - ztna
  - reference
status: published
---

# Prisma Access / SASE — Training Manual

A self-contained reference manual covering Palo Alto Networks' Prisma Access — from the traditional networking problems that drove SASE adoption, through planning and design, routing and SD-WAN, ZTNA Connector, activation, day-2 Panorama operations, and remote network/mobile user onboarding — compiled from a structured training course into one progressive resource.

---

## Table of Contents

1. [What Is It?](#1-what-is-it)
2. [Who Is It For?](#2-who-is-it-for)
3. [What's Inside](#3-whats-inside)
4. [How It Was Built](#4-how-it-was-built)
5. [How to Use It](#5-how-to-use-it)
6. [Status](#6-status)
7. [Manual Contents](#7-manual-contents)

---

## 1. What Is It?

**Prisma Access / SASE** is a 54-chapter reference manual covering Palo Alto Networks' Prisma Access platform end to end — from the architectural case for SASE through hands-on activation, configuration, and day-2 operations. It was compiled from nine training modules and four appendix chapters authored from official documentation, brought together into one structured, self-contained Markdown resource.

The problem it solves: Prisma Access documentation is split across nine separate PDFs, PaloAlto's own admin guides, and scattered vendor knowledge base articles. This manual consolidates all of it into a single document with a clear progression from "why SASE" through to "how to configure and troubleshoot it."

> **Key idea:** Everything from why hub-and-spoke networking broke down to configuring SAML authentication for Explicit Proxy — in one place, in reading order.

Want to see these concepts applied to a real deployment? The [Prisma Access SASE Implementation](../projects/prisma-access-sase-implementation.md) case study covers a 30-branch-site, 5,000-endpoint rollout built on exactly this architecture.

---

## 2. Who Is It For?

The manual is structured for five entry points — pick the one that matches your role.

### First-time learners

Start at **[Chapter 1 — Traditional VPN & Branch Architecture: Problems & Drivers](prisma-access-sase/part1/ch01-traditional-network-challenges.md)**. Read **Parts 1–3** in order — SASE overview, planning, and routing fundamentals — before moving into deployment-specific material.

### Architects & designers

**Parts 1–3** (Chapters 01–19) cover the conceptual grounding: SASE concepts, Prisma Access architecture, licensing and management models, and routing/SD-WAN design.

### Deploying engineers

Start at **[Part 5 — Activate & Configure](prisma-access-sase/part5/ch26-activation-planning-checklist.md)** (Chapter 26), then continue into **Parts 7–8** for Remote Network and Mobile User onboarding.

### Operations teams

**Parts 6–8** (Chapters 32–48) cover Panorama day-2 operations, Remote Networks, and Mobile Users — the material you'll return to most often after go-live.

### Security architects

**[Part 4 — ZTNA Connector](prisma-access-sase/part4/ch20-ztna-connector-overview-and-components.md)** (Chapters 20–25) and **[Part 9 — Cloud Secure Web Gateway](prisma-access-sase/part9/ch49-how-explicit-proxy-works.md)** (Chapters 49–54) cover the platform's two zero-trust, agentless access models.

---

## 3. What's Inside

| Part | Chapters | Topics |
|---|---|---|
| **1 — Prisma Access Overview** | Ch 01–05 | Why hub-and-spoke and traditional VPN broke down; SASE concepts and key components; Prisma Access architecture (MU-SPN, RN-SPN, service infrastructure); how networks and users each connect |
| **2 — Planning & Designing** | Ch 06–11 | Licensing and management models (Strata Cloud Manager vs. Panorama); service infrastructure and subnet planning; Service Connection, Remote Network, and Mobile User design; resiliency and redundancy |
| **3 — Routing & SD-WAN Design** | Ch 12–19 | Traffic flow scenarios; default (cold potato) and hot potato routing; Prisma SD-WAN architecture, ION devices, controllers, and data centre scaling |
| **4 — ZTNA Connector** | Ch 20–25 | ZTNA Connector components, use cases, and packet flow; comparison with Service Connections; network prerequisites; configuration and ESXi onboarding |
| **5 — Activate & Configure** | Ch 26–31 | Activation planning checklist; integrating Panorama; service infrastructure configuration; IPSec tunnels and static/BGP routing for Service Connections |
| **6 — Panorama Operations** | Ch 32–36 | Templates and template stacks; device groups and hierarchy; policies and objects; inheritance precedence; Prisma Access zones |
| **7 — Remote Networks** | Ch 37–41 | Remote network templates and zone mapping; bandwidth allocation; onboarding via static route, BGP, and multi-tunnel ECMP/BGP |
| **8 — Mobile Users** | Ch 42–48 | Templates and zone mapping; LDAP and authentication profiles; GlobalProtect onboarding, verification, split tunneling, app settings, and staged upgrades |
| **9 — Cloud Secure Web Gateway** | Ch 49–54 | How Explicit Proxy works; configuration guidelines; app/browser support and PAC file requirements; Azure AD (Microsoft Entra ID) SAML integration |
| **Appendix** | A–D | Glossary of terms, CLI & API command reference, architecture quick-reference, known documentation ambiguities |

---

## 4. How It Was Built

| Source | What it contributed |
|---|---|
| **Training modules** | Nine PaloAlto Prisma Access training PDFs — the core narrative content for Chapters 01–54 |
| **Gap chapters** | The four appendix chapters, authored from [PaloAlto Networks documentation](https://docs.paloaltonetworks.com) and the Prisma Access Admin Guide where the training material didn't cover a topic (glossary, CLI/API reference, architecture quick-reference, documentation ambiguities) |

Unlike a straight PDF-to-Markdown conversion, this manual went through a dedicated accuracy-verification pass cross-referencing claims against PaloAlto's current documentation — catching and correcting several since-changed details: Kerberos as a current GA authentication option for Explicit Proxy (previously SAML-only in the source material), the region-count-based VPN IP pool sizing model, and Microsoft's rename of Azure AD to Microsoft Entra ID. **[Appendix D — Known Documentation Ambiguities](prisma-access-sase/appendix/appD-known-documentation-ambiguities.md)** documents two cases where PaloAlto's own current documentation genuinely contradicts itself, and tracks twenty further claims investigated but left unconfirmed rather than guessed at.

---

## 5. How to Use It

**Sequential read:** Parts 1–3 (Chapters 01–19) form a complete concepts-to-design progression. Read in order for the full grounding before touching deployment-specific parts.

**Reference mode:** Each chapter is self-contained. Jump directly to the topic you need using the [index below](#7-manual-contents) — no need to read preceding chapters.

**Appendix as a companion:** The appendix isn't a sequence — consult the glossary or CLI/API reference while working through the main parts, or use it afterward as a standalone lookup.

---

## 6. Status

All 54 chapters and 4 appendix entries are complete and published.

---

## 7. Manual Contents

### Part 1 — Prisma Access Overview

| Chapter | Title |
|---|---|
| [Ch 01](prisma-access-sase/part1/ch01-traditional-network-challenges.md) | Traditional VPN & Branch Architecture — Problems & Drivers |
| [Ch 02](prisma-access-sase/part1/ch02-sase-concepts-and-components.md) | The Perimeter Is Now Everywhere — SASE Concepts & Key Components |
| [Ch 03](prisma-access-sase/part1/ch03-prisma-access-architecture.md) | Prisma Access Architecture — Components, Presence & Services |
| [Ch 04](prisma-access-sase/part1/ch04-prisma-access-for-networks.md) | Prisma Access for Networks |
| [Ch 05](prisma-access-sase/part1/ch05-prisma-access-for-users.md) | Prisma Access for Users & SaaS Design Benefits |

### Part 2 — Planning & Designing

| Chapter | Title |
|---|---|
| [Ch 06](prisma-access-sase/part2/ch06-licensing-and-management-models.md) | Prisma Access Licensing & Management Models (Cloud vs Panorama) |
| [Ch 07](prisma-access-sase/part2/ch07-service-infrastructure-planning.md) | Service Infrastructure & Subnet Planning |
| [Ch 08](prisma-access-sase/part2/ch08-service-connections-planning.md) | Service Connections Planning (Networks & Mobile Users) |
| [Ch 09](prisma-access-sase/part2/ch09-remote-networks-planning.md) | Remote Networks Planning (Bandwidth Model & Compute Location) |
| [Ch 10](prisma-access-sase/part2/ch10-mobile-user-deployment-planning.md) | Mobile User Deployment Planning |
| [Ch 11](prisma-access-sase/part2/ch11-resiliency-and-redundancy-design.md) | Resiliency & Redundancy Design Considerations |

### Part 3 — Routing & SD-WAN Design

| Chapter | Title |
|---|---|
| [Ch 12](prisma-access-sase/part3/ch12-traffic-flow-scenarios.md) | Traffic Flow Scenarios (RN, MU, SC Permutations) |
| [Ch 13](prisma-access-sase/part3/ch13-default-routing-and-backbone.md) | Default Routing (Cold Potato) — With & Without Backbone |
| [Ch 14](prisma-access-sase/part3/ch14-hot-potato-routing.md) | Hot Potato Routing |
| [Ch 15](prisma-access-sase/part3/ch15-sd-wan-overview-and-architecture.md) | Prisma SD-WAN Overview & Architecture |
| [Ch 16](prisma-access-sase/part3/ch16-sd-wan-controllers-and-ion-security.md) | SD-WAN Controllers, ION Device Communication & Trust Chain |
| [Ch 17](prisma-access-sase/part3/ch17-ion-devices-and-high-availability.md) | ION Device Family & High-Availability Design |
| [Ch 18](prisma-access-sase/part3/ch18-sd-wan-design-considerations.md) | SD-WAN Design Considerations (PATH, QoS, NAT) |
| [Ch 19](prisma-access-sase/part3/ch19-data-center-scaling-design.md) | Data Center Scaling Design |

### Part 4 — ZTNA Connector

| Chapter | Title |
|---|---|
| [Ch 20](prisma-access-sase/part4/ch20-ztna-connector-overview-and-components.md) | ZTNA Connector Overview — Components, Connectors, Groups & Targets |
| [Ch 21](prisma-access-sase/part4/ch21-ztna-use-cases-and-packet-flow.md) | ZTNA Use Cases & Packet Flow |
| [Ch 22](prisma-access-sase/part4/ch22-ztna-connector-vs-service-connections.md) | ZTNA Connector vs Service Connections & Supported Hosting Environments |
| [Ch 23](prisma-access-sase/part4/ch23-ztna-network-requirements-and-prerequisites.md) | Network Requirements & Prerequisites for ZTNA |
| [Ch 24](prisma-access-sase/part4/ch24-enable-and-configure-ztna-connector.md) | Enable & Configure ZTNA Connector |
| [Ch 25](prisma-access-sase/part4/ch25-onboard-ztna-connector-esxi-and-upgrade.md) | Onboard ZTNA Connector in VMware ESXi & Upgrade |

### Part 5 — Activate & Configure

| Chapter | Title |
|---|---|
| [Ch 26](prisma-access-sase/part5/ch26-activation-planning-checklist.md) | Prisma Access Activation Planning Checklist |
| [Ch 27](prisma-access-sase/part5/ch27-integrating-panorama-with-prisma-access.md) | Integrating Panorama with Prisma Access |
| [Ch 28](prisma-access-sase/part5/ch28-configuring-service-infrastructure.md) | Configuring the Service Infrastructure |
| [Ch 29](prisma-access-sase/part5/ch29-ipsec-tunnel-configuration.md) | IPSec Tunnel Configuration for Service Connections |
| [Ch 30](prisma-access-sase/part5/ch30-service-connection-static-routes.md) | Configure Service Connection — Static Routes |
| [Ch 31](prisma-access-sase/part5/ch31-service-connection-bgp.md) | Configure Service Connection — BGP |

### Part 6 — Panorama Operations for Prisma Access

| Chapter | Title |
|---|---|
| [Ch 32](prisma-access-sase/part6/ch32-templates-and-template-stacks.md) | Templates & Template Stacks |
| [Ch 33](prisma-access-sase/part6/ch33-device-groups-concepts-and-hierarchy.md) | Device Groups — Concepts, Components & Hierarchy |
| [Ch 34](prisma-access-sase/part6/ch34-device-group-policies-and-objects.md) | Device Group Policies & Objects |
| [Ch 35](prisma-access-sase/part6/ch35-inheritance-and-predefined-templates.md) | Inheritance Precedence & Predefined Templates for Prisma Access |
| [Ch 36](prisma-access-sase/part6/ch36-prisma-access-zones.md) | Prisma Access Zones |

### Part 7 — Remote Networks

| Chapter | Title |
|---|---|
| [Ch 37](prisma-access-sase/part7/ch37-remote-network-templates-and-zone-mapping.md) | Remote Network Templates, Device Groups & Zone Mapping |
| [Ch 38](prisma-access-sase/part7/ch38-remote-network-bandwidth-allocation.md) | Remote Network Bandwidth Allocation |
| [Ch 39](prisma-access-sase/part7/ch39-onboard-remote-network-static-route.md) | Onboard Remote Network — Single IPSec Tunnel & Static Route |
| [Ch 40](prisma-access-sase/part7/ch40-onboard-remote-network-bgp.md) | Onboard Remote Network — Single IPSec Tunnel & BGP |
| [Ch 41](prisma-access-sase/part7/ch41-onboard-remote-network-ecmp-bgp.md) | Onboard Remote Network — Multiple IPSec Tunnels, ECMP & BGP |

### Part 8 — Mobile Users

| Chapter | Title |
|---|---|
| [Ch 42](prisma-access-sase/part8/ch42-mobile-user-templates-and-zone-mapping.md) | Mobile User Templates, Device Groups & Zone Mapping |
| [Ch 43](prisma-access-sase/part8/ch43-ldap-and-authentication-profile.md) | LDAP Server & Authentication Profile |
| [Ch 44](prisma-access-sase/part8/ch44-onboard-mobile-users-globalprotect.md) | Onboard Mobile Users — GlobalProtect |
| [Ch 45](prisma-access-sase/part8/ch45-verify-mobile-users-globalprotect.md) | Verify Mobile Users — GlobalProtect |
| [Ch 46](prisma-access-sase/part8/ch46-globalprotect-split-tunneling.md) | GlobalProtect Split Tunneling |
| [Ch 47](prisma-access-sase/part8/ch47-globalprotect-app-settings.md) | GlobalProtect App Settings |
| [Ch 48](prisma-access-sase/part8/ch48-globalprotect-app-upgrades.md) | GlobalProtect App Upgrades — Staged Rollout |

### Part 9 — Cloud Secure Web Gateway

| Chapter | Title |
|---|---|
| [Ch 49](prisma-access-sase/part9/ch49-how-explicit-proxy-works.md) | How Explicit Proxy Works |
| [Ch 50](prisma-access-sase/part9/ch50-explicit-proxy-configuration-guidelines.md) | Explicit Proxy Configuration Guidelines |
| [Ch 51](prisma-access-sase/part9/ch51-app-support-browser-and-pac-requirements.md) | App Support, Browser Guidelines & PAC File Requirements |
| [Ch 52](prisma-access-sase/part9/ch52-pac-file-guidelines-and-sample.md) | PAC File Guidelines — Detailed & Sample |
| [Ch 53](prisma-access-sase/part9/ch53-configure-azure-ad-saml-profile.md) | Configure Azure AD SAML Profile & Authentication Profile |
| [Ch 54](prisma-access-sase/part9/ch54-configure-saml-in-azure-ad.md) | Configure SAML in Azure AD for Prisma Access Explicit Proxy |

### Appendix

| Chapter | Title |
|---|---|
| [App A](prisma-access-sase/appendix/appA-glossary.md) | Glossary of Terms |
| [App B](prisma-access-sase/appendix/appB-cli-and-api-commands.md) | Key CLI & API Commands Reference |
| [App C](prisma-access-sase/appendix/appC-architecture-quick-reference.md) | Architecture Quick-Reference |
| [App D](prisma-access-sase/appendix/appD-known-documentation-ambiguities.md) | Known Documentation Ambiguities |
