# Phase 1: Detection & Evidence Preservation

## Objective
Identify the scope of the IAM compromise and secure forensic evidence without alerting the adversary.

## Overview
The first phase of incident response is critical—we must understand the breach's scope while preserving evidence for forensic analysis and regulatory compliance. In a cloud environment, the priority is to stop the 'bleeding' by identifying exactly which credentials have been leaked, then preserve the state of EKS pods and RDS logs before taking any destructive actions.

## Key Principles

### 1. Forensic Integrity is Paramount
- Every action taken during investigation must be logged and documented
- Preserve original evidence in an immutable state
- Use read-only snapshots and backups for analysis
- Maintain chain of custody for legal admissibility

### 2. Minimize Adversary Awareness
- Avoid triggering additional malware execution
- Don't alert the attacker that we've detected the compromise
- Use out-of-band investigation channels
- Implement stealthy monitoring without changing system states

### 3. Prioritize Critical Infrastructure
- Focus first on payment processing systems
- Secure secrets and credential stores immediately
- Protect customer data at all costs
- Ensure compliance systems remain intact

## Investigation Steps

### Step 1: Establish Secure Investigation Environment

**Objective:** Create an isolated workspace for forensic analysis

**Actions:**
1. Create a dedicated forensic analysis IAM role with read-only permissions
2. Set up a CloudTrail log analysis workstation in an isolated VPC
3. Enable VPC Flow Logs to capture network traffic metadata
4. Configure Splunk to aggregate all logs in real-time
5. Document all investigative IAM roles and permissions

**Deliverable:**
```yaml
# Document your forensic environment setup
- Forensic Analysis IAM Role ARN: ___________________
- Isolated VPC CIDR Block: ___________________
- CloudTrail Bucket Location: ___________________
- Splunk Instance Details: ___________________
```

### Step 2: Identify Compromised IAM Entities

**Objective:** Determine which IAM users, roles, and access keys have been compromised

**Investigation Approach:**
1. Extract CloudTrail logs covering the 48-hour period before detection
2. Identify all API calls from the suspicious IP addresses in Eastern Europe
3. Cross-reference with legitimate developer patterns
4. Document each compromised IAM entity and its access scope

**Key CloudTrail Events to Investigate:**
- `AssumeRole` - Lateral movement between accounts/roles
- `GetSecretValue` - Secrets Manager access
- `CreateAccessKey` - Creation of backdoor credentials
- `ModifyDBInstance` - Changes to database configurations
- `PutBucketPolicy` - S3 permission modifications
- `CreateNetworkInterface` - Network persistence attempts

**Deliverable: CloudTrail Analysis Report**

Create a comprehensive report including:
```markdown
## CloudTrail Event Analysis Report

### Compromised IAM Entities
- **IAM User:** developer-workstation-user
  - Access Key 1: AKIA...[REDACTED]
  - First Suspicious Activity: [TIMESTAMP]
  - Last Activity: [TIMESTAMP]
  - API Calls Made: [COUNT]
  - Resources Accessed: [LIST]

### Lateral Movement Timeline
1. **[TIMESTAMP]** - Initial compromise via phishing
2. **[TIMESTAMP]** - Credential theft from developer machine
3. **[TIMESTAMP]** - AssumeRole to production IAM role
4. **[TIMESTAMP]** - Access to Secrets Manager in production
5. **[TIMESTAMP]** - Database reconnaissance queries

### Affected Resources
- S3 Buckets Accessed: [LIST WITH RECORD COUNTS]
- RDS Instances Queried: [LIST]
- EKS Clusters Affected: [LIST]
- Secrets in Secrets Manager: [LIST]
```

### Step 3: Preserve EKS Worker Node Evidence

**Objective:** Capture forensic snapshots of affected Kubernetes infrastructure

**Actions:**
1. Identify which EKS worker nodes processed traffic from compromised accounts
2. Create EBS snapshots of worker node volumes before any cleanup
3. Capture container logs and audit trails
4. Document network traffic patterns between pods

**Deliverable: EKS Forensic Snapshot Inventory**

```yaml
EKS_Forensic_Snapshots:
  - Cluster_Name: production-payment-cluster
    Worker_Node_ID: i-0a1b2c3d4e5f6g7h8
    EBS_Volume_ID: vol-0x1y2z3a4b5c6d7e8f
    Snapshot_ID: snap-0abc123def456ghi
    Snapshot_Time: 2026-06-10T03:45:00Z
    Image_IPFS_Hash: QmABC123...
    Container_Logs_Preserved: true
    Network_Policies_Captured: true
    
  - Cluster_Name: production-payment-cluster
    Worker_Node_ID: i-1b2c3d4e5f6g7h8i9
    EBS_Volume_ID: vol-1x2y3z4a5b6c7d8e9f
    Snapshot_ID: snap-1bcd234efg567hij0
    Snapshot_Time: 2026-06-10T03:50:00Z
    Image_IPFS_Hash: QmDEF456...
    Container_Logs_Preserved: true
    Network_Policies_Captured: true
```

### Step 4: Preserve RDS Database Evidence

**Objective:** Create forensic snapshots of database state without performing destructive operations

**Actions:**
1. Create automated RDS snapshots of all affected instances
2. Enable enhanced monitoring and performance insights
3. Enable database audit logging
4. Preserve database activity streams
5. Document all SQL queries executed by compromised accounts

**Important:** Do NOT reboot RDS instances or perform hard resets, as this clears volatile memory evidence

**Deliverable: RDS Forensic Snapshot Inventory**

```yaml
RDS_Forensic_Snapshots:
  - Instance_Identifier: nflo-prod-primary-db
    Engine: PostgreSQL
    Snapshot_ID: rds:nflo-prod-primary-db-2026-06-10-03-40
    Snapshot_Time: 2026-06-10T03:40:00Z
    Backup_Type: Snapshot
    Size_GB: 250
    Audit_Logs_Enabled: true
    Activity_Stream_Enabled: true
    
    Suspicious_Queries_Detected:
      - Query: "SELECT * FROM customers WHERE balance > 100000"
        Timestamp: 2026-06-10T02:45:30Z
        Source_IAM_Role: arn:aws:iam::ACCOUNT:role/compromised-role
        Row_Count: 12450
        
      - Query: "SELECT * FROM transactions WHERE status='pending'"
        Timestamp: 2026-06-10T02:48:15Z
        Source_IAM_Role: arn:aws:iam::ACCOUNT:role/compromised-role
        Row_Count: 3827
```

### Step 5: Centralized Investigation Timeline in Splunk

**Objective:** Correlate all evidence into a single timeline for analysis

**Actions:**
1. Ingest all CloudTrail logs into Splunk
2. Correlate with VPC Flow Logs
3. Add Okta/identity provider login logs
4. Include application-level audit logs
5. Create visualization of attacker's kill chain

**Deliverable: Splunk Investigation Dashboard**

Create a Splunk search pattern that shows:
```splunk
index=cloudtrail OR index=vpc_flow OR index=okta
| transaction source_ip, user_name
| stats count, min(_time), max(_time) by source_ip, user_name, action
| sort - count
```

Visualize:
- Timeline of first suspicious activity to detection
- Geographic distribution of attacker IP addresses
- Progression of privilege escalation attempts
- Resources accessed before and after privilege escalation

### Step 6: Inventory Accessed S3 Buckets and RDS Instances

**Objective:** Determine what data was accessed or potentially exfiltrated

**Actions:**
1. Enable S3 Object Lock for critical buckets (retrospective)
2. Review S3 access logs for suspicious downloads
3. Analyze database query logs for mass data extraction
4. Use S3 inventory to identify modified objects
5. Check for unusual data transfer sizes

**Deliverable: Data Exposure Assessment Report**

```markdown
## Data Exposure Assessment

### S3 Buckets Accessed

| Bucket Name | Purpose | Objects Accessed | Suspicious Activity | PII Exposed |
|---|---|---|---|---|
| nflo-prod-transactions | Transaction records | 1,247 | Yes - bulk download attempt | No |
| nflo-prod-customers | Customer profiles | 845 | Yes - search queries | Partial |
| nflo-prod-secrets-backup | Secrets backup | 12 | Yes - all accessed | Yes |

### RDS Instances Accessed

| Instance | Database | Queries Executed | Rows Selected | Rows Modified | PII Risk |
|---|---|---|---|---|---|
| nflo-prod-primary-db | nflo_prod | 847 | 127,482 | 0 | High |
| nflo-staging-db | nflo_staging | 12 | 4,521 | 0 | Low |
| nflo-analytics-db | nflo_analytics | 3 | 234 | 0 | Medium |

### Conclusion
- **No customer PII confirmed exfiltrated** ✓
- **No database modifications detected** ✓
- **No unauthorized code deployments** ✓
```

## Constraints & Requirements

### NIST SP 800-61 Framework Compliance
- All actions must align with NIST Incident Response procedures
- Document evidence chain from collection through analysis
- Prepare for potential law enforcement cooperation
- Maintain separate evidence storage and analysis systems

### PCI-DSS Evidence Requirements
- Preserve evidence in forensic-ready state to support potential legal action
- Maintain 90-day minimum retention of all audit logs
- Document all access to cardholder data
- Prove no unauthorized data exfiltration occurred

### Forensic Chain of Custody
- Document who accessed evidence and when
- Use cryptographic hashing (SHA-256) for evidence integrity
- Maintain immutable audit logs of all investigative actions
- Separate forensic evidence from operational systems

## Guidance & Tips

### DO ✓
- Use Splunk to correlate CloudTrail events with Okta login logs to identify when the attacker authenticated
- Create EBS/RDS snapshots IMMEDIATELY to preserve volatile evidence
- Document every AWS CLI command in audit trail for final report
- Leverage Okta's immutable audit logs for authentication timeline
- Treat the forensic environment as critical infrastructure
- Use read-only IAM roles for all investigative actions

### DON'T ✗
- Do NOT delete the compromised IAM user until forensic logs are fully exported
- Do NOT perform a 'hard reboot' of RDS instances as this clears volatile memory evidence
- Do NOT make manual changes in the AWS Console; use Terraform/CLI for auditability
- Do NOT ignore the staging environment—attackers often stage operations there first
- Do NOT assume the breach is contained after initial detection

## Success Criteria for Phase 1

You have successfully completed Phase 1 when:

✓ CloudTrail analysis identifies all compromised IAM entities with timestamps  
✓ EKS worker node forensic snapshots are preserved and documented  
✓ RDS database snapshots capture the incident state  
✓ Splunk investigation timeline clearly shows the attack progression  
✓ Data exposure assessment confirms (or denies) PII exfiltration  
✓ All evidence maintains forensic chain of custody  
✓ Zero customer PII confirmed exfiltrated  

---

## Next Steps

Once Phase 1 is complete, proceed to **Phase 2: Containment & Eradication** to neutralize the adversary's access.
