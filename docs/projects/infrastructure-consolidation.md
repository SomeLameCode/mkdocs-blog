---
title: "Infrastructure Consolidation — Six Servers to Three"
description: "Collapsed a six-server Windows homelab into three through DC virtualization, SCCM stack consolidation, and structured server decommission — completed in ten days."
date: 2026-05-16
tags:
  - windows-server
  - active-directory
  - sccm
  - proxmox
  - homelab
  - infrastructure
status: published
---

# Infrastructure Consolidation — Six Servers to Three

!!! info "At a Glance"
    Windows Server 2022 · Active Directory · SCCM/MECM CB 2207 · SQL Server · Proxmox · AD CS · WSUS

    6 servers → 3 · 10 days · 57 WBS tasks · 17 decisions logged · 3 operator runbooks produced

## 1. What Is It?

The domain.dom Windows Server environment had accumulated three structural problems that were worth fixing in one go.

**Physical DC as single point of hardware failure.** DC01 was the primary domain controller — it held all five FSMO roles and ran the enterprise CA. It was also a physical machine, which meant a hardware failure would take down Active Directory entirely. DC02 existed as a secondary DC but couldn't absorb a primary failure gracefully without manual intervention.

**SCCM split across three servers for no good reason.** The configuration management stack had grown by accumulation: SCCM01 ran the site server, SQL01 ran the database, W2016-1 ran the WSUS/SUP role. Three machines for one function. Three machines to patch, back up, and keep running.

**Two servers with poorly documented roles.** W2016-1 and W2016-2 were running things — but Phase 1 discovery was needed to establish exactly what. W2016-1 turned out to host WSUS and HTTP CRL distribution (the CRL part wasn't in anyone's head). W2016-2 hosted an AD CS Online Responder (OCSP) that no service actually depended on.

The goal: eliminate the physical DC by virtualizing it on Proxmox, collapse the SCCM stack onto a single physical host, and decommission both W2016 servers. Ten days, working alone, zero planned downtime for domain services.

---

## 2. Before and After

**Before — six servers:**

| Server | Role | Type |
|---|---|---|
| DC01.domain.dom | Primary DC, all FSMOs, AD CS CA, DNS | Physical |
| DC02.domain.dom | Secondary DC, DNS | Proxmox VM |
| SQL01.domain.dom | SCCM SQL database | Physical |
| SCCM01.domain.dom | SCCM site server, MP, DP, SSRS | Physical |
| W2016-1.domain.dom | WSUS/SUP | Proxmox VM |
| W2016-2.domain.dom | AD CS Online Responder (OCSP) | Proxmox VM |

**After — three servers:**

| Server | Role | Type | Notes |
|---|---|---|---|
| DC03.domain.dom | Primary DC, all FSMOs, AD CS CA, DNS | Proxmox VM | 2 vCPU, 4 GB RAM, Windows Server 2022 |
| DC02.domain.dom | Secondary DC, DNS | Proxmox VM | Unchanged throughout |
| SQL01.domain.dom | SCCM site server, SQL, SUP, SSRS | Physical | — |

DC01 lives on as a DNS A record pointing at DC03's IP — a deliberate decision explained below. W2016-1, W2016-2, SCCM01, and physical DC01 are powered off and removed from AD.

---

## 3. Phase Ordering

Three independent workstreams; the execution order was deliberate.

**SCCM consolidation first.** Moving the site server from SCCM01 to SQL01 was the most complex operation — it touched the database, IIS, WSUS, reporting services, and all client-facing roles. Getting it done first, while the environment was at its most stable, was the right call. Any problems here were easier to isolate before DC changes were in flight.

**W2016 decommission second.** Once the SUP role moved off W2016-1 to SQL01, the decommission was straightforward — power off, remove AD accounts. The catch: Phase 1 had missed that W2016-1 was also hosting HTTP CRL distribution. That surfaced during decommission planning (DNS resolution, not inventory) and was relocated before anything was powered off.

**DC rebuild last.** DC02 provided continuous domain services throughout the project, so the DC rebuild carried no availability risk. Doing it last meant the environment was already reduced and stable — fewer variables during the most sensitive operation.

---

## 4. Key Decisions

### Fresh DC build, not P2V

The obvious move when virtualizing a physical machine is P2V — clone it, boot the image on the hypervisor. The problem with P2V on a domain controller is hardware abstraction: cloned DCs frequently have driver issues on new hardware and can surface subtle replication problems. Building DC03 from scratch on Windows Server 2022 was cleaner. DC02 held the domain together throughout, so there was no urgency to rush the cutover.

After DC03 was promoted and replication verified clean, all five FSMO roles transferred in a single command:

```powershell
Move-ADDirectoryServerOperationMasterRole `
    -Identity "DC03" `
    -OperationMasterRole SchemaMaster,DomainNamingMaster,PDCEmulator,RIDMaster,InfrastructureMaster `
    -Confirm:$false

netdom query fsmo
# Schema owner:               DC03.domain.dom
# Domain role owner:          DC03.domain.dom
# PDC role:                   DC03.domain.dom
# RID pool manager:           DC03.domain.dom
# Infrastructure owner:       DC03.domain.dom
```

### SCCM stays on physical hardware

The original design had assumed Proxmox could absorb both the new DC VM and the SCCM stack. A resource check early in the project showed 16.87 GB free RAM and ~1.07 TB storage on the Proxmox host — enough for the DC VM, not enough for DC VM + SQL Server + SCCM + WSUS running concurrently. Scope reduction was the right call. The DC is the value here; keeping SCCM physical costs nothing.

### SQL01 as the consolidation target, not SCCM01

Two candidates for hosting the consolidated SCCM stack. SQL01 has 16 GB RAM; SCCM01 has 8 GB. With SQL Server capped at 10 GB, plus SCCM site server, WSUS, and OS overhead, 16 GB is workable and 8 GB is dangerous. The second reason was more convenient: SQL Server stays in place. No database migration, no SQL Server re-registration, no SPN updates for the SQL instance. SCCM01's roles move to SQL01 rather than the database moving to SCCM01.

### Site Recovery instead of passive site server promotion

The standard upgrade path for moving a ConfigMgr site server is passive site server promotion: install a passive server, verify synchronization, promote. The wizard has one hard requirement — the target must have zero site system roles assigned. SQL01 holds the Site Database Server role, which is auto-assigned during site installation and cannot be removed. Passive promotion is structurally impossible.

The alternative is ConfigMgr Setup's Site Recovery mode. Recovery installs the site server alongside existing roles, using the existing database. No role restriction, no pre-cleanup required. Once the approach was understood, the path forward was clear.

One prerequisite: confirming the SQL instance name before running the wizard, because the answer determines whether you enter `SQL01` or `SQL01\instancename` on the database screen:

```powershell
(Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Microsoft SQL Server\Instance Names\SQL").PSObject.Properties |
    Select-Object Name, Value
# Name = MSSQLSERVER → default instance → enter "SQL01" in the wizard
# Name = <other>     → named instance   → enter "SQL01\<name>"
```

Wizard selections: **Recover a site** → **Reinstall this site server** → **Use an existing database** → point at CM_P01. The "Use existing database" option is the critical distinction — it connects to the live database without touching it. Choosing "Recover using a backup" here would have been wrong.

### DNS alias instead of DC rename

The new domain controller would be DC03, but most of the existing infrastructure referenced DC01 by name — DNS records, NTP configuration, hardcoded references accumulated over years. The clean solution would be to rename DC03 to DC01 after cutover.

The research into DC rename via `netdom computername` documented enough failure modes to rule it out:

- Login fails after reboot due to LSA/SPN mismatch
- AD replication errors (-2146893022)
- DFSR SYSVOL member object silently not renamed (KB 2001271)
- Old SPNs persist after cleanup commands
- Microsoft's own guidance (2024): demote, rename, re-promote — "supported but not recommended"

The alternative: assign DC03 the same IP as old DC01 (192.168.1.201) and create a DNS A record `DC01.domain.dom → 192.168.1.201`. The IP reuse handles IP-based references. The A record handles name-based references for HTTP, LDAP, and NTLM. Kerberos requires SPNs, not just DNS — a `HOST/DC01` SPN was added to DC03's account. It didn't survive the first restart (Netlogon replaces the full `servicePrincipalName` attribute on each startup), but no Kerberos dependency on the DC01 name was found, so the omission was acceptable. DC03 stays DC03 permanently.

Implementation — three steps, fifteen minutes:

```powershell
# 1. Assign DC03 the freed IP (once physical DC01 is off)
$nic = Get-NetAdapter | Where-Object Status -eq "Up"
New-NetIPAddress -InterfaceIndex $nic.ifIndex `
    -IPAddress "192.168.1.201" -PrefixLength 24 -DefaultGateway "192.168.1.1"
ipconfig /registerdns
Restart-Service Netlogon

# 2. A record — deliberately not a CNAME; CNAMEs break Kerberos and SMB loopback on DCs
Add-DnsServerResourceRecordA -ZoneName "domain.dom" `
    -Name "DC01" -IPv4Address "192.168.1.201" -TimeToLive 01:00:00

# 3. SPN transfer — remove from DC01 account first; duplicates cause Kerberos failures
setspn -D HOST/DC01 DC01
setspn -D HOST/DC01.domain.dom DC01
setspn -A HOST/DC01 DC03
setspn -A HOST/DC01.domain.dom DC03
setspn -X   # confirm no duplicates
```

### Fresh CA over backup/restore migration

Migrating the enterprise CA from physical DC01 to DC03 via `certutil -backup` / `certutil -restore` is documented as the supported path. The backup completed cleanly. The restore completed cleanly. Certsvc would not start on DC03.

The error chain: NTE_BAD_KEY → ERROR_FILE_NOT_FOUND. The CACertHash registry value, the key container name in the Cryptographic Service Provider, and the certificate store state were inconsistent — the restore had overwritten the key material installed during DC03's CA setup without fully replacing the associated metadata. After roughly two hours of troubleshooting with no clear resolution path, the decision was to stop and install a fresh CA.

Fresh install took twenty minutes. The new CA is named `homelab-ca` rather than the old `homelab-DC01-CA`, which decouples the CA name from the hostname it runs on. In a homelab context, replacing the CA means re-enrolling SCCM clients — which they do automatically — and redistributing the new root certificate. No long-term trust dependencies were at risk.

---

## 5. What Broke

### IIS port 80 conflict

SCCM's management point requires port 80 on the Default Web Site. After installing IIS on SQL01, the Default Web Site would not start. Port 80 was already registered in HTTP.SYS. Stopping W3SVC and WAS left port 80 still listening under PID 4 (the HTTP.SYS kernel driver), which ruled out IIS itself as the owner.

`netsh http show urlacl` shows the reservation but only identifies the account (`NT AUTHORITY\LOCAL SERVICE`), not which service. The request queue name — the actual identifier — requires `show servicestate verbose`:

```powershell
# Step 1 — shows the reservation, but not which service holds it
netsh http show urlacl url=http://*:80/
# Reserved URL : http://*:80/
#   User: NT AUTHORITY\LOCAL SERVICE

# Step 2 — find the request queue name registered against port 80
$ss = netsh http show servicestate verbose=yes
$i  = [array]::FindIndex($ss, [Predicate[string]]{param($s) $s -match "HTTP://\*:80/"})
$ss[[Math]::Max(0,$i-20)..([Math]::Min($ss.Count-1,$i+5))]
# Request queue name: SyncSharePool{A1A3CEFD-57F7-40C4-835E-536D984911AA}
# → SyncShareSvc (Windows Work Folders)
```

Fix: disable SyncShareSvc, move SSRS from port 80 to 8080 via Reporting Services Configuration Manager. Management point started cleanly on the next installation retry.

### Passive site server wizard blocked

Documented above under Key Decisions. The wizard error message says "the computer already has site system roles assigned" without explaining that Site Database Server cannot be removed or that Site Recovery is the alternative. Finding the correct path required reading the ConfigMgr documentation on site server high availability and understanding what Site Recovery actually does vs. what passive promotion does.

### Management point HTTP 500 after successful install

`mp.msi` completed with exit code 0. `mpcontrol.log` immediately began reporting HTTP 500 on every five-minute health check. The error message — "management point encountered an error when connecting to SQL Server" — was a red herring.

Direct test confirmed the real error code:

```powershell
Invoke-WebRequest "http://localhost/SMS_MP/" -UseBasicParsing
# 500.19  /  0x8007007e  (ERROR_MOD_NOT_FOUND)
# Module: DynamicCompressionModule

Get-Content "C:\Windows\System32\inetsrv\config\applicationHost.config" |
    Select-String "xpress|suscomp"
# <scheme name="xpress" dll="C:\Program Files\Update Services\WebServices\suscomp.dll" .../>
```

WSUS had been installed on SQL01 in a previous configuration and later removed. The Windows feature uninstall removed `suscomp.dll` but left the `xpress` compression scheme registered globally in IIS. IIS attempted to load the missing DLL on every request across every application on the server — not just WSUS applications. The fix:

```powershell
# Remove the orphaned global compression scheme
& "$env:SystemRoot\System32\inetsrv\appcmd.exe" set config `
    -section:system.webServer/httpCompression `
    /-"[name='xpress']"

# Remove orphaned WSUS app pool and virtual applications left behind by the uninstall
Remove-WebSite    -Name "WSUS Administration"
Remove-WebAppPool -Name "WsusPool"

iisreset /restart
Invoke-WebRequest "http://localhost/SMS_MP/" -UseBasicParsing
# 403.2 — correct; no directory listing is the expected MP response
```

After this, `mpcontrol.log` reported HTTP 200 and "Successfully performed Management Point availability check."

### CA migration failure

Documented above under Key Decisions. Two hours spent establishing that the backup/restore state was irreconcilable; one clean decision to stop. The lesson isn't that certutil backup/restore is unreliable — it's that testing a CA migration on a non-production target before attempting it on a live CA would have surfaced the issue in a lower-stakes context.

### DC SMB loopback restriction

After relocating CRL hosting to DC03, the CA's CRL publication URL was set to `\\crl.domain.dom\crl\` — a UNC path via a CNAME pointing at DC03. Certificate publication failed with ERROR_DIRECTORY.

Windows domain controllers block SMB loopback access via non-NetBIOS hostnames, including CNAMEs. The DC rejects the connection because `crl.domain.dom` doesn't resolve to a NetBIOS name it recognizes as itself. The fix is to use the local filesystem path directly: `C:\CRL\`. When the CA and the CRL web share are co-located on the same server, there's no reason to use a UNC path for the CA's write operation — the UNC path is for clients reading the CRL over HTTP, which is a different URL entry in the CA configuration.

### `_msdcs` NS delegation misconfiguration

During DC cutover DNS cleanup, the `_msdcs.domain.dom` delegation zone was reviewed. DC02 was not listed as an NS record for `_msdcs` — only DC01 was. This pre-existing misconfiguration had been in place since the domain was built. Its effect: if DC01 was slow to respond, DC locator SRV lookups in `_msdcs` would fail, causing intermittent domain authentication failures. The failure mode was intermittent and hard to attribute without inspecting the delegation zone.

Fix: remove the stale DC01 NS record, add DC02 and DC03 as authoritative. The discovery happened incidentally during cleanup rather than planned Phase 1 discovery — this kind of DNS audit should be a standard step in any AD health review.

### WsusPool private memory exhaustion

The WSUS application pool crashed overnight during the first sync on SQL01, returning HTTP 503 on all WSUS requests. Event logs showed WsusPool exceeded its private memory limit during a `GetSubscriptionState` call.

The default IIS application pool private memory limit is 1.8 GB. WSUS with a full product catalog exceeds this during sync. `4194304` KB (4 GB) is the well-known standard fix — raise the limit, increase the queue length from 1,000 to 2,000, recycle the pool. Sync completed cleanly on the next run.

### Site control file owns the site server identity

After Site Recovery completed on SQL01, every `SMS_EXECUTIVE` log entry showed inbox paths referencing `\\SCCM01.domain.dom\SMS_P01\inboxes\...`. Updating the `Site Server` value under `HKLM:\SOFTWARE\Microsoft\SMS\Identification` reverted on the next service cycle, every time.

The root cause: `SC_SiteDefinition.SiteServerName` is the authoritative source for site server identity in the SCCM site control file. On every cycle, hman and the site control manager read this value and write it back to the registry. A direct registry edit cannot win. The same field drives the server name shown in the console under Sites → P01 and every inbox UNC path that SMS_EXECUTIVE constructs — which means the console, the logs, and the inbox routing are all wrong until this one column is corrected.

```sql
-- The registry looks authoritative but isn't — confirm the real source
SELECT SiteServerName, SiteCode FROM SC_SiteDefinition
-- SCCM01.domain.dom

UPDATE SC_SiteDefinition
SET    SiteServerName = 'SQL01.domain.dom'
WHERE  SiteCode = 'P01'
```

```powershell
Restart-Service SMS_EXECUTIVE

(Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\SMS\Identification")."Site Server"
# SQL01.domain.dom — and this time it stays

# SMS_EXECUTIVE log immediately switches to:
# Refresh status manager inbox (\\SQL01.domain.dom\SMS_P01\inboxes\statmgr.box\...)
```

---

## 6. What Was Produced

The project generated documentation that would support a handover or a future rebuild:

- **As-is inventory** — complete pre-migration snapshot of all six servers, hardware specs, roles, service accounts, SCCM configuration, and Proxmox resources
- **Design documents** — consolidated target-state architecture, DC rebuild rationale (including DC rename research), SCCM consolidation options analysis
- **Three operator runbooks** — DC03 provisioning and cutover, SCCM consolidation and Site Recovery, W2016 decommission; each includes pre-checks, validation gates, and rollback procedures
- **Implementation record** — detailed execution log capturing every deviation from planned runbooks, every issue encountered, and how each was resolved
- **BAU handover note** — post-consolidation maintenance tasks, WSUS cleanup frequency, ADR review cadence, NTP hierarchy, PKI enrollment status, backup locations
- **Two maintenance scripts** — `Invoke-WsusCleanup.ps1` (weekly: decline expired/superseded, run cleanup wizard, recover disk) and `Invoke-AdrPackageCleanup.ps1` (monthly: remove stale ADR update content)

`Invoke-WsusCleanup.ps1` runs each cleanup flag individually rather than in a combined call — a combined `PerformCleanup` call on a large SUSDB frequently exceeds the default WCF operation timeout (~10 min) and fails silently. The script also attempts to extend the WCF channel timeout before starting:

```powershell
# Extend WCF channel operation timeout to 2 hours before running cleanup
try {
    Add-Type -AssemblyName 'System.ServiceModel' -ErrorAction Stop
    $icc = $cleanupManager -as [System.ServiceModel.IContextChannel]
    if ($icc) { $icc.OperationTimeout = [TimeSpan]::FromHours(2) }
} catch { <# non-fatal; proceed with default timeout #> }

# Each flag runs in its own CleanupScope call — a slow step cannot time out the others
foreach ($step in $cleanupSteps) {
    $scope = New-Object Microsoft.UpdateServices.Administration.CleanupScope
    $scope.$($step.Flag) = $true
    $result = $cleanupManager.PerformCleanup($scope)
}
```

`Invoke-AdrPackageCleanup.ps1` queries the SCCM SQL database directly instead of using the WMI content chain (`SMS_PackageToContent` → `SMS_CIToContent`). The WMI chain undercounts packages that share content blobs — common with Defender definition updates, where many accumulated versions share a single ContentID and most are invisible to the WMI query. The SQL view is authoritative:

```powershell
# v_DeploymentPackageUpdateContent records every CI_ID explicitly added to a package
# The WMI chain (SMS_PackageToContent → SMS_CIToContent) misses updates that share
# content blobs, because SMS_CIToContent returns at most one CI_ID per ContentID
$ciIds = (Invoke-SccmQuery -Server $SiteServer -Database "CM_$SiteCode" `
    -Query "SELECT CI_ID FROM v_DeploymentPackageUpdateContent
            WHERE PackageID = '$($pkg.PackageID)'").CI_ID

# Stale = expired, OR superseded AND WSUS has declined them (LocalUpdateState 4)
# Run after Invoke-WsusCleanup.ps1 so superseded updates are already declined
$stale = $allUpdates | Where-Object {
    $_.IsExpired -eq $true -or ($_.IsSuperseded -eq $true -and $_.LocalUpdateState -eq 4)
}
```

---

## 7. Observations

**What worked well:**

The DNS alias + IP reuse approach to the DC rename problem was the right call. It avoided every documented failure mode, took fifteen minutes to implement, and left DC03 with a clean identity. Any future service that hardcodes DC01 by name will still resolve correctly.

Site Recovery as the SCCM migration path was the right call once the passive promotion blocker was understood. It's not the first approach most people try because the documentation leads with passive promotion — but for any migration where the target already has roles assigned, it's the correct tool.

The decision to stop the CA migration after two hours and install fresh was the right call. The alternative was continuing to debug an underdetermined key/certificate state problem with no clear resolution path. Knowing when to stop and restart cleanly is a useful judgment to have.

The Phase 1 inventory produced a document thorough enough to drive all subsequent design decisions. Proxmox resource verification early in the project was what ruled out SCCM virtualization before significant work was invested.

**What to do differently:**

Phase 1 missed W2016-1's CRL hosting. The inventory covered services and roles but not DNS records. A DNS review — walking every CNAME and A record, cross-referencing against running services — should be part of any role discovery that precedes a server decommission.

The CA migration should have been tested on a snapshot or non-production target before being attempted on the live CA. The certutil backup/restore process appeared to succeed at every step; the failure only surfaced when certsvc attempted to start. Fifteen minutes of testing in a scratch VM would have surfaced that before the live attempt.

The `_msdcs` NS delegation misconfiguration should have been caught in Phase 1. An AD health check that validates replication summary, NS records, `_msdcs` delegation, SRV registration, and FSMO holders is worth running at the start of any consolidation project — not as a post-cutover cleanup step.

---

[← Back to Projects](../projects/index.md)
