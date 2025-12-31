# Azure SQL Database Security Hardening
## Technical Implementation Guide

**Author:** Praise Effiom  
**Role:** Cloud Security Engineer  
**Contact:** [LinkedIn](https://linkedin.com/in/praiseeffiom)  
**Last Updated:** December 2025  
**Version:** 1.0

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Architecture](#architecture)
4. [Implementation Steps](#implementation-steps)
5. [Validation & Testing](#validation--testing)
6. [Troubleshooting](#troubleshooting)
7. [Next Steps](#next-steps)

---

## Overview

### Purpose
This guide provides step-by-step instructions for implementing enterprise-grade security controls on Azure SQL Database. The implementation transforms a baseline database deployment into a fully hardened, compliance-ready environment suitable for production workloads.

### What You'll Build
- Azure SQL Database with AdventureWorksLT sample data
- Microsoft Defender for SQL with threat protection
- Data classification with GDPR compliance labels
- Comprehensive audit logging with 5-day retention
- Security monitoring through Microsoft Defender for Cloud

### Time Requirements
- **Initial Deployment:** 15 minutes
- **Security Configuration:** 15-20 minutes
- **Testing & Validation:** 10 minutes
- **Total:** ~45 minutes

### Skill Level
Intermediate - Requires basic Azure portal navigation and understanding of database security concepts.

---

## Prerequisites

### Required Azure Resources
- Active Azure subscription (Free trial, Pay-as-you-go, or Student)
- Resource Group in East US region
- Sufficient permissions to create:
  - SQL Servers
  - SQL Databases
  - Storage Accounts
  - Security configurations

### Required Knowledge
- Basic SQL Server concepts
- Azure portal navigation
- Understanding of PaaS services
- Security and compliance fundamentals

### Tools & Access
- Web browser (Chrome, Edge, or Firefox recommended)
- Azure Portal access: https://portal.azure.com
- ARM template file (provided in `/arm-templates/azuredeploy.json`)

### Cost Considerations
- **Azure SQL Database:** ~$5-15/day (Basic/Standard tier)
- **Storage Account:** ~$0.10-0.50/day (for audit logs)
- **Microsoft Defender for SQL:** 30-day free trial, then ~$15/server/month
- **Estimated Total:** ~$25-30 for complete lab (if using paid subscription)

**Note:** Use Azure Cost Calculator for precise estimates based on your region.

---

## Architecture

### Logical Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Azure Subscription                       │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              Resource Group: rg_eastus_XXXXX          │  │
│  │                                                        │  │
│  │  ┌──────────────────────────────────────────────┐    │  │
│  │  │         SQL Server: sqlserver-whizlabs       │    │  │
│  │  │                                              │    │  │
│  │  │  ┌────────────────────────────────────┐    │    │  │
│  │  │  │  Database: AdventureWorksLT        │    │    │  │
│  │  │  │  - Sample Data (SalesLT schema)    │    │    │  │
│  │  │  │  - TDE Enabled                     │    │    │  │
│  │  │  │  - Data Classification Applied     │    │    │  │
│  │  │  └────────────────────────────────────┘    │    │  │
│  │  │                                              │    │  │
│  │  │  Security Features:                         │    │  │
│  │  │  • Microsoft Defender for SQL               │    │  │
│  │  │  • Vulnerability Assessment                 │    │  │
│  │  │  • Advanced Threat Protection               │    │  │
│  │  │  • SQL Auditing → Storage Account          │    │  │
│  │  └──────────────────────────────────────────────┘    │  │
│  │                                                        │  │
│  │  ┌──────────────────────────────────────────────┐    │  │
│  │  │    Storage Account: whizstorage[name]        │    │  │
│  │  │    - Audit Logs (5-day retention)            │    │  │
│  │  │    - Blob Storage                            │    │  │
│  │  └──────────────────────────────────────────────┘    │  │
│  │                                                        │  │
│  │  ┌──────────────────────────────────────────────┐    │  │
│  │  │     Microsoft Defender for Cloud             │    │  │
│  │  │     - Security Recommendations               │    │  │
│  │  │     - Security Alerts                        │    │  │
│  │  │     - Compliance Dashboard                   │    │  │
│  │  └──────────────────────────────────────────────┘    │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Security Layers

**Layer 1: Data Protection**
- Transparent Data Encryption (TDE) - Enabled by default
- TLS 1.2 for data in transit
- Data classification for sensitive columns

**Layer 2: Access Control**
- SQL authentication with strong passwords
- Database-level permissions
- Network security rules (configurable)

**Layer 3: Threat Detection**
- Microsoft Defender for SQL
- Real-time threat detection
- SQL injection prevention
- Anomalous access detection

**Layer 4: Compliance & Auditing**
- Comprehensive audit logging
- 5-day retention in Azure Storage
- GDPR-compliant data classification
- Security posture monitoring

---

## Implementation Steps

### Phase 1: Initial Deployment

#### Step 1.1: Access Azure Portal

1. Navigate to https://portal.azure.com in incognito/private browsing mode
2. Sign in with your Azure credentials
3. Verify you're in the correct subscription

**Why incognito?** Prevents cache issues and ensures clean session.

#### Step 1.2: Deploy ARM Template

1. In the Azure Portal search bar, type: `Deploy a custom template`
2. Click **Deploy a custom template** from search results
3. On the Custom deployment page, click **Build your own template in the editor**

#### Step 1.3: Load ARM Template

1. Download the ARM template from the project repository:
   ```
   /arm-templates/azuredeploy.json
   ```
2. In the template editor, click **Load file**
3. Select the downloaded `azuredeploy.json` file
4. Click **Save**

#### Step 1.4: Configure Deployment Parameters

Enter the following details in the **Basics** tab:

| Parameter | Value |
|-----------|-------|
| **Subscription** | Your Azure subscription |
| **Resource Group** | Select `rg_eastus_XXXXX` (or create new) |
| **Region** | (US) East US |

**Additional Parameters** (auto-populated from template):
- SQL Server Name: `sqlserver-whizlabs` (or custom)
- Database Name: `AdventureWorksLT`
- Administrator Login: `sqladmin`
- Administrator Password: (auto-generated secure password)

5. Click **Review + create**
6. Review the configuration
7. Click **Create**

**Deployment Time:** 5-10 minutes

#### Step 1.5: Verify Deployment

1. Once deployment completes, click **Go to resource group**
2. Verify the following resources exist:
   - SQL Server (e.g., `sqlserver-whizlabs`)
   - SQL Database (`AdventureWorksLT`)
3. Note down the SQL Server name for later use

---

### Phase 2: Microsoft Defender for SQL Configuration

#### Step 2.1: Navigate to SQL Server

1. In your resource group, click on the **SQL Server** resource
2. In the left panel, under **Security**, click **Microsoft Defender for Cloud**

#### Step 2.2: Enable Microsoft Defender

1. On the overview page, click **Enable Microsoft Defender for SQL**
2. Wait for the notification: "Microsoft Defender for SQL has been enabled"
3. Refresh the page

**What This Enables:**
- Vulnerability assessment scanning
- Advanced threat protection
- Security recommendations
- Real-time alerts

#### Step 2.3: Configure Server Settings

1. In the **Enablement Status** section, click **(Configure)**
2. Review the following settings:

**Vulnerability Assessment Settings:**
- Storage account for scan results (auto-created or select existing)
- Periodic recurring scans: Enabled
- Send scan reports to: (optional email addresses)

**Advanced Threat Protection Settings:**
- Enabled by default
- Detects:
  - SQL injection attempts
  - Brute force attacks
  - Anomalous access patterns
  - Data exfiltration attempts

3. Configure alert recipients (optional):
   ```
   Email: your-security-team@company.com
   ```

4. Click **Save** if you made changes

#### Step 2.4: Review Security Recommendations

1. Return to **Microsoft Defender for Cloud** blade
2. Click on **Recommendations** tab
3. Review security recommendations (may take 10-15 minutes to populate)

**Common Initial Recommendations:**
- Enable transparent data encryption (already enabled)
- Configure vulnerability assessment
- Enable auditing
- Restrict network access

**Note:** Keep this tab open as you'll address these recommendations in subsequent phases.

---

### Phase 3: Data Classification Implementation

#### Step 3.1: Access Data Classification

1. Navigate back to your resource group
2. Click on **SQL Databases** in the left panel
3. Select your database (e.g., `AdventureWorksLT`)
4. Under **Security**, click **Data Discovery & Classification**
5. Click the **Classification** tab

#### Step 3.2: Add First Classification

1. Click **+ Add classification**
2. Configure the first classification:

| Field | Value |
|-------|-------|
| **Schema name** | SalesLT |
| **Table name** | Customer |
| **Column name** | FirstName |
| **Information type** | Name |
| **Sensitivity label** | Confidential - GDPR |

3. Click **Add classification**

#### Step 3.3: Add Remaining Classifications

Repeat the above process for these columns:

**Classification 2:**
- Schema: `SalesLT`
- Table: `Customer`
- Column: `LastName`
- Information Type: `Name`
- Sensitivity Label: `Confidential - GDPR`

**Classification 3:**
- Schema: `SalesLT`
- Table: `Customer`
- Column: `EmailAddress`
- Information Type: `Contact Info`
- Sensitivity Label: `Confidential - GDPR`

**Classification 4:**
- Schema: `SalesLT`
- Table: `Customer`
- Column: `Phone`
- Information Type: `Contact Info`
- Sensitivity Label: `Confidential - GDPR`

#### Step 3.4: Save and Verify

1. After adding all 4 classifications, click **Save** at the top
2. Switch to the **Overview** tab
3. Verify the dashboard shows:
   - 4 classified columns
   - 1 table
   - 2 information types
   - 100% of sensitive data classified

**What This Achieves:**
- Complete inventory of sensitive data
- GDPR compliance labeling
- Foundation for data protection policies
- Audit trail of data access

---

### Phase 4: Audit Logging Configuration

#### Step 4.1: Enable Server-Level Auditing

1. Navigate back to your **SQL Server** (not the database)
2. Under **Security**, click **Auditing**
3. Set **Enable Azure SQL Auditing** to **ON**

#### Step 4.2: Configure Storage Destination

1. Select the **Storage** checkbox
2. Click **Subscription**: Select your subscription
3. Click **Storage account**: Select **Create new**

Configure new storage account:
```
Name: whizstorage[yourname]
Performance: Standard
Replication: Locally-redundant storage (LRS)
```

4. Click **OK**

#### Step 4.3: Set Retention Policy

Under **Advanced properties**:
1. Set **Retention (days)**: `5`
2. Leave other settings at default

**Retention Considerations:**
- 5 days: Suitable for labs/demos
- 30-90 days: Common for production
- 365+ days: Required for regulated industries

3. Click **Save**
4. Wait for confirmation: "Successfully updated auditing settings"

#### Step 4.4: Verify Audit Logs

1. Navigate to your **SQL Database** (not the server)
2. Under **Security**, click **Auditing**
3. Note that auditing is inherited from server
4. Click **View audit logs**

**Available Views:**
- Server audit: All databases on the server
- Database audit: Specific database only

5. In the **Audit records** blade, select a time range
6. Note: Fresh deployments may not have events yet

**Events That Will Be Logged:**
- Authentication attempts (success/failure)
- Data access (SELECT, INSERT, UPDATE, DELETE)
- Schema changes (CREATE, ALTER, DROP)
- Permission changes (GRANT, REVOKE)
- Database connections
- Query execution

---

### Phase 5: Security Validation

#### Step 5.1: Microsoft Defender for Cloud Dashboard

1. Navigate to **Microsoft Defender for Cloud**
2. Review the main dashboard sections:

**Security Posture:**
- Secure Score: Should be improving (target: 80%+)
- Active recommendations
- Security alerts (should be 0 for new deployment)

**Key Metrics to Monitor:**
- Number of recommendations
- High/Medium/Low severity distribution
- Resolved vs. Active recommendations

#### Step 5.2: Test Data Classification

1. Navigate to **Data Discovery & Classification**
2. Click **Overview** tab
3. Verify metrics:
   - ✓ 4 classified columns
   - ✓ 100% coverage
   - ✓ Confidential - GDPR labels applied

#### Step 5.3: Verify Audit Configuration

1. Check **Auditing** status on SQL Server: ✓ Enabled
2. Check storage account exists: ✓ `whizstorage[name]`
3. Verify retention: ✓ 5 days

#### Step 5.4: Security Controls Checklist

Use this checklist to verify your implementation:

- [ ] SQL Server deployed in East US
- [ ] AdventureWorksLT database created
- [ ] Microsoft Defender for SQL enabled
- [ ] Vulnerability assessment configured
- [ ] Advanced threat protection active
- [ ] 4 columns classified with GDPR labels
- [ ] Server-level auditing enabled
- [ ] Storage account created for audit logs
- [ ] 5-day retention configured
- [ ] All changes saved successfully

---

## Validation & Testing

### Security Testing Scenarios

#### Test 1: Verify Data Classification

**Objective:** Confirm sensitive data is properly labeled

**Steps:**
1. Navigate to Data Discovery & Classification
2. Run a query to access classified columns:
   ```sql
   SELECT FirstName, LastName, EmailAddress, Phone 
   FROM SalesLT.Customer 
   WHERE CustomerID = 1;
   ```
3. Check audit logs for the query
4. Verify access is logged with sensitivity labels

**Expected Result:** Query executes successfully, audit log contains classification metadata

#### Test 2: Trigger Security Alert (Optional)

**Objective:** Verify threat detection is working

**Warning:** Only perform in non-production environment

**Steps:**
1. Attempt multiple failed login attempts
2. Try a basic SQL injection pattern in a query
3. Wait 5-10 minutes
4. Check Microsoft Defender for Cloud for alerts

**Expected Result:** Security alert generated in Defender for Cloud

#### Test 3: Audit Log Verification

**Objective:** Confirm comprehensive logging

**Steps:**
1. Perform various database operations:
   - Login to database
   - Read data (SELECT)
   - Modify data (UPDATE)
   - Create object (CREATE TABLE test)
2. Wait 2-3 minutes for log processing
3. Navigate to Audit records
4. Verify all operations are logged

**Expected Result:** All operations visible in audit logs with timestamps and user information

---

## Troubleshooting

### Common Issues and Solutions

#### Issue 1: Deployment Fails

**Symptoms:**
- ARM template deployment error
- "Resource already exists" error

**Solutions:**
1. Ensure resource names are unique globally
2. Check subscription quotas and limits
3. Verify you have Owner or Contributor role
4. Try different region if East US has capacity issues

**Prevention:**
- Use unique naming convention with your initials
- Review template parameters before deployment

---

#### Issue 2: Defender for SQL Won't Enable

**Symptoms:**
- "Failed to enable Microsoft Defender" error
- Button stays grayed out

**Solutions:**
1. Refresh the browser page
2. Wait 2-3 minutes after deployment
3. Verify you have Security Admin role
4. Check if free trial is still available
5. Clear browser cache and try again

**Prevention:**
- Wait for full deployment before configuring security

---

#### Issue 3: Classifications Not Saving

**Symptoms:**
- "Failed to save classification" error
- Classifications disappear after saving

**Solutions:**
1. Ensure database is online and accessible
2. Verify column names match exactly (case-sensitive)
3. Add classifications one at a time
4. Wait 1 minute between additions

**Prevention:**
- Verify schema and table names before adding
- Double-check spelling and casing

---

#### Issue 4: No Audit Logs Appearing

**Symptoms:**
- Audit records blade is empty
- "No data available" message

**Solutions:**
1. Confirm auditing is enabled (check green checkmark)
2. Wait 5-10 minutes for initial log processing
3. Verify storage account has no access restrictions
4. Perform database operations to generate events
5. Check correct time range is selected

**Expected Behavior:**
- New deployments: Logs appear within 10 minutes
- Existing databases: Logs appear within 2-3 minutes

---

#### Issue 5: Storage Account Creation Fails

**Symptoms:**
- "Storage account name not available"
- Creation hangs or times out

**Solutions:**
1. Use a more unique name: `whizstorage[yourname][date]`
2. Ensure name is 3-24 characters, lowercase letters and numbers only
3. Try different storage account type
4. Select existing storage account instead

**Prevention:**
- Check name availability before submitting
- Use naming convention with timestamp

---

### Getting Help

**Azure Support Resources:**
- Azure Documentation: https://docs.microsoft.com/azure
- Microsoft Q&A: https://aka.ms/azureqa
- Azure Support: https://azure.microsoft.com/support

**Project-Specific Help:**
- Contact: [Praise Effiom](https://linkedin.com/in/praiseeffiom)

---

## Next Steps

### Immediate Enhancements

1. **Network Security Hardening**
   - Implement Azure Private Endpoints
   - Configure Network Security Groups (NSGs)
   - Enable IP whitelisting
   - Remove public endpoint access

2. **Advanced Authentication**
   - Enable Azure AD authentication
   - Disable SQL authentication
   - Implement multi-factor authentication (MFA)
   - Configure managed identities

3. **Data Protection**
   - Implement Dynamic Data Masking
   - Configure Row-Level Security
   - Enable Always Encrypted for highly sensitive data
   - Set up customer-managed encryption keys

### Production Readiness

4. **High Availability & Disaster Recovery**
   - Configure geo-replication
   - Set up automatic backups (already enabled)
   - Test restore procedures
   - Implement failover groups

5. **Monitoring & Alerting**
   - Integrate with Azure Sentinel
   - Configure alert rules
   - Set up automated response playbooks
   - Create monitoring dashboards

6. **Compliance & Governance**
   - Implement Azure Policy for enforcement
   - Configure compliance reports
   - Schedule regular security assessments
   - Document security procedures

### Learning Resources

- [Azure SQL Database Security Best Practices](https://docs.microsoft.com/azure/azure-sql/database/security-best-practice)
- [Microsoft Defender for SQL Documentation](https://docs.microsoft.com/azure/defender-for-cloud/defender-for-sql-introduction)
- [Data Classification in Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/data-discovery-and-classification-overview)

---

## Appendix

### A. ARM Template Parameters

Full list of customizable parameters in `azuredeploy.json`:

```json
{
  "serverName": "sqlserver-whizlabs",
  "databaseName": "AdventureWorksLT",
  "location": "eastus",
  "administratorLogin": "sqladmin",
  "administratorLoginPassword": "[secure-string]"
}
```

### B. Security Control Reference

| Control | Status | Configuration Location |
|---------|--------|----------------------|
| TDE | ✓ Enabled | Automatic |
| TLS 1.2 | ✓ Enforced | Automatic |
| Defender for SQL | ✓ Configured | SQL Server → Security → Defender |
| Data Classification | ✓ Configured | Database → Security → Classification |
| Auditing | ✓ Enabled | SQL Server → Security → Auditing |
| Vulnerability Assessment | ✓ Scheduled | Defender for Cloud → Recommendations |

### C. Compliance Mapping

| Requirement | Implementation | Evidence |
|------------|----------------|----------|
| GDPR - Data Inventory | Data Classification | 4 classified columns |
| GDPR - Encryption | TDE + TLS 1.2 | Default enabled |
| GDPR - Audit Trail | SQL Auditing | 5-day retention |
| HIPAA - Access Control | SQL Authentication | Server-level |
| PCI-DSS - Monitoring | Defender for SQL | Real-time alerts |
| SOC 2 - Logging | Audit logs | Comprehensive tracking |

### D. Useful SQL Queries

**List all classified columns:**
```sql
SELECT 
    SCHEMA_NAME(t.schema_id) AS SchemaName,
    t.name AS TableName,
    c.name AS ColumnName,
    sl.name AS SensitivityLabel,
    it.name AS InformationType
FROM sys.sensitivity_classifications sc
JOIN sys.tables t ON sc.major_id = t.object_id
JOIN sys.columns c ON sc.major_id = c.object_id AND sc.minor_id = c.column_id
JOIN sys.sensitivity_labels sl ON sc.label_id = sl.label_id
JOIN sys.information_types it ON sc.information_type_id = it.information_type_id;
```

**Check database encryption status:**
```sql
SELECT 
    DB_NAME(database_id) AS DatabaseName,
    encryption_state,
    CASE encryption_state
        WHEN 0 THEN 'No encryption'
        WHEN 1 THEN 'Unencrypted'
        WHEN 2 THEN 'Encryption in progress'
        WHEN 3 THEN 'Encrypted'
        WHEN 4 THEN 'Key change in progress'
        WHEN 5 THEN 'Decryption in progress'
    END AS EncryptionStatus
FROM sys.dm_database_encryption_keys;
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | December 2024 | Praise Effiom | Initial release |

---

**End of Technical Implementation Guide**

For questions or feedback, contact:  
**Praise Effiom** - Cloud Security Engineer  
LinkedIn: https://linkedin.com/in/praiseeffiom