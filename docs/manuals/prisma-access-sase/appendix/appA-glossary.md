---
title: "Glossary of Terms"
description: "Definitions for all key terms, acronyms, and named components used throughout the Prisma Access training material."
date: 2026-05-28
tags:
  - prisma-access
  - sase
  - paloalto
  - glossary
  - reference
status: published
---

# Appendix A — Glossary of Terms

---

## A

**ACS (Assertion Consumer Service)**
The endpoint on the service provider (Prisma Access) that receives and processes SAML assertions from the identity provider. For Explicit Proxy, the ACS URL is `https://global.acs.prismaaccess.com/saml/acs`. Must be bypassed in the PAC file.

**Auth Override Certificate**
A certificate used by the GlobalProtect portal to issue an encrypted session cookie after initial user authentication. Subsequent GP reconnections use this cookie for transparent re-authentication, avoiding repeated credential prompts.

**Autonomous System (AS) Number**
A globally unique identifier used in BGP routing. Prisma Access Service Connections use AS **65534**. Branch CPEs and on-premises routers use RFC 6996 private AS numbers (64512–65534).

---

## B

**Bandwidth Allocation**
The provisioned throughput at a Prisma Access compute location for Remote Network traffic. Two models: aggregate (pooled per location) and site-based (guaranteed CIR per branch, PA 6.0+). Must be configured before onboarding branch sites. Current Palo Alto documentation is genuinely contradictory about which model is the default for new deployments — see ch38 (and the planned Appendix D once available).

**BGP (Border Gateway Protocol)**
The inter-domain routing protocol used between Prisma Access and branch CPEs or data center routers. Used for dynamic route exchange in Remote Networks and Service Connections. ECMP requires BGP.

**BFD (Bidirectional Forwarding Detection)**
A lightweight protocol for detecting link failures. Used with SD-WAN path monitoring in Prisma Access to trigger fast failover between WAN links.

---

## C

**CIR (Committed Information Rate)**
The guaranteed bandwidth rate per branch site in the site-based bandwidth model (PA 6.0+). Prevents high-traffic branches from consuming bandwidth at the expense of others in the same compute location.

**CloudBlades**
The third core component of Prisma SD-WAN, alongside the cloud controller and ION devices. An API-based abstraction layer connecting the SD-WAN controller to third-party services (e.g. AWS Transit Gateway, Azure Virtual WAN, Zscaler, Prisma Access) without requiring controller or ION software updates when the third-party service changes. Managed from the same Strata Cloud Manager interface as the controller and ION devices. See ch15.

**Cloud Identity Engine (CIE)**
A cloud-based identity broker that Palo Alto increasingly recommends as an alternative to direct configuration for user/group mapping and SAML authentication. For Device Groups, CIE plus Identity Redistribution is the Strata Cloud Manager-native replacement for Panorama's Master Device mechanism (ch33). For Explicit Proxy's Azure AD SAML integration, Palo Alto recommends CIE over the direct metadata-import approach, though the direct approach remains valid (ch53). Direct LDAP server profile configuration for GlobalProtect mobile users is unaffected — CIE has not superseded it (ch43).

**Cloud SWG (Secure Web Gateway)**
See *Explicit Proxy*.

**Colo-Connect**
A high-bandwidth Service Connection alternative using GCP Dedicated or Partner cloud interconnects instead of an IPSec tunnel over the public internet — up to 100 Gbps per compute region (Prisma Access 6.1+), with up to 8 Colo-Connects per region for redundancy (6.2+). Coexists with, rather than replaces, standard IPSec-based Service Connections. See ch08.

**Compute Location**
A geographic region where Prisma Access infrastructure (MU-SPN, RN-SPN, SC-CAN) is deployed. Examples: `US-East`, `EU-West`, `APAC-Southeast`. Users and branches connect to the nearest compute location.

**Cortex Data Lake**
Former name for Strata Logging Service. See *Strata Logging Service*.

---

## D

**Device Group**
A Panorama object hierarchy level that holds security policy rules and shared objects. Prisma Access uses predefined device groups (`Mobile_User_Device_Group`, `Remote_Network_Device_Group`, `Service_Conn_Device_Group`, `Explicit_Proxy_Device_Group`). Follows a 4-level hierarchy; policy rules inherit downward.

**DH Group (Diffie-Hellman Group)**
The mathematical group used for key exchange in IKE Phase 1 (and optionally Phase 2 with PFS). Higher DH group numbers provide stronger security. Common values: Group 14 (2048-bit), Group 19/20 (ECC), Group 21 (521-bit ECC).

---

## E

**ECMP (Equal-Cost Multipath)**
A load-balancing method that distributes traffic across multiple equal-cost paths simultaneously. For Remote Networks, ECMP uses up to 4 parallel IPSec tunnels to a single RN-SPN, requiring BGP. Provides both increased bandwidth (up to 8 Gbps) and built-in resilience. QoS and static routes are **not supported** once ECMP is enabled, and the tenant scale limits are conditional on Local BGP IP Address usage — see ch41.

**Explicit Proxy**
An agentless Prisma Access service that secures HTTP/HTTPS browser traffic without a VPN client. Endpoints are directed to the proxy via PAC file. Supports both SAML (port 8080) and Kerberos (port 8081) authentication — corrected 2026-07-10, was previously (and incorrectly) stated as "port 8080 only; SAML only," see ch49/ch51. Also referred to as Cloud SWG.

**Explicit_Proxy_Device_Group**
The predefined Panorama device group for Explicit Proxy security policy rules and objects.

**Explicit_Proxy_Template / Explicit_Proxy_Template_Stack**
The predefined Panorama template and template stack for Explicit Proxy configuration, including the SAML authentication profile.

---

## F

**FEC (Forward Error Correction)**
A Prisma SD-WAN Performance Policy feature that repairs a degraded link in place — using Link Quality Monitoring (loss/latency) to add redundant data so lost packets can be reconstructed without retransmission — as an alternative to switching paths. Runs in Always ON (continuous, for designated mission-critical apps) or Adaptive (triggered by an SLA packet-loss threshold) mode. Applies only to Prisma SD-WAN VPN paths, not direct-internet or MPLS paths. Requires ION software 6.3.2+. See ch18.

**Forwarding Profiles**
An add-on capability for Explicit Proxy that allows more than one PAC file per tenant (the default is one PAC file per tenant) or PAC-free forwarding. Complements, rather than replaces, standard PAC file configuration. See ch52.

---

## G

**GlobalProtect**
The Palo Alto Networks VPN/ZTNA client used for mobile user connectivity. Consists of a portal (entry point) and gateways (enforcement points). Supports user-logon, pre-logon, on-demand, and hybrid connect methods.

**GlobalProtect Gateway**
The Prisma Access enforcement node where mobile user traffic is inspected and policy is applied. Each compute location has one or more gateways.

**GlobalProtect Portal**
The entry point for GlobalProtect clients. Distributes gateway lists, agent configurations, and client settings to the GP app. The portal FQDN is the address configured in the GP client.

---

## H

**HIP (Host Information Profile)**
A GlobalProtect feature that collects endpoint posture data (OS version, patch level, disk encryption state, antivirus status) and makes it available for security policy enforcement. Enables ZTNA-style conditional access based on device compliance.

**Hot Potato Routing**
A Service Connection routing model where Prisma Access forwards traffic to the DC as quickly as possible — traffic exits the Prisma Access backbone near the source. Minimises latency within the Prisma Access cloud but relies on DC routing for final delivery.

**Cold Potato Routing**
A Service Connection routing model where Prisma Access retains traffic on its own backbone for as long as possible before handing off to the DC. Reduces reliance on public internet transit.

---

## I

**IKE (Internet Key Exchange)**
The protocol that negotiates and manages IPSec security associations. IKEv2 is current standard. Operates in two phases: Phase 1 (IKE SA — authentication and key exchange) and Phase 2 (IPSec SA — data encryption parameters).

**Infrastructure Subnet**
A dedicated IP range (`/24` or `/23`) reserved for internal Prisma Access use — inter-node communication, tunnel endpoints, and management traffic. Must not overlap with corporate subnets, `169.254.0.0/16`, or `100.64.0.0/10`. Configured during Service Infrastructure setup (ch28); see ch07 for detailed deployment-scale sizing tiers (small/medium/large).

**ION Device**
The Prisma SD-WAN forwarding device deployed at branch and data centre sites — the SD-WAN counterpart to Prisma Access's own SPN/CAN nodes. Operates in Analytics or Control mode, provisions via zero-touch (calling home to the controller), and continues forwarding autonomously for up to 72 hours if controller connectivity is lost. See ch15–ch19.

**IPSec (Internet Protocol Security)**
The protocol suite used to encrypt and authenticate IP packets between a branch CPE and an RN-SPN (Remote Network), or between a DC router and an SC-CAN (Service Connection).

---

## K

**Kerberos**
A ticket-based network authentication protocol. A current, GA authentication option for Prisma Access Explicit Proxy alongside SAML, using port 8081 (SAML uses 8080) — added 2026-07-10 while correcting this glossary's prior "SAML only" claim, see ch49/ch51. Requires a Kerberos Keytab and Kerberos Realm; usernames must be in `userPrincipalName` (UPN) format.

---

## L

**Logging Service**
See *Strata Logging Service*.

---

## M

**MRAI Timer (Minimum Route Advertisement Interval)**
A BGP timer controlling how frequently route update messages are sent. Default: 30 seconds. Reducing this value allows faster route convergence but increases BGP overhead.

**MED (Multi-Exit Discriminator)**
A BGP attribute used to signal preferred entry points to an AS. Prisma Access does **not** honour MED attributes received from CPEs — load balancing decisions are not influenced by CPE MED values. Prisma Access does, however, send its own outbound MED (0 for primary, 500 for backup Service Connections) to influence which path the CPE prefers for return traffic — see ch31.

**Mobile_User_Device_Group**
The predefined Panorama device group for mobile user (GlobalProtect and Explicit Proxy) security policy rules.

**Mobile_User_Template / Mobile_User_Template_Stack**
The predefined Panorama template and template stack for mobile user (GlobalProtect) network configuration, zones, and authentication profiles.

**MRAI Timer**
See above.

**MU-SPN (Mobile User — Service Processing Node)**
The Prisma Access cloud node that terminates GlobalProtect VPN tunnels from mobile user endpoints. Also the egress point for Explicit Proxy traffic. Located at each compute location.

---

## P

**PAC File (Proxy Auto-Config)**
A JavaScript file (`FindProxyForURL`) that tells a browser which proxy server to use for a given URL. For Explicit Proxy, the PAC file routes HTTP/HTTPS traffic to `*.proxy.prismaaccess.com:8080` and must bypass IdP, Cloud Identity Engine (CIE), and ACS URLs — added the CIE bypass 2026-07-10, previously missing here, see ch52.

**Panorama**
Palo Alto Networks' centralised management platform. Manages Prisma Access configuration via templates, template stacks, and device groups. The primary management path for Prisma Access (alongside SCM).

**PFS (Perfect Forward Secrecy)**
An IPSec feature that generates new Diffie-Hellman key material for each Phase 2 SA, independent of the Phase 1 key. Prevents compromise of one session from exposing past or future sessions. DH group for PFS must match on both peers.

**Pre-logon**
A GlobalProtect connect method where the GP client establishes a VPN tunnel using a machine certificate before the user logs in. Enables domain logon scripts, GPO, and certificate-based authentication.

**Prisma Access**
Palo Alto Networks' cloud-delivered SASE platform. Provides network security (NGFW, SWG, CASB, DLP) and network connectivity (SD-WAN, ZTNA) as a cloud service. Managed via Panorama or Strata Cloud Manager.

**Prisma Access Agent**
GlobalProtect's next-generation successor agent, sharing the same core tunnel-and-inspection architecture but adding local endpoint DLP, Dynamic Privilege Access (per-project network scoping), anti-tampering protection, and automatic IPSec-to-SSL fallback. GlobalProtect remains fully supported; the two agents can coexist on the same tenant/endpoint during a transition period, though only one can be actively connected at a time. See ch05.

---

## R

**Remote Network**
A branch site connected to Prisma Access via one or more IPSec tunnels to an RN-SPN. Branch subnets are reachable via static routes or BGP. See ch37–ch41.

**Remote_Network_Device_Group**
The predefined Panorama device group for Remote Network security policy rules.

**Remote_Network_Template / Remote_Network_Template_Stack**
The predefined Panorama template and template stack for Remote Network IPSec tunnel and zone configuration.

**RN-SPN (Remote Network — Service Processing Node)**
The Prisma Access cloud node that terminates IPSec tunnels from branch CPEs. Also called the IPSec Termination Node. Each compute location may have multiple RN-SPN nodes depending on bandwidth allocation.

---

## S

**SAML (Security Assertion Markup Language)**
An XML-based open standard for federated authentication. One of two supported authentication protocols for Prisma Access Explicit Proxy, alongside Kerberos — corrected 2026-07-10, was previously (and incorrectly) stated as the only option, see ch49/ch51. Used between the browser, the SAML IdP (e.g. Azure AD, Okta), and the Prisma Access ACS.

**SC-CAN (Service Connection — Cloud Access Node)**
The Prisma Access cloud node that terminates IPSec tunnels from data centre or HQ routers (Service Connections). Routes traffic between Prisma Access and the corporate network.

**SCM (Strata Cloud Manager)**
Palo Alto Networks' next-generation SaaS-based management platform. The preferred management interface for new Prisma Access deployments. Replaces Panorama for cloud-managed tenants.

**SD-WAN Controller**
The cloud-hosted management and control plane for Prisma SD-WAN, accessed via Strata Cloud Manager. Centralises routing configuration, policy, and CloudBlades management across all ION devices; communicates with ION devices over a TLS-encrypted control channel carrying only configuration, policy, and anonymised telemetry — never customer traffic content, which flows directly between ION devices instead. See ch15–ch16.

**Secure Fabric Links**
The native mechanism for direct ION-to-ION data-plane traffic in Prisma SD-WAN — distinct from IPsec/GRE, which is reserved specifically for connecting to non-Prisma-SD-WAN third-party endpoints ("Standard VPN"). The SD-WAN Controller manages configuration and policy over a separate TLS control channel and is never in the traffic forwarding path. See ch16.

**Service Connection**
An IPSec tunnel between a corporate data centre or HQ and a Prisma Access SC-CAN. Provides mobile users and remote branches with access to on-premises resources. See ch29–ch31.

**Service_Conn_Device_Group**
The predefined Panorama device group for Service Connection security policy rules. Parent device group: `Shared`.

**Service_Conn_Template / Service_Conn_Template_Stack**
The predefined Panorama template and template stack for Service Connection IPSec tunnel configuration.

**Service Endpoint Address**
The public IP or FQDN of an RN-SPN node. This is the VPN peer address configured on the branch CPE. Available after bandwidth allocation and Commit & Push.

**Split Tunneling**
A GlobalProtect feature that routes only specified traffic through the VPN tunnel, allowing other traffic to go directly to the internet. Configured per gateway agent config using access route (IP/subnet) or domain/application rules.

**Strata Cloud Manager**
See *SCM*.

**Strata Logging Service**
Palo Alto Networks' cloud-based log storage and analytics service. Formerly known as Cortex Data Lake. Required for Prisma Access log forwarding and Panorama log viewing.

**Symmetric Return**
An ECMP option that ensures return traffic for a session exits through the same IPSec tunnel that received the ingress traffic. Required for stateful firewalls at branch CPEs.

---

## T

**Template**
A Panorama object that holds Network tab and Device tab configuration (interfaces, zones, routing, IKE/IPSec profiles). Applied to managed devices via template stacks. Prisma Access uses predefined templates for each service type.

**Template Stack**
An ordered set of up to 8 Panorama templates. The stack defines which templates are pushed together to a managed device or Prisma Access service. Higher-order templates override lower-order templates for the same setting.

---

## V

**VPN IP Pool**
The IP address range from which Prisma Access assigns addresses to GlobalProtect clients. Minimum size is driven by regional deployment scope, not a flat per-location figure — /23 minimum for 1–2 regions, /19 minimum for 3+ regions — corrected 2026-07-10, see ch07/ch10. Must not overlap with any other subnet in the environment.

---

## Z

**Zone Mapping**
The process of assigning custom security zones to the Prisma Access Trust or Untrust classification. Required after zone creation — all zones default to Untrust until explicitly mapped.

**ZTNA (Zero Trust Network Access)**
A security model that grants users access to specific applications based on identity and device posture, rather than broad network access. Prisma Access implements ZTNA via GlobalProtect (with HIP checks) and the ZTNA Connector (ch20–ch25).

**ZTNA Connector**
A lightweight VM deployed in the customer's private network that establishes outbound-only tunnels to Prisma Access. Provides agentless or agent-based access to private applications without opening inbound firewall ports. See ch20–ch25.

---

*Previous: [Chapter 54 — Configure SAML in Azure AD](../part9/ch54-configure-saml-in-azure-ad.md)* · *Next: [Appendix B — Key CLI & API Commands Reference](./appB-cli-and-api-commands.md)*
