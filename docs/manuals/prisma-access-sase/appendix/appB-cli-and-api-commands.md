---
title: "Key CLI & API Commands Reference"
description: "Quick-reference for PAN-OS CLI and Prisma Access API commands most useful during deployment, troubleshooting, and verification of IPSec tunnels, BGP sessions, mobile users, and service status."
date: 2026-05-28
tags:
  - prisma-access
  - sase
  - paloalto
  - cli
  - api
  - reference
  - troubleshooting
  - panorama
  - panos
status: published
---

# Appendix B — Key CLI & API Commands Reference

All CLI commands are run on the **Panorama** management plane or on a **PAN-OS firewall** acting as a gateway. Panorama operational commands use `> request` and `> show` syntax. Some commands require dropping into a Prisma Access managed device shell — use with care in production.

> **This is the Panorama-managed troubleshooting path, not a universal one.** Everything below the "IPSec Tunnel Status" through "Panorama Push and Commit Status" sections is Panorama's operational CLI (`show`/`clear`/`debug`). **Strata Cloud Manager (SCM) has no direct equivalent interactive CLI shell for this kind of live troubleshooting.** For SCM-managed deployments, the real equivalents are the **Insights dashboards** (already established in ch30/ch45 — IPSec/BGP/tunnel status, GlobalProtect authentication success/failure counts, and similar health views, not repeated here) and the dedicated **Configuration > Operation > Push Status** page for commit/push job tracking. If you're troubleshooting an SCM-managed tenant, start there rather than expecting a CLI-equivalent command for each row below.

---

## IPSec Tunnel Status

### Show all IPSec tunnels

```
> show vpn tunnel
```

Displays all configured IPSec SA tunnels, their state (active/inactive), and traffic counters.

### Show IKE SA (Phase 1)

```
> show vpn ike-sa
```

Shows IKE Phase 1 security associations — gateway name, peer IP, IKE version, state, and encryption/DH parameters negotiated.

### Show IPSec SA (Phase 2)

```
> show vpn ipsec-sa
```

Shows Phase 2 SAs — proxy IDs, encryption, authentication, PFS group, and SA lifetime remaining.

### Show IKE gateway detail

```
> show vpn ike-sa gateway <gateway-name>
```

Detailed output for a single IKE gateway — useful when a specific tunnel fails to come up.

### Clear and renegotiate IKE SA

```
> clear vpn ike-sa gateway <gateway-name>
```

Clears the Phase 1 SA and triggers renegotiation. Use when the tunnel is stuck in a failed state.

### Clear IPSec SA

```
> clear vpn ipsec-sa tunnel <tunnel-name>
```

Clears Phase 2 SA for a specific tunnel. The tunnel will re-key on next traffic.

---

## BGP Status

### Show BGP peer summary

```
> show routing protocol bgp peer
```

Lists all BGP peers with their state (Established / Active / Connect / Idle), AS number, prefixes received, and session uptime.

### Show BGP RIB (received prefixes)

```
> show routing protocol bgp rib-in peer <peer-name>
```

Shows all prefixes received from a specific BGP peer before policy filtering.

### Show BGP loc-rib (installed routes)

```
> show routing protocol bgp loc-rib
```

Shows routes in the local BGP RIB — prefixes that passed policy and are candidates for the routing table.

### Show routing table

```
> show routing route
```

Displays the active routing table — all installed routes with next-hops and interfaces.

### Clear BGP peer (soft reset)

```
> clear routing protocol bgp peer <peer-name> soft-reset-out
```

Triggers outbound route refresh without dropping the BGP session. Use after changing route policy.

---

## Remote Network Status

### Show Remote Network tunnel state

From Panorama UI: `Cloud Services > Status > Remote Networks`

CLI equivalent (run on the RN-SPN managed device):

```
> show vpn tunnel | match <remote-network-name>
```

### Show Service Endpoint Addresses

```
> show cloud-services status
```

Displays Prisma Access service status including allocated Service Endpoint Addresses (RN-SPN public IPs).

---

## Mobile User Status

### Show current GlobalProtect users

From Panorama UI: `Cloud Services > Status > Status > Current Users`

CLI on MU-SPN managed device:

```
> show global-protect-gateway current-user
```

Shows all currently authenticated GlobalProtect users — username, IP, gateway, connect time.

### Show GP gateway statistics

```
> show global-protect-gateway statistics
```

Summary of connected users, tunnels, and throughput per gateway.

### Show GP gateway IKE SA

```
> show global-protect-gateway ike-sa
```

IPSec/IKE SA state for all GlobalProtect user tunnels.

---

## Panorama Push and Commit Status

### Show commit status

```
> show jobs all
```

Lists all Panorama jobs (commits, pushes, content updates) with their status and  completion percentage.

### Show last push result

```
> show jobs id <job-id>
```

Detailed status for a specific job ID. Use to diagnose failed commits or pushes.

### Show device and template push state

```
> show template-stack name <stack-name>
```

Shows which devices are associated with a template stack and their last push state.

---

## Log and Monitoring Commands

### Show traffic logs (last 20 entries)

```
> show log traffic last-20
```

### Show system logs

```
> show log system
```

### Show IKE negotiation failures

```
> debug ike global on debug
```

Enables verbose IKE debug logging. Review with `> show log ike`. **Disable after troubleshooting:**

```
> debug ike global off
```

### Show SSL decrypt logs

```
> show log decryption
```

Shows decryption policy matches and SSL/TLS session details.

---

## Prisma Access API — Common Operations

The Panorama XML API is available at `https://<panorama-ip>/api/`. Authentication uses an API key retrieved with:

```
GET https://<panorama-ip>/api/?type=keygen&user=<user>&password=<pwd>
```

### Retrieve running config

```
GET https://<panorama-ip>/api/?type=config&action=show&key=<apikey>
```

### Trigger a commit

```
POST https://<panorama-ip>/api/?type=commit&cmd=<commit-xml>&key=<apikey>
```

Where `<commit-xml>` is:

```xml
<commit><description>API commit</description></commit>
```

### Check job status

```
GET https://<panorama-ip>/api/?type=op&cmd=<show-jobs-xml>&key=<apikey>
```

Where `<show-jobs-xml>` is:

```xml
<show><jobs><id>JOB_ID</id></jobs></show>
```

### Prisma Access Configuration API (SCM) — a genuinely different model

The Prisma Access cloud management API (separate from the Panorama XML API above) is used for tenants managed by Strata Cloud Manager. It does **not** work like the Panorama API's single-step commit — it follows a three-step **candidate → push → poll** model: (1) issue one or more configuration changes, which are held as a **candidate configuration**; (2) **push** the candidate, which creates a **configuration job**; (3) poll that job until it completes, at which point the candidate becomes the running configuration. Verified directly against pan.dev — every path below is quoted as shown on its own endpoint page, not inferred from a summary.

**Authentication (OAuth2 client credentials — confirmed via pan.dev, a different flow and a different hostname than the config APIs themselves):**

```
curl -d "grant_type=client_credentials&scope=tsg_id:<tsg_id>" \
  -u <client_id>:<client_secret> \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -X POST https://auth.apps.paloaltonetworks.com/oauth2/access_token
```

Client ID/Secret come from a service account, authenticated here via HTTP Basic auth (not sent in the body). The response contains an `access_token` used as a Bearer token on subsequent requests. Tokens are short-lived — confirmed **15-minute** lifetime.

**1. Retrieve remote network configuration (confirmed exact path):**

```
GET https://api.sase.paloaltonetworks.com/sse/config/v1/remote-networks
Authorization: Bearer <access-token>
```

Returns the list of remote network configurations. Creating or modifying a remote network follows the same base path with POST/PUT — the exact request body schema wasn't confirmed in this pass, so it isn't reproduced here; consult pan.dev directly before scripting a write operation.

**2. Push the candidate configuration (confirmed exact path):**

```
POST https://api.sase.paloaltonetworks.com/sse/config/v1/config-versions/candidate:push
Authorization: Bearer <access-token>
```

Creates a configuration job from whatever candidate changes are pending. The request body includes a `folders` field (confirmed — this field was renamed from `devices` in a past API revision, per pan.dev's release notes). A `201 Created` response confirms the push was accepted; the exact response body schema (e.g. whether a job ID field is returned inline) wasn't confirmed in this pass — described here in prose rather than guessed.

**3. Poll the job for completion (confirmed exact path):**

```
GET https://api.sase.paloaltonetworks.com/sse/config/v1/jobs/<job-id>
Authorization: Bearer <access-token>
```

Retrieves the status of a specific configuration job by ID. The exact status field name and its possible values (e.g. pending/in-progress/complete/failed) weren't confirmed in this pass — poll this endpoint and inspect the actual response in your tenant rather than assuming a specific field name.

> Note on URL versioning: Palo Alto migrated these APIs from a `/config/v1/...` path to the current `/sse/config/v1/...` path (confirmed via pan.dev release notes) — older examples you may find elsewhere using the bare `/config/v1/` prefix are for the prior version.

---

## Troubleshooting Quick Reference

| Symptom | First Command | What to Look For |
|---|---|---|
| IPSec tunnel not up | `show vpn ike-sa` | Phase 1 state — `init` or missing entry = IKE failure |
| Tunnel up but no traffic | `show vpn ipsec-sa` | Check proxy IDs match; check SA lifetime |
| BGP not Established | `show routing protocol bgp peer` | State stuck in Active/Connect — check peer IP reachability |
| Routes not in table | `show routing protocol bgp loc-rib` | Route filtered by policy or not advertised by peer |
| GP users not connecting | `show global-protect-gateway current-user` | Empty = auth failure or portal unreachable |
| Commit/push failing | `show jobs all` | Look for FAIL status and error details in job output |
| Auth loop (SAML or Kerberos) | PAC file check | Verify IdP, Cloud Identity Engine (CIE), and `*.acs.prismaaccess.com` are bypassed — the CIE bypass is required and easy to miss (see ch52); Kerberos deployments have different bypass considerations entirely (see ch49/ch51) |

---

*Previous: [Appendix A — Glossary of Terms](./appA-glossary.md)* · *Next: [Appendix C — Architecture Quick-Reference](./appC-architecture-quick-reference.md)*
