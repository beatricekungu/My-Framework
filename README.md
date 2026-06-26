# ZeroTrustHealthDB – A GRC‑Driven Azure SQL Data Protection Engagement

**Consultant:** Beatrice – Microsoft Certified: Azure Database Administrator Associate  
**Client:** GlobalMed Health & Financial (fictitious multinational healthcare & insurance provider)  


---

## Engagement Overview

### The Client Problem
GlobalMed is migrating legacy on‑premises databases to the Azure Cloud. A recent internal IT audit uncovered a critical vulnerability:  
- Offshore data analysts  
- Marketing teams  
- Tier‑1 IT support  

All had **unrestricted, raw access** to tables containing Patient PII (SSN, Email) and PCI data (Credit Card numbers).  

This violated the **Principle of Least Privilege** and directly contravened **HIPAA, GDPR, and PCI‑DSS** requirements. A single breach could result in tens of millions in regulatory fines and irreparable reputational harm.

### My Role as Senior GRC Cloud Security Consultant
GlobalMed engaged me to architect and **technically implement** a “Zero‑Trust” data layer that enforces compliance at the database engine level—not merely through policy documents.  

The deliverable: a functional proof‑of‑concept in Azure SQL Database that proves sensitive data can be protected automatically, regardless of the application or user connecting to the database.

---

## Compliance Frameworks Addressed

| Regulation | Key Requirement | Technical Control Implemented |
|------------|-----------------|-------------------------------|
| **HIPAA** §164.312(a)(1) & §164.312(b) | Access controls, audit controls | RBAC, Dynamic Data Masking, Auditing |
| **GDPR** Art. 25, Art. 32 | Data protection by design & security of processing | DDM, Data Classification, Vulnerability Assessment |
| **PCI‑DSS** Req 3.4, Req 7.1, Req 10.1 | Render PAN unreadable, restrict access, log all access | DDM on Credit Card, UNMASK for auditor only, SQL Auditing |

---

## Solution Architecture: Zero‑Trust Data Layer

I built a single Azure SQL Database (free tier) with three distinct security personas, each with strictly scoped permissions. All enforcement happens **inside the database engine**—zero reliance on application‑side masking or filtering.

### Personas & Permissions

| Persona | Database Role | DDM Masking | UNMASK Privilege | Notes |
|---------|---------------|-------------|------------------|-------|
| **Cloud Architect (dbo)** | db_owner | None | Yes (implicitly as owner) | Full administrative control; builds and maintains the system |
| **Chief Compliance Officer (Auditor)** | db_datareader | Bypassed | **GRANT UNMASK** | Can view plaintext PII/PCI only during regulatory audits; every access is fully logged by Auditing |
| **Data Analyst (Tier‑1)** | db_datareader | Enforced | No | All SSNs, Emails, and Credit Cards are forcibly masked by the engine; cannot bypass |

### Technical Controls Deployed

1. **Dynamic Data Masking (DDM)**  
   - Email masked with `email()` function – e.g., `a...@healthmail.com`  
   - SSN masked to show only last 4 digits – `XXX‑XX‑6789`  
   - Credit Card masked to show only last 4 digits – `XXXX‑XXXX‑XXXX‑1111`  
   *Result: The Analyst sees nothing but placeholders, even with a direct `SELECT *`.*

2. **Granular RBAC & UNMASK Permission**  
   - `GRANT UNMASK TO AuditorUser` alone.  
   - No other user, not even the Analyst, can bypass masking.  
   *This enforces separation of duties and auditability.*

3. **SQL Auditing**  
   - Server‑level audit logs capture every query, including the auditor’s unmasked `SELECT`.  
   - Logs stored in Azure Blob Storage, retention configurable.  
   *Provides a non‑repudiable trail for HIPAA §164.312(b) and GDPR Art. 30.*

4. **Microsoft Defender for SQL (30‑day trial)**  
   - **Data Discovery & Classification** – automatically scans and labels sensitive columns (SSN, Credit Card, Email) with regulatory tags.  
   - **Vulnerability Assessment** – runs periodic scans against CIS/Microsoft security baselines, flagging misconfigurations.  
   *Demonstrates continuous compliance monitoring and proactive risk management.*

---

## Evidence Portfolio: Before‑and‑After Screenshots

These screenshots are the core deliverable—they visually prove that compliance is **technically enforced**, not just documented.

| Screenshot | Description | Compliance Story |
|------------|-------------|------------------|
| **Admin Plaintext View** | `SELECT *` as dbo – all PII/PCI visible | Baseline: data is properly stored and accessible to admin |
| **Analyst Before DDM** | Analyst sees SSN, email, credit card in clear | The vulnerability: unrestricted PII/PCI access |
| **Auditor Before UNMASK** | Auditor also sees everything (mask not yet applied) | Even privileged roles should start with least access |
| **Analyst After DDM** | Same query, data now masked – `XXX‑XX‑6789`, `a...@healthmail.com`, etc. | Enforces HIPAA/GDPR/PCI directly at the database level |
| **Auditor After UNMASK** | Auditor can still view plaintext, but with `UNMASK` privilege logged | Separation of duties: override is granted, not default |
| **Audit Log** | Portal view of audit records showing Auditor’s unmasked query | Complete audit trail for any PII access |
| **Data Classification Dashboard** | Auto‑classified SSN as *National ID*, Credit Card as *Credit Card*, etc. | GDPR Art. 30 data inventory, PCI‑DSS requirement |
| **Vulnerability Assessment** | Scan results showing secure configuration (or identified remediations) | Evidence of ongoing security assessment (GDPR Art. 32) |

---

## How This Maps to the Azure Database Administrator Associate Certification

- **Implement a secure environment** (DDM, RBAC, T‑SQL `GRANT UNMASK`)  
- **Monitor, audit, and optimize** (SQL Auditing, Defender for SQL)  
- **Configure database authentication and authorization** (contained users, logins, role membership)  
- **Automate database tasks** (all scripts are idempotent, ready for CI/CD)  

This project proves I can **architect, deploy, and manage** Azure SQL databases in a heavily regulated context—exactly the skill set the certification validates.

---

## Repository Contents

