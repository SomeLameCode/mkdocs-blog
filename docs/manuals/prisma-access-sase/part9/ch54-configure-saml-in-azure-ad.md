---
title: "Configure SAML in Azure AD for Prisma Access Explicit Proxy"
description: "Step-by-step Azure AD Enterprise Application setup for Explicit Proxy SAML SSO: Entity ID, ACS URL, user attributes, group assignment, optional certificate signing, and metadata XML download."
date: 2026-05-28
tags:
  - prisma-access
  - sase
  - paloalto
  - explicit-proxy
  - cloud-swg
  - saml
  - azure-ad
  - sso
  - enterprise-application
status: published
---

# Chapter 54 — Configure SAML in Azure AD for Prisma Access Explicit Proxy

This chapter walks through the complete Microsoft Entra ID (formerly Azure AD) configuration required to enable SAML SSO for Prisma Access Explicit Proxy. The output is a metadata XML file that is imported into Panorama or Strata Cloud Manager in Chapter 53.

> ⚠️ **Scope of this chapter, relative to Chapter 53's Cloud Identity Engine finding — read this before starting.** The exact Entity ID and ACS URL values used throughout this chapter (`https://global.acs.prismaaccess.com/saml/metadata` and `.../saml/acs`) are specific to the **direct (non-CIE) configuration path** — the approach this chapter and Chapter 53 document step by step. Chapter 53 found that Palo Alto's currently-**recommended** path for Azure AD SAML uses **Cloud Identity Engine** instead, which requires **different Entity ID / Reply URL values sourced from the Cloud Identity Engine app's own SP Metadata**, not the fixed Prisma Access values below. If you're following the CIE-recommended path, the specific values in this chapter do not apply — see Chapter 53's Cloud Identity Engine section and the CIE Azure AD configuration documentation instead; this chapter's scope is the Microsoft-side steps for the direct path specifically, not re-explained for CIE.

**Note on terminology, confirmed via Microsoft Learn (not guessed):** Microsoft has rebranded **Azure Active Directory to Microsoft Entra ID**, and the **Azure Portal** section for this is now the **Microsoft Entra admin center** (`entra.microsoft.com`). This chapter uses current Microsoft terminology throughout — if you've used the older Azure AD portal before, the navigation labels below will look different, but the underlying SAML concepts and the Prisma-Access-side values are unchanged.

---

## Prerequisites

- A Microsoft Entra ID tenant, with one of: **Cloud Application Administrator**, **Application Administrator**, or **owner of the service principal** — confirmed current via Microsoft Learn; this differs from "Global Administrator or Application Administrator" previously stated here
- Users/groups to be assigned to Explicit Proxy already exist in Microsoft Entra ID

---

## Steps 1–6 — Create the Enterprise Application

**Step 1** — **Terminology updated 2026-07-09, confirmed via Microsoft Learn.** Log in to the **Microsoft Entra admin center** (`entra.microsoft.com`) as at least a Cloud Application Administrator → browse to **Entra ID > Enterprise apps > All applications** → click **+ New application**

> 📷 [Azure AD screenshot — Enterprise Applications > New Application](https://docs.paloaltonetworks.com/prisma-access/administration/prisma-access-mobile-users/mobile-users-explicit-proxy/set-up-explicit-proxy)

**Step 2** — Click **Create your own application** *(confirmed unchanged wording)*

**Step 3** — Provide the application details:
- **Name:** `Prisma-Access-Explicit-Proxy` (or any descriptive name)
- **Integrate any other application you don't find in the gallery (non-gallery)** — select this option *(confirmed current wording)*

**Step 4** — Navigate to the newly created application and click on it

**Step 5** — Click **Set up single sign on**

**Step 6** — Select **SAML** as the SSO method

---

## Steps 7–11 — Configure Basic SAML Settings

**Confirmed unchanged via Microsoft Learn:** the "Basic SAML Configuration" section and the "Identifier (Entity ID)" / "Reply URL (Assertion Consumer Service URL)" field names in Steps 7–11 below are still current terminology in the Microsoft Entra admin center — no rewording needed for these specific steps.

**Step 7** — Click **Edit** in the **Basic SAML Configuration** section

**Step 8** — Click **Add identifier** under **Identifier (Entity ID)**

**Step 9** — Enter the Entity ID and set as Default:

| Field | Value |
|---|---|
| **Identifier (Entity ID)** | `https://global.acs.prismaaccess.com/saml/metadata` |
| **Default** | Yes |

> 📷 [Azure AD screenshot — Entity ID configuration](https://docs.paloaltonetworks.com/prisma-access/administration/prisma-access-mobile-users/mobile-users-explicit-proxy/set-up-explicit-proxy)

**Step 10** — Click **Add reply URL** under **Reply URL (Assertion Consumer Service URL)**

**Step 11** — Enter the ACS URL and set as Default:

| Field | Value |
|---|---|
| **Reply URL (ACS URL)** | `https://global.acs.prismaaccess.com/saml/acs` |
| **Default** | Yes |

> 📷 [Azure AD screenshot — ACS URL configuration](https://docs.paloaltonetworks.com/prisma-access/administration/prisma-access-mobile-users/mobile-users-explicit-proxy/set-up-explicit-proxy)

---

## Steps 12–15 — Configure User Attributes & Claims

**Step 12** — In the **Set Up Single Sign-On with SAML** pane, click **Edit** in the **User Attributes & Claims** section

**Step 13** — Set the **Unique User Identifier (Name ID)**:

| Field | Value |
|---|---|
| **Name ID** | `user.userprincipalname` |

Then click **Add new claim**

**Step 14** — Create the `username` claim:

| Field | Value |
|---|---|
| **Claim Name** | `username` |
| **Source Attribute** | `user.userprincipalname` |

Click **Save**

**Step 15** — Verify the new `username` claim appears in the claims list

> The `username` claim is what Panorama or Strata Cloud Manager reads to identify the authenticated user. Its value must match the **Username Attribute** field in the authentication profile (ch53).

---

## Steps 16–19 — Assign Users and Groups

**Step 16** — Click **Assign users & groups** (or navigate to **Users and groups** in the left menu)

**Step 17** — Click **Add user/group**

**Step 18** — Select the Microsoft Entra ID users and/or groups that should have access to Explicit Proxy → click **Assign**

**Step 19** — Verify that assigned users appear in the users and groups list

> Only users assigned to this Enterprise Application can authenticate through Explicit Proxy. Unassigned users will receive an access denied error from Microsoft Entra ID.

---

## Steps 20–26 — Optional: SAML Signing Certificate

These steps configure a custom signing certificate so Panorama (or Strata Cloud Manager, per Chapter 53) can validate the SAML assertion signature. This is optional but recommended for production deployments.

> ℹ️ **Navigation updated 2026-07-09, confirmed via Microsoft Learn.** The signing-certificate area is now reached via the **SAML Certificates** section, under its **Token Signing Certificate** heading — select the **Edit (pencil icon)** there to open the **SAML Signing Certificate** page, rather than a single flat section as previously described. The underlying capability (import a custom certificate) is unchanged.

**Step 20** — In the SAML setup pane, locate the **SAML Certificates** section, and select the **Edit (pencil icon)** next to **Token Signing Certificate** to open the **SAML Signing Certificate** page

**Step 21** — Generate a self-signed certificate in Panorama:
`Panorama > Device > Certificates > Generate`

Provide a certificate name, CN, and validity period.

**Step 22** — Export the generated certificate from Panorama (PEM or CER format)

**Step 23** — Back in the Microsoft Entra admin center, on the **SAML Signing Certificate** page, click **Edit**

**Step 24** — Click **Import certificate**

**Step 25** — Select the exported Panorama certificate and provide the password if applicable → save

> This binds Microsoft Entra ID's SAML assertions to the Panorama certificate, allowing Panorama (or Strata Cloud Manager) to validate the signature on incoming SAML responses.

---

## Step 26 — Download the Metadata XML

**Step 26** — In the **SAML Certificates** section, click **Download** next to **Federation Metadata XML** *(confirmed unchanged terminology)*

Save this file — it will be imported into Panorama or Strata Cloud Manager in the next step (ch53, Step 2).

> 📷 [Azure AD screenshot — Download Federation Metadata XML](https://docs.paloaltonetworks.com/prisma-access/administration/prisma-access-mobile-users/mobile-users-explicit-proxy/set-up-explicit-proxy)

---

## Summary of Key Values

| Item | Value |
|---|---|
| Entity ID | `https://global.acs.prismaaccess.com/saml/metadata` |
| ACS URL | `https://global.acs.prismaaccess.com/saml/acs` |
| Name ID attribute | `user.userprincipalname` |
| Username claim | `username` = `user.userprincipalname` |
| Metadata file | Downloaded from Microsoft Entra ID — import into Panorama or Strata Cloud Manager |

> Reminder: these values apply to the **direct (non-CIE) path** documented in this chapter and Chapter 53 — see the scoping note near the top of this chapter if you're using Palo Alto's CIE-recommended path instead.

---

## After This Chapter

Return to **Chapter 53** to:
- Import the metadata XML into Panorama (`Device > Server Profiles > SAML Identity Provider > Import`) or Strata Cloud Manager (`Mobile Users > Explicit Proxy Setup > User Authentication > SAML IdP Profile`, confirmed in Chapter 53)
- Create the authentication profile in `Explicit_Proxy_Template` (Panorama) or via `Identity Services > Authentication > Authentication Profiles` (Strata Cloud Manager, confirmed scoped under Prisma Access — see Chapter 53)
- Commit & Push (Panorama) or Push Config (Strata Cloud Manager, per Chapter 28) to the **Explicit Proxy** scope

> **No SCM-specific section was added to this chapter itself** — confirmed, not assumed: all 26 steps in this chapter happen entirely within the Microsoft Entra admin center, none touch Panorama or Strata Cloud Manager configuration. The Prisma-Access-side SCM navigation belongs to, and is already covered in, Chapter 53.

---

## Key Takeaways

- Entity ID and ACS URL are fixed Prisma Access values for the **direct (non-CIE) path** specifically — do not customise them, and don't reuse them if you're on the CIE-recommended path instead (see Chapter 53)
- The `username` claim (mapped to `user.userprincipalname`) must match the authentication profile's Username Attribute field (Panorama or Strata Cloud Manager, per Chapter 53)
- Only users/groups assigned to the Enterprise Application can authenticate — unassigned users are blocked at the IdP
- The metadata XML file auto-populates all SAML IdP profile fields on import, on either management platform
- The optional Panorama-issued signing certificate allows Microsoft Entra ID to sign assertions that Panorama can verify
- **Terminology corrected 2026-07-09**: "Azure Portal" is now the **Microsoft Entra admin center**; "Azure AD" is **Microsoft Entra ID**; the minimum required role is **Cloud Application Administrator, Application Administrator, or owner of the service principal** — not Global Administrator
- This chapter needs no SCM-specific section of its own — it's entirely about the Microsoft-side configuration; the SCM-relevant Prisma Access side lives in Chapter 53

---

*Previous: [Chapter 53 — Configure Azure AD SAML Profile & Authentication Profile](./ch53-configure-azure-ad-saml-profile.md)* · *Next: [Appendix A — Glossary of Terms](../appendix/appA-glossary.md)*
